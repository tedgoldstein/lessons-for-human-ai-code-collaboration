---
name: measure-dont-guess
description: When something is slow, heavy, or janky, capture a real profiler trace before changing any code. The instinct — especially an AI agent's — is to reason about which line is the culprit and optimise it; that reasoning is plausible and frequently wrong. The skill is twofold — reach for a trace (Apple Instruments Time Profiler, os_signpost intervals, perf, Tracy, Chrome tracing, OpenTelemetry spans) before optimising, and instrument your own code with named signposts so the trace speaks your domain's vocabulary instead of raw stack frames.
version: 0.1.0
---

# Measure, Don't Guess

The app drops a frame. A request takes 800ms. Memory climbs over
an hour. The instinct is to look at the code, find the loop or the
allocation or the lock that *looks* expensive, and optimise it.

That instinct is a hypothesis dressed as a conclusion. It is
plausible — and on real systems it is wrong often enough that
acting on it without checking is a coin flip. The hot path is
usually somewhere you didn't expect: a logging call inside a tight
loop, a layout pass triggered by a property you set "for safety,"
a JSON re-parse on every access, a retain cycle nobody sees.

The principle: **before you optimise anything, capture a real
measurement of where the time or memory actually goes.** A
profiler trace is the ground truth; your intuition about the
bottleneck is a guess until the trace confirms it.

The second half of the principle, the one that makes the first
half pay off repeatedly: **instrument your own code so the trace
is legible.** A raw sampled profile is a wall of stack frames. A
trace annotated with named intervals — `os_signpost` /
Points of Interest on Apple platforms, spans elsewhere — reads as
a narrative of *your* operations: "decode 12ms, layout 4ms, draw
0.5ms." You design that legibility in, the same way you design
logs (see [`LogsAreAFeature`](../LogsAreAFeature/SKILL.md)).

## When to invoke

- "This feels slow" / "the UI stutters" / "scrolling janks."
- A request, job, or render is over budget and you're about to
  guess which part is responsible.
- You're tempted to rewrite a loop, add a cache, or swap a data
  structure *for performance* without a number that says it's the
  problem.
- Memory grows and you're about to start commenting out code to
  find the leak.
- It's slow but the CPU looks idle — a sign of lock contention or
  threads blocked waiting, which a sampling profiler can't see.
- An AI agent (or you) proposes an optimisation whose justification
  is "this is probably the expensive part."
- A benchmark moved and nobody can say which change moved it.
- You're optimising something that turns out to run once at startup
  and never again.
- A code review comment says "this could be slow" with no trace
  attached.

## The shape — capture a trace first

On Apple platforms the tool is **Instruments**. The two workhorse
templates:

- **Time Profiler** — samples the call stack on every core at a
  fixed interval. Tells you where wall-clock time goes, weighted by
  how often each frame is on the stack. Start here for "why is this
  slow."
- **Allocations / Leaks** — every allocation, retain/release, and
  the leaked graph. Start here for "why does memory grow."

The flow that matters is not "open Instruments and stare." It is:

1. **Reproduce the slow thing under the profiler.** Drive the exact
   gesture, request, or workload. A profile of the wrong workload
   answers the wrong question.
2. **Read the heaviest frames, top-down.** Instruments' call tree,
   inverted, puts the genuinely hot leaf functions first. That leaf
   is your suspect — not the function you assumed.
3. **Change one thing.** See [`OneChangeAtATime`](../OneChangeAtATime/SKILL.md).
   Re-profile. Confirm the number moved *and* that you didn't push
   the cost elsewhere.

The same discipline, other stacks: `perf record` / `perf report`
and `flamegraph` on Linux, the Tracy or Optick profiler in games,
Chrome DevTools' Performance panel for web, `py-spy` /
`cProfile` + `snakeviz` in Python, async-profiler / JFR on the
JVM, `pprof` in Go. The tool changes; the move — *trace, then
touch* — does not.

## Making your code measurable

Time Profiler tells you *which function*. It does not tell you
*which operation* — and your operations rarely map one-to-one onto
functions. To get a trace in your own vocabulary, mark named
intervals in the code:

```swift
import OSLog

let signposter = OSSignposter(subsystem: "com.example.radar",
                              category: "render")

func renderFrame(_ scene: Scene) {
    let state = signposter.beginInterval("renderFrame")
    defer { signposter.endInterval("renderFrame", state) }

    let decode = signposter.beginInterval("decode")
    let pixels = decode(scene.image)
    signposter.endInterval("decode", decode)

    let layout = signposter.beginInterval("layout")
    let frame = layout(scene, pixels)
    signposter.endInterval("layout", layout)
}
```

Open the **Points of Interest** instrument and the timeline now
shows `renderFrame` containing `decode` and `layout` as labelled
bars with exact durations — a story, not a sample cloud. Signposts
are nearly free when no Instruments session is attached, so they
can live in shipping code. The non-Apple equivalents are tracing
spans (OpenTelemetry, `tracing` in Rust, Zipkin/Jaeger for
distributed work) — same idea: the program emits a structured,
named timeline of what it's doing.

### Signpost the wait, not just the work

The case where this earns its keep most is **lock contention**,
because contention is invisible to a sampling profiler. Time
Profiler samples the CPU; a thread *blocked* on a lock burns no
CPU, so it shows up as idle — or doesn't show up at all. The
800ms you're chasing can be 780ms of threads parked waiting on
each other, and the sampler will never point at it.

