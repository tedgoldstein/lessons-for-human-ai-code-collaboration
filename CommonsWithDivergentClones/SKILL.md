---
name: commons-with-divergent-clones
description: When multiple projects need to share substantial infrastructure but each has its own product identity, model them as a **commons** (the shared kernel, in its own repo) plus **descendant clones** (full git clones that diverge product-side). Periodic one-way merge from commons → descendants; changes never flow back. The commons evolves; descendants specialize. Cheaper than a monorepo when the products are genuinely separate; less brittle than copy-pasting shared code; honest about which direction value flows.
version: 0.1.0
---

# Commons With Divergent Clones

A pattern that comes up whenever multiple products legitimately
share a substrate but have separate product identities:

```
        commons-repo
        /    |    \
   clone1  clone2  clone3
   (diverges product-side)
```

Each clone is a *full git clone* of commons-repo, taken at some
point in history. After the clone, the descendant diverges —
adds its own product code, deletes pieces it doesn't need,
re-skins the user-facing surface. The commons evolves; the
descendant periodically merges from commons → itself.
**Changes never flow descendant → commons.** That direction is
explicitly closed — what makes commons a commons is that it's
project-agnostic.

The pattern is older than git (Plan 9's union mounts, Unix
forks of BSD, the LLVM/Clang split), but git's branching model
makes it cheap. Compared to alternatives:

- **vs. monorepo:** Cheaper governance per product. No
  coordination overhead for releases. Each product can have
  its own CI, deployment, contributor list, security posture.
- **vs. published library:** Heavier than a library would be,
  but the descendant can modify shared code locally if needed
  (and either upstream the change or accept the local
  divergence). Libraries make the published interface the
  contract; commons-with-clones makes the *source* the
  contract.
- **vs. copy-paste:** Honest about the relationship. The
  shared kernel keeps evolving and descendants periodically
  refresh; copies left to rot diverge silently.

## When to invoke

- Two or three products are about to grow that share
  substantial infrastructure (auth, UI primitives, data
  pipelines, a methodology, a domain model).
- You're choosing between "monorepo," "publish a library,"
  and "fork."
- A bug in code-you-share-with-another-team needs fixing in
  five places.
- A piece of methodology / convention / pattern is being
  invented in product-A and you sense product-B will need
  the same thing.
- You're tempted to make the existing product's code
  "general" to support an emerging second product — and
  the changes are starting to feel hostile to product-A's
  reality.

## The shape

| Property | Aim for | Why |
|---|---|---|
| **Commons repo** | one, named for its identity (e.g. `harp`, `radar`) | The commons is its own thing, not a "shared utilities" appendage |
| **Descendant relationship** | git clone, then diverge | History is shared up to the clone point; diff is clear |
| **Merge direction** | commons → descendant only, periodic | Descendants pick up improvements without coordination |
| **Reverse direction** | explicitly closed | What makes commons a commons is project-agnostic content |
| **Descendant identity** | product code, branding, deployment | The descendant is a real product, not "commons configured" |
| **Refresh cadence** | weeks, not days; deliberate | Each merge is a small migration project; don't churn the descendants |

The commons should have its own README, its own LICENSE,
its own contributor guide. Treating it as a first-class repo
(not a "utilities folder") is what keeps it cohesive.

## Why it matters

1. **Shared improvements without shared release pressure.**
   When commons fixes a bug, descendants pick it up at their
   own schedule. They don't have to coordinate; they don't
   have to release together; they don't have to align CI.

2. **Product identities stay clean.** Each descendant has
   its own brand, its own deployment, its own contributors,
   its own roadmap. Monorepo blurs these — descendants
   keep them separate.

