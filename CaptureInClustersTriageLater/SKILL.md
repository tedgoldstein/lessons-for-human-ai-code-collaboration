---
name: capture-in-clusters-triage-later
description: Ideas arrive in clusters — after a meeting, during a design conversation, while reading something, while filing one related thing. **Capture them all fast, even imperfectly. Triage in a separate, cooler phase.** The cognitive cost of triaging while ideas are arriving is what loses most of them. The methodology / tracker / TODO list should be cheap enough that filing is friction-free; refinement is a different activity, done with cooler attention.
version: 0.1.0
---

# Capture In Clusters, Triage Later

A common loss pattern: someone has five ideas in two minutes
during a design conversation. They write down one, mean to come
back to the others, lose four. The lost four weren't bad ideas;
they were victims of the moment.

The reason is cognitive. *Capture* and *triage* are different
activities with different attention requirements. Capture is
fast, broad, low-discrimination. Triage is slow, narrow,
discriminating. Doing them at the same time forces a tradeoff:
either you slow capture to triage-quality (and lose the
in-flight ideas), or you let triage suffer to keep up with
capture (and file half-baked items the system then can't
process).

The discipline: **separate the two phases.** When ideas arrive,
capture them all, fast, even imperfectly placed. Refinement
and triage happen later, in a cooler attention mode, when the
backlog is the focus rather than a side-effect.

For this to work, the capture surface has to be cheap enough
to use without ceremony. A bug tracker that demands fifteen
fields per item discourages capture. A markdown file in git
encourages it. The methodology should be a scratchpad, not a
formal filing cabinet — formal filing happens during triage.

## When to invoke

- A conversation, meeting, or design session produces a cluster
  of related ideas.
- A user (or you) says "add in: X" three times in a row.
- You catch yourself thinking *"I should refine this before
  filing"* — and the result is you don't file at all.
- A reviewer or meeting produces five action items; the
  tracker only has time for one.
- Reading documentation, code, or specs surfaces several
  improvements at once.
- You're about to lose track of a thought while triaging
  a related one.
- The methodology is being treated as a perfect filing
  cabinet rather than scratchpad — every entry must be
  pristine before it lands.

## The shape

| Property | Aim for | Why |
|---|---|---|
| **Capture friction** | minimal — one command + a title is enough | Friction at capture time loses ideas |
| **Capture quality** | "good enough to find later" — not pristine | Pristine is a triage-phase concern |
| **Triage cadence** | scheduled, deliberate — daily, weekly | Triage is its own activity, not interrupt-driven |
| **Capture velocity** | matches incoming-idea rate | If capture is the bottleneck, ideas are lost |
| **Cross-linking** | done at triage, not at capture | Capture is local; cross-links require backlog view |
| **Re-triage tolerance** | items that get re-triaged 3+ times are fine — they're being refined | Quality grows by re-touch, not by upfront perfection |

## Why it matters

1. **Most lost ideas are capture-stage losses, not triage-stage
   losses.** People can re-prioritize a captured idea later;
   they can't re-summon a forgotten one.

2. **Refining at capture-time interrupts the flow that's
   producing ideas.** Five ideas arrive in two minutes because
   you're in the right cognitive state. Stopping to triage the
   first kills the state and the four behind it.

3. **The tracker becomes a thinking surface, not just a
   record.** When capture is free, you file thoughts that
   *might* turn into work — and discover during triage which
   ones actually do. The tracker becomes part of how you
   reason about the work, not a downstream archive of it.

4. **AI agents capture at superhuman speed.** A modern AI
   coding agent can file an issue in a few seconds. If the
   methodology requires manual triage during filing, the agent
   becomes a bottleneck. Decoupling capture and triage lets
   the agent be a fast-capture machine, with humans handling
   triage at their own pace.

5. **Cooler attention triages better.** A backlog read on
   Tuesday morning finds duplicates, dependencies, and "this
   is now obsolete" entries the in-the-moment capture couldn't
   see. Triage benefits from distance.

## Practical guidance

- **Make capture a single command.** `radar new "<title>"`.
  `gh issue create -t "<title>"`. `echo "<idea>" >> TODO.md`.
  If filing requires more than one verb + a title, it's
  too heavy for clustered capture.

- **Defer fields.** Title + brief problem statement is
  enough at capture. Files affected, acceptance criteria,
  Verify-required, owners — all of these are triage-phase
  fills.

