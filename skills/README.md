# skills/

Reusable, self-contained skills — procedural knowledge an agent loads on
demand instead of carrying in its core rules. Each skill is a directory
with a `SKILL.md` (YAML frontmatter: `name`, `description`; body is the
procedure).

| Skill | When to load |
|-------|--------------|
| [`rust-tdd`](rust-tdd/SKILL.md) | Before nontrivial Rust implementation or any bug fix: red-green-refactor, mockable seams (no real processes/network in unit tests), the regression-test rule, coverage ratchets, jj-style test layering, compile_fail guarantees. |
| [`pyo3-wrapping`](pyo3-wrapping/SKILL.md) | Before creating or modifying Python bindings: maturin layouts (bin wheels never link libpython), inverse embedding for Python-extensible servers, GIL↔tokio bridging (detach/attach), README + examples/ requirements, wheel-testing and release-pipeline gotchas. |

Skills are distilled from working projects (newt-agent, modulex-mcp,
agent-bridle, scrybe, gilabot) — they grow by extraction, not speculation.
When a project teaches a new lesson, the skill gets the update in the same
spirit as a regression test.
