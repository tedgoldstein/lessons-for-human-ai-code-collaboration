---
name: critical-event-gate
description: A mandatory, recorded, challenge-response pause before crossing from safe preparation into live, costly, credentialed, user-impacting, security-sensitive, or irreversible action. Complex expert work creates autopilot risk — humans and AI collaborators alike drift from "the plan is probably right" to "run the command" without noticing the consequence class has changed. Borrowing from surgical time-outs, aviation challenge-response checklists, nuclear STAR behavior, and military Go/No-Go polls, the gate interrupts autopilot at the boundary. Load this skill before any action that can mutate live infrastructure, deploy code, create/delete cloud resources, set or rotate or expose secrets, spend material money, call real provider APIs, touch customer/PII/PHI/regulated data, alter remote (GitHub) state, change production config, grant credentials, cross from sandbox to host/network execution, make a breaking API/schema change, take an expensive-to-unwind architecture decision, or authorize broad-scope autonomous AI execution. Release Authority is human-only.
version: 0.1.0
---

# Critical Event Gate Skill

**Skill name:** Critical Event Gate  
**Skill type:** High-reliability operational pause / team safety protocol  
**Status:** Draft 1.3  
**Primary use:** Prevent unsafe execution at critical boundaries in human/AI collaboration systems.  
**Applies to:** Humans, AI collaborators, Codex, Claude Code, ChatGPT, Gemini, reviewers, implementors, operators.

---

## 1. One-line definition

A **Critical Event Gate** is a mandatory, recorded, challenge-response pause before crossing from safe preparation into live, costly, credentialed, user-impacting, security-sensitive, or irreversible action.

---

## 2. Why this skill exists

Complex expert work creates autopilot risk. Humans and AI collaborators can both become task-focused and proceed from "the plan is probably right" to "run the command" without noticing that the consequence class has changed.

High-reliability fields use deliberate pauses to interrupt autopilot:

- surgical time-outs before incision,
- aviation challenge-response checklists,
- independent double checks for high-risk medication,
- nuclear STAR behavior: Stop, Think, Act, Review,
- deployment canaries and rollback gates,
- military Go / No-Go polls.

The Critical Event Gate adapts that pattern for AI collaboration, software operations, cloud infrastructure, credentials, cost-bearing workflows, and architecture-changing work.

The goal is not bureaucracy. The goal is to prevent catastrophe.

---

## 3. Core vocabulary

| Term | Meaning |
|---|---|
| **Critical Event Gate** | The whole safety mechanism. |
| **Critical Event Boundary** | The boundary being crossed, such as local to live, mock to real, read-only to mutating, no-secrets to secrets. |
| **Gate Checklist** | The structured challenge-response checklist. |
| **Gate Criterion** | A required condition that must be satisfied. |
| **Stop Condition** | A condition that blocks proceeding. |
| **Gate Record** | The written artifact recording evidence, decisions, stop conditions, and final disposition. |
| **Gate Record Status** | The lifecycle status of the Gate Record itself: Draft, Held, Opened, Closed, Aborted. |
| **Gate Evidence Packet** | The evidence used to support the Gate decision. |
| **Gate Owner** | The accountable human who can open or hold the Gate. |
| **Safety Verifier** | The reviewer, human or AI, tasked with finding stop conditions. |
| **Gate Poll** | The final explicit scoped-GO / NO-GO / HOLD poll. |
| **Release Authority** | The authority to proceed with the exact approved action. Release Authority is human-only. |
| **Gate Disposition** | The final decision about what action, if any, may proceed. |

---

## 4. When to invoke this skill

Invoke a Critical Event Gate before any action that can:

- mutate live infrastructure,
- deploy code,
- create, delete, or modify cloud resources,
- set, rotate, expose, or consume secrets,
- spend material money,
- call real provider APIs,
- touch customer, user, PII, PHI, or regulated data,
- alter GitHub or other remote state,
- change production-like configuration,
- grant credentials or permissions,
- cross from local/sandbox work to host-side or network-capable execution,
- change public API/schema vocabulary in a breaking way,
- make an architecture decision that will be expensive to unwind,
- authorize autonomous AI execution with broad scope.

