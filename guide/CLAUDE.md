# guide/

mdBook user guide for PyO3. Published at https://pyo3.rs/. Source is in `src/`, built with mdBook.

## Build

```bash
mdbook build    # outputs to book/
mdbook serve    # live-reload dev server
```

Two Python preprocessors run before mdBook:
- `pyo3_version.py` — replaces `{{#pyo3_version}}` placeholders with the current version from `Cargo.toml`
- `glossary_linker.py` — auto-links glossary terms across all pages

## Structure

```
guide/
  src/
    SUMMARY.md          # table of contents (mdBook entry point)
    *.md                # top-level guide pages
    advanced/           # advanced topics (sharing types)
    building-and-distribution/  # multiple Python versions
    class/              # class customizations (protocols, object, numeric, call, thread-safety)
    conversions/        # tables, traits
    ecosystem/          # logging, tracing, async
    function/           # signature, error-handling
    python-from-rust/   # function-calls, calling-existing-code
  theme/
    tabs.css / tabs.js  # tab widget for OS-specific code examples
  book.toml             # mdBook configuration
  pyclass-parameters.md # standalone reference for #[pyclass] parameters (not in SUMMARY)
```

## Key Pages (by size / importance)

| File | Size | Content |
|---|---|---|
| `migration.md` | 97K | Appendix A: full migration guide across all versions |
| `class.md` | 51K | `#[pyclass]` — the comprehensive class reference |
| `building-and-distribution.md` | 26K | maturin, setuptools-rust, cross-compilation, ABI3 |
| `free-threading.md` | 17K | Free-threaded Python (GIL-disabled) support |
| `types.md` | 17K | Python object type system in PyO3 |
| `trait-bounds.md` | 16K | Appendix B: trait bound reference |
| `features.md` | 15K | Cargo feature flags reference |
| `debugging.md` | 15K | GDB/LLDB, Python memory debugging |
| `function.md` | 11K | `#[pyfunction]` reference |
| `python-typing-hints.md` | 11K | Appendix C: typing hints for PyO3 types |
| `faq.md` | 8K | Frequently asked questions |
| `async-await.md` | 5K | Async/await support |
| `exception.md` | 5K | Defining and raising Python exceptions |
| `module.md` | 5K | `#[pymodule]` reference |
| `type-stub.md` | 4K | Type stub generation via pyo3-introspection |
| `parallelism.md` | 9K | GIL management and Rayon patterns |

## Conventions

- Code blocks use the `tab` shortcode for Rust/Python paired examples.
- `{{#include ...}}` pulls in code snippets from the `tests/` or `src/` directories.
- The guide is versioned alongside the crate; `migration.md` documents every breaking change.
