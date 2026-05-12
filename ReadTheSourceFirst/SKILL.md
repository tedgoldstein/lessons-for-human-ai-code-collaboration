---
name: read-the-source-first
description: When a dependency behaves unexpectedly, read its source code before chasing maintainers, Stack Overflow, or stale docs. Most surprises are answered in <30 minutes of source reading; the alternatives often take longer and yield less. The source is the only source of truth that's guaranteed to be current.
version: 0.1.0
---

# Read the Source First

The dependency does something you didn't expect. Maybe it's
silently swallowing an error. Maybe a flag isn't doing what its
docstring says. Maybe a method returns one thing in some cases
and another in others.

Two paths from here:

1. Search the docs, browse Stack Overflow, file a GitHub issue,
   ping the maintainer's Discord, wait.
2. Open the source. Find the function. Read it.

Path 2 is often *faster*, and the source is always current.

The principle: **when a dependency does something surprising,
your first move is to read its source.** Not always
exhaustively — often a 5-minute trace of one function is enough.

## When to invoke

- "Why does this library do that?" + the docs are vague.
- A flag's documented behaviour doesn't match what you observe.
- An error message comes from inside a dependency and you can't
  tell what triggered it.
- A method "sometimes returns null" with no documented reason.
- Stack Overflow's top answer is from 2017 and the library has
  had three major versions since.
- A GitHub issue thread is 4 years old and unresolved.
- You're tempted to "just try things and see" instead of
  reading the few lines that would explain it.
- The maintainer's Discord is the official support channel and
  has 8000 unread messages.

## The shape — how to read source efficiently

You're not reading a library cover-to-cover. You're answering
*one question.*

1. **Locate the entry point.** Search for the function or
   class name you're calling. (`grep -r`, your IDE's "Go to
   Definition", `npm explore`, `gem open`, `python -c "import
   x; print(x.__file__)"`.)

2. **Follow only the path that matters.** When the function
   you called branches, follow the branch your inputs hit.
   Skip the other branches. Don't get distracted.

3. **Trust your eyes over the docstring.** The docstring
   may be wrong; the code can't be (about what it does).

4. **Note what surprised you, narrowly.** "Returns null when
   the input is empty" is the answer; you don't need to
   understand the entire module to be done.

5. **Capture the finding.** A one-line comment at your call
   site (`# library X returns null on empty input, not []`)
   pays the next reader.

## Why it matters

1. **The source is always current.** Docs lag, blog posts
   ossify, Stack Overflow is frozen in 2018. The source is
   the version you're actually running.

2. **The source can be grep'd, jumped through, and
   searched.** You can navigate it with the same tools you
   use on your own code. Docs are prose; source is a graph.

3. **You learn the library properly.** Every time you read
   a dependency's source, you understand it deeper. After a
   few such reads, you stop being surprised.

4. **You find bugs you might be hitting.** A behaviour that
   looks wrong might *be* wrong — and the source confirms
   it. Filing a useful bug report is then easy.

5. **It's often faster than the alternatives.** Searching
   takes 20 minutes and yields three half-relevant answers.
   Reading the source for 10 minutes often gives the exact
   answer.

## Practical guidance

- **Use your IDE's "Go to Definition" / "Go to Source."**
  Most IDEs jump through `node_modules`, `site-packages`,
  vendor directories. If yours doesn't, configure it; the
  skill compounds.
- **`bundle open`, `gem open`, `pip show -f`, `npm explore`**
  all open dependency source in your editor. Memorise the
  one for your stack.
- **Skim, don't deep-read.** You're answering a specific
  question; you don't need to understand the architecture.
- **Watch for "TODO" / "FIXME" / "XXX" comments in
  dependency source.** They sometimes encode known
  edge-case behaviour that the docs omit.
- **Read the tests too.** A dependency's test suite is often
  the clearest documentation of expected behaviour.
- **Pin versions and read the source of the pinned
  version.** GitHub's `main` may have moved on; you want
  to know what the version *you depend on* does.
- **Keep a note of finds.** A `notes/dep-quirks.md` in
  your repo captures the surprising-behaviour-of-X you
  discovered, so you don't rediscover it.

## When this principle DOES NOT apply

- **Closed-source dependencies.** SaaS APIs, proprietary
  libraries — there is no source. Fall back to docs +
  empirical testing.
- **Genuinely massive codebases.** A bug deep in Chromium's
  rendering layer is not a 30-minute read. At that scale,
  asking maintainers is rational. (Even then: a focused
  read on the specific subsystem is often productive.)
- **When the docs are first-class and the source is
  obfuscated/minified.** A few libraries genuinely have
  better docs than source. Rare.
- **When you don't read the language.** If the dependency
  is in Rust and you don't read Rust, the cost of reading
  is higher. (But: trying anyway often works; programming
  languages have more in common than not.)

## Common failure modes

- **The fear-of-reading reflex.** Treating the dependency as
  a magic black box. It's not — it's the same kind of code
  you write. The unfamiliarity is the only barrier, and the
  cure is more reading.
- **Reading the wrong version.** Looking at `main` on
  GitHub when you're running v3.2.1. Check the tag/version
  in your lockfile.
- **Going too deep.** Reading half the library to answer a
  ten-line question. Stop when you have your answer.
- **Re-deriving the answer next month.** A finding worth a
  comment at the call site or a note in your repo. Without
  it, future-you re-reads the source for the same answer.
- **Treating GitHub Issues as source.** Issue threads
  sometimes resolve to "yes, that's by design" or "yes,
  that's a bug" — the source tells you which is true *now*.

## Tagline

> Read the source. It's just code.

The thing on your machine is the source of truth. Everything
else (docs, posts, issues, threads) is downstream.

## Sources

Distilled from general engineering practice; echoes the
"read the f*ing source" tradition and the open-source ethic
that source IS the primary documentation.
