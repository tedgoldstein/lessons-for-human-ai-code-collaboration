# Lessons

Software-engineering principles captured as Anthropic-format skills.
Each lesson is a small directory with a single `SKILL.md` (YAML
frontmatter + body). The skills are general — they encode patterns
worth remembering across projects, not project-specific guidance.

The intent: when Claude is working on any codebase and hits a
recognizable situation, the relevant skill here surfaces. Lessons are
written from real cases — each one is a principle plus the worked
example that taught it.

## Skills

Start with **[Index](Index/SKILL.md)** — it's the router. It lists
the situations each downstream skill applies to, so loading it
first lets you decide which specific skill the work needs.

| Skill | One-liner |
|---|---|
| **[Index](Index/SKILL.md)** | **Routing — when to load which skill.** Read this first. |
| [DontReinventTheWheel](DontReinventTheWheel/SKILL.md) | Recognize when a custom thing you're building is isomorphic to a mature platform — and map onto the platform instead of rebuilding it. |
| [MultiLevelTesting](MultiLevelTesting/SKILL.md) | Combine many unit tests, fewer integration tests, fewer E2E, and a small set of smoke tests. Each level brings different benefits; together they make a codebase changeable. |
| [SmallBatchCommitsMergedOften](SmallBatchCommitsMergedOften/SKILL.md) | Favour small, short-lived feature branches that land on main quickly. Smaller PRs review better, conflict less, surface bugs earlier, and keep main always shippable. |
| [OrganisingGitPullRequests](OrganisingGitPullRequests/SKILL.md) | Reshape the commits on a feature branch into a coherent story before review. Reviewers can follow the intent; bisect lands on the right commit; revert doesn't drag unrelated files. Covers the Sexton reset-and-recompose pattern + the force-push / squash / valid-at-every-commit debates. |
| [LocalAWSenvironmentUsingLocalstack](LocalAWSenvironmentUsingLocalstack/SKILL.md) | When developing services that talk to AWS, run those AWS services locally via LocalStack rather than mocking, faking, or sharing a dev account. Recipe: a docker-compose service, an init script that creates resources, and a client wrapper that targets the local endpoint or real AWS by config. |
| [OneChangeAtATime](OneChangeAtATime/SKILL.md) | When debugging or shipping, isolate variables. Don't bundle a refactor + dep bump + feature in one commit / PR / deploy. The unit of change should be the unit of investigation. |
| [MakeTheWrongThingHard](MakeTheWrongThingHard/SKILL.md) | Design APIs so incorrect usage is harder than correct usage — types, RAII, typestates, builders, lints. The user shouldn't have to "remember" anything. |
| [NamingIsAPI](NamingIsAPI/SKILL.md) | Names of functions, types, modules, variables are the most-used surface of code. Long descriptive over short clever, specific over generic, intention-revealing. Rename freely. |
| [LogsAreAFeature](LogsAreAFeature/SKILL.md) | Production logging is a deliberate design surface, not `printf` left behind. Structured, correlation-ID'd, levelled consistently, sensitive data redacted at the source. |
| [BoringTechWherePossible](BoringTechWherePossible/SKILL.md) | At the bottom of your stack, choose mature, well-supported tools. Reserve novelty budget for the actual product. |
| [IdempotentByDefault](IdempotentByDefault/SKILL.md) | Design mutating operations to be safe to retry. Either naturally idempotent (PUT-style) or carrying an idempotency key the receiver dedupes against. Retry is not optional in distributed systems. |
| [SingleSourceOfTruth](SingleSourceOfTruth/SKILL.md) | Every piece of state has exactly one authoritative owner. Derive everything else. Duplicates drift; "sync scripts" are evidence of a missing owner. |
| [TimeBoxedExperiments](TimeBoxedExperiments/SKILL.md) | Handle technical uncertainty with a fixed time budget. "I will spend N hours; at the budget I will either have the answer or pivot." The budget itself is the tool. |
| [ReadTheSourceFirst](ReadTheSourceFirst/SKILL.md) | When a dependency does something surprising, read its source before chasing docs / Stack Overflow / maintainers. The source is the only source of truth that's current. |
| [TheBisectMindset](TheBisectMindset/SKILL.md) | When a regression exists between a known-good and known-broken state, `git bisect` finds the introducing commit in log₂(N) steps. Reach for it early; keep the repo bisectable. |
| [PostmortemsWithoutBlame](PostmortemsWithoutBlame/SKILL.md) | After an incident, target the system, not the person. Action items change the system so the same class of failure can't recur. The person was the trigger; the system loaded the gun. |
| [FeatureFlagsAreInfrastructure](FeatureFlagsAreInfrastructure/SKILL.md) | Treat feature flags as first-class infrastructure. Decouple deployed-vs-enabled. Centralised flag store, named clearly, owner + expiry per flag, aggressive cleanup of dead flags. |

## Format

Each skill follows the Anthropic skill format: a directory containing a
`SKILL.md` whose YAML frontmatter declares `name`, `description`, and
`version`. The body is the prose for Claude to read on invocation.

```
Lessons/
├── README.md
└── <SkillName>/
    └── SKILL.md
```

To add a new lesson: make a directory in PascalCase, write `SKILL.md`,
link it from the table above.

## License

MIT — these are general principles meant to be shared.
