---
name: boring-tech-where-possible
description: At the bottom of your stack, pick mature, well-supported tools — Postgres, SQLite, Redis, Linux, established languages and frameworks — and reserve your novelty budget for the actual product. Mature tools come with manuals, hires, war stories, and bounded surprise. Load when choosing a foundational tool (database, queue, language, framework, OS), when tempted to adopt a months-old framework as a service's foundation, when someone proposes a rewrite in the language of the month, or when a production incident traces to an obscure dependency with no Stack Overflow hits.
version: 0.1.0
---

# Boring Tech Where Possible

You have a fixed novelty budget. Spend it on the thing your
company actually does, not on the foundation under it. The
database, the queue, the OS, the language, the framework: pick
boring, mature, well-supported options where you can. Save the
exotic choices for the layer where exoticism is the value.

The principle: **at the bottom of your stack, mature beats
new.** A 30-year-old database is supported by ten thousand
articles, dozens of consultancies, every cloud provider, and a
deep bench of hires. A 2-month-old database is supported by
its three core developers' Discord.

## When to invoke

- You're about to adopt a 2-month-old framework as the
  foundation of a new service.
- The plan calls for a new datastore where Postgres / SQLite /
  the existing store would do.
- A teammate says "let's rewrite in $LANGUAGE_OF_THE_MONTH."
- Onboarding requires learning three custom tools nobody
  outside the team has heard of.
- A production incident traces to an obscure dep with no
  Stack Overflow hits.
- Hiring is hard because nobody on the market has used your
  stack.
- A library you depend on just announced "we're rewriting
  from scratch for v3."
- Most of your build / dev tooling is bespoke.

## The shape — what "boring" looks like by layer

| Layer | Boring choice (often defensible) | Novel choice (more risk) |
|---|---|---|
| **Datastore** | Postgres, SQLite, MySQL | Custom-built KV; bleeding-edge graph DB; 12-month-old SaaS |
| **Cache** | Redis, Memcached | Custom in-process cache with bespoke eviction |
| **Queue** | Postgres `LISTEN`/`NOTIFY`, Redis, SQS, RabbitMQ | New eventing fabric not yet in production at scale |
| **Language** | Python, Go, TypeScript, Java, C# | A six-month-old language for the core service |
| **Framework** | Boring framework with 10+ years of usage | Framework that fundamentally restructures every year |
| **OS** | Linux (Debian, Ubuntu, Amazon Linux, …) | Niche distro nobody uses |
| **CI** | GitHub Actions, GitLab CI, Jenkins | Bespoke CI runner project |
| **Editor / IDE** | Whatever your team uses | Mandating an unusual one |

Counter-table: places novelty is the *value* and should be
budgeted for:

- The product itself
- The algorithms specific to your domain (ML model, compiler,
  consensus, scheduler)
- The user experience
- A genuinely new capability not the platform can offer

## Why it matters

1. **Operational tail.** Boring tech has known failure modes
   you can search. New tech has unknown failure modes you
   discover at 3am.

2. **Hiring + onboarding.** "We use Python and Postgres" is a
   sentence anyone can act on. "We use $obscure_lang on
   $obscure_runtime" filters your candidate pool to almost
   nobody.

3. **The internet helps.** Stack Overflow, blog posts, vendor
   support, conference talks, books. None of that exists yet
   for the new tool.

4. **Surprises shrink.** Mature tools have been pushed in every
   direction. Edge cases you'd find with novel tech are
   already documented.

5. **You can spend the saved attention on the actual problem.**
   Every hour debugging the bespoke event bus is an hour not
   building the thing customers pay you for.

## Practical guidance

- **Default to boring; let the requirement *force* novelty.** If
  Postgres would work for the workload, use Postgres. Reach for
  the new option only when boring genuinely doesn't fit.
- **"It's mature when its conference is mature."** A loose rule
  of thumb: tech with a long-running annual conference has been
  used in production at enough scale that the war stories are
  shared. The conference is downstream evidence of maturity.
- **Count years, not enthusiasm.** A tool that's been in production
  at multiple companies for five years is qualitatively different
  from one with active GitHub stars and no users.
- **Read the issue tracker before adopting.** Public bug
  patterns, response time, and triage rigor tell you what
  living with this tool will be like.
- **Stagger your novelty.** If you must use a new database, pick
  a boring language. Don't stack novelty in three layers at
  once.
- **Be honest about novelty's cost.** That cost is real and you
  pay it every quarter for years. Compare it to the *win*
  you get from novelty.

## Common failure modes

- **Resume-driven development.** Adopting a tool because the
  team wants to learn it. Learning happens; so does the on-call
  pager. Decouple learning from production.
- **The pendulum swing.** "We had bad experiences with X, let's
  use Y." Sometimes Y is just newer and you'll have the same
  bad experiences with Y in two years. Diagnose what actually
  went wrong before swapping.
- **"It's web-scale."** A blog post extolling a tool that
  Google or Meta uses doesn't mean it fits your team-of-five.
  Their problems are different.
- **Novelty in the wrong layer.** A novel database under a
  boring web app — high risk for low reward. A boring database
  under a novel ML pipeline — appropriate.
- **Sunk-cost stickiness.** A tool you adopted three years ago
  hasn't matured. Boring isn't the same as legacy. If
  something *stayed* obscure, it might still be the wrong
  choice.

## When the principle DOES NOT apply

- **The novel thing IS the product.** If you're a database
  company, you obviously build a novel database. If you're a
  compiler company, you write a novel compiler. The "stack"
  layer where novelty pays off is the product layer.
- **An order-of-magnitude advantage exists and you've
  verified it.** Sometimes a new tool genuinely solves a
  problem the old ones can't (a real-time database for an
  inherently real-time product). When the math works, novelty
  is correct — but verify the math, don't take the marketing.
- **You explicitly want to learn.** Side projects, sandbox
  experiments, R&D — these are where you spend the learning
  budget. Just don't conflate them with production.

## Tagline

> Choose boring underneath, novel on top.

The novel thing should be the thing your business *does*, not
the foundation under it.

## See also

- [DontReinventTheWheel](../DontReinventTheWheel/SKILL.md) —
  the top-of-stack counterpart; both are leverage
  disciplines that ask which work is *yours* to do and
  which has already been done by someone with more time
  to do it well.
- [PolishWhenLoadBearing](../PolishWhenLoadBearing/SKILL.md)
  — when to invest in your foundations at all; choosing
  mature tools defers that question by inheriting
  someone else's polish.
- [FormalVsImprovisational](../FormalVsImprovisational/SKILL.md)
  — boring foundations are precisely what lets the
  product layer above move improvisationally without
  the substrate moving underneath it.
- [ReadTheSourceFirst](../ReadTheSourceFirst/SKILL.md) —
  mature tools have readable, vetted source you can
  actually navigate when the abstraction leaks.

## Sources

Distilled from general engineering practice; echoes Dan
McKinley's "Choose Boring Technology" essay and the "Innovation
Tokens" framing — each team has a small number of tokens to
spend on novelty per project.
