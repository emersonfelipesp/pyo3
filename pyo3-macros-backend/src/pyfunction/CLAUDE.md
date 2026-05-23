# pyo3-macros-backend/src/pyfunction

Signature parsing and attribute handling for `#[pyfunction]`. The parent module
`pyfunction.rs` (in `src/`) handles the top-level macro entry and options; this
subdirectory provides the two most complex sub-tasks.

## Files

### signature.rs (22 K)

Parses and models the `#[pyo3(signature = (...))]` attribute.

Key types:

| Type | Purpose |
|---|---|
| `Signature` | Parsed `(arg, arg=default, *args, **kwargs)` signature expression |
| `SignatureItem` | One item inside the signature: argument, `/`, `*`, `*args`, or `**kwargs` |
| `SignatureItemArgument` | Named argument with optional type annotation and default value |
| `FunctionSignature` | The resolved signature after aligning the declared `Signature` against the actual Rust `FnArg` list |
| `SignatureAttribute` | The `#[pyo3(signature = ...)]` attribute parsed from a `#[pyfunction]` or `#[pymethods]` method |
| `ConstructorAttribute` | `#[pyo3(constructor = ...)]` variant for `#[new]` methods |

`FunctionSignature::from_arguments_and_attribute` is the main entry point: it takes
the list of `FnArg`s from `method.rs` and the optional `SignatureAttribute`, validates
that every declared parameter exists, assigns default values, and determines where the
positional-only `/` and keyword-only `*` separators fall.

`PyTypeAnnotation` handles optional type annotations in the signature (`arg: SomeType`),
used by `experimental-inspect` for generating Python type stubs.

### attributes.rs (parent file, not in this subdirectory)

`src/attributes.rs` defines all `kw::*` custom keywords shared across the entire
backend (including `signature`, `name`, `from_py_with`, `pass_module`,
`text_signature`, `warn`, etc.) and the shared `KeywordAttribute<K, V>` generic
parser. Attribute types specific to `#[pyfunction]` arguments (`FromPyWithAttribute`,
`CancelHandle`) are parsed in the parent `pyfunction.rs` via
`PyFunctionArgPyO3Attributes`.

## Data flow

```
#[pyfunction] attr tokens
    -> PyFunctionOptions (parsed in pyfunction.rs)
        -> SignatureAttribute (optional, from pyo3(signature = ...))
            -> FunctionSignature::from_arguments_and_attribute
                -> FnSpec (in method.rs)
                    -> impl_arg_params (in params.rs)  -> generated argument extraction code
```

## Relationship to other modules

- `method.rs` provides `FnArg` (the Rust-level argument description) which
  `FunctionSignature` consumes.
- `params.rs` consumes `FunctionSignature` to emit the runtime argument-parsing
  code that handles positional, keyword, `*args`, and `**kwargs` arguments.
- `pyclass.rs` and `pyimpl.rs` call into `pyfunction.rs` for method codegen,
  reusing `FunctionSignature` for `#[new]`, getters, setters, and regular methods.
