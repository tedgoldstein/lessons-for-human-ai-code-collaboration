---
name: tarzan-migration-strategy
description: When migrating a high-risk system, don't cut over all at once. Run old and new in parallel — dual-write or bridge state, route a small canary fraction of traffic to the new system, validate, increment, eventually flip the rest. Never let go of the old vine until you have a firm grip on the new one. Reversibility is the point — a config flip routes traffic back to legacy. The alternative — Big Bang — turns every bug into an outage.
version: 0.1.0
---

# The Tarzan Migration Strategy

A Big Bang migration cuts over all infrastructure or data at once.
Everything goes live at midnight; the old system is shut down. If
anything is wrong, you find out at peak load with no easy retreat.

The Tarzan approach is different: **you never let go of the old
vine until you have a firm grip on the new one.** Old and new
systems run in parallel. Data is dual-written or kept in sync. A
trickle of traffic moves to the new system. If it fails, you
simply stay on the legacy vine while debugging — you don't fall
to the jungle floor. Once the new vine proves it can take your
weight, you swing more traffic over. Eventually, you let go of the
old vine; until then, both bear weight.

The principle: **physical overlap and incremental momentum, with
reversibility as a first-class feature**, not a contingency plan.

## When to invoke

- A production system needs to be replaced, but downtime is
  unacceptable.
- A database migration touches data the application reads in hot
  paths.
- An identity provider, payment processor, or third-party
  integration is being swapped — wrong cutover is customer-visible.
- A team has been told "you have a four-hour maintenance window
  for the cutover" and the change is too big to verify in four
  hours.
- A previous Big-Bang migration left scars — incident review still
  open, leadership wary.
- A monolith is being split into services and the new service
  isn't fully proven.
- An API version (`v1 → v2`) is being deprecated while external
  consumers are still on the old version.
- Renaming a core identifier or type that's referenced in
  hundreds of places — too much to ship as one PR.

## The shape

Tarzan migrations have four moving parts, all at once.

| Part | What it means | Why |
|---|---|---|
| **Both systems live** | Old and new run simultaneously, both reachable. | If new fails, traffic stays on old. No outage. |
| **State bridge** | Dual-write to both, OR one canonical store both systems read from. | A consistent picture of reality, regardless of which system handled the write. |
| **Canary routing** | Start at 1% of traffic to new, increment to 5%, 10%, 50%, 100%. | Real-world validation under graduated load — catches what staging missed. |
| **Reverse switch** | A feature flag, reverse proxy, or DNS toggle that returns traffic to old in seconds. | Rollback is a config change, not an incident. |

If you don't have all four, you don't have a Tarzan migration —
you have a slow Big Bang with extra steps.

## Why it matters

1. **Rollback is trivial.** When something breaks at 2am, you
   flip the routing switch. The new system goes back to inert;
   the old system handles traffic as before. No redeploy, no data
   restoration, no all-hands incident. Pairs with
   [`FeatureFlagsAreInfrastructure`](../FeatureFlagsAreInfrastructure/SKILL.md).

2. **Bugs surface under real conditions.** Synthetic staging
   tests miss real-world traffic shapes — long-tail queries, weird
   user-agent strings, the one customer who hits an endpoint
   1000x/minute. Sending 1% of real traffic to the new system
   finds what staging couldn't.

3. **No "maintenance window" pressure.** Big-Bang migrations
   create deadline panic — everything must be perfect by midnight.
   Tarzan migrations have no single deadline; cutover happens
   when metrics say it's safe.

4. **Stakeholder confidence compounds.** Each percentage
   increment is a successful proof. By the time you're at 50%
   traffic, the organisation trusts the new system. The final
   cutover is a non-event.

5. **The team learns the new system before bet-the-company.** At
   1%, an engineer is debugging a real edge case under low
   stakes. At 100%-only, the same engineer is debugging it under
   outage pressure.

## Practical guidance

- **Decide the bridge first.** How does state stay consistent
  between old and new during the parallel period? Common
  patterns:
  - **Dual-write**: every write goes to both systems. The
    application is responsible for fan-out. Risks: partial-write
    inconsistencies, dual-failure modes.
  - **Single canonical store**: both systems read/write a shared
    database. Risks: schema must be compatible; the database
    becomes the synchronization point — see
    [`SingleSourceOfTruth`](../SingleSourceOfTruth/SKILL.md).
  - **Event-sourced bridge**: a change log replicated to both
    sides asynchronously. Risks: eventual consistency; the new
    system lags the old in subtle ways.

- **Make the routing decision a config flip, not a deploy.** The
  whole point is fast reversibility. If "rollback to old"
  requires shipping a new version, you've lost the property that
  makes Tarzan work.

- **Start at 1%, not 50%.** The first canary cohort should be
  small enough that a total failure is bearable. Scale up only
  after you've watched real metrics — error rate, latency,
  business KPIs — for long enough to trust them.

- **Pick canary cohorts deliberately.** Internal users first,
  then a low-risk geography, then a non-revenue-critical product
  segment, then broader populations. Don't canary by random
  sample if customer impact differs by cohort.

