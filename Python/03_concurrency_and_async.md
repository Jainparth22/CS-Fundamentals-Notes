# Python Concurrency, Async and Performance: Interview Notes

## 1. Concurrency vs parallelism
- **Concurrency:** dealing with many tasks at once (interleaving). **Parallelism:** executing many tasks at the same instant (multiple cores).
- Python options:

| Tool | Model | Best for | Beats the GIL? |
|---|---|---|---|
| `threading` | OS threads | I/O-bound (network, disk) | no, for CPU work |
| `multiprocessing` / `ProcessPoolExecutor` | separate processes | CPU-bound | yes |
| `asyncio` | single thread, cooperative tasks | many concurrent I/O connections | not needed for I/O |
| C extensions, NumPy, Numba | native code | numeric work | releases GIL |

## 2. Threads
```python
from concurrent.futures import ThreadPoolExecutor
with ThreadPoolExecutor(max_workers=8) as pool:
    results = list(pool.map(fetch, urls))
```
- Shared memory means race conditions. Protect shared state with `threading.Lock`, or use thread-safe `queue.Queue`.
- `+=` on a shared int is **not atomic** (read, add, write are separate bytecodes).
- Deadlock: two threads each hold a lock the other needs. Avoid by consistent lock order or timeouts.
- Daemon threads die when the main thread exits.

## 3. Processes
- Each process has its own interpreter and memory; communicate with `Queue`, `Pipe`, shared memory or by returning results.
- Arguments and results must be picklable.
- Start method: `fork` (Linux default historically), `spawn` (Windows, macOS default), `forkserver`. Guard entry point with `if __name__ == "__main__":` on Windows.

## 4. asyncio
```python
import asyncio, httpx

async def fetch(client, url):
    r = await client.get(url)      # yields control while waiting
    return r.status_code

async def main(urls):
    async with httpx.AsyncClient() as client:
        return await asyncio.gather(*(fetch(client, u) for u in urls))

asyncio.run(main(urls))
```
- `async def` defines a coroutine; calling it returns a coroutine object that does nothing until awaited or scheduled.
- `await` suspends the coroutine so the **event loop** can run others. One thread, cooperative multitasking.
- `asyncio.gather` runs awaitables concurrently; `asyncio.create_task` schedules in the background; `asyncio.TaskGroup` (3.11+) gives structured concurrency; `asyncio.wait_for(coro, timeout)` adds timeouts.
- **Never block the event loop.** `time.sleep`, blocking DB drivers, `requests` or heavy CPU code freeze every other task. Use `await asyncio.sleep`, async libraries, or `await loop.run_in_executor(None, blocking_fn)` / `asyncio.to_thread`.
- Async generators (`async for`), async context managers (`async with`).
- `asyncio.Semaphore` limits concurrency; `asyncio.Queue` for producer-consumer.

### async vs threads
- Async scales to thousands of connections with little memory; code needs async-aware libraries.
- Threads are simpler with blocking libraries but cost more per task.

## 5. The GIL again
- Only one thread runs bytecode at a time; switching happens every ~5 ms or on I/O.
- It does not make code thread-safe; compound operations still race.
- Workarounds: processes, native extensions, `asyncio` for I/O, or sub-interpreters (3.12+) and the free-threaded build (3.13+, experimental).

## 6. Performance tips
- Measure first: `timeit`, `cProfile`, `py-spy`, `line_profiler`, `tracemalloc`.
- Use the right data structure (set/dict for membership, deque for queues, `heapq`).
- Avoid repeated attribute lookups in hot loops; use local variables, comprehensions, built-ins (`sum`, `map`, `any`).
- Cache with `functools.lru_cache`; batch I/O; stream with generators.
- Vectorise with NumPy/pandas; JIT with Numba; Cython or Rust for hot spots.
- String concatenation: `''.join`. Avoid `list.insert(0, x)`; use `deque`.

## 7. Testing
- `pytest`: fixtures, parametrize, `monkeypatch`, `tmp_path`; `unittest.mock.patch`, `MagicMock`.
- Test concurrency bugs with repeated runs and threads (e.g. 50 threads trying to book the same slot and asserting one winner).
- `pytest-asyncio` or `anyio` for async tests.

## 8. Logging and config
- `logging` module: named loggers, levels (DEBUG to CRITICAL), handlers, formatters. Do not use `print` in services.
- Configuration via environment variables (12-factor), `pydantic-settings`, `.env` files.

## Quick interview Q&A
- **Threads or processes for a CPU-bound task?** Processes (or native code), because of the GIL.
- **When is asyncio better than threads?** Many concurrent I/O-bound connections with async libraries.
- **What blocks an event loop?** Any synchronous blocking call or long CPU computation inside a coroutine.
- **What is a race condition? Show one.** Two threads read-modify-write a shared counter and lose updates; fix with a lock, atomic operation or by avoiding shared state.
- **What is the difference between `gather` and `create_task`?** `gather` waits for a set of awaitables and returns results; `create_task` schedules one coroutine to run in the background.
- **What does `await` do?** Suspends the current coroutine until the awaited one finishes, letting the loop run others.
- **How do you run blocking code inside async code?** `asyncio.to_thread` or `run_in_executor`.
- **How would you speed up processing 1M records?** Profile; vectorise or batch; use multiprocessing for CPU; async/threads for I/O; avoid loading everything in memory (generators or chunks).
