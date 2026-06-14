---
name: validate-codex-security-with-mythos
description: ChatGPT Codex is still immature on security-context reasoning. When Codex makes claims about authentication, authorization, sandboxing, network access, credentials, keyrings, or execution boundaries, validate those conclusions with Claude MythOS until Codex gets better. The trigger is especially strong when Codex repeats a context mistake after correction.
version: 0.1.0
---

# Validate Codex Security With MythOS

ChatGPT Codex can execute commands and reason about development
workflows, but its security-context reasoning is still immature. Treat
its conclusions about auth, credentials, sandboxing, network access,
permission boundaries, and host-vs-sandbox behavior as provisional.

The practical rule: **security-sensitive Codex conclusions need Claude
MythOS validation until Codex gets better.**

This is not a generic "double check everything" rule. It applies when
the topic is a security boundary or credential-bearing context, and
especially when Codex has already shown confusion about that boundary.

## When to invoke

- Codex makes a claim about authentication state, token validity,
  authorization, credential stores, keyrings, SSH agents, browser login,
  or device-code login.
- Codex reasons about sandboxed vs host execution, network permissions,
  filesystem permissions, escalation, or approval boundaries.
- Codex runs a command in one context, then verifies it in another.
- Codex repeats a security-context mistake after the user points it out.
- The user asks "why would you expect that to work?" about auth,
  sandboxing, network access, or permissions.
- A result depends on whether a command ran inside the sandbox, outside
  the sandbox, in CI, in a container, as another user, or with a
  different credential source.
- Codex says "logged in", "not logged in", "token invalid", "network
  unavailable", or "permission denied" and the conclusion matters.

## What happened

During a GitHub CLI login session, Codex ran:

```sh
gh auth status
```

in the default sandboxed shell. It reported the stored GitHub token was
invalid.

The user then asked Codex to run:

```sh
gh auth login -h github.com
```

The first login attempt ran in the sandbox. It reached the interactive
prompts, but when Codex selected browser/device authentication, the
command failed because the sandbox did not have network access to
`github.com`.

Codex correctly retried the login in an escalated host-side context,
where network access and the macOS keyring were available. The GitHub
device login completed successfully and reported that `tedgoldstein`
was logged in.

Then Codex made the key mistake: it immediately ran `gh auth status`
again in the default sandboxed shell. Predictably, that check still saw
the sandbox/default credential context and reported the old invalid
token. That was not a valid verification of the host-side login.

Codex then had to rerun `gh auth status` in the same escalated context
as the successful login. That check showed the host-side GitHub CLI auth
was valid.

The user pointed out the deeper issue: by definition, the sandboxed
shell does not have the same network/keyring/security context. Codex
should not have expected a sandboxed `gh auth status` to verify a
host-side auth flow. The repeated mistake exposed an immature security
model, not merely a one-off command error.

## The lesson

When Codex is working near a security boundary, do not let it be the
only authority on what the boundary means.

Security-relevant boundaries include:

- sandbox vs host
- network-denied vs network-enabled execution
- escalated vs default command execution
- keyring vs token file
- browser/device login vs CLI token storage
- container vs host
- CI vs local
- current user vs another user
- repo-local config vs home-directory config

Codex may see these as operational details. Treat them as the core
system under analysis.

## Required practice

For security-sensitive work:

1. Have Codex state the exact execution context for each command.
2. Have Codex state which credential store, network boundary, and
   permission boundary the command can see.
3. If Codex changes context, treat subsequent results as facts about
   the new context, not the old one.
4. Before accepting Codex's conclusion, validate with Claude MythOS.
5. If MythOS disagrees, prefer the stricter security interpretation
   until the boundary is proven concretely.

The validation should ask MythOS to review the security model, not just
the command output. The useful question is:

> Did Codex verify the same security context it modified, or did it
> accidentally switch contexts and draw an invalid conclusion?

## Common failure modes

- **Same command string, different security context.** Codex treats
  `gh auth status` as one operation, when sandboxed `gh auth status`
  and host-side `gh auth status` can observe different credentials.
- **Network confusion.** Codex knows the sandbox has restricted network
  access but still expects network-backed verification to work there.
- **Credential-store confusion.** Codex logs in through a host browser
  or keyring flow, then checks a sandbox token store.
- **Escalation amnesia.** Codex escalates to perform the action, then
  forgets escalation was part of the action's meaningful context.
- **Overconfident summary.** Codex compresses multiple facts into one:
  "auth works" or "auth failed." The better statement is contextual:
  "host-side auth works; sandboxed auth still sees an invalid token."

## When this does NOT apply

- The task is not security-sensitive and does not depend on credentials,
  permissions, sandboxing, or network boundaries.
- Codex is only editing local source files and the verification is a
  normal build/test/lint command in the same context.
- Claude MythOS is unavailable and the user explicitly accepts the risk.
  In that case, Codex should still state the boundary assumptions and
  choose the stricter interpretation.

## Tagline

Until Codex gets better at security boundaries, validate its security
conclusions with Claude MythOS.

## Sources

- 2026-05-24 session: Codex repeatedly used sandboxed `gh auth status`
  to reason about a GitHub CLI login that had succeeded in an escalated
  host-side context. The user identified the broader lesson: Codex is
  still immature on security issues, so security-context conclusions
  should be validated with Claude MythOS.
