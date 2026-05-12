---
name: postmortems-without-blame
description: After an incident, the goal is systemic understanding — not assignment of blame to an individual. The person who pushed the button was the proximate cause; the system that let one push break production is the root cause. Frame postmortems around what the *system* should change so the same class of failure can't happen again.
version: 0.1.0
---

# Postmortems Without Blame

A production incident happened. Someone deployed something that
broke. The temptation is to ask "who was the engineer that
pushed it?" The wrong question.

The right question is: **what about the system allowed one
person's mistake to break production?** No CI gate caught it.
No staged rollout caught it. No alerting fired before users
felt it. The engineer was the trigger; the system was the
loaded gun.

The principle: **postmortems aim at the system, not the
person.** Assign actions to systems, not to people. The
outputs of a good postmortem are changes that make the same
class of failure impossible — not promises to "be more
careful next time."

## When to invoke

- An incident just happened. (Severity: anything from "minor
  latency bump" to "site down.")
- Someone wants to "find out who did this."
- A previous incident has recurred — the prior postmortem
  didn't produce systemic fixes.
- A near-miss happened (no customer impact, but it almost
  did).
- An engineer is being singled out as "the one who broke
  prod" rather than the team examining what made the break
  possible.
- A scheduled deploy went wrong and the room's energy is
  *who* rather than *what.*

## The shape — what blameless postmortem looks like

A good postmortem document covers, in order:

### 1. Timeline

Just facts, in chronological order. What happened, when,
who-acted-when (with names, but as actors in the system, not
suspects). E.g.:

```
14:02   Engineer A merged PR #4234 to main.
14:04   Deploy pipeline started.
14:11   Deploy completed; new version in production.
14:18   Synthetic monitor reported elevated 500s.
14:21   On-call engineer B paged.
14:23   Engineer B confirmed elevated error rate in metrics.
14:29   Engineer B initiated rollback.
14:34   Rollback complete; error rate returned to normal.
```

Names are present because the timeline is meaningless without
who acted. The framing is *not* "Engineer A caused the outage";
it's "merge → deploy → manifested → detected → mitigated."

### 2. Impact

What did customers / users experience? For how long? What
volume? Quantify.

### 3. Root cause (the technical one)

What in the code or config triggered the failure. Specific,
concrete. "PR #4234 introduced a misconfigured connection
pool; under production load the pool exhausted in 4 minutes."

### 4. Contributing factors (the systemic ones)

What about the *system* allowed this trigger to reach
production? These are the interesting ones:

- "Connection pool size was hardcoded; no review checklist
  flagged it."
- "Staging environment had different traffic patterns; the
  bug didn't manifest there."
- "No alert on connection pool exhaustion existed."
- "Rollback took 15 minutes because deploys are serial."
- "On-call wasn't paged for 7 minutes because the alert
  threshold required 3 consecutive failures."

Each contributing factor is an *opportunity* to make the
system safer.

### 5. What went well

Not a courtesy section. Real value: the rollback worked, the
synthetic monitor caught it before customers complained, the
on-call response was fast. Recognise the parts of the system
that *did* work — those are the parts to preserve and grow.

### 6. Action items

For each contributing factor, a concrete change:

| Factor | Action | Owner | Due |
|---|---|---|---|
| Connection pool size hardcoded | Move to config; add lint for hardcoded pool sizes | Team A | 2 weeks |
| Staging traffic differs from prod | Add a synthetic load generator to staging | Team B | 1 month |
| No alert on pool exhaustion | Add an alert at 80% utilisation | Team A | 1 week |
| 15-minute rollback | Investigate canary deploys | Team C | quarter |

Actions are owned by *teams* or *roles* (not "Engineer A"),
and have a due date. An action without an owner and a date is
a wish.

## Why it matters

1. **Systems break, not people.** The same engineer who
   pushed the bad change pushes good changes every day.
   What was different about this one? Usually: the system
   didn't catch what the system catches in other cases.

2. **Blame discourages reporting.** When mistakes lead to
   punishment, near-misses stop being reported. The team
   loses visibility into the *most informative* failures
   (the ones that almost happened).

3. **Blame doesn't fix anything.** Disciplining the engineer
   doesn't prevent the next engineer from making the same
   mistake. Fixing the system does.

4. **Patterns surface.** When postmortems target the system,
   you start to see recurring themes — "we keep getting
   bitten by config drift," "our alerts are slow," "rollback
   is brittle." Those are the strategic improvements.

5. **Trust compounds.** Teams that run blameless
   postmortems are teams that escalate near-misses early
   and propose risky-but-correct fixes. Teams that don't,
   bury problems.

## Practical guidance

- **Use the word "system" relentlessly.** "What about the
  *system* let this happen?" "What about the *system* didn't
  catch this?" The vocabulary shapes the room.
- **Names are facts, not accusations.** Engineer A merged
  the PR — that's the timeline. The framing is "what about
  our merge process let a bad PR through," not "why didn't
  Engineer A notice."
- **Action items are systemic, owned, and dated.** "Be more
  careful" is not an action item. "Add a lint that catches
  this pattern, owned by Team A, due 2 weeks" is.
- **Distinguish near-misses from incidents.** A near-miss
  is a free lesson — celebrate it being reported. An
  incident is paid-for lessons — extract maximum value.
- **Schedule postmortems within 48 hours.** Memory fades;
  reconstruction gets harder. Soon-after-the-fact is the
  right moment.
- **Cross-team participation.** The team that caused the
  break + adjacent teams + sometimes representation from
  product / customer-success. Different perspectives, same
  document.
- **Publish broadly.** A postmortem hidden in a team's
  Notion helps that team. Published widely, it helps the
  whole org learn. (Redact customer-specific or
  security-sensitive details as needed.)
- **Follow up on action items.** A postmortem with
  un-completed actions a quarter later is the second
  incident's preamble. Track them.

## Common failure modes

- **The blame-laundering postmortem.** Uses the *vocabulary*
  of blamelessness but the *content* points at one person.
  "Mistakes were made" with implied culprit. Worse than
  honest blame.
- **The "be more careful" action item.** Means nothing. The
  next person won't be more careful either. Reformulate as
  a system change.
- **No follow-up.** Postmortems written, never re-read.
  Action items half-done. Reschedule the incident.
- **Postmortems only for big incidents.** Near-misses and
  small bumps are *cheaper* to learn from. Lower the bar.
- **One-person postmortems.** The engineer who caused the
  break writes the document alone. The view is narrow.
  Bring others in.
- **Treating "we should rewrite" as an action item.**
  Rewrite is a project, not an action. Break it into
  shippable steps.

## When the principle DOES NOT apply

There is essentially no case where blame is more useful than
systemic analysis. The narrow exception:

- **Repeated negligence by an individual.** If someone has
  caused the same kind of failure repeatedly *after* the
  system was improved, that's a different conversation —
  performance management, not postmortem. But that
  conversation is *separate* from the postmortem document
  and shouldn't contaminate it.

## Tagline

> The person was the trigger. The system was the cause.

Postmortems exist to change the system. People will keep
making mistakes; the system can stop letting one mistake
reach production.

## Sources

Distilled from SRE / DevOps tradition; echoes Google's *Site
Reliability Engineering* book, John Allspaw's blameless-
postmortem essays, and Etsy's pioneering culture in this
area.
