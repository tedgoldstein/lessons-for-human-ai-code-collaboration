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
