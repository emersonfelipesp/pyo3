# src/pycell/ — PyClassObject Memory Layout

Defines `PyClassObject<T>`, the in-memory representation of every
`#[pyclass]`-annotated Rust type when allocated by Python. This is the
boundary between Rust's ownership model and Python's refcounting GC.

Public types are re-exported from `src/pycell.rs` (top-level, 30K) which also
contains the `PyRef<T>` and `PyRefMut<T>` public aliases.

## Files

| File | Size | Description |
|---|---|---|
| `impl_.rs` | 32K | Core implementation: `PyClassObject<T>` struct layout, `PyClassMutability` trait, `PyClassBorrowChecker`, `BorrowChecker`/`EmptySlot`/`AtomicBorrowChecker` (for free-threaded builds), `PyClassObjectLayout` trait with `tp_dealloc` logic |

## What `PyClassObject<T>` Looks Like

```
PyClassObject<T> {
    ob_base: <base class layout>,  // CPython header (PyObject or sub-layout)
    contents: UnsafeCell<T>,       // the actual Rust struct
    borrow_checker: <Storage>,     // empty or BorrowChecker depending on mutability
    // optional: dict offset, weakref offset (computed at type creation)
}
```

The layout is controlled by the `PyClassImpl` trait (defined in
`src/impl_/pyclass.rs`), which specifies offsets for the dict and weakref
list, and by `PyClassMutability` which determines whether a runtime borrow
checker is needed.

## Key Traits in `impl_.rs`

- `PyClassMutability` — associated types for `Storage` (borrow state) and
  `Checker` (the active checker type).
- `PyClassBorrowChecker` — implemented by `EmptySlot` (immutable classes),
  `BorrowChecker` (mutable, single-threaded), and `AtomicBorrowChecker`
  (free-threaded builds).
- `PyClassObjectLayout` — `tp_dealloc` and memory initialization logic.

## Relationships

- `src/pycell.rs` re-exports `PyBorrowError`, `PyBorrowMutError`,
  `PyClassObject` and the `PyRef`/`PyRefMut` aliases.
- `src/pyclass/guard.rs` builds `PyClassGuard`/`PyClassGuardMut` on top of
  the borrow checker stored here.
- `src/internal/get_slot.rs` provides `tp_free`/`tp_dealloc` lookups used
  by the dealloc logic in `impl_.rs`.
- `src/impl_/pyclass.rs` queries `PyClassObject` layout constants when
  populating the `PyTypeObject`.
