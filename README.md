# Reflex Browser CLI

`@reflexautomation/browser-cli` is the CLI for running and controlling browser sessions through Reflex Agent.

Use it from your terminal or coding-agent workflows to open pages, interact with sites, and manage browser sessions.

## Install (Global CLI)

Configure your user npm scope once:

```bash
npm config set @reflexautomation:registry "https://git.bqa-solutions.nl/api/packages/reflex/npm/" --location=user
npm config set "//git.bqa-solutions.nl/api/packages/reflex/npm/:_authToken" "YOUR_DEPLOY_TOKEN" --location=user
```

Install globally:

```bash
npm install -g @reflexautomation/browser-cli
```

## Use

Start shell:

```bash
reflex-browser
```

Quick test:

```bash
printf '{"id":"1","action":"status"}\n{"id":"2","action":"exit"}\n' | reflex-browser
```

Optional persistence flag:

```bash
reflex-browser --session my-agent --profile /tmp/reflex-profile --timeout 15000
```

## Configuration

Config precedence:

1. CLI flags (`--session`, `--profile`, `--timeout`, `--config`)
2. Environment variables
3. Project config discovered by `cosmiconfig` (for example `.reflex-browser.json`)
4. Global config:
   - Windows: `%APPDATA%\reflex-browser\config.{json,yaml,yml}`
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

Prerequisites:

- `mise` (uses `.mise.toml`)
- `npm`

Setup and run locally:

```bash
mise install
npm ci
npm run build
node dist/index.js
```

Before committing changes:

```bash
npm run fix
npm run build
```

## Docs for Agent Integrations

- `skills/reflex-browser/SKILL.md`
- `skills/reflex-browser/references/`
