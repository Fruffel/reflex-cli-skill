# Session Management

## Core Principles

1. Use one session per automation flow.
2. Reuse the same session while context is stable.
3. Use `new`/`restart` when state becomes ambiguous.
4. End sessions explicitly with `session_kill`.

## CLI Flags and Session Overrides

- `--session <name>`:
  - Sets the session for this CLI shell run only.
  - Use this when a coding agent needs deterministic session reuse.
  - This value is intentionally not read from config files or environment variables.

- `--profile <path>`:
  - Sets a persistent browser profile directory (cookies, local storage, other browser state).
  - Use this only when persistent browser state is intentionally required.
  - If omitted, no explicit persistent profile is forced by the CLI.

- `session` (per-command payload field):
  - Use this only when a coding agent intentionally needs to target a specific existing session.
  - If omitted, the agent controls session assignment for the shell flow.

- Recommended default for normal use:
  - Start with `reflex-browser` and do not pass `--profile` unless persistence is required.

## Bootstrap and Reuse

- Shell startup auto-sends `start`.
- `open` also auto-starts session when missing.
- Use `session_list` to inspect existing sessions before reuse.

## Lifecycle Commands

- `start`: start if absent
- `new` / `restart`: force clean session
- `session_kill arg1=<session>`: terminate one session

Global stale sessions are cleaned by the agent background cleanup job.

## Failure Recovery

For transport/protocol failures:

1. reconnect shell
2. run `status`
3. run `session_list`
4. either continue with valid session or run `new`

For action failures with stale/changed DOM:

1. re-check `url`/`title`
2. run `summary`
3. rerun `selector_helper`
4. retry action with updated selector

## Hygiene Checklist

- Do not leave long-lived orphan sessions.
- Kill sessions when flow is complete.
- Prefer deterministic session names for multi-step workflows.
- Use `profile` only when persistence is intentionally required.
