# Command Reference

## Session/System

- `status`
- `start`
- `session`
- `session-current` -> backend action `session_current`
- `session-list` -> backend action `session_list`
- `session-kill [targetSession]` -> backend action `session_kill`

## Navigation/Page

- `open <url>`
- `url`
- `title`
- `html` (alias: `source`)
- `back`
- `forward`
- `refresh`
- `eval <javascript>`

## Tabs

- `switch-next-tab` -> `switch_next_tab`
- `switch-prev-tab` -> `switch_prev_tab`
- `close-tab` -> `close_tab`

## Interactions

- `click <selector>`
- `fill <selector> <text>`
- `type <selector> <text>`
- `enter <selector>`
- `tab <selector>`

## Reads/Validation

- `text <selector>` (alias: `gettext`)
- `value <selector>` (alias: `getvalue`)
- `attribute <selector> <attributeName>`
- `property <selector> <propertyName>`
- `tag <selector>`
- `wait <selector> [timeoutMs]`
- `visible <selector> [timeoutMs]`
- `enabled <selector> [timeoutMs]`
- `selected <selector> [timeoutMs]`

Read output note:

- prefer `response.data.value` for extracted values (`text`, `value`, `attribute`, `property`, `tag`, `title`, `url`)

Interaction output note:

- `click`, `fill`, `type`, `enter`, `tab` include:
  - `response.selector`
  - `response.lua`
  - optional `response.data` metadata

## Analysis/Debug

- `summary [maxItems] [--intent <query>] [--scope <interactive|content>]`
- `selectors`
- `lua` (alias: `generate`)
- `screenshot`

Summary output note:

- parse `response.data.summary.version`
- parse `response.data.summary.targets[]` as the selector contract

Lua output note:

- parse `response.data.script` (canonical generated script)
- optional `response.data.generationGuidance` for rewrite/refactor constraints

Screenshot output note:

- parse `response.data.imageBase64` and `response.data.mimeType`

## Flags

Global flags for all commands:

- `--config <path>`
- `--profile <path>`
- `--cli-timeout <ms>`

Session flag rule:

- `--session [id]` optional on all commands
- omitted `--session` uses machine+repo scoped auto-session
- `--session <id>` pins a manual session id
- `--session` without value requests a backend-assigned id (only for `start`, `open`)

Bootstrap/open flags (only on `start`, `open`):

- `--width <px>`
- `--height <px>`
- `--headless <true|false>`
- `--timeout <ms>` (backend Selenium retry timeout for that session)
- `--open-wait <domcontentloaded|load|networkidle>`

`open` URL handling:

- absolute URL input is used directly
- relative URL input is automatically resolved against current session URL

Error output note:

- for failed commands (`ok: false`), parse:
  - `response.errorCode`
  - `response.recoveryHint`
  - `message` / `response.message` for human detail
