# word-count

Example demonstrating Rust string processing exposed to Python, including parallel execution via `rayon` and GIL management with `py.detach()`.

## What It Demonstrates

- `#[pyfunction]` — exposing standalone Rust functions to Python
- `rayon` parallelism from PyO3: releasing the GIL during CPU-bound work
- `py.detach()` — the idiomatic way to release the GIL and run Rust code concurrently
- Comparing parallel vs. sequential implementations

## Source Files

- `src/lib.rs` — three exported functions:
  - `search(contents, needle)` — parallel word count using `rayon::par_lines()`
  - `search_sequential(contents, needle)` — single-threaded word count
  - `search_sequential_detached(py, contents, needle)` — sequential but releases the GIL via `py.detach()`
- `word_count/` — Python package directory
- `tests/` — pytest tests comparing all three implementations

## Dependencies

- `pyo3` (from workspace path, `extension-module` feature)
- `rayon` — data parallelism library

## Build

```bash
maturin develop
pytest tests/
nox
```

## Key Pattern

```rust
fn search_sequential_detached(py: Python<'_>, contents: &str, needle: &str) -> usize {
    py.detach(|| search_sequential(contents, needle))
}
```

`py.detach()` releases the GIL for the duration of the closure, allowing other Python threads to run while the Rust code executes. This is the correct pattern for CPU-bound Rust called from Python.

## Relationship to Guide

Referenced in `guide/src/parallelism.md` as the canonical parallelism example.
