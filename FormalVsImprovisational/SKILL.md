---
name: formal-vs-improvisational
description: Match the level of formality to the stakes, not to temperament — formal at high stakes / low reversibility / many consumers; improvisational at throwaway / single-consumer / cheap-to-redo; in the wide middle, automated guardrails (CI, tests, types, feature flags, idempotency) buy informality at lower risk — the DevOps synthesis. Load when deciding whether to design-first or code-first or test-now-or-later, when about to write a week of design docs for a one-day spike, when shipping a customer-facing service with zero tests "we'll add later", or when a review rejects a research spike for not matching production conventions.
version: 0.1.0
---

# Formal vs Improvisational

Two styles compete in software engineering:

1. **The formal approach** ("Yekke"): design first, strict types,
   comprehensive tests, gated reviews. Treats software as
   high-stakes engineering — bridges, avionics, medical devices.
   Strong at correctness and longevity, slow at delivery.
2. **The improvisational approach** ("move fast and break things"):
   code first, ship the MVP, learn from production, fix bugs as
   they surface. Strong at speed and finding product–market fit,
   weak at long-term maintainability.

Neither is universally right. The principle: **the level of
formality should match the cost of being wrong, not the
developer's (or AI agent's) temperament.**

Where stakes are high or reversibility is low, formalise. Where the
work is exploratory or throwaway, improvise. In the mid-range — the
vast bulk of real software — buy informality at lower risk by
investing in *automated guardrails*: CI, comprehensive tests,
strong types at boundaries, feature flags, idempotency,
automated rollback. That's the DevOps synthesis: **formal
infrastructure, informal behaviour at the application layer.**

## When to invoke

- You're about to write a week of design documents for what is
  plausibly a one-day spike.
- You're about to ship a customer-facing production service with
  zero tests because "we'll add them later."
- An AI agent is generating 200 lines of defensive error handling,
  type annotations, and docstrings for a 20-line throwaway script.
- A prototype quietly became production infrastructure — and still
  has no tests, no validation, no rollback path.
- "Move fast and break things" is being invoked in a regulated
  domain (medical, financial, safety-critical).
- A code review is rejecting a research spike for not following
  the conventions of the production codebase.
- A research spike has spent three weeks on architecture and zero
  on the actual research question.
- Someone's about to add 30 unit tests to a script that will be
  deleted next week.

## The shape — two styles, one axis

| Axis | Formal | Improvisational |
|---|---|---|
| **Design** | Written before code; code conforms. | Discovered through code; design emerges. |
| **Types** | Strict, comprehensive, every signature. | Loose, inferred, dynamic where convenient. |
| **Tests** | Required up front; coverage gated. | Written when something breaks. |
| **Docs** | Required before merge. | Written when someone asks. |
| **Reviews** | Multi-stage, gated, sign-off. | Lightweight, post-hoc, optional. |
| **Strength** | Maintainable, predictable, audit-friendly. | Fast feedback, fast iteration, finds the right product. |
| **Weakness** | Slow; risks building the wrong thing perfectly. | Tech debt; production fires; eventual rewrite. |

The decision isn't *which style to be* but *which style to be
where*. Pick a position per surface:

| Surface | Default formality |
|---|---|
| Database schema, migrations | High — hard to reverse |
| Public API of a library | High — many consumers depend on the shape |
| Production service touching money / health / safety | High — externalities |
| Internal service with one consumer | Medium — guardrails substitute for ceremony |
| Throwaway experiment / spike | Low — improvise |
| Research notebook exploring an idea | Low — improvise |
| One-off migration script | Low formality, but **idempotent** |
| Generated code, scaffolding | Low formality on the inside, sharp at the boundary |

## The DevOps synthesis — guardrails replace ceremony

You don't have to choose. Modern high-performing teams operate
with **formal infrastructure** and **informal application
behaviour**. The guardrails are formal — typed APIs, CI gates,
automated tests, idempotency keys, feature flags, blue/green
deploys, automated rollback. The day-to-day work is rapid —
small PRs, frequent merges, ship today.

The formal *machinery* lets you be improvisational *at the
surface*. Without the machinery, "move fast and break things"
really does break things. With the machinery, you move fast and
the machinery catches what would have broken.

The substitutions:

| Formal practice | Automated substitute |
|---|---|
| "Verify the spec is met" review | Comprehensive test suite |
| "Review every call site" review | A type system at the boundary |
| "QA team signs off" | CI / integration tests |
| "No production change without approval" | Feature flags, default-off |
| "Atomic transactional release protocol" | Idempotent operations, retry-safe |
| "Release window with operations team" | Automated rollback, blue/green |
| "Document every change" | Small PRs with clear titles + git log |

Each row is an exchange: invest once in the automated mechanism,
recoup the time forever after by skipping the manual ceremony.

## Why it matters

1. **Mismatched formality is expensive.** A two-week design
   document for a prototype that should have been a two-day spike
   is two weeks of waste. Two months of unstructured hacking on a
   production billing service is a future incident with real
   money on the line. The cost is symmetric — either direction
   hurts.

2. **AI agents default poorly without calibration.** Without
   guidance, an AI agent will often over-engineer (defensive
   error handling, comprehensive docstrings, exhaustive tests
   for a 20-line script) *or* under-engineer (no tests, no
   types, no validation for a service that will serve 10,000
   users). Both happen frequently. The agent needs to read the
   stakes before choosing.

3. **The guardrails *are* infrastructure.** Treating CI, tests,
   types, flags, and idempotency as infrastructure — not as
   project work that competes with features — is what makes the
   synthesis possible. Skipping the guardrails forces you back
   to a binary choice.

4. **The right answer is local.** Different parts of the same
   system warrant different formality. The migration script:
   high formality (idempotent, tested). The dashboard layout:
   low formality (try it, see if it looks right). Match per
   surface; don't impose a single style globally.

