# Page To Export One-Off Workflow

Use this example when an agent needs to:

- inspect a page in the live browser
- collect repeated items or records
- export the results once
- finish without creating a saved script file

This is a one-off workflow: browser discovery followed by direct export. No saved script needed.

## User Request Shape

`Open <target-url>, collect the relevant rows/cards/items, and export the result once.`

## Correct Mental Model

1. Browser first: prove the live page flow, clear overlays, and validate selectors.
2. Chaining next: once the data shape is known, use libraries/CLI calls to write the output.
3. Stop there if the task is one-off. Do not create a saved script unless reuse is actually needed.

## Phase 1: Browser Discovery

Start with the live page:

```bash
reflex browser start
reflex browser open "<target-url>"
reflex browser summary 20 -i -c
```

If the first summary only shows consent/chat/banner widgets, clear them and rerun:

```bash
reflex browser click "@r7"
reflex browser summary 20 -i -c
```

If the page is still noisy, refine `summary` instead of changing tools:

```bash
reflex browser summary 20 -i -c -s "<results-container>"
reflex browser summary 20 -i -c -s "<results-container>" -C
```

Validate the first 1-2 items before collecting all of them:

```bash
reflex browser attribute "@r21" href
reflex browser open "<first-item-href>"
reflex browser summary 20 -i -c
reflex browser text "<validated-title-selector>"
reflex browser text "<validated-summary-selector>"
```

At this point the agent should know:

- how to reach the listing or results page
- how to clear overlays if present
- how to identify one item reliably
- how to read the fields needed for export

## Phase 2: One-Off Export

Now extend the same validated flow with libraries/CLI chaining.

Choose the smallest useful output path:

- `Csv` for simple tabular export
- `Fl.write(...)` for JSON, Markdown, or plain text output
- `Xlsx` only when a real `.xlsx` workbook is required

For a one-off but slightly complex export, `reflex lua exec ...` is usually the right level.

Example shape:

```bash
reflex lua exec "...browser-backed collection plus Csv/Fl/Xlsx output..."
```

The important part is not the exact output format; it is that the export extends the known browser flow rather than replacing it.

## Inline Lua Shape

```lua
local function collectItems()
  local items = {}
  local browser = Sel.start()

  browser.navigate("<target-url>")

  -- Reuse the exact validated selectors from browser discovery.
  -- Example shape:
  -- 1. clear overlay if needed
  -- 2. collect item URLs or fields from the listing page
  -- 3. open detail pages when needed
  -- 4. read the validated fields

  browser.close()
  return items
end

local items = collectItems()

-- Then write once with the chosen output library.
-- Examples:
-- Fl.write("out/items.json", Json.encode(items))
-- Csv/ Xlsx flow when tabular output is needed
```

## Why This Pattern Is Good

- It uses the browser to validate reality before exporting anything.
- It uses library chaining only when it adds concrete value.
- It avoids unnecessary saved scripts for one-off work.
- It keeps the flow unified from discovery to export.

## Avoid These Mistakes

- Do not start with output format work before the browser flow is validated.
- Do not treat chaining as a reason to abandon the known browser flow.
- Do not jump to `html` after the first weak `summary`; refine `summary` first.
- Do not create a saved script file when `browser` plus one-off chaining already solves the task.
