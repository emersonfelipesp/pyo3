# src/impl_/pyclass/ — PyClass Type Object Runtime Detail

Sub-module of `src/impl_/`. Contains the machinery for lazily creating and
caching `PyTypeObject` instances at runtime, plus compile-time probes and
assertion helpers used by the proc-macro system.

All items are `#[doc(hidden)]`.

## Files

| File | Size | Description |
|---|---|---|
| `lazy_type_object.rs` | 9K | `LazyTypeObject<T>` — wraps a `GILOnceCell<PyClassTypeObject>` so that the CPython type object for a `#[pyclass]` struct is created exactly once, on first use; also provides `type_object_init_failed` panic helper |
| `create_type_object.rs` | 28K | `create_type_object()` — the function that actually allocates and populates a `PyTypeObject` from `PyClassImpl` metadata; handles slot assignment, inheritance, dict/weakref support |
| `probes.rs` | 3K | Zero-sized `Probe<T>` types that evaluate Boolean properties of a type at compile time via impl specialization (e.g., `ImplementsIntoPyObject`, `IsBaseNativeType`) |
| `assertions.rs` | 1K | Compile-time assertion helpers that emit readable error messages when invalid `#[pyclass]` combinations are used |
| `doc.rs` | 4K | `PyClassDocGenerator<T, HAS_NEW_TEXT_SIGNATURE>` — assembles the `__doc__` string for a `#[pyclass]` at compile time using associated constants |

## Relationships

- `lazy_type_object.rs` is used by `src/type_object.rs` (`PyTypeInfo::type_object_raw`) to vend the singleton `*mut PyTypeObject`.
- `create_type_object.rs` calls into `src/pyclass/create_type_object.rs` (the public-facing wrapper in the `pyclass/` module).
- `probes.rs` probe results are read by code in `src/impl_/pyclass.rs` to conditionally wire slots.
- `doc.rs` is called from proc-macro-generated `PyClassImpl::doc` constants.
