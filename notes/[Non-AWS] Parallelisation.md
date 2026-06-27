# Python Parallelism, in Practice

A working explanation of threads, processes, pools, and joblib/loky — written against a real pandas pipeline (the Layer 1 Sales Gap Analysis), so the concepts stay concrete.

---

## The one fact everything hinges on: the GIL

Python has a **Global Interpreter Lock**. Inside a single process, only one thread runs Python bytecode at a time. Threads take turns; they don't execute Python in parallel.

This single constraint decides every parallelism choice:

- **CPU work** — pandas loops, a per-group `apply`, numpy math written in Python. Threads can't speed this up. They serialize on the GIL.
- **I/O work** — a SQL query, a file read, a network call. While a thread waits on the database, it *releases* the GIL, so another thread runs. Threads genuinely help here.

Rule of thumb: **I/O-bound work → threads. CPU-bound work → processes.**

---

## Multithreading vs multiprocessing

### Threads
Multiple lines of execution inside *one* process, sharing the same memory.

- Cheap to start. Share data for free (same memory space).
- GIL-bound for CPU — no speedup on computation.
- Good for overlapping I/O.

In the pipeline: the country-level `ThreadPoolExecutor` exists to overlap each country's SQL loads. That's a correct use of threads. It does **not** help the heavy `groupby`/`apply` math — that work is CPU-bound and stays stuck behind the GIL.

### Processes
Multiple *separate* Python interpreters, each with its own memory and its own GIL.

- True parallel CPU: 4 processes = 4 cores actually computing at once.
- Cost: starting them is heavy, and any data sent to/from them must be **serialized** (pickled). Large DataFrames mean copy time and extra memory.

In the pipeline: `parallel_groupby_apply` uses processes to split millions of groups across cores. That's the right tool for the CPU bottleneck.

**The trade in one line:** threads share memory (cheap data, no CPU parallelism); processes copy memory (CPU parallelism, but a pickling tax).

---

## "Pool" / Executor

A **pool** is a reusable set of pre-created workers you hand tasks to, instead of spawning a fresh worker per task (which is expensive, especially for processes).

- `ThreadPoolExecutor(max_workers=5)` — 5 worker *threads*, reused across tasks.
- `ProcessPoolExecutor` — the same idea with *processes*.

"Pool" just means: keep N workers alive and feed them work.

---

## joblib, backend, loky

**joblib** is a library that wraps all of the above behind one API:

```python
from joblib import Parallel, delayed
results = Parallel(n_jobs=4, backend="loky")(delayed(func)(x) for x in items)
```

**backend** selects the engine joblib runs the work on:

- `"threading"` — a thread pool. For I/O, or work that releases the GIL.
- `"loky"` — a process pool. For CPU work. **This is joblib's default.**
- `"multiprocessing"` — an older process backend.

**loky** is a specific, hardened process-pool implementation that joblib ships. Think of it as a sturdier `ProcessPoolExecutor`:

- It handles crashed workers.
- It pickles tricky objects — lambdas, closures, locally-defined functions — using **cloudpickle**. (Standard `pickle`, used by the plain `multiprocessing` backend, cannot pickle a closure. This matters below.)
- It keeps **one reusable executor per process**, so repeated `Parallel` calls don't re-spawn workers each time.

That last property is the source of a subtle bug — see the lock section.

**`n_jobs`** is simply how many workers to use.

---

## `delayed` and pickling — what actually crosses the process boundary

```python
Parallel(n_jobs=4, backend="loky")(delayed(func)(x) for x in items)
```

**`delayed(func)(x)`** does **not** run `func`. It packages it — roughly `(func, (x,), {})`, "a function plus the args to call it with, later." Lazy. The generator yields one package per item.

**`Parallel(...)(...)`** then:

1. Starts or reuses N worker processes.
2. **Pickles** each package — the function *and* its arguments — into bytes.
3. Ships the bytes to a worker over a pipe.
4. The worker **unpickles**, runs `func(x)`, pickles the **return value**, ships it back.
5. The parent unpickles results and returns them in order.

### Why pickling is the thing to watch

