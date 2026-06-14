---
name: replace-dont-refactor
description: When an existing approach is structurally wrong — not just buggy or inelegant but built on the wrong substrate — refactoring incrementally toward the right shape is often slower and more brittle than building the right shape from scratch and switching over. Tag the old work as recoverable, build the new in parallel, cut over once, delete the old when nothing depends on it. Wave-based replacement, not gradual transformation.
version: 0.1.0
---

# Replace, Don't Refactor

Most code improvements are refactors: rename a variable, extract a
function, untangle a class hierarchy, change a data shape. The
existing shape is mostly right; you're nudging it.

Sometimes the existing shape is **wrong** — not in detail, but in
substrate. The data model is the wrong abstraction. The
architectural choice has rotted under load. The framework you
bought into has fought you for two years. In those cases, the
refactor will stall because every step toward the right shape
requires touching code that doesn't want to be touched. You spend
weeks rearranging chairs.

The discipline: **tag the old as recoverable, build the new from a
clean root, run them briefly in parallel, cut over once, delete the
old.** A wave-replacement, not a continuous transform.

This is harder than refactoring emotionally — it feels wasteful to
delete working code. It's not. Carrying the wrong substrate forward
is the more expensive choice.

## When to invoke

- The refactor keeps stalling because each step requires re-cutting
  the design.
- "We could refactor X into Y, but we'd have to first refactor Z,
  A, B…" — the chain has no obvious end.
- You've been refactoring the same module / framework / data layer
  for three weeks without converging.
- The codebase has shapes from two design eras coexisting awkwardly,
  with bridge code that nobody understands.
- "Should we just start over?" is being asked unironically.
- The framework / library / pattern you adopted has actively fought
  you on the last three features.
- Tests pass but every change touches files in unrelated parts of
  the system.

## The shape

| Property | Aim for | Why |
|---|---|---|
| **Snapshot of the old** | a git tag with an evocative name | Old work isn't lost; you can `git checkout <tag>` to see it |
| **New work location** | fresh branch from a parallel root (or new directory in the same repo) | The new isn't constrained by the old's shapes |
| **Parallel period** | short — long enough to migrate data, short enough to avoid two-eras confusion | Don't let "transition" become permanent |
| **Cutover** | a single decision moment | "Wave2 is over; Wave3 is canonical" — documented, dated, irrevocable |
| **Deletion of the old** | after cutover, when nothing imports it | The recoverable tag is enough; the working tree is for the current wave |
| **Documentation** | a written note explaining why the wave happened | Future readers (and future you) need to know the reasoning, not just the result |

## Why it matters

1. **The right shape isn't reachable by small steps from the wrong
   shape.** That's what "structurally wrong" means. Refactoring
   moves you within the space of mutations the current shape
   admits; the right shape requires moving to a different space.

2. **Bridge code is forever.** "Just for the transition" code
   that translates between old and new shapes tends to outlive
   the transition by years. It accumulates special cases. It
   becomes load-bearing because *something* depends on it. The
   wave-replacement approach has a single bridge moment, not a
   permanent bridge layer.

3. **The new shape attracts contributors.** A fresh root invites
   people in; a perpetually-refactoring codebase warns them off.
   When you say "Wave3 starts here, here's the new shape,"
   contributors orient themselves to the new shape immediately.

4. **The old shape is preserved as documentation.** A git tag
   on the old wave is a perfectly good museum exhibit. Future
   contributors can `git checkout` it to see exactly what was
   replaced and why. Better than a comment that says "we used
   to do this differently."

5. **Sunk-cost is recognized, not hidden.** Refactoring lets
   sunk-cost fallacy hide as "incremental progress." Wave-
   replacement forces the conversation: *we built the wrong
   thing; we're rebuilding.* That's painful, but it's honest.

## Practical guidance

- **Name the wave.** "Wave2 → Wave3 (HTMLJUNTA)." "v1 → v2."
  A name carries the decision; reviewers know which era a
  file belongs to.
- **Tag, don't branch, the old.** A tag is a frozen snapshot;
  a long-lived branch is a place where people might keep
  working. The point is to fossilize the old, not keep it
  living.
- **New wave starts at a fresh root, not as a transform.**
  Don't `git checkout` the old code and start editing.
  Make a new directory / new file / new branch from main
  and design forward.
- **Migrate data, not code.** The data (the contract — see
  [TheContractIsTheArtifact](../TheContractIsTheArtifact/SKILL.md))
  is the bridge. Old code reads it; new code reads it.
  Replace the readers, not the data.
- **Set a cutover date.** Open-ended parallel periods rot.
  "Wave3 is canonical as of <date>; new work goes there;
  Wave2 is on the HTMLJUNTA tag for archaeology."
- **Delete the old when the new ships.** Don't keep both in
  the working tree. The git tag remembers; the file tree
  shouldn't.
- **Write up the why.** A short README or design doc
  explaining what was wrong with the old wave. Future
  contributors need to know not to reach for the old
  pattern.

## Common failure modes

