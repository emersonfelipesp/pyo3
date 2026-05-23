# pyo3-macros-backend

The actual implementation of all PyO3 proc macros. This crate is a regular library
crate (not a `proc-macro = true` crate), which allows it to be unit-tested and to
share code across macros without restrictions.

## Public surface (re-exported by pyo3-macros)

| Symbol | Purpose |
|---|---|
| `build_py_class` | `#[pyclass]` on structs |
| `build_py_enum` | `#[pyclass]` on enums |
| `build_py_methods` | `#[pymethods]` |
| `build_py_function` | `#[pyfunction]` |
| `pymodule_module_impl` / `pymodule_function_impl` | `#[pymodule]` |
| `build_derive_from_pyobject` | `#[derive(FromPyObject)]` |
| `build_derive_into_pyobject` | `#[derive(IntoPyObject/IntoPyObjectRef)]` |
| `PyClassArgs`, `PyClassMethodsType`, `PyFunctionOptions`, `PyModuleOptions` | Parsed option types |
| `get_doc` | Utility: extracts doc comments from attributes |

## Source files

| File | Size | Role |
|---|---|---|
| `pyclass.rs` | 112 K | Core `#[pyclass]` codegen: slot generation, descriptor dispatch, `PyTypeObject` builder, enum support |
| `pymethod.rs` | 68 K | Method wrappers, slot definitions (`__repr__`, `__hash__`, `__new__`, etc.), getter/setter codegen |
| `method.rs` | 45 K | `FnSpec`, `FnArg`, `RegularArg`, `VarargsArg`, `KwargsArg` — the IR for a single Python-callable function |
| `module.rs` | 27 K | `#[pymodule]` codegen for both the module-item and legacy function forms |
| `frompyobject.rs` | 24 K | `#[derive(FromPyObject)]` codegen |
| `intopyobject.rs` | 23 K | `#[derive(IntoPyObject/IntoPyObjectRef)]` codegen |
| `introspection.rs` | 22 K | `experimental-inspect`: emits introspection metadata for Python type stubs |
| `pyfunction.rs` | 16 K | `#[pyfunction]` entry + `PyFunctionOptions`; delegates signature parsing to `pyfunction/` |
| `pyimpl.rs` | 18 K | `#[pymethods]` dispatch, const attribute support, cfg passthrough |
| `attributes.rs` | — | All `kw::*` custom keywords; `KeywordAttribute`, `CrateAttribute`, `NameAttribute`, etc. |
| `params.rs` | 11 K | `impl_arg_params`: generates the argument-extraction and default-value code |
| `utils.rs` | 13 K | `Ctx` (crate-path context), `PythonDoc`, `get_doc`, `apply_renaming_rule`, `locate_tokens_at` |
| `py_expr.rs` | 11 K | `PyExpr` — Python expression AST for `experimental-inspect` type annotations |
| `derive_attributes.rs` | 8 K | Attribute parsing shared by derive macros |
| `konst.rs` | 3 K | `#[classattr]` constant support |
| `quotes.rs` | 1 K | Reusable `quote!` fragments |
| `combine_errors.rs` | 1 K | `CombineErrors`: accumulates multiple `syn::Error` values |

`pyfunction/` is a subdirectory; see `pyfunction/CLAUDE.md`.

## Key design patterns

- **`FnSpec`** (in `method.rs`) is the central IR: every Python-callable Rust
  function is parsed into a `FnSpec` before code generation. It holds the calling
  convention, argument list, return type, and attribute options.
- **`Ctx`** (in `utils.rs`) carries the user's `crate` path (defaulting to `::pyo3`)
  so generated code uses the correct crate even in re-export scenarios.
- **Slot definitions** in `pymethod.rs` use `SlotDef` structs that know which
  `PyTypeObject` field they populate and what calling convention they require.
- **`PyClassMethodsType`**: `Specialization` (default, single `#[pymethods]` block)
  vs `Inventory` (`multiple-pymethods` feature, uses the `inventory` crate for
  distributed registration).
- The `experimental-inspect` feature gates all `introspection.rs` and `py_expr.rs`
  usage; compiled-out when the feature is absent.

## Features

| Feature | Effect |
|---|---|
| `experimental-async` | Enables `async fn` support in `#[pyfunction]` / `#[pymethods]` |
| `experimental-inspect` | Enables `introspection.rs` and `py_expr.rs` for Python type-stub emission |
