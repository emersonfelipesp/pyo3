# setuptools-rust-starter

Alternative getting-started template that uses `setuptools-rust` instead of `maturin` as the build backend. Implements the same module as `maturin-starter` (a `#[pymodule]` with `#[pyclass]` and a submodule).

## What It Demonstrates

- Building a PyO3 extension with `setuptools-rust` and `pyproject.toml`
- Same PyO3 patterns as `maturin-starter`: `#[pymodule]`, `#[pyclass]`, `#[pymethods]`, submodule registration

## Source Files

- `src/lib.rs` — identical structure to `maturin-starter/src/lib.rs`
- `src/submodule.rs` — nested submodule (same as `maturin-starter`)
- `setuptools_rust_starter/` — Python package directory
- `tests/` — pytest tests
- `requirements-dev.txt` — `setuptools-rust`, `wheel` (build deps)

## Build Configuration

`pyproject.toml` uses `setuptools` as the build backend with a `[[tool.setuptools-rust.ext-modules]]` entry pointing to the Rust extension. Unlike maturin, setuptools-rust requires the Rust extension to be explicitly declared.

```bash
pip install -r requirements-dev.txt
pip install -e .       # editable install via setuptools-rust
pytest tests/
nox                    # builds and tests
```

## Difference from maturin-starter

| Aspect | maturin-starter | setuptools-rust-starter |
|---|---|---|
| Build backend | maturin | setuptools + setuptools-rust |
| Config | `[tool.maturin]` | `[[tool.setuptools-rust.ext-modules]]` |
| Extra deps | none | `requirements-dev.txt` |
| Rust source | same | same |

## When to Use

Prefer `maturin-starter` for new projects. Use this example if you need `setuptools-rust` for integration into an existing `setup.py` / `setuptools` workflow.
