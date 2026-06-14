---
name: bootstrap-by-hand
description: When a tool is meant to manage some kind of artifact (issues, schedules, configs, documents), and the tool isn't yet ready to manage *itself* — the file that fixes the broken file-editor, the issue that tracks the broken issue-tracker, the schedule that schedules the scheduler — fall back to the contract directly. Edit the file by hand. The tool can land later; the work doesn't have to wait. This is the chicken-and-egg lens — design systems so the egg can be laid before the chicken hatches.
version: 0.1.0
---

# Bootstrap By Hand

A subtle class of design problem: a tool is meant to manage some
artifact, and at some moment the tool itself *is* the thing that
needs managing. The bug tracker can't track its own bug. The
scheduling system can't schedule its own deploy. The config
manager can't configure itself. The editor can't edit its own
source.

The naive answer is "we need to fix the tool first, then we can
use it on itself." That's the deadlock. The principle that
breaks it is **bootstrap by hand**: when the tool isn't ready,
fall back to whatever cheap substrate the contract uses (plain
text, a config file, a directory of markdown), edit the
artifact directly, and keep moving. The tool catches up later;
in the meantime, the work doesn't stop.

This works **only if the contract is cheap enough to manipulate
without the tool** — which is why this skill pairs with
[TheContractIsTheArtifact](../TheContractIsTheArtifact/SKILL.md).
A system whose canonical state lives in an opaque database
behind a broken UI is genuinely deadlocked. A system whose
canonical state is markdown in git is bootstrap-by-hand-able.

## When to invoke

- The new tool isn't ready, and you need to use the system *now*.
- A bug in the tool prevents using the tool on itself.
- An AI agent or script needs to participate in a workflow, but
  the only documented surface is the (broken / unfinished) tool.
- You're tempted to wait for the tool to be polished before
  filing the issue that's blocking the polish.
- "How will we test this when there's no UI yet?" — by hand
  against the contract.
- The first user of a new methodology / format / system has no
  prior tools to use.
- A meta-Track / meta-bug needs to be filed about the tool that
  files Tracks / bugs.

## The shape

| Property | Aim for | Why |
|---|---|---|
| **Contract substrate** | cheap enough to edit with `vim` / `cat` / `cp` | Without this, there's no bootstrap path at all |
| **Documented file format** | section names, field shapes, validation rules | Hand-editing only works if the contract is legible |
| **Validator separate from editor** | a parser/CLI that says "this file is valid" without requiring the UI | You can write by hand and check correctness |
| **Tool-less workflow** | every action the tool does has a manual fallback | Filing, status updates, verdicts, close-outs — all doable from a text editor |
| **Honest documentation** | the README acknowledges the hand-edit path is real | Not a hidden shame; a celebrated property |

## Why it matters

1. **No tool is ever finished.** Tools are always partway through
   a wave. Bootstrap-by-hand means progress doesn't stall on
   tool-readiness. The tool can be perpetually a draft as long
   as the contract works.

2. **Self-reference is everywhere.** Any system that tracks its
   own work — a bug tracker tracking its own bugs, a CI
   pipeline shipping CI improvements, a documentation system
   maintaining its own docs — has bootstrap moments. Designing
   for them is the difference between "we'll get there
   eventually" and "we can already work."

3. **AI agents are first-class hand-editors.** A modern AI
   coding agent doesn't need your UI; it reads the file and
   writes the file. If your system's contract is hand-editable,
   AI agents can participate without you building an "AI API."
   They just edit the file like any other contributor.

4. **The hand-edit path proves the contract is honest.** If
   the only way to file an issue is through the issue-tracker
   UI, the data model is whatever the UI implementer happened
   to write — undocumented, ad-hoc. If the file format is
   hand-editable, it has to be documented, validated, and
   stable. Hand-editability is a forcing function for clean
   contract design.

5. **Bootstrap-by-hand is the escape hatch.** Tools break.
   Releases regress. Cloud APIs go down. Browser plugins
   stop working. When the tool is gone and the work needs to
   happen, the contract being editable is the difference
   between "blocked" and "less convenient but possible."

## Practical guidance

- **Design the contract to be readable + writable without the
  tool.** Markdown over binary blobs; lines over compressed
  state; literal field names over indexes; UTF-8 over esoteric
  encodings. The principle is "if `cat` works, `vim` works."
- **Write a CLI validator early.** Before the GUI, before the
  fancy app, a tool that says "this file conforms to the
  contract." That validator is the manual editor's safety net.
- **Document the hand-edit path explicitly.** The README's
  "How to use" section should include "you can also just open
  the file in your editor." Not buried in a footnote — listed
  as a first-class workflow.
