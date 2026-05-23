# maturin-starter

Canonical getting-started example for PyO3. The simplest complete maturin-based Python extension module. Used as a template reference for new PyO3 projects.

## What It Demonstrates

- `#[pymodule]` — defining a Python module in Rust
- `#[pyclass]` with `#[pyo3(get, set)]` fields — a basic Python-accessible Rust struct
- `#[pymethods]` with `#[new]` — constructor
- Submodule registration via `wrap_pymodule!` and inserting into `sys.modules`

## Source Files

- `src/lib.rs` — main module `maturin_starter` with `ExampleClass` (holds an `i32 value`) and registration of `submodule`
- `src/submodule.rs` — a nested `submodule` module with its own `SubmoduleClass`
- `maturin_starter/` — Python package directory (may contain `__init__.py`)
- `tests/` — pytest test files

## Build

Uses `maturin` (configured in `pyproject.toml`). The `[tool.maturin]` section sets `features = ["pyo3/extension-module"]`.

```bash
maturin develop        # install in current venv
pytest tests/          # run Python tests
nox                    # build + test via nox
```

## Relationship to Other Examples

`setuptools-rust-starter/` provides the same module structure built with `setuptools-rust` instead of `maturin`. Both share `submodule.rs` logic.

## Template Use

`cargo-generate.toml` allows this to be scaffolded via:
```bash
cargo generate --git https://github.com/PyO3/pyo3 examples/maturin-starter
```
