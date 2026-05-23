# pyo3-macros

Thin proc-macro entry crate. Rust requires that a crate declaring `#[proc_macro]`
attributes contain no other public items, so all real logic lives in
`pyo3-macros-backend`. This crate is the glue layer only.

## src/lib.rs

Declares every exported macro. Each entry point parses its inputs using `syn` and
delegates to a builder function from `pyo3-macros-backend`.

### Exported macros

| Macro | Kind | Backend function |
|---|---|---|
| `#[pymodule]` | attribute | `pymodule_module_impl` / `pymodule_function_impl` |
| `#[pyclass]` | attribute | `build_py_class` (struct) / `build_py_enum` (enum) |
| `#[pymethods]` | attribute | `build_py_methods` |
| `#[pyfunction]` | attribute | `build_py_function` |
| `#[derive(FromPyObject)]` | derive | `build_derive_from_pyobject` |
| `#[derive(IntoPyObject)]` | derive | `build_derive_into_pyobject::<false>` |
| `#[derive(IntoPyObjectRef)]` | derive | `build_derive_into_pyobject::<true>` |

`#[pymodule]` dispatches on the item kind: `Item::Mod` goes to the module impl,
`Item::Fn` goes to the deprecated function impl.

`#[pyclass]` dispatches on `Item::Struct` / `Item::Enum`.

`#[pymethods]` prepends the attribute tokens as a `#[pyo3(...)]` attr on the
`impl` block before forwarding, so the backend sees a uniform input.

### Error handling

The local `UnwrapOrCompileError` trait converts `syn::Result<TokenStream2>` into a
compiler error token stream. All macro entry points use `.unwrap_or_compile_error()`.

## Features

| Feature | Effect |
|---|---|
| `multiple-pymethods` | Passes `PyClassMethodsType::Inventory` to the backend, enabling multiple `#[pymethods]` blocks per class via the `inventory` crate |
| `experimental-async` | Forwarded to `pyo3-macros-backend` |
| `experimental-inspect` | Forwarded to `pyo3-macros-backend` |

## Relationship to other crates

- Depends on `pyo3-macros-backend` (implementation) and `syn`/`proc-macro2`/`quote`
  (parsing/quoting).
- The `pyo3` crate re-exports these macros to end users; end users never depend on
  `pyo3-macros` directly.
