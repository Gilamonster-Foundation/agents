<p align="center">
  <img src="docs/logos/gilabot-256.png" alt="Gilamonster" width="256" height="256">
</p>

# Gilamonster Style Guides

Rules templates and skills for **Centaur Developers** — human/agent teams that
contribute as a single unit.

> "Computer Science has as much to do with Computers as Astronomy does with
> Telescopes." — Edsger W. Dijkstra

## What is a Centaur Developer?

In advanced chess, a "centaur" is a human paired with an engine — and for
years that pairing beat both the best humans *and* the best engines playing
alone. A **Centaur Developer** is the same idea applied to software: a human
contributor and one or more AI agents working together as one contributor —
shared context, shared standards, one body of work.

The division of labor is deliberate:

- **The human** brings intent, judgment, taste, and accountability. They
  decide *what* is worth building and *whether* what was built is right.
- **The agent** brings throughput, recall, and rigor-on-demand. It carries the
  checklists, runs the full test suite every time, never gets bored of writing
  the regression test, and never forgets the convention that was agreed on
  three months ago — because the convention is written down *here*.

Neither half works as well alone. The rules in this repository are the
connective tissue: they are how a human and an agent stay coherent across
sessions, projects, and time.

## Why rules as a repository?

Agents arrive empty. Every session starts from zero unless context is
deliberately preserved and handed back. That makes the rules file — the
`AGENTS.md`, the `CLAUDE.md`, the skill, the soul file — *load-bearing
infrastructure*, not documentation. It deserves the same treatment as code:

- **Versioned.** Rules change by pull request, with history and review.
- **Shared.** One canonical template, many projects — instead of N divergent
  copies that drift apart.
- **Plain text.** No lock-in, no proprietary formats. The rules outlive any
  particular tool that consumes them.

## What lives here

| Path | Contents |
|------|----------|
| `rules/` | Drop-in rules templates (`AGENTS.md` / `CLAUDE.md` style) for starting a new project or onboarding an agent to an existing one |
| `skills/` | Reusable, self-contained skills — procedural knowledge an agent can load on demand instead of carrying in its core rules |
| `souls/` | Role definitions: who an agent *is* in a given seat, so role-coherence is authored rather than accidental |
| `docs/logos/` | Brand assets (see below) |

(Sections land as they are extracted and generalized from working projects —
this repository grows by distillation, not speculation.)

## Ground rules of the style

The templates here encode a few non-negotiables of how Centaur teams work:

1. **The PR is the deliverable.** Conversation points at the work; the pull
   request *is* the work. Everything lands through review with CI gates.
2. **Hooks mirror pipelines.** Local push hooks run the same checks as CI.
   If the gate would fail remotely, it fails at the desk first.
3. **Every bug fix carries a regression test** that failed before the fix and
   passes after it. No exceptions.
4. **User data is sovereign.** Content, history, and metadata belong to the
   user in plain-text formats. Tools are replaceable; the work is not.
5. **Context is preserved, not reconstructed.** Decisions, conventions, and
   feedback get written down where the next session — human or agent — will
   find them.

## Crediting agents

Centaur work is honest about who did what. Agent and LLM contributions are
**disclosed in the git record**, not laundered through a human name — and not
inflated, either. The conventions:

### Identity format

An agent identity is written as `<model> <<model>@<harness>>` — the model
that did the thinking, at the harness that gave it hands. The address is
**synthetic, not a real email**; the `@<harness>` names the agent runtime
(its repository or deployment), giving every credit a resolvable home:

```text
nemotron@newt-agent          # nemotron model running in the newt-agent harness
qwen3-coder@newt-agent       # pin the model id you actually dispatched
```

Pin the model as precisely as the harness reports it (`nemotron3:33b` beats
`nemotron`). Vendor-prescribed forms are fine when a vendor defines one
(e.g. Claude Code's `Claude <model> <noreply@anthropic.com>` trailer); do not
invent addresses at real domains.

### Interactive (centaur) commits

When a human is at the keyboard steering, the human is the **Author** —
intent and accountability are theirs. The agent is credited with a trailer:

```text
Co-Authored-By: nemotron3:33b <nemotron3:33b@newt-agent>
```

### Autonomous (dispatched) commits

When a worker model produces the change end-to-end — dispatched by an
orchestrator, no human in the loop until review — the **worker model is the
Author**, using its synthetic identity:

```text
Author: nemotron3:33b <nemotron3:33b@newt-agent>
```

The human enters the record where they actually act: as the PR reviewer and
merger. Review is the accountability gate, so autonomous work always lands
through a pull request — never directly on a mainline branch.

### Why this matters

Credit is not ceremony — it is provenance. A model's record of contributions
is how competence is measured, compared, and selected for. You cannot build
a track record on commits that pretend someone else made them.

## Logos

The `docs/logos/` directory carries the Gilamonster mark at standard sizes:

| File | Size |
|------|------|
| `gilabot-512.png` | 512×512 |
| `gilabot-256.png` | 256×256 |
| `gilabot-128.png` | 128×128 |
| `gilabot-64.png` | 64×64 |
| `gilabot-32.png` | 32×32 |
| `gilabot-16.png` | 16×16 |

## License

Dual-licensed under [MIT](LICENSE-MIT) or [Apache-2.0](LICENSE-APACHE), at
your option.
