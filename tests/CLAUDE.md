# tests/

Rust integration test suite for PyO3. Tests run against the main `pyo3` crate using `cargo test`. Each file exercises a specific feature or set of related features.

## Running

```bash
cargo test                          # all tests, current Python
cargo test test_pyclass_basics      # specific test file
nox -s test                         # via nox (sets up Python env)
nox -s test-py3.12                  # specific Python version
```

## Test Files (48 total)

Each `test_<feature>.rs` is a standalone integration test that imports from `pyo3`. Representative files:

| File | Coverage |
|---|---|
| `test_class_basics.rs` | `#[pyclass]` fundamentals: new, repr, str, hash, bool |
| `test_class_init.rs` | `#[new]`, `__init__`, inheritance constructors |
| `test_class_comparisons.rs` | Rich comparison (`__eq__`, `__lt__`, etc.) |
| `test_class_attributes.rs` | `#[classattr]`, class variables |
| `test_class_conversion.rs` | `FromPyObject` / `IntoPyObject` on `#[pyclass]` types |
| `test_methods.rs` | Instance methods, static methods, class methods |
| `test_proto_methods.rs` | Protocol methods (`__len__`, `__iter__`, `__contains__`, etc.) |
| `test_pyfunction.rs` | `#[pyfunction]`, argument handling, return types |
| `test_frompyobject.rs` | `#[derive(FromPyObject)]` |
| `test_intopyobject.rs` | `#[derive(IntoPyObject)]` |
| `test_enum.rs` | `#[pyclass]` on Rust enums |
| `test_exceptions.rs` | Custom exception types, `PyErr` |
| `test_module.rs` | `#[pymodule]`, declarative module syntax |
| `test_declarative_module.rs` | `mod`-based declarative modules |
| `test_inheritance.rs` | `#[pyclass(extends=...)]` |
| `test_gc.rs` | `#[pyclass(unsendable)]`, cyclic GC |
| `test_coroutine.rs` | Async/coroutine support |
| `test_buffer_protocol.rs` | `PyBuffer` implementation |
| `test_datetime.rs` | `PyDateTime`, `PyDate`, `PyTime`, `PyDelta` |
| `test_string.rs` | `PyString` encoding and extraction |
| `test_serde.rs` | `serde` feature integration |
| `test_anyhow.rs` | `anyhow` feature integration |
| `test_compile_error.rs` | Drives `tests/ui/` trybuild compile-error tests |
| `test_macro_docs.rs` | Documentation attribute passthrough |
| `test_multiple_pymethods.rs` | `multiple-pymethods` feature |

## ui/ — Compile-Error Tests (115 files)

`tests/ui/` contains trybuild test cases: each pair `<name>.rs` + `<name>.stderr` verifies that invalid PyO3 usage produces the exact expected compiler error.

- Driven by `test_compile_error.rs` via `trybuild::TestCases`.
- `*.stderr` files are the expected `rustc` error output; update them with `TRYBUILD=overwrite cargo test`.
- Tests are gated on Python version and feature flags (`#[cfg(Py_3_9)]`, `#[cfg(Py_LIMITED_API)]`, etc.) so the correct stderr is matched per configuration.
- Example files: `invalid_pyclass_args.rs`, `invalid_pyfunction_argument.rs`, `abi3_dict.rs`, `reject_generics.rs`, `not_send.rs`

## Conventions

- Each test file starts a Python interpreter using `pyo3::prepare_freethreaded_python()` or `Python::attach(|py| { ... })`.
- Feature-gated tests use `#[cfg(feature = "...")]` or `#[cfg(Py_3_N)]` at the file or test level.
- `test_utils/` — shared test helpers used across multiple test files.
