# plugin

Example demonstrating embedding Python in a Rust binary (the reverse of the normal extension module pattern), with a plugin system where Python code creates and manipulates Rust objects.

## What It Demonstrates

- `pyo3::append_to_inittab!` — registering a module before Python initializes
- `Python::initialize()` — starting the Python interpreter from Rust
- `Python::attach()` — acquiring the GIL and running Python code from a native binary
- `PyRefMut<T>` — mutable borrow of a `#[pyclass]` instance from Rust
- A plugin architecture: Python creates a `Gadget`, Rust modifies it natively

## Sub-crates

| Directory | Role |
|---|---|
| `plugin_api/` | Rust library defining `Gadget` (`#[pyclass]`) and `plugin_api` module |
| `python_plugin/` | Python scripts: `gadget_init_plugin.py` (creates `Gadget`), `rng.py` |
| `src/main.rs` | Rust binary: initializes Python, loads the plugin, mutates the Gadget |

## Key Pattern

```rust
pyo3::append_to_inittab!(pylib_module);  // register before init
Python::initialize();
Python::attach(|py| {
    let plugin = PyModule::import(py, "gadget_init_plugin")?;
    let gadget = plugin.getattr("start")?.call0()?;
    let mut gadget_rs: PyRefMut<'_, plugin_api::Gadget> = gadget.extract()?;
    gadget_rs.prop = 42;  // native Rust mutation, reflected in Python
    Ok(())
})
```

## Structure Notes

- `plugin_api/src/lib.rs` — defines `Gadget` with a public `prop: usize` (Python-accessible) and `rustonly: Vec<usize>` (Rust-only field, not visible to Python)
- `python_plugin/gadget_init_plugin.py` — creates a `Gadget` and returns it from `start()`
- This is a binary (`src/main.rs`), not a `cdylib`; it links Python directly

## Build

This example uses maturin for the `plugin_api` sub-crate and runs as a native Rust binary. See `noxfile.py` for the build steps.
