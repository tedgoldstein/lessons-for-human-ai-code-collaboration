---
name: polish-when-load-bearing
description: Don't bring infrastructure (CI, signing, notarization, monitoring, release scripts) to ship-quality before its lack is actually causing problems; premature polish costs both the time and the rigidity of a recurring maintenance obligation. Polish when the lack of polish is the limiting factor, not before. Load when about to add CI / signing / monitoring before the thing being shipped has stabilized, when a polish task is taking longer than the work it's meant to enable, when a new contributor proposes heavyweight process for a pre-product project, or when "best practices" is invoked without naming the problem it solves.
version: 0.1.0
---

# Polish When Load-Bearing

A reflex pattern in engineering culture: when a new project
appears, surround it with "best practices" — CI, code signing,
notarization, monitoring, documentation, contributor guidelines,
release scripts, semantic versioning — *before* the project has
its first non-developer user, its first second contributor, or
in some cases its first feature.

These practices aren't wrong. They're solving real problems. The
issue is timing: a practice that's load-bearing in a mature
project is **busy-work pretending to be discipline** in a project
that hasn't yet earned the practice. A team-grade CI on a
one-developer hobby project is theater. A semantically-versioned
0.0.x → 0.0.y release cadence on a pre-user project is
ceremony. Notarization on a build no second person has ever run
is preparation, not progress.

The discipline: **polish what's currently load-bearing, defer
what isn't, and notice the moment the deferred polish becomes
load-bearing.** That moment is the cue, not the calendar.

This skill is the timing-counterpart to
[FormalVsImprovisational](../FormalVsImprovisational/SKILL.md).
That skill says "match formality to stakes." This skill says
"and apply formality *when* it's actually paying for itself,
not before."

## When to invoke

- About to add CI / signing / monitoring / release automation
  to a project that hasn't yet stabilized.
- "Shouldn't we have notarization on this build?" before
  anyone other than the developer has run the build.
- A polish task is taking longer than the actual work it's
  meant to enable.
- An old infrastructure decision (KV key format, branch
  naming, file structure) is "wrong" but nothing currently
  depends on it being right.
- A new contributor proposes a heavyweight process for a
  pre-product project.
- You're solo on a project and considering team-grade
  processes.
- "Best practices" is being invoked without naming the
  problem it solves.

## The shape

| Property | Aim for | Why |
|---|---|---|
| **First polish** | the minimum that makes today's workflow possible | Don't gold-plate before the first lap |
| **Threshold to polish more** | "the lack of this is now blocking real work" | A concrete trigger, not a calendar trigger |
| **Documented imperfections** | a README or Status section that names what's adequate-not-polished | New contributors know what's deferred and why |
| **Cheap interim solutions** | a `just`/`make`/`script` step instead of a full CI pipeline | The job gets done; the infrastructure is one step deep |
| **The next polish target** | the closest "this is starting to hurt" thing | Triage by where the pain is, not where convention says to start |

## Why it matters

1. **Premature polish has dual cost.** The time spent
   building it, plus the recurring time to *maintain* it.
   A CI pipeline isn't a one-time investment; it's a
   permanent maintenance load. Doing it before it's
   load-bearing means paying maintenance on infrastructure
   that wasn't helping anyone.

2. **Polish hardens the design too early.** A release
   script implies a release format; a notarization
   pipeline implies a deployment shape. The earlier you
   commit to those, the harder it is to change direction.
   Projects in early-design churn benefit from leaving
   these choices reversible.

3. **The right polish is recognizable when the pain
   arrives.** "We can't ship to a second user without
   signing" — that moment is unambiguous. "Code signing
   is best practice" — that abstraction is too generic
   to time correctly.

4. **Documented imperfections invite contribution.** A
   README that says "honestly, this is a developer-mode
   workflow; we'll add proper distribution at the v1.0
   ship" is honest. A README that pretends the project
   is production-ready (because it has CI badges and
   notarization!) misleads.

5. **It frees attention.** Engineering attention is
   finite. Time spent on infrastructure-that-doesn't-
   matter-yet is time not spent on the contract /
   product design that does matter. The contract is
   load-bearing; the infrastructure exists to support it,
   not to substitute for it.

## Practical guidance

- **Name the trigger before you build the polish.** "I'll
  add Apple Developer signing when the first non-developer
  needs to run this." Now there's a clear contract — when
  the trigger fires, do the work; until then, don't.
- **Document what's adequate-not-polished.** A README
  Status section. A CLAUDE.md / contributing doc that
  lists known shortcuts. A short "Workaround" section in
  the project's docs. Not hidden shame; visible technical
  decisions.
- **One step deeper than the manual workflow is usually
  enough.** Instead of full CI, a `just test` recipe.
  Instead of notarization, a `right-click → Open` first-
  launch instruction. Instead of Sparkle auto-update, a
  `git pull && rebuild` README line. These are stop-gaps
  that work.
- **Watch for "best practices" without a problem
  statement.** "We should have CI" — for what? "We
  should sign the build" — for whom? If you can't answer
  in one sentence, the practice isn't load-bearing yet.
