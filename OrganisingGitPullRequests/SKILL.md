---
name: organising-git-pull-requests
description: Reshape the commits on a feature branch into a coherent story before review, so each commit is a deliberate step the reviewer (and the future bisector) can read. The "WIP, wip2, fix the thing" tip-of-branch reality is fine while you're working; cleaning it up before review is the discipline. Multiple techniques (`amend`, interactive rebase, the reset-and-recompose pattern); the right one depends on the change.
version: 0.1.0
---

# Organise Your Git Pull Requests

Git history is something the team has to live with for years. A
clean, deliberate sequence of commits in a PR pays compound interest:
**reviewers can follow how the change came together; bisect lands on
the right commit; a revert of one commit doesn't drag unrelated
files with it.** A "30 commits of WIP" history pays compound debt:
reviewers can't review the commits, only the squashed diff; revert
is dangerous; bisect is useless.

This skill is the **shape-the-history-before-you-ship** discipline.

The how is a productive argument — `git commit --amend`, `git rebase
-i`, the reset-and-recompose pattern from Annie Sexton's *Git
Organized*, "new branch and cherry-pick across" — different
engineers prefer different techniques. The skill is recognising that
*some* shaping should happen, and picking the technique that fits
the change.

## When to invoke

- You're about to open a PR and the commit log looks like
  `wip / wip2 / fixes / more fixes / actually working`.
- A commit on main introduced a bug; the revert touches files
  unrelated to the bug because the original commit bundled
  unrelated work.
- Bisect runs and lands on a commit message of `wip` — useless.
- A reviewer asks "why did you change *file X* in this PR?"
  and you have to dig to remember whether it was relevant.
- You tried something, abandoned it, and the diff still has
  vestiges of the abandoned attempt.

## The Sexton pattern (one good technique)

The pattern from Annie Sexton's *Git Organized* / Changelog
*Git your reset on*:

1. Work on the branch normally. Commit `WIP` whenever — the
   tip-of-branch is just a save-point during development. The
   *content* you produce is what matters, not the
   commit-graph yet.
2. When the feature is working, `git reset origin/HEAD` (or
   `git reset main` from the branch). All your changes are now
   uncommitted but staged-or-unstaged in the index.
3. Now build a *deliberate* sequence of commits. `git add -p`
   to stage individual hunks. Group them by intent:
   - "Add the helper function X (with tests)"
   - "Refactor caller to use X"
   - "Implement feature Y on top of X"
4. Each commit has a meaningful message. Each commit, ideally,
   compiles and passes tests.

**What it buys:**
- Compartmentalised history — each commit is one intent.
- Reviewers read a story of how the change arrived.
- Revert / bisect operate on the *intent*, not the keystroke
  trace.
- The PR is no harder to make than the development was —
  the shaping is at the *end*, not throughout.

