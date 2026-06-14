---
name: push-is-publication
description: Local commits are cheap and reversible. `git push` is publication — once on origin, CI runs, deploys trigger, downstream consumers may act on it, other sessions see it as truth. In AI-augmented workflows especially, where an AI may generate many commits autonomously, treat push as a deliberate review-then-publish checkpoint, not a reflex that follows every commit. The discipline complements (doesn't contradict) "small batches merged often" — small batches are about *what's in a PR*; this skill is about *when a branch becomes visible*.
version: 0.1.0
---

# Push Is Publication

`git commit` writes to your local machine. `git push` sends to a
remote. The two operations are commonly conflated — "save my
work" — but they have very different semantics:

- **Commit** is local, reversible, free, private. You can amend
  it, rebase it, drop it, branch from it, ignore it. Your
  collaborators don't know it exists.
- **Push** is publication. Once on origin: CI runs, hooks fire,
  deploy pipelines may trigger, the branch is visible to every
  collaborator, mirroring tools sync it, downstream consumers
  may act on it. Some operations become irreversible (force-push
  is sometimes blocked; CI logs can't be unwritten).

In a solo developer's slow-paced workflow, the two roughly
collapse — you commit small, push frequently, no harm done.
**In an AI-augmented workflow**, where an AI generates many
commits autonomously in a short window, the conflation becomes
expensive:

- The AI commits 8 times in twenty minutes.
- Each push triggers CI.
- Each push notifies every other Claude Code session of the
  branch state.
- A push you didn't review may contain a bug, an unintended
  scope expansion, a phrase you didn't want in a public commit
  message.
- A subsequent local rebase (cheap if unpushed) becomes a
  force-push (visible, sometimes blocked, harder to undo).

The discipline: **commit reflexively, push deliberately.** A push
is a "I've reviewed this batch and it's ready to be seen"
moment. It's a checkpoint, not a reflex.

This is the *timing-of-publication* complement to
[SmallBatchCommitsMergedOften](../SmallBatchCommitsMergedOften/SKILL.md).
That skill is about PR size and merge cadence at the
*organizational* level (when teams work together). This skill
is about *when an individual session* — human or AI — makes
its work visible. Both compatible.

## When to invoke

- An AI is generating commits autonomously (Claude Code, Cursor,
  Aider, etc.).
- You set up — or are tempted to set up — a `post-commit` hook
  that auto-pushes.
- Multiple AI sessions might trigger CI in racing ways.
- An AI just pushed a change you wanted to review locally first.
- You're working solo and notice your push log mirrors your
  commit log exactly.
- A force-push is being considered to fix something that was
  recently pushed reflexively.
- CI is firing on commits that aren't yet meaningful units of
  work.
- You're operating in a multi-Claude-session repo where another
  session might react to your push.
- You're about to push without having just run the project's
  test suite. Local commits are reversible; pushed regressions
  are visible. The right order is **test → push**, not
  **push → test → fix-forward**. If the test suite is too slow
  for every push, push to a `claude/*` branch first and only
  promote to `main` after the slow suite is green.

## The shape

| Property | Aim for | Why |
|---|---|---|
| **Commit cadence** | as often as makes sense; small batches | Cheap reversibility, granular history |
| **Push cadence** | when a batch is reviewed + ready to be visible | Push is publication, not auto-save |
| **Review moment** | between last commit and push | `git log` + `git diff origin/main..` + maybe `git show <last-commit>` |
| **Force-push policy** | only on `claude/*`-prefix branches you own; never on `main` | Force-push on shared branches surprises others |
| **CI triggers** | use push, not commit | Push is the meaningful event; commit is local |
| **Hooks at push, not commit** | run tests, lint, signing checks at push-time | Don't slow down local iteration |

## Why it matters

1. **AIs generate commits faster than humans review them.**
   A Claude Code session can land 6 commits in a coherent
   working block. Pushing each commit individually means 6
   CI runs, 6 notification fires, 6 chances for downstream
   tools to act on partial state. One thoughtful push at the
   end of the block is one CI run with the actual unit of
   work.

2. **Local rebase is cheap; remote rewrite is expensive.**
   Unpushed commits can be reordered, amended, squashed, or
   dropped with no coordination cost. Pushed commits require
   force-push (sometimes blocked, sometimes contentious) to
   modify. The "I'll clean it up before pushing" workflow
   only works if push hasn't happened yet.

3. **Review is the value-add of human + AI collaboration.**
   The AI does the work; the human does the review. If push
   is automatic, the review is bypassed. Decoupling push
   from commit *reserves the review moment* — you can't
   skip it accidentally.

4. **Pushed = visible to other sessions.** In multi-session
   repos, push is how sessions learn about each other's
   work. A push that's not yet ready to be seen is a
   collaboration hazard — the other session may build on
   work you intend to revise.

5. **The push is the signal.** When you push, you're
   saying "this is ready." The next person (CI, a
   reviewer, another session, a deploy hook) responds to
   that signal. If you push reflexively, the signal is
   noise; if you push deliberately, the signal carries
   meaning.

## Practical guidance

- **No `post-commit` auto-push hooks.** Tempting; counter-
  productive. Auto-push removes the moment you wanted to
  preserve. (Note: a `pre-push` hook to run tests is
  fine — it gates the push but doesn't trigger it.)
- **Review between last commit and push.** `git log
  origin/<branch>..HEAD --oneline` — quick scan of what's
  about to ship to origin. If anything looks wrong, fix
  before pushing.
- **Squash or rebase if needed, before pushing.** Trivial
  fixup commits ("typo," "actually fix things") are
  fine locally; squash them before push. Pushed commits
  with `fixup!` messages in their history are a smell.
- **Batch related commits in one push.** If you make
  three commits that together form a coherent
  acceptance-criterion-complete state, push them
  together. Each individual commit may not stand alone;
  the batch does.
- **Use `--force-with-lease`, not `--force`.** When force-
  pushing a personal branch, `--force-with-lease` refuses
  to push if someone else has updated the branch since
  you last fetched. Catches the case where another
  session has been pushing too.
- **Document a personal push policy.** "I batch-push at
  end-of-task / end-of-day / when the acceptance criteria
  for a sub-piece are all green." Whatever the policy,
  having one means push is a decision, not a default.

## Common failure modes

- **The auto-push commit hook.** "Commits and pushes in
  one step" — sold as convenience. Actual effect: every
  WIP commit, every fix-typo commit, every half-thought-
  through commit, all visible immediately. CI burn, noise
  for collaborators, no review moment.
- **Force-push to fix a reflexive push.** You pushed
  something you shouldn't have; now you force-push to
  rewrite it. Sometimes fine on personal branches;
  catastrophic if anyone else has already started from
  the pushed state. The right fix is "don't push until
  reviewed."
- **AI sessions racing to push.** Two AI sessions
  working in parallel; both `push` reflexively at their
  end-of-task. CI fires twice; one push fails (non-fast-
  forward); the AI hits an error condition it doesn't
  understand and starts trying to "fix" the conflict.
  Better: one push per logical unit, coordinated
  through some shared signal (a task tracker, a Track,
  a `pending-push` file).
- **CI as a guardrail substitute.** "If I push and CI
  fails, I'll fix it." This makes CI the review surface
  and treats pushed-and-broken as the normal state.
  Better: review locally, then push, then CI confirms.
- **Push-then-test-then-fix-forward.** Pushing before
  running the full local test suite, then discovering a
  regression and shipping a fix-forward commit. The
  intermediate broken state is now permanently in
  `origin/main` history; anyone who pulled in that
  window saw the regression. The right order is
  **test → push**; if the suite is too slow, push to a
  `claude/*` branch first and promote to `main` only after
  the slow suite is green. (Observed 2026-05-23 with the
  `track land` rollout: `MasterChildParserTests` was
  failing pre-existing, the push went out anyway, and a
  follow-up Track had to fix-forward.)
- **Push as a save mechanism.** "I push so I don't lose
  my work." git is local; commits are persistent on
  your laptop. Push is for *publication*. If laptop
  durability is the concern, use a personal mirror /
  backup, not the project's origin.
- **The "publish unfinished work for review" pattern is
  fine — when intentional.** Draft PRs, feature
  branches marked WIP, work-in-progress visible to
  collaborators. The discipline isn't "never push
  unfinished" — it's "push deliberately, knowing what
  the push signals."

## When push really should be every commit

- **Pair programming with shared origin.** Both
  participants push to share state in real time. The
  "review moment" happens face-to-face, not before push.
- **Pre-merge CI on every commit.** Some teams require
  CI to be green on each commit for bisect to work. In
  that case, push-per-commit is mandatory. (But the
  team-level convention should be explicit — not a
  default.)
- **Solo work on a throwaway branch.** A spike branch
  you'll discard regardless; nothing visible benefits
  from review-before-push.
- **Backup before a risky local rebase.** Push the
  pre-rebase state to a backup branch as insurance.
  (This is a workflow exception, not a habit.)

## Worked example

In a session working across three repos (Track/,
medbook.org, labbook.ai), an AI generated 12 commits
over several hours:

- 8 on Track/ (cleanup commits, new Track filings,
  acceptance-criteria adds)
- 3 on medbook.org (skill refresh, Execution → Running
  rename, new Tracks + routine artifact)
- 1 on labbook.ai (bootstrap commit)

None were pushed. The user's memory note was explicit:
*"No pushes without explicit permission — never `git push`
until the user says so; local commits are fine."*

The result: 12 commits accumulated locally, each one a
coherent unit, each one reversible. Review happens
asynchronously — the user reads the commit log, reviews
the diff, and at some later moment decides to push (or
asks for changes, or amends, or drops a commit). The
push moment is reserved as a separate decision.

Cost: a moderately growing pile of unpushed commits.
Benefit: every push, when it eventually happens, is
deliberate — the user knows exactly what's going up to
origin and what downstream effects (CI runs, mirror
syncs, deploy hooks) will fire from it.

Counter-example from the same period: in Medbook, an
earlier session had committed three Track status updates
but not pushed. When a second session walked into the
repo, those uncommitted modifications were visible
(modified-not-staged), and the second session had to
work around them. *Pushed work, by contrast, would have
been the second session's starting state — clean,
coordinated, with a clear delta.* So in multi-session
work, "deliberate push" doesn't mean "never push"; it
means *push when the work is ready to be coordination-
material for others*.

## Tagline

> Commit fast; push deliberately. Push is publication,
> not auto-save.

The push moment is where the work becomes real for
everyone else. Reserve it.

## See also

- [SmallBatchCommitsMergedOften](../SmallBatchCommitsMergedOften/SKILL.md)
  — small PRs merged often is the team-level discipline;
  this skill is the individual-publication-cadence
  discipline. They pair: small commits, batched into
  reviewed pushes, into small PRs that merge often.
- [MultiAICollaborationViaGit](../MultiAICollaborationViaGit/SKILL.md)
  — in multi-session repos, push is how sessions become
  visible to each other. Push deliberately is part of
  the multi-session hygiene.
- [CommitHygiene](../CommitHygiene/SKILL.md) — clean
  commit messages plus deliberate push produce a clean
  history that survives bisect, blame, and audit.

## Sources

Inferred from observing a multi-session AI-augmented
work session (2026-05) where the user's explicit
memory rule (*"No pushes without explicit permission"*)
created a clear separation between commit cadence (high,
AI-driven) and push cadence (low, user-reviewed). The
general principle is older — distributed VCS (Git,
Mercurial) was designed around the commit/push split —
but the AI-augmented context makes the discipline more
load-bearing than it was for the single-developer era.
