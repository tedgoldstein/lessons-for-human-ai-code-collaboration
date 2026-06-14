---
name: master-child-merge-discipline
description: When the master-child Track pattern is in play (one Master Trackfile + N Phase children + a long-lived master branch like `track-session-manager`), the canonical checkout MUST stay on `main` so the operator has a stable read-only inspection point. Use a dedicated worktree for the master branch — every Phase merge, every Master Trackfile update (Children list, Child progress, Files-likely-affected), every post-merge child-progress row goes there. Also — when merging a Phase branch into the master branch, the commit subject is `Track <phaseid>:` (the Phase owns the implementation scope), NOT `Master <masterid>:`. Trigger on `git checkout <master-branch>` in the canonical repo, on `git merge claude/track/<phaseid>`, on filing a new Phase child of an existing Master, on updating Master Children / Child progress, on `Master <id>:` subjects for commits that touch implementation files.
version: 0.1.0
---

# Master / child merge discipline

The master-child pattern (see `skills/master-child-track`) gives a
project a *master branch* — a long-lived integration line that
collects Phase merges and fast-forwards to `main` only on
`MasterApprove`. The pattern works only if **the canonical checkout
stays on `main`** as the operator's stable read-only inspection
point. The moment the canonical checkout is on the master branch,
the operator loses the ability to `git status` / `ls` / browse the
shipped state without first remembering to switch back.

The discipline: **never `git checkout <master-branch>` in the
canonical checkout.** Add a dedicated worktree for the master
branch and do every master-branch operation there:

- Phase merges (`git merge --no-ff claude/track/<phaseid>`).
- Master Trackfile updates (Children list, Child progress,
  Files-likely-affected, Internal notes).
- Post-merge Child-progress rows.
- Any `Master <id>:`-subjected commit.

The canonical checkout serves only as the `main` view.

## When to invoke

- You are about to `git checkout <master-branch>` (e.g.
  `track-session-manager`) in the canonical repo.
- You are about to `git merge claude/track/<phaseid>` from the
  canonical repo's HEAD.
- You are filing a new Phase child of an existing Master and
  registering it in the Master's Children list.
- You are appending a row to the Master Trackfile's `## Child
  progress` after a Phase event (`filed`, `stage:…→…`,
  `verify:…:…`, `closed:…`, `regression`, `design-update`).
- You are about to compose a commit subject `Master <id>:` for a
  commit that touches implementation files (not just the Master
  Trackfile itself).
- A `git status` in the canonical checkout reports the
  master-branch head instead of `main`.

## Why it matters

1. **Inspection point.** The canonical checkout is what the user
   `cd`s to when they want to see "what does main currently look
   like?" If it's been temporarily switched to the master branch
   for a merge, every subsequent unfamiliar `ls` / `cat` /
   `grep` in that directory shows the *master-branch* view, not
   `main` — silently. The user can't tell from a quick glance.
   Discipline restores their default-view contract.

2. **Master-branch state is not safe-to-publish state.** `main`
   is the published timeline. The master branch carries
   in-flight integration of Phase work that hasn't yet passed
   per-Phase Verify or MasterApprove. Showing the master-branch
   view as if it were `main` invites a `git push` (or a
   reader-conclusion) that publishes pre-Verify state.

3. **Worktree-per-branch is cheap.** `git worktree add` takes
   one command. The cost of a dedicated master-branch worktree
   (a few hundred MB on disk, one extra path in `worktree list`)
   is trivial next to the cost of the user repeatedly catching
   you on the wrong branch.

4. **Scope hook does not save you.** The commit-msg scope hook
   protects against committing files outside declared Track
   scope, but it does NOT detect "this commit went to the wrong
   branch." A master-subject commit on the wrong branch still
   commits. Discipline is the only barrier.

## The merge-subject rule

When merging a Phase branch into the master branch, the commit
subject is **`Track <phaseid>:` (Phase scope)**, not **`Master
<masterid>:` (Master scope)**.

Why: the Master Trackfile's `## Files likely affected` typically
lists only Trackfile paths (the Master + each Phase Trackfile).
A Phase merge brings *implementation files* (source code, tests,
prompts, docs) that live in the Phase's `## Files likely
affected`, not the Master's. The commit-msg scope hook reads
the allow-list from `Tracks/<id-from-subject>.md`. With a
`Master <id>:` subject the hook reads Master's narrow allow-list
and refuses the implementation files. With a `Track <phaseid>:`
subject the hook reads the Phase's allow-list, which already
covers the implementation files (it had to, for the Phase's own
commits during development).

Concrete pattern:

```
# Phase merge: subject names the Phase
git merge --no-ff claude/track/3yw6tysm -m "Track 3yw6tysm: merge Phase 1 …"

