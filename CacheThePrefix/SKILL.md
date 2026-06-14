---
name: cache-the-prefix
description: When expensive preparation is needed before useful work begins — loading skills and plugins into an AI session, hydrating a dev environment, warming a JIT, fetching a dependency graph, rebuilding a container — the first instinct is *prepare once and clone*. But nearly every layer of the modern stack amortizes setup cost differently — through **prefix caching** — identical inputs reuse the previously-computed result, automatically. Anthropic's prompt cache, Docker's layer cache, Bazel's action cache, Nix's binary cache, the CDN edge, npm/pnpm content stores, the HTTP cache, the kernel page cache, V8 code caches, browser persistent sessions — they all share one shape — hash the prefix, key the cached output by that hash. The engineering move is to make your setup deterministic and prefix-shaped so the substrate's existing cache catches it, not to demand a snapshot or clone primitive the substrate doesn't have. Load this skill when an actor (especially an AI agent) asks for a *clone the session* / *snapshot the env* primitive, when expensive setup is repeating across many sessions or runs, when a system's cold-start cost dominates total work, or when designing the loading order of a system that will be invoked repeatedly.
version: 0.1.0
---

# Cache the Prefix

> "Can I prepare a Claude Code session — load all my skills and
> plugins — and then *clone it*, so the next session doesn't have to
> redo the setup?"

The literal answer is usually no. Most user-facing systems don't have
a "fork-this-running-process-and-keep-its-state" primitive. But the
question is worth taking apart, because the instinct behind it — *the
prepared state is expensive; I shouldn't pay for it twice* — is
right. The substrate almost always *does* amortize the cost. It just
doesn't do it by cloning. It does it by **caching the prefix**.

A second actor starting from the same setup does not re-run the
setup. It pays the *cache-hit* price on the bytes it shares with the
first actor, which is typically an order of magnitude cheaper than
the cache-miss price. The amortization is real; the mechanism is
not the one the user reached for.

This pattern repeats at every layer of the modern stack. Once you
see it, the urge to invent your own clone / snapshot / save-state
primitive fades — most of the time, the cache is already there. The
engineering work is to make your setup *cacheable*: deterministic and
prefix-shaped, so the existing cache machinery catches it.

## When to invoke

- An AI agent (or you) wants to "clone the session," "fork the env,"
  or "snapshot the runtime" before doing N more runs from the same
  starting point.
- An expensive initialization step is repeating across many short-
  lived actors — CI jobs, test runs, lambda cold starts, parallel
  AI sessions.
- A build is slow and the slowness is the *same* every time, not
  workload-dependent.
- Someone proposes to checkpoint a process, save a VM image, or
  dump-and-restore memory because re-initializing is "too slow."
- A workflow is dominated by setup time and the user is paying for
  it on every invocation.
- You're designing a system whose users will instantiate it many
  times in quick succession.
- The phrase "warm start" appears in the architecture but no caching
  layer exists to make a start warm.
- A new abstraction is being designed (a "session," a "context," a
  "workspace") — decide *now* what its prefix looks like and where
  it gets cached.
- A teammate is about to build their own session-checkpoint /
  snapshot system without checking whether the substrate's cache
  already amortizes the cost.

## The shape

Across the stack, the same shape recurs: **a deterministic prefix
produces a hash; the hash keys a cache of the expensive output;
identical prefixes get cache hits.**

