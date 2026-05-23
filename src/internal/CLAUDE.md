# src/internal/ — Private Utilities

Crate-private utilities that do not belong to any public module. Re-exported
via `src/internal.rs` as `crate::internal`. Nothing here is part of the public
API; downstream crates cannot access these types.

## Files

| File | Size | Description |
|---|---|---|
| `get_slot.rs` | 10K | `Slot<S>`, `GetSlotImpl` — type-safe accessor for `PyTypeObject` slot function pointers; abstracts over Limited API differences; used by `Bound<PyType>::get_slot()` to read `tp_dealloc`, `tp_free`, and other slots |
| `state.rs` | 20K | Thread-local GIL attachment state: `ATTACH_COUNT` thread-local counter, `AttachGuard` (RAII guard that increments/decrements the counter), `SuspendAttach` (releases the interpreter, used by sync primitives before blocking); also manages the reference pool for deferred `Py::drop` |

## Key Roles

`state.rs` is the lowest-level GIL tracking layer. It maintains a
per-thread `isize` counter (`ATTACH_COUNT > 0` means attached; a negative
value signals that GIL attach is temporarily forbidden, e.g., during
`__traverse__`). Every `Python::attach` call increments this counter; every
drop of the resulting guard decrements it.

`SuspendAttach` (used by `once_lock.rs` and `critical_section.rs`) drops
the attachment before a blocking call and restores it afterward, preventing
GIL-held deadlocks on mutex waits.

`get_slot.rs` exists because the Limited API (ABI3) does not expose
`PyTypeObject` fields directly — they must be read through `PyType_GetSlot`.
This module unifies both code paths behind the same `Slot<S>` const-generic
interface.

## Relationships

- `state.rs` is used by `src/marker.rs` (`Python::attach`) and
  `src/sync/once_lock.rs` (`PyOnceLock`).
- `get_slot.rs` is used by `src/pycell/impl_.rs` to find `tp_dealloc`/`tp_free`
  when deallocating `PyClassObject` memory.
