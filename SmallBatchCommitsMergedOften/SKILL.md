---
name: small-batch-commits-merged-often
description: Favour small, short-lived feature branches that land on main quickly. Smaller changes are easier to review, less likely to merge-conflict, and more likely to surface design problems early. A PR doesn't have to be a complete user-facing feature — incremental, well-tested slices are fine, as long as everything builds and tests pass at every commit.
version: 0.1.0
---

# Small Batches, Merged Often

The most teachable git habit isn't *which* branching model you use
— git flow, trunk-based, GitHub flow, your own — that's a team
preference, and reasonable teams pick differently. The teachable
habit is **how you break work into pull requests**: keep them small,
keep them short-lived, and merge them back to main soon.

A long-lived branch is a debt that compounds. Every day it lives,
main moves under it, context fades, conflicts grow, the reviewer's
attention deficit grows, and the chance that something subtle hides
inside the diff grows with the diff. Small batches keep all four
costs low simultaneously.

## When to invoke

- "Should I roll this into the same PR, or open a new one?"
- A PR description is starting to read like a list of unrelated
  things.
- The diff for review has crossed 500 lines and you're still
  not at the point of the change.
- A branch has been alive for more than a few days and main has
  moved.
- Someone is "blocked waiting on review" before they can keep
  working.
- A merge conflict is harder to resolve than the original change
  was to write.
- You catch yourself writing "WIP — will clean up before merging."

## The shape

| Property | Aim for | Why |
|---|---|---|
| **Size of diff** | small (review-able in 15 min) | Reviewers can hold it in head; bugs have less surface |
| **Branch lifetime** | hours to a couple of days | Main hasn't moved much; context is fresh |
| **Scope of one PR** | one coherent change | Easy to describe, easy to revert |
| **Branch base** | tip of `main` (refreshed often) | Conflicts surface immediately, not at merge time |
| **State at every commit** | builds, all tests green | Bisect works; revert is safe |

A PR doesn't have to ship a user-facing feature. Incremental slices
are fine — a refactor, a new utility module, a test harness, a
dependency bump — as long as the slice is **complete for what it
is** and the build + tests pass. The "feature" can land across many
small PRs.

## Why it matters

1. **Reviews are easier and better.** A 100-line PR gets a
   thorough review; a 2,000-line PR gets "LGTM." Smaller diffs
   let the reviewer think about intent and design instead of
   piecing the diff together.

2. **Merge conflicts shrink dramatically.** The longer a branch
   lives, the more main moves, the more lines they fight over.
   Daily merges from main into a live feature branch keep the
   conflict surface tiny; even when conflicts happen, the
   relevant code is still fresh in mind.

3. **Bugs surface earlier.** A small PR that breaks production
   is easy to revert and easy to bisect. A 30-commit feature
   branch that breaks something is a forensic exercise.

4. **Throughput goes up.** Counterintuitive but true: parallel
   small PRs flow faster than serial large ones. Less waiting
   for review, less context-switching back to a stale branch,
   less merge-day pain.

5. **The codebase is always shippable.** If every PR leaves
   main green and complete-for-what-it-is, you can deploy at
   any commit. Long-lived branches mean main is shippable
   *minus* whatever's in flight — a much weaker invariant.

## Practical guidance

- **Branch from the tip of main, every time** unless there's
  a real reason not to (and "I forgot to pull" isn't one).
- **Rebase or merge from main often** — daily on any branch
  that lives past a day. Conflicts you resolve now are tiny;
  conflicts you defer compound.
- **One coherent change per PR.** If the PR description
  starts to need "also" or "and", split. A refactor + a
  behavior change should be two PRs (refactor first; reviewer
  can verify the refactor is mechanical before the behavior
  diff lands).
- **Stacked PRs when work depends on unmerged work.** Open
  PR-B against PR-A's branch. Review proceeds in parallel.
  When PR-A lands, rebase PR-B onto main and ship.
- **Compile + test at every commit, not just at the tip.**
  Bisect needs this. Revert needs this. CI on every push
  enforces it.
- **Keep WIP out of the branch.** A WIP commit at the tip
  is fine; a *series* of "wip", "wip2", "actually fix
  things" commits is a sign the PR should be smaller or
  squashed before merge.

## Common failure modes

- **The mega-PR.** A two-month feature branch with 80 files
  changed. Review is impossible; conflicts are guaranteed;
  bugs are inevitable. Split before review, not at review
  time.
- **The "blocked" cascade.** Engineer A's PR sits unreviewed;
  engineer A starts building on the unmerged branch; now
  there are three branches stacked in fragile sequence. Use
  stacked PRs explicitly, with the stack visible.
- **Squash-everything-at-merge as a substitute.** A 4-week
  branch squashed to one commit at merge looks tidy but
  loses bisect granularity and doesn't address the review
  problem at all. The point is small PRs, not small
  history.
- **WIP commits as branch decor.** "wip", "fixes",
  "more fixes" — a sign the branch wasn't ready to open and
  the commits weren't meaningful units. Rebase before
  review.
- **Reviewer overwhelm by the opposite extreme.** 30 PRs in
  one morning is its own kind of bad: every PR open is a
  context switch for someone. Aim for *small enough*, not
  *as small as possible*.

## When a larger branch is genuinely the right call

- **Atomic-by-nature changes.** A protocol version bump that
  touches every endpoint; a schema migration that has to ship
  with the matching code. Splitting would break main between
  PRs. Land it in one piece — but keep it *rebased on main*
  and reviewed by someone who can hold the whole change in
  head.
- **Spike / exploration branches.** Sometimes you need to
  build something throwaway to learn. Don't pretend it's a
  PR — keep it as a spike branch, throw it away, then build
  the real thing in small batches.
- **Vendoring / mass codemods.** A find-and-replace across
  the whole tree is one logical change even if it's 5,000
  lines. Land it as one PR, but flag it as such and don't
  bury non-mechanical changes inside it.

## Tagline

> Land small, often. Main is the artifact; branches are
> scratchpads.

The PR isn't the unit of work. The merged change is the unit
of work. PRs only matter inasmuch as they make those merged
changes good.
