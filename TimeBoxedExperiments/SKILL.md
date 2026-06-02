---
name: time-boxed-experiments
description: Handle technical uncertainty with a fixed time budget declared up front — a specific question, a fixed N hours/days, success criteria, and pivot options; when the budget elapses, decide (extend, pivot, or done). Load when about to say "let me just try X and see if it works", when a spike has run for weeks with no decision, when estimating something you don't yet understand, when a proof-of-concept is drifting into a half-finished production system, or when you've been deep in a problem for hours and can't tell if you're close.
version: 0.1.0
---

# Time-Boxed Experiments

Uncertain technical questions — "is this library good?", "will
this approach scale?", "can we make X work?" — eat unbounded
time if you let them. Three weeks in, you don't have an answer
and you don't know whether three more days will help.

The principle: **declare the budget before you start.** "I will
spend two days on this. At the end of two days I will either
have an answer or pivot." The budget itself is the tool; it
forces a decision point that "until I figure it out" doesn't.

## When to invoke

- "Let me just try X and see if it works."
- "I've been exploring this for three weeks."
- A spike that started Monday is still going at the end of
  the sprint.
- A PR is open titled "WIP: figuring out the right approach."
- The user / team / manager asks for an estimate on something
  you don't yet understand.
- A proof-of-concept is dragging into a half-finished
  production system.
- You've been deep in a problem for hours and can't tell if
  you're close or far.
- A new library / framework / tool is under evaluation with
  no decision point.

## The shape

A time-boxed experiment has four parts, declared *up front*:

| Part | Example |
|---|---|
| **Question** | "Can we use SQLite as the metadata store for X workload?" |
| **Budget** | "Two engineering days" |
| **Success criteria** | "I can write a benchmark showing read latency < 5ms at 1M rows, and a sketch of the schema." |
| **Pivot options** | "If no: revisit Postgres; if maybe: extend by one day; if obvious yes: write up and propose adopting." |

When the budget elapses, you stop and make a call. You may decide
to extend (deliberately) by another budget. You may decide the
answer is "no, this won't work, try Y." You may have the answer.
What you don't do is *drift* — the budget elapsing is a forcing
function for a decision.

## Why it matters

1. **Open-ended exploration drifts.** Without a budget, "I'll
   know it when I see it" turns into a month of partial
   answers. With a budget, partial answers are explicit
   inputs to the pivot decision.

2. **Attention is protected.** Three weeks of "investigating"
   isn't three weeks of work; it's three weeks of avoiding
   other work. The budget puts a cap on opportunity cost.

3. **Pivot becomes a decision, not a defeat.** When the
   budget runs out and you didn't find the answer, that's
   data. It tells you the approach is harder than expected.
   Pivoting is the rational next step, not a failure.

4. **Reporting works.** "I have a 2-day spike on X" is
   accountable. "I'm exploring X" is not. Stand-ups and
   status updates can speak about budgets in concrete terms.

5. **Sunk-cost is bounded.** When you've spent two days, the
   sunk cost is small enough to walk away from. When you've
   spent three weeks, you'll be tempted to push through even
   if pushing through is wrong.

## Practical guidance

- **Set the budget before you start.** Not after a week,
  not after a discouraging morning. Before.
- **Write the question down.** "Can we...?" is sharper than
  "Let me see if...". Phrase it so an answer is recognisable
  when it appears.
- **Define success criteria specifically.** Not "it seems
  to work" but "a benchmark, a sketch, a one-pager." Make
  the deliverable visible.
- **Calendar the deadline.** Put the time-box on the
  calendar with a reminder for the decision moment. The
  decision is the *deliverable*.
- **At the deadline, decide.** "Extend by one more day"
  is a valid decision *if made deliberately*. The thing
  to avoid is the implicit "I'll just keep going."
- **Cap extensions.** A spike that's been extended three
  times is no longer a spike — it's the project. Reframe.
- **Document the negative result.** "Tried X for two days,
  hit walls at Y and Z, recommend Postgres" is useful and
  saves the next person.
- **Keep the spike branch.** Don't merge it (it's a spike;
  it's not production-ready). Keep it for reference. If
  you adopt the approach, rebuild cleanly in small commits
  — see [`SmallBatchCommitsMergedOften`](../SmallBatchCommitsMergedOften/SKILL.md).

## Common failure modes

- **No budget set.** The most common failure: implicit
  exploration that drifts.
- **Budget set but ignored at the deadline.** "Just one more
  day" without a deliberate decision. The budget was real
  yesterday; pretend it's real today too.
- **Sunk-cost spiral.** "I've already spent two weeks; I
  can't stop now." The two weeks are gone whether you stop
  or continue. The question is what the *next* week will
  return.
- **Spike becomes production.** The throwaway code you wrote
  to answer "can this work?" was never meant to ship — but
  here it is in main. Rebuild cleanly; the spike was for
  learning.
- **Vague success criteria.** "I'll know if it feels right"
  isn't a criterion. Specify what you'll measure.
- **No pivot option.** "If it doesn't work, then what?"
  prepared in advance turns failure into data instead of
  panic.

## When the principle DOES NOT apply

- **Genuinely fundamental research.** Sometimes the unknown
  is large enough that two days is meaningless. Set a
  larger budget (weeks, a quarter) but *still set one* —
  the discipline is the budget, not the duration.
- **Open-ended creative work.** Writing prose, design,
  product thinking — the rhythm is different. Time-boxing
  can still apply but the deliverable shape is different.
- **When the answer becomes obvious mid-spike.** If you've
  spent 30 minutes and the answer is clearly "yes, this
  works," you're done. The budget was the upper bound, not
  the target.

## Tagline

> Set the clock. Decide when it rings.

A spike is not a state of being. It's a budget with a
deliverable at the end.

## See also

- [ReplaceDontRefactor](../ReplaceDontRefactor/SKILL.md)
  — both are about recognising when to stop pushing on
  the current shape; the spike's clock is one of the
  signals that the shape itself is the problem.
- [PolishWhenLoadBearing](../PolishWhenLoadBearing/SKILL.md)
  — the same timing discipline applied to
  infrastructure investment; the spike decides what's
  worth carrying past the budget.
- [CaptureInClustersTriageLater](../CaptureInClustersTriageLater/SKILL.md)
  — what to do with the ideas a spike surfaces but
  isn't going to pursue; the off-ramp for
  out-of-scope discoveries.
- [FormalVsImprovisational](../FormalVsImprovisational/SKILL.md)
  — formality calibration for a spike versus
  production code; a timeboxed experiment runs at the
  improvisational end on purpose.

## Sources

Distilled from general engineering practice; echoes
agile-spike culture, scientific-method experiment design,
and the broader "sunk-cost fallacy" literature.
