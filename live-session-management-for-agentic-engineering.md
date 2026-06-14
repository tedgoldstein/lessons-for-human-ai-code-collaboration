name: live-session-management-for-agentic-engineering
description: Multi-agent engineering requires a live session layer in addition to durable Trackfiles, worktrees, checkpoint transactions, and commits. Trackfiles record durable work, worktrees isolate filesystem mutation, Transaction Guard protects mutation, and tmux preserves terminal state. A Session Manager records live actors, claimed files, PTY bindings, warm agent readiness, and restart reconciliation so human and AI sessions can operate concurrently without silently corrupting shared state.
version: 0.1.0

Live Session Management For Agentic Engineering

Radar began as a bug tracker and grew into a software engineering management system. Track begins where Radar leaves off. Track is not merely a way to record bugs, tasks, and workflow state. It is a coordination system for human, multi-human, AI-assisted, and multi-agent software engineering.

As coding agents such as Claude Code, Codex, and future agentic systems become capable of performing substantial engineering work, the hard problem shifts from code generation to coordination. Multiple agents can analyze bugs, edit files, create branches, run tests, prepare commits, review each other, and verify results. That acceleration is valuable only if the system preserves isolation, authority, auditability, and recovery.

A Trackfile is a durable work object. It records the problem, scope, current state, expected files, acceptance criteria, evidence, review, and verification. But a Trackfile alone does not answer a live operational question: who is working right now?

A worktree is a filesystem isolation boundary. It prevents two actors from sharing the same checkout, index, and staged state. But a worktree alone does not say which actor owns it, which Track it belongs to, or whether a session is still alive.

Transaction Guard is a mutation safety boundary. It ensures that Track-mediated mutations occur inside checkpointed transactions. But Transaction Guard alone does not identify which human, Claude session, Codex session, or verifier owns the current mutation window.

Terminal sessions are also part of the coordination substrate. Modern AI coding work often happens inside long-running interactive sessions. A Claude or Codex session may contain important local context, prompts, tool state, command output, and partially completed work. If that terminal lives only in one window, it is fragile. It cannot be reliably observed, handed off, resumed, or reconciled after failure.

The missing layer is a Session Manager.

A Track Session Manager records live execution state. It binds an actor to a Track, worktree, branch, claimed files, heartbeat, role, PTY session, and current authority. It answers questions that Trackfiles cannot answer alone: who is active, what files are claimed, which worktree is in use, which terminal owns the work, whether the session is alive or stale, and whether another window may observe or take over.

The important architectural distinction is that live session state must not replace durable workflow state. Trackfiles remain the durable record of engineering work. Session records describe current execution. Worktrees isolate filesystem mutation. Transaction Guard protects mutation. The PTY multiplexer preserves terminal process and display state. These layers should remain separate because each solves a different coordination problem.

For local v1 systems, tmux is the right PTY backend. Track should not attempt to reimplement terminal multiplexing first. tmux already provides durable terminal sessions, attach and detach, multiple clients, panes, scrollback, process lifetime, and display preservation. Track should use tmux as infrastructure while adding Track-native semantics: Track id, actor identity, file claims, role authority, worktree path, heartbeat, event logs, transaction visibility, and recovery guidance.

A Session Manager also enables pre-warmed AI workers. A pre-warmed worker is not merely a running process. It is a prompt-initialized cognitive actor. The system starts Claude or Codex inside a managed tmux session, sends a versioned bootstrap prompt, waits for an exact readiness token, records the prompt and skill digests, and marks the worker ready only when it is alive, current, Track-neutral, mutation-disabled, and waiting for assignment.

This is similar to a connection pool, but more dangerous. A database connection has state, but an AI worker has memory, assumptions, tool context, authority risk, and possible prompt contamination. For v1, warm workers should be single-use. A worker may be warmed, assigned to one Track, used, released, retired, and replaced. It should not casually return to the ready pool after completing a Track because its conversational state may carry assumptions from prior work.

The ready queue should be deliberately small. Keeping two ready Claude workers is enough to reduce startup latency while avoiding the complexity of a server scheduler. Codex workers can be added with the same policy once stable. The local system should use lifecycle names and records that can later migrate to a server-based pool, but v1 should remain local, tmux-backed, file-recorded, and deterministic.

