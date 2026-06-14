---
name: fluid-vs-tricky
description: Build order under uncertainty. Two regimes, two strategies. When requirements and design are *fluid* — still being shaped by what you discover building — build the GUI and the foundation simultaneously, in feedback with each other; the GUI surfaces gaps the foundation didn't anticipate, and the foundation forces concrete decisions the GUI was hiding. When the going gets *tricky* — when the layer you're working on has subtle correctness, concurrency, or contract concerns — collapse to bottom-up, build only the foundation, harden it, and defer the GUI until the foundation is solid. The mistake is mixing regimes: doing GUI-and-foundation-parallel while the foundation is tricky burns time on UI work that gets thrown away when the foundation shifts.
version: 0.1.0
---

# Fluid vs Tricky

Software gets built in two regimes. The right development strategy
is different in each, and confusing them costs real time.

## Fluid regime

Requirements aren't settled. You don't yet know the right
abstractions, the right primitives, or which features will turn
out to matter. Each implementation choice teaches you something
about the design. The system is being shaped *by* the act of
building it.

**Strategy: build the GUI and the foundation in parallel.** They
feed each other. The GUI forces you to make concrete decisions
the foundation was hiding (what does the user see when X is
missing? what's the affordance for Y?). The foundation forces
the GUI to commit to a data model (what does this view actually
need, exactly?). Each side surfaces gaps the other side missed.

You will throw work away. That's fine — the throwaway *is* the
learning. The point of the parallel build isn't to ship; it's
to discover what to ship.

## Tricky regime

You hit a layer with subtle correctness concerns. Concurrency.
Contracts that must hold across actors. Cryptographic invariants.
Atomic updates. Or: the new feature steps on an old one in a
non-obvious way and you can't tell from inspection whether the
combined behavior is right.

The act of building no longer informs the design; it forces a
proof. You can't tell if the foundation is correct until you've
thought through every interleaving / every failure mode / every
adversarial input.

**Strategy: collapse to bottom-up.** Build only the foundation.
Harden it. Add tests, write down invariants, prove
correctness as far as you can. Don't build any GUI on top —
the GUI's premise is "the foundation is solid"; if the foundation
isn't yet solid, the GUI is built on a shifting layer.

When the foundation is hardened, *then* return to the GUI.

## The mistake

The expensive mistake is **applying fluid-regime strategy when
the regime has shifted to tricky.** You keep building GUI in
parallel; the foundation keeps changing under it; every iteration
throws away more UI work; the human-facing surface grows ahead
of its load-bearing layer; and one day a real correctness issue
ships because the GUI was telling the user something the
foundation never guaranteed.

The symptom: "we keep redoing the same view." The diagnosis: the
view was built on a layer that wasn't ready. The fix: collapse
to bottom-up, ship the foundation, *then* build the view once.

## How to know which regime you're in

Quick test: ask yourself, "if I had to write down the invariants
this layer must hold, could I?" If yes, you can build on it
fluidly. If no, you're in tricky regime — stop building above
and write down the invariants first. Building above without
knowing them is what shifts the regime against you.

Second test: "if a senior engineer reviewed this commit, would
they ask any factual questions about the layer's behavior, or
would they only ask design-taste questions?" Factual questions
about correctness mean you're tricky. Design-taste questions
mean you're fluid.

## Worked example — 2026-05-18

The Tracker project hit this transition in real time. Early in
the day: fluid regime. A multi-phase rename across nine layers
of the system. We built UI affordances and CLI behavior in
parallel; both sides surfaced gaps; the design solidified.

Then: a real concurrency incident (two Claude Code sessions
sharing one `.git/index` produced a commit whose title described
2% of its diff). The diagnosis named the property that had been
silently violated: ACID Isolation. The remediation required
defense in depth — six layers of structural primitives, several
load-bearing — and the question shifted from "what should this
look like?" to "is this *correct* under every interleaving?"

We had been building Tracker.app (the GUI side) and the CLI
foundation simultaneously. The operator made the explicit call:
*"finish [the CLI piece]. commit. push. Then on to tracker.app."*
That's the regime shift in one sentence. Tracker.app didn't
disappear; it just got deferred until the CLI's correctness was
nailed down — at which point Tracker.app gets built once on a
stable base, not three times on a moving one.

## When to apply

- A new feature is "obvious" — build fluid. Both sides at once.
- A feature touches concurrency, atomicity, persistence, security,
  or a contract that other actors rely on — build tricky.
  Foundation first. GUI later.
- A bug surfaces and the diagnosis names a class of incident
  rather than a single missed case — build tricky. Class-of-incident
  diagnoses are correctness-property diagnoses.
- You've thrown away the same UI flow twice — build tricky.
  The foundation isn't ready; stop polishing the surface.

## When NOT to apply

This skill isn't permission to skip the GUI forever. Bottom-up
work without a UI plan tends to over-engineer the foundation.
The discipline is to *know which regime you're in* — and switch
when the regime changes. Tricky→fluid transition is just as
real as fluid→tricky. After the foundation hardens, return to
parallel build for the next layer up.

## Anti-pattern: "let's just ship the GUI and worry about correctness later"

The trap looks like progress: visible UI motion makes stakeholders
feel things are happening. But every time the foundation shifts
underneath, the UI work has to be redone, and the next foundation
shift gets harder because there's now UI to migrate too. By the
time the correctness issue gets serious attention, the migration
cost dominates the actual fix.

Cheaper to pause UI work as soon as the regime shift is detected.

## Related skills

- `causal-divergence` — names the structural property that often
  causes regime shifts (multiple-actors discovering they disagree).
- `acid-in-tracker` (Track repo) — a worked-example skill where
  the regime shifted on 2026-05-18 and the right call was made.
