# src/conversions/std/ — Rust Standard Library Conversions

Implements `IntoPyObject` and `FromPyObject` for the most common Rust `std`
types. All files are `pub(crate)` only — the implementations are registered
globally through the trait system and are not re-exported as a module.

## Files

| File | Size | Rust Type(s) → Python |
|---|---|---|
| `num.rs` | 37K | All primitive integers (`i8`…`u128`, `isize`, `usize`) and `f32`/`f64` ↔ Python `int`/`float`; also `bool` |
| `string.rs` | 8K | `String`, `&str`, `Cow<str>` ↔ `PyString` |
| `osstr.rs` | 12K | `OsStr`, `OsString` ↔ `PyString` (platform encoding aware) |
| `path.rs` | 8K | `Path`, `PathBuf` ↔ `PyString`/`os.PathLike` |
| `cstring.rs` | 7K | `CStr`, `CString` ↔ `PyBytes` |
| `vec.rs` | 5K | `Vec<T>` ↔ `PyList` |
| `array.rs` | 10K | `[T; N]` ↔ `PyList`; const-generic over N |
| `slice.rs` | 6K | `&[T]` → `PyList` (extract-only via `Vec`) |
| `map.rs` | 7K | `HashMap<K,V>`, `BTreeMap<K,V>` ↔ `PyDict` |
| `set.rs` | 6K | `HashSet<T>`, `BTreeSet<T>` ↔ `PySet`/`PyFrozenSet` |
| `option.rs` | 2K | `Option<T>` ↔ Python object or `None` |
| `cell.rs` | 1K | `Cell<T>` — thin wrapper (copies T in/out) |
| `ipaddr.rs` | 6K | `IpAddr`, `Ipv4Addr`, `Ipv6Addr` ↔ Python `ipaddress` objects |
| `time.rs` | 13K | `std::time::Duration`, `SystemTime` ↔ Python `datetime.timedelta` / `datetime.datetime` |
| `mod.rs` | <1K | Private module declarations |

## Notes

`num.rs` is the largest and most complex file (37K). It handles all primitive
numeric conversions including overflow checking and the `__index__` protocol
for extracting Rust integers from Python ints. The conversion of `f32`/`f64`
also handles `NaN` and infinity correctly.

`option.rs` maps `None` to Python's `None` singleton and `Some(x)` to the
conversion of `x`. On extraction, Python `None` maps to `None`; any other
object is extracted as `Some(T::extract(obj))`.

`ipaddr.rs` converts to/from Python's `ipaddress.IPv4Address` and
`ipaddress.IPv6Address` classes, importing the `ipaddress` module lazily.

`time.rs` (std) is distinct from `conversions/time.rs` (the `time` crate).
