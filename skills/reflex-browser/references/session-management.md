# Session Management

## Core Principles

1. Use one session id per automation flow.
2. By default, let CLI infer auto-session from machine+repo scope when `--session` is omitted.
3. Use `reflex browser start` (idempotent) to ensure a session exists before work.
4. End sessions explicitly with `reflex browser session-kill`.
5. Reuse active sessions before starting a new browser window.
6. Do not keep repeating `--session` on every command unless you intentionally need an explicit override.
7. When engine changes (`selenium` vs `playwright`), use `start` or `open` to recreate the same inferred auto-session id with the new engine.

## Stateless Workflow

1. Reuse an existing session when it already matches the target flow.
2. Run `reflex browser start` when a fresh session is needed.
3. Read returned JSON and capture `session` when explicit reuse is needed.
4. Run subsequent commands without `--session` (or pass explicit `--session <sessionId>` to override).
5. Run `reflex browser session-kill [targetSession]` when complete.

## Session and Profile Options

- `--session [id]`:
  - optional on all commands
  - when the flag is omitted, CLI resolves scoped auto-session
  - `--session <id>` sets deterministic override session id
  - bare `--session` (no value) requests backend-assigned id on `start` or `open`

- `--profile <path>`:
  - enables persistent browser profile state
  - use only when persistent cookies/local storage are intentionally required

- `--engine <selenium|playwright|sel|play>`:
  - selects the backend engine for new or recreated sessions
  - changing the engine does not mint a new inferred auto-session id; bootstrap commands recreate the existing one

## Lifecycle Commands

- `reflex browser start`: start if absent / reuse active
- `reflex browser session-list`: inspect active sessions
- `reflex browser session-kill [targetSession]`: terminate one named session or inferred session when omitted

## Failure Recovery

For transport failures:

1. rerun the same command (each invocation reconnects)
2. run `reflex browser status` (or `reflex browser status --session <sessionId>` for explicit-session flows)
3. run `reflex browser session-list` (or `reflex browser session-list --session <sessionId>` for explicit-session flows)
4. continue with `reflex browser start` (or `reflex browser start --session <sessionId>` for explicit-session flows)

For action failures with stale DOM:

1. re-check `url` and `title`
2. verify expected page context before selector actions
3. run `reflex browser summary`
   - add `-s` to focus on the likely container
   - add `-C` when the UI is cursor-driven
4. retry action with updated high-confidence selector hint
5. request `html` only if hints remain weak after retries

For auto-session engine mismatches:

1. run `reflex browser start` or `reflex browser open` with the desired `--engine` to recreate the same inferred auto-session id
2. rerun non-bootstrap commands only after that bootstrap succeeds

For looped detail-page traversal:

1. collect `href` values from listing page
2. use `open <href>` directly (CLI auto-resolves relative URLs against current session URL)
3. re-check `url` after each `open` before reading detail selectors
