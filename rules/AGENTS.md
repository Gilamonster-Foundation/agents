# AGENTS.md — Centaur Contributor Rules (template)

> **How to use this template:** copy it into a new repository as `AGENTS.md`,
> fill in the `<angle-bracket>` blanks, delete sections that genuinely do not
> apply (a pure-Python repo drops the Rust section), and keep the rest. If
> your harness reads `CLAUDE.md` instead, mirror this file there and state
> which one is canonical. Rules change by pull request, like code.
>
> Canonical source: <https://github.com/Gilamonster-Foundation/agents>
> (`rules/AGENTS.md`). When this copy and the canonical template diverge on a
> ground rule, propose the fix upstream rather than forking the rule.

# <Project Name> — Agent Instructions

## What this repo is

<One paragraph: what the project does, who it serves, and the one-sentence
"telescope" statement — what the tool is FOR, beyond what it does.>

> "Computer Science has as much to do with Computers as Astronomy does with
> Telescopes." — Edsger W. Dijkstra

## Non-negotiables

These are the ground rules every contribution — human, agent, or both —
lands under. They are not style preferences; CI and review enforce them.

### 1. A human is always in the loop

- Agents **never** push to the default branch and **never** merge.
- Every change lands through a pull request that a human reviews and merges.
  Review is the accountability gate; autonomous work without a human merge
  is not a contribution, it is a liability.
- Hard-to-reverse actions (force-push, history rewrites, deletions of
  non-dead code, publishing artifacts) require explicit human approval
  *every time* — approval in one context does not carry to the next.

### 2. Identify the model that did the work

Agent and LLM contributions are disclosed in the git record — not laundered
through a human name, and not inflated either. Identity format is
`<model> <<model>@<harness>>` (synthetic address; the harness names the
runtime that gave the model hands):

- **Interactive (centaur) commits** — human at the keyboard steering: the
  human is Author; the agent is credited with a trailer:
  `Co-Authored-By: nemotron3:33b <nemotron3:33b@newt-agent>`
- **Autonomous (dispatched) commits** — worker model end-to-end: the model
  is Author with its synthetic identity; the human enters the record as
  reviewer and merger.
- Pin the model as precisely as the harness reports it (`nemotron3:33b`
  beats `nemotron`). Vendor-prescribed trailers are fine where a vendor
  defines one; never invent addresses at real domains.

Credit is provenance: a model's track record is how competence is measured
and selected for.

### 3. TDD — red, green, refactor

- Write the failing test first. Watch it fail for the right reason. Make it
  pass. Then clean up.
- **Every bug fix carries a regression test** that (a) fails on the old
  code, (b) passes on the fix, and (c) documents the bug in its name or
  docstring (reference the issue number). Verify the failure on the old
  code path before committing. A fix without its test is incomplete.
- Load the `rust-tdd` skill (skills/rust-tdd/) before nontrivial Rust work.

### 4. Coverage floor: 80%, ratchets up, never down

- Workspace test coverage holds at or above **80%** (line coverage via
  `cargo llvm-cov` / `pytest --cov`, as the project wires it).
- A bootstrap project may start lower with the floor written into CI — but
  the floor only ever moves up. Lowering it is not a code change, it is a
  policy change, and it needs the human to say so in the PR description.

### 5. Every crate/package ships its own README

- crates.io and PyPI render the package README as its front page. A new
  crate or package lands **with** a `README.md` in its root: what it is,
  how to install it, one runnable example.
- Every version bump includes a README freshness review; a bump PR that
  leaves the README untouched says why. A stale README is an incomplete
  release.

### 6. PyO3 / Python-facing packages show working Python

- The package README (= the PyPI long_description) contains at least one
  **complete, runnable** Python example: `import x` … `x.do_thing()` —
  copy-paste-run, no pseudo-code.
- The repository carries an **`examples/` directory** with at least one
  ordinary Python program using the wheel the way a user would (not a test,
  not a snippet — a program).
- Load the `pyo3-wrapping` skill (skills/pyo3-wrapping/) before building or
  modifying bindings.

### 7. Hooks mirror pipelines

- The pre-push hook runs the same checks as CI, with the same flags and
  toolchain pins. If the gate would fail remotely, it fails at the desk
  first.
- Editing a CI pipeline file REQUIRES auditing the hook in the same change
  (and vice versa). Both carry cross-reference comments naming each other.
- Never bypass hooks (`--no-verify` is not a tool you have).

### 8. Zero warnings

- Rust: `cargo clippy --all-targets --all-features -- -D warnings` and
  `cargo fmt -- --check` are clean.
- Python: `ruff check` clean, `black --check` passes.
- Warnings that accumulate become noise that hides real defects.

### 9. No credentials, ever

- No secrets in code, config, tests, examples, issues, or commit messages.
  Config carries *references* to where a secret lives (env var name, file
  path, command), never values.
- The user's data is sovereign: plain-text formats, exportable state, no
  lock-in.

## Code style (Rust)

We write Rust in the spirit of the [jujutsu (jj) style guide] — the
load-bearing points:

- **Panics are not allowed** in library code. `.unwrap()`/`slice[0]` only
  where a previous check or a *documented invariant* guarantees safety.
  Errors are values; soft-fail where the design calls for resilience.
- **Doc comments** (`///`) on public types, functions, and fields, following
  rust-lang RFC 1574: first line is a single short sentence in third-person
  indicative ("Returns …", "Spawns …"), blank line, then complete sentences.
- **Prefer lower-level tests to end-to-end tests.** Test the library seam;
  use a handful of end-to-end cases to prove the wiring, not the logic.
  Mock external resources (network, subprocess, filesystem, clocks) behind
  injectable seams — unit tests spawn no real processes and touch no real
  network.
- Wrap Markdown (and these rules) at ~80 columns.
- Explain *why* in comments, not *what*; name things so the *what* is plain.

[jujutsu (jj) style guide]: https://github.com/jj-vcs/jj/blob/main/docs/style_guide.md

## Build, test, validate

```bash
just check          # fmt + clippy + tests — THE gate; hooks and CI run this
just install-hooks  # REQUIRED after clone
<project-specific: coverage command, demo command, …>
```

## Branch + PR policy

- Branch from the default branch; one logical change per branch; branches
  live hours-to-days, not weeks.
- Branch name: `<area>/<short-kebab-description>` (project may refine).
- PR body: `## Summary`, `## Test plan`, and `Fixes #N` when it closes an
  issue. Title under 70 characters.
- Delete branches after merge. Stale branches are tech debt.
- All tests pass — including the ones you didn't touch.

## Context is preserved, not reconstructed

Decisions, conventions, and feedback get written down where the next
session — human or agent — will find them: this file, the issue tracker,
ADRs, or the project's knowledge base. If you had to discover something the
hard way, write it down before you finish.

## Project specifics

<Project-specific sections go below: architecture map, key seams, things
that bit us, deploy notes. Keep the generic rules above intact so they can
be diffed against the canonical template.>
