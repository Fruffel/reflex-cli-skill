# Reflex CLI

`@reflexautomation/reflex-cli` unifies browser automation, scripting, generated library commands, and local Reflex agent management in one CLI.

It has four built-in command groups and metadata-driven top-level library commands:

- browser automation commands: `reflex browser open`, `reflex browser click`, `reflex browser summary`, ...
- local agent commands: `reflex agent start`, `reflex agent status`, `reflex agent download`, ...
- Lua scripting commands: `reflex lua run`, `reflex lua exec`, `reflex lua libs`, ...
- Python scripting commands: `reflex python run`, `reflex python exec`, `reflex python libs`, ...
- generated library commands: `reflex rest get ...`, `reflex xlsx open ...`, ...

Browser commands keep the machine-friendly contract:

1. connect to the agent
2. send exactly one backend action
3. print exactly one JSON object to stdout
4. exit (`0` on success, `1` on failure)

## Install

Configure npm scope:

```bash
npm config set @reflexautomation:registry "https://git.bqa-solutions.nl/api/packages/reflex/npm/" --location=user
npm config set "//git.bqa-solutions.nl/api/packages/reflex/npm/:_authToken" "YOUR_DEPLOY_TOKEN" --location=user
```

Install:

```bash
npm install -g @reflexautomation/reflex-cli
```

## Browser Commands

```bash
reflex browser <command> [args] [flags]
```

Global flags:

- `--config <path>`
- `--engine <selenium|playwright|sel|play>`
- `--profile <path>`
- `--cli-timeout <ms>`

Session behavior:

- `--session [id]` is optional on all browser commands
- omitted `--session` uses an auto-session scoped to machine + repo path
- `--session <id>` pins a manual session id
- bare `--session` requests a backend-assigned session id on `start` and `open`

Bootstrap flags (`start`, `open`):

- `--chrome-connect <false|automatic|port>`
- `--auto-connect`
- `--width <px>`
- `--height <px>`
- `--headless <true|false>`
- `--timeout <ms>`
- `--open-wait <domcontentloaded|load|networkidle>`

Chrome attach notes:

- `chromeConnect: "automatic"` is intended for Playwright and attaches to an already-running Chromium browser over CDP
- Selenium attach is supported only with an explicit debugging port, using a browser launched in the classic way with `--remote-debugging-port` and `--user-data-dir`
- `chrome://inspect/#remote-debugging` works best with Playwright; do not rely on it for Selenium attach

Examples:

```bash
reflex browser start
reflex browser start --engine playwright --auto-connect
reflex browser start --engine selenium --chrome-connect 9333
reflex browser start --session my-session
reflex browser open https://example.com
reflex browser fill "css=input[name='email']" "user@example.com"
reflex browser wait "css=.dashboard" 8000
reflex browser summary 25 -i -c
reflex browser summary 25 -i -c --state
reflex browser summary 25 -i -c --state --screenshot
```

Summary add-on flags:

- `reflex browser summary --state` adds optional page context at `response.state`
- `reflex browser summary --screenshot` adds an optional screenshot artifact at `response.artifacts.screenshot`
- both flags are opt-in; default summary output stays unchanged for existing parsers

## Script Commands

```bash
reflex lua run scripts/test.lua
reflex lua exec "print(Rest.get('https://httpbin.org/get').status)"
reflex python run scripts/test.py
reflex python exec "print(Rest.get('https://httpbin.org/get')['status'])"
```

The script endpoints stream NDJSON events such as `start`, `stdout`, `result`, `error`, and `end`.

For browser automation scripts, first use `reflex browser ...` to inspect the page, session behavior, and selectors. Then turn that knowledge into Lua or Python script code.

## Generated Library Commands

Generated commands come from agent metadata built from `@Lua`, `@LuaDoc`, and `@LuaInner`.

```bash
reflex rest get https://httpbin.org/get
reflex xlsx open report.xlsx sheet Sheet1 get-cell 1 1
reflex lua libs
reflex python libs
reflex help rest
reflex rest --help
```

