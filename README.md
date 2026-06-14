# Lessons for Human-AI Code Collaboration

A small library of software-engineering principles, formatted so that
both humans and AI agents can load them at the moment they apply.

Each lesson is a directory with a single **`SKILL.md`** — Anthropic
skill format, YAML frontmatter + body. A SKILL.md isn't a blog post;
it's an *operative* document. An AI agent (Claude, Claude Code, an
agentic IDE) can be told *"before you act, check the Index — if a
trigger matches, load that skill and follow it."* The lesson becomes
a precondition of action, not advice the reader may or may not have
read.

This shifts what a "lesson" is. The principles here are mostly
things experienced engineers already know. The value isn't telling
*you*; it's telling the AI agent working alongside you. When the
agent is about to bundle a refactor with a bug fix, the
[`OneChangeAtATime`](OneChangeAtATime/SKILL.md) skill is the
intervention. When it's about to write a parser for structured input
that HTML+DOM already solves,
[`DontReinventTheWheel`](DontReinventTheWheel/SKILL.md) fires. The
skills don't replace human judgement — they share it with the AI
co-author so the co-author isn't operating from a blank slate.

The skills are general. They're principles worth remembering across
projects, not project-specific guidance. Each one is a principle
plus the recognisable trigger plus the worked example that taught it.

## Skills

Start with **[Index](Index/SKILL.md)** — it's the router. It lists
the concrete trigger phrases each downstream skill applies to, so an
agent (or a human picking which one to read) can decide which one
the current situation needs.

