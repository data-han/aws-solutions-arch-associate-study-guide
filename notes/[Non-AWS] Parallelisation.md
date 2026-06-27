# Python Parallelism, in Practice

A working explanation of threads, processes, pools, joblib/loky, and the thread+lock
pattern — written against a real pandas pipeline (the Layer 1 Sales Gap Analysis), so
the concepts stay concrete.

---

## The one fact everything hinges on: the GIL

Python has a **Global Interpreter Lock**. Inside a single process, only one thread runs Python bytecode at a time. Threads take turns; they do not execute Python in parallel.

This single constraint decides every parallelism choice:

- **CPU work** — pandas loops, a per-group `apply`, numpy math written in Python. Threads cannot speed this up. They serialize on the GIL.
- **I/O work** — a SQL query, a file read, a network call. While a thread waits on the database, it *releases* the GIL, so another thread runs. Threads genuinely help here.

Rule of thumb: **I/O-bound work → threads. CPU-bound work → processes.**

---

## Threads vs processes

### Threads
Multiple lines of execution inside *one* process, sharing the same memory.

- Cheap to start. Share data for free (same memory space).
- GIL-bound for CPU — no speedup on computation.
- Good for overlapping I/O.

In the pipeline: the country-level `ThreadPoolExecutor` overlaps each country's SQL
loads. Correct use of threads. It does **not** help the heavy `groupby`/`apply` math —
that work is CPU-bound and stays stuck behind the GIL.

### Processes
Multiple *separate* Python interpreters, each with its own memory and its own GIL.

- True parallel CPU: 4 processes = 4 cores actually computing at once.
- Cost: starting them is heavy, and any data sent to/from them must be **serialized**
  (pickled). Large DataFrames mean copy time and extra memory.

In the pipeline: `parallel_groupby_apply` uses processes to split millions of groups
across cores.

**The trade in one line:** threads share memory (cheap data, no CPU parallelism);
processes copy memory (CPU parallelism, but a pickling tax).

| | Threads | Processes |
|---|---|---|
| Memory | shared | separate (copied via pickle) |
| CPU parallelism | no (GIL) | yes |
| I/O parallelism | yes | yes |
| Startup cost | cheap | heavy |
| Data transfer | free | pickled in/out |
| Use for | waiting (I/O) | computing (CPU) |

---

## Pool / Executor

A **pool** is a reusable set of pre-created workers you hand tasks to, instead of
spawning a fresh worker per task (expensive, especially for processes).

- `ThreadPoolExecutor(max_workers=5)` — 5 worker *threads*, reused across tasks.
- `ProcessPoolExecutor` — the same idea with *processes*.

---

## joblib, backend, loky

**joblib** wraps all of the above behind one API:

```python
from joblib import Parallel, delayed
results = Parallel(n_jobs=4, backend="loky")(delayed(func)(x) for x in items)
```

**backend** selects the engine:

- `"threading"` — thread pool. For I/O, or work that releases the GIL.
- `"loky"` — process pool. For CPU work. **joblib's default.**

**loky** is a hardened process-pool implementation joblib ships. Think "sturdier
`ProcessPoolExecutor`":

- handles crashed workers,
- pickles closures / lambdas / nested functions via **cloudpickle** (plain `pickle`,
  used by the `multiprocessing` backend, cannot — this is why our nested
  `_calc_expected_sales` closure works under loky),
- keeps **one reusable executor per process**, so repeated `Parallel` calls don't
  re-spawn workers.

That last property — *one shared executor per process* — is the source of the bug below.

**`n_jobs`** = how many workers.

---

## `delayed` and pickling — what actually crosses the process boundary

`delayed(func)(x)` does **not** run `func`. It packages it — roughly `(func, (x,), {})`,
"a function plus the args to call it with, later." Lazy.

`Parallel(...)(...)` then, per item:

1. picks a worker process,
2. **pickles** the function + its args to bytes,
3. ships the bytes over a pipe,
4. the worker unpickles, runs `func(x)`, pickles the **return value**, ships it back,
5. the parent unpickles results, returns them in order.

Why pickling matters:

- **Copy cost** — each argument DataFrame is serialized and copied into the worker.
- **Memory** — N workers each hold a chunk. Parent + N copies.
- **Picklability** — closures need cloudpickle (loky has it).
- **No shared state** — workers can't see or mutate the parent's variables. Everything
  in by value, everything out by value.