- **Make the contract describable in one paragraph.** A
  newcomer should be able to skim a single paragraph and know
  enough to file a record by hand. If it takes a tutorial,
  the contract is too complex for hand-editability.
- **AI agents follow the same path.** When telling Claude
  Code or any other AI agent how to interact with the
  system, point at the file format, not the UI.

## Common failure modes

- **"We'll add the UI later, but for now use this dashboard."**
  Translation: until the UI is ready, everyone is locked out
  except the people who can find the dashboard. Wrong default.
  Until the UI is ready, *everyone* uses the contract directly.
- **The "real users won't edit files."** Maybe not. But
  developers will. AI agents will. Scripts will. Bootstrap-by-
  hand isn't claiming users want to hand-edit; it's claiming
  that *the system* should support it so the bootstrap moments
  work.
- **Letting the tool become the only interface.** Over time,
  features creep into the tool that depend on tool-private
  state. The hand-edit path silently rots. Audit periodically:
  *can I still file / update / close from a text editor?*
- **Hidden state.** The contract is markdown, but the tool
  also stores half its state in a sidecar database. The
  hand-edit path now leaves the system inconsistent. Either
  the sidecar is reconstructable from the contract, or the
  sidecar belongs in the contract.
- **Treating hand-edit as shameful.** "We have to support
  hand-editing because the UI isn't done yet." No — you have
  to support hand-editing because hand-editing is a first-
  class operation. The UI is one of several consumers of the
  contract.

## When the tool genuinely is the contract

Sometimes the tool's interactions *are* the artifact: an IDE
with live collaborative editing, a graphical CAD program, a
DAW recording audio. There's no useful "file format" beneath
the UI; the UI is doing real-time stateful work that text
can't represent.

For those systems, bootstrap-by-hand doesn't apply. But
they're a smaller set than people assume. Most "we need a
GUI" systems actually have a text-representable contract
underneath; the GUI is a renderer of that contract.

## Worked example

Track.app (the macOS bug-tracking app for the Track
methodology) wasn't yet packaged for distribution. A Track
needed to be filed *to track* the work of making Track.app
packageable. The chicken couldn't lay the egg.

The bootstrap-by-hand path: the Track methodology's contract
is `Track/<id>.md` — plain markdown in git. The CLI's
`Track new` verb mints an id and writes a skeleton file.
The user filled in the substantive sections (Problem,
Proposed Resolution, Acceptance criteria, Files likely
affected, Verify required) by editing the markdown
directly. The CLI's `Track status` verb confirmed the file
parsed cleanly. The Track.html dashboard rendered the new
Track via the JSON index built by `build-Track-index.js`.

The polished Track.app GUI played no role. It couldn't —
it's the thing being fixed. The Track moved through
Analyze (status `open` → `analyze` → `integrate`), with
every transition done by edit-and-commit on the markdown
file. Two more Tracks (about the same set of methodology
improvements) were filed the same way, in the same
session, while the laptop session and the user were both
actively working without any "wait until the app is
ready" pause.

The whole episode — file the Track, fill it out, transition
its status, cross-reference siblings — was bootstrap-by-
hand from start to finish. The cost: about ten minutes of
manual file edits. The benefit: no deadlock, no pause, no
"I'll come back to this when the app is ready."

## Tagline

> When the chicken isn't ready, lay the egg by hand. The
> contract has to be cheap enough that you can.

A tool is supposed to be a convenience. A system that
*requires* the tool to operate is a system with a hidden
single point of failure.

## See also

- [TheContractIsTheArtifact](../TheContractIsTheArtifact/SKILL.md)
  — the precondition. Bootstrap-by-hand only works when the
  contract is cheap to read and write.
- [DontReinventTheWheel](../DontReinventTheWheel/SKILL.md) —
  pick contract substrates (markdown, JSON, YAML, SQL) that
  every editor and every AI agent already understands. Don't
  invent a hand-editable format that nobody knows how to
  hand-edit.
- [ReplaceDontRefactor](../ReplaceDontRefactor/SKILL.md) —
  during a wave-replacement, the new wave's tools aren't
  ready. Bootstrap-by-hand is what lets you ship the
  contract and the tool gradually instead of all at once.

## Sources

Inferred from observing the Track methodology bootstrap
(2026-05): filing Track vr9xsmxk about Track.app's
distribution gap, from the CLI + text editor, because
Track.app couldn't yet host the workflow that would fix
it. The general principle is older — it's the same
instinct behind plain-text Unix config files, the
Internet's text-protocol culture (HTTP, SMTP all readable
with `telnet`), and the "everything is a file" credo.
