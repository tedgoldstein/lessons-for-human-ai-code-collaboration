---
name: sharpen
description: Mid-session reflective pass on the skill library. Look at the skills that were actually loaded during this session, edit the weak ones in place (sharper triggers, tighter descriptions, document the failure modes that came up), and scaffold net-new skill files for gaps that recurred without coverage. Auto-applies edits anywhere on disk we can reach (Lessons/, project-local, ~/.claude/plugins/...) — review by git diff afterwards. Invoke during a long or complex session, while the evidence is still fresh.
version: 0.1.0
---

# Sharpen

A skill library improves by absorbing session friction. The
freshest, highest-fidelity evidence about which skills are sharp
and which are dull is sitting in the conversation you just had —
which triggers fired correctly, which fired late, which never
fired but should have, which prescriptions you agreed with, which
you overrode. That evidence evaporates when the session ends.

The discipline: **mid-session, take stock.** Don't wait for the
next time the same friction bites. Edit the loaded skills in
place while you still remember what was vague and where; scaffold
new skills for the recurring gaps. Write to disk anywhere you can
reach — including the plugin cache — and accept that some of
those edits may be overwritten on plugin update. The cost of a
lost edit is one re-pass; the cost of forgotten friction is
re-learning the same lesson next session.

A vague skill is worse than no skill. It eats context, fires at
the wrong moments, and trains the agent to ignore the library.
Sharpening is library maintenance. Treat it as work, not chore.

## When to invoke

- A skill loaded this session but its description didn't actually
  match when it should fire — the trigger phrasing was generic,
  the *when* was hand-wavy, the failure modes weren't listed.
- A skill fired late: you noticed mid-task you should have
  invoked it earlier — its triggers missed the symptom you
  actually saw.
- You did something repeatedly across the session with no skill
  covering it — that's a gap, not just a one-off task.
- A skill's body was vague exactly where you hit the edge case —
  the body needs a section the edge case writes for you.
- You disagreed with a skill and overrode it. That's evidence
  either the skill's prescription is wrong, or its scope was
  too broad and it shouldn't have fired here.
- The user types `/sharpen`, "sharpen this session," "what did
  we learn about our skills today," or asks for a library pass.
- The session is ending or about to compact — capture before the
  context is gone.

## The shape

| Step | What you do | Output |
|---|---|---|
| **1. Enumerate** | List every skill loaded this session (from the available-skills list and any `Skill` tool invocations). Include the description text as it currently reads. | Inventory of N skills with their current descriptions. |
| **2. Score** | For each: did its trigger match the moment it should have fired? Was its description specific? Did its body answer the question that came up? Tag each as **sharp**, **dull**, **mis-aimed**, or **gap-pointing**. | Per-skill verdict + the specific evidence (one sentence of what happened in this session). |
| **3. Note gaps** | What recurred this session that no skill covered? A pattern that happened twice is a candidate. A pattern that happened three times is a gap. | List of proposed new skills (name, one-line description, the recurring symptom). |
| **4. Sharpen in place** | For dull and mis-aimed skills: edit the file. Tighten the description (add concrete trigger phrases, name the failure mode), and add a section to the body capturing what just came up. | File edits, anywhere reachable (Lessons/, project-local, plugin cache). |
| **5. Scaffold new** | For genuine gaps: create the skill directory with frontmatter (name, description, version 0.1.0) and a stub body marked *draft*. The body can be short — the *trigger phrasing* and *what to do* are what matters at v0.1. | New skill files. |
| **6. Report** | Output a diff summary: what was sharpened, what was created, where each file is. Note plugin-cache edits as *may-be-overwritten*. | Final user-facing report. |

## Why it matters

1. **Evidence beats theory.** A skill description written
   in the abstract is guesswork. A skill description written
   after the skill fired (or failed to fire) in a real
   session is calibrated against reality. Mid-session is the
   window where that calibration is cheap.

2. **Skills compound.** A library with 30 sharp skills is
   not just 30 ÷ 10 better than one with 10 sharp skills —
   it's qualitatively better, because the coverage starts
   to match the work. A library with 30 dull skills is worse
   than 10 sharp ones; the dull ones drown out the sharp ones
   when the agent picks which to load.

3. **Triggers are the whole game.** The body of a skill only
   helps if it loads at the right moment. A perfect body
   behind a vague trigger is dead code. Sharpening triggers
   is the highest-leverage edit a skill library admits.

4. **Sessions are perishable.** "I'll fix that skill later"
   loses to compaction, context-switching, and the next
   session's distractions. The cost of editing now is
   minutes; the cost of editing later is re-running the
   investigation that produced the insight.

