# Critical Event Gate Skill

**Skill name:** Critical Event Gate  
**Skill type:** High-reliability operational pause / team safety protocol  
**Status:** Draft 1.0  
**Primary use:** Prevent unsafe execution at critical boundaries in human/AI collaboration systems.  
**Applies to:** Humans, AI collaborators, Codex, Claude Code, ChatGPT, Gemini, reviewers, implementors, operators.

---

## 1. One-line definition

A **Critical Event Gate** is a mandatory, recorded, challenge-response pause before crossing from safe preparation into live, costly, credentialed, user-impacting, security-sensitive, or irreversible action.

---

## 2. Why this skill exists

Complex expert work creates autopilot risk. Humans and AI collaborators can both become task-focused and proceed from “the plan is probably right” to “run the command” without noticing that the consequence class has changed.

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
| **Gate Evidence Packet** | The evidence used to support the Gate decision. |
| **Gate Owner** | The accountable human who can open or hold the Gate. |
| **Safety Verifier** | The reviewer, human or AI, tasked with finding stop conditions. |
| **Gate Poll** | The final explicit GO / NO-GO / HOLD poll. |
| **Release Authority** | The authority to proceed with the exact approved action. |
| **Gate Disposition** | The final result: GO, NO-GO, HOLD, ABORT, GO WITH LIMITATION, GO FOR PREFLIGHT ONLY, GO FOR LIVE DEV ONLY, or GO FOR PRODUCTION. |

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
No Critical Event Boundary may be crossed until the Critical Event Gate is completed, recorded, and explicitly opened by the Gate Owner.
```

If any required participant says **NO-GO**, the operation stops.

No averaging.  
No “probably fine.”  
No silent overrides.

---

## 6. Gate dispositions

| Disposition | Meaning |
|---|---|
| **GO** | Proceed with the exact approved action. |
| **NO-GO** | Do not proceed. A required criterion failed. |
| **HOLD** | Pause pending missing evidence or human decision. |
| **ABORT** | Stop and execute rollback/cleanup if appropriate. |
| **GO WITH LIMITATION** | Proceed only with explicitly recorded limits and accepted risk. |
| **GO FOR PREFLIGHT ONLY** | May run non-mutating validation, but not live mutation. |
| **GO FOR LIVE DEV ONLY** | May mutate approved dev resources, not production. |
| **GO FOR PRODUCTION** | May execute the production action exactly as approved. |

A Gate may be reopened after a failed or held criterion is resolved, but the new evidence must be recorded.

---

## 7. Challenge-response format

The Gate is performed as challenge-response.

The operator or Gate Owner reads the challenge. The responsible person or AI collaborator responds with an explicit status and evidence.

Example:

```text
Challenge: Are we in the correct repo and branch?
Response: GO — repo is ai-collaboration-system, branch is poc1c/cloudflare-dev, worktree is ~/Code/ai-legion-tracker.

Challenge: Has host-side smoke:local passed?
Response: NO-GO — smoke:local has not yet been run outside the sandbox.
```

Uncertainty is not a soft GO.

```text
If uncertain, answer NO-GO or HOLD.
```

---

## 8. Required inputs

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

## 9. Required outputs

The Gate produces a **Gate Record**.

Recommended file naming:

```text
<project>/<milestone>_CRITICAL_EVENT_GATE.md
```

Examples:

```text
cloudflare/legion-worker/POC1C_CRITICAL_EVENT_GATE.md
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
- Release Authority Statement.

---

## 10. Standard Gate Checklist

### 10.1 Mission

Challenge:

```text
What are we about to do?
Why are we doing it?
What is the expected success signal?
```

Example response:

```text
GO — Mission is to execute POC 1C live Cloudflare dev deployment and smoke test.
Success is live dev Worker deployed, smoke test passed, evidence recorded.
```

### 10.2 Scope

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

### 10.3 Execution context

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

### 10.4 Preflight evidence

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

