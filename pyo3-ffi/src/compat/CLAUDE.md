# pyo3-ffi/src/compat

Version polyfill shims for C API functions that were added in specific Python versions.
Each shim provides an inline Rust reimplementation that works on older versions,
while simply re-exporting the real C function on versions where it natively exists.

## mod.rs

Defines the `compat_function!` macro and includes/re-exports all version-specific
submodules. The macro generates:

1. An `unsafe fn` body for the polyfill (compiled when the real function is absent).
2. A `pub use $crate::$name` re-export (compiled when the native version is present).
3. A compile-test module verifying the two definitions do not accidentally coexist.

Submodules are sourced from the `pythoncapi-compat` project
(<https://github.com/python/pythoncapi-compat>).

## Submodule layout

| File | Backports functions introduced in |
|---|---|
| `py_3_9.rs` | Python 3.9 (`Py_NewRef`, `Py_XNewRef`) |
| `py_3_10.rs` | Python 3.10 |
| `py_3_13.rs` | Python 3.13 (`PyDict_GetItemRef`, `PyList_GetItemRef`, and others) |
| `py_3_14.rs` | Python 3.14 |
| `py_3_15.rs` | Python 3.15 |

## How to add a new shim

Use `compat_function!` with `originally_defined_for(Py_3_N)` matching the CPython
version that first introduced the function. Place it in the appropriate `py_3_N.rs`
file. The macro handles the cfg-based dispatch automatically.

## Relationship to the rest of pyo3-ffi

`compat::*` is re-exported from `src/lib.rs` into the crate root, so callers always
import from the flat `pyo3_ffi::` namespace without knowing whether they are getting
a polyfill or a native re-export.