5. **The library is a product.** Other Claude sessions —
   yours and (via descendant clones) other people's — will
   load these skills. Sharpening here is shipping
   improvements to a shared tool. Treating skill quality as
   "good enough" leaves productivity on the table for every
   future session.

## Practical guidance

- **Sharpen the description first, body second.** The
  description is what the agent reads to *decide whether to
  load*. The body is what the agent reads *after* deciding.
  A skill with a sharp description and a mediocre body still
  fires at the right time; a skill with a great body and a
  vague description never gets the chance.

- **Make triggers concrete.** A trigger is a phrase the user
  might say or a symptom the agent might notice — not a
  category. "When you're debugging" is a category. "When a
  test passed locally but failed in CI" is a trigger.
  Imitate the trigger style of the strongest existing
  skills.

- **Capture the failure mode.** When a skill fires at the
  wrong moment, the fix is often a single sentence in the
  description explaining when *not* to apply it. Negative
  triggers are as valuable as positive ones.

- **Edit anywhere you can reach — but know what each
  location implies.** Skill files live in places with very
  different durability and review semantics. Auto-apply
  draws its safety from the surrounding system, not from
  sharpen itself:

  - **Git-tracked location** (`Lessons/`, project-local
    `.claude/` inside a git repo, any commons or descendant
    clone). The edit produces a diff. **The diff is the
    audit trail.** Review by `git diff`, roll back with
    `git checkout -- <file>` or a discarded commit. Safe
    to auto-apply because the change-control system is
    doing the work — sharpen is just a producer of
    diffs. In repos that use the Track coordination
    discipline (`Tracks/` directory or root `SKILL.md`), a
    multi-file sharpen pass is non-trivial mutation and
    should be entered into Track before edits begin, same
    as any other multi-file work.

  - **Ephemeral / plugin cache** (`~/.claude/plugins/cache/...`).
    No git, no audit, overwritten on the next plugin
    update. Auto-apply is still useful — the edit improves
    *this* session and the next few — but it is lossy.
    Flag each plugin-cache edit in the report so the user
    can propagate it upstream (PR against the plugin) if
    the change is worth keeping.

  - **Read-only locations** (no write permission, mounted
    read-only, etc.). Don't attempt the edit. Record the
    proposed change in the report as a *suggested edit
    you couldn't apply* and surface the path.

  The cost of a lost cache edit is small; the cost of not
  sharpening at all is compounding session friction.
  *Worth doing imperfectly* — but locate honestly.

- **One pass per session, not one pass per insight.** Wait
  until late in a complex session to invoke sharpen — that
  way one pass collects all the session's evidence. Invoking
  it three times in a session re-does the inventory step
  each time.

- **Gap-pointing is harder than sharpening.** Noticing that
  a skill is dull is easier than noticing that a skill is
  *missing*. To find gaps, look for: tasks you did multiple
  times, decisions you made without guidance, friction you
  worked around, things you wished someone had told you at
  the start of the session. Those are skill candidates.

- **Stub new skills at v0.1.0.** A draft skill with a clear
  description, a few trigger phrases, and a one-paragraph
  body is more valuable than nothing. The body grows by
  re-touch (`CaptureInClustersTriageLater` applies). Don't
  block on writing the perfect body.

- **Keep edits small and local.** Don't rewrite a skill
  end-to-end during sharpen — that's a different activity.
  Sharpening adds the missing trigger, adds the missing
  failure mode, fixes the one vague sentence. If a skill
  needs full rewrite, file a Track for it; don't try to
  do it inside sharpen.

## Common failure modes

- **Sharpening hypothetically.** Editing a skill based on
  what you imagine its weakness is, not on what actually
  happened this session. The whole point of sharpen is
  evidence-driven editing — abstract edits are best left to
  a deliberate review activity, not this one.

- **Scaffolding too many new skills.** Every interesting
  thought is not a skill. The bar is *recurring* — a
  pattern you saw multiple times, or that has clear future
  recurrence. One-off observations belong in a note, not a
  new skill file.

- **Overwriting good skills.** A skill that fired correctly
  is *not a candidate for editing*. Touching it because
  "while I'm here" makes the library noisier, not sharper.
  Score first; edit only the dull ones.

- **Library bloat from gap-filling.** Filing a new skill for
  every minor gap inflates the index until the *router* can't
  route. New skills should earn their place; consolidate
  with existing ones where possible.

- **Forgetting the report.** Auto-applying edits without
  reporting which files changed leaves the user with no
  audit trail. Always end with the diff summary, even when
  edits were small.

