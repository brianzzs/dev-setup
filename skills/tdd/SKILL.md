---
name: tdd
description: Test-driven development for .NET with XUnit. Use when the user wants to build features or fix bugs test-first, mentions "red-green-refactor", or wants integration tests.
---

# Test-Driven Development (.NET / XUnit)

TDD is the red → green loop. This skill is the reference that makes that loop produce tests worth keeping: what a good test is, where tests go, the anti-patterns, and the rules of the loop. Every section applies on every cycle: consult them before and during the loop, not after.

Use **XUnit** for every test project. Never use MSTest, even if an older project already contains it — flag it instead of matching it.

When exploring the codebase, read `CONTEXT.md` (if it exists) so test names and interface vocabulary match the project's domain language, and respect ADRs in the area you're touching.

## What a good test is

Tests verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't. A good test reads like a specification: "user can checkout with valid cart" tells you exactly what capability exists, and it survives refactors because it doesn't care about internal structure.

See [tests.md](tests.md) for examples and [mocking.md](mocking.md) for mocking guidelines.

## Seams: where tests go

A **seam** is the public boundary you test at: the interface where you observe behavior without reaching inside. Tests live at seams, never against internals.

**Test only at pre-agreed seams.** Before writing any test, write down the seams under test and confirm them with the user. No test is written at an unconfirmed seam. You can't test everything, so agreeing the seams up front is how testing effort lands on the critical paths and complex logic instead of every edge case.

Ask: "What's the public interface, and which seams should we test?" Before asking, check for a plan file at `.claude/plans/*.md` matching the work — the output of a prior `grill-me` session. If one exists, read it: its "Seams to implement" section satisfies this confirmation directly, and its "Decisions" section is ground truth for behavior. Don't re-litigate what it settled; only ask about what it left open. If no plan file exists but a `grill-me` round happened earlier in this same conversation, that confirmation still counts — the file is for surviving across sessions, not a requirement within one.

When the shape of that interface is itself in question (how deep the module is, where the seam belongs, what the interface should expose), call the Skill tool with "codebase-design" for the vocabulary. It is the shared source of the module, interface, depth, seam, adapter, leverage and locality terms, and it is a reference to consult, not a session to run.

## Anti-patterns

- **Implementation-coupled**: mocks internal collaborators, tests private methods, or verifies through a side channel (querying the database instead of using the interface). The tell: the test breaks when you refactor but behavior hasn't changed.
- **Tautological**: the assertion recomputes the expected value the way the code does (`Assert.Equal(a + b, Add(a, b))`, a snapshot derived by hand the same way, a constant asserted equal to itself), so it passes by construction and can never disagree with the code. Expected values must come from an independent source of truth: a known-good literal, a worked example, the spec.
- **Horizontal slicing**: writing all tests first, then all implementation. Bulk tests verify _imagined_ behavior: you test the _shape_ of things rather than user-facing behavior, the tests go insensitive to real changes, and you commit to test structure before understanding the implementation. Work in **vertical slices** instead: one test → one implementation → repeat, each test a **tracer bullet** that responds to what the last cycle taught you.

## Rules of the loop

- **Red before green.** Write the failing test first, then only enough code to pass it. Don't anticipate future tests or add speculative features.
- **One slice at a time.** One seam, one test, one minimal implementation per cycle.
- **Refactoring is not part of the loop.** Once the vertical slices are done, hand the diff to the `code-review` skill — its Standards axis carries a test-quality baseline that catches exactly the anti-patterns above (implementation coupling, tautological assertions, verification overreach), in addition to production-code smells. `code-review` only reports findings; apply them yourself, or dispatch the `simplify` skill for the pure quality cleanups it flags.

## If a Failing Test Is Impractical

Do not silently skip the regression step. Before fixing, explicitly explain why a failing test is impossible or not worth the cost, then choose the closest executable regression check available. Examples include a targeted script, manual reproduction command, browser automation, snapshot comparison, log assertion, or focused integration check.

Prefer no new test over a bad test. A bad test is one that mostly tests mocks, encodes current implementation details, depends on timing or unrelated global state, needs expensive infrastructure for a small fix, or would be deleted immediately after proving the fix.

## Guardrails

- Do not change tests merely to match a wrong implementation.
- Do not weaken existing assertions unless the expected behavior has genuinely changed and the reason is clear.
- Do not add tests when the practical signal is weak; use manual or scripted verification and say why.
- If you are inspecting a bug that is flaky, make the test deterministic where possible and document the signal being locked down.
- Keep the regression test focused on the bug ; avoid broad fixture churn or unrelated coverage expansion.
- If the bug exposes a broader class of failures, first land the focused regression path, then consider additional sibling coverage.

## Running tests

Use `dotnet test` (optionally scoped with `--filter` to a namespace or `[Trait]`) as the narrowest relevant validation, then the full suite before calling a cycle done.