**What it costs:**
- Bad approaches you tried and abandoned aren't in the
  history. (Some teams value the dead-ends as documentation;
  most don't.)
- Some changes don't compartmentalise into per-file groups —
  the same file may contain hunks that belong to *different*
  intents. Tool support for "commit only this hunk inside
  this file" is mediocre (`git add -p` is the standard but
  it's terse).

## Other shaping techniques (pick by fit)

| Technique | Best for |
|---|---|
| `git commit --amend` | Last commit needed a small fix or a better message. Cheapest tool; lowest risk. |
| `git rebase -i HEAD~N` | Reorder, squash, fixup, or reword the last N commits. Powerful; mildly risky if you've shared the branch. Splitting a commit is awkward (`edit` action + reset, restage). |
| The **reset-and-recompose** above | The branch's whole shape is wrong and you want to redraw it. Most flexible. |
| New branch + cherry-pick | The branch has unsalvageable history; build a clean branch alongside, cherry-pick the meaningful commits, abandon the original. |
| `git absorb` (extension) | Auto-distributes uncommitted hunks into the right earlier commit. Good for "fix a typo in commit-from-3-days-ago" without manually rebasing. |

A team that uses *one* technique uniformly is fine. A team that uses
*the right technique for the situation* is sharper. None of these
is the One True Way.

## The recurring debates

There's no neutral answer to any of these — but here's the shape
of the disagreement so you can have it productively with your team.

### Force-push: friend or foe?

- **"Force is dangerous"** camp: `git push --force` is a red flag.
  Use `--force-with-lease` at minimum. Never on `main`. Even on a
  shared branch, force-push silently loses other people's work.
- **"Force on your own branch is fine"** camp: a feature branch
  *before* its PR is yours alone. Reshape freely. After the PR
  opens, force-push is honest about what you changed — reviewers
  can re-read the diff cleanly.

Reasonable rule: **`--force-with-lease` on your own pre-review
branches, never on main, never on a branch someone else has
committed to without coordinating.**

### Squash-on-merge?

- **Pro:** `--squash` on merge produces a clean main where each
  commit ≈ one ticket. Easy to scan; easy to revert at the
  ticket level.
- **Con:** loses the deliberate sequence the developer just
  spent effort building. The "story" collapses to a single
  paragraph. Bisect granularity drops to the ticket level.

Tradeoff: squash-merge is a *substitute* for organising commits
*before* the merge. If you've shaped a coherent commit sequence,
squash throws it away. If you haven't, squash hides the mess —
but at the cost of bisect resolution.

The non-default sweet spot for many teams: **don't squash;
require organised commits in the PR.** That keeps the history
informative *and* shippable.

### "Software valid at every commit"?

- **Yes** camp: every commit should compile and pass tests.
  Bisect works. Revert is safe. Main is shippable at any
  point in history.
- **No / not always** camp: some workflows are anti-this —
  classical TDD commits a *failing test*, then a *passing
  implementation*, separately. The intermediate state is
  by-design red.

Compromise: **green-at-every-commit ON the integration branch
(main)**; *during* development the rules are looser. The
shaping pass before review is where you reconcile — either
collapse the red commit into the green one, or commit tests +
code together so each commit is internally consistent.

## Practical guidance

- **Shape before review.** The tip-of-branch reality is fine
  while you're working. The PR is the point where shape
  starts to matter.
- **One coherent change per commit.** "Refactor + new
  behavior" is two commits. The refactor reviewer can verify
  is mechanical; the behavior diff stands on its own.
- **Test changes accompany the code they test.** Don't put
  the test in commit N and the implementation in commit
  N+1; the intermediate state breaks bisect.
- **Use `--force-with-lease`, not `--force`.** It refuses to
  overwrite commits you didn't fetch — protects against
  silently obliterating a teammate's push.
- **Stop fearing rewrites on unshared branches.** Rewriting
  history *before* anyone else has based work on it is
  free. Rewriting *after* is coordination.
- **If you can't shape it cleanly, ship a draft PR and
  ask for help.** Better than five revisions of "rebased
  again, please re-review."

## Common failure modes

- **The unstructured force-push spree.** Force-pushing after
  every minor review comment with rebased history loses
  reviewer trust ("I can't find what changed since my last
  pass"). Use range-diffs or merge-commits between review
  rounds; only do the clean rebase once at the end.
- **The squash-as-laundry workflow.** Engineer commits
  garbage all branch; squash-merge to main hides the
  garbage. Looks tidy until someone needs to bisect — then
  the squashed commit is a single 2,000-line lump with no
  internal structure.
- **The pristine-history obsession.** Spending three hours
  reshaping a branch into a perfect sequence of seven
  commits for a 50-line change. The point is *legible*,
  not *artisanal*.
- **Mixing the refactor and the behavior change in one
  commit.** Then someone bisects and the failing commit
  contains both — they can't tell which part broke.

## When the principle DOES NOT apply

- **Trivial single-commit PRs.** A typo fix doesn't need
  shaping. Move on.
- **Spike branches you'll throw away.** Don't waste time
  organising a branch you're abandoning. Learn from it;
  rebuild the real version in clean commits.
- **Branches with significant external work on them.**
  If someone else has commits on the branch, rewriting
  history is a coordination problem; either coordinate or
  use merge commits instead of rebase.

## Tagline

> The tip-of-branch is where you save. The PR is where you
> tell the story. They're not the same artifact.

History is something the next person reads. Shape it for
them.

## Sources

The premise (clean history matters because of revert / bisect
behavior), Annie Sexton's *Git Organized* reset-and-recompose
pattern, the force-push / squash-on-merge / valid-at-every-commit
debates, and the practitioner survey at 67 Bricks come from the
67 Bricks engineering blog (67bricks.com), citing the *Changelog*
podcast episode "Git your reset on" and Annie Sexton's "Git
Organized" post. This SKILL.md is a restatement in our own voice;
the techniques and the framing of the debates are theirs.
