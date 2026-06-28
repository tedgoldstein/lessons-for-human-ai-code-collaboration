# Impact Analysis — Replacing the Level-0 Immutable Database with AIFS

**Status:** Work-in-process draft (first pass)
**Date:** 2026-06-28
**Author:** Ted Goldstein + Claude (claude/aifs-impact-analysis-h7m9l9)

---

## 0. Reading note — scope and provenance of this draft

This document was started in a remote Claude Code container that had
**only** the `lessons-for-human-ai-code-collaboration` repo checked out.
The AIFS source tree (`~/Code/AIFS`) was **not reachable** from that
environment, so this first pass is built from:

- the architectural facts stated in the task ("replace the level-0
  immutable database with AIFS; give up immutability; substitute git
  versioning; mind the security implications"), and
- the design principles already written down in this repo
  (`IsolationAndTransactionalDiscipline`, `TheContractIsTheArtifact`,
  `SingleSourceOfTruth`, `CausalDivergence`, `IdempotentByDefault`,
  `MakeTheWrongThingHard`, `PushIsPublication`).

Every place where a concrete AIFS or level-0 detail is **assumed** rather
than **confirmed** is tagged **[ASSUMPTION]**. The "Open questions" section
(§8) lists what must be pulled from the actual source to turn this draft
into a decision-grade analysis. Treat §8 as the to-do list for the next
pass, ideally run against the real tree.

---

## 1. What is actually being proposed

> Replace the current **level-0 database** — an *immutable* store at the
> bottom of the stack — with **AIFS**, accepting the loss of immutability
> and substituting **git versioning** as the mechanism that recovers the
> properties immutability was buying.

Restated as a swap of *guarantees*, not just of *components*:

| Layer role | Today (level-0 DB) | Proposed (AIFS) |
|---|---|---|
| Substrate | Database engine **[ASSUMPTION: content-addressed / append-only immutable store]** | A filesystem-shaped store (AI File System) |
| Mutability | Immutable — writes append, never overwrite | Mutable files; history is *recorded*, not *enforced* |
| History / versioning | Intrinsic to the store (every version physically retained) | Git commit DAG over the file tree |
| Identity of a value | Content hash / immutable key **[ASSUMPTION]** | Path + git blob SHA at a commit |
| Audit trail | The store *is* the trail | `git log` / reflog / signed commits |

The key reframing for the whole analysis: **immutability and git
versioning are not the same guarantee.** Immutability is a *structural*
property — the substrate makes overwrite impossible. Git versioning is a
*recorded-history* property — overwrite is possible, but the prior state
is retained and recoverable *as long as the history is not rewritten and
the object is still reachable.* The gap between "impossible to lose" and
"recoverable unless someone rewrites history" is the center of gravity of
this decision.

---

## 2. What immutability at level 0 is currently buying

Before costing the swap, name precisely what the immutable level-0 store
provides today. (This list is the yardstick the AIFS+git design must be
measured against — §4.)

1. **Tamper-evidence / tamper-resistance.** Content-addressed immutable
   storage means a value's identity *is* its content. You cannot silently
   change a value and keep its name. **[ASSUMPTION: level-0 is
   content-addressed.]**
2. **Stable references.** Anything holding a key/hash is guaranteed that
   the referent never changes underneath it. No "the row I read is no
   longer the row that's there."
3. **Trivial caching & idempotency.** Immutable values can be cached
   forever and dedup'd by identity — this is `IdempotentByDefault` for
   free: writing the same content twice is a no-op.
4. **Time-travel / point-in-time reads** without a separate versioning
   subsystem. The old version is *physically still there*.
5. **Concurrency safety at the substrate.** Two writers can't corrupt a
   value that can't be overwritten. This is the cheap end of
   `IsolationAndTransactionalDiscipline` — the substrate itself supplies
   isolation for reads.
6. **Audit by construction.** "Who changed this and to what" reduces to
   "what new immutable versions exist," because nothing is ever erased.

Each of these is a property AIFS must either reproduce, replace with git,
or consciously give up. §4 walks them one by one.

---

## 3. What AIFS plausibly buys (the upside of the swap)

These are the reasons one would *want* to make this change. **[ASSUMPTION:
these match AIFS's actual goals — confirm in §8.]**

1. **Files as the contract** (`TheContractIsTheArtifact`). A filesystem is
   `cat`-able, `ls`-able, diff-able, and editable by *any* tool, human, or
   AI agent without going through one database's API. The contract becomes
   the durable thing; tools become renderers. This is the single
   strongest argument for the swap, and it's squarely one of this repo's
   own principles.
2. **AI agents become first-class participants.** Modern coding agents
   operate natively on files and on git. A level-0 *database* forces every
   agent to speak a bespoke API; a *filesystem + git* lets them participate
   with the tools they already have.
3. **Git versioning is a mature, boring substrate** (`BoringTechWherePossible`).
   Branching, diffing, merging, blame, bisect, signed commits, and a
   decade of tooling come for free instead of being reinvented inside a
   database.
4. **Causal divergence becomes legible** (`CausalDivergence`). The git
   commit DAG is *exactly* the reconciliation point that skill argues for:
   parent SHAs, branch refs, merge commits — a natural place where
   multiple actors' timelines are visible and reconcilable. An immutable
   DB gives you many versions but not necessarily a *branch/merge* model
   for divergent-but-valid timelines.
5. **Branching as a feature, not an accident.** Git makes "many
   simultaneously-valid views" (`CommonsWithDivergentClones`,
   `CausalDivergence`) a supported, first-class operation. Immutable stores
   usually give you linear version history, not cheap divergent branches
   with a defined merge direction.
6. **Lower operational surface.** A filesystem + git may remove a whole
   database engine from the deployment (backups, connection pools, schema
   migrations). **[ASSUMPTION: level-0 is a separately-operated engine.]**

---

## 4. Immutability → git versioning: property-by-property

This is the core of the analysis. For each property from §2, is git an
**equivalent**, a **weaker substitute**, or a **stronger** replacement?

### 4.1 Tamper-evidence — ⚠️ WEAKER unless hardened
- **Immutable store:** structural. Content *is* identity.
- **Git:** git is content-addressed *internally* (every blob/tree/commit is
  a SHA-1/SHA-256 hash), so a given commit is tamper-evident **in place**.
  BUT history can be **rewritten** (`git commit --amend`, `rebase`,
  `push --force`, `filter-repo`). A force-push can erase the record that a
  value ever existed. Immutability forbids this structurally; git only
  forbids it by *policy*.
- **To recover the guarantee:** signed commits (GPG/SSH/sigstore), a
  protected/append-only remote that rejects non-fast-forward pushes, and
  ideally a transparency-log-style witness so the tip can't be silently
  rolled back. SHA-1 is also weakening — prefer git's SHA-256 object
  format for a security-load-bearing store. **This is the property most
  degraded by the swap and needs the most explicit design.**

### 4.2 Stable references — ⚠️ WEAKER by default, EQUIVALENT if you pin
- **Immutable store:** any key is permanently stable.
- **Git:** a *commit SHA* (or blob SHA) is permanently stable and is the
  honest analog of an immutable key. A *path* or a *branch ref* is **not**
  stable — `main` moves, files are overwritten. The migration hazard: code
  that today holds an immutable key and assumes permanence must, in the
  AIFS world, hold a `(commit-SHA, path)` pair, not a bare path. Any
  reference that degrades to "latest" silently loses the guarantee.

### 4.3 Caching & idempotency — ✅ EQUIVALENT (arguably better)
- Git blobs are deduplicated by content hash — identical content stored
  twice is one object. This preserves `IdempotentByDefault` at the object
  layer. Caching by blob SHA is as safe as caching by immutable key.
- Caveat: idempotency of *writes* (commits) is **not** automatic — the
  same logical change committed twice yields two different commit SHAs
  (different timestamps/parents). Write paths that relied on the level-0
  store's natural dedup must add an explicit idempotency key at the commit
  layer.

### 4.4 Time-travel / point-in-time reads — ✅ EQUIVALENT (often better)
- Git is *built* for this: `git show <sha>:<path>`, `git log`, `git
  bisect`. This is at least as good as an immutable store's version list,
  and the DAG additionally encodes *causality* (parentage), which a flat
  version history may not. Net win — provided history isn't rewritten
  (see 4.1).

### 4.5 Concurrency safety — ⚠️ DIFFERENT shape, needs discipline
- **Immutable store:** writers literally cannot collide on a value.
- **Git:** concurrency is handled by branch isolation + merge, not by
  structural impossibility. This is precisely the regime
  `IsolationAndTransactionalDiscipline` and `CausalDivergence` were written
  for. The good news: git's branch/worktree model is a *strong* isolation
  primitive. The cost: you now need the discipline those skills prescribe —
  branch-per-actor, defined merge direction, a reconciliation point — and
  you inherit **merge conflicts**, which an immutable store never has.
  Concurrent writers to the *same path* now require a conflict-resolution
  policy (last-writer-wins? CRDT-on-top? lock/lease?) that the immutable
  store made unnecessary. **[OPEN: what is AIFS's concurrent-write story?
  §8]**
- A specific hazard worth calling out: a filesystem invites **in-place
  mutation between commits**. There is a window where a file is changed but
  not yet committed — state that is neither the old immutable value nor a
  recorded new version. The immutable store had no such window. Recovery
  (`IsolationAndTransactionalDiscipline`: "atomicity / durability") must
  define what an uncommitted working-tree change *means*.

### 4.6 Audit by construction — ⚠️ EQUIVALENT only with enforcement
- `git log` + signed commits + protected history ≈ audit trail. But again,
  the immutable store's audit was *structural*; git's is *policy-enforced*.
  An attacker (or a buggy agent) with push access and force-push rights can
  edit the audit trail. The audit guarantee is only as strong as the
  weakest write-control on the remote. See §5.

### Summary table

| Property (from §2) | Git verdict | What must be added |
|---|---|---|
| Tamper-evidence | ⚠️ weaker | signed commits, append-only remote, SHA-256, no force-push |
| Stable references | ⚠️ weaker | reference by `(commit-SHA, path)`, never bare path |
| Caching / idempotency | ✅ equivalent | explicit idempotency key on the *write* path |
| Time-travel | ✅ equivalent+ | nothing (it's a git strength) |
| Concurrency safety | ⚠️ different | branch isolation + conflict policy + reconciliation point |
| Audit by construction | ⚠️ equivalent* | enforced protected history + signing |

The headline: **git recovers the *read-side* guarantees of immutability
well, and the *write-side / tamper guarantees only under enforced
policy*.** Immutability moved those guarantees into the substrate where
they couldn't be turned off; git moves them into *configuration and
process*, where they can. That shift is the true cost of the swap.

---

## 5. Security implications

This is called out separately because the swap changes the **threat model**,
not just the feature set.

### 5.1 New / amplified risks
1. **History rewrite as data destruction.** Force-push / rebase /
   `filter-repo` can erase or alter records that, under immutability, were
   undeletable. *This is the single biggest security regression.*
   Mitigation: protected refs, deny non-fast-forward, deny force-push,
   off-box append-only mirror, signed tags as checkpoints.
2. **Write-access blast radius.** In an immutable store, a compromised
   writer can only *add* new versions (the old truth survives). In a
   mutable git store, a compromised writer with force-push can *rewrite the
   past*. Least-privilege on push rights becomes security-critical, not
   merely operational.
3. **Integrity of the hash.** Git's default SHA-1 has known collision
   attacks. For a *security-load-bearing* content store, move to git's
   SHA-256 object format, or the content-addressing guarantee is softer
   than the immutable store's. **[OPEN: what hash does level-0 use today? §8]**
4. **Confidentiality / secrets-in-history.** A filesystem + git makes it
   *easy* to commit a secret, and git's "never forgets" then works against
   you: the secret persists in history even after deletion. The immutable
   DB may have had a narrower, API-mediated surface. Needs: pre-commit
   secret scanning, `.gitignore` discipline, and a documented
   history-purge procedure (which itself conflicts with append-only — a
   genuine tension to resolve).
5. **Supply-chain / unreviewed agent writes.** `TheContractIsTheArtifact`'s
   strength (any agent can write files) is also an attack surface: more
   actors can mutate the substrate directly. Couple with
   `IsolationAndTransactionalDiscipline` — scoped write authority,
   capability boundaries, review-before-merge on protected branches.
6. **Working-tree exposure.** Files on disk are readable by anything with
   filesystem access (other processes, other agents, backup tooling). A
   database mediated access through a connection + authz layer. Filesystem
   ACLs / encryption-at-rest now carry weight the DB used to carry.

### 5.2 Security *improvements* from the swap
1. **Signed, attributable commits** give cryptographic authorship that a
   bare DB row may lack — *who* wrote *what*, verifiable.
2. **Reviewable diffs.** Every change is a human/AI-reviewable diff
   (`PushIsPublication` — push is the deliberate moment of making state
   visible). Malicious or buggy changes are inspectable before they land
   on a protected branch.
3. **Reproducible point-in-time state** aids forensics: you can check out
   the exact tree as of any incident.
4. **Boring, audited tooling.** Git's security properties are
   well-understood and widely tooled, vs. a bespoke store's.

### 5.3 The core security trade in one sentence
> Immutability put integrity in the **substrate** (un-turn-off-able); git
> puts integrity in **policy + cryptography + access control** (strong, but
> only if configured and enforced). The swap is safe **iff** the
> enforcement is treated as load-bearing infrastructure
> (`FeatureFlagsAreInfrastructure`-style), not as convention.

---

## 6. Migration considerations (preliminary)

- **`MakeTheWrongThingHard`:** the migration must make force-push /
  history-rewrite *structurally hard* on the canonical store (server-side
  hooks, protected refs), so the lost-immutability footgun is disarmed by
  default rather than by reminder.
- **`SingleSourceOfTruth`:** during migration there will be two stores.
  Name the authoritative one explicitly and make the other a clearly-marked
  derived copy; do not run a bidirectional sync (the skill's classic
  drift-bomb). Prefer a one-way cutover with a frozen, read-only level-0
  store kept as the historical record.
- **`TarzanMigrationStrategy` / `ReplaceDontRefactor`:** (both are lessons
  in this repo) — favor a clean swing to the new store behind a flag over
  an in-place gradual mutation of the old one.
- **Idempotency keys:** anywhere the old store's natural dedup was relied
  on, add explicit idempotency keys before cutover (§4.3).
- **Reference rewriting:** audit every consumer that holds a level-0 key;
  it must move to `(commit-SHA, path)` not a bare path (§4.2).
- **Keep the immutable store as an archival witness.** Cheapest mitigation
  for the tamper-evidence regression: don't delete level-0 — freeze it
  read-only as an independent, immutable audit anchor that the git store
  can be checked against.

---

## 7. Preliminary recommendation

**Directionally supportive, conditional on enforcement.** The swap aligns
with this repo's own deepest principles — `TheContractIsTheArtifact`,
`CausalDivergence`, `BoringTechWherePossible` — and the upside (files +
git as a contract that humans and AI agents share without permission) is
real and large.

But immutability was doing **security and integrity** work, and git only
reproduces that work **when force-push is forbidden, history is protected,
commits are signed, and the object format is SHA-256.** The recommendation
is therefore:

> **Proceed**, but treat the lost-immutability guarantees as a *named
> infrastructure requirement*, not an afterthought:
> 1. canonical remote is append-only / non-fast-forward-only,
> 2. commits signed and verified,
> 3. SHA-256 object format,
> 4. the old immutable store frozen read-only as an audit anchor,
> 5. explicit concurrent-write conflict policy,
> 6. secret-scanning on the write path.
>
> If those six cannot be guaranteed, the swap is a **net security
> regression** and should not ship.

---

## 8. Open questions — what the next pass needs from the AIFS source

These could not be answered without `~/Code/AIFS`. Answering them turns
this draft into a decision-grade document.

1. **What is the level-0 database, concretely?** Engine, on-disk format,
   is it content-addressed, what hash, what is its API surface?
2. **What is AIFS, concretely?** Is it a FUSE filesystem, a libgit2
   wrapper, a virtual FS over a git bare repo, something else? What is its
   read/write API?
3. **Concurrent-write model.** What happens when two agents write the same
   path? Locks? Leases? Merge? Last-writer-wins? (Drives §4.5.)
4. **Commit granularity.** Who commits, and when? Per-write? Batched?
   Agent-driven? (Drives the "uncommitted working-tree" window in §4.5.)
5. **Remote topology.** Is there a canonical remote? Who can push? Can it
   be made append-only? (Drives all of §5.)
6. **Object format & signing.** SHA-1 or SHA-256? Are commits signed today?
7. **What currently depends on immutability?** Which consumers hold
   level-0 keys assuming permanence? (Drives §4.2 + §6.)
8. **Performance envelope.** Object count, write rate, repo size — git
   degrades at very large blob counts / huge binaries; does AIFS need
   partial clone / sparse checkout / a chunking layer?
9. **Encryption-at-rest / filesystem ACL story** (Drives §5.1.6).
10. **Existing AIFS code review findings.** The original task referenced an
    in-progress code review of `~/Code/AIFS`; its findings feed §3 and §5.

---

## 9. Cross-references (this repo's lessons that bear on the decision)

- `TheContractIsTheArtifact` — the strongest *pro* argument (files as
  contract).
- `IsolationAndTransactionalDiscipline` — what you must rebuild once the
  substrate stops enforcing isolation for you.
- `CausalDivergence` — the git DAG as the reconciliation point; the
  branch/merge upside.
- `SingleSourceOfTruth` — migration discipline; name the authority, no
  bidirectional sync.
- `IdempotentByDefault` — what to add on the write path (§4.3).
- `MakeTheWrongThingHard` — disarm the force-push footgun structurally (§6).
- `PushIsPublication` — push as the deliberate visibility/security boundary.
- `BoringTechWherePossible` — git as the mature substitute substrate.
- `CommonsWithDivergentClones` — divergent timelines as a supported feature.
- `ReplaceDontRefactor` / `TarzanMigrationStrategy` — swing-don't-mutate
  migration shape.