- **Copy cost.** Each argument DataFrame is serialized and copied into the worker. Big frames cost real time and memory. (joblib's `max_nbytes` setting tells it to memory-map arrays above a threshold instead of copying them — an optimization for large data.)
- **Memory.** N workers each hold their chunk. Parent plus N copies. This is the main risk when running process pools in a memory-capped container.
- **Picklability.** The function must serialize. loky's use of cloudpickle is what lets closures and nested functions run in workers; the plain `multiprocessing` backend would fail on them.
- **No shared state.** Workers can't see or mutate the parent's variables. Everything goes in by value and comes out by value. (Threads are the opposite: shared memory, no pickling, but GIL-bound.)

---

## A real gotcha: loky + threads racing (and the lock that fixes it)

loky keeps **one shared executor per process**. If you call a loky-backed `Parallel` from *multiple threads at once*, they all try to drive that one shared executor. One thread finishes and tears down or resizes the executor while another is mid-flight, and you get:

```
cannot schedule new futures after shutdown
```

This is exactly what happens when a process-pool call sits *inside* a thread pool (a CPU `groupby` running inside the per-country threads).

Two ways to handle it:

- **Force serial** — disable the process pool whenever called from a worker thread. Safe, but you lose all CPU parallelism (the work runs single-core).
- **Serialize the loky calls with a lock** — let the process pool run, but allow only one loky call at a time across all threads:

```python
import threading
_LOKY_CALL_LOCK = threading.Lock()

# ... inside the helper ...
if backend == "loky":
    with _LOKY_CALL_LOCK:
        results = run_parallel()   # one loky session at a time, but multi-core
else:
    results = run_parallel()       # threading backend is GIL-bound, safe unlocked
```

Each loky call still uses all cores; the calls just queue instead of racing. Meanwhile other threads that aren't running a loky call don't hold the lock, so they keep overlapping their I/O. You get CPU parallelism **and** I/O overlap, without the race.

---

## How it all stacks in the pipeline

```
country ThreadPoolExecutor          threads   -> overlaps per-country SQL loads (I/O)
  └─ per country: sales-gap CPU work
       └─ parallel_groupby_apply     joblib Parallel(backend="loky")
                                     processes -> real multi-core for the groupby
                                     guarded by _LOKY_CALL_LOCK so threads don't race
```

Two distinct levers, two distinct problems:

- Threads at the country level → overlap the **waiting** (database I/O).
- Processes inside each country → parallelize the **computing** (pandas groupby).

---

## Reading a log: is a step I/O-bound or CPU-bound?

**1. Look at what the timed operation is.**

```
load took 67.09 seconds            -> SQL query        -> I/O-bound
save took  8.19 seconds            -> SQL write        -> I/O-bound
calculate_expected_sales 616 s     -> pandas crunching -> CPU-bound
```

`load`/`save` are database round-trips (I/O). `calculate_*`, `_calc_*`, and anything doing a per-group `apply` are computation (CPU).

**2. Check whether threads already helped it.** If several "Loading…" lines from different worker threads interleave at the same timestamps and finish faster together, that work is I/O — threads overlapped it. If a compute step shows no benefit from running under threads, it's CPU-bound (GIL).

**3. Wall-clock intuition.** A long `load` where your CPU is idle and the database is busy is I/O. A long compute step pinning a core at 100% is CPU.

**4. The decisive test — throw cores at it.**
- Speeds up roughly N× with N processes → it was CPU-bound.
- No change → I/O-bound: the bottleneck is the database/network, not your cores. The fix there is concurrency/threads or a better query, not more processes.

| Log line | Kind | Right lever |
|---|---|---|
| `load took N seconds` | I/O | threads / better SQL / fewer queries |
| `save took N seconds` | I/O | batch writes |
| `calculate_expected_sales` | CPU | vectorize, or processes |
| `_calc_confidence_index_vio` | CPU | processes |

---

## One more nuance: distributing work isn't free

A process pool pays a tax — pickling and copying each chunk to workers and back. When the per-item work is tiny, that tax can dominate, and you see *less* than N× speedup.

That is why **vectorizing** a hot `groupby`/`apply` often beats **parallelizing** it: vectorizing removes the work (one bulk operation over the whole frame, in C) instead of paying to spread millions of tiny Python calls across processes. Reach for processes when the per-item work is genuinely heavy and can't be vectorized.

---

## Cheat sheet

| Term | One line |
|---|---|
| GIL | One thread runs Python at a time, per process. |
| thread | Shares memory, cheap, only helps I/O. |
| process | Own memory + own GIL, true CPU parallelism, pickling cost. |
| pool / Executor | Reusable set of workers fed tasks. |
| joblib | Library wrapping parallel execution. |
| backend | Engine choice: `threading` (threads) vs `loky` (processes). |
| loky | joblib's robust process pool; one shared executor per process; cloudpickle. |
| `delayed` | Packages a call to run later; doesn't execute it. |
| pickling | Serializing function + args to ship to a worker process. |
| `n_jobs` | How many workers. |

**The whole thing in two sentences:** If the slow part is *waiting* (network, disk), use threads. If it's *computing* (pandas, numpy, Python loops), either vectorize it away or run it across processes — and if those processes get launched from inside threads, serialize the launches so they don't race.
