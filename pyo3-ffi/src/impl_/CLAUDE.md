# pyo3-ffi/src/impl_

Internal implementation helpers for `pyo3-ffi`. Nothing here is part of the public
API; all items are `#[doc(hidden)]` or used only inside the crate.

## mod.rs

Defines `AtomicCULong` — a platform-width atomic integer type that matches C's
`c_ulong` (32-bit on 32-bit targets, 64-bit on 64-bit targets). Only compiled under
`#[cfg(Py_GIL_DISABLED)]` for use in free-threaded CPython's reference-count fields.

## macros.rs (12 K)

Contains two layers of internal macros for handling Windows x86 name-mangling of
`_Py*` private symbols when using `raw-dylib`:

- `extern_libpython_cpython_private_fn!` — adds an extra leading underscore to the
  `link_name` on x86 Windows so that the `cdecl` undecoration leaves the symbol
  starting with `_Py` instead of `Py`.
- `extern_libpython_maybe_private_fn!` — dispatches on the function name: known
  `_Py*` functions route through `extern_libpython_cpython_private_fn!`; all others
  pass through unchanged. The list of matched names includes `_PyObject_CallFunction_SizeT`,
  `_PyObject_MakeTpCall`, `_PyBytes_Resize`, `_PyLong_AsByteArray`, and roughly 20
  other semi-private CPython exports.
- `extern_libpython!` (the outer macro, also in this file) — the actual `extern`
  block wrapper consumed by the rest of the crate to declare C function imports.

## When to edit this directory

Only when:
- The set of `_Py*` private functions exposed by `pyo3-ffi` changes (update the
  match arms in `extern_libpython_maybe_private_fn!`).
- The free-threaded build adds a new field type that needs a platform-width atomic.

Do not add public types or functions here; they belong in the top-level `src/` files.
