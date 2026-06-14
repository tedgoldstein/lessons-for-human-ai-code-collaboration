---
name: multi-ai-collaboration-via-git
description: When two or more AI coding sessions share a repository concurrently — or an AI session and a human work in parallel — git is the coordination protocol. Sessions share files and history, not state. Load when spawning a second session on a live repo, when uncommitted changes appear that you didn't make, when a `git stash` is tempting in a multi-session repo, or when cloud routines need to know rules your laptop session has internalized.
version: 0.1.0
---

# Multi-AI Collaboration via Git

A growing reality: a developer runs an interactive Claude Code
session on their laptop. They also have a cloud-scheduled
"routine" (a headless agent) working on a different branch of
the same repo. And maybe a third tab with another AI session
on a different feature. Three concurrent agents, one repo.

The agents can't share state directly. Each loads its own
context window, holds its own working memory of what it's
doing, and can't see the others' in-flight reasoning. **What
they share is the file tree and the git history.** That's the
coordination protocol. The discipline is to make the
file-tree-and-git-history rich enough for the agents to
collaborate without colliding.

This skill is the hygiene that keeps multi-agent work
productive instead of chaotic.

## When to invoke

- You're about to run a second Claude Code (or similar) session
  on the same repo while a first session is mid-task.
- You're setting up a `/schedule`d cloud routine that will work
  on the same repo as your laptop session.
- A `git stash` is tempting in a multi-session repo.
- You notice another session has uncommitted changes in your
  working tree — and you didn't make them.
- Two sessions are about to file conflicting designs for the
  same feature.
- You're writing the prompt for a cloud agent and realize the
  agent can't see your `~/.claude/memory/` rules.
- A cloud routine just committed code that contradicts a rule
  you've told your laptop session a hundred times.

## The shape

| Property | Aim for | Why |
|---|---|---|
| **Branch per session** | one branch per concurrent session, named for its purpose | Sessions don't collide on the working tree; merges are auditable |
| **Scope rule per session** | explicit list of files each is allowed to touch | A session that goes outside scope halts and reports rather than expanding silently |
| **Uncommitted state** | belongs to whichever session is in flight; never trample | Stashing or resetting destroys another session's working memory |
| **Cloud context inlining** | LOCAL MEMORY rules baked into the cloud session's prompt | The cloud agent literally cannot see `~/.claude/` |
| **Cross-session handoff** | through a committed file (a Track, a routine artifact, a TODO) | Two ships in the night need a written letter |
| **Merge cadence** | each branch lands its own PR; no automatic cross-branch coordination | Reviewer (or human owner) is the final arbiter |

## Why it matters

1. **Stashing destroys hidden work.** `git stash` is a personal
   tool from the single-user era. In a multi-session repo, the
   uncommitted changes you're about to stash might be another
   session's in-flight work. Asking before stashing is the
   minimum; the better default is "don't stash; switch branch
   or work around the dirty tree."

2. **Cloud agents can't see what laptops can.** A cloud-
   scheduled routine boots in Anthropic's (or anyone's)
   container. It clones the repo fresh. It doesn't have access
   to the user's `~/.claude/memory/`, their local CLAUDE.md
   that lives outside the repo, their settings, their
   conversational history. **Any rule the user has trained
   the laptop session to follow must be re-inlined into the
   cloud session's prompt or it doesn't exist for that
   session.**

3. **Two sessions will independently invent the same feature.**
   When you fan out work to multiple agents, expect collisions.
   Two agents working in parallel on parts of a methodology
   will both reach for the same orthogonal-to-the-stages
   field; they'll just give it different names ("Running" vs
   "Execution"). The collision isn't a bug; it's evidence
   they're both right.

4. **Branches encode intent.** A branch named
   `claude/Track/<id>` says "this branch is doing work on
   Track `<id>`, started by a Claude session." A reviewer
   scanning `git branch -a` sees the in-flight Claude work
   at a glance and can decide whether to wait for it.

