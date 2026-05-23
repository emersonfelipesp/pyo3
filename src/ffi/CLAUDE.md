# src/ffi/ — Raw CPython C API Re-exports

A thin pass-through module that re-exports everything from the `pyo3-ffi`
crate. All of CPython's raw C API declarations (structs, constants, function
pointers, unsafe functions) are accessible via `pyo3::ffi::*`.

This module exists so users who depend only on `pyo3` do not also need to
add `pyo3-ffi` as a direct dependency when they need raw FFI access.

## Files

| File | Description |
|---|---|
| `mod.rs` | Single `pub use pyo3_ffi::*;` re-export; module-level safety docs |
| `tests.rs` | Sanity tests for a small set of raw FFI values (sizes, alignment, constant values) |

## Usage

The `pyo3::ffi` module is intended for advanced use only. Normal PyO3 code
should use the safe wrappers in `src/types/` and `src/instance.rs`. Use
`pyo3::ffi::*` only when:

- Implementing PyO3 internals that must drop to the raw API.
- Writing glue code for CPython extensions not yet covered by PyO3's safe
  layer.
- Calling C extension functions that take `*mut PyObject` directly.

## Relationships

- `pyo3-ffi/` (workspace crate) is the actual source of all declarations;
  it contains one `.rs` file per CPython header.
- `pyo3-build-config` sets the `Py_3_8`…`Py_3_14`, `PyPy`, `GraalPy`,
  `Py_LIMITED_API`, `Py_GIL_DISABLED` cfg flags that gate version-specific
  FFI declarations inside `pyo3-ffi`.
- Most files in `src/` import `use crate::ffi;` and use symbols from here
  to call the underlying C API.