- **Instrument both systems identically.** You can't compare
  apples and pears if old emits log format A and new emits log
  format B. Standardise observability before you start
  migrating, not after.

- **Set termination criteria explicitly.** "When the new system
  handles 100% of traffic for two weeks with no rollbacks, the
  old system is decommissioned." Write this down before you
  start, or the parallel period will outlive everyone's
  patience.

- **Time-box the parallel period.** Open-ended dual-running
  rots — see common failure modes. Pick a target cutover date,
  even if it slips. "Both systems live forever" is not a stable
  state.

- **Document the bridge protocol.** When something inconsistent
  shows up between old and new, the on-call engineer needs to
  know which side is canonical and why. A short note on disk
  beats a Slack-thread reconstruction at 3am.

## Variants by substrate

The pattern adapts to different layers of the stack. The shape is
the same; the bridge mechanism differs.

| Layer | Old vine | New vine | Bridge |
|---|---|---|---|
| **Production service** | Legacy app | New app | Reverse proxy / load balancer with weighted routing; shared DB or dual-write |
| **Database** | Old database | New database | Application dual-writes during parallel; or change-data-capture replicates one to the other |
| **Identifier rename in code** | Code referring to `OldName` | Code referring to `NewName` | Both names exist as aliases for the same data structure; deprecate old after callers migrate |
| **API version** | `v1` endpoints | `v2` endpoints | Both versions live; new clients use v2; old clients still hit v1; shared backend implementation where possible |
| **Library / framework** | Old framework still wired | New framework wired in parallel | Adapter layer translates between framework APIs; migrate per-feature; eventually delete the adapter |
| **Identity provider** | Old IdP | New IdP | Auth layer accepts tokens from both; user records keyed by stable user ID; cohort-migrate users to new IdP |

The code-level variant deserves special attention: when renaming
a core identifier across hundreds of files, you can't sensibly
ship it as one PR. Instead, introduce the new name as an alias
pointing to the same underlying data structure. Both names work.
Migrate callers incrementally. Once no caller uses the old name,
remove it. The shared data structure is the bridge — pairs with
[`TheContractIsTheArtifact`](../TheContractIsTheArtifact/SKILL.md).

## Common failure modes

- **The parallel period that never ends.** "We'll cut over in
  Q3" becomes "next quarter" becomes "after this launch" becomes
  "when we have bandwidth." Two systems run forever; every
  change has to be implemented twice; the bridge becomes
  load-bearing infrastructure. Termination criteria and a
  target date matter.

- **The bridge becomes the system.** What started as "temporary
  dual-write logic" becomes a critical production layer with
  its own bugs, its own owner, its own runbook. The bridge
  outlives what it was bridging.

- **Asymmetric instrumentation.** Old system emits one metric
  shape; new system emits another. You can't tell whether new
  is actually performing better. Migration stalls because the
  comparison is impossible.

- **Drift between old and new.** During the parallel period, a
  feature is added to old but not to new. Or vice versa. When
  cutover happens, the new system is missing functionality.
  Discipline: every change during the parallel period applies
  to both, or is explicitly scoped to one side with a
  documented reason.

- **Canary cohort isn't representative.** The 1% canary is
  internal users on the staging-like network; the other 99% are
  mobile users on flaky connections. New system passes canary,
  fails at 50%. Pick canary cohorts that resemble the broader
  population.

- **Forgetting the reverse switch.** Migration goes well, 100%
  traffic is on new, two weeks pass, old system is
  decommissioned — and then a bug surfaces. Without the reverse
  switch, rollback is now a multi-day project. Keep the old
  system warm until you're truly confident.

- **Dual-write inconsistency.** Application writes to old first,
  then new. Write to old succeeds; write to new fails. The two
  systems are now out of sync — and reads will see different
  things depending on which they hit. Pair with
  [`IdempotentByDefault`](../IdempotentByDefault/SKILL.md) and
  retry, or use a transactional outbox pattern.

- **Treating "100% traffic on new" as completion.** It isn't.
  Decommissioning the old system (stopping its writes, deleting
  its data, removing the bridge code) is the actual completion.
  Until then, you're still in the migration.

## When the principle DOES NOT apply

- **Small / contained changes.** A two-file refactor doesn't
  need a Tarzan migration. The infrastructure cost exceeds the
  risk. Use feature flags, not full system overlap.

- **The substrate is structurally wrong.** When the old system
  is fundamentally the wrong shape — not just outdated, but
  unable to express what the product now needs — running it in
  parallel doesn't save you. You're forcing two-era coexistence
  on a design that can't accommodate the new. Use
  [`ReplaceDontRefactor`](../ReplaceDontRefactor/SKILL.md): tag
  the old, build the new from a clean root, cut over once.

- **One-way / irreversible operations.** Some changes can't be
  rolled back — a destructive schema migration, a one-way data
  transformation, a regulatory cutover with a fixed date. Tarzan
  assumes the reverse switch is real; if it isn't, the strategy
  doesn't apply.