5. **Audit trail by default.** The git log records who did
   what when. If a session writes a commit message that
   says "wrote env fingerprint, did NOT print secret
   values," that's documentary. Future investigators
   (humans or AIs) can read the trail.

## Practical guidance

- **Branch convention per session type.** Pick a prefix and
  enforce it. `claude/Track/<id>` for Track-driven work;
  `claude/<area>/<topic>` for cross-cutting; `feature/<name>`
  for human work. The prefix makes it obvious which branches
  came from which kind of session — and CI/branch-protection
  rules can act on the prefix (e.g., "branches starting
  `claude/` are allowed to be force-pushed by the
  scheduling system; `main` is not").

- **One scope rule per session.** Tell each session:
  - which branch it operates on,
  - which files it may touch (whitelist or blacklist),
  - which verbs / verdict types it may issue,
  - what to do if it discovers the scope is wrong (halt
    and report, not silently expand).
  The cloud-routine artifact pattern (a `routines/<id>.md`
  file with this baked into the prompt) is one
  implementation.

- **Never trample uncommitted work.** If your laptop session
  encounters uncommitted changes you didn't make in a
  multi-session repo, the default is to leave them alone.
  Commit your own work to a fresh branch; stage only what
  you authored; let the other session land its own work
  later. This sometimes means an "ugly" working tree for
  a while. That's fine; it's the price of safety.

- **Inline LOCAL MEMORY for cloud agents.** Memory rules
  that live in `~/.claude/memory/`, in personal preferences,
  in conversational history — none of them propagate to a
  cloud agent. The cloud prompt must literally restate the
  rules. "User said never push to main: do not push to
  main. User dislikes 'I'm sorry to hear that' phrasings:
  don't use them. Past Claude sessions have made the
  push-to-main mistake; the rule is non-negotiable." All
  of this goes IN the cloud prompt because the cloud agent
  has no other way to learn it.

- **Document predictable collisions.** When you discover
  two sessions independently invented the same thing, file
  it. The Track / RFC / design doc that names "Running vs
  Execution: both invented; Running wins" is itself a
  cross-session-coordination artifact for the next time
  this happens.

- **A "scratchpad" file in the repo for cross-session
  notes.** Sometimes the cleanest handoff is a file —
  `NOTES.md`, `Track/<id>.md`, a routine artifact —
  written by session A so session B can pick it up. The
  file is the letter; the next session reads it.

- **Don't push without permission.** Local commits are
  cheap and reversible. Push is publication; once it's on
  origin, other sessions / CI / cron jobs may act on it.
  Treat the push as a deliberate ceremony, not a
  reflex. (A common user preference, and one that
  decouples local AI-driven work from upstream effects.)

## Common failure modes

- **Stash-and-go.** Session A runs `git stash` to clear
  the working tree, does its work, restores the stash —
  except session B's uncommitted changes were in there
  too and never come back. *Don't.* Switch branches
  cleanly or work around the dirty tree.

- **The phantom rule.** "But I told the cloud agent never
  to push to main!" — except the user told the *laptop*
  agent. The cloud agent never heard. Inline the rule in
  the prompt or it doesn't exist for that session.

- **The race-to-name.** Session A introduces a field
  called `Execution`; session B (working in parallel)
  introduces an equivalent field called `Running`. Both
  are pushed before anyone notices. Resolution is
  cheap (pick one, rewrite the other) but the lesson
  is to plan field names in a shared design surface
  (a Track) before two sessions reach for the same
  concept independently.

- **The unfinished PR cascade.** Session A's PR sits
  unmerged. Session B starts work that depends on
  session A's branch (or duplicates it because session
  B didn't know session A's branch existed). Three
  branches now stack fragilely. The fix is making
  in-flight work visible — a status dashboard, a
  `git branch -a | grep claude/` glance, a check at
  session start.

- **Silent scope expansion.** Session A is told to
  modify file X. Session A decides "while I'm here,
  the related file Y also needs work." Session A
  modifies Y. Now session B's working tree includes
  unexpected changes to Y. *Halt-and-report* is the
  rule: when the scope feels wrong, stop and surface
  it rather than silently expanding.

