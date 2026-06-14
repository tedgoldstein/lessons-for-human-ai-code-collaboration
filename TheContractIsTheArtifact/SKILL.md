---
name: the-contract-is-the-artifact
description: When a system has both a data contract (file format, schema, protocol) and the tooling that renders/mutates it, make the contract the durable thing and the tools mere renderers; cheap substrates (plain text + git, JSON, YAML, a flat directory of markdown) let humans, multiple AIs, scripts, and future tools all participate without permission. Load when designing a system with both persistent state and a UI/CLI, when tempted to lock canonical data inside one app's private database, when a migration is forced because the storage format is tied to one tool, or when an AI or script must join a workflow whose only surface is a clickable UI.
version: 0.1.0
---

# The Contract Is the Artifact

A common shape: someone builds a system, and the system has both
*data* and *tooling*. Bug-tracking software has bug records and a
UI. A CI system has pipeline definitions and a runner. A note-taking
app has notes and an editor. The natural reflex is to design the
tooling first, treat its internal data structures as private, and
expose the system *through* the tooling.

That design choice is more consequential than it looks. It binds
every consumer of the data — every script, every other tool, every
future AI agent, every human with a text editor — to the success and
availability of the original tool. When the tool dies or stops being
maintained, the data dies with it (or requires expensive migration).

The discipline is to invert the priority. **The contract is the
durable thing.** Make the file format / schema / data shape so
cheap to read and write — plain text in git, JSON on disk, a
directory of markdown — that any consumer can join without
permission. The tools become *renderers* and *mutators* of that
contract. When a tool is wrong or stale, replace it; the data is
untouched. When a new participant arrives (a new tool, a new AI
agent, a new team), they read the file format and join.

## When to invoke

- Designing a new system that has both persistent state and a UI.
- You're tempted to put the canonical data structure inside the
  app's memory or its database, with the UI as the only access
  surface.
- You're choosing between "JSON file on disk" and "row in our
  internal DB" for state that other tools or scripts might want
  to read.
- An AI agent or script needs to participate in a workflow, but
  the only documented surface is a clickable UI.
- You're about to write a custom binary format for something that
  could plausibly be text.
- The roadmap depends on one specific app being polished before
  users can engage with the system.
- A migration from your-tool to their-tool requires writing an
  export script — and you realize you don't have one.

## The shape

