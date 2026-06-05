---
name: rust-tdd
description: >
  Test-driven development discipline for Rust workspaces: red-green-refactor,
  mockable seams instead of real processes/network, the regression-test rule,
  coverage ratchets, and jj-style test layering. Load before nontrivial Rust
  implementation or bug-fix work in any Gilamonster project.
---

# Rust TDD

## The loop

1. **Red** — write the test that describes the behavior you want. Run it.
   Watch it fail *for the right reason* (a compile error is not a failing
   test; an assertion failure on the right axis is).
2. **Green** — write the least code that makes it pass. Resist gold-plating.
3. **Refactor** — clean up with the test as your net. Re-run the whole
   suite, not just the new test.

Never write the implementation first and back-fill tests "to cover it" —
back-filled tests encode what the code *does*, not what it *should do*.

## The regression-test rule (non-negotiable)

Every bug fix includes at least one test that:

1. **fails on the old code** — check out / stash the fix and run it, or
   write the test first against the buggy behavior;
2. **passes after the fix**;
3. **names the bug** — test name or doc comment references the issue and
   states the broken behavior, e.g.:

```rust
#[test]
fn declared_programs_include_cmd_credentials() {
    // Regression (#41): {cmd=..} credential programs were missing from
    // the declared default grant, so every reference was denied at run
    // time and its step silently skipped.
    ...
}
```

A fix without its regression test is an incomplete fix. Do not commit it.

## Seams, not side effects

Unit tests spawn **no real processes**, touch **no real network**, and
depend on **no wall-clock**. Anything external lives behind an injectable
trait seam with a production impl and a canned test impl:

```rust
/// The mockable spawn seam. Production uses TokioSpawner; tests inject
/// canned outputs.
#[async_trait]
pub trait Spawner: Send + Sync {
    async fn spawn(&self, req: &ExecRequest) -> io::Result<ExecOutput>;
}

pub struct TokioSpawner;          // production: real subprocess
pub struct MockSpawner { ... }    // tests: scripted outputs + recorded calls
```

Patterns that pay for themselves:

- The mock **records calls** (`Vec<(program, args)>`) so tests assert *what
  would have run*, not just the output.
- Expose the test double to downstream crates behind a feature:
  `#[cfg(any(test, feature = "test-support"))] pub mod test_support` —
  consumers add it in `[dev-dependencies]` only.
- **Pin time**: factor date/time-dependent logic into pure functions taking
  `today: NaiveDate` (etc.) and test those; the production caller reads the
  clock once at the edge. Never use wall-clock as a coordination primitive
  anywhere — counters and generations order events.
- One (and only one) integration test may exercise the real seam
  end-to-end against something harmless (`echo`), to prove the production
  impl is wired.

## Test layering (jj style)

Prefer lower-level tests to end-to-end tests — they are ~100x faster and
reach edge cases e2e can't:

- **Unit tests** (`#[cfg(test)] mod tests` in the module): the logic, the
  edge cases, the error paths. The bulk of the suite.
- **Integration tests** (`tests/`): the crate's public API as a consumer
  sees it; cross-module behavior; the one real-seam test.
- **End-to-end** (binary in, output out): a handful of cases proving the
  CLI/server wiring — flags respected, exit codes right — never the logic.

If a behavior needs an e2e test to be testable, that's a design smell:
extract the logic behind a seam.

## Compile-time guarantees are tests too

When the invariant is "this must never be possible", enforce it in the type
system and pin it with a `compile_fail` doctest:

```rust
/// `Secret` must never become serializable.
///
/// ```compile_fail
/// fn requires_serialize<T: serde::Serialize>() {}
/// requires_serialize::<crate::Secret>();
/// ```
pub struct Secret(String);
```

## Coverage ratchet

- The workspace floor is **80%** (line coverage). Bootstrap projects may
  start lower **with the floor written into CI** (`just cov-ci` or
  equivalent), and the floor only moves UP.
- `cargo llvm-cov --workspace --fail-under-lines <floor>` is the standard
  wiring; put the floor in one place (justfile variable) so the ratchet is
  a one-line diff with history.
- Coverage is a tripwire, not a target: do not write assert-free tests to
  inflate it. A test that can't fail is worse than no test.

## Async + concurrency testing

- `#[tokio::test]` for async units; multi-thread runtime only when the
  code under test requires it.
- Test ordering guarantees deterministically: feed a concurrent component
  scripted inputs and assert the *re-ordered, deterministic* output (e.g.
  parallel batch results re-sorted to config order) rather than sleeping.
- Budget/limit logic gets an exhaustion test (N allowed, N+1 denied) and a
  "denied attempt does not consume budget" test.

## Definition of done

```bash
cargo fmt -- --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test --all          # ALL tests, not just yours
<coverage gate>           # at or above the floor
```

All green locally **before** push — the pre-push hook runs the same gate,
and CI runs it again. If any step is red, the work is not done, no matter
how good the diff looks.
