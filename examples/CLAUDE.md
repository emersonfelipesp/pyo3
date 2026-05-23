# examples/

Six standalone example projects, each demonstrating a specific PyO3 pattern. Every example is a self-contained Cargo project with its own `Cargo.toml`, `pyproject.toml`, and test suite.

## Common Structure

Each example follows the same layout:

```
<name>/
  src/lib.rs (or main.rs)   # Rust extension source
  tests/                    # Python test files (pytest)
  <snake_name>/             # Python package directory (some examples)
  pyproject.toml            # maturin (or setuptools-rust) build config
  Cargo.toml
  noxfile.py                # nox session to build + test
  cargo-generate.toml       # metadata for cargo-generate templating
  MANIFEST.in
```

All examples use `cargo-generate.toml` so they can serve as templates via `cargo generate`.

## Examples

| Directory | Build tool | Pattern demonstrated |
|---|---|---|
| `maturin-starter/` | maturin | Canonical getting-started template: `#[pymodule]`, `#[pyclass]`, submodules |
| `word-count/` | maturin | String processing; parallel (`rayon`) and sequential variants; `py.detach()` |
| `decorator/` | maturin | Python decorator in Rust: `#[pyclass]` with `__call__` |
| `getitem/` | maturin | `__getitem__` / `__setitem__` with integer and slice support |
| `plugin/` | maturin | Embedding Python in Rust; `append_to_inittab!`; two sub-crates |
| `setuptools-rust-starter/` | setuptools-rust | Same module as maturin-starter but built with setuptools-rust |

## Workspace Membership

All examples are listed under `[workspace.members]` in `examples/Cargo.toml` (a separate mini-workspace). They are not members of the root PyO3 workspace.

## Running an Example

```bash
cd examples/maturin-starter
nox          # builds with maturin and runs pytest
# or manually:
maturin develop
pytest tests/
```

## Relationship to the Guide

Each example corresponds to one or more guide sections. The `maturin-starter` is referenced in `guide/src/getting-started.md`. The `decorator` and `getitem` examples are referenced in class customization sections.
