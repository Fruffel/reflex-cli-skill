# Protocol Reference (JSONL)

## Transport Model

- Input: one JSON object per line on stdin.
- Output: one JSON object per line on stdout.
- Startup emits a readiness line (`shell_ready`) after auto-bootstrap `start`.

## Input Payload

Top-level fields:

- `id`: optional request id
- `action`: required command name
- `arg1`, `arg2`: optional command args
- `session`: optional session override
- `profile`: optional profile override
- `timeoutMs`: optional timeout override
- `options`: optional bootstrap/open options
  - `width`
  - `height`
  - `headless`
  - `openWait` (`domcontentloaded|load|networkidle`)

Important:

- Do not send `options.browser`.
- Bridge sessions are Chrome-only.

## Output Envelope

Shell lines:

- readiness:
  - `{"ok":true,"action":"shell_ready",...}`
- per command:
  - `{"ok":true|false,"id":"...","action":"...","timingMs":...,"response":{...}}`

`response` typically contains:

- `success`: backend result status
- `message`: result/error summary
- optional `data`: page/context metadata
- optional `errorCode`, `recoveryHint` on failures

## Example Flow

```json
{"id":"1","action":"open","arg1":"https://example.com"}
{"id":"2","action":"summary","arg1":"20"}
{"id":"3","action":"selector_helper","arg1":"login button","arg2":"10"}
{"id":"4","action":"click","arg1":"css=button[type='submit']"}
{"id":"5","action":"wait","arg1":"css=.dashboard","arg2":"10000"}
```

## Screenshot Response

For `action: "screenshot"`:

- `response.data.imageBase64`: base64 PNG
- `response.data.mimeType`: `image/png`
- `response.data.byteSize`: byte length