# Master-Trackfile-only commit (Children/Child-progress update):
# subject names the Master, scope is Master's narrow allow-list
git commit -m "Master 53d47epe: Phase 1 stage:analyze→verify …"
# or via wrapper:
node consumers/cli/bin/track.js git commit 53d47epe --message "Phase 1 stage:analyze→verify"
```

## Practical guidance

- **Bootstrap the master-branch worktree at the same moment the
  master branch is created.** When the operator picks the
  master-child pattern, immediately create both the branch and
  its worktree:

  ```
  git checkout -b <master-branch>     # in canonical
  git worktree add <repo>-<master-branch> <master-branch>
  git checkout main                   # canonical back to main
  ```

  Then never touch the master branch from the canonical checkout
  again.

- **Set a path convention.** Mirror the Phase-worktree
  convention. If Phase worktrees live at `<repo>-<phaseid>`,
  the master-branch worktree lives at
  `<repo>-<master-branch-name>` or `<repo>-master`. Pick one
  and stick with it.

- **Master commits and Phase merges both happen in that
  worktree.** Don't split: "small Master edit in canonical,
  Phase merge in dedicated worktree." Both go to the dedicated
  worktree. Consistency removes the "do I need to switch?"
  question.

- **Watch for the canonical checkout's git status.** If a
  routine `git -C <canonical> status` ever shows a non-`main`
  branch, the discipline has been broken — restore by
  `git checkout main` and audit what was just done.

- **Use `node consumers/cli/bin/track.js git commit <id>` for
  the Master-Trackfile commit.** The wrapper auto-prefixes the
  subject, pre-flights scope, and takes the owner-lock. Same
  reasons as for any other Track-scoped commit. (See
  `feedback_use_track_git_commit_wrapper` in project memory.)

## Common failure modes

- **"It's just a quick merge"** — the canonical checkout
  switches to the master branch for one merge, and the user's
  next `ls` shows post-merge state. Even one breach erodes the
  inspection-point contract. Dedicate the worktree once;
  benefit forever.

- **`Master <id>:` subject for a Phase merge** — gets refused
  by the scope hook because the Phase's implementation files
  aren't in the Master's narrow allow-list. The lesson is
  cheap to learn (the refusal is explicit and recoverable with
  `git merge --abort`) but easy to re-learn every project.

- **Updating Master Trackfile from the wrong worktree** — if
  the master-branch worktree exists but you accidentally edit
  the canonical-checkout copy of the Master Trackfile, your
  edit lands on `main` instead of the master branch. Subtle
  because both paths point at the "same" file, but they're
  different branch checkouts of it.

- **Pushing the master branch without authorization** —
  separate from this skill, but easy to fall into during a
  master-branch sweep. Pair this discipline with
  [`CommitHygiene`](../CommitHygiene/SKILL.md)'s push rule:
  commit freely, push only on explicit operator approval.

- **Naming the master branch `master`** — collides with the
  legacy default-branch name and creates `git push origin
  master` ambiguity. Pick a project-specific name like
  `track-session-manager`, `auth-rewrite`, `q3-platform`.

## When the principle DOES NOT apply

- **No master-child pattern in play.** A standalone Trackfile
  with no children doesn't need a master branch and doesn't
  need this discipline. The Phase worktree IS the work; the
  canonical checkout stays on `main` for inspection by default.

- **The operator explicitly opted into the single-worktree
  variant.** If during master-child setup the operator chose
  "phases land sequentially on `main`, no master branch", this
  discipline is moot — there is no master branch to keep out
  of the canonical checkout.

- **The master branch is being closed and fast-forwarded.**
  At master close (`MasterApprove`), the master branch
  fast-forwards into `main`. That FF can happen from the
  canonical checkout because it's the final landing — but
  it's still a moment worth pausing on (it's a publication-
  shaped act).

## Worked example

Master `53d47epe` (Track Session Manager) was set up with the
master-child pattern and a `track-session-manager` master branch
forked off `main`. Ten Phase children were filed
(`3yw6tysm` … `86chwwxa`).

The agent (this session) merged each Phase branch into the
master branch by:

1. `git -C <canonical> checkout track-session-manager` ← breach
2. `git -C <canonical> merge --no-ff claude/track/<phaseid>`
3. Edit Master Trackfile's Child progress in the canonical
   checkout
4. `node consumers/cli/bin/track.js git commit 53d47epe …`
5. `git -C <canonical> checkout main` ← restored

Steps 1 and 5 happened ten times. The operator's `git status`
on the canonical checkout could have caught the master branch
at any of those moments. They noticed at the end and called it
out: *"The whole point of making a distinguished branch like
`track-session-manager` is to give me an opportunity to review
globally where things stand before you merge to main."*

The fix going forward:

```
git -C <canonical> worktree add \
  /Users/tedgoldstein/Code/Track-master track-session-manager
