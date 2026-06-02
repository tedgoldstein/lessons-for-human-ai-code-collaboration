---
name: multi-level-testing
description: Build confidence with a pyramid of tests at multiple levels — many unit tests, fewer integration tests, fewer end-to-end, a small set of smoke tests. Each level brings different benefits; combining them yields high-confidence software. Tests also act as documentation, design feedback, regression guards, and the safety net that makes refactoring possible.
version: 0.1.0
---

# Test at Multiple Levels

Tests are A Good Thing — that's not the controversial part. The
useful insight is that **different levels of tests bring different
benefits, and you should combine them.** A codebase with only unit
tests is brittle; a codebase with only end-to-end tests is slow and
flaky. The well-tested codebase has many small tests, fewer mid-scale
tests, fewer still whole-system tests, and a small set of
post-deployment smoke checks.

This skill is the **pyramid + the multiple benefits**.

## When to invoke

- "How should I test this?"
- "Should I mock this dependency or use a real one?"
- "We have integration tests — why do we need unit tests too?" (or
  the inverse).
- A bug shipped that the existing tests didn't catch.
- A refactor feels risky because the safety net is missing or
  uneven.
- You're being asked to test something *after* writing all the
  code, and noticing that the design isn't testable.
- A new team member asks "how do I know what this function is
  supposed to do?" and you find yourself wishing the answer were
  "read its tests."

## The pyramid

| Level | Volume | What it tests | Speed | Setup cost |
|---|---|---|---|---|
| **Unit** | many | One small unit of code in isolation | very fast | trivial |
| **Integration** | fewer | Multiple units together; often a real database / queue / cache | seconds | nontrivial |
| **End-to-end (E2E)** | fewer still | The whole stack from the outside — headless browser, real HTTP, real auth | slow | high |
| **Smoke** | a handful | Is the service up + responsive? Runs post-deploy or on a schedule | fast (one-shot) | minimal |

The exact shape varies — where the line sits between unit and
integration is a productive argument and not worth settling in the
abstract. The shape that matters: **a wide base of fast, focused
tests, narrowing to a few slow ones at the top.**

Anti-shape: the "ice-cream cone" — many slow E2E tests, few unit
tests. Symptoms: a 90-minute CI; tests that go red on weather; a
team afraid to refactor.

## Why multiple levels — the benefits

Tests do more than verify correctness. The other benefits are why
you write them throughout development, not in a batch at the end.

1. **They verify functionality.** The obvious one. The code does
   what you think it does — *now*.

2. **They drive cleaner code.** A unit that's hard to test is
   telling you something: it's coupled too tightly, it has
   undocumented side effects, it does too many things, its
   interface is ambiguous. The pain of testing is a signal worth
   listening to before the code ships.

3. **They are executable specifications.** A well-named test —
   `it('throws an error when null is passed in', …)` or
   `throwsAnErrorWhenNullIsPassedIn()` — documents intent more
   precisely than prose. When a future reader wonders "what
   should this method do when X?", a test answers in code.

4. **They are examples of use.** A new user of a module reads
   the tests to learn how to call it. This is one of the most
   underrated forms of documentation; tests stay correct because
   they break when they don't.

5. **They guard against regression.** As features arrive and
   people come and go, tests stay. A change that quietly breaks
   an old contract fires an immediate, named alarm — and a
   descriptive test name tells you whether your change is wrong
   or the test is stale.

6. **They make refactoring possible.** This is the big one. With
   good tests at the right levels, you can refactor with
   confidence: if the tests pass, the behavior is preserved.
   Without them, every refactor is a gamble, and the codebase
   ossifies.

## Practical guidance

- **Interleave tests with code.** Whether or not you practice TDD,
  write tests as the code arrives. Saving them for the end means
  you'll cut them under time pressure.
- **Test names should read as specifications.** In JS/TS use
  Jest's `it('…')` strings; in languages that demand identifiers,
  use long descriptive function names —
  `throwsAnErrorWhenNullIsPassedIn`. The reader of the test
  should understand the expectation without reading the body.
- **Worth-testing-the-trivial sometimes.** Some trivial code is
  worth a test purely to enshrine intended behavior against
  future accidental change — particularly around invariants,
  constants, or API surface.
- **Mock judiciously, not reflexively.** Mocks make tests fast
  and focused; they also make tests lie. Wherever a real
  dependency is fast and deterministic, prefer the real thing
  (especially in integration and E2E tests).
- **Test from the right altitude.** Unit tests verify
  implementation contracts; E2E tests verify user-facing
  behavior. A bug in your auth flow probably wants an E2E test;
  a bug in your date-parser probably wants a unit test. Pick
  the level where the failing case is most cheaply expressed
  *and* most stably described.

## Common failure modes

- **Test-everything-at-one-level.** All unit tests, no
  integration → mocks lie and your integrations rot. All E2E
  → CI runs in an hour and breaks on a holiday. Pyramid, not
  pillar.
- **Brittle tests.** Tests so coupled to implementation that
  *every refactor* breaks them. Sign the test was written at
  the wrong altitude or knows too much about internals.
- **Tests as decoration.** Tests that exist to make CI green
  but assert nothing meaningful (`expect(thing).toBeDefined()`
  after constructing it). The test count is high; the
  confidence is zero.
- **"We'll add tests later."** Later doesn't arrive. The cost
  of retrofitting tests onto code that wasn't designed for it
  is what makes "later" infeasible. Write them now.
- **Skipping the trivial.** A one-line constant export with no
  test is fine until the day someone changes it and three
  downstream systems silently break. For load-bearing
  constants and invariants, a tiny test prevents the silent
  drift.

## When the pyramid shape needs adjusting

- **Pure library / SDK code** — almost all your tests are unit;
  there's no stack to E2E. That's fine.
- **Mostly-integration systems** (ETL pipelines, data
  transforms) — your "real work" is at the integration level;
  many unit tests don't add much. Push the pyramid's middle
  layer wide.
- **Hard-real-time / safety-critical** — formal verification,
  property-based testing, and exhaustive E2E scenarios may
  dwarf unit tests in importance. The shape inverts.

## Tagline

> Tests verify, document, design, and protect — all four. Lose
> any one and the codebase ossifies.

A test suite isn't there to prove the code works today. It's
there so the code can change tomorrow.

## See also

- [OneChangeAtATime](../OneChangeAtATime/SKILL.md) —
  testability collapses when commits bundle changes;
  a green pyramid against a multi-intent commit
  doesn't tell you which intent the test covered.
- [TheBisectMindset](../TheBisectMindset/SKILL.md) —
  bisect only lands somewhere useful when every level
  of test is runnable at every commit; the pyramid
  pays its diagnostic dividend through bisect.
- [LocalAWSenvironmentUsingLocalstack](../LocalAWSenvironmentUsingLocalstack/SKILL.md)
  — the practical home for the AWS-integration tier
  of the pyramid: real SDK calls, local endpoint.
- [FormalVsImprovisational](../FormalVsImprovisational/SKILL.md)
  — how much testing to invest in is itself a
  stakes-calibration decision rather than a fixed
  ceremony.

## Sources

The pyramid + the "tests as spec / examples / regression guard /
refactor net" framing comes from the 67 Bricks engineering blog
(67bricks.com). This SKILL.md is a restatement in our own voice;
the substance and the headline benefits are theirs.
