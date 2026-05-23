# src/types/ — Python Built-in Type Bindings

Safe Rust wrappers for CPython's built-in types. Each file defines one or more
`Py*` structs and an associated `Py*Methods` trait containing the actual API.
The `Py*Methods` traits are implemented for `Bound<'py, Py*>` so all methods
require a live GIL token via the `'py` lifetime.

`mod.rs` re-exports all public types and houses shared macros used across the
module. `PyAny` in `any.rs` is the root type — every other Python type derefs
to it.

## Files

| File | Size | Types / Notes |
|---|---|---|
| `any.rs` | 68K | `PyAny`, `PyAnyMethods` — root type; all Python objects are `PyAny` |
| `dict.rs` | 56K | `PyDict`, `PyDictMethods`, `IntoPyDict`; includes `PyDictKeys/Values/Items` |
| `list.rs` | 58K | `PyList`, `PyListMethods` |
| `tuple.rs` | 58K | `PyTuple`, `PyTupleMethods` |
| `capsule.rs` | 41K | `PyCapsule`, `PyCapsuleMethods`, `CapsuleName` — C-compatible pointer containers |
| `string.rs` | 32K | `PyString`, `PyStringMethods`, `PyStringData` (non-limited API) |
| `datetime.rs` | 32K | `PyDate`, `PyDateTime`, `PyTime`, `PyDelta`, `PyTzInfo`; access traits |
| `sequence.rs` | 26K | `PySequence`, `PySequenceMethods` — abstract sequence protocol |
| `module.rs` | 19K | `PyModule`, `PyModuleMethods` |
| `mappingproxy.rs` | 18K | `PyMappingProxy` — read-only dict view |
| `iterator.rs` | 15K | `PyIterator`; `PySendResult` (3.10+ send protocol) |
| `bytearray.rs` | 21K | `PyByteArray`, `PyByteArrayMethods` |
| `frame.rs` | 11K | `PyFrame`, `PyFrameMethods` (non-limited API, non-PyPy/GraalPy) |
| `typeobject.rs` | 13K | `PyType`, `PyTypeMethods` |
| `set.rs` | 12K | `PySet`, `PySetMethods` |
| `frozenset.rs` | 9K | `PyFrozenSet`, `PyFrozenSetMethods`, `PyFrozenSetBuilder` |
| `mapping.rs` | 11K | `PyMapping`, `PyMappingMethods` — abstract mapping protocol |
| `mutex.rs` | 15K | `PyMutex`, `PyMutexGuard` (3.13+, non-limited API) |
| `bytes.rs` | 15K | `PyBytes`, `PyBytesMethods` |
| `float.rs` | 11K | `PyFloat`, `PyFloatMethods` |
| `boolobject.rs` | 10K | `PyBool`, `PyBoolMethods` |
| `num.rs` | 4K | `PyInt` — Python `int` object |
| `complex.rs` | 9K | `PyComplex`, `PyComplexMethods` |
| `slice.rs` | 8K | `PySlice`, `PySliceMethods`, `PySliceIndices` |
| `traceback.rs` | 7K | `PyTraceback`, `PyTracebackMethods` |
| `function.rs` | 7K | `PyCFunction`; `PyFunction` (non-limited API) |
| `range.rs` | 3K | `PyRange`, `PyRangeMethods` |
| `ellipsis.rs` | 2K | `PyEllipsis` — the `...` singleton |
| `none.rs` | 2K | `PyNone` — the `None` singleton |
| `notimplemented.rs` | 3K | `PyNotImplemented` — the `NotImplemented` singleton |
| `pysuper.rs` | 2K | `PySuper` |
| `genericalias.rs` | 3K | `PyGenericAlias` (3.9+) |
| `memoryview.rs` | 2K | `PyMemoryView` |
| `code.rs` | 5K | `PyCode`, `PyCodeInput`, `PyCodeMethods` |
| `mod.rs` | 12K | Public re-exports and shared iteration macros |

## Subdirectories

| Directory | Contents |
|---|---|
| `weakref/` | Weak reference types: `PyWeakref`, `PyWeakrefReference`, `PyWeakrefProxy` |

## Pattern

All types follow the `Py*` / `Py*Methods` split: the struct is a marker type
wrapping `PyAny`; all methods live in the trait which is blanket-implemented
for `Bound<'py, Py*>`. This allows methods to be called directly on a
`Bound<'py, PyList>` etc.

`datetime.rs` is special: it uses `PyDateTime_IMPORT` lazily and has
additional accessor traits (`PyDateAccess`, `PyTimeAccess`, `PyDeltaAccess`)
that are gated on `not(Py_LIMITED_API)`.
