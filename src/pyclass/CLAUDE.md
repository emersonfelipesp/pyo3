# src/pyclass/ — PyClass Trait and GC/Borrow Infrastructure

Public-facing `PyClass` trait definition and its direct runtime support.
Distinct from `src/impl_/pyclass/` which holds the hidden proc-macro support;
this directory contains the types that downstream `#[pyclass]` users interact
with.

## Files

| File | Size | Description |
|---|---|---|
| `create_type_object.rs` | 28K | `create_type_object()` — allocates and populates a CPython `PyTypeObject` from the metadata collected by `PyClassImpl`; handles slots, base classes, dict/weakref support, GC flag |
| `guard.rs` | 38K | `PyClassGuard<T>`, `PyClassGuardMut<T>` — runtime borrow guards for `#[pyclass]` instances; implement interior mutability with dynamic borrow checking analogous to `RefCell`; `PyClassGuardError`, `PyClassGuardMutError` |
| `gc.rs` | 1K | GC protocol types: `PyTraverseError`, `PyVisit<'a>` — used in `__traverse__` implementations declared via `#[pymethods]` |

## Key Concepts

`guard.rs` is the boundary between Rust's ownership model and Python's
shared-reference model. When `#[pyclass]` structs are accessed from Python,
PyO3 cannot statically guarantee `&mut` uniqueness, so `PyClassGuard` /
`PyClassGuardMut` track borrows at runtime. In free-threaded CPython builds
this gets more complex — the guards use atomic borrow counts instead of a
simple `RefCell`-style flag.

`gc.rs` provides the visitor type passed to `__traverse__`. User-defined
`#[pymethods]` with `fn __traverse__(&self, visit: PyVisit<'_>)` call
`visit.call(&self.field)` for each Python-object field.

## Relationships

- `create_type_object.rs` is called from `src/impl_/pyclass/lazy_type_object.rs`
  the first time a `#[pyclass]` type is used.
- `guard.rs` is used by `src/pycell/impl_.rs` which stores the borrow checker
  inside the `PyClassObject<T>` allocation.
- `gc.rs` types appear in user-written `__traverse__` methods and in the GC
  slot wired by `src/impl_/pyclass.rs`.
