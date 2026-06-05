# rules/

Drop-in rules templates for starting a new project or onboarding an agent
to an existing one.

| File | Use |
|------|-----|
| [`AGENTS.md`](AGENTS.md) | The Centaur contributor template: copy into a repo root, fill the blanks, delete what doesn't apply. Encodes the non-negotiables — human in the loop, model crediting, TDD + regression-test rule, 80% coverage floor, README-per-crate, PyO3 example requirements, hook/pipeline parity, zero warnings, no credentials — plus the jj-style Rust code rules. |

Conventions: templates are harness-agnostic (`AGENTS.md` first; mirror to
`CLAUDE.md` where a harness wants it and say which is canonical). Rules
change by pull request; project copies that improve a ground rule send the
fix back here.