With modern Swift concurrency (`actor`s, `OSAllocatedUnfairLock`,
or a hand-rolled critical section) you can map that wait-time
straight onto the timeline by wrapping the lock and signposting
the two intervals that matter — the **wait** (blocked before
acquisition) and the **hold** (time inside the critical section,
i.e. how long you keep everyone else out):

```swift
import os

final class InstrumentedLock {
    private let lock = OSAllocatedUnfairLock()
    private let signposter = OSSignposter(subsystem: "com.example.radar",
                                          category: "locks")

    func withLock<R>(_ name: StaticString, _ body: () -> R) -> R {
        let id = signposter.makeSignpostID()

        // The contention: time spent blocked before we get in.
        let wait = signposter.beginInterval("lock wait", id: id, "\(name)")
        lock.lock()
        signposter.endInterval("lock wait", wait)

        // The hold: time we keep the critical section to ourselves.
        let held = signposter.beginInterval("lock held", id: id, "\(name)")
        defer {
            signposter.endInterval("lock held", held)
            lock.unlock()
        }
        return body()
    }
}
```

Now the Points of Interest timeline shows `lock wait` and
`lock held` bars right alongside the CPU and Thread State tracks.
A fat `lock wait` bar that lines up with another thread's long
`lock held` bar *is* the contention, drawn to scale — the thing
the sample cloud structurally cannot show you. The same wrap goes
around any custom synchronisation; an `actor`'s serial executor
can be instrumented the same way at its hop points.

This is the durable investment. The first trace finds today's
bottleneck; the signposts you leave behind make *every future*
trace legible in seconds instead of minutes.

## Why it matters

1. **Intuition mis-predicts the hot path.** The expensive thing is
   routinely the boring thing — a string format, an autorelease, a
   re-layout — not the algorithm you were proud of. The trace
   corrects you for free.

2. **You stop optimising code that doesn't run.** Half of
   "obviously slow" code executes once or never on the hot path.
   A trace shows zero samples there and saves you the work.

3. **The number is the acceptance test.** "Faster" is an argument;
   "240ms → 90ms in the trace" is a fact. It also catches the
   optimisation that moved the cost rather than removing it.

4. **Named signposts compound.** Once your operations are
   instrumented, the next regression is diagnosable by anyone who
   opens the trace — including a cold reviewer or another agent.

5. **It guards against confident AI guesses.** An agent will
   cheerfully generate a plausible 40-line optimisation for a path
   that the trace shows costs 0.3%. Requiring a trace first turns a
   guess into a measurement.

## Practical guidance

- **Profile a release/optimised build.** A debug build's hot spots
  are not the shipping build's. On Apple platforms profile the
  Release configuration; elsewhere build with optimisations on.
- **Reproduce, then record.** Have the exact repro ready before you
  hit record, so the trace is mostly the thing you care about.
- **Invert the call tree.** The top-down tree hides leaf costs
  under framework frames; the inverted tree surfaces the actual hot
  leaves.
- **Leave the signposts in.** They cost almost nothing idle and pay
  back on the next investigation. Treat them like logs: a designed,
  permanent surface.
- **Capture a baseline number, then a post-change number.** Without
  the before, "it's faster now" is unfalsifiable.
- **Match the workload to the question.** Startup time, steady-state
  throughput, and worst-case latency are three different traces.
- **Pair with [`TimeBoxedExperiments`](../TimeBoxedExperiments/SKILL.md):**
  give the investigation a budget so a profiling rabbit-hole doesn't
  swallow the afternoon.

## Common failure modes

- **Optimising from a code read.** Deciding the bottleneck by
  staring at the source. This is the default failure; the cure is a
  trace.
- **Profiling the wrong build or workload.** A debug build, an
  unrepresentative input, or the simulator instead of the device —
  the trace is real but answers a question you didn't ask.
- **No baseline.** Changing code and declaring victory without a
  before-number to compare against.
- **Moving the cost, not removing it.** The function got faster; the
  work reappeared in its caller. Only a re-trace catches this.
- **Micro-optimising a cold path.** Hand-tuning code the trace shows
  at 0.1%. Effort with no payoff.
- **A trace with no signposts.** A wall of `__platform_memmove` and
  `objc_release` frames with no domain labels. Readable, but it
  takes ten times longer than a signposted timeline.

## When this principle DOES NOT apply

- **The cost is asymptotic and obvious.** An O(n²) over a list
  that's about to be millions of rows doesn't need a trace to know
  it's wrong. Fix the complexity; measure later.
- **Correctness, not performance, is the problem.** Don't profile a
  bug; debug it.
- **The measurement costs more than the win.** For a script that
  runs once a quarter and takes four seconds, the trace is overkill.
  Match effort to stakes
  (see [`FormalVsImprovisational`](../FormalVsImprovisational/SKILL.md)).
- **You already have the trace.** If a recent, representative
  profile already names the culprit, act on it — the principle is
  "measure first," not "re-measure endlessly."

## Tagline

> Trace, then touch.

The profiler is not a last resort for when you're stuck. It's the
first move — the thing that turns "I think this is slow" into "this
is slow, here, by this much."

## Sources

Distilled from general engineering practice and a macOS profiling
session using Apple Instruments (Time Profiler + `os_signpost`
Points of Interest). Echoes the long-standing "measure, don't
guess" / "profile before you optimise" tradition — Knuth's caution
against premature optimisation, and Apple's own guidance to
instrument code with signposts rather than reason about hot paths
in the abstract.
