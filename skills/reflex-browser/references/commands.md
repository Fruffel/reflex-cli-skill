# Command Reference

## Session/System

- `status`
- `start`
- `new`
- `restart`
- `session`
- `session-current` -> backend action `session_current`
- `session-list` -> backend action `session_list`
- `session-kill <targetSession>` -> backend action `session_kill`

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

## Analysis/Debug

- `summary [maxItems] [--intent <query>]`
- `selectors`
- `lua` (alias: `generate`)
- `screenshot`

## Flags

Global flags for all commands:

- `--config <path>`
- `--profile <path>`
- `--timeout <ms>`

Session flag rule:

- `start`: `--session <id>` optional
- all other commands: `--session <id>` required

Bootstrap/open flags (only on `start`, `new`, `restart`, `open`):

- `--width <px>`
- `--height <px>`
- `--headless <true|false>`
- `--open-wait <domcontentloaded|load|networkidle>`

`open` URL handling:

- absolute URL input is used directly
- relative URL input is automatically resolved against current session URL
