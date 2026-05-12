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

| Skill | One-liner |
|---|---|
| [DontReinventTheWheel](DontReinventTheWheel/SKILL.md) | Recognize when a custom thing you're building is isomorphic to a mature platform — and map onto the platform instead of rebuilding it. |
| [MultiLevelTesting](MultiLevelTesting/SKILL.md) | Combine many unit tests, fewer integration tests, fewer E2E, and a small set of smoke tests. Each level brings different benefits; together they make a codebase changeable. |
| [SmallBatchCommitsMergedOften](SmallBatchCommitsMergedOften/SKILL.md) | Favour small, short-lived feature branches that land on main quickly. Smaller PRs review better, conflict less, surface bugs earlier, and keep main always shippable. |

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
