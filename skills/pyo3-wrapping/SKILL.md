---
name: pyo3-wrapping
description: >
  The Gilamonster house pattern for exposing Rust to Python: PyO3 0.28 +
  maturin, every crate usable from Python, bin wheels that never link
  libpython, the inverse-embedding trick for extensible servers, GIL/tokio
  bridging, and the README + examples/ requirements. Load before creating or
  modifying any Python bindings.
---

# PyO3 wrapping

## The shape: every crate usable from Python

A Rust workspace ships to Python as **multiple PyPI packages from one
repo**, built by maturin, version-locked together:

| Crate kind | maturin bindings | Links libpython? | Example |
|---|---|---|---|
| Binary (CLI, server) | `bindings = "bin"` | **never** | `modulex-cli`, `modulex-mcp` |
| Bindings crate (`*-py`) | `bindings = "pyo3"` + `extension-module` | loaded BY python | `modulex-py`, `agent-bridle-py` |

Check PyPI **and** crates.io name availability before scaffolding — bare
names are often squatted (`modulex`, `bridle` were taken; the `-py`/`-mcp`
prefix family was free). Never publish under a name you didn't verify.

## Rule 1: bin wheels never link libpython

A binary wheel that links libpython breaks on every minor Python mismatch,
and abi3 does not apply to `bindings = "bin"`. So:

- The Rust binaries stay pure Rust. Their pyproject:

```toml
[tool.maturin]
manifest-path = "Cargo.toml"
module-name = "modulex_cli"   # EXPLICIT — without it maturin derives an
bindings = "bin"              # invalid hyphenated module name (scrybe gotcha)
```

- If a server must be *extensible from Python*, use **inverse embedding**
  (Rule 2) — do not reach for `pyo3/auto-initialize` in the binary.

## Rule 2: inverse embedding for Python-extensible servers

Python hosts the engine; the engine never hosts Python. The `*-py` cdylib
exposes the core's entrypoints so a normal Python program can embed the
whole system and register Python callbacks in-process:

```python
import modulex_py
engine = modulex_py.Engine.from_config()

@engine.step("standup-notes")           # Python handler, in-process
def standup(spec: dict, ctx: dict) -> dict:
    return {"success": True, "output": "…"}

engine.serve_stdio()                     # the Rust server loop, Python
                                         # handlers included
```

To make this possible, structure the Rust side so the server loop lives in
a **library target** (bin is a thin main) and registries accept dynamic
registration. A Python callback wraps into the native handler trait; a
`&'static str` name requirement is satisfied with `Box::leak` (bounded:
names are few, registered once).

For *untrusted or isolated* plugins, prefer a **subprocess JSON protocol**
(one JSON object stdin → one JSON object stdout) over in-process loading —
it is language-agnostic and runs under the exec leash.

## Rule 3: GIL ↔ tokio bridging (pyo3 0.28 idioms)

One shared runtime, built lazily; detach the GIL around `block_on`; attach
from worker threads:

```rust
fn runtime() -> &'static tokio::runtime::Runtime {
    static RT: OnceLock<tokio::runtime::Runtime> = OnceLock::new();
    RT.get_or_init(|| tokio::runtime::Builder::new_multi_thread()
        .enable_all().build().expect("runtime"))
}

// Sync pymethod calling async Rust: DETACH, or callbacks deadlock.
fn run(&mut self, py: Python<'_>) -> PyResult<…> {
    let engine = self.ensure_built()?;
    py.detach(|| runtime().block_on(engine.run(…)))   // 0.28: detach
        .map_err(to_py_err)                            // (was allow_threads)
}

// Rust worker thread calling a Python callback: ATTACH (was with_gil),
// inside block_in_place so a slow handler doesn't starve the runtime.
let out = tokio::task::block_in_place(|| {
    Python::attach(|py| -> PyResult<Value> {
        let ret = self.callback.bind(py).call1((spec, ctx))?;
        py_any_to_json(&ret)
    })
});
```

Deadlock rule of thumb: the thread that holds the GIL must never block on
work that needs the GIL. `py.detach` before `block_on` is what breaks the
cycle.

Cross the boundary with JSON values (`py_any_to_json` / `json_to_py`
helpers — copy from agent-bridle-py or modulex-py). Check `bool` before
`int` (Python bool subclasses int). Python exceptions in callbacks map to
*soft* domain failures, never process aborts.

## Rule 4: README example + examples/ directory (mandatory)

Every PyPI-facing package:

1. **README.md** (this becomes the PyPI long_description) carries at least
   one complete, runnable Python example — `import x; x.do_thing()`,
   copy-paste-run, with real output shapes. Not pseudo-code.
2. The repo carries an **`examples/` directory** with at least one ordinary
   Python *program* (shebang, `main()`, exit codes) using the wheel the way
   a user would. Examples are kept compiling/running — treat a broken
   example as a broken build.

## Rule 5: test against the real wheel

- Rust-side: the `*-py` crate is cdylib-only with **no Rust test targets**
  (a test harness would try to link libpython); its surface is exercised
  by pytest.
- Python-side: `maturin develop` (or `maturin build` + `pip install` of
  the wheel) into a venv, then a real pytest suite under
  `crates/<name>-py/tests/` covering: registration, happy path, error
  mapping (Rust error → Python exception type), and lifecycle rules
  (e.g. "register before first run").
- **maturin ≥ 1.7** — 1.4-era versions mis-handle these layouts.
- `pytest.importorskip("<module>")` so the suite skips (not fails) where
  the wheel isn't installed; CI that builds the wheel runs it for real.

## Rule 6: workspace + packaging hygiene

- The `*-py` crate is `publish = false` on crates.io when it exists only
  for PyPI; internal deps pin exact versions (`=X.Y.Z`), all crates bump
  lock-step.
- `#![forbid(unsafe_code)]` holds in bindings crates too.
- Release pipeline (see newt-agent/modulex `release.yml`): maturin wheel
  matrix (bin wheels per-OS, cdylib per-interpreter via
  `--interpreter "3.9 3.10 3.11 3.12"`, `manylinux: auto` on Linux) +
  `pypa/gh-action-pypi-publish` with **`attestations: false`** when using
  an account-scoped API token (the action's OIDC default otherwise fails
  the upload), `skip-existing: true`, and a dist filter so binary tarballs
  never reach PyPI (`InvalidDistribution: No PKG-INFO` lesson).

## Checklist before opening the PR

- [ ] Names verified free on PyPI (and crates.io if publishing there)
- [ ] Bin wheels: `bindings = "bin"` + explicit `module-name`, no libpython
- [ ] cdylib: `extension-module`, no Rust test targets
- [ ] GIL: `detach` around `block_on`; `attach` (+`block_in_place`) in
      callbacks
- [ ] README has a runnable Python example
- [ ] `examples/` has a real program
- [ ] pytest green against a freshly built wheel (maturin ≥ 1.7)
- [ ] Python exceptions ↔ Rust errors mapped both directions, soft where
      the domain says soft