| Skill | One-liner |
|---|---|
| **[UsingLessons](UsingLessons/SKILL.md)** | **Bootstrap discipline — before any engineering action, check whether a lesson applies, load it, announce it, follow it.** Load first. |
| **[Index](Index/SKILL.md)** | **Routing catalog — concrete trigger phrases per skill.** Consulted by `UsingLessons` when descriptions aren't surfaced natively. |
| [DontReinventTheWheel](DontReinventTheWheel/SKILL.md) | Recognise when a custom thing you're building is isomorphic to a mature platform — map onto the platform instead of rebuilding it. |
| [MultiLevelTesting](MultiLevelTesting/SKILL.md) | Combine many unit tests, fewer integration tests, fewer E2E, and a small set of smoke tests. Each level brings different benefits; together they make a codebase changeable. |
| [SmallBatchCommitsMergedOften](SmallBatchCommitsMergedOften/SKILL.md) | Favour small, short-lived feature branches that land on main quickly. Smaller PRs review better, conflict less, surface bugs earlier, keep main always shippable. |
| [OrganisingGitPullRequests](OrganisingGitPullRequests/SKILL.md) | Reshape commits into a coherent story before review. Reviewers can follow the intent; bisect lands on the right commit; revert doesn't drag unrelated files. |
| [LocalAWSenvironmentUsingLocalstack](LocalAWSenvironmentUsingLocalstack/SKILL.md) | When developing services that talk to AWS, run those services locally via LocalStack rather than mocking, faking, or sharing a dev account. |
| [OneChangeAtATime](OneChangeAtATime/SKILL.md) | When debugging or shipping, isolate variables. The unit of change should be the unit of investigation. |
| [MakeTheWrongThingHard](MakeTheWrongThingHard/SKILL.md) | Design APIs so incorrect usage is harder than correct usage — types, RAII, typestates, builders, lints. The user shouldn't have to "remember" anything. |
| [NamingIsAPI](NamingIsAPI/SKILL.md) | Names of functions, types, modules, variables are the most-used surface of code. Long descriptive over short clever, specific over generic, intention-revealing. |
| [LogsAreAFeature](LogsAreAFeature/SKILL.md) | Production logging is a deliberate design surface. Structured, correlation-ID'd, levelled consistently, sensitive data redacted at the source. |
| [BoringTechWherePossible](BoringTechWherePossible/SKILL.md) | At the bottom of the stack, choose mature, well-supported tools. Reserve novelty budget for the actual product. |
| [IdempotentByDefault](IdempotentByDefault/SKILL.md) | Design mutating operations to be safe to retry. Either naturally idempotent (PUT-style) or carrying an idempotency key the receiver dedupes against. |
| [SingleSourceOfTruth](SingleSourceOfTruth/SKILL.md) | Every piece of state has exactly one authoritative owner. Derive everything else. |
| [TimeBoxedExperiments](TimeBoxedExperiments/SKILL.md) | Handle uncertainty with a fixed time budget. "I will spend N hours; at the budget I will either have the answer or pivot." |
| [ReadTheSourceFirst](ReadTheSourceFirst/SKILL.md) | When a dependency does something surprising, read its source before chasing docs / Stack Overflow / maintainers. |
| [TheBisectMindset](TheBisectMindset/SKILL.md) | `git bisect` finds the introducing commit of a regression in log₂(N) steps. Reach for it early; keep the repo bisectable. |
| [PostmortemsWithoutBlame](PostmortemsWithoutBlame/SKILL.md) | After an incident, target the system, not the person. Action items change the system so the same class of failure can't recur. |
| [FeatureFlagsAreInfrastructure](FeatureFlagsAreInfrastructure/SKILL.md) | Treat feature flags as first-class infrastructure. Decouple deployed-vs-enabled. Centralised flag store, named clearly, owner + expiry per flag. |
| [FormalVsImprovisational](FormalVsImprovisational/SKILL.md) | Match formality to stakes, not temperament. Formal where stakes are high or reversibility low; improvisational where work is throwaway. In the wide middle, automated guardrails (CI, tests, types, flags, idempotency) buy informality at lower risk — the DevOps synthesis. |
| [TheContractIsTheArtifact](TheContractIsTheArtifact/SKILL.md) | Make the file format / schema / data shape the durable artifact; tools are renderers. Cheap substrates (plain text + git, JSON, markdown) attract humans, scripts, multiple AIs as participants without permission. Tools die; contracts survive. |
| [CommonsWithDivergentClones](CommonsWithDivergentClones/SKILL.md) | Multiple products with shared infrastructure → model the shared kernel as a **commons** in its own repo, with descendant projects as git clones that diverge. Periodic one-way merge from commons → descendants; changes never flow back. |
| [MultiAICollaborationViaGit](MultiAICollaborationViaGit/SKILL.md) | When two or more AI sessions work on the same repo concurrently, git is the coordination protocol. Branch per session, scope rules, never trample uncommitted work, inline LOCAL MEMORY into cloud-agent prompts, expect predictable collisions. |
| [CausalDivergence](CausalDivergence/SKILL.md) | Modern software runs across many simultaneously-valid timelines — branches, replicas, sessions, threads, and parallel AI agents. The design pattern is not to force one absolute reality but to designate one common central artifact where divergence is detectable — a reconciliation point every actor reads. AI-agentic programming makes this the common case. |
| [FluidVsTricky](FluidVsTricky/SKILL.md) | Build-order under uncertainty. *Fluid* regime — design still being shaped by building — build GUI and foundation in parallel, each side surfacing what the other missed. *Tricky* regime — correctness, concurrency, or contract concerns — collapse to bottom-up, foundation first, GUI later. The expensive mistake is keeping the fluid strategy after the regime has shifted to tricky. |
| [ReplaceDontRefactor](ReplaceDontRefactor/SKILL.md) | When an approach is structurally wrong, refactoring stalls. Tag the old as recoverable, build the new from a fresh root, co-exist briefly, cut over once, delete the old. Wave-replacement, not gradual transform. |
| [BootstrapByHand](BootstrapByHand/SKILL.md) | When a tool is meant to manage some artifact (issues, schedules, docs) but isn't yet ready to manage itself, fall back to the contract directly. Hand-edit the file. The tool can land later; the work doesn't wait. |
| [PolishWhenLoadBearing](PolishWhenLoadBearing/SKILL.md) | Don't bring infrastructure (CI, signing, notarization, monitoring) to ship-quality before its lack is actually causing problems. Polish when load-bearing; defer until the trigger fires. The timing-counterpart to `FormalVsImprovisational`. |
| [CaptureInClustersTriageLater](CaptureInClustersTriageLater/SKILL.md) | When ideas arrive in clusters, capture them all fast — even imperfectly. Triage in a separate, cooler phase. The methodology should be a scratchpad at capture-time and a filing cabinet at triage-time, not both at once. |
| [PushIsPublication](PushIsPublication/SKILL.md) | Local commits are cheap and reversible. `git push` is publication — triggers CI, makes work visible to other sessions, harder to rewrite. Commit reflexively, push deliberately. Complements (doesn't contradict) `SmallBatchCommitsMergedOften`. |
| [GrammarIsAlsoAPI](GrammarIsAlsoAPI/SKILL.md) | Beyond identifier names, the *grammar* of commands — verb forms, sigils, casing, phrasing — is a UX surface. Self-identifying identifiers (`R<body>`) admit precise regex matching and survive in prose. Permissive in input, canonical in output. |
| [AskExactlyWhenAmbiguous](AskExactlyWhenAmbiguous/SKILL.md) | Under autonomous scope ("work largely unsupervised"), default proceed for reversible local decisions; pause for hard-to-reverse, externally-visible, or under-specified ones. Over-confirming is friction; silent deciding the wrong thing is worse. |
| [SymptomsRevealRootCauses](SymptomsRevealRootCauses/SKILL.md) | A small symptom (an unexpected prompt, a flaky test, a one-off error) is usually a window into a deeper architectural or environmental cause. Trace upstream — surface → behaviour → mechanism → architecture → assumption — until you find structure. Fresh-clone / fresh-env is a cheap probe for distinguishing drift from architecture. |
| [Sharpen](Sharpen/SKILL.md) | Mid-session reflective pass on the skill library itself. Score the skills that fired this session, sharpen the dull ones in place (tighter triggers, missing failure modes), and scaffold new skills for recurring gaps. The session is perishable evidence — spend it on the library. |
| [ValidateCodexSecurityWithMythOS](ValidateCodexSecurityWithMythOS/SKILL.md) | Codex is still immature on security-context reasoning. For auth, credentials, sandboxing, network access, and permission boundaries, validate Codex conclusions with Claude MythOS until it gets better. |
| [TarzanMigrationStrategy](TarzanMigrationStrategy/SKILL.md) | Migrate a high-risk system by never letting go of the old vine until you have a firm grip on the new one. Run both live, bridge state, route a canary fraction to the new system and increment, keep a reverse switch back to legacy. Reversibility is the point; the alternative — Big Bang — turns every bug into an outage. |
| [CacheThePrefix](CacheThePrefix/SKILL.md) | When expensive setup precedes useful work, don't reach for a snapshot/clone primitive — nearly every layer of the stack already amortizes via **prefix caching** (hash a deterministic prefix, key the cached output by it). Make setup deterministic and prefix-shaped (stable front, variable back) so the substrate's existing cache catches it. |
| [CriticalEventGate](CriticalEventGate/SKILL.md) | A mandatory, recorded, challenge-response pause before crossing from safe preparation into live, costly, credentialed, or irreversible action. Borrows from surgical time-outs, aviation checklists, and military Go/No-Go polls to interrupt autopilot at the boundary. Release Authority is human-only. |
| [FidelityOverScaffolding](FidelityOverScaffolding/SKILL.md) | Scaffolding (fakes, stubs, mocks, fixtures) proves the scaffolding, not the building. Only fidelity — the real runtime, API, OS, peer, storage — proves the structure stands in the earthquake. A passing test on a wrong fake is a load-bearing lie. Run the live path at least once before declaring done. |
| [MasterChildMergeDiscipline](MasterChildMergeDiscipline/SKILL.md) | In the master-child Track pattern, keep the canonical checkout on `main` as a stable inspection point and do all master-branch work in a dedicated worktree. A Phase merge's commit subject is `Track <phaseid>:`, not `Master <id>:`. The scope hook won't catch a wrong-branch commit — discipline is the only barrier. |
| [CommitHygiene](CommitHygiene/SKILL.md) | A commit is a permanent record; a push is a public commitment — two operations, two checklists, two approvals. Stage explicitly (no sweeping in secrets / build artifacts), write messages future-you can learn from, branch + PR rather than commit to `main`, force-push only after asking. |
| [MeasureDontGuess](MeasureDontGuess/SKILL.md) | When something is slow or heavy, capture a real profiler trace before changing code — the hot path is rarely where intuition points. Instrument your own code with named signposts (Apple Instruments / `os_signpost`, tracing spans) so the trace reads as a narrative of your operations. Trace, then touch. |
| [IsolationAndTransactionalDiscipline](IsolationAndTransactionalDiscipline/SKILL.md) | As concurrent actors rise, engineering systems converge on ACID-like coordination — atomicity, consistency, isolation, durability — not as database trivia but as the recurring answer to concurrent mutation pressure. Smart actors corrupt shared mutable state as readily as careless ones; the fix is architecture (worktrees, transactions, durable audit, separated storage responsibilities), not discipline. |
| [LiveSessionManagementForAgenticEngineering](LiveSessionManagementForAgenticEngineering/SKILL.md) | Durable Trackfiles, worktrees, and commits aren't enough for multi-agent work — you also need a live session layer answering "who is working right now?" A Session Manager records live actors, claimed files, PTY bindings (tmux), warm-worker readiness, role authority, and restart reconciliation so human and AI sessions run concurrently without ambiguous authority or silent corruption. |

## Why human-AI specifically

The principles in this library aren't new. *Read the source.*
*Bisect early.* *Don't bundle unrelated changes.* Experienced
engineers internalise these through years of being burned.

The new thing is that an AI agent helping write code does *not* have
years of being burned. It's smart, fast, capable — and it can
cheerfully bundle three unrelated changes into one commit, or
reinvent HTML for a custom layout problem, or skip the bisect and
read 80 commits by hand. The agent doesn't *know* it shouldn't, and
nothing in its training necessarily intervenes at the moment of
action.

A SKILL.md does intervene. Load the Index at session start. The
Index's trigger phrases match the situation. The matching skill
loads. The agent now has the principle in working context at the
moment it would otherwise have done the wrong thing.

That's the value proposition: **principles that operate on the
agent, not just on the human.** The human reading them is a
secondary audience; the AI co-author is the primary one.

## Format

Each skill follows the Anthropic skill format: a directory
containing a `SKILL.md` whose YAML frontmatter declares `name`,
`description`, and `version`. The body is what the agent (or
human) reads on invocation.

```
lessons-for-human-ai-code-collaboration/
├── README.md
├── LICENSE
├── Index/SKILL.md                    ← router; load first
└── <SkillName>/SKILL.md              ← individual lessons
```

`name` in the frontmatter is kebab-case (e.g. `one-change-at-a-time`).
The directory is PascalCase (`OneChangeAtATime`). The directory
holds the human-facing path; the frontmatter holds the AI-facing
name.

## Adding a new lesson

1. Identify the recurring principle. Did it fire in at least two
   separate contexts? Is the trigger recognisable in the moment
   (not just visible in hindsight)? Is the cost of *not* invoking
   it real?
2. Make a PascalCase directory.
3. Write `SKILL.md` following the established shape: frontmatter
   (name / description / version), opening principle, when-to-
   invoke triggers, the shape / mechanism, why it matters,
   practical guidance, common failure modes, when it does NOT
   apply, tagline, sources.
4. Add a `→ Load <SkillName>` section to `Index/SKILL.md` with
   5–10 concrete trigger phrases.
5. Add a one-liner to the table in this README.
6. Commit.

## Using the library

There are three intended use modes. The first is the one that
makes the agent *self-select* the right lesson with no manual
routing; the others are fallbacks.

### How an agent decides which skill to load

A model does not load all skills, and it shouldn't need you to
name one. The Agent Skills mechanism uses **progressive
disclosure**: at session start the harness reads only the
`name` + `description` from each skill's frontmatter — cheap,
a few hundred tokens for the whole library. The model scans
those descriptions and, when one matches the situation in
front of it, loads that skill's full `SKILL.md` on demand.

So **the `description` field is the entire routing surface.**
Every description here is written to carry concrete trigger
keywords ("Load when about to author a parser…", "after an
incident whose only signal was `error`…") precisely so a model
recognises the moment and self-loads.

Description-matching is necessary but not sufficient. A model
won't *check* the descriptions reliably unless something tells
it to check, every task, at a low threshold. That's what
[**UsingLessons**](UsingLessons/SKILL.md) does — it's the
standing discipline that says *before any engineering action,
ask whether a lesson applies; if one plausibly does, load it,
announce which one, follow it.* Adapted from the
`using-superpowers` bootstrap pattern in
[obra/superpowers](https://github.com/obra/superpowers).

Together the two pieces self-route the library:
`UsingLessons` makes the agent *check* on every task; the
trigger-rich descriptions make the check *land* on the right
skill.

### As native Claude Code skills (auto-loading — recommended)

Install the lessons where the harness discovers skills, so
their descriptions are surfaced automatically and the model
self-routes:

```bash
# project-level (team-shared, travels with the repo)
git clone https://github.com/tedgoldstein/lessons-for-human-ai-code-collaboration \
  /tmp/lessons && cp -r /tmp/lessons/*/ .claude/skills/

# or user-level (applies to all your projects)
cp -r /tmp/lessons/*/ ~/.claude/skills/
```

No `CLAUDE.md` preamble required — the agent sees every
description at session start and loads the matching skill when
a trigger fires. This is the "besides loading all or being
told" path: description-matching does the routing.

### As an Index-routed reference (no install)

If you'd rather not install them as skills, clone the library
as a sibling directory and point `CLAUDE.md` at the Index:

```bash
git clone https://github.com/tedgoldstein/lessons-for-human-ai-code-collaboration \
  ../lessons-for-human-ai-code-collaboration
```

Then in `CLAUDE.md`:

> Before any software-engineering action, read
> `../lessons-for-human-ai-code-collaboration/Index/SKILL.md`.
> If any trigger phrase there matches the current situation,
> load the named skill (`../lessons-for-human-ai-code-collaboration/<SkillName>/SKILL.md`)
> and follow its guidance.

The Index is the router; downstream skills load on demand. If
several skills match the situation, load them all — they
compose.

### As a human reference

The lessons read well as prose. Browse the table above, click
into anything that looks relevant. The "When the principle DOES
NOT apply" section in each skill is unusually useful — it tells
you when the heuristic stops being a good idea.

## License

MIT. See [`LICENSE`](LICENSE). These are general principles meant
to be shared.

## Sources

Several lessons restate principles from the 67 Bricks engineering
blog (67bricks.com): `MultiLevelTesting`,
`SmallBatchCommitsMergedOften`, `OrganisingGitPullRequests`, and
`LocalAWSenvironmentUsingLocalstack`. Where a skill derives
substantively from an outside source, the skill's own "Sources"
section names it.

Other skills draw on the wider tradition: Dan McKinley's "Choose
Boring Technology," Yaron Minsky's "make illegal states
unrepresentable," Google's SRE book + the blameless-postmortem
tradition (Allspaw, Etsy), Annie Sexton's *Git Organized*,
Stripe's idempotency-key pattern, the trunk-based-development and
12-factor-app traditions.
