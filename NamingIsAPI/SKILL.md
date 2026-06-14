---
name: naming-is-api
description: Names of functions, types, modules, and variables are the most-used surface of your code — favour long-descriptive over short-clever, specific over generic, intention-revealing over implementation-revealing; rename freely. Load when naming or renaming anything, when about to commit a function named `process` / `handle` / `data` / `result` / `helper` / `tmp`, when a comment is explaining what a variable holds, or when two names like `getUser` and `fetchUser` force a reader to open the bodies to tell them apart.
version: 0.1.0
---

# Naming Is API

Every function name is read more than it is written. Every type
name is read more than it is constructed. The cost of a bad name
is paid by the next reader (often you, in three months), every
time they encounter it.

The principle: **names are the smallest unit of API design.** A
function called `processData` is a worse API than one called
`mergeEntriesByEmail`, even though the body is identical. The
caller of `mergeEntriesByEmail` doesn't need to read the body to
know what will happen.

## When to invoke

- You're about to commit code with a function named `process`,
  `handle`, `do`, `run`, `manage`, `data`, `result`, `helper`,
  `util`, or `tmp`.
- A comment is explaining what a variable is called or what a
  function does — the name should have done that.
- A code reviewer asks "what does this return?" when they could
  have read it off the name.
- You wrote `let x = ...` and you can no longer remember what `x`
  is twenty lines later.
- Two functions in the same module are called `getUser` and
  `fetchUser` and you have to read them to learn the difference.
- A constant is named `THRESHOLD` with no unit, no domain, no
  reason.

## The shape — naming heuristics

| Heuristic | Bad | Better |
|---|---|---|
| **Specific > generic** | `processData()` | `mergeDuplicatesByEmail()` |
| **Reveal intent, not implementation** | `quickSort()` | `sortAlphabetically()` |
| **Include units / domains** | `THRESHOLD` | `RETRY_DELAY_MILLIS` |
| **Match the noun-verb shape** | `validation(x)` | `isValid(x)` / `validate(x)` |
| **Match the level of abstraction** | `httpPost` inside a "send email" function | `sendNotification` |
| **No type encoding when the type system already encodes it** | `strName` | `name` |
| **No prefixes for the obvious** | `getUser` when `user` would be a clearer field | `user` |
| **Pairs are symmetric** | `open` / `release` | `open` / `close` |
| **No "data" / "info" / "manager" suffixes** | `UserManager` | `UserDirectory`, `UserSessionStore`, `UserAuthority` — pick one |
| **Async-named when async** | `loadUser()` returning a Promise | `loadUserAsync()` or just `loadUser()` if the language signals async in types |

## Why it matters

1. **Fewer comments are needed.** A function named
   `markTrackAsClosedWithResolution(id, text)` doesn't need a
   docstring repeating the signature in prose.

2. **Onboarding accelerates.** Reading a well-named codebase is
   *learning*. Reading a poorly-named one is *decoding*.

3. **Refactoring is cheaper.** When `process` does too much,
   splitting it is a research project. When
   `mergeEntriesByEmail` does too much, the violation is
   obvious; splitting is mechanical.

4. **Bugs from confusion drop.** Variables named `userId` and
   `user_id` and `uid` floating around a single function is a
   recipe for swapping them. Consistent names eliminate the
   confusion.

5. **Future-search works.** Want to know everywhere we close a
   Track? `grep "closeTrack"`. If the name is `process` or
   `handle` you can't grep anything useful.

## Practical guidance

- **Rename early; rename often.** Modern IDEs make rename
  refactor near-free. A name that fit when the function was
  small often doesn't fit after it grew.
- **Name what it *does*, not how it works.** `sortByDate`
  beats `quickSortByDate` (the algorithm is implementation
  detail). `cache` beats `lruCache` unless the LRU-ness is
  semantic.
- **Name what it *is*, not what it *contains*.** A class
  named `Notification` is clearer than `NotificationData`.
- **Use full words.** Abbreviations are write-time-cheap and
  read-time-expensive. `usr` saves two characters and costs
  you every reader's brain cycle. (Exceptions: established
  domain abbrevs like `URL`, `HTTP`, `id`.)
- **Symmetric pairs match.** `open`/`close`, `connect`/
  `disconnect`, `subscribe`/`unsubscribe`, `enable`/`disable`.
  Don't mix vocabulary: `start`/`close` is jarring.
- **Avoid encoding the type in the name.** `userList` is
  worse than `users`. The type is already in the type. If
  your language is dynamic and the type isn't visible,
  prefer a docstring over hungarian.
- **Boolean names should read as predicates.** `isActive`,
  `hasPermission`, `canRetry`, `shouldFlush`. Not `active`,
  `permission`, `retry`, `flush`.
- **Length scales with scope.** A loop counter can be `i`. A
  module-level constant cannot.
- **Search-friendliness.** Unique names are greppable. Common
  words like `data` or `info` are not. Prefer a unique name
  for things you'll want to find later.

## Common failure modes

- **Premature abbreviation.** `usrSvc.proc(req)` — three
  tokens saved, ten readers confused.
- **Type-suffix sprawl.** `UserData`, `UserInfo`,
  `UserDetails`, `UserDTO`, `UserVO`, `UserModel`, all in
  the same codebase, each subtly different. Pick one.
- **Generic "manager" classes.** `OrderManager`,
  `UserManager`, `ProductManager` — the name is doing no
  work. Replace with a more specific role.
- **Stale names after refactor.** `loadFromDatabase` after
  the function was changed to load from cache. Rename when
  behaviour changes.
- **The hungarian flu.** `strName`, `intCount`, `bIsActive`
  in a typed language. The compiler knows the type; you
  don't need to.
- **Cute names.** `Spaceship`, `Wizard`, `Phoenix` as class
  names for non-spaceship things. Cleverness costs every
  reader.

## When the principle DOES NOT apply

- **Throwaway scripts.** A 20-line one-off can use `x` and
  `data`. The discipline is for code that has a future.
- **Mathematical / domain-conventional names.** In a numerics
  module, `i`, `j`, `k`, `n`, `eps` are conventional; in DSP,
  `fft`, `iir`; in linear algebra, `A`, `b`, `x`. Use the
  field's conventions.
- **Lambda / closure parameters.** A two-line `xs.map(x =>
  x.id)` is fine with short names — the scope is small and
  the meaning is local.

## Tagline

> Read a hundred times, written once.

Names are how the next reader navigates your code. Make
navigation easy.

## See also

- [GrammarIsAlsoAPI](../GrammarIsAlsoAPI/SKILL.md) — the
  next layer up: once the identifiers are right, the
  grammar of invocation (argument order, required vs
  optional, builder vs flat call) is the next surface
  the caller reads.
- [MakeTheWrongThingHard](../MakeTheWrongThingHard/SKILL.md)
  — names are the cheapest layer of making wrong usage
  obviously wrong; what a name can't carry, the type
  system or API shape has to.
- [TheContractIsTheArtifact](../TheContractIsTheArtifact/SKILL.md)
  — the contract's vocabulary outlives any single tool
  that reads it; the names you pick today are what the
  next decade's tooling will key off.

## Sources

Distilled from general engineering practice; echoes Clean Code
(Martin), Code Complete (McConnell), and the
intention-revealing-names tradition.