- **Sharpening during a rigid skill.** Some skills (TDD,
  systematic-debugging) are *rigid* — they say "follow this
  exactly." Don't invoke sharpen *inside* one of those.
  Wait until that skill's procedure is done; sharpen is a
  reflective activity, not an interrupt.

## When the principle DOES NOT apply

- **Short, smooth sessions.** A session that touched two
  files and finished cleanly has no friction to capture.
  Sharpen needs evidence; if there was none, skip it.

- **First session in a new project.** Early sessions
  produce a lot of false signal — apparent gaps are often
  just unfamiliarity. Wait until you've worked in the area
  enough to know which patterns recur.

- **Without write access.** If the relevant skills live in
  a directory you can't reach (e.g., a read-only mounted
  plugin, no write permission), record the proposed edits
  in a note and surface them to the user instead of
  silently dropping them.

- **When the user has paused or stepped away.** Auto-
  applied edits during an unsupervised idle window can
  surprise the user when they return. Prefer to report the
  proposed edits and apply on the next interaction, unless
  the user pre-authorised autonomous sharpening.

## Worked example

A long session debugging a flaky test ends. Skills loaded
during the session included `systematic-debugging`,
`test-driven-development`, `verification-before-completion`,
and `TheBisectMindset`. The user invokes `/sharpen`.

The pass:

1. **Enumerate** — four loaded skills, plus the
   conversation showed a fifth pattern (writing a
   reproducer script *before* bisecting) that no skill
   covered.

2. **Score** —
   - `systematic-debugging`: **sharp** (fired correctly,
     body matched the situation).
   - `test-driven-development`: **mis-aimed** (fired
     for a debugging task, where TDD-discipline added
     friction without value — its description should
     have said "for new feature work" more clearly).
   - `verification-before-completion`: **dull** — fired
     correctly but its body didn't mention how to verify
     a *flake* (which requires running the test multiple
     times, not once). Edge case missing.
   - `TheBisectMindset`: **sharp**.
   - Gap: writing a deterministic reproducer before
     bisecting. Came up three times across the session,
     not in any skill.

3. **Sharpen in place** —
   - Edit `test-driven-development`'s description to add
     "for *new* feature/bugfix work — not for debugging
     existing flaky behaviour." One-sentence fix.
   - Add a "Flaky tests" subsection to
     `verification-before-completion`'s body explaining
     that verification of intermittent failure requires
     multiple runs.

4. **Scaffold new** — create
   `Lessons/ReproducerBeforeBisect/SKILL.md` at v0.1.0
   with a description, four trigger phrases, and a stub
   body sketching the principle.

5. **Report** — diff summary: 2 files edited
   (`test-driven-development` was in plugin cache —
   flagged as *may-be-overwritten on plugin update,
   consider propagating upstream*), 1 file created
   (`ReproducerBeforeBisect/SKILL.md`, draft).

The next session that hits a flaky test loads
`verification-before-completion` and the "Flaky tests"
note is right there — the multi-run pattern doesn't have
to be re-discovered.

## Tagline

> Every session is a free training pass on the library.
> Spend it.

The session is the evidence. The library is the artifact.
Update one with the other while it's still fresh.

## See also

- [CaptureInClustersTriageLater](../CaptureInClustersTriageLater/SKILL.md)
  — sharpen is exactly the "triage later" half of the
  capture/triage split. The session captures friction;
  sharpen triages it.
- [NamingIsAPI](../NamingIsAPI/SKILL.md) — the description
  field of a skill *is* its API to the agent. Sharpening
  triggers is naming-as-API at the skill-library scale.
- [TheContractIsTheArtifact](../TheContractIsTheArtifact/SKILL.md)
  — SKILL.md (YAML frontmatter + body) is the contract.
  Sharpen edits the contract directly; tools are
  optional.
- [OneChangeAtATime](../OneChangeAtATime/SKILL.md) —
  during a sharpen pass, edit one skill at a time. Don't
  bundle "fix description" + "rewrite body" + "scaffold
  new" into one mega-edit. Small, observable edits.
- [PolishWhenLoadBearing](../PolishWhenLoadBearing/SKILL.md)
  — only sharpen skills whose dullness actually mattered
  this session. Polishing skills whose dullness didn't
  cost anything is misdirected effort.

## Sources

Distilled from the observation that skill libraries decay
when treated as static and improve when treated as living
artifacts updated against session evidence. Echoes
Anthropic's own writing-skills practice (skills evolve;
re-read the current version), and the broader "small,
frequent updates beat large, rare ones" principle from
trunk-based development.
