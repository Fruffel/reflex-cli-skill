---
name: reflex-browser
description: Use this skill for browser automation through Reflex Agent using the reflex-browser CLI, including session handling, command flow, selectors, and protocol-safe request patterns.
---

# Reflex Browser Skill (Agent-Only)

## Purpose

Use this skill when you need browser automation through Reflex Agent via stateless CLI commands.

The CLI is single-action per process:

- run one `reflex-browser <command>` invocation
- pass `--session` on all non-`start` actions
- read one JSON response from stdout

## Quick Start

1. Install globally from Gitea npm:
   - `npm config set @reflexautomation:registry "https://git.bqa-solutions.nl/api/packages/reflex/npm/" --location=user`
   - `npm config set "//git.bqa-solutions.nl/api/packages/reflex/npm/:_authToken" "YOUR_DEPLOY_TOKEN" --location=user`
   - `npm install -g @reflexautomation/browser-cli`
2. Start or reuse a session:
   - `reflex-browser start`
3. Capture returned `session` from JSON response.
4. Reuse that session id on each next command:
   - `reflex-browser open https://example.com --session <sessionId>`
5. End with `session-kill <targetSession> --session <sessionId>` for sessions created by this flow.

## Command Lifecycle (Required)

1. Send actions as separate CLI invocations.
2. Keep one logical session id for the task.
3. Do not omit `--session` on non-`start` commands.
4. Execute sequentially: run one command, inspect response, then run the next.
5. On transport failure, rerun the command as a new invocation.
6. Cleanup with `session-kill` when done.

## Session Selection Rules

1. Default: run `start` without `--session` and reuse returned session id.
2. Use `start --session <id>` only when deterministic naming is required.
3. Pass `--profile` only when persistent browser state is intentionally needed.

## Hard Rules

1. Bridge is Chrome-only.
2. Do **not** send `options.browser`.
3. Recompute selectors after DOM changes (`summary` + `selector-helper`).

## Wait Strategy (Required)

1. `click`, `fill`, `type`, and `open` include waiting behavior; avoid redundant waits.
2. Use explicit `wait` for real state transitions:
   - after navigation
   - after `back` / `forward` / `refresh`
   - after async UI updates
3. Prefer stable page-level wait targets over fragile positional selectors.
4. If an action fails once, retry once. If it fails again, run `summary` + `selector-helper` and continue with updated selectors.

## Anti-Patterns (Forbidden)

1. Running commands without checking each JSON response.
2. Using `eval` as default extraction when `text`, `summary`, `attribute`, or `property` can answer the task.
3. Running long blind command chains without validating page state.

## References

- Worked examples:
  - `examples/books-fiction-horror.md`
- Command catalog and flags:
  - `references/commands.md`
- CLI protocol contract:
  - `references/protocol.md`
- Selector workflow:
  - `references/selectors.md`
- Session lifecycle and recovery:
  - `references/session-management.md`
