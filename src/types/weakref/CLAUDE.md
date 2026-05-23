# src/types/weakref/ — Weak Reference Type Bindings

Bindings for Python's `weakref` module types. Exported from `src/types/mod.rs`
as `PyWeakref`, `PyWeakrefMethods`, `PyWeakrefProxy`, and `PyWeakrefReference`.

## Files

| File | Size | Description |
|---|---|---|
| `mod.rs` | 189B | Re-exports `PyWeakref`, `PyWeakrefMethods`, `PyWeakrefProxy`, `PyWeakrefReference` |
| `anyref.rs` | 23K | `PyWeakref`, `PyWeakrefMethods` — abstract weak reference (covers both reference and proxy) |
| `proxy.rs` | 33K | `PyWeakrefProxy` — transparent weak proxy (`weakref.proxy(obj)`) |
| `reference.rs` | 17K | `PyWeakrefReference` — explicit weak reference (`weakref.ref(obj)`) |

## Relationships

- `PyWeakref` is the abstract base; `PyWeakrefProxy` and `PyWeakrefReference`
  are the two concrete subtypes.
- All three follow the standard `Py*` / `Py*Methods` pattern from `types/`.
- Consumed by `src/types/mod.rs` which re-exports the whole set.
- PyClass objects that declare `weakref` support (via `#[pyclass(weakref)]`)
  interact with these types through `src/impl_/pyclass.rs` slot wiring.
