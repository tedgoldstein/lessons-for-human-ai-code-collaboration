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
| **[Index](Index/SKILL.md)** | **Routing — when to load which skill.** Read / load first. |
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
| [ReplaceDontRefactor](ReplaceDontRefactor/SKILL.md) | When an approach is structurally wrong, refactoring stalls. Tag the old as recoverable, build the new from a fresh root, co-exist briefly, cut over once, delete the old. Wave-replacement, not gradual transform. |
| [BootstrapByHand](BootstrapByHand/SKILL.md) | When a tool is meant to manage some artifact (issues, schedules, docs) but isn't yet ready to manage itself, fall back to the contract directly. Hand-edit the file. The tool can land later; the work doesn't wait. |
| [PolishWhenLoadBearing](PolishWhenLoadBearing/SKILL.md) | Don't bring infrastructure (CI, signing, notarization, monitoring) to ship-quality before its lack is actually causing problems. Polish when load-bearing; defer until the trigger fires. The timing-counterpart to `FormalVsImprovisational`. |
| [CaptureInClustersTriageLater](CaptureInClustersTriageLater/SKILL.md) | When ideas arrive in clusters, capture them all fast — even imperfectly. Triage in a separate, cooler phase. The methodology should be a scratchpad at capture-time and a filing cabinet at triage-time, not both at once. |
| [PushIsPublication](PushIsPublication/SKILL.md) | Local commits are cheap and reversible. `git push` is publication — triggers CI, makes work visible to other sessions, harder to rewrite. Commit reflexively, push deliberately. Complements (doesn't contradict) `SmallBatchCommitsMergedOften`. |
| [GrammarIsAlsoAPI](GrammarIsAlsoAPI/SKILL.md) | Beyond identifier names, the *grammar* of commands — verb forms, sigils, casing, phrasing — is a UX surface. Self-identifying identifiers (`R<body>`) admit precise regex matching and survive in prose. Permissive in input, canonical in output. |
| [AskExactlyWhenAmbiguous](AskExactlyWhenAmbiguous/SKILL.md) | Under autonomous scope ("work largely unsupervised"), default proceed for reversible local decisions; pause for hard-to-reverse, externally-visible, or under-specified ones. Over-confirming is friction; silent deciding the wrong thing is worse. |

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

There are two intended use modes:

### As a Claude / Claude Code skill set

In a project using Claude Code, point at this library's `Index`
skill at session start (or as a global preamble). The agent will
consult it before acting and load any specific skill whose
triggers match the current situation.

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