| Layer | Prefix | Cache | What it amortizes |
|---|---|---|---|
| Anthropic prompt cache | First N tokens of the request (system prompt, tool defs, skill descriptions) | Ephemeral KV cache, ~5-min TTL (1-hour beta) | Tokenization + KV-warming for repeated prefixes; ~10× cheaper reads |
| Docker / OCI | Sequence of `RUN` / `COPY` / `ADD` instructions | Layer cache (local + registry) | Re-running build steps that haven't changed |
| Bazel | Hash of action inputs (sources + flags + toolchain) | Action cache (local + remote) | Recompiling unchanged code |
| Nix | Derivation hash (input-addressed) | Binary cache | Re-deriving identical store paths |
| npm / pnpm / bun | Lockfile + package tarball hash | Content-addressed global store | Re-fetching + re-extracting the same package |
| HTTP | URL + `Vary`-relevant headers | Browser cache, CDN, edge | Re-fetching the same resource |
| Filesystem | Block address | Page cache | Re-reading the same disk blocks |
| JIT / V8 / HotSpot | Bytecode hash | Code cache | Re-JITting the same hot function |
| Linker / ThinLTO | Module summary hash | Module cache | Re-linking identical modules |
| Browser persistent sessions ("Persist Preview") | Origin + cookie store | Persistent profile dir | Re-authenticating on every dev-server restart |

The variations are in encoding (token IDs, content hashes,
instruction sequences) and in TTL (milliseconds for HTTP, minutes
for Anthropic, hours-to-weeks for npm, until-eviction for Bazel).
The principle is the same.

**The two questions the principle answers:**

1. *Will the substrate's cache catch my setup?* Yes if your prefix
   is identical byte-for-byte to a recent prior run. No if anything
   in the prefix shifts — even something you didn't think mattered,
   like an embedded timestamp, a `RUN apt-get update` that fetches
   different bytes each time, or a per-host path in a skill
   description.
2. *What's the prefix?* The first N inputs the substrate processes
   before workload-specific input begins. Designing for cacheability
   means putting all stable setup at the *front* and all variable
   workload at the *back*.

## Why it matters

1. **The instinct to clone state is often a UX gap, not a missing
   feature.** When the user feels "I want to clone this prepared
   session," there is usually already a cache amortizing ~90% of
   the cost — but they can't *see* it. They can't see the cache-hit
   token counts, the layer-reuse percentages, the action-cache hit
   rate. So they reach for a heavier mechanism than the substrate
   needs. The right intervention is often visibility, not new
   plumbing.

2. **Prefix caches compose across layers; snapshots don't.** Every
   layer of a stack tends to cache its own prefix. A Bazel build
   inside a Docker container inside a CI runner gets cache hits at
   all three layers if each layer's prefix is stable. Snapshots are
   usually opaque to layers above and below — a VM snapshot doesn't
   help the inner build cache, and an inner snapshot doesn't help
   the outer orchestrator.

3. **Caches reward determinism — and determinism is independently
   valuable.** Making your setup cacheable means making it
   deterministic, which means making it reproducible, which means
   making it debuggable, which means making it portable. The
   cacheability work pays in many directions at once.

4. **Caches invalidate cleanly; snapshots go stale silently.** Change
   one byte of the prefix and every cache below it misses —
   automatically, no human intervention. Take a snapshot, change the
   world, and the snapshot is still there pretending to be fresh
   until someone remembers to refresh it. Content-addressed
   invalidation is self-correcting; identity-addressed snapshots are
   not.

5. **Caches let many actors share without coordinating.** A prefix
   cache is read-mostly: the first actor pays the miss cost; every
   subsequent actor with the same prefix pays the hit cost. The
   actors do not have to know about each other. They do not have to
   coordinate. They do not need a shared "session manager." This is
   exactly the property you want when N AI agents, M CI jobs, and K
   developer laptops are all spinning up against the same setup.

6. **The substrate's cache is built and operated for you.** Anthropic,
   Docker, Bazel, npm, the kernel, the browser — all have invested
   years in cache correctness. Rolling your own clone/snapshot
   mechanism is rolling your own cache, *worse than the one already
   underneath your code*.

## Practical guidance

- **Put the stable parts first.** In any sequence the substrate
  hashes top-to-bottom, your setup should be a long stable prefix
  followed by a short variable suffix. Docker: base layers first,
  source last. Anthropic prompts: system prompt + tool defs + skill
  descriptions first, the per-turn user message last. Bazel:
  toolchain + flags first, sources second. Inverting this defeats
  the cache: a variable byte at position 1 invalidates everything
  below it.

