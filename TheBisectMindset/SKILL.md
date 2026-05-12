---
name: the-bisect-mindset
description: When a regression exists between a known-good and a known-broken state, you don't have to read the diff. Bisect finds the introducing commit in log₂(N) steps. The skill is twofold — knowing to reach for `git bisect` early, and keeping the repository in a state where bisect actually works (every commit compiles, tests are runnable, commits are single-intent).
version: 0.1.0
---

# The Bisect Mindset

A bug exists in production. It didn't exist last week. There
are 80 commits in between. You can read every diff and try to
reason about which one caused it (slow, often wrong). Or you
can let the tooling do the search.

The principle: **for any regression that has a clean before-and-
after, `git bisect` finds the introducing commit in log₂(N)
steps**. 80 commits collapse to ~7 build-and-test cycles.
Reach for bisect *first*, not after you've exhausted intuition.

The second half of the principle: **keep your repo bisectable.**
Every commit should compile and pass tests, and each commit
should be a single intent — otherwise the bisect lands on a
useless commit message or a half-broken middle state.

## When to invoke

- A regression appeared between a known-good version (last
  week / last release / a specific tag) and now.
- A test that was green is suddenly red, and the failure isn't
  obvious from recent commits.
- "I think it broke somewhere this sprint…"
- A user reports a bug; the previous release didn't have it.
- A performance regression appeared without anyone noticing.
- A flaky test is flaking *more* than it used to.
- You're tempted to read 50 commits manually to find a cause.

## The shape — how bisect actually works

The mechanics:

```bash
# 1. Tell git the search bounds.
git bisect start
git bisect bad                    # current HEAD is broken
git bisect good v1.4.2            # this older tag was OK

# 2. Git checks out the midpoint. Build, run the test that
#    exhibits the bug. Decide:
git bisect good   # or `git bisect bad`

# 3. Git picks the next midpoint. Repeat.
# After log₂(N) iterations, git names the introducing commit.

git bisect reset    # restore HEAD
```

Even better — **automate the check**. If you have a script
that exits 0 when the build is "good" and non-zero when it's
"bad" (a failing test, a perf regression check):

```bash
git bisect start
git bisect bad HEAD
git bisect good v1.4.2
git bisect run ./scripts/check-regression.sh
```

Bisect runs the script at each midpoint and ratchets the
search without you intervening. 80 commits, 7 minutes, the
introducing commit is named.

## Why it matters

1. **log₂(N) beats N.** 1000 commits collapse to ~10 build-
   and-test cycles. Linear inspection is 100× slower.

2. **It removes your bias.** You stop looking for the
   "obvious" cause and let the search find the actual one.
   Often the actual cause is *not* the suspect commit.

3. **Works on commits you didn't write.** Bisect doesn't
   need you to understand the diffs — only to recognise
   "this build is good" vs "this build is bad."

4. **Catches non-obvious regressions.** Performance,
   flakiness, memory growth, security issues — anything you
   can write a check for, bisect can find.

5. **Documents the cause.** The named commit is the
   forensic record. Future "why was X introduced" questions
   point at it.

## What makes a repo bisectable

For bisect to work, the build must be reproducible at every
historical commit. Habits that preserve this:

1. **Every commit compiles.** If a commit doesn't build,
   bisect at that commit returns "skip" and the search
   continues. A few skips are fine; pervasive skips break
   the search.

2. **Each commit is single-intent.** When bisect names a
   commit and that commit bundles three changes, you're
   back to manual diff-reading. See
   [`OneChangeAtATime`](../OneChangeAtATime/SKILL.md).

3. **Tests run at every commit.** If your test suite
   changed and the old tests don't run at the introducing
   commit, you need to bisect with the *new* tests
   cherry-picked or with a check-script that targets the
   specific regression.

4. **Long-running branches stay rebased on main.** If a
   branch is stale, bisecting on main misses bugs the
   branch introduced. Rebase + merge brings the bug into
   the bisectable history.

5. **Use signed/release tags** as known-good anchors. "Last
   release was tagged v1.4.2 and it didn't have this bug"
   gives you a starting `good` ref instantly.

## Practical guidance

- **Reach for bisect early.** It feels like overkill until
  you do it once and it finds the bug in 12 minutes. After
  that, it becomes the first move.
- **Write the check-script before bisecting.** "How will I
  decide good vs bad at each midpoint?" is the work.
  Sometimes it's `run the failing test`. Sometimes it's
  `start the app, curl an endpoint, grep the response`.
- **Use `git bisect run`.** Manual bisect is slow because
  context-switching at each step is slow. Automate the
  check; let the tool churn.
- **Mark non-buildable commits with `git bisect skip`.**
  Bisect tolerates a few skips. Don't manually inspect a
  commit you marked skip — they're skipped for a reason.
- **Bisect across branches.** Use a merge commit as the
  bad end if the regression came in through a merge.
- **For external regressions** (dep upgrade introduced a
  bug), bisect the dep's own commits. `git bisect` works on
  *any* git repo.
- **Combine with the rest:** see
  [`OneChangeAtATime`](../OneChangeAtATime/SKILL.md),
  [`OrganisingGitPullRequests`](../OrganisingGitPullRequests/SKILL.md),
  and [`SmallBatchCommitsMergedOften`](../SmallBatchCommitsMergedOften/SKILL.md)
  — they're all upstream conditions that make bisect work.

## Common failure modes

- **"I'll just read the diffs."** For more than 10 commits
  it's slower than bisect. For more than 50, much slower.
- **Bisect lands on a `wip` commit.** Useless — the commit
  bundles too much. Fix the cause: shape commits before
  merging.
- **The middle commits don't build.** Bisect grinds to a
  halt. Fix the cause: every commit on main should build.
- **No check-script.** Doing the check manually at every
  midpoint is tedious; people give up. Automate.
- **Bisecting on a branch that diverged from main.** The
  bug came through a merge; bisecting only one parent
  misses it. Use `git bisect` with the appropriate roots.
- **Forgetting `git bisect reset`.** Easy to leave the
  repo in a detached-HEAD state.

## When the principle DOES NOT apply

- **Regressions with no clean before-and-after.** Some
  bugs were always there but only manifested under new
  load. Bisect needs a "good" baseline.
- **Non-deterministic bugs.** Flakiness that's roughly
  50/50 makes bisect unreliable; you'd need a check-script
  that runs many trials per commit. Possible, but expensive.
- **Tiny commit counts.** If only 3 commits are suspects,
  read them.

## Tagline

> log₂(N), not N.

Bisect doesn't need you to be smart. It needs you to be
able to say "good" or "bad."

## Sources

Distilled from general engineering practice; the
`git bisect` man page is the canonical reference.
