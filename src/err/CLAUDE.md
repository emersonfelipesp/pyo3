# src/err/ — PyErr and Exception Infrastructure

Defines `PyErr`, PyO3's representation of a Python exception, and all of the
machinery for creating, normalizing, storing, and converting exceptions.

## Files

| File | Size | Description |
|---|---|---|
| `mod.rs` | 37K | `PyErr` struct definition; public API: `new`, `value`, `get_type`, `is_instance`, `restore`, `fetch`, `take`, `cause`, `traceback`, `warn`, `from_value`; `PyResult` type alias |
| `err_state.rs` | 16K | `PyErrState` — internal enum storing exceptions in lazy (not-yet-materialized) or normalized form; `PyErrStateLazyFnOutput`; `PyErrStateNormalized` (holds a `Bound<'py, PyBaseException>`) |
| `impls.rs` | 13K | `From<E> for PyErr` impls for many Rust error types (`io::Error`, `std::fmt::Error`, etc.); `Into<PyErr>` for `PyErr`; `Error` and `Display` for `PyErr` |
| `cast_error.rs` | 8K | `CastError<F,T>`, `CastIntoError<T>` — errors returned by `Bound::cast` / `Bound::cast_into` when the Python type does not match |
| `downcast_error.rs` | 5K | `DowncastError`, `DowncastIntoError` — deprecated predecessor to `CastError`; retained for backward compatibility |

## Key Design Points

`PyErr` stores its state lazily: a Rust closure that constructs the Python
exception object is held until the first time the exception is actually needed
as a Python object (e.g., when being raised into Python or when `value()` is
called). This avoids requiring a `Python<'py>` token at construction time,
allowing `?`-based propagation from contexts without a GIL token.

`err_state.rs` is the only file that directly stores a reference to the
underlying `PyBaseException` object; everything else goes through `PyErr`.

## Relationships

- `src/exceptions.rs` defines the concrete Python exception structs (e.g.,
  `PyValueError`) using `create_exception!` macros; they all convert into
  `PyErr` via `PyErr::new::<PyValueError, _>(msg)`.
- `src/panic.rs` uses `PyErr` to turn Rust panics into `PanicException`.
- The `anyhow` and `eyre` features in `src/conversions/` extend `impls.rs`
  by adding `From<anyhow::Error> for PyErr` etc.
