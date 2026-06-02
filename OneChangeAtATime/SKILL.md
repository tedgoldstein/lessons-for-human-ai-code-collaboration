---
name: one-change-at-a-time
description: When debugging or fixing a regression, isolate variables. Don't combine "I upgraded the dep AND refactored AND added a feature" in one commit / PR / deploy. The cost of bundling is exponential in the number of bundled changes — every interaction has to be considered. The cure is discipline at scope boundaries (commit, PR, deploy).
version: 0.1.0
---

# One Change at a Time

The hardest bugs are the ones where two changes interacted. The hour
you "save" by bundling a dep bump + a refactor + a feature into one
PR comes back as a day when production breaks and you have to figure
out which of the three did it.

The principle: **the unit of change should be the unit of
investigation.** If a commit, a PR, or a deploy includes only one
intentional change, then when something breaks, you have exactly
one suspect.

## When to invoke

- You're about to commit a bug fix and notice you also did "a small
  cleanup along the way."
- A regression appeared in the last week and bisect lands on a
  commit containing four unrelated changes.
- You're upgrading a major dependency and tempted to do
  the "while we're in there" refactor.
- Someone says *"it works on my machine"* — different environment,
  different version, different config: too many variables changed
  between you and them.
- A failing test starts passing after a change you don't understand;
  you don't isolate which part of the change fixed it.
- A deployment ships multiple decoupled features at once.

## The shape

A change has three observable scopes:

| Scope | "One change" means |
|---|---|
| **Commit** | One *intent* — refactor, feature, bug fix, dep bump. Not two. |
| **PR / branch** | One coherent story (multiple commits OK). Doesn't mix orthogonal work. |
| **Deploy** | One unit of new behaviour or one fix. Not "ten things from the sprint." |

The unit doesn't have to be small in lines (a sweeping codemod can
be one change). It has to be **one decision** that, if reverted,
removes one cohesive effect.

## Why it matters

1. **Bisect works.** When a regression is somewhere in the last 30
   commits, `git bisect` finds it in log₂(30) ≈ 5 steps — but only
   if each commit is one intentional thing. Otherwise you can find
   the *commit* and still not know which line in it broke things.

2. **Revert is safe.** Reverting a single-intent commit removes
   exactly the broken behaviour. Reverting a bundled commit removes
   the bug *and* the refactor you wanted to keep.

3. **Review is faster.** Reviewers can hold one change in head and
   verify it. Reviewers cannot reliably review four unrelated
   changes in one diff.

4. **Confidence accumulates.** Each separate change either ships
   cleanly (and you trust the next one) or breaks immediately (and
   you know which one). Bundled changes give you only "the whole
   batch is uncertain."

## Practical guidance

- **Refactor first, change behaviour second.** Two commits. The
  refactor reviewer verifies it's *mechanical*; the behaviour
  reviewer sees only the behavioural diff.
- **Bump deps in their own PR.** A dep bump is its own change.
  When it breaks, you know it's the dep — not your feature.
- **Don't do "drive-by" cleanups in a feature branch.** Save them
  for a separate cleanup PR. The drive-by adds review surface and
  obscures bisect.
- **`git stash` is your friend mid-change.** You're working on
  feature A, you see a typo, you're tempted to fix it inline:
  stash A, fix and commit the typo, pop. Two clean commits.
- **For debugging: change one thing, observe.** Don't tweak three
  config values at once and re-run. Even if it works, you don't
  know which mattered.
- **When forced to bundle (codemod-style sweep), name it
  explicitly.** "MECHANICAL: rename Foo → Bar across N files."
  The reader sees instantly that it's mechanical.

## Common failure modes

- **"While I was in there..."** Almost every drive-by cleanup
  bundled with a real change harms the real change. Discipline:
  resist; queue the cleanup for later.
- **The Friday-afternoon bundle deploy.** Several decoupled
  features ship in one release because it's the end of the week.
  Monday a bug appears; nobody can localise it. Deploy one thing
  at a time, even on Fridays.
- **The "tiny refactor" hidden inside a bug fix.** The refactor
  *is* the change. The bug fix is the change. Two commits.
- **Re-running tests after multiple config changes.** The test
  passing tells you nothing about which config mattered. Toggle
  one at a time.

## When the principle DOES NOT apply

- **Atomic-by-nature changes.** A protocol version bump that
  requires coordinated client + server changes. Split would break
  the build. Land as one — but flag the *atomicity* explicitly so
  reviewers know why it's bundled.
- **Mass mechanical codemods.** A find-and-replace across the
  tree. Bundle freely, but make the diff inspectable as one
  pattern (and never sneak non-mechanical changes inside it).
- **Time-critical hotfixes.** When prod is down, fix it. Bundle
  if you must. Pay the cleanup cost later.

## Tagline

> When debugging or shipping, change one thing.

The bisect / revert / review machinery only works as well as your
commits do. Bundled commits silently break all three.

## See also

- [TheBisectMindset](../TheBisectMindset/SKILL.md) —
  bisect only lands on a useful commit when each commit
  represents a single intent; this skill is the
  precondition that makes that machinery pay off.
- [CommitHygiene](../CommitHygiene/SKILL.md) — the
  per-commit craft (staging, messages, author identity)
  inside which this scope rule operates.
- [SmallBatchCommitsMergedOften](../SmallBatchCommitsMergedOften/SKILL.md)
  — the branch-level analogue: small scope per PR is
  the same discipline applied one tier up.
- [OrganisingGitPullRequests](../OrganisingGitPullRequests/SKILL.md)
  — reshaping mid-branch history toward this rule
  before review, when the commits arrived bundled.

## Sources

Distilled from general engineering practice; pairs with
[OrganisingGitPullRequests](../OrganisingGitPullRequests/SKILL.md)
and [TheBisectMindset](../TheBisectMindset/SKILL.md).
