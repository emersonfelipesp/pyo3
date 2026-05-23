# pytests/

Python-side integration tests for PyO3. The Rust extension is compiled as `pyo3_pytests` (a `cdylib`), then pytest drives the tests from the Python side. This gives coverage that Rust-only tests cannot — actual Python import, calling convention, type checking, and interoperability.

## Running

```bash
nox -s pytests              # build extension + run pytest
nox -s pytests-py3.12       # specific Python version
# manually:
maturin develop --manifest-path pytests/Cargo.toml
pytest pytests/tests/
```

## Structure

```
pytests/
  src/              # Rust extension source (17 modules)
  tests/            # pytest test files (17 files)
  stubs/            # hand-written .pyi stubs (17 files + __init__.pyi)
  Cargo.toml        # pyo3-pytests cdylib
  build.rs          # calls pyo3_build_config::use_pyo3_cfgs()
  pyproject.toml    # maturin build config
  conftest.py       # pytest fixtures
  noxfile.py        # nox test session
  MODULE_DOC.md     # included as the module docstring
```

## Rust Source Modules (`src/`)

Each `.rs` file compiles to a Python submodule of `pyo3_pytests`:

| Module | Tests |
|---|---|
| `awaitable.rs` | Async/coroutine objects |
| `buf_and_str.rs` | Buffer protocol and string conversions |
| `comparisons.rs` | Rich comparison operators |
| `consts.rs` | Module-level constants |
| `datetime.rs` | datetime types (disabled under `Py_LIMITED_API` without Py 3.11+) |
| `dict_iter.rs` | Dict iteration patterns |
| `enums.rs` | `#[pyclass]` enums |
| `exception.rs` | Custom exception types |
| `misc.rs` | Miscellaneous patterns |
| `objstore.rs` | `Py<T>` storage and GIL-independent handles |
| `othermod.rs` | Cross-module type sharing |
| `path.rs` | `std::path::Path` / `PathBuf` conversions |
| `pyclasses.rs` | `#[pyclass]` — comprehensive class patterns |
| `pyfunctions.rs` | `#[pyfunction]` — signature variations |
| `sequence.rs` | Sequence protocol |
| `subclassing.rs` | Subclassing Python types from Rust |

## Stubs (`stubs/`)

Hand-written `.pyi` files for each module, used to verify PyO3's type stub conventions and as test fixtures for the `experimental-inspect` feature. Named to match the Rust modules.

## Key Differences from `tests/`

- `tests/` = Rust integration tests that embed Python internally
- `pytests/` = Python tests that import the Rust extension from the outside

The pytest tests are more realistic: they import `pyo3_pytests` the way end-users would, exercising the full `cdylib` → Python import pipeline.

## Features

- `experimental-async` — enables awaitable tests
- `experimental-inspect` — enables type inspection / introspection tests
