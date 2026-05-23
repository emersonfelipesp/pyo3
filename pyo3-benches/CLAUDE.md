# pyo3-benches

In-tree Criterion benchmark suite for PyO3. Not published to crates.io (`publish = false`). Lives in its own Cargo workspace (`[workspace]` at the bottom of `Cargo.toml`) to avoid affecting the main workspace build.

## Structure

All benchmarks are in `benches/`, each declared as a `[[bench]]` entry with `harness = false` (Criterion handles its own harness).

## Benchmark Files (20 total)

| File | What it measures |
|---|---|
| `bench_any.rs` | `PyAny` downcasting and type checking |
| `bench_attach.rs` | GIL attach overhead |
| `bench_bigint.rs` | Python `int` ↔ `num_bigint::BigInt` conversion |
| `bench_call.rs` | Various forms of calling Python callables from Rust |
| `bench_comparisons.rs` | Rich comparison operators on PyO3 objects |
| `bench_critical_sections.rs` | Free-threaded critical section primitives |
| `bench_decimal.rs` | `rust_decimal` ↔ Python `Decimal` conversion |
| `bench_dict.rs` | `PyDict` creation, insertion, lookup |
| `bench_err.rs` | `PyErr` creation and matching cost |
| `bench_extract.rs` | `.extract::<T>()` for various types |
| `bench_frompyobject.rs` | `FromPyObject` derive macro performance |
| `bench_int128.rs` | Python `int` ↔ `i128`/`u128` conversion |
| `bench_intern.rs` | `intern!()` macro for interned string overhead |
| `bench_intopyobject.rs` | `IntoPyObject` derive macro performance |
| `bench_list.rs` | `PyList` creation, append, iteration |
| `bench_py.rs` | `Py<T>` / `PyObject` clone and drop cost |
| `bench_pyclass.rs` | `#[pyclass]` instantiation and method calls |
| `bench_pystring_from_fmt.rs` | `PyString` creation via format strings |
| `bench_set.rs` | `PySet` operations |
| `bench_tuple.rs` | `PyTuple` creation, index access, iteration |

## Dependencies

- `pyo3` (from workspace path, features: `auto-initialize`, `full`)
- `criterion 0.8` + `codspeed-criterion-compat 4.0` (CodSpeed CI integration)
- `num-bigint`, `rust_decimal`, `hashbrown` (for type-specific benchmarks)
- `pyo3-build-config` (build-dependency for Python cfg flags)

## Running Benchmarks

```bash
cargo bench -p pyo3-benches
# Run a specific bench:
cargo bench -p pyo3-benches --bench bench_call
# ABI3 variant:
cargo bench -p pyo3-benches --features abi3
```

## Relationship to CI

CodSpeed tracks benchmark regressions automatically via the `codspeed-criterion-compat` adapter. The `bench_critical_sections` benchmark is specific to free-threaded Python (`Py_GIL_DISABLED`).