The above list is illustrative, not exhaustive. The boundary between "gate-worthy" and "ordinary work" is intentionally fuzzy. The Gate Owner is the final classifier; if you are uncertain whether a boundary is gate-worthy, invoke the Gate.

Examples of Critical Event Boundaries:

```text
local -> host-side
sandbox -> network-capable
mock -> live
read-only -> mutating
dev -> production
no-secrets -> secrets
no-spend -> metered spend
manual review -> autonomous execution
conversation vocabulary -> collaboration vocabulary
planning -> deployment
```

---

## 5. Core rule

```text
No Critical Event Boundary may be crossed until the Critical Event Gate is completed, recorded, and explicitly opened by the human Gate Owner.
```

Only a human Gate Owner can open a Gate.

AI collaborators may:

- provide evidence,
- identify Stop Conditions,
- call NO-GO,
- call HOLD,
- recommend a disposition.

AI collaborators may **not** grant Release Authority.

If any required participant says **NO-GO**, the operation stops.

No averaging.  
No "probably fine."  
No silent overrides.  
No unscoped GO.

---

## 6. Gate dispositions

There are two families of dispositions:

1. **Opening dispositions**, decided before an action starts.
2. **In-flight dispositions**, decided during or after execution.

Every opening disposition must name an explicit scope. There is no unscoped "GO"; the operator must always state what is being approved.

### 6.1 Opening dispositions

| Disposition | Meaning |
|---|---|
| **NO-GO** | Do not proceed. A required criterion failed. The current attempt is terminated. A new Gate is required to re-attempt. |
| **HOLD** | Pause pending missing evidence that is in flight or recoverable in the current session. May resume without a new Gate once the evidence lands. |
| **GO FOR ACCOUNT INSPECTION ONLY** | May run read-only account identity checks such as `wrangler whoami`. May not mutate resources, set secrets, or deploy. |
| **GO FOR PREFLIGHT ONLY** | May run non-mutating validation in any context. May not perform live mutation. |
| **GO FOR HOST-SIDE PREFLIGHT ONLY** | May run host-side, non-cloud-mutating validation such as local runtime checks and `smoke:local`. May not run live cloud mutation, deploy, secret setting, or resource creation. |
| **GO WITH LIMITATION** | Proceed only within explicitly recorded limits, with the accepted risk named. |
| **GO FOR LIVE DEV ONLY** | May mutate approved dev resources. May not touch production. |
| **GO FOR STAGE N ONLY** | Approved for one named stage of a staged plan. Subsequent stages require their own Gate. |
| **GO FOR PRODUCTION** | May execute the production action exactly as approved. |

### 6.2 HOLD vs NO-GO

Use `HOLD` if the missing or failed criterion can be resolved inside the current session.

Examples:

```text
A smoke test is still running.
A reviewer is available and actively reviewing.
A command output is being collected.
```

Use `NO-GO` if the problem requires replanning, a new evidence cycle, a fresh Gate, or a change in scope.

Examples:

```text
Wrong account.
Wrong branch.
Rollback plan missing.
Secret value was exposed.
The proposed action is broader than approved.
The implementation does not match the active spec.
```

### 6.3 In-flight dispositions

| Disposition | Meaning |
|---|---|
| **ABORT** | Stop an action that has already begun and execute the rollback/cleanup plan. Applies only after a Gate has been opened and execution has started. |

`ABORT` is not an alternative to `NO-GO`.

If the action has not begun, the correct call is `NO-GO`, not `ABORT`.

A Gate may be reopened after a held criterion is resolved, but the new evidence must be recorded.

A NO-GO Gate Poll terminates the current Gate attempt. A later re-attempt requires a new Gate Record unless the campaign's governance explicitly allows reopening a NO-GO record.

---

## 7. Gate Record status lifecycle

Gate Record Status is separate from Gate Disposition.

The disposition answers:

```text
What was decided?
```

The status answers:

```text
Where is the record in its lifecycle?
```

