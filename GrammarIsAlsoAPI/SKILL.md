---
name: grammar-is-also-api
description: Beyond identifier names (which NamingIsAPI covers), the grammar of commands — verb forms, phrasing, capitalization, sigils, proper-noun choices — is a UX surface; be permissive in input, canonical in output. Load when designing a CLI's verb table or a methodology's vocabulary, when choosing whether to prefix identifiers with a sigil (`R12345678` vs `12345678`), when a conversational AI must recognize commands embedded in natural-language messages, when your identifiers are visually identical to those in an adjacent system, or when the same operation has different phrasings across docs / UI / CLI / chat.
version: 0.1.0
---

# Grammar Is Also API

[NamingIsAPI](../NamingIsAPI/SKILL.md) covers identifier names —
function names, type names, variable names, module names. The
principle: long-descriptive over short-clever, intention-revealing,
specific over generic.

This skill covers the next layer up: **command grammar**. The way
verbs are phrased, how identifiers are presented, what punctuation
or capitalization rules they carry, what sigils mark them. Even
when individual names are good, the *grammar of invocation* can
be sloppy or clean.

The cost of sloppy grammar isn't readability of one identifier —
it's the ability to *recognize* a command in flowing text. The
benefit of well-designed grammar is unambiguous recognition: a
human or AI agent reading a sentence can tell at a glance which
tokens are command invocations, which are identifiers from your
system, and which are just words.

The classic example: a bug-tracker id `vr9xsmxk` looks identical
to a commit SHA prefix, a KV key, an EHR identifier, or any other
8-character lowercase string. Prefix it as `Rvr9xsmxk` — a single
capital letter — and the id is now self-identifying anywhere it
appears in any text.

## When to invoke

- Designing a CLI's verb vocabulary (`fix`, `tap`, `status`,
  `verify`, `close`).
- Designing a methodology / framework / DSL's invocation syntax.
- Choosing whether to prefix identifiers with a sigil (`R12345678`
  vs `12345678`, `#123` vs `123`, `@user` vs `user`).
- Capitalization decisions for product names (`Track.app` vs
  `Track.app`).
- A conversational AI needs to recognize commands embedded in
  natural-language messages.
- An identifier in your system is visually identical to identifiers
  in adjacent systems and the ambiguity causes confusion.
- The same operation has two or three different phrasings across
  the docs / UI / CLI / chat surface.

## The shape

| Property | Aim for | Why |
|---|---|---|
| **Verb forms** | proper-noun-anchored or sigil-marked | Distinguishable from natural-language verbs |
| **Identifier sigils** | a single mandatory character that says "this is *our* identifier" | One-byte tax for unambiguous recognition |
| **Casing of product names** | proper-noun casing for proper nouns | The product name in prose matches the product name in code |
| **Verb phrasing** | parallel construction across the verb table | One memorable shape; everything is a variant |
| **Optional words for clarity** | accepted as synonyms, not required | Permissive in input, canonical in output |
| **One canonical form per operation** | documented; the tooling normalizes to it | Predictable diffs and stable text matching |

## Why it matters

1. **Identifiers appear in flowing prose.** Commit messages, PR
   bodies, email threads, chat, comments, AI responses. A bare
   8-character id in that prose is invisible — you have to know
   what to look for. A prefixed id (`R...`, `#...`, `@...`) is
   self-identifying.

2. **AI agents can do precise regex matches.** Bare-form
   identifiers force the AI to do fuzzy pattern matching or rely
   on context. Prefixed identifiers admit a tight regex
   (`R[a-z0-9]{8}\b`) with near-zero false positives. The whole
   "paste an email, get matching items" workflow becomes
   feasible.

3. **Grammar signals which surface a token belongs to.** `fix
   X` could be a thousand things. `Fix Track X` belongs
   unambiguously to one methodology. `@person fix issue #123`
   wires together GitHub conventions; if you're not using
   GitHub conventions, your grammar should be just as legible.

4. **Casing carries semantic weight.** A product name capitalized
   matches how it reads in marketing copy and how users
   reference it in conversation. An app called `Track.app` reads
   as "the Track dot app file"; `Track.app` reads as "the Track
   application." For the maintainer, the casing is just
   convention; for the user, it's invitation.

5. **Parallel construction is mnemonic.** When every verb has
   the same shape — `Verb Track <id>`, or `Verb #<id>`, or
   `@person Verb #<id>` — learning one verb teaches the
   grammar of all of them.

## Practical guidance

- **Pick a single sigil and apply it ruthlessly to your
  identifiers.** `R` for Track ids. `#` for issue numbers.
  `@` for users. `:` for labels (`bug:high-priority`). The
  choice doesn't matter as much as the consistency.
- **Document the canonical form once.** A SKILL.md, schema,
  or README that says "Track ids look like `R<8-char-body>`"
  and is unambiguous. Every consumer reads the spec; every
  consumer agrees.