---

## The bug: nested loky inside threads (and why the GIL does NOT save you)

Our pipeline nests the two layers:

```
country loop -> ThreadPoolExecutor          (threads, OUTER)
   each country runs in a thread
      -> parallel_groupby_apply              (loky processes, INNER)
```

loky keeps **one shared executor per process**. When two country threads each call loky,
they both reach for that *same* executor. One thread finishes and tears down / resizes it
while another is mid-flight →

```
cannot schedule new futures after shutdown
```

The natural assumption is "the GIL serializes threads, so this can't happen." It does
happen, because the dangerous moment is exactly when the GIL is *released*.

### Step-by-step (no lock)

```
Thread A: holds GIL, runs Python, reaches its loky call
Thread A: launches worker processes, then WAITS for them
            -> waiting on subprocesses is I/O-like -> GIL RELEASED here
                                                          |
Thread B: GIL is now free, so B gets scheduled, runs Python
Thread B: reaches its OWN loky call, starts driving the SAME shared executor
                                                          |
   >>> now BOTH threads have live loky sessions on the one shared executor <<<
                                                          |
Thread A: its workers finish -> A tears down / resizes the shared executor
                                                          |
Thread B: was mid-flight on that executor -> "cannot schedule new futures after shutdown"  ✗
```

Key point: **the GIL released precisely during the part that races** (waiting on the
process pool). So the GIL provides no protection here. The race is on loky's shared
executor, not on Python bytecode.

The GIL and this error are unrelated problems:
- **GIL** → why threads don't speed up CPU work.
- **loky shared-executor race** → the `cannot schedule` error.

### What the old code did about it

It dodged the race by forcing `n_jobs=1` whenever the call came from a worker thread —
i.e. it refused to use processes at all. Safe, but the CPU-heavy groupby then ran
**single-core**. That was the performance bug.

---

## The fix: a lock around loky calls

```python
import threading
_LOKY_CALL_LOCK = threading.Lock()

# inside parallel_groupby_apply:
if backend == "loky":
    with _LOKY_CALL_LOCK:
        results = run_parallel()    # one loky session at a time, but multi-core
else:
    results = run_parallel()        # threading backend is GIL-bound, safe unlocked
```

### Step-by-step (with lock)

```
Thread A: acquire _LOKY_CALL_LOCK
Thread A: run loky -> uses ALL allotted cores -> finish -> release lock
                                                          |
Thread B: tried to acquire while A held it -> BLOCKED (parks, no work)
Thread B: A released -> B acquires -> run loky -> uses ALL cores -> release
```

Only one thread is ever inside a loky call, so the two threads never touch the shared
executor at the same time. Race gone. Each loky call still uses every core; the calls
just queue instead of overlapping.

### "Doesn't queuing kill the parallelism?"

No, for two reasons:

1. **Each loky call already uses all cores.** Two loky sessions running "at once" would
   only fight over the same cores — no faster. Serializing them loses nothing on CPU
   throughput.
2. **The lock only covers the loky (CPU) call.** A thread doing a SQL `load` is not
   holding the lock, so loads from other countries still overlap. You keep I/O
   parallelism across countries *and* get real multi-core on each groupby.

### Before vs after

| | Old code | After (lock) |
|---|---|---|
| Country threads | yes | yes |
| loky inside a thread | forced `n_jobs=1` (single-core) | runs, all cores, serialized by lock |
| The race | avoided by killing parallelism | avoided by ordering the calls |
| CPU groupby speed | slow (1 core) | fast (N cores) |
| I/O overlap across countries | yes | yes |

---

## How loky "uses all cores" — and the k8s `cpu_count` trap

`n_jobs` decides how many worker processes (cores) a loky call uses. So "use all cores"
means sizing `n_jobs` to the cores you're actually allowed.

The trap: **`os.cpu_count()` / `mp.cpu_count()` report the NODE's cores, not the pod's
CPU limit.**

```
Node has 64 cores.   Pod limit: cpu: 16.
os.cpu_count()  -> 64     (the node — WRONG for sizing the pool)
real quota      -> 16     (the cgroup limit)
```

Size a pool off `cpu_count()` and you launch 64 workers for a 16-core allowance → the
kernel cgroup **throttles** you → context-switch thrash → slower than 16. This is why
the old code hardcoded a safe small constant (4): it never over-subscribed, but it also
left most of a 16-core box idle.