| Property | Aim for | Why |
|---|---|---|
| **Contract substrate** | plain text + git (or equally cheap) | Any text editor, any script, any AI agent can participate. No SDK required. |
| **Contract location** | files on disk, paths users can `ls` | Discoverable without a tool. `cat` works. |
| **Contract documentation** | a single readable spec (markdown, schema file) | Strangers can implement a new consumer correctly |
| **Tooling role** | render + mutate the contract | Tools are interchangeable; if one is broken, others still work |
| **Migration cost** | near-zero (it's already text) | The contract outlives the tools that read it |
| **Tool diversity** | many consumers possible, including ones you didn't write | The contract attracts implementations |

The contract isn't *necessarily* plain text — it could be a SQL
schema, a protobuf, a well-documented binary format. But the
cheaper the substrate, the more participants you get. Plain text
+ git is the cheapest commonly-available substrate that already
has versioning, diffing, and merging.

## Why it matters

1. **Tools die; contracts survive.** Every app you've ever used
   has been replaced. The data, if you're lucky, you can still
   read. If you're unlucky, you can't, and a piece of your work
   is locked behind a dead vendor. Designing the contract first
   ensures the data is portable from day one.

2. **Multiple consumers without permission.** A contract that's
   readable with `cat` invites scripts, dashboards, mobile
   companions, AI agents, scratch Python notebooks. None of them
   need approval from the original tool's maintainer. The system
   grows surface without growing complexity in any one tool.

3. **The bootstrap problem dissolves.** When a new tool can't
   yet do something — Track.app can't file Tracks about
   Track.app — you fall back to the contract directly. Edit
   the file in a text editor; the next-newest tool will pick
   up the change. There's never a *blocking* dependency on a
   single tool.

4. **AI agents become first-class participants.** Modern AI
   coding agents (Claude Code, Cursor, others) operate
   natively on files. A system whose contract is files is one
   they can meaningfully participate in. A system whose
   contract is private app state is one they can only watch
   through a screen-scraping window.

5. **Implementations can be ugly.** You're free to ship a
   half-finished app, a janky CLI, a placeholder web view —
   because none of them are load-bearing. The contract is
   load-bearing; the implementations are drafts. This
   permission to leave implementations rough is what lets you
   iterate fast on the actual hard part (the contract design).

## Practical guidance

- **Write the spec before the app.** Even if the spec is a
  short markdown file in the repo describing the field
  shapes and the validation rules. Get the contract right
  first; tools follow.
- **Keep the contract literally readable.** A maintainer
  with a text editor must be able to open a file and
  understand the state. If the answer is "you need the app
  to read this," the contract isn't yet the artifact.
- **Make the contract diff-able.** Lines, not blobs.
  Markdown, JSON-with-newlines, TOML. The diff is part of
  the value — git becomes the audit log.
- **Resist private fields.** If a tool wants to stash
  metadata, write it to the file with a clear marker.
  Tool-private state is fine for cache; tool-private state
  in the canonical contract is the start of vendor lock.
- **Document the contract once, prominently.** A SKILL.md,
  a SCHEMA.md, an OpenAPI doc, a README §The file format
  — one canonical document, linked from everywhere.
- **Validate at the trust boundary, not later.** When
  reading a file, normalize and validate immediately.
  Downstream code sees the canonical shape; bad inputs
  never propagate.
- **When the tool can't host the workflow, edit the file
  by hand.** This is normal, not an emergency. The
  contract isn't broken because the tool is incomplete.

## Common failure modes

- **The DSL trap.** "Let's write a custom format so our
  tool can be more efficient." The format is now
  proprietary; no other consumer joins. Use a mature
  format (JSON, YAML, markdown, SQLite, protobuf) unless
  there's a *strong* reason not to. See
  [DontReinventTheWheel](../DontReinventTheWheel/SKILL.md).
- **The "API is the contract" mistake.** A REST or GraphQL
  API treats the underlying data as private — only the
  current API shape is documented. When the API changes,
  consumers break; when the service is down, nobody can
  read anything. The API is a *renderer*; the file
  format / database schema underneath is the contract.
- **The hidden state.** Half the system's behavior depends
  on user preferences stored in app-private locations
  (`~/Library/Preferences/`, browser localStorage, an
  app database). Other tools can't honor those
  preferences. The contract is incomplete.
- **Migration paralysis.** When the existing storage isn't
  the contract, every redesign requires a migration script.
  Teams accumulate "migration debt" and stop redesigning.
  A plain-text contract sidesteps this entirely.
- **The "one true tool" assumption.** "Everyone will use
  our app." They won't. Someone will need to script
  against your data; someone will want to read it on
  their phone; an AI agent will be asked to triage. The
  contract that's open to all of them is the one that
  endures.

## When tooling-private state is genuinely the right call

- **Performance-critical hot paths.** A render cache, a
  search index, a pre-computed view — these can be
  tool-private as long as they're derivable from the
  canonical contract. The contract is the source of truth;
  the cache is reconstructable.
- **Truly secret state.** A private key, a session token,
  user-specific UI preferences that don't belong in
  shared storage. Keep these in the tool; document the
  separation.
- **Highly-structured datasets where text would lose
  fidelity.** Genomics, image data, video, scientific
  measurements. Use a mature binary format (HDF5,
  Parquet, OME-TIFF) but treat *that* as the contract —
  not your app's in-memory representation.

The pattern isn't "everything must be plain text." It's
"the canonical data must be readable by something other
than your one tool, and ideally by `cat`."

## Worked example

The Track methodology was designed contract-first. Every Track
is a markdown file at `Track/<id>.md` with a known section
shape. The CLI, the macOS app, the web dashboard, future
agents — all consume the same files. When one consumer
(Track.app) was incomplete, Tracks were filed and driven from
the CLI + a text editor. When a second adopting project
(Medbook) had been hand-maintaining a fork of the methodology
that drifted, refreshing was a `Track init --force` away —
the new vendored skill files dropped into the same directory
structure, and every existing on-disk Track continued to parse
under the canonical CLI. Multiple concurrent AI sessions all
worked from the same files; their handoff was git, not a
shared service. The contract — markdown + git + an 8-char id —
was so cheap that *the contract was the system*. Tools came
and went (a SwiftUI wave2 was entirely replaced by a WebKit
wave3 — "HTMLJUNTA" — without any data migration, because the
data wasn't in the app).

## Tagline

> The contract is durable. Tools are drafts.

When you can't yet build the polished tool, you can still
participate via the contract. When the polished tool finally
ships, the contract is what it renders.

## See also

- [SingleSourceOfTruth](../SingleSourceOfTruth/SKILL.md) — every
  piece of state has one owner. Pairs with this skill: the file
  *is* the owner.
- [DontReinventTheWheel](../DontReinventTheWheel/SKILL.md) — use
  mature substrates (JSON, markdown, git). Don't invent a
  private format.
- [BoringTechWherePossible](../BoringTechWherePossible/SKILL.md)
  — plain text + git is boring; that's the strength.

## Sources

Inferred from observing the Track methodology (Track repo,
2026-05) where the file-format-as-contract approach let the
methodology spread across three projects (Track/, medbook.org,
labbook.ai) without any single consumer being load-bearing.
The pattern is older — it's the same instinct behind plain-text
configuration files, RFC-defined wire protocols, and "everything
is a file" in Unix. This skill is the restatement specific to
modern AI-augmented development, where the cheap contract is
also what makes AI agents first-class participants.
