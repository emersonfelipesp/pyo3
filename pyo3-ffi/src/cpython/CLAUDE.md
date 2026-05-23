# pyo3-ffi/src/cpython

Bindings for CPython's implementation-private headers (`Include/cpython/`). These APIs
are not part of the Limited API and are therefore unavailable when compiling with
`Py_LIMITED_API` (i.e., when the `abi3` feature is enabled).

## mod.rs

Declares all 37 sub-modules with `pub(crate)` visibility and re-exports them as `pub`.
Includes per-module `#[cfg]` guards where a module requires a specific runtime or
version (e.g. `#[cfg(not(PyPy))]` for `initconfig`, `#[cfg(Py_3_13)]` for `lock`).

## Key files by size

| File | Size | Binds |
|---|---|---|
| `unicodeobject.rs` | 27 K | `cpython/unicodeobject.h` — internal Unicode struct layout, codec internals |
| `object.rs` | 14 K | `cpython/object.h` — `_Py_Dealloc`, internal type layout extensions |
| `pythonrun.rs` | 7 K | `cpython/pythonrun.h` — interpreter startup/shutdown details |
| `initconfig.rs` | 7 K | `cpython/initconfig.h` — `PyConfig`, `PyPreConfig` (CPython 3.8+) |
| `code.rs` | 4 K | `cpython/code.h` — internal code object fields |
| `funcobject.rs` | 3 K | `cpython/funcobject.h` — function object internals |
| `genobject.rs` | 3 K | `cpython/genobject.h` — generator/coroutine object layout |
| `pystate.rs` | 3 K | `cpython/pystate.h` — `PyThreadState` internals |
| `pymem.rs` | 2 K | `cpython/pymem.h` — allocator API |
| `abstract_.rs` | 9 K | `cpython/abstract.h` — fast-call helpers (`_PyObject_Vectorcall`) |

## Runtime and version guards

- Most modules are unconditional; some use `#[cfg(not(PyPy))]` because PyPy does
  not expose the corresponding CPython internal.
- `lock.rs` — `#[cfg(Py_3_13)]` (mutex API introduced in 3.13).
- `critical_section.rs` — `#[cfg(all(Py_3_14, Py_GIL_DISABLED))]`.
- `ceval.rs` / `initconfig.rs` / `pylifecycle.rs` / `methodobject.rs` —
  `#[cfg(not(PyPy))]`.

## Relationship to the public header tree

Items from `cpython/` are re-exported at the crate root via the parent `src/lib.rs`
alongside the public-header items, so callers see a flat namespace. The `cpython/`
split exists only to mirror the CPython header directory structure and to clearly mark
which symbols are NOT covered by PEP 384.

Do not use any symbol from this directory when targeting `abi3` builds.
