---
name: lessons-index
description: Routing index for the Lessons skill library. Load this when starting any software-engineering task; it lists the situations each downstream skill applies to so you can load the right one. Each entry has concrete trigger phrases — when you see those phrases or symptoms in the work, load the matching specific skill.
version: 0.1.0
---

# Lessons — routing index

This is the **router**, not a teacher. It tells you which sibling
skill to load when a recognisable situation appears. Each entry lists
concrete triggers — phrases, symptoms, or tasks. When you see one,
load the named skill.

> **Note on the bootstrap.** The standing discipline that says
> *check before every engineering action* lives in
> [`UsingLessons/SKILL.md`](../UsingLessons/SKILL.md). This
> Index is the catalog `UsingLessons` consults. Under a native
> `.claude/skills/` install, the harness surfaces every skill's
> description at session start and `UsingLessons` directs the
> agent to match against those; under a cloned-reference
> install where descriptions aren't surfaced, this Index *is*
> what `UsingLessons` matches against.

The triggers are *symptoms* you might notice or *phrases* the user
might say, not categories of file or task. Read them as: "if any of
these is happening, load this skill."

## Routing table

### → Load `DontReinventTheWheel`

When the work involves **building a custom system that mature
infrastructure already provides**.

Triggers:

- You're about to author a parser, renderer, layout engine, query
  language, routing system, schema language, event bus, or
  dependency resolver.
- The user says "I need to parse my YAML / markdown / DSL and
  render it as a UI."
- You catch yourself writing the word *my-own* in a noun phrase
  ending in *engine*, *language*, *format*, *store*, or *protocol*.
- You're considering a custom storage layer for records-with-queries.
- You're considering hand-rolled auth / OAuth / session handling.
- A plan sketches a parallel implementation of something HTML+DOM,
  SQLite, POSIX, ICU, or the language's package manager already
  does.

