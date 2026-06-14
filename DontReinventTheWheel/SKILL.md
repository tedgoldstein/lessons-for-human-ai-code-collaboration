---
name: dont-reinvent-the-wheel
description: Recognize when a custom system you're building is isomorphic to a mature platform, then map onto the platform instead of rebuilding it. The residual — what the platform genuinely can't reach — is your real work; everything else is leverage.
version: 0.1.0
---

# Don't Reinvent the Wheel

Before writing a parser, a renderer, a layout engine, a routing system, a
schema language, a query system, or an event bus — pause and ask: **what
mature platform already solves a problem isomorphic to mine?**

If the answer is "HTML + DOM" or "SQLite" or "POSIX" or "ICU" or "the
language's package manager," strongly consider mapping your domain onto
that platform instead of building a parallel stack. The mature platform
has thirty years of edge cases, accessibility, internationalization,
security review, debuggability, and tooling. Your custom thing will be
worse along every one of those axes — and you will pay forever.

This skill is the **recognition + redirect** discipline.

## When to invoke

Trigger phrases / situations:

- "I need to parse my YAML / JSON / markdown / DSL and render it as a UI."
- "I need a query language over my structured documents."
- "I need bidirectional sync / replication / change tracking on records."
- "I need to lay out graphical elements with styling, focus, selection,
  copy-paste, accessibility…"
- "I'm building a routing system for *X* navigation."
- "I'm authoring a custom dependency-resolution algorithm."
- "I need to format messages between services with field validation,
  versioning, and tooling."
- You catch yourself writing the word *"my-own"* in a noun phrase that
  ends in "engine," "language," "format," "store," or "protocol."

Most of these have a mature substrate that solves the bulk of the
problem. The substrate might be:

| Problem you're tempted to build | Substrate that already solves it |
|---|---|
| Document → UI rendering with rich interaction | **HTML + CSS + DOM** (and a browser or WKWebView) |
| Records, queries, indexes, ACID | **SQLite** (or Postgres) |
| Hierarchical config, schema validation | **JSON Schema** (or Protobuf) |
| Wire format with versioning + codegen | **Protobuf / Cap'n Proto** |
| Cross-locale text, dates, sort orders | **ICU** |
| Process spawn, IPC, signals, pipes | **POSIX** |
| Package + dependency resolution | The language's package manager |
| Async pub/sub with replay | **A real message broker** (NATS, Kafka, Redis Streams) |
| Mature OAuth / SSO flow | The platform's identity stack |
| Fast incremental file watching | **fsevents / inotify** via a small wrapper |
| Text-mode terminal emulation | **xterm.js** (in a webview) or **SwiftTerm** (native) |

If your domain admits an isomorphism to one of these, do the mapping
work and let the substrate carry the weight.

## The procedure

1. **Notice the symptom.** You're authoring a stack of: a parser, a
   model, a layout/render engine, a styling system, a way to handle
   user events, a focus model, an undo log. You're going to maintain
   all of that. Forever.

2. **Inventory the substrates.** Before drawing a box on your own
   architecture diagram, list every mature platform whose primitives
   look approximately right. Don't filter — be promiscuous.

3. **Probe for isomorphism.** For each candidate, ask:
   - Can my domain types map to its types? (HTML elements ≅ my rendered
     nodes; SQL tables ≅ my collections; OAuth scopes ≅ my permissions.)
   - Can my domain operations map to its operations? (My queries ≅
     SQL; my user input ≅ DOM events; my schema ≅ a JSON Schema.)
   - Where the mapping is lossy, what specifically am I losing?

4. **Inversion test.** If you ship your custom thing, will it be *worse*
   than the substrate along axes you can't undo: accessibility,
   internationalization, security review, conformance, performance,
   tooling, debuggability, hiring? If yes along multiple axes — map,
   don't build.

5. **Isolate the residual.** What does your domain genuinely need that
   no substrate provides? *That* is your real codebase. Everything
   else is leverage. The residual is usually small, and it's almost
   always the interesting part — the part where your domain expertise
   actually lives.

## Worked example: Track's wave3 pivot

