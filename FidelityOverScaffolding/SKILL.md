---
name: fidelity-over-scaffolding
description: Scaffolding — fakes, stubs, mocks, hand-written fixtures — is fine, useful, and often required to build fast and shape an interface. But a green test against scaffolding proves the scaffolding, not the building. Only fidelity (the real runtime, API, OS, peer, storage) proves the structure stands in the earthquake. Use when a test "passes" but the code has never run against the real thing, especially at a boundary.
version: 0.1.0
---

# Fidelity Over Scaffolding

**Scaffolding is fine, useful, and often required — but it won't hold the building up in an earthquake.**

Fakes, stubs, mocks, in-memory doubles, and hand-written fixtures let you build fast, shape an interface before the real thing exists, and keep a tight inner loop. Keep using them. But a green test against scaffolding proves the *scaffolding* — not the building. Only **fidelity** — exercising the real runtime, the real protocol, the real platform, the real peer — proves the structure stands when the ground moves: production, a different OS, real concurrency, another agent's *actual* response.

This is the companion to [MultiLevelTesting]: the pyramid tells you to combine levels; this tells you that **the doubles you stand in for real dependencies must be faithful, or the lower levels lie to you.**

## When to invoke

- "It passes" — but the code has never actually run against the real API / runtime / OS / other process / peer.
- You're asserting against a fixture whose shape *you wrote* (response JSON, error codes, return types, event ordering).
- "Should I mock this or use the real thing?" — for something that crosses a boundary.
- A bug shipped (or a live run 404'd / hung) that the green unit tests didn't catch.
- You're about to build *another storey* on a layer only ever proven against scaffolding.
- Porting to a new OS / runtime / dependency version and the existing tests are all fakes.

## The trap

A hand-written fake encodes **your assumption** of how the real thing behaves. When the assumption is wrong, the fake passes *and* reality fails — and the green check makes the bug invisible. The more convenient the fake, the easier it is to assume wrong. **A passing test built on a wrong fake is a load-bearing lie — worse than no test, because the green check hides the crack.**

## What earthquakes are actually made of

The failures live exactly at the boundary the fake replaced. Real cases — generalize them:

- **Response shape.** A client read `r.campaign_id`; the real API returns `{ campaign: { campaign_id } }`. The unit test used the wrong fake shape, passed green, and the *first* live call 404'd.
- **Platform semantics.** Reading a PTY master after the child exits returns `0`/EOF on macOS but `-1`/EIO on Linux. macOS-shaped logic hung on Linux. No mock would have told you.
- **Environment.** A binary on `PATH` in an interactive shell but absent in a background process → silent `exec` failure → empty result, no error.
- **Storage semantics.** A fake `transaction` with no isolation/rollback cannot prove idempotency or a capture-lock the way a real (SQLite) store does — it "passes" the test it can't actually exercise.

## What to do

1. **Keep the scaffolding** — for speed, interface shaping, pure-logic units, and the happy-path inner loop. Fakes aren't the enemy; they're temporary.
2. **For anything that crosses a boundary** (network, runtime, OS, another process, storage, another agent), get **fidelity before you trust it**: run it live, or test against the *real* runtime — real workerd (Miniflare), a real container, the real CLI, the real endpoint.
3. **Run the live path at least once** before declaring done — and again whenever the boundary could shift (new OS, new dependency version, new peer, new prod config). The *first* live run is where the real bugs surface; budget for it.
4. **When a fake and reality could disagree, the fake is the suspect.** Pin the fake to a *captured real response*, not to your memory of the shape.
5. **Don't build the next floor on unproven scaffolding.** Prove the slab, then build — otherwise a later migration forces re-validating everything above.

## The line

Scaffolding gets you *up* the building. Fidelity is what keeps it *standing* when the earthquake comes. Use both — and never mistake one for the other.
