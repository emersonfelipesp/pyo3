# src/impl_/ — Proc-Macro Runtime Support

All items here are `#[doc(hidden)]` — they form the runtime half of the
proc-macro system. The `pyo3-macros-backend` crate generates code that calls
into these modules; they should not be used directly by downstream crates.

## Files

| File | Size | Description |
|---|---|---|
| `extract_argument.rs` | 37K | Argument extraction machinery for `#[pyfunction]` and `#[pymethods]`: keyword/positional parsing, `**kwargs`, `*args`, default values, type coercion |
| `pyclass.rs` | 50K | Core runtime for `#[pyclass]`: slot wiring, `tp_dealloc`, dict/weakref offsets, items iteration, `PyClassImpl` trait |
| `pymethods.rs` | 23K | Method descriptor machinery: `PyMethodDefType`, `PyGetterDef`, `PySetterDef`, `PyDeleterDef`, slot function signatures |
| `pymodule.rs` | 20K | Module initialization: `ModuleDef`, `PyModuleExt`, submodule support, multi-phase init (`Py_mod_exec`) |
| `trampoline.rs` | 12K | `trampoline()` — monomorphised panic-catching entry point for all Python-callable Rust functions; avoids code bloat from inlining `catch_unwind` everywhere |
| `callback.rs` | 6K | `PyCallbackOutput` trait — maps Rust return values to C-level output conventions (`*mut PyObject`, `c_int`, etc.) |
| `frompyobject.rs` | 4K | `derive(FromPyObject)` generated helper traits |
| `pyfunction.rs` | 5K | `PyFunctionImpl` — runtime support for `#[pyfunction]` |
| `pyclass_init.rs` | 6K | `PyObjectInit<T>` — initializing `PyClassObject` memory from `#[new]` |
| `concat.rs` | 3K | Const-time string concatenation for building C strings in macros |
| `unindent.rs` | 8K | Strips leading indentation from docstrings at runtime |
| `freelist.rs` | 2K | Optional free-list allocator for `#[pyclass(freelist = N)]` |
| `introspection.rs` | 4K | Linker-section metadata emission for `pyo3-introspection` |
| `coroutine.rs` | 1K | Internal coroutine helpers |
| `deprecated.rs` | <1K | Compatibility shims for removed APIs |
| `exceptions.rs` | <1K | `PyErrArguments` helper |
| `panic.rs` | <1K | `PanicTrap` — ensures panics surface as Python exceptions |
| `wrap.rs` | 4K | `wrap_pyfunction!` and `wrap_pymodule!` macro support |
| `pycell.rs` | <1K | Re-export shim into `pycell/` |

## Subdirectory

| Directory | Description |
|---|---|
| `pyclass/` | Deeper PyClass runtime detail: lazy type object, type object construction, probes, assertions, doc generation |

## Key Relationships

- `pyo3-macros-backend` generates calls to `::pyo3::impl_::extract_argument::*`,
  `::pyo3::impl_::pyclass::*`, `::pyo3::impl_::trampoline::*` etc.
- `trampoline.rs` is the single point where `std::panic::catch_unwind` is
  compiled — all Python-callable Rust functions funnel through it to keep
  binary size manageable.
- `pyclass.rs` is where the `PyClassImpl` trait is defined; the proc-macro
  generates an impl of that trait for each `#[pyclass]` struct.