- **The eternal refactor.** Six months in, you're still
  refactoring. The original code grew faster than the
  refactor. Stop. Tag the current state. Restart from a
  fresh root.
- **Wave-replacement as a euphemism for rewrite.** A
  "rewrite" is a wave-replacement done without discipline —
  no tag, no documented cutover, no preservation of the
  old, no clear new design. The wave-replacement pattern
  is a *disciplined* rewrite: the difference is in the
  ceremony.
- **Two eras coexisting permanently.** You meant to cut
  over but it kept getting pushed out. Both eras now live
  in the codebase, with bridge code keeping them in sync.
  Force the decision: cut over now, or admit the new wave
  failed and re-do.
- **Throwing away the data.** Wave-replacement doesn't
  mean re-deriving every database row. It means
  rebuilding the *code that reads and writes* the data.
  The data should mostly survive a wave. (If it can't,
  you may be looking at a deeper redesign that touches
  the contract — be careful.)
- **Doing it too often.** Every wave costs context loss
  for contributors. A wave once every year or two is
  bearable; one per quarter is chaos. If you're waving
  frequently, the deeper problem is design judgment, not
  refactoring discipline.

## When refactoring is the right call

- **The shape is mostly right; one module is wrong.**
  Refactor that module. Don't burn the whole village to
  re-roof one house.
- **The migration is mechanical.** A find-and-replace
  across 200 files, a codemod, a `tsc --strict` pass —
  these are refactors at machine scale and they work.
- **The team is small and stable.** Big-bang rewrites are
  more expensive for big teams (more re-orientation). A
  one-or-two-person codebase can wave-replace cheaply.
- **The downside risk is contained.** A wave-replacement
  on a critical-path service mid-launch is reckless.
  Refactor in place; wave-replace between launches.

## Worked example

The Track repo went through three waves in three weeks
(2026-04 → 2026-05):

- **Wave1**: a JavaScript CLI + an HTML dashboard built
  against a sample-data file. Proof of concept.
- **Wave2**: a full native SwiftUI macOS app — BrandRow,
  TabStripView, DashboardView in four modes, TrackDetailView
  with stage-timeline spine, etc. Thousands of lines of
  view code, four full design audits, pixel-faithful to a
  design handoff.
- **Wave3 (HTMLJUNTA)**: deposed Wave2 by fiat. The visual
  surface moved from SwiftUI to HTML/JSX rendered in a
  WebKit shell; protocols between layers became
  HTTP-shaped; the native Swift host shrank to what HTTP
  can't reach (PTY-backed subprocess, file-system writes,
  AI-key-bearing threads).

The Wave2 SwiftUI surface — weeks of work — was discarded
in one design decision. It is preserved on the
**`HTMLJUNTA` git tag**: any time anyone wants to see what
Wave2 looked like, `git checkout HTMLJUNTA` does it. The
working tree carries Wave3 only.

The whole pattern was named *junta* — "a sudden seizure of
power by a coordinated group" — because the design
decision wasn't gradual. It was a single moment of saying
"HTML and HTTP are now the governing protocols," followed
by demolition of the deposed surface.

The cost: weeks of SwiftUI work, deleted. The savings:
the new substrate (HTML/JSX + WebKit + the AI-renderer)
matched the actual product instinct, and Wave3's progress
in two weeks exceeded Wave2's in four. Refactoring Wave2
into Wave3 would have been an open-ended slog of view-by-
view re-implementation.

## Tagline

> When the substrate is wrong, replace it. Refactoring
> optimizes the wrong shape.

Tag the old; build the new from a fresh root; cut over
once; delete what's deprecated. A wave is brief and
named; the alternative — perpetual refactor — is long
and shapeless.

## See also

- [FluidVsTricky](../FluidVsTricky/SKILL.md) — the cousin
  discipline of recognising the current strategy is the
  wrong one. Replace-don't-refactor is the move when the
  *substrate* is wrong; FluidVsTricky is the move when the
  *build order* is wrong. Same diagnostic muscle, different
  axis.
- [TheContractIsTheArtifact](../TheContractIsTheArtifact/SKILL.md)
  — wave-replacement is cheap precisely when the contract
  (the data shape) is durable. Replace the code that reads
  the contract; the data survives unchanged.
- [OneChangeAtATime](../OneChangeAtATime/SKILL.md) — within
  a wave, still one change at a time. Wave-replacement is
  the *unit-of-change for the architecture*, not for the
  individual commits.
- [TimeBoxedExperiments](../TimeBoxedExperiments/SKILL.md)
  — sometimes the right way to decide between "refactor
  the current shape" vs "wave-replace" is to time-box a
  spike on the new shape and see if it's actually better.

## Sources

Inferred from observing the Track repo's three-wave
arc (Track/, 2026-04 → 2026-05). The pattern is older:
Joel Spolsky's *"Things You Should Never Do, Part 1"*
warns against rewrites; Martin Fowler's strangler-fig
pattern is the *gradual* alternative; this skill names
the *non-gradual* version with discipline so it's not
just "rewriting." The wave-replacement-with-tag
discipline is what differentiates it from the
undisciplined rewrites Spolsky was warning about.