### 10.5 Architecture confirmation

Challenge:

```text
Are the architecture decisions that constrain this action recorded and reflected in config/code?
```

Examples:

```text
CampaignDurableObject uses new_sqlite_classes.
Storage access remains KV-style for POC 1C.
SQL table rewrite deferred to POC 1D.
Critical Event Gate required before live mutation.
Conversation renamed to Collaboration in Worker boundary.
```

### 10.6 Secrets confirmation

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

### 10.7 Rollback plan

Challenge:

```text
If this fails, how do we stop, roll back, or clean up?
```

If rollback is unknown:

```text
NO-GO
```

### 10.8 Stop-condition review

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

### 10.9 Gate Poll

Challenge:

```text
Each required participant states GO, NO-GO, or HOLD.
```

Minimum roles:

- Gate Owner,
- Implementor,
- Safety Verifier.

One NO-GO stops the operation.

---

## 11. Stop Conditions

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

## 12. Gate Record template

```markdown
# Critical Event Gate Record

## Gate

Name:
Date:
Gate Owner:
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
| Gate Owner | GO/NO-GO/HOLD | | |
| Implementor | GO/NO-GO/HOLD | | |
| Safety Verifier | GO/NO-GO/HOLD | | |

## Gate Disposition

GO / NO-GO / HOLD / ABORT / GO WITH LIMITATION / GO FOR PREFLIGHT ONLY / GO FOR LIVE DEV ONLY / GO FOR PRODUCTION

## Release Authority Statement

## Notes
```

---

## 13. Example: POC 1C Cloudflare transition

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

## 14. Example: Conversation -> Collaboration vocabulary gate

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

## 15. Team behavior rules

### 15.1 GO must be explicit

Allowed:

```text
GO — worker tests passed at 14:32, evidence in POC1C_EVIDENCE.md.
```

Not allowed:

```text
Looks good.
Probably fine.
Should work.
I think so.
```

### 15.2 NO-GO is protected

Any participant, human or AI, may call NO-GO.

A NO-GO must be recorded with:

- reason,
- evidence,
- required resolution,
- owner.

A NO-GO is not failure. It is the system working.

### 15.3 The operator must know the next command

Before GO, the operator must be able to state the exact next command.

If the next command is unclear, the operation is NO-GO.

### 15.4 No hidden scope expansion

The Gate opens only for the exact approved action.

Any request to do something broader re-closes the Gate.

---

## 16. Relationship to SpecForge

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

---

## 17. Relationship to AI Collaboration Systems

Critical Event Gates are first-class collaboration events.

They should be represented as part of the collaboration timeline, not as side notes.

Future event model:

```text
CollaborationEvent.type = "critical-event-gate"
```

or:

```text
CollaborationEvent.type = "gate"
gate_class = "critical"
```

A Gate Event records the moment the team deliberately stopped autopilot before action.

---

## 18. Skill invocation prompt

Use this prompt when asking an AI collaborator to apply the skill:

```text
Apply the Critical Event Gate skill.

We are about to cross this Critical Event Boundary:

<boundary>

Create or update a Gate Record.

Do not execute the high-risk action.

Inventory the mission, scope, active spec, execution context, evidence, secrets, rollback plan, stop conditions, and Gate Poll.

Return a Gate Disposition:
GO, NO-GO, HOLD, ABORT, GO WITH LIMITATION, GO FOR PREFLIGHT ONLY, GO FOR LIVE DEV ONLY, or GO FOR PRODUCTION.

If any required evidence is missing, return HOLD or NO-GO.
```

---

## 19. Summary

The Critical Event Gate is a reusable high-reliability skill for human/AI teams.

It prevents unsafe action by forcing a structured, recorded pause at consequence-changing boundaries.

Its purpose is to break autopilot.

Its rule is simple:

```text
No Critical Event Boundary is crossed until the Gate is completed, recorded, and explicitly opened.
```
