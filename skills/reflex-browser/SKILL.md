---
name: reflex-browser
description: Use this skill for browser automation through Reflex Agent using the reflex-browser CLI, including session handling, command flow, selectors, and protocol-safe request patterns.
---

# Reflex Browser Skill (Agent-Only)

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
3. Default behavior:
   - Reuse the session returned by `shell_ready` / command responses.
   - Do not set `--session` by default.
4. Edge-case overrides only:
   - `reflex-browser --session ai-run --profile /tmp/reflex-profile`
5. Wait for `shell_ready`.
6. Send commands sequentially.
7. End with `session_kill` for every session created by this flow.

## Shell Lifecycle (Required)

1. Start one long-running `reflex-browser` shell per task and keep it open.
2. Send all normal step-by-step commands through that same shell connection.
3. Do not reopen/restart the shell between normal steps.
4. Reconnect only when transport/session errors occur.
5. When done, run `session_kill` for created sessions, then `exit`.

## Session Selection Rules

1. Default: let Reflex Agent assign the session and reuse the returned session id.
2. Do not pass `--session` unless you intentionally need a deterministic session name.
3. Do not pass `session` in every command unless intentionally switching/targeting sessions.
4. Treat `--session` the same as `--profile`: edge-case override, not default flow.

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
