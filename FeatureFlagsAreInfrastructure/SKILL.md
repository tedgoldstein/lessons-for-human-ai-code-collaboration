---
name: feature-flags-are-infrastructure
description: Treat feature flags as first-class deployment infrastructure, not an ad-hoc `if` here and there — they decouple "what's deployed" from "what's enabled", let big features land in main as inert code, support per-user/cohort/percent rollout, and give one-config rollback. Load when a feature would otherwise live in a long-running branch, when a risky change needs to ship 1% → 10% → 100%, when a change might need fast rollback without a redeploy, when a multi-step rollout needs coordinated DB + code + cleanup, or when a team is debating "should we deploy this Friday?".
version: 0.1.0
---

# Feature Flags Are Infrastructure

Two concepts that look the same and aren't: **deployed** (the
code is live in production) and **enabled** (the code is doing
something users see). Feature flags separate them.

A feature flag is a runtime switch — config, database row,
environment variable, dedicated service — that gates a code
path. The code is in production, compiled, tested, sitting
inert. The flag flips on; the code becomes active. The flag
flips off; the code becomes inert again. Rollback is a config
change, not a redeploy.

The principle: **treat feature flags as infrastructure, not as
scattered `if`s.** Centralise the flag system, name flags
clearly, expire them aggressively, and make flag flips
auditable.

## When to invoke

- A feature would otherwise live in a long-running branch
  (high merge-conflict risk, slow review).
- A risky change needs to ship to 1%, then 10%, then 100% of
  users.
- A multi-step rollout requires coordinated DB migration +
  code deploy + cleanup.
- A change might need fast rollback without a redeploy.
- An A/B test or gradual experiment is being designed.
- A team is debating "should we deploy this Friday?" — flags
  let you separate the deploy from the launch.
- A change has cross-team coordination (frontend ships first,
  backend later) — flags let each ship on its own schedule.

## The shape — flag lifecycle

A flag has a *lifecycle*, not just an existence.

| Phase | What it means |
|---|---|
| **Birth** | Flag introduced in code; defaults to off. Both paths (old + new) coexist. |
| **Internal** | Flag enabled for internal users / dogfooding only. |
| **Gradual** | Flag rolled out by percentage / cohort. Metrics watched. |
| **Full** | Flag at 100%. New behaviour is the behaviour. |
| **Death** | Flag removed from code; old path deleted. Cleanup PR. |

Most flag failures are skipping the *death* phase — flags
accumulate forever, the codebase ages, every read becomes "is
this still relevant?"

## Why it matters

1. **Deploy small + decouple from launch.** A 50-line PR
   that adds a flag-gated path can ship daily. The launch
   ("flip the flag") happens when product is ready, not when
   code is ready. No merge-day pain.

2. **Rollback is a config change.** When something breaks at
   2am, you flip the flag off. No redeploy, no rebuild, no
   pager-duty escalation for the deploy pipeline. Often
   seconds vs hours.

3. **Gradual rollout limits blast radius.** 1% → 10% → 50% →
   100% with metrics watched between steps catches problems
   before they hit everyone.

4. **A/B tests fall out naturally.** A flag with a percentage
   = an experiment. Pair with analytics and you have a
   testing platform.

5. **Long branches become short PRs.** Instead of a 3-month
   feature branch, ship behind a flag in many small PRs.
   Pairs naturally with
   [`SmallBatchCommitsMergedOften`](../SmallBatchCommitsMergedOften/SKILL.md).

## Practical guidance

- **One flag store.** Whatever you use — a config service, a
  database table, a SaaS (LaunchDarkly, Flagsmith, Unleash) —
  pick one. Multiple flag systems in one codebase is a
  drift problem.
- **Name flags by what they gate, not how.** `new-checkout-
  ui` is clearer than `flag-2026-q2`. The reader of the code
  should know what flipping the flag does.
