---
name: causal-divergence
description: Modern software runs across multiple simultaneously-valid views of state — git branches, distributed replicas, concurrent sessions, concurrent threads, simulations, multiple programmers, and (the new and most frequent case) multiple AI agents. These views can disagree without either being wrong. The design pattern is *not* to force one absolute truth; it is to designate **one common central artifact where divergence is detectable** — a reconciliation point every actor reads. For local agentic work that artifact is usually a contract file (a Trackfile / Radar / workflow ticket); for federated systems it is a registry, a vector clock, a consensus log. Load this skill when spawning parallel agents, when two sessions report different facts about the same state, when a cold reviewer arrives, or when designing a system that multiple actors will read and write concurrently.
version: 0.1.0
---

# Causal Divergence

Two AI agents open the same project from two checkouts on the same
laptop. They read the same source files. They read the same workflow
ticket. They reach opposite conclusions about whether a bug is fixed —
and **neither is wrong**. Each is reading a different point in causal
history. The methodology said where each agent is in the *workflow*;
it didn't say where each agent is in the *content* timeline. Without
that, every parallel actor is correct from its own vantage and the
human is the only thing reconciling them.

This isn't a bug. It is a structural property of modern software.
Branches, distributed databases, concurrent sessions, concurrent
threads, simulations, multiple programmers — and now multiple AI
agents — all create simultaneously-valid but potentially contradictory
views of state. Causal divergence is the technical name for the
result. A system that has more than one actor has, by construction,
more than one timeline.

**The design pattern is one common central artifact where the
divergence is detectable.** Not a single source of truth — there can't
be one when state is sharded across owners. Not "merge fast and never
diverge" — that defeats the parallelism that made the actors useful
in the first place. The discipline is to designate a *reconciliation
point*: a file, a registry, a log, a clock that every actor reads
before claiming "the state is X." When the actor can show you the
reconciliation point it consulted, divergence becomes legible. When
it can't, divergence is silent and a human has to be the detective.

## When to invoke

- You're about to spawn a second AI agent (parallel Claude Code
  session, scheduled cloud routine, headless verifier) against a
  shared repo or shared state store.
- Two sessions report different facts about the same Radar, ticket,
  record, or file.
- You're about to claim work is "done" or "fixed" but the change
  exists only on a branch / replica / worktree that not every consumer
  can see.
- A cold reviewer — human or AI agent — arrives to verify and there's
  no field in the artifact telling them which version of the world the
  prior verdicts were cast against.
- A merge surfaces work you didn't know existed.
- Distributed replicas of the same record disagree.
- A user says "I thought this was fixed — why is your session showing
  it as not fixed?"
- You're designing a system where multiple actors will read and write
  the same record concurrently.
- You're tempted to enforce "merge fast" or "one writer at a time" as
  a way to *avoid* the divergence problem rather than make it legible.

## The shape — the reconciliation point

The design pattern in one diagram:

```
   actor A ─┐
            ├─→  reconciliation point  ─→  legible divergence
   actor B ─┤    (one common artifact)     (every actor can see it)
   actor C ─┘
```

Every actor — human, AI agent, replica, thread — consults the
reconciliation point before making claims about state. Claims are
tagged with what they read.

Three implementations, sized to the system:

| System scope | Reconciliation point | What it carries |
|---|---|---|
| **Local file-based methodology** (Track / Radar, Apple Radar, Jira-like tickets, design docs) | The ticket / Trackfile / spec markdown file | Content-state fields: which commit(s) implement the work, which trunks have absorbed them, which SHA each verdict was cast against. See the Trackfile `## Commit` / `## Integration state` example in [worked example](#worked-example) below. |
| **Git itself** (multi-branch, multi-worktree development) | The commit DAG | Parent SHAs, branch refs, merge commits. Always present; consult it. `git log --all --grep <id>` is the cold-reader recovery move. |
| **Distributed system** (replicas, federated services, multi-region) | A registry, vector clock, version vector, consensus log, CRDT metadata | "What does each replica know about each other replica's state, and as of when?" |

The **right reconciliation point is whatever every actor naturally
reads anyway**. A Radar markdown is the right place because every
consumer (Claude session, Verify Agent, dashboard, human) opens that
file. A side-channel JSON index is the wrong place because consumers
don't naturally consult it; they would have to be told to. The
reconciliation point should be on the path of least resistance.

## Why it matters

1. **AI-agentic programming makes causal divergence frequent, not
   occasional.** A single developer with one terminal hits divergence
   rarely. A developer with a laptop session, a cloud routine, a
   parallel verifier, and a colleague's worktree open simultaneously
   hits it daily. The number of timelines scales with the number of
   parallel actors, which scales with how many AI agents you're
   running. Five concurrent agents means ten pairwise consistency
   surfaces. The methodology has to assume divergence is the common
   case, not the edge case.

2. **AI agents lack out-of-band channels.** Humans patch silent
   divergence with Slack threads, hallway conversations, "I saw a
   memo," tribal knowledge. An AI agent has its conversation context
   and the filesystem. Nothing else. Whatever a human would have
   learned in the hallway, the AI agent has to learn from a file. The
   reconciliation point *is* the hallway for AI agents.

3. **Workflow state ≠ content state.** A ticket can say `status:
   verify` while the actual code change lives only on a branch nobody
   else has checked out. A reader of the trunk sees no fix; a reader
   of the branch sees the fix; both readings of the ticket agree on
   `verify`. The methodology has to track both dimensions or any cold
   reader will draw the wrong conclusion.

4. **The failure mode is silent.** Each actor's view is internally
   consistent. There is no error, no exception, no compile failure.
   The divergence surfaces only when (a) two actors compare notes,
   (b) a merge conflicts, or (c) a human notices contradictions hours
   later. By then time has been spent on the wrong premise.

5. **Forcing single-consistency defeats the parallelism.** "Merge
   fast and never have unmerged work" eliminates divergence but also
   eliminates the speed gain from parallel work. The right trade is
   the CAP-theorem trade: accept temporary inconsistency, invest in
   making the inconsistency legible. Strong-consistency-everywhere is
   a discipline that doesn't scale to many agents.

6. **The reconciliation point is a design choice that has to be made
   early.** Retrofitting `## Commit` and `## Integration state` onto
   thousands of existing tickets is painful. Choosing where the
   reconciliation point lives, and what fields it carries, at design
   time is cheap. Choosing it after divergence has already burned
   hours is expensive.

## Practical guidance

- **Name the reconciliation point explicitly.** In your system's
  contract document (the SKILL.md, the README, the design doc), say:
  "This is the file every actor reads to know where state lives. If
  you're claiming X, show what you read here." The naming makes the
  expectation enforceable.

- **Carry content-state alongside workflow-state.** For each piece of
  work tracked in your system, the reconciliation point should answer:
  - Which commit(s) implement it?
  - Which trunks / branches / replicas carry those commits?
  - Which version was each verdict / decision / approval cast against?
  - When did each integration moment happen?

- **Tag every claim with what it was made against.** A verdict, a
  test result, a review comment is all of: *who* / *what* / *against
  what version*. The third part is what enables staleness detection
  later. An EngineeringVerify APPROVE against SHA `6cc0ab6` is not
  automatically valid against SHA `7f1aabc`.

- **Push truth onto disk in self-describing fields, not into
  conversation history.** Anything an AI agent needs to know to
  reconcile should be a field in the contract file, not something the
  user is expected to say in chat. The conversation context dies when
  the session ends; the file lives.

- **For distributed systems, pick a reconciliation mechanism
  proportional to the scale.** Vector clocks for partial-order
  reasoning across N nodes. CRDTs for fields that need automatic
  merge. Consensus logs (Raft, etcd) when you need a globally-ordered
  decision sequence. Version vectors for client-server replication.
  Each tool has a different cost; match it to the divergence rate.

- **Make the reconciliation point read-first for any actor making
  claims.** If an actor (especially an AI agent) is about to assert
  "the state is X," its first move should be to consult the
  reconciliation point. If the contract says "read `## Commit` before
  claiming a fix exists or doesn't exist," the agent has a rule it
  can follow. Without that rule, the agent reads source code, reads
  workflow state, draws a conclusion — and the conclusion can be
  silently wrong.

- **Document the divergence-detection move for cold readers.** The
  recovery move from a silent-divergence state should be in the
  contract. For Radar that's "if `## Commit` is empty or stale, run
  `git log --all --grep <id>` and `git branch --contains <sha>` before
  re-implementing." Cold readers should have a one-line escape hatch
  back to the truth.

## Common failure modes

- **Treating workflow state as content state.** The ticket says
  `verify`, the reader assumes that means the fix is on the trunk,
  the assumption is wrong. The reader's mental model is correct for a
  small-team, single-checkout world; not for the multi-agent reality.

- **Assuming AI agents will use out-of-band channels.** "I'll just
  tell the agent in chat that the fix is on branch X." That works
  until the next session, which can't see chat. The next session has
  only the file. If the file doesn't carry it, the next session
  doesn't know.

- **Adding a reconciliation point that isn't on the read path.** A
  separate `reconciliation.json` file in the repo, generated by a
  side-tool, that consumers are supposed to consult. They won't. The
  reconciliation point has to be the artifact actors are *already*
  reading.

- **Recording timestamps without sources.** A verdict logged with a
  timestamp but not "against what SHA / version / build" gives no
  staleness signal. The timestamp tells you *when* the verdict was
  cast; the source tag tells you *whether it still applies*.

- **Mistaking "single source of truth" for "single reconciliation
  point."** SSoT says one authoritative owner for each piece of state.
  Causal-divergence says one place where all owners' positions can be
  observed and reconciled. They solve different problems. A federated
  system has no SSoT *and* needs a reconciliation point. A monolith
  has SSoT *and* may still need a reconciliation point for cross-
  branch work.

- **"Just merge fast and the divergence goes away."** It doesn't. It
  shifts the divergence window from "between feature work and merge"
  to "between merge and the next reader who hasn't pulled." The
  problem is structural, not solvable by speed.

- **AI-agent-specific: assuming the agent's prior conversation is
  shared.** Two sessions running in parallel each have their own
  context window. State that lives only in conversation context
  vanishes the moment another agent shows up. Push the state to disk
  *while* the original session is alive, not "later when we have time
  to write it down."

## When the principle does NOT apply

- **Single-actor, single-checkout work.** A solo developer with one
  terminal on one branch genuinely doesn't have divergence. Don't
  over-engineer the reconciliation point for a system that has one
  reader.

- **Throwaway scratch work.** Experiments, exploratory branches you
  intend to delete, prototype scripts. The cost of designing a
  reconciliation point exceeds the benefit when the work won't
  survive.

- **Tightly-controlled production paths where strong consistency is
  cheap.** Some systems can afford global locks, serialized writes,
  one-writer-at-a-time. If your scale allows it, strong consistency
  eliminates the problem rather than making it legible. Most
  systems' divergence cost is below their strong-consistency cost,
  which is why the AP side of CAP usually wins — but not always.

- **When the actors genuinely share state in real time.** Two threads
  with a mutex around a shared variable have no causal-divergence
  problem; they have a concurrency problem. Different discipline.

The principle is for systems where multiple actors are *legitimately*
operating on overlapping state asynchronously and cannot be forced to
synchronize. That is the modern default; it isn't every system.

## Worked example

> The example below describes events from May 2026 in the project
> then called *Radar* and since renamed *Track* (Trackfile / Tracks/
> / Tracker.app — see [tedgoldstein/radar](https://github.com/tedgoldstein/radar)
> master Trackfile `7j4aw332` for the rename rationale). The paths
> in the narrative reflect the post-rename layout (`Tracks/<id>.md`,
> `claude/track/<id>`); the events themselves predated those names.

A user is driving the Track methodology — a markdown-file-per-ticket
system — with two concurrent Claude sessions. Session A is on a laptop
in a git worktree at `.claude/worktrees/claude+track+1tbhk4x6`, on
branch `claude/track/1tbhk4x6`. It has just committed a fix for the
bug (commit SHA `6cc0ab6`). The main tree at `/Users/.../Code/radar`
is on branch `plugin-experiment`, which does not yet carry the fix.

The user, intending to verify the fix, opens a second Claude session
from the main tree's directory. They ask "is this bug fixed?" The
second session:

1. Reads `consumers/radar.html/radar-detail.jsx` in its checkout. It
   sees the un-patched code — the conditional rendering that destroys
   xterm.js scrollback on subtab switch. Source says "not fixed."
2. Reads `Tracks/1tbhk4x6.md` in its checkout. It sees `Status: open`,
   `Running: Waiting`, EngineeringVerify pending, no Verify-history
   entries. Workflow state says "not fixed."
3. Concludes "the bug is not fixed" and proposes to implement the fix.

The user pushes back: "I thought this was fixed. I'm here just to
verify." The second session runs `git log --all --grep 1tbhk4x6`,
discovers commit `6cc0ab6` exists on `claude/track/1tbhk4x6` (a
worktree it didn't check), and recovers: "Got it — I had a stale
picture."

**Diagnosis.** The second session was not buggy. Its reading of source
and workflow state was correct *for its checkout*. The methodology
never said "before claiming a fix is or isn't present, consult the
reconciliation point that says which branch / commit carries it" —
because there was no such field. The Trackfile carried workflow
state (`Status`, `Running`, verdicts) but no content state. The two
sessions held two valid timelines; the contract had no place where
their relationship could be detected.

**Fix.** Add to the Trackfile template — the file every actor reads:

- `## Commit` — table: `SHA | branch | date | summary`. Every commit
  implementing this Trackfile gets a row. The cold reader knows where
  the fix lives without running git plumbing.
- `## Integration state` — table: `trunk | last-merge-SHA |
  as-of-date | how`. Which trunks carry the fix as of when. The cold
  reader knows whether their checkout has the fix.
- An `against-SHA` column on `## Verify history`. Every verdict
  records the version it was cast against. A passing verdict against
  `6cc0ab6` is not automatically valid against `7f1aabc`; staleness is
  detectable.

The Trackfile is now the reconciliation point. The next cold session
opens the file, reads `## Commit`, and knows immediately where the
fix lives. No `git log --all` archaeology. No silent divergence. No
re-implementation of a fix that already exists.

**Generalization.** The same shape applies anywhere actors operate on
shared state asynchronously. The reconciliation point is whatever
artifact the actors naturally read. For Radar it is the markdown
file. For a federated database it is the version vector. For a
distributed log it is the offset map. The mechanism varies; the
principle is constant: **one place where divergence is detectable,
on the path every actor already takes**.

## Tagline

> Causal divergence is the new normal. Pick the artifact every
> actor reads; carry provenance on its face; make the disagreement
> legible.

## See also

- [MultiAICollaborationViaGit](../MultiAICollaborationViaGit/SKILL.md)
  — the operational-hygiene companion. That skill is about *how* AI
  sessions coordinate through git (branch conventions, scope rules,
  never trample uncommitted work). This skill is about *why* — the
  underlying theory of divergence that makes the hygiene necessary,
  and what to add when git alone isn't enough.

- [TheContractIsTheArtifact](../TheContractIsTheArtifact/SKILL.md) —
  the contract is the durable thing. This skill says *which fields*
  the contract must carry to survive concurrent actors: content state
  alongside workflow state, provenance, against-SHA tags.

- [SingleSourceOfTruth](../SingleSourceOfTruth/SKILL.md) — the
  opposite corner. SSoT prescribes one authoritative owner of state.
  Causal-divergence applies when forced into multiple owners
  (branches, replicas, parallel agents) and shows how to keep the
  multiplicity legible.

- [CommonsWithDivergentClones](../CommonsWithDivergentClones/SKILL.md)
  — an explicit divergence pattern: a parent repo with N descendant
  clones that periodically merge one-way from parent. The clones are
  *intended* timelines; the reconciliation point is the parent's
  history. This skill generalizes the discipline.

- [PushIsPublication](../PushIsPublication/SKILL.md) — push is the
  moment a local timeline becomes visible to other actors. Until then,
  the reconciliation point has no record of the work. Push deliberately
  for the same reason you update the reconciliation point deliberately:
  it changes what other actors can see.

- [LogsAreAFeature](../LogsAreAFeature/SKILL.md) — distant cousin.
  Structured logs with correlation IDs are a per-event reconciliation
  point for runtime systems; provenance fields are a per-artifact
  reconciliation point for state systems. Same instinct, different
  scope.

## Sources

The framing is the user's: *modern software engineering increasingly
resembles a managed system with many simultaneously-valid views; the
"paradox state" occurs when they collide, and the design response is
synchronization, provenance, conflict resolution, consensus, and safe
isolation of alternate timelines rather than enforcement of a single
absolute reality.* Surfaced live during a 2026-05-17 working session
on the Radar methodology, when two concurrent Claude Code sessions on
the same repo reached opposite conclusions about whether a bug was
fixed — neither wrong, both reading valid but different points in
causal history.

Intellectual ancestors: Leslie Lamport, *Time, Clocks, and the
Ordering of Events in a Distributed System* (1978) — the foundational
paper on causality without a global clock. Eric Brewer's CAP theorem
— the trade between consistency, availability, and partition
tolerance; the lesson here is largely the AP side applied to the
development process. CRDTs (Shapiro, Preguiça, Baquero, Zawirski,
2011) — types that mathematically converge regardless of write order.
Version vectors and vector clocks — the standard mechanism for
partial-order causality across distributed actors. Git's commit DAG
itself — a working implementation of the same principle for source
code.

The specifically-new piece — AI agents as first-class causal-
divergence participants without out-of-band channels — appears to be
a 2025–2026 observation as agentic coding patterns mature. The
implication for methodology design (the reconciliation point must be
on the natural read path, must carry content state, must tag every
claim with the version it was made against) is documented here as it
surfaces, and will mature as practice does.