## Practical guidance

- **Read the stakes first.** Who depends on this? What happens
  if it's wrong? How reversible is the change? The answers
  determine where on the formal–improvisational axis you should
  sit. Don't choose by temperament; choose by consequence.
- **Use the guardrails to buy informality.** Good tests → you
  can refactor freely. Feature flags → you can ship behind one
  and validate in production. Idempotency → you can retry. The
  infrastructure earns you flexibility *above* it.
- **Formalise at the boundary, improvise inside.** Public
  function: typed, documented, tested. Internal helpers: name
  them well and move on. Blast radius shrinks rapidly as you
  go inward; spend formality where it pays.
- **Promote, don't refactor-in-place.** When a prototype
  graduates to production, *rewrite it* deliberately as
  production code. Don't try to incrementally formalise a
  script that was never designed for it; the bones are wrong.
- **Match style to phase.** Discovery phase (what should this
  even do?) → improvise. Build phase (how do we deliver it?)
  → formal at the right surfaces. Maintenance phase → formal
  again. The same project at different times wants different
  styles.
- **Don't bring production discipline to a spike.** A research
  spike with commits like `try v3`, `nope`, `actually maybe
  this` is exactly right. Forcing those commits into "production
  shape" before merging defeats the spike.
- **Don't bring spike discipline to production.** Once real
  users depend on it, the bar is the production bar. Stop saying
  "but I built it in a week" and start saying "yes, and we'll
  spend two weeks hardening it."
- **Name the style choice out loud.** When you start work, say
  whether this is a spike or production-grade. It saves a later
  argument about whether the absence of tests is a defect or a
  deliberate choice.

## Common failure modes

- **The "real engineers do it formally" stance.** Treats all
  software as if it were avionics. Produces beautiful designs
  for products that never ship. Wastes prototype budget.
- **The "real engineers move fast" stance.** Treats all
  software as if it were a side project. Ships customer-facing
  code that handles money with no tests. Sleeps badly.
- **The prototype that quietly became production.** Nobody made
  a decision to promote it. The level of formality never
  updated. Two years later it is mission-critical and still has
  zero tests, no types, no docs.
- **The over-formal AI script.** The agent generated 200 lines
  of error handling, type annotations, and docstrings for a
  20-line script that will be deleted in a week. The formality
  did not fit the lifespan.
- **The under-formal AI service.** The agent generated a
  customer-facing endpoint with no input validation, no tests,
  no logging, no rollback path. The improvisation did not fit
  the stakes.
- **Confusing rigour with formality.** You can write rigorous
  *improvisational* code (well-named, single-purpose, tested
  at the points that matter) and sloppy *formal* code (typed
  but unreadable, documented but wrong). Rigour is orthogonal
  to formality; don't substitute one for the other.
- **Mistaking guardrails for ceremony.** Tests are guardrails
  (they catch things). Mandatory three-paragraph commit
  messages are ceremony (they don't). Build the first; delete
  the second.

## When the principle DOES NOT apply

- **Regulated environments (avionics, medical, financial,
  legal).** The formal style is mandated by regulation or
  industry standard. The synthesis still applies internally
  (CI, automated tests, flags), but you cannot skip the
  documentation, design reviews, or audit trails — they are
  evidence required by the regulator, not just engineering
  hygiene.
- **One-person side project with no users.** Improvise
  freely. Nothing depends on it; no one is harmed by a bug;
  you'll rewrite it anyway. This lesson is mostly for shared
  work.
- **Genuinely exploratory research.** A notebook that exists to
  answer "is this even possible?" should not have a
  comprehensive test suite. The answer comes from the
  notebook, not from a hardened library. If the answer is yes,
  *then* you write the hardened version.
- **Crisis hotfixes.** Production is down; improvise. Pay the
  cleanup cost later. The formality is what kept it up most of
  the time; momentary informality is the right trade.

## Tagline

> Formality fits the stakes, not the temperament.

Build formal foundations — CI, tests, types, feature flags,
idempotency — so the work above them can stay informal. Where
the stakes rise, the formality climbs with them. Where the
stakes drop, let it fall.

## See also

- [PolishWhenLoadBearing](../PolishWhenLoadBearing/SKILL.md)
  — the timing counterpart; this skill calibrates *how
  much* ceremony, that one calibrates *when* the polish
  is worth paying for.
- [MultiLevelTesting](../MultiLevelTesting/SKILL.md) —
  testing intensity is one of the main axes of
  formality, and the pyramid is itself a stakes-tiered
  shape.
- [FeatureFlagsAreInfrastructure](../FeatureFlagsAreInfrastructure/SKILL.md)
  — the canonical guardrail that lets the product layer
  stay improvisational at much lower blast radius.
- [BoringTechWherePossible](../BoringTechWherePossible/SKILL.md)
  — boring foundations underneath are what make
  improvisational product work safe in the first place.

## Sources

Distilled from general engineering practice. The "formal vs
rapid" framing follows the Waterfall-vs-Agile and
regulated-vs-startup contrasts. The *Yekke* terminology comes
from the German-Jewish tradition of meticulous formality;
"Move Fast and Break Things" is early Facebook's stated style
(later softened to "Move Fast With Stable Infra"). The DevOps
synthesis traces to the *Phoenix Project* and *Accelerate*
lineage (Kim, Forsgren, Humble). Pairs with
[FeatureFlagsAreInfrastructure](../FeatureFlagsAreInfrastructure/SKILL.md),
[MultiLevelTesting](../MultiLevelTesting/SKILL.md),
[IdempotentByDefault](../IdempotentByDefault/SKILL.md), and
[TimeBoxedExperiments](../TimeBoxedExperiments/SKILL.md).
