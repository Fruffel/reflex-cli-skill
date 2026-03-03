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
3. Recompute selectors after DOM changes (`summary --intent`).
4. Stop on first failed command (`ok: false`) to avoid cascading selector errors.
5. Pass relative links directly to `open`; CLI resolves them against current session URL.
6. For repeated-item extraction, anchor selectors at the collection parent (for example list/grid item), then index that parent; do not index unrelated descendants.
7. Treat repeated `no such element` or timeout on the same intent as a selector-state mismatch, not a transient flake.

## Wait Strategy (Required)

1. `click`, `fill`, `type`, and `open` include waiting behavior; avoid redundant waits.
2. Use explicit `wait` for real state transitions:
   - after navigation
   - after `back` / `forward` / `refresh`
   - after async UI updates
3. Prefer stable page-level wait targets over fragile positional selectors.
4. If an action fails once, retry once. If it fails again, run `summary --intent` and continue with updated selectors.
5. After 2 consecutive failures for the same intent, stop the loop and run recovery; never keep incrementing positional selectors blindly.
6. For `wait`/`visible`/`enabled`/`selected`, pass per-check timeout as the positional argument (`wait "<selector>" 8000`); reserve global `--timeout` for transport/command envelope timeout.

## Anti-Patterns (Forbidden)

1. Running commands without checking each JSON response.
2. Using `eval` as default extraction when `text`, `summary`, `attribute`, or `property` can answer the task.
3. Running long blind command chains without validating page state.
4. Continuing extraction loops after a failed `open`.
5. Using positional selectors on the wrong structural level (for example `article:nth-of-type(n)` when siblings are actually `li` elements).
6. Repeating the same failing selector pattern across increasing indexes without re-discovery.
7. Jumping to full `html` dumps before trying `summary --intent` for selector recovery.

## Repeated Items Pattern (Required)

1. Identify the repeated container first (for example `css=ol.row > li`).
2. Validate the container count/visibility with `wait` or `visible`.
3. Extract child fields within indexed container selectors (for example `... > li:nth-of-type(2) h3 a`).
4. Prefer semantic field reads:
   - `attribute ... title` for titles when present
   - `text ... .price_color` for visible price text

## Bash URL Handling (Optional)

Use this only when writing shell wrappers around CLI JSON output.
For normal CLI usage, prefer `open <href>` directly (relative URLs are auto-resolved).

```bash
get_value() { jq -r '.response.data.value // empty'; }
get_url() { reflex-browser url --session "$S" | get_value; }
open_target() {
  local href="$1"
  reflex-browser open "$href" --session "$S" >/dev/null
  get_url
}
```

## PowerShell URL Handling (Optional)

Use the same pattern in PowerShell when running on Windows.

```powershell
function Get-Value {
  param([Parameter(ValueFromPipeline = $true)] $obj)
  process { $obj.response.data.value }
}

function Get-Url {
  reflex-browser url --session $env:S | ConvertFrom-Json | Get-Value
}

function Open-Target {
  param([string]$Href)
  reflex-browser open $Href --session $env:S | Out-Null
  Get-Url
}
```

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
