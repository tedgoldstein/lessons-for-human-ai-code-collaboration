---
name: using-lessons
description: Bootstrap directive for this skill library. Before acting on any non-trivial software-engineering task — committing, pushing, refactoring, naming, debugging a regression, designing an API, choosing a tool, running a postmortem, setting up multi-agent work, deploying — check whether a lesson applies; load it if it plausibly does; announce which one and why; follow it. Skill loading is an internal, cheap, reversible action; bias hard toward loading. Reconciles with AskExactlyWhenAmbiguous: load skills eagerly, pause the user only when the decision is hard to reverse, externally-visible, or under-specified.
version: 0.1.0
---

# Using Lessons

A library of principles is only useful if the agent actually
consults it at the moment of action. Good descriptions (the
[Index](../Index/SKILL.md), the per-skill frontmatter) make a
lesson *findable*; this skill makes the agent *find* it.

The principle: **before acting on an engineering task, check
whether a lesson applies. If one plausibly does, load it,
announce that you did, and follow it.** Loading is cheap. Not
loading risks the mistake the lesson exists to prevent.

This is a meta-skill — it doesn't teach an engineering principle
itself; it makes the rest of the library effective.

## When to invoke

Continuously, as a standing discipline rather than a one-time
session-start check. Specifically, evaluate before:

- Any commit, push, force-push, or merge.
- Any refactor, replacement, or migration.
- Any API / type / module / file / variable named for the
  first time.
- Any debugging session that's about to read more than a few
  commits, or about to bundle a "small cleanup" with a fix.
- Any design decision — what to test, how to log, which tool
  to adopt, how to handle retries, where state lives.
- Any incident response — during or after.
- Any session involving a second AI agent, a cloud routine,
  or a parallel human worker on the same repo.
