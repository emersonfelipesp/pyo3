# pyo3-build-config

Build-time Python detection and configuration crate. Used as a `build-dependency` by `pyo3`, `pyo3-ffi`, `pyo3-benches`, `pyo3-pytests`, and `pyo3-ffi-check`. It runs at compile time, not at runtime.

## Role

Detects the Python interpreter, reads its configuration (via `sysconfig` or `PYO3_CONFIG_FILE`), and emits `cargo:rustc-cfg` flags consumed by the rest of the workspace. These flags gate version-specific code across all PyO3 crates.

## Source Files

- `src/impl_.rs` (115K) — the bulk of the implementation: interpreter detection, sysconfigdata parsing, cross-compilation logic, config serialization, triple parsing via `target-lexicon`.
- `src/lib.rs` (18K) — public API surface: `use_pyo3_cfgs()`, `add_extension_module_link_args()`, `InterpreterConfig`, `BuildFlags`, `PythonVersion`, `PythonImplementation`, `CrossCompileConfig`, and re-exports from `impl_`.
- `src/errors.rs` — error types for detection failures.

## Key Public API

- `use_pyo3_cfgs()` — call from a `build.rs` to emit all `#[cfg(Py_3_N)]` / `#[cfg(PyPy)]` / `#[cfg(Py_GIL_DISABLED)]` flags.
- `add_extension_module_link_args()` — emits platform-specific linker args for extension modules (e.g., `-undefined dynamic_lookup` on macOS).
- `InterpreterConfig` — the parsed result of Python interpreter interrogation; can be serialized to/from a config file.
- `find_all_sysconfigdata()` / `parse_sysconfigdata()` — helpers for cross-compilation scenarios.

## cfg Flags Emitted

| Flag | Meaning |
|---|---|
| `Py_3_8` through `Py_3_14` | Code valid on that version and above |
| `Py_LIMITED_API` | ABI3 / Limited API build |
| `Py_GIL_DISABLED` | Free-threaded CPython |
| `PyPy` | Compiling for PyPy |
| `GraalPy` | Compiling for GraalPy |

## Key Environment Variables (read at build time)

| Variable | Effect |
|---|---|
| `PYO3_CONFIG_FILE` | Path to a hand-written config file; skips interpreter detection entirely |
| `PYO3_NO_PYTHON` | Disables all Python detection; useful for cross-compilation |
| `PYO3_CROSS_LIB_DIR` | Directory containing the Python library when cross-compiling |
| `PYO3_CROSS_PYTHON_VERSION` | Target Python version for cross-compilation |
| `PYO3_CROSS_PYTHON_IMPLEMENTATION` | `CPython`, `PyPy`, or `GraalPy` |

## Relationships

- `pyo3/build.rs` calls `pyo3_build_config::use_pyo3_cfgs()` and forwards metadata to downstream crates via `links = "pyo3-python"` in `Cargo.toml`.
- Downstream crates that depend on `pyo3` receive the build metadata automatically without needing their own dependency on this crate.
- `pyo3-benches` and `pyo3-pytests` depend on this crate directly in `[build-dependencies]`.

## Deprecated Features

`resolve-config`, `extension-module`, and `generate-import-lib` features are deprecated no-ops kept for compatibility.