- **Conflating "AI did this" with "human approved
  this."** A cloud routine wrote a commit; that's
  not yet a human-reviewed change. The PR review is
  where the human reviews. Branches AI agents push
  to are not automatically trustworthy.

## When you need stronger coordination

The patterns above are lightweight — they keep concurrent
sessions from stomping each other but don't impose heavy
process. When the stakes are higher (shared production
infrastructure, multiple humans + AIs, regulated
environments), add:

- **Branch protection rules** that match the session-prefix
  convention. `claude/*` branches can be force-pushed
  by the scheduler; `main` cannot, and PRs to `main`
  require human review.
- **A dispatcher.** A meta-process that knows which
  sessions are running, what scopes they're working in,
  and refuses to start a new session that would collide.
- **Locks at the file level.** `git lfs locks`-style
  exclusive holds on files actively being modified.
  Heavy; reserve for genuinely hot files.
- **A merge queue.** If your CI / deploy depends on
  ordered landings, use a tool like Mergify, Aviator, or
  GitHub's native merge queue. Sessions land via the
  queue, not by direct merge.

For most small-team / personal-AI work, the lightweight
patterns are enough.

## Worked example

A user runs three concurrent Claude Code sessions on a
shared methodology repo. Session A (laptop) refreshes the
methodology's canonical contract. Session B (laptop,
earlier) is independently adding a new orthogonal-to-the-
stages field called `## Execution`. Session C (cloud
routine) is verifying a deployed preview against the
contract.

Session A walks in, finds:
- An in-flight branch `claude/skill/execution-dimension`
  with one committed change (B's work) and three
  uncommitted Track status updates (B's close-out work).
- An untracked test artifact from session C's previous
  run.

Session A:
1. Surfaces the collision to the user — "B's
   `## Execution` is the same concept as canonical's
   `## Running`; which wins?"
2. After the user picks "Running wins," renames B's
   `## Execution` sections to `## Running` across every
   affected Track.
3. Refreshes the canonical files via `init --force`.
4. Stages only the files A actually modified — leaves
   B's three uncommitted Track mods alone (those are
   B's work; A didn't author them and won't commit
   them).
5. Commits with a message that explicitly notes the
   intentionally-left-out files.
6. Writes a routine artifact for session C's next run
   that inlines the LOCAL MEMORY rules (never push to
   main, branch naming, token-handling rules) because
   session C can't see them otherwise.

The repo state after A's work: clean for the files A
touched, dirty for B's in-flight work (untouched by A),
ready for B to land B's own commit when B comes back.
No one's work was destroyed; everyone's intentions are
auditable from `git log`; the collision was resolved by
explicit decision rather than by accident.

## Tagline

> Concurrent agents share files, not state. The git tree
> is the protocol; the prompt is the rule book; the
> commit is the handoff.

## See also

- [TheContractIsTheArtifact](../TheContractIsTheArtifact/SKILL.md)
  — multi-AI works best when the system's contract is a
  cheap file format. The contract is the substrate; git
  is the coordination layer.
- [CommitHygiene](../CommitHygiene/SKILL.md) — commit
  messages that describe intent, scope, and "what NOT
  this commit does," become essential for cross-session
  legibility.
- [SmallBatchCommitsMergedOften](../SmallBatchCommitsMergedOften/SKILL.md)
  — small commits make cross-session merges cheap.
  (Note: "push when ready" can override "merge often"
  in personal-AI work — local commits are cheap to
  accumulate, but the small-batch logic still applies
  within each session's branch.)

## Sources

Inferred from observing concurrent Claude Code sessions
working on a shared methodology repo (Track / Medbook /
Labbook, 2026-05), including a real collision where two
sessions independently invented the same field with
different names. The branch-prefix convention, the
routine-artifact-with-inlined-LOCAL-MEMORY pattern, and
the never-trample-uncommitted-work discipline are all
drawn from that work. As multi-agent coding becomes
more common, the patterns will mature; this is an
early-2026 snapshot.