- Any moment when a user phrase matches a recognisable trigger
  ("I'll figure it out as I go," "we'll add tests later,"
  "should I roll this into the same PR," "I think it broke
  somewhere this sprint").

If you can't tell whether a lesson applies, that *is* the
signal to glance at the [Index](../Index/SKILL.md) or scan
the frontmatter descriptions of installed skills. The check
itself takes seconds.

## The shape — the discipline in four steps

1. **Notice the moment.** An engineering task has begun, or
   a phase boundary has been crossed (about to commit, about
   to push, about to refactor, about to commit + ship).
2. **Check applicability.** Either (a) scan installed skill
   descriptions surfaced by the harness, or (b) consult
   [Index/SKILL.md](../Index/SKILL.md). The trigger phrases
   there are written to be matched against the situation in
   front of you.
3. **Load and announce.** If a trigger matches, load the
   skill's full body and announce in a single line which one
   and why: *"Loading `OneChangeAtATime` — this commit
   bundles a refactor with the bug fix."* If multiple
   triggers match, load all of them and announce all of
   them; they compose.
4. **Follow.** Read the skill, then act according to it. If
   the skill says "don't bundle changes," don't bundle them.
   Loading without following is theatre.

## Why it matters

Without a standing discipline, the lessons stay inert. A model
that *could* have loaded `OneChangeAtATime` but didn't is
indistinguishable from a model that doesn't have the library
installed. The skills' value is conditional on their being
consulted at the moment they apply.

The announce step matters separately. Announcing makes the
routing **legible** — to the human watching the session, to a
second agent that may later read the transcript, to a cold
reviewer arriving without context. It's the same pattern
[CausalDivergence](../CausalDivergence/SKILL.md) recommends
for state: make divergence detectable by putting the claim on
a surface other actors can read.

The discipline of "bias toward loading" works because the cost
asymmetry favours it: reading a skill that turns out not to
apply costs a page of context; *not* reading one that did
apply costs the bug. The expected cost of over-loading is
small and bounded; the expected cost of under-loading is the
class of mistakes the library was built to prevent.

## Practical guidance

**Bias toward loading.** When in doubt, load. The threshold
is "plausibly relevant," not "certainly relevant." If you can
articulate a one-sentence reason the skill might apply, that's
enough. Don't paralyze on the decision — the decision should
take seconds.

**Announce in one line.** Format: *"Loading `<SkillName>` —
<the specific trigger that matched, in your own words>."*
Skip ceremony. The announcement is for the reader, not for
the agent itself.

**Compose freely.** A risky refactor of an AWS-touching
service in a long-lived branch wants `MultiLevelTesting`,
`SmallBatchCommitsMergedOften`, *and*
`LocalAWSenvironmentUsingLocalstack`. Load all three;
announce all three; follow all three.

**Skill loading is not user interruption.** This is the
critical reconciliation with
[AskExactlyWhenAmbiguous](../AskExactlyWhenAmbiguous/SKILL.md):
*loading a skill into your own context is internal, reversible,
and cheap; pausing the user is external, expensive, and
reserved for genuine ambiguity.* The "bias hard toward
loading" rule applies only to the first; it does not weaken
the rule against over-confirming with the user.

**When descriptions are surfaced natively** (the library
installed in `.claude/skills/` or similar), the frontmatter
description of each skill is visible to the harness at
session start. In that mode, the agent doesn't need to
consult the [Index](../Index/SKILL.md) first; description-
matching does the routing. This skill remains the discipline
that says *do match, every task*.

**When descriptions are not surfaced** (library cloned as a
reference directory), this skill instructs the agent to
consult [Index/SKILL.md](../Index/SKILL.md) as the catalog.
The mechanism is different; the discipline is the same.

**Don't game the check.** If the work is genuinely trivial
(typo fix, one-line config change, pure conversational
question, status report), no skill applies and that's a
correct outcome of the check. The discipline is to *do* the
check, not to find something every time.

## Common failure modes

- **Loading-as-theatre.** Announcing the load but then
  proceeding to act in violation of the loaded skill. The
  announcement creates an expectation; ignoring it is worse
  than not loading.
- **Checking once at session start, then forgetting.**
  Engineering tasks arrive throughout a session; the check
  is per-task, not per-session.
- **Reading the Index but never loading from it.** The Index
  is the catalog, not the substance. Matching a trigger means
  loading the full skill.
- **Loading on pure information lookups.** Answering "what
  does this function do?" doesn't need a lesson; running
  `git bisect` against an unknown regression does.
- **Conflating with AskExactlyWhenAmbiguous.** The two
  operate at different scopes. This skill governs *internal*
  reading; that one governs *external* user interaction.
  Confusing them produces either over-confirming users or
  under-loading skills.
- **Treating the announcement as optional.** The announcement
  is what makes the routing observable. Skipping it is the
  difference between a transparent session and an opaque one.

## When the principle DOES NOT apply

- **Pure conversational turns** ("what's the difference
  between X and Y?", "explain this code"). No engineering
  action is being taken; no lesson applies.
- **Trivial mechanical edits** that fall below the threshold
  of any skill's triggers — fixing a typo in a comment,
  renaming a single local variable inside a function.
- **When the user has already named the principle.** "Please
  apply `OneChangeAtATime` to this PR" — load it and act;
  the check is redundant.
- **In a context where the skill library is genuinely not
  available** (no `.claude/skills/` install, no cloned
  reference directory, no `Index` reachable). The discipline
  presupposes the artifacts; without them, fall back to
  general engineering judgment.

## Tagline

> Loading a skill is reading a page; acting without one that
> fit is the mistake the skill exists to prevent.

A library of lessons routes itself only if the agent
reflexively asks, before every engineering act, *"is one of
these the moment?"* — and answers in one observable line.

## See also

- [Index](../Index/SKILL.md) — the catalog this discipline
  consults. The Index lists the trigger phrases; this skill
  is the standing instruction to scan them.
- [AskExactlyWhenAmbiguous](../AskExactlyWhenAmbiguous/SKILL.md)
  — the sibling discipline that governs *user interruption*.
  Skill loading is eager; user interruption is reserved.
  Together they cover both directions of agent action.
- [CausalDivergence](../CausalDivergence/SKILL.md) — the
  announce-on-load ritual makes routing legible to other
  actors, the same way `CausalDivergence` recommends making
  state divergence legible across timelines.

## Sources

Adapts the `using-superpowers` bootstrap pattern from
[obra/superpowers](https://github.com/obra/superpowers) — a
mature agentic-skills framework — which establishes the
"check skills before acting" discipline via a permissive
relevance threshold ("1% chance → invoke") plus a mandatory
announce step. This skill keeps the spirit (bias hard toward
loading; make routing observable) but reframes the threshold
for a *principles* library rather than a *procedures* one:
"load when a trigger plausibly matches" fits reflective
lessons better than a numeric percentage suited to procedural
recipes. The reconciliation with
[AskExactlyWhenAmbiguous](../AskExactlyWhenAmbiguous/SKILL.md)
is original to this library — Superpowers does not face the
same tension because its skills are procedural rather than
judgment-shaped.
