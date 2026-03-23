# Protocol Reference (Stateless CLI)

The structured CLI/browser contract now lives in:

- `reflex browser help --json`

Use this document for transport/model reminders, not as the primary command listing.

## Transport Model

- Invocation: `reflex-browser <command> [args] [flags]`
- Most commands send exactly one backend action.
- Output: exactly one JSON object on stdout.
- Exit code: `0` on success, `1` on failure.
- Prefer explicit sequential invocations over bundled shell scripts so each step is inspectable.

## Command Contract

Global flags:

- `--config <path>`
- `--engine <selenium|playwright|sel|play>`
- `--profile <path>`
- `--cli-timeout <ms>`

Session contract:

- `--session [id]` is optional on all commands
- if omitted, CLI resolves a deterministic auto-session per `machine + repo`
- `--session <id>` explicitly overrides auto-session inference
- bare `--session` (no value) requests backend-assigned id on `start`, `open`

Bootstrap/open options (`start`, `open` only):

- `--width <px>`
- `--height <px>`
- `--headless <true|false>`
- `--timeout <ms>`
- `--open-wait <domcontentloaded|load|networkidle>`

## Backend Payload Mapping

CLI builds one backend payload with:

- `action`
- `arg1`
- `arg2`
- `arg3`
- `session`
- `profile`
- `options` (only for bootstrap/open commands)
  - `options.engine` is the canonical engine field for Selenium vs Playwright

`wait`, `visible`, `enabled`, and `selected` optional timeout positional arg maps to an action-level check timeout override (not `arg2` and not `--cli-timeout`).

Smart preflight behavior:

- if `open` receives a relative URL, CLI requests current URL and resolves `arg1` before sending `open`
- missing/stale auto-sessions are recreated only by bootstrap navigation commands (`start`, `open`)
- engine changes reuse the same inferred auto-session id; `start`/`open` recreate it when the active engine differs
- non-bootstrap commands fail fast on auto-session engine mismatch until the session is recreated

Connection lifecycle:

- each CLI invocation opens one WebSocket request lifecycle and closes it after completion/failure

## Output Envelope

All command responses follow:

- `ok`
- `action`
- `session` (effective session when known)
- `timingMs`
- `response` (backend payload, compacted by CLI)
- `message` (error details)

Compaction rules:

- `response.success` is omitted (use envelope `ok`)
- duplicated `response.session` matching envelope `session` is omitted
- duplicated `response.data` context fields that mirror envelope/response state are omitted (`action`, `session`, repeated page-state keys)
- optional `summary` add-ons live beside `response.data`:
  - `response.state` for lightweight page context (`tabs`, `scroll`, `viewport`, `page`)
  - `response.artifacts.screenshot` for `{ mimeType, base64 }` when `--screenshot` is requested
- existing parsers can ignore these optional add-ons safely

Response-consumption rule:

- capture full JSON first, then parse/select fields from that captured response object

Shell helper option:

- source `skills/reflex-browser/scripts/capture_json.sh` (from project root) to enforce capture-first parsing in Bash workflows

Use `url` explicitly to verify page context before selector-heavy sequences.

Read command value contract:

- for `text`, `value`, `attribute`, `property`, `tag`, `title`, and `url`, the CLI normalizes extracted output to `response.data.value`

## Screenshot Response

For `screenshot`:

- `response.data.imageBase64`: base64 PNG
- `response.data.mimeType`: `image/png`
- `response.data.byteSize`: byte length
