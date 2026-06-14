---
name: idempotent-by-default
description: Design operations so that retrying them is safe. A POST that creates a duplicate on retry is a bug. Networks fail, queues redeliver, jobs restart, users double-click. The way to survive is to make every operation either truly read-only or carry an idempotency key that the receiver dedupes against.
version: 0.1.0
---

# Idempotent by Default

In distributed systems, retry is not optional — networks fail,
timeouts fire, queues redeliver, jobs restart, users double-click.
The question isn't *whether* an operation will be retried; it's
whether your system *survives* the retry.

The principle: **every mutating operation should be safe to invoke
twice.** Either the operation is naturally idempotent (PUT a known
resource to a known state) or it carries an idempotency key that
the receiver uses to detect and dedupe duplicates.

## When to invoke

- The word *"retry"* appears anywhere in the design — at the
  client, in a queue consumer, in a circuit breaker.
- A POST creates a duplicate row when the client retries after
  a timeout.
- A queue's at-least-once delivery is double-charging customers.
- A scheduled job catches up after an outage and re-runs items
  it already ran.
- A user double-clicks "Pay" and gets two charges.
- A migration script can't be re-run safely.
- A workflow that "almost completed" can't resume from where it
  left off because the partial state is uninspectable.

## The shape — three flavors of idempotency

### 1. Naturally idempotent

The operation's *meaning* is "set state to X" — running it twice
has the same effect as running it once.

- HTTP **PUT** of a known resource: `PUT /users/42 { ... }`
- "Set status to closed-fixed" — idempotent by construction.
- Setting a feature flag value.
- Writing a file with a known name and known contents.

When you can shape your operations as "put the world into state
X," idempotency is free.

### 2. Idempotency-key-based

The operation has side effects that *do* duplicate (charge,
send email, insert row), so you can't naturally re-run. The
caller supplies a unique key with the request; the receiver
remembers it and refuses duplicates.

- Stripe's `Idempotency-Key` header pattern. Caller generates
  a UUID per logical operation; server stores
  `(key, result)` for some window; on retry the stored result
  is returned without re-executing.
- Workflow IDs: each step in a workflow has a deterministic
  key built from `(workflow_id, step_name)`. The worker
  checks "have I done step X of workflow Y" before
  performing the side effect.

### 3. Compensable

When the operation can't be made idempotent (sending a
physical SMS, triggering a refund), pair it with a
compensating action and a saga / outbox pattern. The system
can re-enter a state machine and unwind partial progress.

## Why it matters

1. **Networks are unreliable.** Every HTTP call can time out
   *after* the server processed it. Without idempotency, the
   client can't safely retry — it doesn't know if the
   operation went through.

2. **Queues are at-least-once.** SQS, Kafka, Pub/Sub all
   guarantee *at-least-once* delivery as their default. Your
   consumer WILL see duplicates. Idempotency is how it
   tolerates them.

3. **Restart is normal.** Pods get killed, jobs restart,
   schedulers re-execute. If your job catches up after an
   outage and replays Friday's batch, idempotency is what
   makes that recovery safe.

4. **Users are unreliable.** A double-click, a "Submit"
   pressed twice, a flaky network on the user's end —
   idempotency keys mean the second click doesn't create
   the second order.

## Practical guidance

- **Prefer PUT over POST when the operation has a known
  target state.** PUT is idempotent by HTTP convention; POST
  is not. Adopt the convention.
- **Use idempotency keys for POSTs that have side effects.**
  Accept a header like `Idempotency-Key`; on receipt, look up
  the key; if found, return the previous result; if not,
  execute and record `(key, result)`.
- **Deterministic keys are better than random ones.** A key
  built from `(user_id, operation, target)` lets the *server*
  notice duplicates even if the client retries with a fresh
  random key. Random is fine if all retries reuse the same
  one.
- **Cap the dedup window with a TTL.** You can't remember
  every idempotency key forever. 24 hours, 7 days, 30 days —
  pick a TTL that's longer than any plausible retry.
- **Make queue consumers idempotent.** When pulling from a
  queue, derive an idempotency key from the message
  contents (often a hash of relevant fields) and check it
  before applying side effects.
- **Use transactions to atomically check + write.** "Did I
  already record this key? No → record it AND apply the side
  effect, in one transaction." Otherwise two concurrent
  retries both check, both find no record, both apply.
- **Log when duplicates are detected.** Each dedup is a
  signal — your retries are working as designed. High dedup
  rates may mean a stuck retry loop somewhere.

## Common failure modes

- **POST-then-retry.** Client times out; retries; server
  processes both. Now you have two orders.
- **At-least-once consumer that's not idempotent.** Kafka
  redelivers; the consumer double-applies. Charges show up
  twice; emails send twice.
- **"It almost always works."** The idempotent path exists
  but only for the happy case; the failure path has a
  side-effect race. Test the retry path explicitly.
- **Side effect before the dedupe check.** Charge the card,
  THEN check if we've seen this key. Out of order. Dedupe
  first; act second.
- **Storing the response, not the result.** If you cache the
  HTTP response body, an upgrade that changes the response
  shape breaks retries. Cache the *result* (the resource
  id, the outcome enum); recompute the response shape on
  read.
- **Forgetting cleanup on the failure path.** A migration
  script runs halfway, fails, leaves the system in a
  half-state that can't be re-entered. Either be transactional
  or be resumable.

## When the principle DOES NOT apply (sort of)

- **Pure-read operations.** GETs are naturally idempotent;
  no extra mechanism needed.
- **One-shot scripts you run by hand.** A migration script
  you run once doesn't *need* idempotency, but you'll regret
  it the day you have to re-run after a partial failure.
  Default to writing them resumable anyway; the cost is
  small.
- **Operations that are fundamentally not safe to repeat.**
  Sending a missile, irreversibly publishing a press
  release. Then you need *exactly-once* semantics, which
  combines idempotency *and* coordination (workflow engines,
  sagas, two-phase commits).

## Tagline

> Retry is not optional. Design for it.

If your operation breaks when invoked twice, your system
breaks the first time the network blinks.

## See also

- [MakeTheWrongThingHard](../MakeTheWrongThingHard/SKILL.md)
  — idempotency is the encoded version of "retry is
  safe"; the wrong thing (double-effect) becomes
  structurally hard rather than a discipline the caller
  has to remember.
- [FeatureFlagsAreInfrastructure](../FeatureFlagsAreInfrastructure/SKILL.md)
  — the rollback path a flag provides assumes the
  underlying operations tolerate re-application;
  without that, flipping back is its own incident.
- [SingleSourceOfTruth](../SingleSourceOfTruth/SKILL.md)
  — duplicate side-effects are state drift in another
  shape; idempotency keeps a single record of truth
  even under at-least-once delivery.
- [TheBisectMindset](../TheBisectMindset/SKILL.md) —
  reproducibility across retries is a precondition for
  useful diagnosis; a non-idempotent operation makes
  every reproduction a different system.

## Sources

Distilled from general engineering practice; echoes Stripe's
idempotency-key pattern, the at-least-once delivery semantics
of every major queue, and the saga / outbox / workflow
literature.