3. **The commons stays generic.** Because changes can't flow
   from descendants to commons, the temptation to slip
   product-A-specific code into commons (because "we'll
   abstract it later") is resisted at the toolchain level.
   Anything in commons has to be generic *before* it lands.

4. **Permission to specialize.** A descendant can delete or
   rewrite anything from commons that doesn't fit its
   reality. The cost is "we won't auto-merge that part
   anymore" — acceptable for genuinely product-specific
   divergence.

5. **Discovery via git.** New contributors to a descendant
   can `git log` to see the entire history, including the
   commons history before the fork. There's no "where does
   this function come from?" — git knows.

## Practical guidance

- **Name the commons after what it *is*, not what it
  serves.** "HARP" (the platform), not "shared-libs."
  "Radar" (the methodology), not "common-bug-tracker."
  The commons has its own identity.
- **Document the relationship explicitly.** A `CLAUDE.md`
  or `README.md` in the commons that says "X, Y, Z are
  descendants; merges flow commons → them only; here's
  how to refresh." Make the relationship visible to
  anyone who walks in.
- **Use `git merge`, not cherry-pick or copy-paste.** The
  merge brings history with it; future bisects work.
  Descendants get the full commit log of the commons
  changes they accepted.
- **Refresh deliberately, not continuously.** A descendant
  doesn't need every commons change; pick the merges that
  add value. Weeks-to-months cadence is fine.
- **Resolve conflicts on the descendant side.** When the
  commons changes a file the descendant also changed, the
  conflict belongs to the descendant. The commons doesn't
  know about descendant-specific edits.
- **When a descendant needs something the commons should
  have, *propose it on the commons*** (not "PR back" —
  there's no PR-back direction). The descendant maintainer
  writes a Radar / issue / PR against commons; the
  commons maintainer decides whether it's generic enough.

## Common failure modes

- **"Just make the commons more configurable."** The
  pressure to add `if (project === 'medbook')` branches
  to commons code. Resist. If a behavior is
  product-specific, it belongs in the descendant, not in
  a configuration switch in commons.
- **The dead-cousin problem.** Descendant-A and
  descendant-B both add the same feature independently
  because neither realized the other had done it.
  Mitigation: a public Radar / RFC process on the commons
  for emerging shared concerns. ("Anyone needs an X?
  Speak up.")
- **One-way-merge in name only.** Someone tries to push
  a commit from descendant to commons via cherry-pick.
  Now the commit has two ancestor paths, future merges
  conflict mysteriously, the relationship gets fuzzy.
  Enforce the direction at the human level — and if the
  team can't, use branch-protection rules.
- **The "fork bomb"** — a descendant that diverges so
  fast and so far that merges from commons become
  impossible. Either the descendant should officially
  detach (rename, remove the relationship from the
  README, stop merging) or pull commons more often so
  divergence is bounded.
- **Reflexive monorepo bias.** "Just put everything in
  one repo." Sometimes right, often not. Monorepo works
  when the products release together, share a CI
  pipeline, share contributors. When they don't —
  separate products, separate teams, separate
  deployments — commons-with-clones is more honest about
  the actual decoupling.

## When a monorepo or library is the better choice

- **Monorepo:** Products release together, CI is shared,
  contributors are shared, change in one always touches
  the other. (Most internal company tooling.)
- **Library:** The shared code has a stable, narrow API
  and consumers don't need to modify it. Standard
  package management works; semver is enough. (Most
  third-party dependencies.)
- **Commons-with-clones:** Products are separate
  identities, can release independently, share an
  evolving *substrate* (not just a frozen API), and
  descendants may legitimately need to modify shared
  code locally before any upstream lands.

## Worked example

The HARP project (Health AI Research Platform) is a
commons. Two descendants exist today: **labbook.ai**
(research platform, public-facing at labbook.ai) and
**healthcompass.org** (patient-facing). Both are full git
clones of HARP; both have diverged product-side; both
periodically merge HARP → themselves to pick up shared
infrastructure improvements. The HARP `CLAUDE.md`
documents the relationship explicitly: *"Descendant clone
= git clone, then diverge. Periodic one-way merge from
HARP → children. Changes never flow child → parent."*
Branch naming for the commons/labbook split is
`split/{phase}-{name}`.

The same pattern recurs fractally at the Radar
methodology: the **radar/** repo is the commons; **Medbook**
and **Labbook** are descendant adopters that hold their own
copies of the four canonical files (`SKILL.md`,
`RADAR_TEMPLATE.md`, etc.) plus their own Radars. When
canonical evolves, descendants refresh via
`radar init --force` — which is just `cp` in a trench
coat, but the trench coat enforces the one-way direction.

## Tagline

> Commons evolves; descendants specialize. The direction
> is fixed.

The commons is its own product (a substrate, a methodology,
a shared kernel). Descendants are real products with their
own identities. Merges flow downhill.

## See also

- [TheContractIsTheArtifact](../TheContractIsTheArtifact/SKILL.md)
  — the commons works best when what it ships *is* a
  contract (data shapes, methodology, conventions) rather
  than a heavy runtime. Cheap contracts merge cleanly.
- [DontReinventTheWheel](../DontReinventTheWheel/SKILL.md)
  — if you're building a commons that "all our products
  will need," check first that mature infrastructure
  doesn't already provide it.
- [BoringTechWherePossible](../BoringTechWherePossible/SKILL.md)
  — boring shared substrate (markdown, git, JSON) is the
  best commons material.

## Sources

Inferred from observing HARP (the Health AI Research
Platform commons) and the radar/Medbook/Labbook adoption
pattern (2026-05). The pattern is older: Plan 9's union
mounts, Unix's BSD/SysV/GNU forks, the LLVM/Clang split,
upstream/downstream in Linux distributions. This skill is
the restatement in the small-team / AI-augmented context
where the cheap-contract version of the pattern
(markdown + git) makes it viable for projects much
smaller than an operating system.
