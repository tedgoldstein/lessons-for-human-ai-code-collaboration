---
name: commit-hygiene
description: A commit is a permanent record; a push is a public commitment. Both deserve thought. Stage explicitly (no `git add -A` sweeping in secrets / build artifacts). Write the message so future-you can read `git log` and learn something. Don't commit to `main` directly — use a feature branch and a PR. Treat commit and push as two separate approvals; don't push proactively. Force-push only after asking — it overwrites the audit trail of the prior push.
version: 0.1.0
---

# Commit Hygiene

A commit is permanent. Even after `git commit --amend` the old SHA
lives in the reflog for weeks; once pushed, it lives in others'
clones, CI logs, and the host's database forever. The commit
message is the first thing anyone digging through history reads.

A push is a public commitment. It's visible to teammates, may
trigger CI, may auto-deploy. Reverting a pushed change is louder
than reverting an unpushed one — it shows up in everyone's
history.

The principle: **commit and push are two different acts with
different consequences.** Commit when work reaches a logical
resting state; push when you've decided it's ready for others to
see. Don't conflate them.

## When to invoke

- You're about to `git commit -am "stuff"`.
- You're about to commit directly to `main` because "it's just a
  small fix."
- You're about to push without anyone (human or otherwise)
  having seen the diff.
- You're about to force-push to overwrite a prior push.
- A commit message says `wip`, `fix`, `update`, `done`, or
  `stuff`.
- The diff includes a `.env`, a build artifact, `node_modules/`,
  IDE settings, or stray `console.log` debug output.
- A commit's author email is `you@laptop.local` because git
  config was never set.
- A `git log --oneline` line gives no clue what the commit did.
- An AI agent is about to commit + push without explicit user
  approval.

## The shape — two operations, two checklists

### Before commit

- **Stage explicitly.** `git add <files>` or `git add -p` for
  granular control. Avoid `git add -A` / `git add .` — they
  sweep up secrets, build artifacts, IDE settings, and stray
  files you didn't mean to commit.
- **Review what's staged.** `git diff --cached` (or
  `git diff --staged`) before committing. The commit will say
  exactly this. Make sure that's what you want.
- **One intent per commit.** A bug fix and a refactor are two
  commits. (Pairs with
  [OneChangeAtATime](../OneChangeAtATime/SKILL.md).)
- **Write a useful message.** Imperative present tense; summary
  line under ~72 chars; body (if any) says *why*, not *what*
  — the diff already shows what.
- **Author identity is correct.** `git config user.email`
  should be the right one for this repo — especially in
  multi-account setups (work vs. personal).

### Before push

- **Check the branch.** Are you on the right branch? Are you
  about to push to `main` when you meant a feature branch?
- **Has the work been seen?** Pushing structural decisions
  before showing them is a process slip. Show the diff / file
  layout / commit summary first; get sign-off; then push.
- **Has the user said "push"?** "Commit" doesn't imply "push."
  Treat them as separate approvals. Working locally on a
  feature branch and pushing it are different acts.
- **Force-push? Ask first.** A force-push overwrites the prior
  push's audit trail — and any review comments anchored to the
  prior SHAs lose their context. Always ask; the cost of
  asking is low, the cost of erasing review history is real.

### Commit-message anatomy

```
<short imperative summary, under ~72 chars>

<longer explanation of WHY this change is needed,
what tradeoffs were considered, what alternatives
were rejected — the diff already shows WHAT changed>

Refs: #123
```

The summary line is what shows up in `git log --oneline`, in
the `git bisect` output, in the PR title (often), in the
revert message. Make it useful in those contexts. *"Fix
bug"* is useless. *"Reject empty usernames in signup"* is
useful.

## Why it matters

1. **History is debugging fuel.** Future-you debugging a
   production incident at 3am has only `git log`, `git blame`,
   and `git bisect`. Good messages turn a four-hour mystery
   into a ten-minute investigation. Bad ones strand you.

2. **Reviewers consume changes commit-by-commit.** A reviewer
   reading a 12-commit PR follows the *story* across commits —
   refactor here, behaviour change there, tests there. Garbled
   history means re-deriving the story from the diff.

3. **Bisect needs single-intent commits.**
   [TheBisectMindset](../TheBisectMindset/SKILL.md) finds the
   introducing commit in log₂(N) steps — but only if each
   commit is one thing. A commit that bundles three changes
   lands the bisect on the right SHA and leaves you still
   guessing which line broke things.

4. **Stray files cause real harm.** A committed `.env` is a
   credential leak — and rotating doesn't remove it from
   history. A committed `node_modules/` is a 200MB review
   surface. A committed IDE setting is a future merge
   conflict.

5. **Push is louder than commit.** A commit you regret you
   `git commit --amend` away (if unpushed). A push you regret
   you have to *explain*: a force-push (overwriting others'
   clones), or a revert commit (now in everyone's history
   forever). Treat the push as the commitment and the commit
   as the rehearsal.