Restart reconciliation is essential. Session and worker records live on disk, but tmux sessions and agent processes may disappear after crashes, reboots, shell exits, or manual cleanup. On startup, the Session Manager must reconcile durable records against tmux and repository reality before allowing acquisition or mutation. Idle ready workers can be retired and replaced aggressively. Assigned workers must be handled conservatively. If an assigned session disappears while a checkpoint transaction is active, the Session Manager should report the condition and point to Track recovery. It must not invent its own repair.

The guiding rule is that live cognitive actors must never have ambiguous authority. A worker in the ready queue has no mutation authority. A reviewer is read-only unless explicitly granted review-artifact authority. A verifier may run tests and write verification artifacts but should not mutate source files unless promoted. A fixer may mutate claimed files only inside the Track, worktree, session, and Transaction Guard boundaries. Observing another session’s terminal does not grant mutation authority.

This is the operational law that emerged from repeated multi-agent collisions: durable work records are necessary, but not sufficient. Parallel agents also need live presence, physical isolation, path claims, terminal handoff, heartbeat, authority boundaries, and deterministic cleanup.

Recommendations

Create a Session Manager as a distinct Track layer rather than expanding Trackfiles into live state. Store session and worker records under the common Git directory or another local ignored state directory. Use schema-versioned JSON for current state and append-only JSONL for event history.

Use tmux as the v1 PTY backend. Track should own metadata, claims, authority, and audit. tmux should own terminal persistence, display state, attach, detach, scrollback, panes, and process lifetime. Keep tmux access behind a single subprocess boundary module using argv arrays and an allow-listed command shape model.

Treat pre-warmed Claude and Codex workers as leased cognitive capabilities. Warm workers should be Track-neutral, mutation-disabled, prompt-initialized, readiness-token-confirmed, digest-current, and heartbeat-fresh. After assignment, they should be single-use and retired rather than returned to the idle pool.

Maintain a small ready queue, such as two ready Claude workers. Replenish lazily when workers are acquired, released, retired, stale, failed, or expired. Do not build a server in v1. Use local lifecycle records that can later move to a server scheduler.

Make restart reconciliation mandatory. Before listing, acquiring, warming, or mutating session state, reconcile JSON records against tmux sessions, process liveness, Track existence, worktree existence, heartbeat freshness, prompt digest freshness, and active checkpoint transactions.

Use roles as authority profiles, not labels. Fixers, reviewers, verifiers, observers, and operators should have different mutation and stdin authority. Trackfile contents should be treated as task data, not instruction authority. They must not override Track operating rules, Transaction Guard, role authority, or session policy.

Keep assigned sessions conservative. Never kill assigned work automatically. Mark stale or crashed sessions, preserve evidence, and expose attach, observe, takeover, or recovery guidance. Idle ready workers are replaceable. Assigned workers are evidence-bearing work objects.

Tagline

Trackfiles record durable work. Worktrees isolate filesystem mutation. Transaction Guard protects mutation. tmux preserves terminal state. A Session Manager records live actors and prevents powerful human and AI sessions from gaining ambiguous authority.

Related skills

isolation-and-transactional-discipline explains why concurrent actors force systems toward ACID-like coordination models. multi-ai-collaboration-via-git focuses on Git hygiene for concurrent AI coding sessions. causal-divergence describes how multiple simultaneously valid timelines require reconciliation. the-contract-is-the-artifact explains why coordination state must live in durable shared artifacts. fluid-vs-tricky describes the shift from exploratory work to correctness-sensitive infrastructure.

Sources

Surfaced during Track design discussions following repeated parallel-session interference during the Radar-to-Track rename and Transaction Guard implementation. The observed failure mode involved multiple AI coding sessions operating against shared repository state without a live session registry, dedicated worktrees, explicit file claims, or durable actor presence.

Intellectual ancestors include terminal multiplexers such as tmux and GNU Screen, operating-system process isolation, Git worktrees, ACID transaction theory, append-only event logs, capability-based security, distributed lease systems, and workflow orchestration systems.

Modern manifestations include AI coding agents such as Claude Code and Codex, tmux-backed terminal workflows, local agent pools, checkpointed mutation systems, Git commit DAGs, and multi-agent verification workflows in which one agent fixes, another reviews, and a third verifies.