| Status | Meaning |
|---|---|
| **Draft** | The Gate Record is being prepared. Evidence is being gathered. No poll has been taken. |
| **Held** | A Gate Poll has been taken and the disposition is HOLD. The Gate is paused, not closed. |
| **Opened** | A Gate Poll has been taken and a scoped GO disposition has been granted by the human Gate Owner. The approved action may proceed within scope. |
| **Closed** | The Gate Record is complete. For successful GO actions, this requires post-action close-out. For NO-GO attempts, this records that the attempt was terminated. |
| **Aborted** | The approved action began and was stopped via the in-flight ABORT disposition, with rollback/cleanup recorded. |

Important distinctions:

```text
A Gate Record with a GO disposition is Opened, not Closed.
It becomes Closed only after post-action close-out is completed.

A Gate Record with NO-GO disposition is Closed once the refusal and required resolution are recorded.
NO-GO is not a failed execution; it is a prevented unsafe execution.
```

---

## 8. Challenge-response format

The Gate is performed as challenge-response.

The operator or Gate Owner reads the challenge. The responsible person or AI collaborator responds with explicit status and evidence.

Example:

```text
Challenge: Are we in the correct repo and branch?
Response: GO FOR LIVE DEV ONLY -- repo is ai-collaboration-system, branch is poc1c/cloudflare-dev, worktree is ~/Code/ai-legion-tracker.

Challenge: Has host-side smoke:local passed?
Response: NO-GO -- smoke:local has not yet been run outside the sandbox.
```

Uncertainty is not a soft GO.

```text
If uncertain, answer NO-GO or HOLD.
```

---

## 9. Required inputs

A Critical Event Gate requires, at minimum:

- mission,
- scope boundaries,
- active spec or decision record,
- current branch/worktree,
- execution plan,
- rollback plan,
- known risks,
- latest validation results,
- resource names,
- secret names, never values,
- authority to proceed,
- responsible owner,
- safety verifier.

For software deployment, the minimum evidence usually includes:

- typecheck result,
- test result,
- local smoke result,
- known limitation list,
- deployment command list,
- rollback command list,
- secret-handling plan.

---

## 10. Required outputs

The Gate produces a **Gate Record**.

Gate Records live alongside other Decisions, under a single decisions directory, so that one audit ledger covers both classes of authority artifact:

```text
docs/decisions/<short-name>_CRITICAL_EVENT_GATE.md
```

Examples:

```text
docs/decisions/POC1C_CRITICAL_EVENT_GATE.md
docs/decisions/VOCABULARY_CRITICAL_EVENT_GATE.md
```

The Gate Record must include:

- mission,
- scope,
- approved actions,
- not-approved actions,
- active spec / decision,
- repo / branch / worktree,
- execution context,
- evidence table,
- architecture confirmation,
- secret-handling confirmation,
- stop-condition review,
- rollback plan,
- final Gate Poll,
- final Gate Disposition,
- Gate Record Status,
- Release Authority Statement,
- post-action close-out.

---

## 11. Standard Gate Checklist

### 11.1 Mission

Challenge:

```text
What are we about to do?
Why are we doing it?
What is the expected success signal?
```

Example response:

```text
GO FOR LIVE DEV ONLY -- Mission is to execute POC 1C live Cloudflare dev deployment and smoke test.
Success is live dev Worker deployed, smoke test passed, evidence recorded.
```

### 11.2 Scope

Challenge:

```text
What is approved?
What is explicitly not approved?
```

Example:

```text
Approved:
  Cloudflare dev Worker
  dev R2 bucket
  dev Queue
  dev secrets
  live dev smoke test

Not approved:
  production deployment
  custom domain
  real AI-provider keys
  GitHub mutation
  user data
  financial/trading integrations
```

### 11.3 Execution context

Challenge:

```text
Where will this execute?
Is this local, sandbox, host-side, cloud-dev, or production?
```

Required checks:

- correct machine,
- correct terminal context,
- correct repo,
- correct branch,
- correct worktree,
- correct cloud account,
- correct Node/Wrangler/tool versions,
- no hidden execution-context mismatch.

### 11.4 Preflight evidence

Challenge:

```text
What evidence proves the system is ready for this boundary?
```

Common checks:

```text
root typecheck
root tests
worker typecheck
worker tests
smoke:in-process
host-side smoke:local
security review
cost estimate
rollback tested or documented
```

### 11.5 Architecture confirmation

Challenge:

```text
Are the architecture decisions that constrain this action recorded and reflected in config/code?
```

