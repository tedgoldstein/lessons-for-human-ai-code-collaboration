# HANDOFF.md

A handoff for a Legion session — a fresh AI agent (or human)
arriving on this branch cold. Read this first.

This file is transient session-state, not a permanent artifact
of the library. See *When to discard this document* at the
bottom; remove it before merging to `main` unless the project
adopts handoff files as a convention.

---

## Where you are

| Field | Value |
|---|---|
| **Branch** | `claude/review-collaboration-guidelines-OnWfg` |
| **Base** | `origin/main` at `545349e` (FluidVsTricky + Radar→Track update) |
| **Tip** | `515c875` |
| **Ahead of main** | 6 commits |
| **Working tree** | clean |
| **PR status** | none opened (the prior session was told not to open one) |
| **Written against** | tree at SHA `515c875` |
| **Written on** | 2026-06-03 |

Verify before acting: `git fetch && git log --oneline origin/main..HEAD`.
If the stack doesn't match section *What this session did*, this
document is stale — read it with skepticism.

---

## What to load before acting

The library has a two-layer routing model documented in
`README.md` § *How an agent decides which skill to load*.
Bootstrap by loading, in order:

1. **`UsingLessons/SKILL.md`** — the standing discipline that
   says *before any non-trivial engineering action, check
   whether a lesson applies; if one plausibly does, load it,
   announce which one and why, follow it.* Without this the
   library is inert.
2. **`Index/SKILL.md`** — the trigger catalog `UsingLessons`
   consults. Not the bootstrap itself.
3. **`README.md`** — orientation, install paths, how the
   routing actually works.

If you intend to act autonomously, also load:

- **`MultiAICollaborationViaGit/SKILL.md`** — branch-per-session,
  never trample uncommitted work, inline rules a cloud agent
  can't see.
- **`CausalDivergence/SKILL.md`** — tag claims with the version
  they were made against; the announce-on-load ritual is the
  same legibility pattern applied to skill routing.
- **`AskExactlyWhenAmbiguous/SKILL.md`** — proceed locally,
  pause externally.
- **`PushIsPublication/SKILL.md`** — push is publication, not
  save; relevant if you generate commits autonomously.

---

## What this session did

Six commits on top of `545349e`:

| SHA | Scope |
|---|---|
| `ba441c9` | Index `CommitHygiene` (was a complete SKILL.md but unreachable from README/Index); cross-link the git-cluster |
| `4287e44` | Tighten 2 long descriptions (`CausalDivergence`, `MultiAICollaborationViaGit`); add native `.claude/skills/` install snippet to `README.md`; 9 `## See also` blocks |
| `a14af15` | 7 remaining `## See also` blocks (`NamingIsAPI`, `OneChangeAtATime`, `PostmortemsWithoutBlame`, `ReadTheSourceFirst`, `SingleSourceOfTruth`, `TheBisectMindset`, `TimeBoxedExperiments`) |
| `cc5e465` | 14 descriptions sharpened to be trigger-rich, so the harness's progressive-disclosure routing can self-select |
| `fa4ccd3` | New `UsingLessons/SKILL.md` — the layer-2 bootstrap discipline; wired into README and Index |
| `515c875` | Index `FluidVsTricky` (new lesson that landed on `main` while this branch was in flight) + 3 reciprocal See-also links |

---

## Decisions already made — don't relitigate

Three calls from this session that a fresh agent might
otherwise re-open from scratch.

### 1. Threshold framing in `UsingLessons`
Adapted from `obra/superpowers`' `using-superpowers` skill,
which uses a literal *"1% chance → invoke"* rule. This library
re-frames it as *"load when a trigger plausibly matches"*,
because principles ask for judgment rather than procedural
recipe-following. The numeric form is a deliberate **rejected
alternative**, not an oversight. Don't revert.

### 2. `FluidVsTricky`'s body styling
The lesson uses `## Related skills` rather than the library's
`## See also` convention. The divergence is the original
author's; the prior session deliberately did not restyle it.
If the project later standardises, that's a separate decision.

### 3. The two-layer routing model is co-dependent
- Trigger-rich descriptions (commit `cc5e465`) make a check
  *land* on the right skill.
- `UsingLessons` (commit `fa4ccd3`) makes the check *happen* on
  every engineering task.

Either alone is insufficient. Don't refactor away one in
service of the other; both are load-bearing.

---

## Open ground — where you can usefully act

- **Open a PR for this branch.** The prior session was
  explicitly told not to (`IMPORTANT: Do NOT create a pull
  request unless the user explicitly asks for one`). If the
  user now wants the work merged, opening the PR is the next
  step.
- **Tighten `FluidVsTricky`'s frontmatter description.** It's
  already reasonable but could match the trigger-rich standard
  of the rest of the library. Not done this session out of
  restraint (the lesson was the user's own freshly-written
  contribution).
- **Install the library natively at `.claude/skills/`** for
  the next interactive session, so `UsingLessons` and the
  trigger-rich descriptions are surfaced by the harness and
  the routing loop closes empirically rather than just on
  paper.
- **Decide whether handoff files are a convention.** This is
  the first one. Either adopt the pattern (and add a skill for
  it — *handoff-by-disk* would pair with `BootstrapByHand` and
  `CausalDivergence`) or treat this file as a one-off and
  delete on merge.

---

## What NOT to do

- Don't force-push to `main`. Branch is feature-only.
- Don't squash this branch on merge — the commit stack tells a
  story (CommitHygiene gap → cross-link pass → description
  pass → bootstrap → FluidVsTricky integration) that bisect
  and reviewers can read. Per `OrganisingGitPullRequests`,
  reshape *before* review, not at merge.
- Don't restyle `FluidVsTricky`'s body unless the user asks
  (see *Decisions already made* §2).
- Don't add `## See also` blocks to skills that already
  have them — every non-Index skill in the library does as of
  `a14af15`. Adding new neighbours into existing blocks is
  fine; recreating the section is duplication.

---

## When to discard this document

This file is session-state, not library content. It goes stale
quickly. Delete it when:

- The branch merges and the session it documents is no longer
  the most recent context.
- The SHAs at the top no longer match what's on disk and the
  Legion has reconciled in some other way.
- The project decides handoff-by-disk is a convention with its
  own skill, and that skill specifies a different filename or
  format than this prototype.

Until then it lives at the repo root because that's where a
cold agent will naturally look. Per `CausalDivergence`: put
state on the path every actor already takes.
