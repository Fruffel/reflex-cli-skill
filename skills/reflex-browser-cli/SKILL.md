---
name: reflex-browser-cli
description: Use this skill for browser automation through Reflex Agent using the reflex-browser CLI, including session handling, command flow, selectors, and protocol-safe request patterns.
---

# Reflex Browser CLI Skill (Agent-Only)

## Purpose

Use this skill when you need browser automation through Reflex Agent via the JSONL shell protocol.

The CLI is agent-only:

- start one long-running shell process
- send one JSON object per line on stdin
- read one JSON object per line on stdout

## Quick Start

1. Install globally from Gitea npm:
   - `npm config set @reflexautomation:registry "https://git.bqa-solutions.nl/api/packages/reflex/npm/" --location=user`
   - `npm config set "//git.bqa-solutions.nl/api/packages/reflex/npm/:_authToken" "YOUR_DEPLOY_TOKEN" --location=user`
   - `npm install -g @reflexautomation/browser-cli`
2. Start shell:
   - `reflex-browser`
3. Only for persistent coding-agent workflows, start with explicit persistence flags:
   - `reflex-browser --session ai-run --profile /tmp/reflex-profile`
4. Wait for `shell_ready`.
5. Send commands sequentially.
6. End with `session_kill` for every session created by this flow.

## Hard Rules

1. Bridge is Chrome-only.
2. Do **not** send `options.browser`.
3. Use explicit waits for state transitions (`wait`, `openWait`).
4. Recompute selectors after DOM changes (`summary` + `selector_helper`).

## References

- Command catalog and argument semantics:
  - `references/commands.md`
- Protocol, payload schema, and response shape:
  - `references/protocol.md`
- Selector generation and stabilization workflow:
  - `references/selectors.md`
- Session lifecycle and recovery strategy:
  - `references/session-management.md`