```

All future Phase merges and Master Trackfile updates happen in
`/Users/tedgoldstein/Code/Track-master`. The canonical
`/Users/tedgoldstein/Code/Track` stays on `main`.

The commit subjects also followed the right shape:

- Phase merge: `Track <phaseid>: merge Phase N (…)`
- Master child-progress row update: `Master <masterid>: Phase N
  stage:analyze→verify …` (or `Track <masterid>:` —
  interchangeable for the wrapper's purposes).

The agent learned the merge-subject rule the hard way on Phase 1
when `Master 53d47epe: merge Phase 1 …` was refused by the scope
hook. Aborted and retried with `Track 3yw6tysm:`. The rule is
captured here so the next agent doesn't have to re-learn it.

## Tagline

> The canonical checkout is the operator's window on `main`.
> Don't fog it up for your own convenience.

## See also

- [skills/master-child-track](https://… project-local) —
  the pattern this discipline supports.
- [CommitHygiene](../CommitHygiene/SKILL.md) — commit ≠ push;
  push is a public commitment requiring operator approval.
- [OneChangeAtATime](../OneChangeAtATime/SKILL.md) — don't
  bundle "switch branch + merge + edit Master Trackfile +
  switch back" into one mental act. Each is an observable
  step.
- [SmallBatchCommitsMergedOften](../SmallBatchCommitsMergedOften/SKILL.md)
  — Phase merges should land often (one per Phase close), which
  makes the canonical-checkout breach more frequent if you do
  it from the canonical checkout. Dedicate the worktree
  *before* you fall into a high-frequency merge cadence.

## Sources

Session evidence (Track Session Manager Master implementation,
2026-05-20): the agent repeatedly switched the canonical
checkout to the master branch during Phase merges, was caught
by the operator only at the end of the session, and learned
the merge-subject rule by hitting a scope refusal on the first
Phase merge.

Echoes the standard Git convention that long-lived integration
branches live in dedicated worktrees in any multi-line-of-work
setup (release branches, feature trains, hotfix branches),
and the Track-specific convention that the canonical checkout
serves as the operator's stable read-only `main` view.
