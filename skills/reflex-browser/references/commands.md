# Command Reference

## Session/System Commands

- `status`
  - Returns bridge health and session count.
- `start`
  - Starts a session if missing; reuses existing active session.
- `new` / `restart`
  - Force new session, closing previous session state.
- `session`
  - Reports the current session name.
- `session_current`
  - Alias of `session`.
- `session_list`
  - Lists active sessions and metadata.
- `session_kill`
  - `arg1=<session>`
  - Kills one named session.

## Navigation and Page Commands

- `open`
  - `arg1=<url>`
  - Starts session if needed, navigates, applies open wait strategy.
- `url`
  - Returns current page URL.
- `title`
  - Returns current page title.
- `html` / `source`
  - Returns current page source.
- `back`, `forward`, `refresh`
  - Browser navigation controls.
- `eval`
  - `arg1=<javascript>`
  - Executes JavaScript in current page context.

## Tab Commands

- `switch_next_tab`
- `switch_prev_tab`
- `close_tab`

## Element Interaction Commands

- `click`
  - `arg1=<selector>`
- `fill`
  - `arg1=<selector>`, `arg2=<text>`
  - Clears then types.
- `type`
  - `arg1=<selector>`, `arg2=<text>`
  - Types without clear.
- `enter`
  - `arg1=<selector>`
- `tab`
  - `arg1=<selector>`

## Element Read/Validation Commands

- `text` / `gettext`
  - `arg1=<selector>`
- `value` / `getvalue`
  - `arg1=<selector>`
- `attribute`
  - `arg1=<selector>`, `arg2=<attributeName>`
- `property`
  - `arg1=<selector>`, `arg2=<propertyName>`
- `tag`
  - `arg1=<selector>`
- `wait`
  - `arg1=<selector>`, optional `arg2=<timeoutMs>`
- `visible`
  - `arg1=<selector>`, optional `arg2=<timeoutMs>`
- `enabled`
  - `arg1=<selector>`, optional `arg2=<timeoutMs>`
- `selected`
  - `arg1=<selector>`, optional `arg2=<timeoutMs>`

## Analysis/Debug Commands

- `summary`
  - optional `arg1=<maxItems>`, default `20`
- `selector_helper` / `selector-helper` / `discover_selectors`
  - `arg1=<intent query>`, optional `arg2=<maxItems>`
- `selectors`
  - Returns discovered selector history.
- `lua` / `generate`
  - Emits generated Lua script from recorded actions.
- `screenshot`
  - Returns in-memory PNG (`response.data.imageBase64`).

## Notes

1. Commands are session-scoped; keep one session per flow.
2. Run dependent commands sequentially.
3. `open`, `start`, `new`, `restart` consume `options` (`width`, `height`, `headless`, `openWait`).
4. `options.browser` is not supported.
5. `stop` is removed; use `session_kill` for explicit cleanup.