Complex table/object arguments accept inline JSON or `@file` JSON input.

Generated direct commands are best for discovery and one-off execution. If you need branching logic, loops, multi-step flows, or browser automation, prefer writing a Lua or Python script instead.

## Agent Commands

```bash
reflex agent <command> [flags]
```

Available commands:

- `run` - run the agent in the foreground
- `start` - run the agent in the background
- `status` - inspect background agent state
- `logs` - print recent background logs
- `stop` - stop the background agent
- `download` - download or update `agent.jar`
- `runtime install` - install managed Java
- `runtime status` - inspect Java runtime selection

Examples:

```bash
reflex agent download
reflex agent download --token "$BQAGiteaToken"
reflex agent runtime install
reflex agent start --token "$BQAGiteaToken"
reflex agent start
reflex agent status
reflex agent logs --lines 100
reflex agent stop
```

`reflex agent run` and `reflex agent start` download `agent.jar` on demand when needed.

When run in an interactive terminal, `reflex agent download` and `reflex agent runtime install` show simple built-in progress bars and phase updates. In non-interactive shells they fall back to plain text.

## Configuration

The canonical config namespace is now `reflex`.

Common locations:

- global: `~/.config/reflex/config.json` on Linux/macOS, `%APPDATA%\reflex\config.json` on Windows
- per-repo: `.reflex/config.json`
- one-off override: `reflex --config ./path/to/config.json ...`

Example `config.json`:

```json
{
  "agentUrl": "http://localhost:7001",
  "agentKey": "dev",
  "engine": "playwright",
  "chromeConnect": false,
  "headless": true,
  "width": 1440,
  "height": 900,
  "timeout": 15000,
  "cliTimeout": 180000,
  "openWait": "domcontentloaded",
  "port": 7001,
  "key": "dev",
  "javaSource": "auto"
}
```

Environment variables for browser commands:

- `REFLEX_URL`
- `REFLEX_KEY`
- `REFLEX_ENGINE`
- `REFLEX_CHROME_CONNECT`
- `REFLEX_AUTO_CONNECT`
- `REFLEX_PROFILE`
- `REFLEX_HEADLESS`
- `REFLEX_WIDTH`
- `REFLEX_HEIGHT`
- `REFLEX_TIMEOUT`
- `REFLEX_CLI_TIMEOUT`
- `REFLEX_OPEN_WAIT`

Recommended config examples:

```json
{
  "engine": "playwright",
  "chromeConnect": "automatic"
}
```

```json
{
  "engine": "selenium",
  "chromeConnect": 9333
}
```

For Selenium attach, launch Chrome with a dedicated debugging port and custom profile dir, for example:

```bash
google-chrome-stable \
  --remote-debugging-port=9333 \
  --user-data-dir=/tmp/reflex-selenium-attach
```

Agent auth/download state:

- global auth token: `~/.config/reflex/auth.json`
- cached jar: `~/.cache/reflex/agent.jar`
- managed runtime: `~/.cache/reflex/runtime/`
- token env override: `BQAGiteaToken`
- one-off CLI token: `reflex agent download --token <value>`
- bootstrap token also works on: `reflex agent run --token <value>`, `reflex agent start --token <value>`

## JSON Output Envelope

Browser commands always print one JSON object with:

- `ok`
- `action`
- `session` (if known)
- `timingMs`
- `response`
- `message` on failure

Use `response.data.value` for read-style commands and `response.data.summary.targets[]` for `summary` parsing.
For `reflex browser summary`, parsers may also see optional add-ons:

- `response.state` with lightweight page context such as `tabs`, `scroll`, `viewport`, and `page`
- `response.artifacts.screenshot` with `{ mimeType, base64 }` when `--screenshot` is requested

Existing consumers can ignore those optional fields safely.

Script and generated library commands print NDJSON events instead of a single JSON envelope.

## Development

```bash
mise install
npm ci
npm run build
npm test
```

## Agent Integration Docs

- `skills/reflex-browser/SKILL.md`
- `skills/reflex-browser/references/`
