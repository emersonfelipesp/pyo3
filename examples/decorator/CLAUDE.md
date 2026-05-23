# decorator

Example demonstrating how to implement a Python decorator in Rust using `#[pyclass]` and `__call__`.

## What It Demonstrates

- `#[pyclass]` as a callable decorator object (the `Counter` pattern)
- `__call__` implementation via `#[pymethods]`
- Storing a Python callable (`Py<PyAny>`) inside a Rust struct
- Forwarding `*args` and `**kwargs` using `PyTuple` and `PyDict`
- Atomic state inside a `#[pyclass]` (`AtomicU64`)
- `#[pyo3(signature = (*args, **kwargs))]` for variadic function signatures

## Source Files

- `src/lib.rs` — `PyCounter` class: wraps any Python callable, counts calls, forwards all arguments
- `tests/` — pytest tests verifying call counting and argument forwarding

## Key Pattern

```rust
#[pyclass(name = "Counter")]
pub struct PyCounter {
    count: AtomicU64,
    wraps: Py<PyAny>,  // stored Python callable
}

#[pymethods]
impl PyCounter {
    #[pyo3(signature = (*args, **kwargs))]
    fn __call__(&self, py: Python<'_>, args: &Bound<'_, PyTuple>,
                kwargs: Option<&Bound<'_, PyDict>>) -> PyResult<Py<PyAny>> {
        self.count.fetch_add(1, Ordering::Relaxed);
        self.wraps.call(py, args, kwargs)
    }
}
```

Usage in Python: `@Counter` or `Counter(fn)` wraps a function and tracks invocations.

## Build

```bash
maturin develop
pytest tests/
nox
```

## Relationship to Guide

Referenced in the class customization section of the guide, specifically the callable protocol (`class/call.md`).