- **Be permissive in input, canonical in output.** Accept
  both `Rvr9xsmxk` and the bare `vr9xsmxk` at parse time;
  normalize to the prefixed form everywhere your tooling
  writes. (Postel's law: "be conservative in what you send,
  liberal in what you accept.")
- **Match the casing of product names in code, docs, and
  prose.** `Track.app` in the README, `Track.app` in the
  Xcode `PRODUCT_NAME`, `Track.app` in commit messages. Lower-
  cased tracked-directory paths (`swift/Track/`) are fine —
  those are file paths, not product names.
- **Make grammar visible in the docs.** A verb table is
  worth printing. `Verb Track <id>` showing the canonical
  form, with bare-form `Verb <id>` noted as an accepted
  synonym, lets new contributors learn the shape quickly.
- **Test the grammar with a regex.** If your canonical
  identifier form can't be matched with a precise regex,
  it's too loose. `R[a-z0-9]{8}\b` is testable; `vr9xsmxk`
  alone is not.

## Common failure modes

- **Sigils everywhere, redundant.** `@user-123` where the
  `@` carries no signal because the rest of the system also
  uses `user-` prefix; you've added a sigil that doesn't
  disambiguate.
- **Inconsistent casing across surfaces.** The README says
  `Track.app`; the Xcode bundle name is `Track.app`; the
  menu bar shows `Track`; the docs alternate. New readers
  can't tell which is "right."
- **Verb table with five different shapes.** `fix <id>`,
  `status [<id>|all]`, `verify <id> <crit> <verdict>`,
  `merge <id1,id2> and run a demo`. No parallel
  construction; nothing's mnemonic. Each verb is its own
  micro-language.
- **The "we'll grandfather in the old form forever" trap.**
  When migrating from bare-form to prefixed-form ids,
  accepting both indefinitely means the canonical form
  never wins. *Tooling normalizes to canonical at write
  time*; old text continues to read; new text uses
  canonical. The grandfathering is at the read boundary,
  not the write boundary.
- **Treating grammar as bikeshedding.** "Who cares about
  casing?" Users care. AI agents care (the regex is
  load-bearing). Future-you cares when reading a commit
  message from 2026 and wondering whether it referred to a
  Track or a Cloudflare KV key. Grammar disambiguates.
- **Inventing grammar in isolation.** If you're building a
  CLI in an ecosystem that already has conventions (`git`,
  `kubectl`, `gh`), borrow them. `verb noun [flags]` is the
  Unix grammar; deviating costs the user the learning
  transfer.

## When grammar should stay invisible

- **Mature ecosystem conventions** — `git commit`,
  `npm install`, `kubectl apply` — fit so cleanly in their
  context that no special grammar is needed. The
  `verb noun [flags]` shape is the grammar.
- **Identifiers that only ever appear inside the tool.**
  An internal database id that no user ever sees in prose
  doesn't need a sigil; the tool always renders it inside
  its own UI. Sigils help in *cross-surface* contexts
  (chat, email, code).
- **Pure data formats.** A JSON object with an `id` field
  doesn't need the id to be self-identifying — the JSON
  schema already signals what it is.

## Worked example

The Track methodology started with bare 8-character ids:
`vr9xsmxk`, `qa6stpkt`, `0d9z30d9`. They were collision-
resistant and short, but visually indistinguishable from
KV keys, commit SHA prefixes, and other 8-char tokens that
appeared anywhere in the codebase. When a Track's Internal
notes referenced `y1bd5y1b`, a reader had to know from
context that it was a Track id.

During a design session, the user observed that an
email-like message — *"Look at R12345678, R7654210"* —
could trigger a paste-and-search feature in Track.app
that pulled up exactly those Tracks. The `R` prefix was
the key: a tight regex `R[0-9a-hjkmnp-tv-z]{8}` matched
Track ids and only Track ids, with no false positives.
The same regex on bare-form ids would have matched commit
SHA prefixes, KV keys, and arbitrary 8-char hashes.

The user filed Track `qa6stpkt` proposing the change: new
canonical id form `R<8-char-body>`, parser accepts both
forms but normalizes to prefixed, migration done via a
new `Track migrate-ids` CLI verb. Then immediately filed
a sibling Track `ykjnb789` for the paste-and-search
feature that the prefix enables. The two Tracks together
make the case for grammar-as-API: the sigil isn't
cosmetic, it's what makes a real workflow possible.

A parallel grammar fix landed in the same session:
`Track.app` (the bundle name) was renamed `Track.app`
(proper-noun casing). The repo path `swift/Track/` stayed
lowercase — that's a directory, not a product name. The
discipline: casing matches the noun's role.

## Tagline

> Names are identifiers; grammar is invitation.

A name says *what*. Grammar says *how you address it*. In
flowing prose and AI-generated text, the grammar is what
lets readers find your system's tokens in the noise.

## See also

- [NamingIsAPI](../NamingIsAPI/SKILL.md) — the parent
  principle. Good names are a precondition; this skill
  layers grammar on top.
- [TheContractIsTheArtifact](../TheContractIsTheArtifact/SKILL.md)
  — the contract's identifiers appear in every consumer's
  text. Grammar is what keeps them recognizable across
  consumers.
- [MultiAICollaborationViaGit](../MultiAICollaborationViaGit/SKILL.md)
  — multi-AI sessions communicate via prose (commit
  messages, PR bodies, file contents). Grammar that
  AIs can match precisely is grammar that survives the
  multi-AI pipeline.

## Sources

Inferred from observing the Track methodology's
mid-design grammar refinements (2026-05): the
`Track.app → Track.app` casing change, the
`<id> → R<id>` sigil migration, the `fix Track <id>`
verb-form proposal. The general principle is older —
linguistics and HCI have known for decades that grammar
disambiguates — but the AI-augmented context elevates
"can a regex match your identifier?" from cosmetic to
load-bearing.