## Practical guidance

- **Commit early, commit often *locally*.** A local commit
  costs nothing; it's a checkpoint. Don't be afraid of WIP
  commits — clean them up before push with `git rebase -i`
  (see
  [OrganisingGitPullRequests](../OrganisingGitPullRequests/SKILL.md)).
- **Never commit to `main` directly.** Even for "trivial"
  fixes. Use a feature branch and a PR. The branch is the
  audit trail; the PR is the review surface. The cost of a
  branch is one command; the cost of a bad direct-to-`main`
  commit is a revert in everyone's history.
- **Commit ≠ push.** Don't auto-push after every commit. Push
  is a separate decision: *is this ready for others to see?*
- **Show structural changes before pushing.** New directory
  layouts, new files, `.gitignore` changes, template shapes —
  these are *easy* to revise before push, *costly* to revise
  after. Show the layout; get sign-off; then push.
- **Force-push: only on your own branch, only after asking.**
  Force-pushing your feature branch after a rebase is fine *if*
  nobody else is working on it. Force-pushing `main` is almost
  never fine. Either way, say what you're about to do.
- **`.gitignore` is part of the codebase.** Keep it current.
  When you find yourself adding `.idea/` or `.DS_Store` to a
  commit, that's a signal your `.gitignore` is incomplete.
- **Generated files are not commits.** Build outputs, compiled
  artifacts, derived files — keep them out. The exception is
  lockfiles that *are* the source of truth (`package-lock.json`,
  `Cargo.lock`, `poetry.lock`); commit those, then leave them
  alone unless the dependency change is the point of the commit.
- **No secrets, ever.** A committed credential is a permanent
  leak; rotating the credential doesn't remove it from
  history. If it happens, rotate immediately, then rewrite
  history with `git filter-repo` if the secret is still fresh
  in the upstream.
- **Write the message for the reader you don't know.** That
  reader might be future-you in two years, a new hire, or a
  security auditor. They have no context. The message provides
  it.

## Common failure modes

- **`git commit -am "stuff"`.** Stages everything (including
  surprises), says nothing. Both halves are wrong.
- **The "I'll fix the messages later" plan.** "Later" rarely
  comes; the WIP messages ship to `main` and live there
  forever.
- **Drive-by stray files.** A `.DS_Store`, a `.env.local`, a
  `tmp.txt`, a `*.log` ride along because `git add .` swept
  them in. Stage explicitly.
- **The committed `console.log` / `print()` / `dbg!()`.**
  Debug output left in committed code. Usually paired with a
  commit message that doesn't mention it.
- **The "merge `main` into branch" loop.** Every time `main`
  moves, the branch acquires another merge commit. History
  becomes unreadable. For a private feature branch, prefer
  `git rebase main` instead.
- **The proactive push.** Work, commit, push — without
  checking with anyone. The push becomes a fait accompli;
  revising now means a force-push or a revert. Both are
  louder than holding the push until approved.
- **Force-push that erases review.** A reviewer asked for
  changes; the author rebased, force-pushed, and overwrote the
  previous SHAs. The review comments anchored to those SHAs
  now point at nothing. Either don't force-push while a
  review is in progress, or summarise *what changed since the
  prior push* in a comment.
- **Committing under the wrong identity.** Personal email on
  work commits (or vice versa). Set `user.email` per-repo
  with `git config user.email`.

## When the principle DOES NOT apply

- **Solo project, no audit trail needed.** Commit how you
  like; push when you like. The hygiene serves a future
  reader; if there is none, the cost may exceed the benefit.
- **Hot-code mode / crisis recovery.** When production is on
  fire, commit and push the fix. Don't write a perfect
  message; don't open a PR. Clean up afterwards. The hygiene
  is overhead during a fire.
- **Throwaway branches.** A branch you'll delete in an hour
  after a one-off experiment doesn't need polished history.
  Just don't merge it.
- **Force-push on a private branch with zero other users.**
  Fine. Just be sure of the "zero other users" part — when
  in doubt, ask.

## Tagline

> Commit is permanent; push is public. Both deserve a moment
> of thought.

A clean history is debugging fuel for future-you. A well-named
commit, a small focused diff, a thoughtful message — these
compound into a codebase that's pleasant to navigate ten
years later.

## Sources

Distilled from general engineering practice. Pairs closely
with [OneChangeAtATime](../OneChangeAtATime/SKILL.md) (one
intent per commit), [SmallBatchCommitsMergedOften](../SmallBatchCommitsMergedOften/SKILL.md)
(branch lifetime), [OrganisingGitPullRequests](../OrganisingGitPullRequests/SKILL.md)
(reshape history before review), and [TheBisectMindset](../TheBisectMindset/SKILL.md)
(why single-intent commits matter). Tim Pope's "A Note About
Git Commit Messages" (2008) is the canonical English-language
essay on message style; the Conventional Commits format
(conventionalcommits.org) is one structured convention layered
on top.