- **Re-evaluate annually (or per major milestone).** What
  was unpolished but adequate last year may be the
  bottleneck this year. The trigger fires; bring up the
  polish.

## Common failure modes

- **Cargo-cult polish.** Adopting all the trappings of a
  mature project (CI, semver, signing, docs site, contrib
  guide) before the project is mature. The trappings are
  visible; they create the appearance of maturity; they
  don't substitute for it.
- **The "best practices" cudgel.** A reviewer or
  contributor demands all the trappings *now*, citing
  general principles. The right response is "what
  specific problem is this solving today?" If the answer
  is generic ("it's just good practice"), the polish is
  premature.
- **The opposite trap: never polish.** Polish does
  eventually need to land. A pre-product project becomes
  a product, gets users, accumulates contributors — at
  that point, the deferred polish becomes the bottleneck.
  Watch for the trigger.
- **Documented imperfections that aren't.** "We'll polish
  this later" — and "later" never arrives. The trigger
  fired three months ago and nobody noticed. A README
  status section is only as good as the discipline of
  reading it.
- **Polish on the wrong axis.** A project that obsesses
  over its CI pipeline but has no monitoring in
  production. The polish was deployed against the
  developer's pain (build feedback) instead of the
  product's pain (production reliability). Pick the polish
  that addresses the *load-bearing* gap, not the
  closest-to-hand gap.

## When polish *is* load-bearing from day one

- **Anything customer-facing.** A landing page goes to
  real users; it needs to be at customer-grade polish
  immediately or it costs trust.
- **Regulated domains.** Medical, financial, safety-
  critical — the polish isn't optional; it's table
  stakes.
- **Multi-contributor from day one.** The moment two
  people commit, CI becomes load-bearing (it's the
  signal "your push didn't break my work"). Branch
  protection, code review, test pipelines all become
  load-bearing immediately.
- **Open-source libraries with a stated API contract.**
  The contract IS user-facing; semver, signed releases,
  changelogs are load-bearing because consumers will act
  on them.

## Worked example

The Radar.app macOS app was built across three design
waves over a few weeks. At no point in those weeks did
it have:

- Code-signing with an Apple Developer ID
- A `.entitlements` file with hardened-runtime claims
- A notarization pipeline (`xcrun notarytool submit`)
- A DMG packaging script
- Sparkle auto-update integration

It built fine via `xcodebuild → DerivedData` and ran on
the developer's laptop. That was adequate. *No second
user existed.*

The polished-distribution work was filed as Radar
`vr9xsmxk` with full acceptance criteria — Apple Developer
Team setup, entitlements audit, hardened runtime,
notarization pipeline, DMG packaging, Sparkle — but
deliberately deferred. The Radar's own Internal notes
state the priority bluntly: *"Priority: not urgent. The
xcodebuild → DerivedData workflow is adequate for the
people building Radar itself (i.e. one developer). This
becomes urgent the moment a second person wants to use
radar.app without compiling it."*

Same project, several other examples:
- The Medbook `RADAR_DEPLOYS` KV namespace uses bare
  Radar ids (no `R` prefix). The R-prefix migration is
  proposed; the KV migration is deliberately scoped
  *out* for v1.0, because nothing currently depends on
  the keys being prefixed.
- The `examples/medbook/README.md` was outdated
  (described `radar init` as "planned" when it had
  shipped). The fix took two minutes; it landed when
  someone noticed the doc was stale, not when convention
  said "documentation must be in sync."
- The `Radar/vr9xsmxk.md` workaround section names an
  unsigned-DMG path as a *preferred interim* — explicitly
  okay with the Gatekeeper "right-click → Open" first-
  launch ceremony because that's adequate for early
  recipients.

In all of these, the discipline is the same: name the
trigger, defer the polish, document the imperfection,
move on to load-bearing work.

## Tagline

> Polish only what's load-bearing today. The rest is
> busy-work pretending to be discipline.

The moment the deferred polish becomes load-bearing,
land it. Until then, the time goes to the contract / the
product / the design that's actually moving forward.

## See also

- [FormalVsImprovisational](../FormalVsImprovisational/SKILL.md)
  — the level-of-ceremony pair to this skill's timing-of-
  ceremony. Match formality to stakes; apply formality
  when stakes appear.
- [TheContractIsTheArtifact](../TheContractIsTheArtifact/SKILL.md)
  — the contract is what *is* load-bearing in early stages,
  so the polish on the contract matters more than polish on
  the implementations.
- [TimeBoxedExperiments](../TimeBoxedExperiments/SKILL.md)
  — when uncertain whether to polish, time-box the polish
  work. If you can't finish it in the budget, the trigger
  isn't yet load-bearing.

## Sources

Inferred from observing the Radar repo's deliberate
non-polish of distribution infrastructure (vr9xsmxk,
2026-05) while the methodology contract evolved through
three waves. The pattern is older — it's the same
instinct behind YAGNI ("You Aren't Gonna Need It"), MVP
("Minimum Viable Product"), and "premature optimization
is the root of all evil." This skill names the *timing*
discipline specifically: not "don't optimize ever," but
"polish *when* it's load-bearing, not before."
