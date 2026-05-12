---
name: single-source-of-truth
description: Every piece of state should have exactly one authoritative owner. Duplicates drift. A constant defined in two places is two bugs waiting; a user record cached in three stores is three sync problems; a config value scattered across deployment scripts is impossible to update. The cure is one owner + derivation everywhere else.
version: 0.1.0
---

# Single Source of Truth

State without an owner becomes state with conflicting copies.
The day two copies disagree is the day someone has to be a
detective: which one was right? when did they diverge? who
last wrote each?

The principle: **for every piece of state, name exactly one
authoritative owner. Everything else derives from that owner,
or stores a snapshot with a clear "this is a copy" provenance.**

## When to invoke

- The same constant value is defined in two files.
- A user / customer / product record lives in three datastores
  and a sync job keeps them consistent.
- A piece of config is set in a `.env`, a Terraform variable,
  a CI secret, and a Kubernetes ConfigMap.
- A bug appears as "the data is wrong in the UI" but right in
  the API.
- A field has to be updated in N places when it changes;
  someone always forgets one.
- A duplicate has drifted and now you can't tell which copy is
  correct.
- A "sync script" exists to keep two systems in agreement; it
  has bugs and silently misses updates.

## The shape — three patterns

### 1. One owner, everything else derives

The classic shape. One module / table / service is the
authority. Other places that need the data call the authority
or render from a view of the authority's data.

- A `users` table is the authority on user records. The
  notifications service doesn't keep its own copy; it queries.
- A central `Stage` enum is the authority on stage names. The
  dashboard, the CLI, and the SKILL.md all reference *it*, not
  three separate copies.
- A single config file is the authority on environment values.
  Other systems read from it; they don't redeclare.

### 2. One owner + explicit cached copies

When derivation is too expensive at read time, cache — but
make the cache an explicit *copy* of the source, with
provenance.

- Search index over a database: the DB is authority; the
  index is a known-stale derivative refreshed by a pipeline.
- A cached read-model in a CQRS system: the event log is
  authority; the read-model is derived.
- A page render cache: the source data is authority; the
  rendered HTML is derived and invalidated on source change.

The rule: the cache is *known* to be derived, and there's a
*named mechanism* by which it refreshes. Drift is bounded.

### 3. One owner + replication with conflict policy

When you can't have a single owner (geo-distributed
databases, offline-first apps, distributed caches), you
need a *conflict resolution* policy that decides who wins.

- Last-writer-wins with timestamps. Crude but understandable.
- CRDTs (counters, sets, sequences that mathematically
  converge regardless of write order).
- Per-field ownership ("the inventory service owns stock_qty;
  the catalog service owns title").

This is harder than (1) or (2). Choose it only when forced.

## Why it matters

1. **Drift is silent.** Two copies of a value can disagree for
   weeks before anyone notices. The cost shows up as
   "mysterious bugs" in places far from the cause.

2. **Updates fan out.** A field that lives in five places is
   five places to update. Someone always misses one.

3. **Provenance becomes impossible.** When the data disagrees,
   you can't tell which copy is correct because there's no
   defined authority.

4. **Tests can't enforce consistency.** You can test that the
   authority is correct. You can't test that 47 scattered
   copies all match.

5. **Onboarding gets worse.** "Where do I set X?" — if the
   answer is "it depends, sometimes here and sometimes
   there," new contributors will pick wrong.

## Practical guidance

- **Name the owner explicitly.** In docs, in the schema, in
  the code comment: "The `users` table is the authoritative
  store for user identity." If you can't write that
  sentence, you don't have a single source of truth.
- **Derive everything else.** Other services / tables / files
  reference the owner, query it, or contain a clearly-labeled
  cache of it.
- **For constants and config: one file.** A constant should
  live in one module; everywhere else `import` it. A config
  value should live in one place (env var? config file?
  vault?); everywhere else look it up.
- **For data: one table.** Replicas for read scale are
  fine; multiple writeable tables for "the same" thing is a
  drift bomb.
- **Make derivation visible.** A view, a materialised view, a
  pipeline, a function — something with a name that says
  "this is computed from X."
- **Avoid sync scripts where possible.** A sync script is
  evidence that you have two sources of truth and you're
  fighting it. Better: one source + a derivation.
- **When schema changes, change the owner first.** Then
  derive. Don't update copies in parallel.

## Common failure modes

- **Two databases, "synced" by a cron.** The cron has bugs,
  races, missed runs. Drift accumulates. Pick one as
  authority.
- **Config in `.env`, `terraform.tfvars`, GitHub Secrets, AND
  Kubernetes ConfigMap.** Different deploy paths read
  different ones. Tracing "where did this value come from"
  is a half-day exercise. Pick one canonical source per
  config kind.
- **Constants duplicated across modules.** `MAX_RETRIES = 3`
  in module A and `MAX_RETRY = 5` in module B (note the
  typo + value drift). One day they should match; they
  don't.
- **"Just denormalise it for performance."** Then the
  denormalised copy drifts. Sometimes the right answer;
  sometimes paying a real performance cost for clarity is
  worth it. Choose deliberately.
- **Caching without invalidation.** Phil Karlton's two hard
  problems. If you cache, decide the invalidation policy
  *before* you cache, not after the first stale-read bug.
- **Distributed mutable state with no policy.** Just
  "everyone writes to it." You'll find out which write wins
  in production.

## When the principle DOES NOT apply (or is moderated)

- **Performance forces denormalisation.** Sometimes a read
  path is hot enough that you need a precomputed copy. Do
  it deliberately, with a named refresh path, knowing the
  drift window.
- **Genuinely distributed systems with offline mode.** A
  mobile app working offline can't query a central
  authority. You need CRDTs / conflict resolution by design.
- **Audit trails and event logs.** Append-only logs of
  "what happened" coexist with the current state. The log
  is authority for *history*; the projection is authority
  for *current state*. Two roles, not duplicates.

## Tagline

> Name the owner. Derive everything else.

If you can't say who owns a piece of state, nobody does —
and that's exactly when it drifts.

## Sources

Distilled from general engineering practice; echoes
"don't repeat yourself," the relational-normalisation tradition,
and CQRS event-sourcing literature.
