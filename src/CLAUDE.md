# src/ — Main Crate Source Root

`src/lib.rs` is the public API surface. Everything exported from there is
the stable interface consumed by downstream crates.

## Top-Level Modules

| File | Size | Description |
|---|---|---|
| `lib.rs` | 23K | Public re-exports, feature flag docs, crate-level docs |
| `instance.rs` | 100K | `Py<T>`, `Bound<'py,T>`, `Borrowed<'a,'py,T>` smart pointers |
| `marker.rs` | 36K | `Python<'py>` GIL token; `Ungil` auto-trait |
| `conversion.rs` | 22K | `IntoPyObject`, `FromPyObject`, `IntoPyObjectExt` traits |
| `exceptions.rs` | 38K | All standard Python exception types as Rust structs |
| `buffer.rs` | 38K | Python buffer protocol (`PyBuffer<T>`) |
| `inspect.rs` | 20K | Experimental type-hint introspection (`experimental-inspect` feature) |
| `sync.rs` | 40K | Re-exports from `sync/` (GIL-aware sync primitives) |
| `pycell.rs` | 30K | Re-exports + public types from `pycell/` |
| `pyclass.rs` | 4K | `PyClass` trait public definition |
| `pyclass_init.rs` | 6K | `PyClassInitializer<T>` for `__new__` |
| `pybacked.rs` | 22K | `PyBackedStr`, `PyBackedBytes` — owned views into Python strings |
| `byteswriter.rs` | 9K | `PyBytesWriter` for building `bytes` objects |
| `call.rs` | 10K | `call_method*`, `call*` convenience helpers |
| `coroutine.rs` | 6K | Public `Coroutine` type; re-exports from `coroutine/` |
| `fmt.rs` | 7K | `Display`/`Debug` implementations for Python objects |
| `interpreter_lifecycle.rs` | 6K | `prepare_freethreaded_python`, `with_embedded_interpreter` |
| `marshal.rs` | 2K | Python marshal protocol bindings |
| `version.rs` | 5K | `PythonVersionInfo` — runtime Python version query |
| `type_object.rs` | 5K | `PyTypeInfo`, `PyTypeCheck`, `PyLayout` traits |
| `macros.rs` | 7K | `py_run!`, `intern!`, `create_exception!` and other utility macros |
| `prelude.rs` | 2K | The `pyo3::prelude::*` re-export set |
| `impl_.rs` | <1K | Thin re-export shim into `impl_/` |
| `internal.rs` | <1K | Thin re-export shim into `internal/` |
| `sealed.rs` | 3K | Sealed trait pattern for API stability |
| `panic.rs` | 1K | `PanicException` — turns Rust panics into Python exceptions |
| `ffi_ptr_ext.rs` | 3K | Extension trait on raw `*mut PyObject` pointers |
| `internal_tricks.rs` | 2K | Compile-time tricks used across the crate |
| `py_result_ext.rs` | <1K | `PyResultExt` helper trait |
| `test_utils.rs` | <1K | Minimal test helpers |

## Subdirectories

| Directory | Description |
|---|---|
| `types/` | Safe Rust wrappers for Python built-in types (35 files) |
| `conversions/` | `FromPyObject`/`IntoPyObject` impls for Rust std and 3rd-party types |
| `impl_/` | `#[doc(hidden)]` runtime support emitted by proc-macros |
| `pyclass/` | `PyClass` trait implementation details and GC support |
| `pycell/` | `PyClassObject<T>` memory layout and borrow checker |
| `err/` | `PyErr` internals and conversion impls |
| `coroutine/` | Rust Future ↔ Python coroutine bridge |
| `sync/` | GIL-aware synchronization primitives |
| `internal/` | Private utilities not part of any public module |
| `ffi/` | Thin re-export of `pyo3-ffi` with a small test file |
| `tests/` | Integration-level unit tests |

## Core Invariants

- Every API that touches Python objects requires a `Python<'py>` token or a
  `Bound<'py, T>` that carries the `'py` lifetime, encoding GIL ownership in
  the type system.
- `Py<T>` is the GIL-independent reference-counted pointer; `Bound<'py, T>`
  is its GIL-bound form.
- `PyAny` is the root type; all typed Python objects deref through it.
