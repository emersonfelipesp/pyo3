# src/sync/ — GIL-Aware Synchronization Primitives

Provides synchronization types that are safe to use with the Python GIL. Naive
use of `std::sync::Mutex` or `OnceLock` while holding the GIL can cause
deadlocks if another thread needs the GIL to finish its work. The types here
release (detach from) the interpreter before blocking, then reacquire it.

All items are re-exported from `src/sync.rs` at the top level.

## Files

| File | Size | Description |
|---|---|---|
| `once_lock.rs` | 8K | `PyOnceLock<T>` — GIL-aware equivalent of `std::sync::OnceLock`; releases the interpreter before blocking on initialization, re-attaches after |
| `critical_section.rs` | 24K | `PyCriticalSection<'py>`, `PyCriticalSection2<'py>` — wrappers for CPython's `Py_BEGIN_ALLOW_THREADS` / `Py_END_ALLOW_THREADS` critical section macros; used to protect per-object locks in free-threaded builds |

## Legacy Types (in `src/sync.rs`)

`src/sync.rs` (top-level, 40K) also contains:

- `GILOnceCell<T>` — older `OnceLock` analog that required a `Python<'py>` token to initialize; superseded by `PyOnceLock`.
- `GILProtected<T>` — wraps a `T` and requires `Python<'py>` to access it.
- `GILMutex<T>` — deprecated; use `critical_section.rs` types instead.

## When to Use

- **`PyOnceLock`**: use for global/static Python object initialization (e.g.,
  interning a string, storing a shared `Py<PyList>`).
- **`PyCriticalSection`**: use in free-threaded Python builds to protect access
  to a `PyObject`'s contents without holding the GIL globally.

## Relationships

- `src/sync.rs` re-exports everything from this directory.
- `src/internal/state.rs` provides `SuspendAttach` (release/reacquire helper)
  that `once_lock.rs` uses.
- `src/types/mutex.rs` provides `PyMutex` (Python 3.13+'s built-in mutex),
  which is distinct from the Rust-side primitives here.
