# Command Reference

The CLI is now the source of truth for browser command syntax.

Use:

- `reflex browser --help`
- `reflex browser help --json`

Use this file only for workflow reminders:

- default to omitted `--session`; scoped auto-session is the normal agent path
- default to `summary` with `-i`, `-C`, `-c`, `-d`, and `-s` for selector discovery and recovery
- prefer visible refs, button labels, link text, and `href` values from `summary.targets[]`
- use browser CLI directly for one-off website tasks; do not create a script unless the user asks for one or the task clearly needs reusable automation
- if browser commands alone become too complex, prefer `reflex lua exec ...` before writing a saved script file

For the exact machine-readable contract, always inspect `reflex browser help --json`.
