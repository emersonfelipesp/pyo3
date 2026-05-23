# src/conversions/ — Type Conversion Framework

Implements `IntoPyObject` and `FromPyObject` for types outside the `pyo3`
core: Rust standard library types (in `std/`) and optional third-party crate
types (sibling `.rs` files), each gated behind a matching Cargo feature flag.

## Third-Party Crate Integrations

Each file is compiled only when its feature flag is enabled in `Cargo.toml`.

| File | Feature | Converts |
|---|---|---|
| `anyhow.rs` | `anyhow` | `anyhow::Error` ↔ `PyErr` |
| `eyre.rs` | `eyre` | `eyre::Report` ↔ `PyErr` |
| `chrono.rs` | `chrono` | `NaiveDate`, `DateTime`, `Duration`, etc. ↔ `PyDate*` |
| `chrono_tz.rs` | `chrono-tz` | `chrono_tz::Tz` ↔ `PyTzInfo` (requires Python 3.9+) |
| `time.rs` | `time` | `time::Date`, `time::OffsetDateTime`, etc. ↔ `PyDate*` |
| `jiff.rs` | `jiff` | `jiff::civil::Date`, `jiff::Timestamp`, etc. ↔ Python datetime |
| `bytes.rs` | `bytes` | `bytes::Bytes` ↔ `PyBytes` |
| `num_bigint.rs` | `num-bigint` | `BigInt`, `BigUint` ↔ Python `int` |
| `num_complex.rs` | `num-complex` | `Complex<T>` ↔ `PyComplex` |
| `num_rational.rs` | `num-rational` | `Ratio<T>` ↔ Python `fractions.Fraction` |
| `bigdecimal.rs` | `bigdecimal` | `BigDecimal` ↔ Python `decimal.Decimal` |
| `rust_decimal.rs` | `rust_decimal` | `Decimal` ↔ Python `decimal.Decimal` |
| `ordered_float.rs` | `ordered-float` | `OrderedFloat<T>`, `NotNan<T>` ↔ Python `float` |
| `hashbrown.rs` | `hashbrown` | `HashMap`/`HashSet` ↔ Python `dict`/`set` |
| `indexmap.rs` | `indexmap` | `IndexMap` ↔ Python `dict` |
| `smallvec.rs` | `smallvec` | `SmallVec<A>` ↔ Python `list` |
| `uuid.rs` | `uuid` | `Uuid` ↔ Python `uuid.UUID` |
| `serde.rs` | `serde` | Derives `Serialize`/`Deserialize` for `Py<T>` |
| `either.rs` | `either` | `Either<L,R>` ↔ Python (tries left then right) |
| `mod.rs` | — | Module declarations; `std` is `pub(crate)` |

## std/ Subdirectory

Maps Rust standard library types. See `std/CLAUDE.md` for details.

## Core Traits

`IntoPyObject<'py>` (in `src/conversion.rs`) converts Rust → Python. It is
the successor to the older `ToPyObject` / `IntoPy` traits (both deprecated).
`FromPyObject<'a, 'py>` extracts Rust types from Python objects.

The two lifetimes on `FromPyObject` are: `'a` for the extracted reference's
lifetime and `'py` for the GIL token. When the extracted type owns its data
both lifetimes are unconstrained.