- **Default to off.** New code paths default to disabled.
  Production behaviour stays the production behaviour until
  you intentionally flip.
- **Make flag reads visible.** A central `isEnabled("flag")`
  call site, not scattered `if config.flags.x` reads. Then
  every flag use is greppable.
- **Add an owner and an expiry to every flag.** "This flag
  is owned by Team A and will be removed by Q3." A flag
  without an owner is a flag that lives forever.
- **Audit flag flips.** Who flipped what, when, to what.
  When prod breaks after a flag flip, you need to know.
- **Limit cross-flag dependencies.** "Flag A only works if
  Flag B is on" creates a combinatorial nightmare. Keep
  flags independent.
- **Schedule cleanup.** Quarterly: "what flags are at 100%?
  remove them." The cost of the cleanup is small if
  scheduled, large if deferred indefinitely.
- **Distinguish kill switches from feature flags.** A *kill
  switch* is a permanent operational tool (turn off this
  feature in an emergency). A *feature flag* is a launch
  tool (gates a new feature during rollout). Different
  lifecycles; the kill switch doesn't expire.

## Common failure modes

- **Flag sprawl.** Hundreds of flags, half of them at 100%
  for two years. The codebase has multi-path complexity for
  features that long since became the only path.

- **Implicit coupling.** Flag A only works if Flag B is on,
  and Flag B has a different owner. Discoveries happen at
  3am.

- **No owner.** Two years later: "who knows what this flag
  does?" Nobody. It stays on forever because nobody is
  willing to remove it.

- **Flag flips by anyone, anywhere.** Production behaviour
  changes via Slack message: "I'm flipping foo to true."
  No audit. No revert. No safety.

- **Flags as branching by another name.** Every feature
  forever lives behind a flag. The "deploy / enable" split
  becomes "deploy / commit-to-anything" — you never make
  the launch call.

- **Flag flips that should have been deploys.** A behaviour
  controlled by a flag that nobody intends to ever toggle.
  That's not a flag; it's an `if true` block. Delete it.

- **Untested flag-off path.** The new path is tested; the
  old path bit-rots. When you flip the flag off to rollback,
  the old path is broken. Run both paths in CI until the
  flag is removed.

## When the principle DOES NOT apply

- **Tiny / single-step changes.** A typo fix doesn't need a
  flag. Use them where the *deploy / launch* decoupling is
  genuinely valuable.
- **Changes that can't be flagged.** Schema migrations, ABI
  changes, anything fundamentally one-way. Use other rollout
  techniques (expand / contract, dual-write, etc.).
- **Tiny teams + simple products.** Sometimes the
  infrastructure cost of a flag system exceeds the value. A
  weekend-project app can ship without flags. The threshold
  is around "we care about not breaking users at 2am."

## Tagline

> Deployed ≠ enabled.

Two switches, not one. The deploy is for engineers; the
launch is for users.

## See also

- [SmallBatchCommitsMergedOften](../SmallBatchCommitsMergedOften/SKILL.md)
  — flags are what turn long-lived branches into a
  stream of short, flag-gated PRs that merge to `main`
  without exposing half-finished work.
- [IdempotentByDefault](../IdempotentByDefault/SKILL.md)
  — flipping a flag off under load is a rollback, and
  rollback only works if the surrounding operations
  tolerate retry and replay.
- [FormalVsImprovisational](../FormalVsImprovisational/SKILL.md)
  — flags are the canonical guardrail that buys
  informality: ship improvisationally to 1%, formalise
  before 100%.
- [PostmortemsWithoutBlame](../PostmortemsWithoutBlame/SKILL.md)
  — flags give incidents a one-config-change rollback
  path, which keeps the postmortem about the design
  rather than the panic.

## Sources

Distilled from general engineering practice; echoes the
trunk-based-development tradition, Martin Fowler's
"FeatureToggle" article, and the platform pattern popularised
by LaunchDarkly, Flagsmith, Unleash, and similar tools.