- **Make the prefix byte-stable.** Strip embedded timestamps,
  randomized comments, "current user" fields, system-specific paths.
  Pin dependency versions in lockfiles. Use deterministic compilers
  and packagers. Two runs that *should* produce identical prefixes
  must produce *literally identical* byte sequences for the cache
  to fire.

- **Surface cache-hit signals.** Whatever metric your substrate
  exposes — `cache_read_input_tokens` in Anthropic responses,
  `CACHED` annotations in Docker, action-cache hit count in Bazel,
  `X-Cache: HIT` headers in CDN responses — bring it to the surface
  so users (and you) can see the amortization working. Invisible
  caches inspire phantom needs for cloning.

- **When designing a new system, design its prefix.** What is the
  longest stable thing every invocation will share? Make that the
  leading input. Caches you don't even know exist yet — your users'
  filesystems, future CDNs, future LLM prompt caches — will then
  catch it.

- **Resist building a snapshot / clone primitive before you've
  checked whether the existing cache solves it.** Snapshots are
  heavy plumbing; caches are usually already running. The order of
  attack is: (a) is there a cache? (b) is my prefix stable enough to
  hit it? (c) can I make it more stable? (d) only then, snapshot.

- **For AI agents specifically: trust the prompt cache.** Before
  worrying about a heavy session-init cost being paid N times, check
  what is actually loaded. Skill *descriptions* are in the system
  prompt (small); skill *bodies* load on Skill-tool invocation
  (lazy). The cache amortizes the rest at ~10× the cache-hit /
  cache-miss ratio. Parallel agents spawned from the same parent
  inherit the parent's context directly — no caching needed because
  no second model call.

- **When two prefixes "should be the same" but aren't, hunt the
  divergence.** Cache misses on what you expected to be hits are
  almost always hidden non-determinism: an embedded timestamp, a
  build-host name, a system random seed, a floating-point ordering,
  a hashmap iteration order. Fix the divergence; the cache fires.

## Common failure modes

- **Embedding timestamps in the prefix.** A `RUN date >
  /etc/build-time` busts every downstream layer on every build.
  Same in Bazel actions, in tokenized prompts that include "today's
  date," in package builds that stamp `__BUILD_TIME__` into the
  binary. Keep timestamps out of cacheable prefixes — or, if they
  are load-bearing, push them past the cacheable region into the
  variable suffix.

- **Variable input before stable input.** Putting per-request user
  data at the *top* of an LLM prompt, or `COPY . /app` before
  `RUN pip install`, defeats the cache because every request has a
  different first byte. Re-order so the stable input is the prefix.

- **Re-rolling the cache because "ours will be better."** A team
  builds a custom session-state-checkpointing system, when
  Anthropic's prompt cache (or Docker's layer cache, or Bazel's
  action cache) was already doing 90% of what they wanted. The
  cache they wrote has fewer person-years of correctness work in it
  than the one underneath. Almost always a mistake.

- **Treating a cache as a snapshot.** A cache promises *if I see
  this prefix again, I'll serve the same answer*. It does not
  promise *I will hold this state for you indefinitely*. TTLs are
  short (Anthropic prompt: 5 minutes; HTTP: depends; layer caches:
  garbage-collected). If the design needs durability, layer a
  snapshot mechanism on top — but don't conflate the two.

- **Hoping for cache hits across machines without a remote cache.**
  Bazel's action cache is local by default. Bazel's full power
  requires a *remote* action cache so a teammate's or CI's build can
  hit cache from your work. Same with Docker (a registry-side layer
  cache) and Nix (a binary cache). Cross-actor amortization needs a
  cross-actor cache.

- **Inventing a "warm pool" of pre-initialized workers when the
  cache would do.** Lambdas, serverless runtimes, dev-environment
  hot pools — sometimes correct, often a complexity trap that exists
  because the cache wasn't designed in. If cold start is dominated
  by deterministic prefix work, fix the prefix-cache path before
  paying for an idle warm pool.