The specific items to confirm are campaign-specific. For one campaign the relevant items might be:

```text
[illustration; replace per campaign]
CampaignDurableObject uses new_sqlite_classes.
Storage access remains KV-style for POC 1C.
SQL table rewrite deferred to POC 1D.
Critical Event Gate required before live mutation.
Conversation renamed to Collaboration in Worker boundary.
```

Treat the examples above as one campaign's confirmation list. Maintain your campaign's actual list in the Gate Record's architecture confirmation table.

### 11.6 Secrets confirmation

Challenge:

```text
Which secrets are involved, and where will their values appear?
```

Rules:

- record secret names only,
- never record secret values,
- never paste secrets into chat,
- never commit `.env`, `.dev.vars`, screenshots, or logs with secret values,
- never turn a secret-setting command into a transcript containing the value.

Additional challenge if any secret involved is being rotated:

```text
If this Gate involves rotating an existing secret, what compensating controls confirm that no in-flight process still depends on the old value?
```

If the answer is unknown, the rotation is `NO-GO` or `HOLD`, depending on whether the dependency check is possible inside the current session.

### 11.7 Rollback plan

Challenge:

```text
If this fails, how do we stop, roll back, or clean up?
```

If rollback is unknown:

```text
NO-GO
```

### 11.8 Stop-condition review

Challenge:

```text
Does any Stop Condition apply?
```

Examples:

- wrong repo,
- wrong branch,
- unexplained dirty worktree,
- wrong cloud account,
- missing rollback plan,
- missing host-side smoke result,
- ambiguous resource names,
- secret values exposed,
- unreviewed production command present,
- operator cannot state the next command.

### 11.9 Gate Poll

Challenge:

```text
Each required participant states scoped-GO, NO-GO, or HOLD.
```

Minimum roles:

- Gate Owner,
- Implementor,
- Safety Verifier.

One NO-GO stops the operation.

AI collaborators may give evidence, call HOLD, or call NO-GO. They cannot grant Release Authority.

---

## 12. Stop Conditions

Automatic NO-GO conditions:

```text
wrong repo
wrong branch
wrong cloud account
unexplained dirty worktree
required smoke test has not passed
rollback plan missing
resource names ambiguous
secret values exposed
production command not explicitly approved
config inconsistent with active spec
operator cannot state exact next command
AI requests broader scope than approved
```

For live cloud deployment, additional Stop Conditions include:

```text
wrangler deploy command not explicitly listed
wrangler secret put not explicitly approved
cloud-mutating commands not labeled
Cloudflare account identity not confirmed
live resource names not final
secrets would be visible in chat/logs/docs
```

For vocabulary/architecture gates:

```text
public API and serialized JSON would become inconsistent
partial rename would leave tests and docs misleading
the proposed term encodes the wrong product model
compatibility/migration assumptions are unverified
```

---

## 13. Post-action close-out

A Gate Record is not closed by the GO call.

After the approved action runs, the Gate Owner reopens the Gate Record and adds a close-out entry recording what actually happened.

| Close-out field | Value |
|---|---|
| Action outcome | succeeded / failed / partial |
| Rollback executed? | yes / no, with detail |
| Deviations from approved scope | none / list |
| New risks observed | none / list |
| New Stop Conditions to add to future Gates | none / list |

A Gate Record with a scoped-GO disposition and no close-out remains **Opened**, not Closed.

The skill exists to prevent catastrophe, not to produce approval-only artifacts; the close-out is what turns a Gate into a learning record.

---

## 14. Gate Record template