The Track project (`~/Code/Track`) hit this pattern in its second wave.

**The custom thing being built (wave2)** was a native SwiftUI surface for
an operator dashboard: BrandRow, TabStripView, DashboardView with four
view modes (Table / Pipeline / Lanes / Stream), TrackDetailView with a
stage-timeline spine, NewTrackView with live preview, LadybugFAB,
DesignTokens, custom Toaster, custom MarkdownText, custom verb buttons,
custom search, custom filter chips. Several thousand lines of SwiftUI.
Four full design-audit passes against a Claude Design handoff to chase
pixel-faithful parity.

**The recognition moment:** the problem was *parse a structured Track
markdown document; render a rich graphical UI with selection, focus,
keyboard shortcuts, copy-paste, accessibility, find-in-page, navigation
history, and styling*. **HTML + CSS + DOM has been solving this since
1993.** The team already had `consumers/Track.html/` — a JSX dashboard
that did the rendering side. Building a parallel SwiftUI stack was
literally re-solving a solved problem.

**The pivot.** The Swift host shrank to a WKWebView shell + a JS↔Swift
bridge. Track.html became the entire visual surface. The wave2 SwiftUI
tree got deleted (recoverable from a `HTMLJUNTA` tag).

**The residual** — the things HTML+browser genuinely couldn't do, and
where Swift kept earning its keep:

- **Subprocess** — PTY-backed Claude Code sessions per Track (browsers
  can't fork or attach to TTYs).
- **Filesystem write** — atomic mutation of `Track/<id>.md` files
  outside the user-data sandbox.
- **AI** — long-lived Anthropic threads, API keys, the DSD pipeline.
- **OS chrome** — native menus, global shortcuts (⌘F find, ⌥⌘C
  console), window/dock integration.

That residual is the whole point of having a native app at all.
Everything else — design tokens, accessibility, find-in-page, focus
order, ⌘+/-/0 zoom, view-source — comes free with the substrate.

The result: ~3500 lines of dead SwiftUI deleted; the native code became
small, focused, and inarguable; the design surface is maintained in one
place by one tool family (HTML+CSS+JSX).

## Common failure modes

- **"But we need pixel parity with a hand-drawn design."** The mature
  platform *gets you closer to parity than you think*. The design's
  shadows, OKLCH colors, font metrics, hover states, focus rings — all
  faithfully renderable. The places you genuinely can't reproduce in
  HTML are vanishingly rare in 2025.

- **"Performance."** This is almost never true at the scale you're
  operating. Profile first. HTML rendering has been hardware-accelerated
  for over a decade.

- **"We need offline / native feel."** Wrap the substrate in a host
  (Electron, Tauri, WKWebView). The substrate runs locally; the host
  provides offline assets and native chrome.

- **"It's just a small custom thing."** Small custom things grow.
  The mature substrate also has *N* years of patches for things you
  haven't thought of yet (RTL languages, screen readers, paste from
  Word, drag-and-drop, ⌘C in Linux, …). Skipping it now buys you
  every one of those bugs to discover yourself later.

- **"We need a DSL."** Often you need a *config schema*, not a
  language. JSON+JSON Schema is almost always enough.

## When the principle DOES NOT apply

This skill is about *substrate*, not *abstraction*. There are reasons to
build custom things:

- **The substrate doesn't exist** for your domain. (Real-time audio DSP;
  novel physics simulation; a new database for a workload no existing
  one fits.) Then build it — but build it knowing you've signed up for
  decades of maintenance and tooling.
- **The substrate is licensed in a way that's incompatible** with your
  shipping model. (Replace what's blocked; keep the rest.)
- **You're explicitly building a substrate itself** as the deliverable.
  Then you ARE the maintainers; the skill points in the other direction.
- **The substrate's failure modes are unacceptable** for your domain.
  (Aviation, medical devices, hard-real-time embedded — different
  conversation.)

For most application-layer work — including most "internal tools" — the
substrate exists, and the right move is the mapping work, not the
rebuild.

## Tagline

> Build the residual. Map everything else onto the substrate.

The mark of a senior engineer is not what they build. It's what they
recognize they don't have to.
