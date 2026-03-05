# Session Management

## Core Principles

1. Use one session id per automation flow.
2. By default, let CLI infer auto-session from machine+repo scope when `--session` is omitted.
3. Use `start` (idempotent) to ensure a session exists before work.
4. End sessions explicitly with `session-kill`.
5. Reuse active sessions before starting a new browser window.

## Stateless Workflow

1. Reuse an existing session when it already matches the target flow.
2. Run `reflex-browser start` when a fresh session is needed.
3. Read returned JSON and capture `session` when explicit reuse is needed.
4. Run subsequent commands without `--session` (or pass explicit `--session <sessionId>` to override).
5. Run `session-kill [targetSession]` when complete.

## Session and Profile Options

- `--session [id]`:
  - optional on all commands
  - when the flag is omitted, CLI resolves scoped auto-session
  - `--session <id>` sets deterministic override session id
  - bare `--session` (no value) requests backend-assigned id on `start` or `open`

- `--profile <path>`:
  - enables persistent browser profile state
  - use only when persistent cookies/local storage are intentionally required

## Lifecycle Commands

- `start`: start if absent / reuse active
- `session-list`: inspect active sessions
- `session-kill [targetSession]`: terminate one named session or inferred session when omitted

## Failure Recovery

For transport failures:

1. rerun the same command (each invocation reconnects)
2. run `status` (or `status --session <sessionId>` for explicit-session flows)
3. run `session-list` (or `session-list --session <sessionId>` for explicit-session flows)
4. continue with `start` (or `start --session <sessionId>` for explicit-session flows)

For action failures with stale DOM:

1. re-check `url` and `title`
2. verify expected page context before selector actions
3. run `summary --intent "<intent>"`
   - add `--scope content` when intent is to locate requirement/description text blocks
4. retry action with updated high-confidence selector hint
5. request `html` only if hints remain weak after retries

For looped detail-page traversal:

1. collect `href` values from listing page
2. use `open <href>` directly (CLI auto-resolves relative URLs against current session URL)
3. re-check `url` after each `open` before reading detail selectors