**The skill teaches:** inventory mature substrates, probe for
isomorphism, isolate the residual (what the substrate genuinely
can't do), map onto the substrate for everything else.

### → Load `MultiLevelTesting`

When the work involves **deciding how to test something**, or
diagnosing a testing strategy that's failing.

Triggers:

- "How should I test this?"
- "Should I mock this dependency?"
- A bug shipped that the existing tests didn't catch.
- A refactor feels risky because the safety net is uneven.
- CI is slow (only E2E tests) or untrustworthy (only mocks).
- A new contributor asks "what's this function supposed to do?"
  and the answer should have been "read its tests."
- Someone proposes writing all tests at one level.
- "We'll add tests later."

**The skill teaches:** the test pyramid (many unit → fewer
integration → fewer E2E → a few smoke), the six benefits of tests
(verify, design feedback, executable spec, usage example,
regression guard, refactor net), and how to pick the right
altitude.

### → Load `SmallBatchCommitsMergedOften`

When the work involves **scoping a feature branch / PR**, or
diagnosing slow / risky merges.

Triggers:

- "Should I roll this into the same PR or open a new one?"
- A PR description starts to need "also" or "and."
- A diff has crossed 500 lines and is still climbing.
- A branch has been alive longer than a couple of days.
- Someone is blocked waiting on review before they can keep working.
- A merge conflict is harder to resolve than the original work was
  to write.
- The team has the "ice-cream cone" anti-shape: long-lived branches
  and end-of-sprint merge weeks.
- The user catches themself writing "WIP — clean up before
  merging" on a six-day branch.

**The skill teaches:** small + short-lived branches, branch from
tip-of-main, rebase often, stacked PRs for dependent work, the
five reasons it matters, and when a larger branch is genuinely
the right call.

### → Load `OrganisingGitPullRequests`

When the work involves **the commit shape of an existing branch
before it gets reviewed**, or when poor history bites somewhere
downstream (bisect, revert, code review).

Triggers:

- A branch has commits like `wip / wip2 / fix / actually fix / fixes
  the fixes`.
- A revert on `main` dragged in unrelated files and broke things.
- `git bisect` lands on a commit named `wip` — useless.
- A reviewer asks "why is file X in this PR?" and you have to dig.
- You tried an approach, abandoned it, and the diff still has
  vestiges.
- The team is debating force-push, squash-on-merge, or "should
  every commit pass tests."
- A PR is about to open with the current commit log and you know
  it's not the shape you want reviewed.

**The skill teaches:** the Sexton reset-and-recompose pattern,
when to use `--amend` vs interactive rebase vs new-branch-+-
cherry-pick vs `git absorb`, and how to think about the three
recurring debates (force-push, squash, valid-at-every-commit).

### → Load `CommitHygiene`

When the work involves **making a commit or a push** — the
two everyday acts whose discipline determines whether the
history is usable later.

Triggers:

- About to `git add -A` or `git commit -am "stuff"`.
- About to commit directly to `main` because "it's just a
  small fix."
- A commit message would be `wip`, `fix`, `update`, `done`,
  or `stuff`.
- A diff includes `.env`, build artifacts, `node_modules/`,
  IDE settings, or stray `console.log` debug output.
- About to push without anyone (human or otherwise) having
  seen the diff.
- About to force-push to overwrite a prior push.
- An AI agent is about to commit + push autonomously.
- `git log --oneline` shows commits whose intent isn't
  recoverable from the message.

**The skill teaches:** commit is permanent; push is public.
Stage explicitly (named paths, not sweeping flags). Write
the message so future-you can read `git log` and learn
something. Use a feature branch + PR rather than committing
to `main`. Treat commit and push as two separate approvals.
Pairs with `PushIsPublication` (cadence),
`OneChangeAtATime` (scope per commit), and
`OrganisingGitPullRequests` (reshaping before review).

### → Load `LocalAWSenvironmentUsingLocalstack`

When the work involves **a service that depends on AWS** and the
question of how it should run in development or test.

Triggers:

- A service depends on S3, SQS, DynamoDB, Lambda, Kinesis,
  Secrets Manager, IAM, SNS, or similar.
- Onboarding doc says "ask someone for AWS credentials" or
  "you'll need to wait for the staging account."
- Integration tests get skipped on laptops because they need real
  AWS.
- Someone is about to write a mock for an AWS client.
- A new dev cannot run the application locally because of AWS
  dependencies.
- CI doesn't run AWS-touching tests because the build agents
  don't have credentials.

**The skill teaches:** a four-piece recipe — a `docker-compose`
service for LocalStack, an init script that creates resources,
two structurally-identical config profiles (local vs. prod
differing only in the `endpoint` field), and a client builder
with one `if` that branches on endpoint-present-or-absent.

### → Load `OneChangeAtATime`

When the work involves **debugging a regression**, **shipping a
fix**, or **bundling unrelated changes**.

Triggers:

- A regression appeared mid-week and you can't tell which of
  several recent changes caused it.
- You're about to commit a bug fix and notice you also did "a
  small cleanup along the way."
- A dep bump and a refactor and a feature in the same PR.
- "It works on my machine" + many simultaneous environment
  differences.
- A failing test starts passing after a multi-change diff and
  you don't isolate which part fixed it.

**The skill teaches:** the unit of change should be the unit of
investigation — bisect, revert, and review only work cleanly when
each commit / PR / deploy is a single intent.

### → Load `MakeTheWrongThingHard`

When the work involves **designing an API, module, or call site**
that other code (or future-you) will use.

Triggers:

- A reviewer is writing comments like "remember to call
  `close()`" or "don't forget to initialise before use."
- A bug came from passing a string where a different kind of
  string was meant ("wrong ID type").
- README has "you must call X before Y" warnings.
- A function takes `(bool, bool, bool, bool)` and people swap
  them.
- `null` snuck somewhere and the test suite didn't catch it.

**The skill teaches:** shift the cost of correctness to compile
time — newtypes, sum types, RAII / scope guards, typestates,
builders, lints. The user shouldn't have to *remember* anything.

### → Load `NamingIsAPI`

When the work involves **naming functions, types, modules, or
variables** — or fixing a confusing existing name.

Triggers:

- You're committing code with a function named `process`,
  `handle`, `data`, `result`, `helper`, or `tmp`.
- A comment is explaining what a variable is — the name should
  have done that.
- A reviewer asks "what does this return?" when the name should
  have answered.
- Two functions named `getUser` and `fetchUser` — you have to
  read the bodies to learn the difference.

**The skill teaches:** names are the most-used surface of code —
specific over generic, intention over implementation, full
words over abbreviations, long descriptive over short clever.
Rename freely; modern IDEs make it cheap.

### → Load `LogsAreAFeature`

When the work involves **production logging** — either designing
it for a new service or diagnosing why an incident gave no
signal.

Triggers:

- A production incident just happened and the only signal is
  `error` with no context.
- You can't tell *which user* or *which request* a log line
  belongs to.
- Logs are full of `console.log("here")` from someone's
  debugging session.
- Sensitive data is being logged.
- "We'll add logging later."

**The skill teaches:** logs are the production debugger. Design
them deliberately — structured (JSON / key-value), correlation
IDs threaded, levels used consistently, module-boundary
placement, redaction at the source.

### → Load `BoringTechWherePossible`

When the work involves **choosing a foundational tool** — a
database, a queue, a language, a framework, an OS.

Triggers:

- About to adopt a 2-month-old framework as the foundation of a
  new service.
- The plan calls for a new datastore where Postgres / SQLite
  would do.
- A teammate says "let's rewrite in $LANGUAGE_OF_THE_MONTH."
- Onboarding requires learning three custom tools.
- A production incident traces to an obscure dep with no Stack
  Overflow hits.
- Hiring is hard because nobody on the market has used your
  stack.

**The skill teaches:** at the bottom of the stack, mature beats
new. Reserve novelty for the *product* layer. Boring tech has
manuals, hires, war stories, and bounded surprise.

### → Load `IdempotentByDefault`

When the work involves **any mutating operation** in a network /
queue / job context — or diagnosing a duplicate-side-effect bug.

Triggers:

- The word "retry" appears anywhere in the design.
- A POST creates a duplicate row when the client retries after
  timeout.
- A queue consumer is double-processing.
- A scheduled job catches up after an outage and re-runs items.
- A user double-clicks "Pay" and gets two charges.
- A migration script can't be re-run safely.

**The skill teaches:** retry isn't optional in distributed
systems. Make operations naturally idempotent (PUT-style) or
carry an idempotency key the receiver dedupes against. Three
flavors: natural, key-based, compensable.

### → Load `SingleSourceOfTruth`

When the work involves **state that lives in multiple places** —
or diagnosing drift.

Triggers:

- The same constant defined in two files.
- A user / customer / product record in three datastores.
- A config value set in `.env`, Terraform, CI secrets, AND a
  ConfigMap.
- A bug appears as "data is wrong in the UI" but right in the API.
- A "sync script" exists to keep two systems in agreement.
- A duplicate has drifted and you can't tell which copy is right.

**The skill teaches:** name exactly one authoritative owner for
each piece of state. Derive everything else. Caches are
explicit copies with named refresh; replication needs a
conflict policy.

### → Load `TimeBoxedExperiments`

When the work involves **technical uncertainty** that doesn't
have a natural endpoint.

Triggers:

- "Let me just try X and see if it works."
- A spike has been going for weeks with no decision.
- A PR titled "WIP: figuring out the right approach."
- Estimating something you don't yet understand.
- A proof-of-concept dragging into a half-finished production
  system.
- You've been deep in a problem for hours and can't tell if
  you're close.

**The skill teaches:** declare a budget before exploring (a
specific question, a fixed time, success criteria, pivot
options). When the budget elapses, decide — extend, pivot, or
done. The budget itself is the tool.

### → Load `ReadTheSourceFirst`

When the work involves **a dependency behaving unexpectedly** —
before reaching for Stack Overflow / docs / maintainers.

Triggers:

- A library does something the docs don't explain.
- A flag's documented behavior doesn't match what you observe.
- An error message comes from inside a dependency.
- A method "sometimes returns null" with no documented reason.
- Top Stack Overflow answer is from 2017; library is on v3.
- A GitHub issue is unresolved for years.

**The skill teaches:** the source is the only source of truth
guaranteed to be current. A focused 30-minute read often beats
hours of searching. Use IDE "Go to Definition," locate the
entry point, follow only the path that matters, capture the
finding.

### → Load `TheBisectMindset`

When the work involves **finding the introducing commit of a
regression** — or maintaining a repo so bisect works.

Triggers:

- A regression appeared between a known-good version (last
  release, last week) and now.
- A test that was green is suddenly red.
- "I think it broke somewhere this sprint…"
- A performance regression appeared without anyone noticing.
- You're tempted to read 50 commits manually.

**The skill teaches:** `git bisect` finds the introducing
commit in log₂(N) steps. Use `git bisect run` with an
automated check-script. The second half: keep the repo
bisectable — every commit compiles, tests run, commits are
single-intent.

### → Load `PostmortemsWithoutBlame`

When the work involves **the response to an incident** — either
running a postmortem or sitting in one.

Triggers:

- A production incident just happened.
- Someone wants to "find out who did this."
- A previous incident has recurred — the prior postmortem
  didn't produce systemic fixes.
- A near-miss happened.
- The room's energy is *who* rather than *what.*

**The skill teaches:** target the system, not the person. The
person was the trigger; the system loaded the gun. Action
items go to teams, are dated, and change the system. Use the
word "system" relentlessly; document broadly; follow up on
actions.

### → Load `FeatureFlagsAreInfrastructure`

When the work involves **decoupling deploy from launch**,
gradual rollouts, A/B tests, or rapid rollback.

Triggers:

- A feature would otherwise live in a long-running branch.
- A risky change needs to ship to 1% → 10% → 100%.
- A multi-step rollout requires coordinated DB + code + cleanup.
- A change might need fast rollback without a redeploy.
- A team is debating "should we deploy this Friday?"
- A change has cross-team coordination (frontend ships first,
  backend later).

**The skill teaches:** deployed ≠ enabled. Centralise a flag
system; name flags by what they gate; default off; rolled out
by percentage / cohort. Track lifecycle (birth → internal →
gradual → full → death). Aggressive cleanup of dead flags.

### → Load `FormalVsImprovisational`

When the work involves **choosing how much ceremony to apply** —
deciding whether to design-first or code-first, whether to write
tests now or later, whether to type things strictly or
loosely — or diagnosing a mismatch between the style and the
stakes.

Triggers:

- About to write a week of design docs for what's plausibly a
  one-day spike.
- About to ship a customer-facing production service with zero
  tests because "we'll add them later."
- An AI agent is generating 200 lines of defensive error
  handling and docstrings for a 20-line throwaway script.
- A prototype quietly became production infrastructure and
  still has no tests / types / rollback path.
- "Move fast and break things" being invoked in a regulated
  domain (medical, financial, safety-critical).
- A code review is rejecting a research spike for not matching
  production conventions.
- A research spike has spent three weeks on architecture and
  zero on the actual question.

**The skill teaches:** match formality to the stakes, not the
temperament. Formal at high stakes / low reversibility / many
consumers; improvisational at throwaway / single-consumer /
cheap-to-redo. In the wide middle, use automated guardrails
(CI, tests, types, feature flags, idempotency) to buy
informality at lower risk — the DevOps synthesis.

### → Load `TheContractIsTheArtifact`

When the work involves **designing a system that has both data
and tooling**, or choosing where the canonical state should
live, or noticing that a workflow is brittle because it depends
on a single tool being available.

Triggers:

- Designing a new system that has both persistent state and a
  UI / app / CLI.
- You're tempted to put the canonical data inside the app's
  memory or a private database with the UI as the only access
  surface.
- A migration is required because the storage format is locked
  to one tool.
- An AI agent or script needs to participate in a workflow,
  but the only documented surface is a clickable UI.
- The roadmap depends on one specific app being polished before
  users can engage with the system.
- "I want to file a bug, but the bug tracker is broken" —
  the contract should let you file the bug anyway.
- You catch yourself writing a custom binary format for
  something that could be text.

**The skill teaches:** make the file format / schema / data
shape the durable artifact; tools become renderers and
mutators of a contract that outlives them. Cheap substrates
(plain text + git, JSON, markdown) attract participants —
humans, multiple AIs, scripts, future tools — without
permission.

### → Load `CommonsWithDivergentClones`

When the work involves **multiple projects that share
substantial infrastructure** but have separate product
identities, or choosing between monorepo / library / fork.

Triggers:

- Two or three products about to grow that share auth, UI
  primitives, data pipelines, a methodology, or a domain
  model.
- "Should we monorepo this?"
- A bug in code-you-share-with-another-team needs fixing in
  five places.
- An emerging pattern in product-A that product-B will
  clearly also need.
- You're tempted to make existing product code "general" to
  support a second product and the changes are starting to
  feel hostile to product-A's reality.
- A copy-pasted shared module has drifted between products
  and nobody knows which copy is right.

**The skill teaches:** model the shared kernel as a
**commons** in its own repo with its own identity. Other
products are git clones that diverged. Periodic one-way
merge from commons → descendants; changes never flow back.
Cheaper than monorepo when the products are genuinely
separate, more honest than copy-paste, more flexible than
a published library.

### → Load `MultiAICollaborationViaGit`

When the work involves **two or more AI coding sessions on the
same repository concurrently**, or one AI session running
while a human works in parallel.

Triggers:

- About to run a second Claude Code session on a repo while
  the first is mid-task.
- Setting up a `/schedule`d cloud routine on the same repo as
  your laptop session.
- A `git stash` is tempting in a multi-session repo.
- You notice uncommitted changes in the working tree you
  didn't make.
- Two sessions appear to be designing the same feature with
  different names.
- Writing a cloud-routine prompt and realizing the agent
  can't see your `~/.claude/memory/` rules.
- A cloud routine just committed something that contradicts
  a rule the laptop session has internalized.

**The skill teaches:** the git tree is the coordination
protocol — sessions share files, not state. One branch per
session, explicit scope rules, never trample uncommitted
work, inline LOCAL MEMORY rules in cloud prompts, expect
predictable collisions when sessions independently reach
for the same concept, document collisions when they happen.

### → Load `CausalDivergence`

When the work involves **multiple actors operating on shared
state with possibly different views of it** — most commonly
AI agents, but also branches, replicas, threads, and the
parallel programmers around them.

Triggers:

- About to spawn a second AI agent (parallel session, cloud
  routine, headless verifier) against a shared repo.
- Two sessions report different facts about the same ticket,
  Radar, record, or file.
- About to claim work is "fixed" or "done" while the change
  lives only on a branch / worktree / replica that not every
  consumer can see.
- A cold reviewer (human or AI agent) arrives and there's no
  field in the artifact saying which version the prior
  verdicts were cast against.
- A merge surfaces work you didn't know existed.
- Distributed replicas of the same record disagree.
- The user says "I thought this was fixed — why is your
  session showing it as not fixed?"
- Designing a system multiple actors will read and write
  concurrently.
- Tempted to fix concurrent-actor pain by "merge fast" or
  "one writer at a time" rather than by making divergence
  legible.

**The skill teaches:** modern software runs in many
simultaneously-valid timelines (branches, replicas, sessions,
threads, agents), and AI-agentic programming makes divergence
the common case rather than the edge case. The design
pattern is one *common central artifact where divergence is
detectable* — a reconciliation point every actor reads.
For local methodologies the artifact is usually a contract
file (a Radar markdown, a ticket); for distributed systems
it's a registry, vector clock, or consensus log. Carry
content state alongside workflow state, tag every claim with
the version it was made against, push truth onto disk
because AI agents lack out-of-band channels.

### → Load `ReplaceDontRefactor`

When the work involves **an existing approach that's
structurally wrong** — not just buggy, but built on the wrong
substrate — and refactoring incrementally toward the right
shape is stalling.

Triggers:

- The refactor keeps stalling because each step requires
  re-cutting the design.
- "We could refactor X into Y, but we'd have to first refactor
  Z, A, B..."
- You've been refactoring the same module for three weeks
  without converging.
- The codebase has shapes from two design eras coexisting
  awkwardly, with bridge code nobody understands.
- "Should we just start over?" is being asked seriously.
- The framework / library / pattern you adopted has actively
  fought you on the last three features.

**The skill teaches:** tag the old as recoverable (a git tag
with an evocative name), build the new from a clean root,
co-exist briefly, cut over once on a documented date, delete
the old. Wave-replacement, not gradual transformation. The
data (contract) survives; the code that reads it is replaced.

### → Load `BootstrapByHand`

When the work involves **a tool that's supposed to manage
some artifact, but the tool isn't yet ready to manage
itself** — a self-referential bootstrap moment.

Triggers:

- The new tool isn't ready, and you need to use the system
  now.
- A bug in the tool prevents using the tool on itself.
- An AI agent or script needs to participate, but the only
  documented surface is the (incomplete) tool.
- You're tempted to wait for the tool to be polished before
  filing the issue that's blocking the polish.
- A meta-bug needs to be filed about the bug-tracker that
  files bugs.

**The skill teaches:** fall back to the contract directly.
Edit the file by hand using whatever cheap substrate
(markdown, JSON, plain text) the contract uses. The tool
can land later; the work doesn't wait. Works only if the
contract is hand-editable — pairs with
`TheContractIsTheArtifact`.

### → Load `PolishWhenLoadBearing`

When the work involves **deciding whether to bring
infrastructure up to ship-quality** — CI, signing,
notarization, monitoring, release scripts — before the
project has clearly earned the polish.

Triggers:

- About to add CI / signing / monitoring before the thing
  being shipped has stabilized.
- A polish task is taking longer than the actual work
  it's meant to enable.
- An old infrastructure decision is "wrong" but nothing
  currently depends on it being right.
- A new contributor proposes heavyweight process for a
  pre-product project.
- You're solo on a project and considering team-grade
  processes.
- "Best practices" is being invoked without naming the
  problem it solves.

**The skill teaches:** polish what's currently load-bearing,
defer what isn't, document the deferred imperfections,
notice the trigger when deferred polish becomes load-
bearing. The timing-counterpart to `FormalVsImprovisational`.

### → Load `CaptureInClustersTriageLater`

When the work involves **rapid idea capture** — meetings,
design sessions, reading sessions, or a user firing
"add in: X" multiple times in quick succession.

Triggers:

- A conversation produces a cluster of related ideas.
- A user (or you) says "add in: X" several times in a row.
- You catch yourself thinking "I should refine this before
  filing" — and the result is you don't file at all.
- The tracker is being treated as a perfect filing cabinet
  rather than a scratchpad.
- A meeting produces 20 ideas; the tracker has time for
  three.
- About to lose track of an idea while triaging a related
  one.

**The skill teaches:** separate capture from triage.
Capture is fast, broad, friction-free; triage is slow,
narrow, discriminating. Do them at different times. The
methodology should be cheap enough to use as a scratchpad;
formal filing happens during triage.

### → Load `PushIsPublication`

When the work involves **deciding when to push commits to
origin**, especially in AI-augmented workflows where many
commits may be generated autonomously.

Triggers:

- An AI is generating commits autonomously and you're
  considering auto-push.
- You set up — or are tempted to set up — a post-commit
  push hook.
- Multiple AI sessions might trigger CI in racing ways.
- An AI just pushed a change you wanted to review locally
  first.
- You're working solo and your push log mirrors your
  commit log exactly.
- A force-push is being considered to fix something
  recently pushed reflexively.

**The skill teaches:** commit reflexively, push
deliberately. Local commits are cheap and reversible;
push is publication (triggers CI, deploys, visibility
to other sessions). Reserve push as a review-then-
publish checkpoint. Complements (doesn't contradict)
`SmallBatchCommitsMergedOften` — small batches are about
PR size; push-deliberately is about individual-session
publication cadence.

### → Load `GrammarIsAlsoAPI`

When the work involves **designing the syntax of how
operations are invoked** — verb forms in a CLI or
methodology, identifier sigils, capitalization of product
names — beyond just choosing the names themselves.

Triggers:

- Designing a CLI's verb table or a methodology's
  vocabulary.
- Choosing whether to prefix identifiers with a sigil
  (`R12345678` vs `12345678`).
- Capitalization decisions for product names (`radar.app`
  vs `Radar.app`).
- A conversational AI needs to recognize commands embedded
  in natural-language messages.
- Identifiers in your system are visually identical to
  identifiers in adjacent systems.
- The same operation has different phrasings across docs /
  UI / CLI / chat.

**The skill teaches:** grammar of invocation (verb forms,
sigils, casing) is a UX surface. Names say *what*;
grammar says *how to address it*. Self-identifying
identifiers (`R<body>` style sigils) survive in prose,
support precise regex matching, and disambiguate from
adjacent identifier-spaces. Permissive in input,
canonical in output.

### → Load `AskExactlyWhenAmbiguous`

When **operating under autonomous scope** ("work largely
unsupervised," "use your judgment," "you decide") and
hitting a decision point.

Triggers:

- The user said "work autonomously" or similar.
- You're tempted to ask permission for a reversible local
  file edit.
- You're tempted to silently make a call on something
  irreversible, externally-visible, or underspecified.
- An ambiguous instruction has two reasonable
  interpretations and the wrong one would be hard to
  undo.
- About to take a destructive action (delete, force-push,
  drop database).
- About to take an action whose side-effects exit the
  local environment (push, deploy, message, charge).
- A decision requires information the user has but you
  don't.

**The skill teaches:** default proceed for reversible local
decisions; pause for hard-to-reverse, externally-visible,
or under-specified ones. Structure the pause ("here are
options A/B/C, I lean A because X, what's your call?")
rather than open-ended. Over-confirming is friction;
silent deciding the wrong thing is worse. The skill is
reading which moment is which.

## How to use this index

1. Read the trigger phrases for each entry; they cover most of
   the situations the library knows about.
2. When you see one of those triggers in the work — in the user's
   message, in the code, in your own plan — **load the named
   skill** and follow its guidance.
3. If a situation matches *multiple* skills, load all of them.
   They compose. (A risky refactor of an AWS-touching service in
   a long-lived branch might want all of `MultiLevelTesting`,
   `SmallBatchCommitsMergedOften`, and
   `LocalAWSenvironmentUsingLocalstack`.)
4. If nothing here matches the situation, fall back to general
   reasoning — the library is intentionally selective.

## What the library doesn't cover

The library is small and biased. It does not cover:

- Performance tuning, profiling, or systems-level optimization.
- Security review or threat modeling.
- Architecture-at-scale (microservices, eventing, CQRS, etc.).
- Specific language idioms or framework-specific best practices.
- Hiring, mentoring, or organisational engineering.

For those, fall back to general knowledge or pull in a different
skill library.

## Adding to the library

A new lesson is a directory named in PascalCase with a single
`SKILL.md` following the same format as the others. After writing
it:

1. Add a new `### → Load <SkillName>` section to this routing
   table with 5–10 concrete trigger phrases.
2. Add a one-liner to the `README.md` table at the repo root.
3. Commit.

A lesson is worth adding when:

- The principle is recurring (it has fired in at least two
  separate contexts).
- The trigger is recognisable in the moment (not just visible in
  hindsight).
- The cost of *not* invoking the lesson is real (not just a
  matter of taste).