```markdown
# Critical Event Gate Record

## Gate

Name:
Date:
Status: Draft / Held / Opened / Closed / Aborted
Gate Owner:          (human)
Safety Verifier:
Implementor:
Project:
Milestone:

## Critical Event Boundary

From:
To:

## Mission

## Scope

### Approved

### Not Approved

## Active Spec / Decision Record

## Repo / Branch / Worktree

## Execution Context

## Gate Evidence Packet

| Criterion | Result | Evidence |
|---|---|---|
| root typecheck | GO/NO-GO/HOLD | |
| root tests | GO/NO-GO/HOLD | |
| worker typecheck | GO/NO-GO/HOLD | |
| worker tests | GO/NO-GO/HOLD | |
| smoke:in-process | GO/NO-GO/HOLD | |
| host-side smoke:local | GO/NO-GO/HOLD | |
| security review | GO/NO-GO/HOLD | |
| rollback plan | GO/NO-GO/HOLD | |

## Architecture Confirmation

| Item | Value | Result |
|---|---|---|
| Active spec | | |
| Worker name | | |
| Durable Object backend | | |
| Storage access mode | | |
| API vocabulary | | |
| Resource names | | |
| SQL migration status | | |

## Secrets

Secret names only. No values.

| Secret | Result |
|---|---|
| LEGION_DEV_TOKEN | GO/NO-GO/HOLD |
| LEGION_INTERNAL_TOKEN | GO/NO-GO/HOLD |

## Rollback Plan

## Stop Conditions Reviewed

| Stop Condition | Present? | Notes |
|---|---|---|
| wrong repo/branch | yes/no | |
| unexplained dirty worktree | yes/no | |
| wrong account/context | yes/no | |
| missing smoke evidence | yes/no | |
| missing rollback | yes/no | |
| secret exposure risk | yes/no | |
| production risk | yes/no | |

## Gate Poll

| Role | Vote | Name/Agent | Notes |
|---|---|---|---|
| Gate Owner | scoped-GO / NO-GO / HOLD | | |
| Implementor | scoped-GO / NO-GO / HOLD | | |
| Safety Verifier | scoped-GO / NO-GO / HOLD | | |

## Gate Disposition

One of: NO-GO, HOLD, GO FOR ACCOUNT INSPECTION ONLY, GO FOR PREFLIGHT ONLY, GO FOR HOST-SIDE PREFLIGHT ONLY, GO WITH LIMITATION, GO FOR LIVE DEV ONLY, GO FOR STAGE N ONLY, GO FOR PRODUCTION.

ABORT is used only as an in-flight disposition during close-out, not here.

## Release Authority Statement

## Post-Action Close-Out

| Field | Value |
|---|---|
| Action outcome | succeeded / failed / partial |
| Rollback executed? | yes / no, with detail |
| Deviations from approved scope | |
| New risks observed | |
| New Stop Conditions to add to future Gates | |

## Notes
```

---

## 15. Example: POC 1C Cloudflare transition

Critical Event Boundary:

```text
local/sandbox validation -> host-side Cloudflare live dev execution
```

Current possible disposition:

```text
GO FOR HOST-SIDE PREFLIGHT ONLY
```

Required next criterion:

```text
host-side smoke:local must pass
```

Release authority required before:

```text
wrangler login
wrangler r2 bucket create
wrangler queues create
wrangler secret put
wrangler deploy
```

Example Release Authority Statement:

```text
I have completed the POC 1C Critical Event Gate. I approve live dev Cloudflare POC 1C execution in the host-side terminal for the resources and commands listed in POC1C_EXECUTION_PLAN.md. I do not approve production deployment, custom domains, real AI-provider keys, GitHub mutation, user data, or financial/trading integrations.
```

---

## 16. Example: Conversation -> Collaboration vocabulary gate

Critical Event Boundary:

```text
chat-centric vocabulary -> collaboration-centric product architecture
```

Stop Condition:

```text
The term Conversation encodes the wrong durable object model before UX and API stabilization.
```

Gate Disposition:

```text
GO FOR STAGE 1 ONLY
```

Approved scope:

```text
Worker/public API/schema/storage/tests/smoke harnesses + decision record
```

Not approved:

```text
root CLI rename
broad docs cleanup
UX prototype
live POC 1C deployment
```

Decision:

```text
Campaign -> CampaignCollaboration -> CollaborationEvent
```

Rationale:

```text
A conversation is one possible mode inside a collaboration. The durable object is a collaboration because it includes messages, AI work, reviews, decisions, artifacts, tests, evidence, Critical Event Gates, budgets, credentials, and audit records.
```

---

## 17. Team behavior rules

### 17.1 Scoped GO is required

Every GO must name its scope.

Allowed:

```text
GO FOR LIVE DEV ONLY -- worker tests passed at 14:32, evidence in POC1C_EVIDENCE.md.
```

Not allowed:

```text
Looks good.
Probably fine.
Should work.
I think so.
GO
```

