# pyo3-introspection

CLI tool and library for introspecting compiled PyO3 extension modules (`.so` / `.pyd`) and generating Python type stub files (`.pyi`).

## Purpose

When building a PyO3 extension with the `experimental-inspect` feature, PyO3 embeds structured metadata in a dedicated linker section of the compiled binary. `pyo3-introspection` reads that section from the binary, parses it, and emits `.pyi` stub files — without importing the module or running any Python code.

## Source Files

- `src/main.rs` (1.3K) — CLI entry point. Takes three arguments: path to binary, module name, output directory. Calls `introspect_cdylib` then `module_stub_files` and writes the results.
- `src/introspection.rs` (25K) — reads the binary using `goblin` (ELF/Mach-O/PE parser) and extracts the PyO3 metadata section. Deserializes JSON embedded in the linker section.
- `src/stubs.rs` (30K) — converts the parsed introspection model into formatted `.pyi` stub file content.
- `src/model.rs` (3.4K) — data model types representing modules, classes, functions, and their type annotations as extracted from the binary.
- `src/lib.rs` — library entry point re-exporting `introspect_cdylib` and `module_stub_files`.

## Dependencies

- `goblin` — cross-platform binary parsing (ELF, Mach-O, PE/COFF)
- `serde` + `serde_json` — deserialization of embedded JSON metadata
- `anyhow` — error handling

## Usage

```
pyo3-introspection <binary_path> <module_name> <output_dir>
```

Example: introspecting a compiled `mymodule.cpython-312-x86_64-linux-gnu.so` generates `mymodule.pyi` in the output directory.

## Relationship to the Main Crate

- Depends on `experimental-inspect` feature being enabled in `pyo3` at extension build time. Without it, no metadata is embedded and introspection will fail.
- The guide documents this workflow in `guide/src/type-stub.md`.
- Not part of the normal workspace build; used as a separate tool.

## Supported Platforms

Supports any platform goblin can parse: Linux (ELF), macOS (Mach-O), Windows (PE/COFF).
