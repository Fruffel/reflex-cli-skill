# Command Reference

The CLI is now the source of truth for browser command syntax.

Use:

- `reflex browser --help`
- `reflex browser help --json`

Use this file only for workflow reminders:

- default to omitted `--session`; scoped auto-session is the normal agent path
- default to `summary` and fine-tune it (`-i -c`, then `-s`, `-C`, `-d`) for selector discovery and recovery; do not jump to `html` on the first weak pass. Start with count 20; increase only when the page has many repeated items
- if `summary` only returns cookie/chat/consent widgets, clear them via refs/selectors and rerun `summary`
- do not chain multiple browser commands with `&&`; inspect each JSON response before continuing
- start with target page discovery before broad `lua libs` or repeated help audits
- prefer visible refs, button labels, link text, and `href` values from `summary.targets[]`
- use browser CLI directly for one-off website tasks; do not create a script unless the user asks for one or the task clearly needs reusable automation
- for scripting, library commands, and export work, see the `reflex-scripting` skill

For the exact machine-readable contract, always inspect `reflex browser help --json`.
