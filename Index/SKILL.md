---
name: lessons-index
description: Routing index for the Lessons skill library. Load this when starting any software-engineering task; it lists the situations each downstream skill applies to so you can load the right one. Each entry has concrete trigger phrases — when you see those phrases or symptoms in the work, load the matching specific skill.
version: 0.1.0
---

# Lessons — routing index

This is the **router**, not a teacher. It tells you which sibling
skill to load when a recognisable situation appears. Each entry lists
concrete triggers — phrases, symptoms, or tasks. When you see one,
load the named skill.

The triggers are *symptoms* you might notice or *phrases* the user
might say, not categories of file or task. Read them as: "if any of
these is happening, load this skill."

## Routing table

### → Load `DontReinventTheWheel`

When the work involves **building a custom system that mature
infrastructure already provides**.

Triggers:

- You're about to author a parser, renderer, layout engine, query
  language, routing system, schema language, event bus, or
  dependency resolver.
- The user says "I need to parse my YAML / markdown / DSL and
  render it as a UI."
- You catch yourself writing the word *my-own* in a noun phrase
  ending in *engine*, *language*, *format*, *store*, or *protocol*.
- You're considering a custom storage layer for records-with-queries.
- You're considering hand-rolled auth / OAuth / session handling.
- A plan sketches a parallel implementation of something HTML+DOM,
  SQLite, POSIX, ICU, or the language's package manager already
  does.

**The skill teaches:** inventory mature substrates, probe for
isomorphism, isolate the residual (what the substrate genuinely
can't do), map onto the substrate for everything else.

### → Load `MultiLevelTesting`

When the work involves **deciding how to test something**, or
diagnosing a testing strategy that's failing.

Triggers:

- "How should I test this?"
- "Should I mock this dependency?"
- A bug shipped that the existing tests didn't catch.
- A refactor feels risky because the safety net is uneven.
- CI is slow (only E2E tests) or untrustworthy (only mocks).
- A new contributor asks "what's this function supposed to do?"
  and the answer should have been "read its tests."
- Someone proposes writing all tests at one level.
- "We'll add tests later."

**The skill teaches:** the test pyramid (many unit → fewer
integration → fewer E2E → a few smoke), the six benefits of tests
(verify, design feedback, executable spec, usage example,
regression guard, refactor net), and how to pick the right
altitude.

### → Load `SmallBatchCommitsMergedOften`

When the work involves **scoping a feature branch / PR**, or
diagnosing slow / risky merges.

Triggers:

- "Should I roll this into the same PR or open a new one?"
- A PR description starts to need "also" or "and."
- A diff has crossed 500 lines and is still climbing.
- A branch has been alive longer than a couple of days.
- Someone is blocked waiting on review before they can keep working.
- A merge conflict is harder to resolve than the original work was
  to write.
- The team has the "ice-cream cone" anti-shape: long-lived branches
  and end-of-sprint merge weeks.
- The user catches themself writing "WIP — clean up before
  merging" on a six-day branch.

**The skill teaches:** small + short-lived branches, branch from
tip-of-main, rebase often, stacked PRs for dependent work, the
five reasons it matters, and when a larger branch is genuinely
the right call.

### → Load `OrganisingGitPullRequests`

When the work involves **the commit shape of an existing branch
before it gets reviewed**, or when poor history bites somewhere
downstream (bisect, revert, code review).

Triggers:

- A branch has commits like `wip / wip2 / fix / actually fix / fixes
  the fixes`.
- A revert on `main` dragged in unrelated files and broke things.
- `git bisect` lands on a commit named `wip` — useless.
- A reviewer asks "why is file X in this PR?" and you have to dig.
- You tried an approach, abandoned it, and the diff still has
  vestiges.
- The team is debating force-push, squash-on-merge, or "should
  every commit pass tests."
- A PR is about to open with the current commit log and you know
  it's not the shape you want reviewed.

**The skill teaches:** the Sexton reset-and-recompose pattern,
when to use `--amend` vs interactive rebase vs new-branch-+-
cherry-pick vs `git absorb`, and how to think about the three
recurring debates (force-push, squash, valid-at-every-commit).

### → Load `LocalAWSenvironmentUsingLocalstack`

When the work involves **a service that depends on AWS** and the
question of how it should run in development or test.

Triggers:

- A service depends on S3, SQS, DynamoDB, Lambda, Kinesis,
  Secrets Manager, IAM, SNS, or similar.
- Onboarding doc says "ask someone for AWS credentials" or
  "you'll need to wait for the staging account."
- Integration tests get skipped on laptops because they need real
  AWS.
- Someone is about to write a mock for an AWS client.
- A new dev cannot run the application locally because of AWS
  dependencies.
- CI doesn't run AWS-touching tests because the build agents
  don't have credentials.

**The skill teaches:** a four-piece recipe — a `docker-compose`
service for LocalStack, an init script that creates resources,
two structurally-identical config profiles (local vs. prod
differing only in the `endpoint` field), and a client builder
with one `if` that branches on endpoint-present-or-absent.

## How to use this index

1. Read the trigger phrases for each entry; they cover most of
   the situations the library knows about.
2. When you see one of those triggers in the work — in the user's
   message, in the code, in your own plan — **load the named
   skill** and follow its guidance.
3. If a situation matches *multiple* skills, load all of them.
   They compose. (A risky refactor of an AWS-touching service in
   a long-lived branch might want all of `MultiLevelTesting`,
   `SmallBatchCommitsMergedOften`, and
   `LocalAWSenvironmentUsingLocalstack`.)
4. If nothing here matches the situation, fall back to general
   reasoning — the library is intentionally selective.

## What the library doesn't cover

The library is small and biased. It does not cover:

- Performance tuning, profiling, or systems-level optimization.
- Security review or threat modeling.
- Architecture-at-scale (microservices, eventing, CQRS, etc.).
- Specific language idioms or framework-specific best practices.
- Hiring, mentoring, or organisational engineering.

For those, fall back to general knowledge or pull in a different
skill library.

## Adding to the library

A new lesson is a directory named in PascalCase with a single
`SKILL.md` following the same format as the others. After writing
it:

1. Add a new `### → Load <SkillName>` section to this routing
   table with 5–10 concrete trigger phrases.
2. Add a one-liner to the `README.md` table at the repo root.
3. Commit.

A lesson is worth adding when:

- The principle is recurring (it has fired in at least two
  separate contexts).
- The trigger is recognisable in the moment (not just visible in
  hindsight).
- The cost of *not* invoking the lesson is real (not just a
  matter of taste).
