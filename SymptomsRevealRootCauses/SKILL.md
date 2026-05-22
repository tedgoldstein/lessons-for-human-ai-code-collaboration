---
name: symptoms-reveal-root-causes
description: A small visible symptom (an unexpected prompt, a flaky test, a one-off error, a setting that "just needs to be tweaked") is usually a window into a deeper architectural or environmental cause. The cheap fix patches the symptom; the durable fix addresses the structure that produced it. Treat the first complaint as data, not the problem statement. Fresh-clone / fresh-env is a cheap triage tactic for distinguishing drift-caused symptoms from architecture-caused ones.
version: 0.1.0
---

# Symptoms Reveal Root Causes

A symptom is the part of a problem that crossed your attention threshold. The interesting question is what is underneath. A permission prompt that "shouldn't be there" is rarely about the prompt — it's about the workspace boundary that decided the prompt was needed. A test that's "just flaky" is often a race condition the test correctly noticed. A config setting that "must be wrong" is sometimes the system telling you the abstraction underneath leaks more than you thought.

The fix to the symptom and the fix to the cause are usually very different changes, with very different blast radii. Symptom-fixes accumulate as workarounds that obscure the next investigation. Root-cause fixes pay back across the whole shape of the problem — sometimes eliminating issues you hadn't even filed.

## When to invoke

- A user reports something specific (a prompt, an error, a missing button) and your first thought is "I'll just suppress that."
- You're about to add a retry, a timeout bump, a conditional special-case, or a "just restart it" cron for a "transient" issue.
- A "tiny" bug keeps coming back, or each fix migrates the symptom to a nearby place.
- You've tried 2+ targeted fixes and none stuck.
- A fresh clone / fresh environment / fresh database "magically" fixes the problem (this is a signal, not a victory).
- Something is "weirdly hard" — UI is hard to test, deploys are harder than they should be, onboarding is rough — and the convention is to work around it.
- The codebase has a cluster of `// HACK:` or `// TODO: revisit` comments in one area.
- Two unrelated symptoms turn out to share a fix — a sign the cause is structural and probably has more surface area you haven't seen yet.

## The shape

A symptom and its cause are usually separated by **layers**: surface → behaviour → mechanism → architecture → assumption.

| Layer | Example (an unexpected permission prompt) |
|---|---|
| Surface | "The tool keeps asking permission for a command I've already allowed." |
| Behaviour | The session-level allowlist doesn't persist between sessions. |
| Mechanism | Settings are read from CWD-rooted config, not walked up the directory tree. |
| Architecture | Working trees are placed outside the workspace boundary the tool considers authoritative. |
| Assumption | "A worktree is part of the project" — but the tool's model decided otherwise. |

The surface complaint is often two or three layers above the structure that caused it. Patches at the surface don't address the structure, so the symptom returns somewhere adjacent.

## Why it matters

1. **Symptom-fixes accumulate.** Each workaround is a small piece of complexity added permanently. Three workarounds for the same underlying cause is three pieces of code that future readers must understand and route around.
2. **Symptoms migrate.** If the cause is structural, "fixing" one symptom often pushes the symptom somewhere else — usually more confusingly than the first time.
3. **The cause is where the leverage is.** Fixing the structure tends to eliminate multiple symptoms at once. Problems you didn't even file disappear.
4. **Surface fixes obscure future debugging.** Workarounds look like load-bearing code. The next investigator can't tell which line is "real behaviour" and which is "patch for an issue we never understood."
5. **Confidence compounds.** When you fix root causes, the codebase gets stronger over time. When you fix symptoms, it gets more entangled.

## Practical guidance

- **Treat the first symptom as a data point.** Ask "what is this telling me about the system?" before asking "how do I make it stop?"
- **Trace the symptom upstream.** What invariant did the system check before producing this complaint? Why did the invariant fire? What configures it? Keep walking until you find structure — usually two or three hops up.
- **Fresh-clone / fresh-env as a probe.** Long-lived working trees accumulate drift: stale hooks, orphan branches, leftover state, cached settings, broken worktree refs. When a problem is hard to reproduce, work from a known-good base. A fresh clone often surfaces the real issue — or makes a "scary" problem evaporate, which tells you it was environmental and not architectural. Either way you've learned something specific.
- **2+ failed fixes ⇒ question the architecture.** If two targeted fixes have failed, you're operating at the wrong layer. Stop fixing, start investigating.
- **Verify hypotheses cheaply before committing.** When you suspect the cause is structural, design a small reproducible probe that *would* distinguish your hypothesis from a competing one. Don't refactor based on theory — the cost of a 30-second test is much less than a half-day mistaken redesign.
- **Pair with [OneChangeAtATime](../OneChangeAtATime/SKILL.md).** Investigation is fixing-in-reverse: change one variable at a time so you can localise the cause.
- **Pair with [TheBisectMindset](../TheBisectMindset/SKILL.md).** When the symptom is "this worked yesterday," bisect tells you *when* the cause entered the system; chase upstream from there.
- **Write the cause in the commit message, not just the fix.** Future-you will need to know which structural issue the commit addressed. The diff alone usually doesn't say.

## Common failure modes

- **The retry loop.** Something fails intermittently → add a retry. The retry hides a race or a leak that will eventually consume the system.
- **The "we just restart it hourly" cron.** A symptom-fix dressed up as operational tooling. The leak is still there.
- **The branching workaround.** `if (config.foo === 'legacy') { ... } else { ... }` — three of these in a file means nobody understood what `foo` actually controls. The branches are tombstones for skipped investigations.
- **The bug closed as "couldn't reproduce."** Often the cause is environmental and reveals itself only under specific conditions. Closing teaches the reporter not to report next time.
- **The fresh-clone reflex.** "I deleted my checkout and started over and now it works" — without investigating what was different. The cause is still there, in someone else's checkout, waiting.
- **Mistaking a credible-sounding hypothesis for a verified one.** Two of your three quick guesses about the cause will be wrong. Verify the third before you act on it.

## When the principle DOES NOT apply

- **Time-critical hotfix.** Prod is down. Apply the workaround; file a follow-up to find the cause. Don't let "we should investigate" delay restoring service.
- **Cost-benefit clearly favours the workaround.** Sometimes the investigation cost dwarfs the lifetime cost of the symptom — a one-line config tweak in a system you're about to retire. Acknowledge the trade-off explicitly; don't pretend you fixed the cause.
- **The symptom and the cause are identical.** A typo in a constant. The fix to the symptom *is* the fix to the cause. Don't manufacture depth where there isn't any.

## Tagline

> The bug you see is the bug the system decided to show you. The bug to fix is upstream.

## Sources

Pairs with [OneChangeAtATime](../OneChangeAtATime/SKILL.md), [TheBisectMindset](../TheBisectMindset/SKILL.md), [ReadTheSourceFirst](../ReadTheSourceFirst/SKILL.md), [ReplaceDontRefactor](../ReplaceDontRefactor/SKILL.md). Distilled from the repeated observation that small surprises point to structural mismatch, and that fresh-clone / fresh-environment is a cheap triage tactic for separating drift-caused symptoms from architecture-caused ones.
