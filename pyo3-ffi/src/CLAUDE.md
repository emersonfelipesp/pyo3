# pyo3-ffi/src

FFI source root. All submodules are declared and re-exported from `lib.rs`, which
is the single aggregation point for the entire public surface of `pyo3-ffi`.

## lib.rs

The crate root (`#![no_std]`). Declares every submodule with `pub mod` and re-exports
their contents via `pub use`. Includes the crate-level doc comment explaining feature
flags, cfg keys, and minimum-version support. Contains no FFI declarations itself.

## Top-level source files (direct bindings to CPython's public `Include/` headers)

The largest / most central files:

| File | Size | Maps to |
|---|---|---|
| `object.rs` | 26 K | `object.h` — `PyObject`, `PyTypeObject`, `Py_INCREF`, `Py_DECREF`, type slots |
| `pyerrors.rs` | 18 K | `pyerrors.h` — exception hierarchy, `PyErr_*` functions |
| `unicodeobject.rs` | 14 K | `unicodeobject.h` — public Unicode API |
| `datetime.rs` | 25 K | `datetime.h` — `PyDateTime_CAPI`, date/time struct layouts |
| `abstract_.rs` | 17 K | `abstract.h` — generic sequence/mapping/number protocol functions |
| `methodobject.rs` | 9 K | `methodobject.h` — `PyMethodDef`, `PyMethodDefPointer`, calling conventions |
| `refcount.rs` | 11 K | Reference counting helpers, including GIL-disabled atomic variants |
| `slots.rs` / `slots_generated.rs` | — | Slot ID constants and generated slot table |

Other files follow the same one-file-per-header convention: `dictobject.rs`,
`listobject.rs`, `tupleobject.rs`, `longobject.rs`, `bytesobject.rs`,
`floatobject.rs`, `setobject.rs`, `moduleobject.rs`, `import.rs`, `pystate.rs`,
`ceval.rs`, `pybuffer.rs`, `pycapsule.rs`, `structseq.rs`, etc.

## Subdirectories

- `cpython/` — bindings for CPython-private headers (`Include/cpython/`); excluded
  under `Py_LIMITED_API`. See `cpython/CLAUDE.md`.
- `compat/` — version polyfill shims for newer C API functions, backported inline.
  See `compat/CLAUDE.md`.
- `impl_/` — internal helper macros (not part of the public API surface).
  See `impl_/CLAUDE.md`.

## Conventions

- All struct fields and function signatures exactly match their C counterparts.
- Version-gated items use `#[cfg(Py_3_N)]` on individual items, not whole files.
- `Py_LIMITED_API` items live only in the top-level files, never in `cpython/`.
- The crate is `#![no_std]`; use `core::ffi::*` types, not `std::ffi::*`.