An AI collaborator may answer with evidence and may call NO-GO or HOLD. The GO call itself belongs to the human Gate Owner. AI collaborators cannot grant Release Authority.

### 17.2 NO-GO is protected

Any participant, human or AI, may call NO-GO.

A NO-GO must be recorded with:

- reason,
- evidence,
- required resolution,
- owner.

A NO-GO is not failure. It is the system working.

### 17.3 The operator must know the next command

Before scoped-GO, the operator must be able to state the exact next command.

If the next command is unclear, the operation is NO-GO.

### 17.4 No hidden scope expansion

The Gate opens only for the exact approved action.

Any request to do something broader re-closes the Gate.

---

## 18. Relationship to SpecForge

SpecForge answers:

```text
Is the artifact good enough?
```

Critical Event Gate answers:

```text
Is it safe to act now?
```

A spec can be accepted while execution remains NO-GO.

A codebase can pass tests while deployment remains HOLD.

A vocabulary decision can be conceptually right while implementation remains STAGE 1 ONLY.

### 18.1 Mapping to SpecForge roles and artifacts

The Critical Event Gate is not a parallel governance system. A Gate Record is a **SpecForge Decision of class `critical-event-gate`**, and the roles defined here map onto SpecForge roles, so that one audit ledger covers both.

| Critical Event Gate concept | SpecForge equivalent |
|---|---|
| Gate Owner | Arbiter, or Campaign Owner for ultimate authority |
| Safety Verifier | Security Verifier |
| Implementor | Implementor / Coder |
| NO-GO call | safety / conscience refusal; cannot be averaged, cannot be arbitrated away |
| Gate Disposition | `Decision.disposition`, extended enum for this class |
| Gate Record | Decision artifact, class = `critical-event-gate` |
| Stop Condition | pre-decision blocker recorded on the Decision |
| Gate Poll | recorded approval/refusal section of the Decision |
| Release Authority Statement | the Decision's authority field |

Where SpecForge and CEG vocabulary appear to differ, the SpecForge definitions in the campaign's active SpecForge profile are authoritative for roles and audit; CEG terms are the operational names used during the pause itself.

A single Decisions directory holds both ordinary SpecForge Decisions and Gate Records.

---

## 19. Relationship to AI Collaboration Systems

Critical Event Gates are first-class collaboration events.

They should be represented as part of the collaboration timeline, not as side notes.

Once the Conversation -> Collaboration rename has landed, the event model includes:

```text
CollaborationEvent.type = "critical-event-gate"
```

or:

```text
CollaborationEvent.type = "gate"
gate_class = "critical"
```

Until that rename ships in code, Gate Records live as standalone artifacts under the decisions directory.

A Gate Event records the moment the team deliberately stopped autopilot before action.

---

## 20. Skill invocation prompt

Use this prompt when asking an AI collaborator to apply the skill:

```text
Apply the Critical Event Gate skill.

We are about to cross this Critical Event Boundary:

<boundary>

Create or update a Gate Record.

Do not execute the high-risk action.

Inventory the mission, scope, active spec, execution context, evidence, secrets, rollback plan, stop conditions, and Gate Poll.

Return a Gate Disposition from this set:
NO-GO, HOLD, GO FOR ACCOUNT INSPECTION ONLY, GO FOR PREFLIGHT ONLY, GO FOR HOST-SIDE PREFLIGHT ONLY, GO WITH LIMITATION, GO FOR LIVE DEV ONLY, GO FOR STAGE N ONLY, GO FOR PRODUCTION.

You may answer with evidence and may call NO-GO or HOLD. The GO call itself belongs to the human Gate Owner. You cannot grant Release Authority.

If any required evidence is missing, return HOLD or NO-GO.

After the action runs, return to this Gate Record and fill in the post-action close-out; only then is the record Closed.
```

---

## 21. Summary

The Critical Event Gate is a reusable high-reliability skill for human/AI teams.

It prevents unsafe action by forcing a structured, recorded pause at consequence-changing boundaries.

Its purpose is to break autopilot.

Its rule is simple:

```text
No Critical Event Boundary is crossed until the Gate is completed, recorded, and explicitly opened by the human Gate Owner.
```
