---
name: logs-are-a-feature
description: Treat production logging as a deliberate feature, not as `printf` left behind. Structured (key-value or JSON) so they're queryable; designed at module boundaries so a future incident has signal; correlation IDs threaded through requests; levels used consistently; sensitive data redacted at the source. Logs are the production debugger — design them with the care you'd give an API.
version: 0.1.0
---

# Logs Are a Feature

In a debugger you have breakpoints, locals, a stack. In production
you have the logs you wrote yesterday. That asymmetry means
production logging deserves *design*, not afterthought.

The principle: **logs are the production debugger. Design them
with the same care you'd give to a public API.** Structured,
queryable, correlated, levelled, redacted.

## When to invoke

- A production incident just happened and the only log message
  is `"error"` with no context.
- You can't tell *which user* triggered the bug from the logs.
- You can't tell *which request* a log line belongs to —
  multiple requests' logs interleave.
- Logs are full of `console.log("here")` and
  `console.log(x)` from someone's debugging session, still in
  the codebase months later.
- Logs leak credentials, PII, or full request bodies.
- A new service is being written and "we'll add logging later."
- Log volume is so high it costs real money but the signal-
  to-noise ratio is near zero.
- A developer asks "what's going wrong?" and the answer requires
  a deploy to add a log line.

## The shape — design dimensions

### 1. Structured, not stringly

Bad:
```
INFO Request from user 42 to /api/orders took 318ms
```

Good (JSON line):
```json
{"ts":"2026-05-12T09:14:33Z","lvl":"info","msg":"request",
 "user_id":42,"path":"/api/orders","method":"GET",
 "duration_ms":318,"trace_id":"abc-123"}
```

Why: you can query "all requests over 1000ms" without parsing
prose. Tools (jq, Athena, ELK, Datadog, CloudWatch Insights) all
speak structured logs natively.

### 2. Correlation IDs threaded through

Every request gets an id (generate at the edge if not provided;
propagate via header `X-Trace-Id` to downstream services).
Every log line in that request carries the id. Then "show me
everything that happened during request abc-123" is one query
across many services.

### 3. Levels used consistently

| Level | Reserved for |
|---|---|
| **TRACE** | Verbose; for active debugging. Off in prod. |
| **DEBUG** | Diagnostic detail. Off in prod unless investigating. |
| **INFO** | Business events — request received, job started, user signed up. |
| **WARN** | Recoverable anomalies — retry succeeded, fallback used. |
| **ERROR** | Real failures — exception, lost write, expired token. Each ERROR should be actionable. |
| **FATAL** | Process is about to die. Always actionable. |

Consistency matters more than the exact taxonomy. If WARNs are
"things I want to look at later" and ERRORs are "wake someone
up," the alerts and dashboards can be built around that split.

### 4. Module boundaries, not function bodies

Log at the *boundary*: when a request arrives, when a job
starts, when an outbound call is made, when a result is
returned. Logging at every line of every function is noise;
logging at boundaries gives a request's whole trajectory.

### 5. Redaction at the source

Passwords, tokens, full credit-card numbers, PII (depending on
domain) should never be logged. Redact at the point of
formatting, not downstream. A `[REDACTED]` placeholder is
better than the field being silently dropped (then you can see
it *exists* but not the value).

## Practical guidance

- **Pick one library + one format for the whole codebase.**
  Different microservices using different log formats is a
  forensic nightmare during an incident.
- **Always include a stable identifier** in error logs — the
  user id, the request id, the resource id. "It failed" with no
  identity is unactionable.
- **Don't log what you can't change.** If logging at INFO level
  costs $5K/month and 99% is noise, drop the level or sample.
  Pay only for signal.
- **Test the alerting paths.** Periodically trigger a synthetic
  ERROR / FATAL in non-prod and confirm the alert fires. Logs
  you can't observe are logs that don't exist.
- **Put the WHY in error messages.** "Failed to write" is
  worse than "Failed to write user.id=42 to users table:
  unique-constraint email."
- **Logs are append-only narrative; metrics are aggregate
  signal.** Don't try to replace metrics with logs (they
  cost more, they're slower to query, they don't trend).
  Likewise don't replace logs with metrics (you lose
  per-event detail).
- **Make `printf` debugging a tooled habit.** Local-only
  TRACE / DEBUG levels are fine and useful; remove them from
  PRs unless they're permanent additions worth keeping.

## Common failure modes

- **String concatenation at the call site.**
  `logger.info("user " + id + " did thing")` defeats
  structuring. Use the logging library's structured form
  (`logger.info("user did thing", user_id=id)`).
- **No correlation id.** Multi-service requests are
  un-untangleable. Add the id at the entry edge and pass it
  everywhere.
- **Levels-as-mood.** WARN used for "this is unusual" not
  "this is recoverable but should be looked at." When the
  meaning of levels drifts, alerts based on them stop
  working.
- **Sensitive data leakage.** A logged request body that
  contains a password ends up in your log store, your
  backup, possibly Datadog. Redact at the source.
- **Logging without a sink.** Logs written to a file on a
  container that's destroyed at shutdown. Ship them to a
  central store, or they're not logs.
- **Verbosity for verbosity's sake.** Logs that say "entered
  function" / "exited function" for every function. Noise,
  cost, no signal.

## When the principle DOES NOT apply (less)

This is one of the few skills with no real exceptions. Even
throwaway scripts benefit from `print("processed:", n)` over
`print()`. Even small services need correlation. The
*intensity* of investment scales with how important production
behaviour is, but the principle holds.

The one nuance: **at the very smallest scale** (a CLI tool a
single user runs), `stderr` with prose is fine. Don't engineer
a structured-logger for a one-page script.

## Tagline

> The logs you wrote yesterday are tomorrow's debugger.

Design them while you have the time. You won't, during the
incident.

## See also

- [PostmortemsWithoutBlame](../PostmortemsWithoutBlame/SKILL.md)
  — logs are the forensic fuel a postmortem runs on;
  without them, the postmortem is reconstructed
  guesswork.
- [TheContractIsTheArtifact](../TheContractIsTheArtifact/SKILL.md)
  — a log stream is a durable contract that outlives
  the service writing it; downstream consumers depend
  on its shape.
- [MakeTheWrongThingHard](../MakeTheWrongThingHard/SKILL.md)
  — redaction and structured fields at the logger
  layer make logging a secret structurally hard rather
  than a code-review discipline.
- [SingleSourceOfTruth](../SingleSourceOfTruth/SKILL.md)
  — correlation IDs are what make a single event
  traceable across many systems instead of fragmenting
  into per-service narratives.

## Sources

Distilled from general engineering practice; echoes "12-Factor
App" (logs as event streams), OpenTelemetry (correlation IDs),
and SRE principles.