- **AI-agent-specific: assuming each new session pays the full
  prompt cost.** It might not. Check `usage.cache_read_input_tokens`
  on the response. If two sessions hit the same project within the
  cache TTL, the second mostly cache-hits. Don't reach for a fork
  primitive until you've measured the actual cost.

## When the principle does NOT apply

- **State that isn't deterministic from inputs.** A running process
  holds open file descriptors, established TCP connections, learned-
  at-runtime data, RNG state. None of that is a function of a stable
  prefix; caching can't reconstitute it. If you need *that*, you
  need a snapshot or migration mechanism — `fork(2)`, VM live
  migration, CRIU checkpoint/restore.

- **Workloads where the variable part dominates the cost.** If 95%
  of the cost is in the per-request workload and 5% is the prefix,
  optimizing prefix-cache hit rate doesn't move the needle. Profile
  first.

- **One-shot tasks.** A computation that runs exactly once doesn't
  benefit from caching its prefix. Cache discipline matters when the
  prefix will be hit again.

- **When the substrate has no cache.** Some layers genuinely don't
  cache. A raw Postgres connection has no prefix cache; a bare TCP
  stream has no prefix cache. If amortization is required and the
  layer doesn't provide it, either add a cache (Redis, a content-
  addressed store) or build a snapshot mechanism. The principle is
  *check first*, not *caching always wins*.

- **When the work genuinely is a snapshot.** Debugger
  record-and-replay, CRDT vector-clock checkpoint, save-game state,
  forensic capture of a running incident — these are legitimately
  snapshot problems. The principle doesn't outlaw snapshots; it says
  check whether the cache already gets you there before building
  one.

## Worked example — 2026-05-19

A user driving HARP — a Claude-Code-based research platform — loads
many skills, plugins, and methodology files into every session. The
question:

> *"I load a lot of state into a session. Many skills and plugins.
> Is it possible to do something like prepare a session and then
> clone it?"*

The literal answer: Claude Code supports forking — primitives that
copy the conversation log into a new session ID so both branches
start from the same prepared history. It is fork, not clone — each
branch gets its own context window from that point on. Useful in
some cases, but not what the user actually needed.

The *expensive* part of the setup is not the conversation log (which
is plain text and small). It is the *first model call* that has to
ingest the system prompt + all the tool definitions + all the skill
descriptions into the KV cache. That ingest is what feels heavy.

And that ingest is already cached. Anthropic's prompt cache catches
the prefix (~150 KB of system prompt + skill descriptions + tool
defs) on the first request, then serves subsequent requests from
the same prefix at ~10% of the original input-token cost for the
next 5 minutes (1 hour on the Max plan). A second session opened
in the same project within that window pays the cache-hit price on
the entire prefix. The `usage.cache_read_input_tokens` field on the
response confirms it.

There was a second amortization the user hadn't seen: **skills load
lazily.** Only the name + description of each skill (a few hundred
bytes) is in the initial system prompt. The full SKILL.md body is
loaded only when the agent invokes the Skill tool. So the "heavy"
startup the user worried about was mostly small descriptions, not
skill bodies. The cost they were trying to avoid by cloning was not
where they thought it was — and what cost remained was being
amortized by a cache they couldn't see.

**The instinct was clone-the-state. The actual mechanism is cache-
the-prefix.** Once that flipped, the engineering work was different:
not "build a session-snapshot primitive" but "make the prefix as
long, stable, and cacheable as possible." Concretely:

- Put global config and skill descriptions *before* per-task content
  in the system prompt, so the cacheable region is maximized.
- Avoid timestamps and per-machine paths in skill descriptions, so
  two machines hit the same cache key.
- For parallel work, prefer the Agent tool (subagents inherit the
  parent context directly, no second cache traversal) and git
  worktrees over the same `.claude/` config (shared cache window).
- Surface `cache_read_input_tokens` somewhere visible so the
  amortization is legible to the operator.

The user did not need a clone primitive. They needed a clearer
picture of the cache already amortizing the cost.

