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
- `--timeout <ms>`

Session flag:

- `start`: `--session <id>` optional
- all other commands: `--session <id>` required

Bootstrap/open flags (supported only by `start`, `new`, `restart`, `open`):

- `--width <px>`
- `--height <px>`
- `--headless <true|false>`
- `--open-wait <domcontentloaded|load|networkidle>`

## Examples

Start a session (agent assigns session id):

```bash
reflex-browser start
```

Start with explicit session id:

```bash
reflex-browser start --session my-session
```

Open a URL in an existing session:

```bash
reflex-browser open https://example.com --session my-session
```

Fail fast if page context is wrong:

```bash
reflex-browser url --session my-session
# stop if URL is not what you expect before running selector commands
```

Open a relative URL safely (resolved against current session URL):

```bash
reflex-browser open "../../../a-book_1/index.html" --session my-session
```

Fill an input:

```bash
reflex-browser fill "css=input[name='email']" "user@example.com" --session my-session
```

Wait with custom timeout:

```bash
reflex-browser wait "css=.dashboard" 8000 --session my-session
```

## JSON Output Envelope

All commands print one JSON object:

- `ok`
- `action`
- `session` (if known)
- `timingMs`
- `response` (raw backend response)
- `message` (error detail on failure)

## Configuration

Config precedence:

1. CLI flags (`--config`, `--profile`, `--timeout`, command-specific options)
2. Environment variables
3. Current working directory config (`.reflex-browser/config.{json,yaml,yml}`)
4. Project config discovered by `cosmiconfig` (for example `.reflex-browser.json`)
5. Global config:
   - Windows: `%APPDATA%\\reflex-browser\\config.{json,yaml,yml}`
   - Linux/macOS: `$XDG_CONFIG_HOME/reflex-browser/config.{json,yaml,yml}` or `~/.config/reflex-browser/config.{json,yaml,yml}`

Environment variables:

- `REFLEX_AGENT_URL`
- `REFLEX_AGENT_KEY`
- `REFLEX_BROWSER_PROFILE`
- `REFLEX_BROWSER_HEADLESS`
- `REFLEX_BROWSER_WIDTH`
- `REFLEX_BROWSER_HEIGHT`
- `REFLEX_BROWSER_TIMEOUT`
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
