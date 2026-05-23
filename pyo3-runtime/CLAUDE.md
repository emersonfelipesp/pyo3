# pyo3-runtime

Small Python helper package used internally by PyO3's test infrastructure. Not published to PyPI.

## Purpose

Provides a Python-side package (`pyo3_runtime`) that PyO3's Rust integration tests and pytests can import. Currently a minimal scaffold — the `src/pyo3_runtime/__init__.py` is empty (version is derived dynamically by hatch).

## Structure

```
pyo3-runtime/
  src/
    pyo3_runtime/
      __init__.py    # empty; version managed by hatch
  tests/             # Python test files
  pyproject.toml     # hatchling build, MIT OR Apache-2.0
  README.md          # placeholder
```

## Build System

- Build backend: `hatchling`
- Version source: `src/pyo3_runtime/__init__.py` (dynamic via `tool.hatch.version`)
- Supports CPython 3.8–3.12 and PyPy/GraalPy
- No dependencies

## Relationship to the Rest of PyO3

- Installed as part of the `nox` test environment alongside the compiled `pyo3` extension.
- Referenced by `noxfile.py` at the root; nox installs it in editable mode before running tests.
- Not a dependency of any Rust crate — purely a Python test helper package.