- **The "old vine" is already dead.** Sometimes the legacy
  system is unsupported, broken, or already gone. There's no
  vine to hold while reaching for the new one. Big Bang or
  rebuild from scratch may be the only path.

- **Trivial code-level changes.** A rename that touches 5 files
  in 1 module is a single PR with a search-and-replace. Reserve
  the dual-naming variant for renames where one-shot ripple
  across many modules is too disruptive.

## Worked example: code-level identifier rename

Imagine a codebase with a core concept called `Radar`. Over time,
the team realises the name is wrong — the concept is closer to a
`Track`. The name `Radar` appears in hundreds of files: type
names, method names, variable names, file names, route names,
schema fields, external API parameters.

The Big-Bang approach: a single PR that renames everything.
Risks:

- A hundreds-of-files diff is unreviewable in practice.
- External consumers break the moment the API parameter
  renames.
- Internal callers that depend on the old type via dynamic
  dispatch break silently.
- Conflicts with every other in-flight branch.

The Tarzan approach:

1. **Introduce `Track` as an alias for `Radar`.** Both names
   point to the same data structure. The new name compiles; the
   old name continues to work. External API parameters accept
   both.
2. **Migrate callers incrementally.** Each PR moves one or two
   modules from the old name to the new name. Small PRs, easy
   review — pairs with
   [`SmallBatchCommitsMergedOften`](../SmallBatchCommitsMergedOften/SKILL.md).
3. **Mark the old name deprecated.** Once new callers default
   to `Track`, mark `Radar` with a deprecation warning. New
   code uses `Track`; existing code using `Radar` still works.
4. **Watch deprecation telemetry.** When the API parameter
   `Radar` stops being used in external traffic, the external
   migration is complete.
5. **Delete the alias.** Once no caller (internal or external)
   uses the old name, remove `Radar`. The migration is done.

The shared data structure is the bridge — both names access the
same underlying state. The canary mechanism is the per-PR
incremental migration. The reverse switch is the fact that the
old name keeps working throughout: at any point during the
migration, a PR can be reverted without breaking anything that
depends on the old name.

Server-side migrations look similar but with bigger pieces:
the bridge is a database or message queue rather than a shared
type; the canary is a percentage of traffic rather than a
percentage of call sites; the reverse switch is a routing config
rather than a code revert. The shape is the same.

## Tagline

> Don't let go of the old vine until you have a firm grip on the
> new one.

Physical overlap, incremental momentum, reversibility as a
feature. The alternative is to hope the cutover works.

## See also

- [`FeatureFlagsAreInfrastructure`](../FeatureFlagsAreInfrastructure/SKILL.md)
  — the mechanism that makes the reverse switch trivial. A
  Tarzan migration without flag infrastructure has a fragile
  reverse switch.
- [`ReplaceDontRefactor`](../ReplaceDontRefactor/SKILL.md) — the
  *non-gradual* alternative. When the substrate is wrong rather
  than just outdated, wave-replacement beats Tarzan because
  there's no point running the wrong shape in parallel. Choose
  between Tarzan and wave-replacement based on whether the new
  system is shape-compatible with the old.
- [`SingleSourceOfTruth`](../SingleSourceOfTruth/SKILL.md) —
  during the parallel period, name one authoritative store for
  each piece of state. Old and new read from it; the bridge is
  a derivation, not a duplication.
- [`IdempotentByDefault`](../IdempotentByDefault/SKILL.md) —
  dual-write protocols must be safely retriable, or one side
  will drift out of sync.
- [`OneChangeAtATime`](../OneChangeAtATime/SKILL.md) — within a
  Tarzan migration, each canary increment is one variable
  changing. Don't ship a new feature *and* shift the canary
  percentage in the same window.
- [`TheContractIsTheArtifact`](../TheContractIsTheArtifact/SKILL.md)
  — when the bridge is a shared data shape (database row,
  schema, file format), the contract is what survives the
  migration unchanged. Replace the code that reads it; keep
  the data.

## Sources

The Tarzan metaphor is folk engineering vocabulary: vivid,
common in war-stories, no canonical citation. The closest formal
pattern is Martin Fowler's **Strangler Fig** (2004), which
describes gradually replacing a legacy system by routing
functionality to a new system module by module. Tarzan and
Strangler Fig overlap substantially; the difference is emphasis.
Strangler Fig framing is about *what gets replaced and when*
(one module at a time, the fig vine slowly killing the old
tree). Tarzan framing is about *how to never lose your grip*
(dual-run, canary, reversibility) — the operational and
data-plane discipline that lets the gradual replacement actually
be safe. In practice, the two patterns are often used together:
Strangler Fig for the code structure, Tarzan for the
operational discipline.

The pattern also echoes Joel Spolsky's *"Things You Should Never
Do, Part 1"* warning against rewrites; expand-and-contract
schema migrations; blue-green deployment; canary releases;
LaunchDarkly's percentage rollouts; and the broader
trunk-based-development tradition.
