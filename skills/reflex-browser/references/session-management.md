# Session Management

## Core Principles

1. Use one session id per automation flow.
2. Start once, then pass the same `--session` for each follow-up command.
3. Use `new` or `restart` when state is ambiguous.
4. End sessions explicitly with `session-kill`.

## Stateless Workflow

1. Run `reflex-browser start`.
2. Read returned JSON and capture `session`.
3. Run subsequent commands with `--session <sessionId>`.
4. Run `session-kill <targetSession> --session <sessionId>` when complete.

## Session and Profile Options

- `--session <id>`:
  - optional on `start`
  - required on all other commands
  - use explicit values only when deterministic naming is needed

- `--profile <path>`:
  - enables persistent browser profile state
  - use only when persistent cookies/local storage are intentionally required

## Lifecycle Commands

- `start`: start if absent / reuse active
- `new`: force a fresh session context
- `restart`: restart session context
- `session-list`: inspect active sessions
- `session-kill <targetSession>`: terminate one named session

## Failure Recovery

For transport failures:

1. rerun the same command (each invocation reconnects)
2. run `status --session <sessionId>`
3. run `session-list --session <sessionId>`
4. continue or reset with `new --session <sessionId>`

For action failures with stale DOM:

1. re-check `url` and `title`
2. verify expected page context before selector actions
3. run `summary --intent "<intent>"`
4. retry action with updated high-confidence selector hint
5. request `html` only if hints remain weak after retries

For looped detail-page traversal:

1. collect `href` values from listing page
2. use `open <href>` directly (CLI auto-resolves relative URLs against current session URL)
3. re-check `url` after each `open` before reading detail selectors
