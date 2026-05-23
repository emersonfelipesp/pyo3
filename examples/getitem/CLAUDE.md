# getitem

Example demonstrating the `__getitem__` and `__setitem__` protocols in PyO3, with both integer index and slice support.

## What It Demonstrates

- `__getitem__` handling both `int` and `slice` arguments
- `__setitem__` using `#[derive(FromPyObject)]` on a Rust enum to dispatch on argument type
- `PySlice::indices()` — computing concrete start/stop/step from a Python slice with bounds checking
- `PySlice::getattr()` — manually extracting individual slice fields
- Two approaches to slice handling: `indices()` (recommended) vs. manual `getattr()`

## Source Files

- `src/lib.rs` — `ExampleContainer` class with `max_length: i32`:
  - `__getitem__` tries `extract::<i32>()` first, then `cast::<PySlice>()`, then raises `TypeError`
  - `__setitem__` uses `IntOrSlice` enum derived from `FromPyObject` for clean dispatch
- `tests/` — pytest tests for integer access, slice access, and type error cases

## Key Pattern

```rust
#[derive(FromPyObject)]
enum IntOrSlice<'py> {
    Int(i32),
    Slice(Bound<'py, PySlice>),
}

fn __setitem__(&self, idx: IntOrSlice, value: u32) -> PyResult<()> {
    match idx { ... }
}
```

`FromPyObject` on an enum tries each variant in order, selecting the first that succeeds. This is the idiomatic PyO3 approach to multi-type dispatch.

## Build

```bash
maturin develop
pytest tests/
nox
```

## Relationship to Guide

Referenced in the class protocols section (`class/protocols.md`) covering sequence and mapping emulation.