The same shape applies elsewhere in HARP and beyond. Docker layer
caches catch unchanged build steps. Bazel's action cache catches
unchanged compilations. npm's content store catches unchanged
tarballs. The CDN edge catches unchanged URLs. Browsers' persistent
profiles catch unchanged session state across server restarts (the
"Persist Preview" feature in Claude Code Desktop is exactly this
pattern at the preview-browser layer). Each level of the stack has
a prefix cache; the engineering work is to feed it a deterministic
prefix.

## Tagline

> The instinct is *clone*; the mechanism is *cache*. Don't snapshot
> what the substrate already amortizes — make your prefix
> deterministic and let the cache hit.

## See also

- [`SingleSourceOfTruth`](../SingleSourceOfTruth/SKILL.md) — one
  canonical authoritative copy of state. This skill complements it
  for the *prefix* dimension: one canonical, deterministic,
  cacheable setup that many actors share without having to
  coordinate.
- [`IdempotentByDefault`](../IdempotentByDefault/SKILL.md) —
  operations safe to repeat. Closely related: idempotency makes
  operations safe to *re-run*; determinism makes them safe to
  *cache*. Both are properties that pay off in distributed and AI-
  agentic systems.
- [`BoringTechWherePossible`](../BoringTechWherePossible/SKILL.md) —
  prefix caches built into mature substrates *are* boring tech. The
  prompt cache, the Docker layer cache, the page cache are decades
  of work to lean on rather than reinvent.
- [`CausalDivergence`](../CausalDivergence/SKILL.md) — about what
  *differs* across actors and how to make the disagreement legible.
  This skill is its counterpart: what is *identical* across actors
  and how to amortize the identical part. The two are duals — a
  system has a stable shared prefix and a divergent actor-specific
  suffix; one needs caching, the other needs reconciliation.
- [`DontReinventTheWheel`](../DontReinventTheWheel/SKILL.md) — same
  instinct one level up. Prefix caching exists in every modern
  substrate; rolling your own session-snapshot/clone system before
  checking is reinventing the wheel.

## Sources

The framing surfaced from a 2026-05-19 conversation about Claude
Code session setup. The user asked whether a prepared session —
heavy with skills, plugins, and methodology files — could be cloned,
motivating a survey of the actual amortization mechanisms in Claude
Code (prompt cache TTL and pricing; lazy skill loading; fork
primitives) and the substrate-spanning observation that *every layer
of the modern stack* amortizes setup through prefix caching rather
than state cloning.

Specific substrates referenced:

- Anthropic prompt cache: ephemeral KV-cache keyed by prefix; ~10×
  cheaper reads; 5-min default TTL, 1-hour beta. Anthropic
  engineering: *Lessons from building Claude Code: prompt caching is
  everything*.
- Docker / OCI layer cache: instruction-sequence hashing; documented
  in Dockerfile best-practices.
- Bazel action cache: input-hash → output mapping; local and remote-
  cache variants.
- Nix derivations and binary caches: input-addressed store paths.
- The HTTP / browser cache (`Cache-Control`, `Vary`, `If-None-
  Match`) — the longest-running prefix cache in production.
- The kernel page cache and filesystem block cache — the substrate
  beneath everything.
- LLVM ThinLTO module summary caches, V8 code caches, HotSpot tiered
  compilation caches — JIT-layer prefix caching at the compiler
  tier.
- macOS `dyld` closure cache and CRIU process checkpoint/restore —
  for the snapshot-mechanism contrast.

Intellectual ancestor: the *content-addressable storage* tradition
(Git's object store, Plan 9's Venti, Bazel's CAS). Keying state by
hash-of-content rather than by mutable identity is the structural
prerequisite for prefix caching to compose across layers. Once you
have content-addressing, prefix-caching falls out.

The AI-specific implication — that LLM applications should design
their prompts as *long stable prefix + short variable suffix* to
exploit prompt caching, and that this design discipline replaces
the urge for a "session clone" primitive — appears to be a
2024–2026 observation as long-context model deployments matured and
per-token cache pricing made the math obvious.