The fix is to read the **cgroup quota** — the pod's real allowance — instead of the node
count:

```
/sys/fs/cgroup/cpu.max  ->  "1600000 100000"  ->  1600000 / 100000 = 16 cores
```

`usable_core_count(buffer=2)` does this (cgroup v2 → v1 → CPU affinity → `cpu_count`
fallback), then subtracts a buffer so the parent + any I/O threads still get CPU. On the
current pinned `cpu: 16` container that yields **14** workers, vs the old **4**.

So with the lock in place, one loky call at a time runs across 14 cores — the whole box
is actually used, and there's no over-subscription because only one loky session exists
at any moment.

(If the pod size is fixed, hardcoding `n_cores = 14` is an equally correct, simpler
alternative — the cgroup detector just auto-adapts if the limit ever changes.)

---

## Where to put the parallelism: match the skew

A natural idea: "if it's CPU-heavy, just run the *countries* as processes." For this data
it doesn't help, because the work is **lopsided** — one country (e.g. FR ≈ 1.75M groups)
holds almost all the work; the others are tiny.

- **Processes at the country level** → the fat country gets *one* core and still crawls;
  the other cores finish their tiny countries and sit idle. The bottleneck doesn't move.
- **Processes inside the country** (loky over its groups) → the fat country's groupby
  splits across all cores. The bottleneck actually shrinks.

**Rule: parallelize at the level where the work concentrates.** Here that's *groups
within one country*, not *countries*. The country layer stays threads, purely for I/O
overlap.

Do **not** nest process pools (country `ProcessPoolExecutor` *and* inner loky): you'd
pickle whole-country DataFrames, run fragile pool-in-pool, and multiply memory by
countries × workers. Worst option.

---

## Reading a log: is a step I/O-bound or CPU-bound?

**1. What the timed operation is.**

```
load took 67.09 seconds          -> SQL query        -> I/O-bound
save took  8.19 seconds          -> SQL write        -> I/O-bound
calculate_expected_sales 616 s   -> pandas crunching -> CPU-bound
```

`load`/`save` are DB round-trips (I/O). `calculate_*`, `_calc_*`, per-group `apply` are
computation (CPU).

**2. Did threads already help it?** Several "Loading…" lines from different worker
threads interleaving at the same timestamps and finishing together = I/O (threads
overlapped it). A compute step that shows no benefit under threads = CPU-bound (GIL).

**3. The decisive test — throw cores at it.** Speeds up ~N× with N processes → it was
CPU-bound. No change → I/O-bound (bottleneck is the DB/network; the fix is
concurrency/threads or a better query, not more processes).

| Log line | Kind | Right lever |
|---|---|---|
| `load took N seconds` | I/O | threads / better SQL / fewer queries |
| `save took N seconds` | I/O | batch writes |
| `calculate_expected_sales` | CPU | vectorize, or processes |
| `_calc_confidence_index_vio` | CPU | processes |

---

## One more nuance: distributing work isn't free

A process pool pays a tax — pickling and copying each chunk to workers and back. When the
per-item work is tiny, that tax can dominate and you see *less* than N× speedup.

That is why **vectorizing** a hot `groupby`/`apply` often beats **parallelizing** it:
vectorizing removes the work (one bulk operation over the whole frame, in C) instead of
paying to spread millions of tiny Python calls across processes. Reach for processes when
the per-item work is genuinely heavy and can't be vectorized.

In this pipeline that played out exactly: the VIO `calculate_expected_sales` was
*vectorized* (work removed, ~600s → ~1s), while the gnarlier paths that resist
vectorization were *parallelized* via the loky lock instead.

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
| the race | Concurrent loky from threads share one executor → "cannot schedule new futures". |
| the lock | Serializes loky calls so they never race; each still uses all cores. |
| cpu_count trap | Reports node cores, not the pod limit; size from the cgroup quota instead. |

**The whole thing in three sentences:** If the slow part is *waiting* (network, disk),
use threads. If it's *computing*, either vectorize it away or run it across processes —
and if those processes get launched from inside threads, serialize the launches with a
lock so they don't race on loky's shared executor. Size the pool from the pod's cgroup
limit, not `cpu_count()`, and put the parallelism where the work actually concentrates.
