# Reflex Browser CLI

`@reflexautomation/browser-cli` runs Reflex Agent browser actions as stateless CLI commands.

Each invocation:

1. connects to the agent
2. sends exactly one backend action
3. prints exactly one JSON object to stdout
4. exits (`0` on success, `1` on failure)

## Install (Global CLI)

Configure npm scope:

```bash
npm config set @reflexautomation:registry "https://git.bqa-solutions.nl/api/packages/reflex/npm/" --location=user
npm config set "//git.bqa-solutions.nl/api/packages/reflex/npm/:_authToken" "YOUR_DEPLOY_TOKEN" --location=user
```

Install:

```bash
npm install -g @reflexautomation/browser-cli
```

## Command Model

```bash
reflex-browser <command> [args] [flags]
```

Global flags (available on every command):

- `--config <path>`
- `--profile <path>`
- `--cli-timeout <ms>`
  - default CLI command-response timeout is effectively `180000` ms unless explicitly overridden

Session flag:

- `--session [id]` is optional on all commands
- when omitted, CLI resolves an auto-session scoped to machine + repo path
- `--session <id>` pins a manual session id
- `--session` without a value requests a backend-assigned session id (only for `start`, `open`)

Bootstrap/open flags (supported only by `start`, `open`):

- `--width <px>`
- `--height <px>`
- `--headless <true|false>`
- `--timeout <ms>` (backend Selenium retry timeout for the session)
- `--open-wait <domcontentloaded|load|networkidle>`

## Examples

Get or create the default auto-session for current machine + repo:

```bash
reflex-browser start
```

Start with explicit session id:

```bash
reflex-browser start --session my-session
```

Start with backend-assigned session id:

```bash
reflex-browser start --session
```

Open a URL without passing `--session` (auto-session is inferred):

```bash
reflex-browser open https://example.com
```

Open a URL in an explicit session:

```bash
reflex-browser open https://example.com --session my-session
```

Fail fast if page context is wrong:

```bash
reflex-browser url
# stop if URL is not what you expect before running selector commands
```

Open a relative URL safely (resolved against current session URL):

```bash
reflex-browser open "../../../a-book_1/index.html"
```

Fill an input:

```bash
reflex-browser fill "css=input[name='email']" "user@example.com"
```

Wait with custom timeout:

```bash
reflex-browser wait "css=.dashboard" 8000
```

Get a compact summary snapshot of interactive elements:

```bash
reflex-browser summary 25 -i -c
```

Include cursor-interactive elements used by modern component UIs:

```bash
reflex-browser summary 25 -i -C
```

Limit depth and scope snapshot to a specific container:

```bash
reflex-browser summary 40 -i -c -d 5 -s "#main"
```

Prefer summary snapshots first and fetch `html` only as a last resort when selector hints are weak or fail validation.

Summary filters:

- `-i, --interactive`: only interactive elements (links/buttons/inputs)
- `-C, --cursor`: include cursor-interactive elements (`onclick`, `cursor:pointer`, `tabindex`)
- `-c, --compact`: remove empty structural elements
- `-d, --depth <n>`: limit tree depth
- `-s, --selector <sel>`: scope to CSS selector

Use `html`, `text`, or field-specific reads for deep content extraction.

Summary parser contract:

- `response.data.summary.version`
- `response.data.summary.targets[]`
  - `selector`
  - `selectorType`
  - `confidence`
  - `score`
  - `reason`
  - Optional steering fields:
    - `status` (`ready`, `retry`, `avoid`)
    - `hint`
    - `fallback`

## JSON Output Envelope

All commands print one JSON object:

- `ok`
- `action`
- `session` (if known)
- `timingMs`
- `response` (action-specific response payload)
- `message` (error detail on failure)

Use envelope fields (`ok`, `action`, `session`) as source-of-truth. The CLI compacts mirrored duplicates from `response` where possible.

For read-style actions (`text`, `attribute`, `property`, `value`, `tag`, `title`, `url`), use `response.data.value` as the extracted value.

For `summary`, parse `response.data.summary.targets[]` (not legacy candidate-style fields).

## Auto-Session Scope

- Auto-session key is deterministic per `machine fingerprint + repo root`.
- Repo root uses `git rev-parse --show-toplevel`; fallback is current working directory.
- Mapping is stored in global `config.json` (`.../reflex-browser/config.json`) under `autoSessions`.
- Stale/missing mapped sessions are recreated only by `start` and `open`.
- `reflex-browser start` without `--session` returns the existing healthy auto-session or recreates it.
- `session-kill [targetSession]`:
  - with argument: kills that target session
  - without argument + `--session`: kills that explicit session
  - without argument and without `--session`: kills inferred auto-session for current scope
  - successful kill clears scope mapping when it points to the killed session

## Configuration

Config precedence:

1. CLI flags (`--config`, `--profile`, `--cli-timeout`, command-specific options)
2. Environment variables
3. Current working directory config (`.reflex-browser/config.{json,yaml,yml}`)
4. Project config discovered by `cosmiconfig` (for example `.reflex-browser.json`)
5. Global config:
   - Windows: `%APPDATA%\\reflex-browser\\config.{json,yaml,yml}`
   - Linux/macOS: `$XDG_CONFIG_HOME/reflex-browser/config.{json,yaml,yml}` or `~/.config/reflex-browser/config.{json,yaml,yml}`

Config keys:

- `timeout`: backend Selenium retry timeout in milliseconds (session-level, default `10000`)
- `cliTimeout`: CLI transport timeout in milliseconds (effective minimum `180000` unless `--cli-timeout` is explicitly passed)

Environment variables:

- `REFLEX_AGENT_URL`
- `REFLEX_AGENT_KEY`
- `REFLEX_BROWSER_PROFILE`
- `REFLEX_BROWSER_HEADLESS`
- `REFLEX_BROWSER_WIDTH`
- `REFLEX_BROWSER_HEIGHT`
- `REFLEX_BROWSER_TIMEOUT`
  - backend Selenium retry timeout in milliseconds (default `10000`)
- `REFLEX_BROWSER_CLI_TIMEOUT`
  - defaults to at least `180000` ms for command-response waiting unless `--cli-timeout` is explicitly passed
- `REFLEX_BROWSER_OPEN_WAIT` (`domcontentloaded`, `load`, `networkidle`)

## Development

```bash
mise install
npm ci
npm run validate
npm run build
npm test
```

## Agent Integration Docs

- `skills/reflex-browser/SKILL.md`
- `skills/reflex-browser/references/`
