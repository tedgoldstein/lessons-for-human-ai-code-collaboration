---
name: make-the-wrong-thing-hard
description: Design APIs and modules so incorrect usage is harder than correct usage — types, newtypes, sum types, RAII / scope guards, typestates, builders, lints; the caller shouldn't have to remember anything. Load when designing a call site others (or future-you) will use, when a reviewer keeps writing "remember to call close()" or "don't forget to initialise first", when a bug came from passing the wrong kind of string or a swapped boolean argument, or when a README carries "you must call X before Y" warnings.
version: 0.1.0
---

# Make the Wrong Thing Hard

The best APIs require no instructions. The user holds the type the
API expects; the type can only be constructed validly; nothing
else compiles. The worst APIs are correct *if used correctly*,
which is to say they ship bugs.

The principle: **shift the cost of correctness from runtime to
compile time, from convention to enforcement, from "remember to..."
to "you literally can't do otherwise."**

## When to invoke

- A code reviewer is writing comments like "remember to call
  `close()`" or "don't forget to initialise before use."
- A bug came from passing a string where a different kind of
  string was meant ("the wrong ID type").
- Documentation has a "you must call X before Y" warning.
- A new contributor's first bug is a misuse of an existing API.
- A method "works for most callers" but fails subtly for one.
- A `null` snuck somewhere and the test suite didn't catch it.
- A function takes `(bool, bool, bool, bool)` arguments and people
  routinely swap them.

## The shape — techniques in order of leverage

1. **Make invalid states unrepresentable** *(types)*
   - Replace `enum Status { OK, ERROR }` + `errorMessage: String?`
     with a sum type: `enum Result { Ok(value) | Err(message) }`.
     The compiler refuses to give you an error message on an OK
     result.
   - Replace `User { id: String }` with `User { id: UserID }`
     where `UserID` is a distinct type from `OrderID`. Mixing
     them stops compiling.
   - Replace `(name: String, isAdmin: Bool)` parameters at every
     call site with a `Principal` enum carrying its own
     authority. The caller can't construct an unauthorised
     `Principal` by mistake.

2. **Make resources self-managing** *(RAII / scope)*
   - Files, locks, sockets, transactions: open in a `using` /
     `with` / `defer` / Drop block so they're closed
     *automatically* on scope exit. "Forgot to close" can't
     happen if the type closes itself.

3. **Make the right path the easy path** *(API ergonomics)*
   - If the safe operation is 12 lines and the unsafe one is 3,
     people will use the unsafe one. Make the safe path the
     shortest. If sanitisation is required, make sanitisation
     the default and "unsafe" the explicitly-named opt-out.

4. **Move guards from runtime to compile time** *(typestates,
   builders)*
   - `Request.build().header("x").body("y").send()` — `send()`
     only exists once required parts are present.
   - A `Connection` type that has separate `Connecting`,
     `Open`, and `Closed` typestates so you can't `write` to a
     closed connection.

5. **Lint or fail when the principle can't reach the types**
   - Custom lints for project-specific rules ("no raw SQL
     strings — use the query builder"). Block at CI.
   - `assert` invariants at module boundaries so violations
     stop in tests instead of in production.

6. **Encapsulate state; never trust callers to mutate
   correctly** *(private / module boundaries)*
   - If a field has invariants, hide it. Expose mutators that
     preserve the invariants.
   - The user of your module shouldn't be able to construct an
     impossible state, even with valid-looking calls.

## Why it matters

- **Bugs that "can't happen" don't happen.** A whole class of
  bug goes away — not "almost never," but mathematically zero.
- **Onboarding cost drops.** New contributors can't make the
  mistake. They don't need to read the warning. They don't
  need to read the docs.
- **Refactoring is mechanical.** Change the type; the compiler
  finds every call site that needs updating. No fan-out fear.
- **Code review focuses on intent.** Reviewers spend zero
  energy on "did you remember to..." and full energy on "is
  this the right design?"

## Practical guidance

- **Use newtypes / typedef wrappers liberally.** `UserID`,
  `OrderID`, `SessionToken` as distinct types are cheap and
  catch real bugs. Don't pass them around as `String`.
- **Prefer enums over booleans + bool flags.** `mode: ReadWrite`
  beats `(readable: true, writable: true)`. Three booleans is
  almost always a sum type in disguise.
- **Validate at the boundary, trust inside.** Convert untrusted
  input to a typed-and-validated form *once* at the system
  edge. Inside, the types guarantee the invariants — no
  defensive re-validation needed.
- **Encode lifecycle in types when it matters.** Half-open
  connections, drafts that aren't yet published, requests
  that haven't been authenticated: separate types, not flags.
- **Reach for builder patterns when there are required +
  optional fields.** `build()` only compiles when required
  fields are set.
- **When typestates are too heavy, use phantom types or
  marker types.** Lightweight, still compile-checked.
- **When you can't reach into the language, write a lint.**
  Encode the rule in something that fails CI, not in a
  README warning.

## Common failure modes

- **"It's just a string."** Then it's also any other string.
  Wrap it.
- **Boolean explosion.** Three or more `bool` parameters
  almost always means a missing enum.
- **The README warning.** "You must call `init()` before
  `start()`." If you can write that in prose, you can encode
  it in types.
- **Stringly-typed APIs.** `dispatch("user.created", payload)`
  where the event name is free-form. Make events typed; the
  dispatcher only accepts known events.
- **Validation at every layer.** Defensive null-checks at
  every internal call. The invariant should be guaranteed by
  construction, not re-checked everywhere.
- **Custom config schemas as untyped maps.** `config["timeout"]`
  with no schema is a runtime crash waiting. Use a typed
  config object.

## When the principle DOES NOT apply

- **Throwaway scripts / experiments.** Don't gold-plate a
  one-off Python script with typestates. The discipline is
  for code that will be maintained.
- **Interop with untyped systems.** At the JSON / HTTP / DB
  boundary you're forced to validate at runtime. Validate
  once, convert to typed form, then trust the type.
- **When the type system isn't expressive enough.** Some
  invariants can't be expressed in your language's types.
  Reach for the next-best option (asserts, lints,
  documentation) — but don't pretend the warning is
  enforcement.

## Tagline

> The user shouldn't have to remember anything.

If the API makes the right thing easy and the wrong thing hard,
the bugs that would have existed are bugs that can't exist.

## See also

- [NamingIsAPI](../NamingIsAPI/SKILL.md) — names are
  the first and cheapest layer of making the right
  thing the obvious thing; a clear identifier resolves
  the question before the type system has to.
- [GrammarIsAlsoAPI](../GrammarIsAlsoAPI/SKILL.md) —
  the next layer up: the grammar of invocation
  (argument order, required vs optional, builder vs
  flat call) steers callers towards correct usage.
- [IdempotentByDefault](../IdempotentByDefault/SKILL.md)
  — "retry is safe" encoded into the operation itself
  rather than left as a discipline the caller must
  remember.
- [TheContractIsTheArtifact](../TheContractIsTheArtifact/SKILL.md)
  — the contract can encode correctness in places the
  renderer or runtime can't reach around, making
  whole categories of misuse unrepresentable.

## Sources

Distilled from general engineering practice. Echoes "make
illegal states unrepresentable" (Yaron Minsky), the typestate
pattern, and Rust's borrow-checker philosophy.
