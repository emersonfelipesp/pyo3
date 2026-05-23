# src/coroutine/ — Async/Await Bridge

Bridges Rust `Future`s with Python coroutines. When a `#[pyfunction]` or
`#[pymethods]` method returns a `Future`, PyO3 wraps it in a `Coroutine`
Python object that implements Python's `__await__` protocol. This lets Python
`await` Rust async functions directly.

## Files

| File | Size | Description |
|---|---|---|
| `cancel.rs` | 2K | `CancelHandle` — allows a Python coroutine to propagate a cancellation exception into the Rust `Future`; backed by `Arc<Mutex<Inner>>` holding the pending exception and a `Waker` |
| `waker.rs` | 4K | `AsyncioWaker` / waker implementation — converts a Python event loop's scheduling mechanism into a Rust `Waker` so that `Future::poll` is driven by the Python event loop when the future is ready to make progress |

## Public Surface

The `Coroutine` struct itself is defined in `src/coroutine.rs` (top-level,
not this subdirectory). That file re-exports the types from this directory.

## How It Works

1. `#[pyfunction] async fn foo() -> PyResult<T>` is wrapped by the proc-macro
   into a synchronous function that returns a `Coroutine`.
2. `Coroutine` holds the Rust `Future` and drives it by implementing Python's
   `send` / `throw` / `close` protocol.
3. When the future is pending, `Coroutine.send()` returns a Python awaitable
   that will reschedule the coroutine when ready. The `waker.rs` waker
   connects the Rust `Waker` to the Python event loop callback.
4. A `CancelHandle` can be embedded in the Rust future to observe a Python
   `throw(CancelledError)` call and propagate it as a future cancellation.

## Relationships

- `src/coroutine.rs` (top-level) re-exports `CancelHandle` from `cancel.rs`.
- The waker in `waker.rs` calls into the Python `asyncio` event loop via
  `src/types/` (specifically using `PyAny` call methods).
- Used by `pyo3-macros-backend` when it detects an `async fn` in
  `#[pyfunction]` / `#[pymethods]` expansion.