- **Status `open` is your friend.** Captured-but-not-
  triaged items live at `open`. Triage moves them to
  `analyze` (where the substantive fields get filled) or
  to `closed-wontfix` / `closed-duplicate` (when they
  don't survive triage).

- **Schedule the triage activity.** Weekly backlog
  review, monthly grooming, end-of-day sweep — pick a
  cadence. Triage that "happens when it happens" doesn't
  happen.

- **Cross-link during triage, not during capture.**
  Capture writes one Radar at a time; triage notices
  "this is related to that other Radar" and adds
  `## Reference` entries. Capture-time cross-linking
  is overhead; triage-time cross-linking is signal.

- **Accept that some captures will be deleted.** A
  fraction of captured ideas turn out to be duplicates,
  obsolete, or wrong. That's not waste — it's the cost
  of capture-friction being low. Closing as duplicate
  or wontfix is *fast*; triage is the right place for
  it.

- **Capture isn't a commitment.** Filing an item is
  not promising to do it. It's promising not to lose it
  while you decide whether to do it.

## Common failure modes

- **The pristine-filing trap.** Every captured item has
  to have full acceptance criteria, owners, links,
  estimates. Filing takes ten minutes. People stop
  filing. The backlog looks tidy; the actual idea-loss
  rate is huge.

- **No triage cadence.** Items get captured and never
  triaged. The backlog fills with `open` items and
  becomes "the place ideas go to die." The fix isn't
  more rigid capture — it's a triage rhythm.

- **Triage during capture.** A user lists five ideas;
  the assistant spends thirty seconds on the first
  trying to decide where it belongs, and the other
  four get forgotten. Capture all five with minimal
  effort; sort them out later.

- **The tracker rejects rough captures.** A tool that
  refuses to file a partial item is doing the wrong
  job. Better: file a permissive partial item and tag
  it as `needs-triage` or status `open`.

- **Capture without ever triaging.** The flip side.
  Items pile up; nothing moves to `analyze`. The
  backlog becomes noise. Schedule triage; treat it as
  important work, not chore.

- **Two-phase becomes three-phase becomes seven-phase.**
  Capture → triage → grooming → backlog refinement →
  prioritization → planning poker → sprint commitment.
  Each phase is its own ceremony. The original capture
  is now buried under seven layers of process. Resist.
  Capture + triage is enough; if more structure is
  needed, add it for a specific reason, not by default.

## When to triage at capture time

- **One-off items with no cluster.** A single idea, captured
  alone, can be triaged at the same time without losing
  anything.
- **Urgent items.** "Production is down right now" — capture
  and triage simultaneously, because the priority is already
  known.
- **Items that have a clear, immediate owner.** "This is
  yours, do it next" — no triage phase needed; the routing
  is obvious.
- **High-stakes items.** A regulated-domain incident report
  needs every field filled at capture; you can't defer the
  detail. (But notice this is a *high-stakes* exception, not
  the default.)

## Worked example

During a session refining the Radar methodology, the user
sent three rapid "Add in:" messages while a single Radar
was being filled out:

1. *"Add in: radar.app should be Radar.app"* (a casing
   convention change, mid-Radar)
2. *"Add in: Radar identifigers should begin with the
   letter R as in R12345678"* (a contract-format change,
   minutes later)
3. *"Add in, there should be a search feature the Radar.app
   that finds all Radar names mentioned in a block of
   text..."* (a feature request, minutes after that)

The naive response would have been to debate where each
belonged before filing. Instead: capture each, fast.

- Item 1 went as a small acceptance criterion (`§0`) on the
  in-flight Radar `vr9xsmxk` (the radar.app distribution
  Radar), because casing is naturally bundled with
  distribution.
- Item 2 was filed as its own new Radar `qa6stpkt` (R-prefix
  ids), with full Proposed Resolution + Acceptance criteria
  filled in.
- Item 3 was filed as a third Radar `ykjnb789`
  (paste-and-search), with an explicit dependency on
  `qa6stpkt` recorded in Internal notes.

All three landed in commits within twenty minutes. Triage
happened *during the filling-out*, not before the filing.
The most valuable observation about the cluster — that
filing the R-prefix Radar itself demonstrates *why* the
R-prefix matters — emerged at triage, not at capture. None
of the items were lost. The total capture friction was
roughly one `radar new` per item.

The methodology behaved as a scratchpad: rapid filing, then
refinement. The result is a backlog with three real,
detailed Radars instead of three forgotten conversational
asides.

## Tagline

> Capture is the moment-to-moment work. Triage is the
> cooler-day work. Don't trade one for the other.

If your methodology demands triage-quality at capture,
your methodology is the bottleneck, not the team.

## See also

- [TheContractIsTheArtifact](../TheContractIsTheArtifact/SKILL.md)
  — capture is cheap when the contract is cheap. Filing an
  item should be as cheap as adding a line to a markdown
  file.
- [BootstrapByHand](../BootstrapByHand/SKILL.md) — sometimes
  the fastest capture is editing the file by hand. Don't
  wait for the UI to be ready.
- [OneChangeAtATime](../OneChangeAtATime/SKILL.md) — at
  *triage*, sort the captured cluster into coherent units of
  change. Capture doesn't need this discipline; triage
  does.

## Sources

Inferred from observing rapid-fire idea capture during a
Radar methodology design session (2026-05). The principle
is older — it's the GTD ("Getting Things Done")
distinction between *capture* and *processing*, and it
echoes "inbox zero" patterns in personal productivity.
This skill names the discipline for AI-augmented work,
where capture velocity is no longer the human's bottleneck
and the methodology has to be cheap enough not to be the
bottleneck either.
