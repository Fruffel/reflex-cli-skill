# Careers To Xlsx Lua Workflow

Use this example when an agent needs to:

- inspect a careers/jobs page in the live browser
- collect repeated job openings and short summaries
- create Excel output
- turn the validated flow into a reusable Lua script

This is a full workflow example: browser discovery → output planning → reusable script. Each phase builds on the previous one.

## User Request Shape

`Go to <careers-url>, find all job openings, collect a summary of each, put them in Excel, and create a reusable Lua script.`

## Correct Mental Model

1. Browser first: discover live page structure, overlays, repeated job links, and detail selectors.
2. Output next: once the data shape is known, decide the Excel path.
3. Script last: codify the validated browser + output flow into Lua.
4. Reuse everything discovered; do not restart discovery when moving into script form.

## Phase 1: Browser Discovery

Start on the target page and make the live browser flow real before thinking about scripting:

```bash
reflex browser start
reflex browser open "<careers-url>"
reflex browser summary 20 -i -c
```

If the first summary only shows a cookie banner, consent modal, or chat widget, clear it and rerun `summary`:

```bash
reflex browser click "@r7"
reflex browser summary 20 -i -c
```

If the page is noisy, keep refining `summary` instead of changing tools:

```bash
reflex browser summary 20 -i -c -s "<jobs-container>"
reflex browser summary 20 -i -c -s "<jobs-container>" -C
```

Validate the repeated-item pattern with the first 1-2 jobs before trying to automate all of them:

```bash
reflex browser attribute "@r21" href
reflex browser attribute "@r21" title
reflex browser open "<first-job-href>"
reflex browser summary 20 -i -c
reflex browser text "<validated-title-selector>"
reflex browser text "<validated-summary-selector>"
```

At this point the agent should know:

- how to reach the jobs listing
- how to clear overlays if present
- how to identify one job link reliably
- how to read title + summary from a detail page

Only after that should it move to output planning and scripting.

## Phase 2: Output Planning

Check the output library once, then keep going:

```bash
reflex lua help xlsx
```

Key rule:

- if the user explicitly wants a real `.xlsx`, choose a template workbook path early, such as `templates/jobs-template.xlsx`
- if Excel-compatible output is enough, `Csv` is simpler

Do not let output planning derail the validated browser flow.

## Phase 3: Move To Script Form

Because the user asked for reusable automation, now codify the known flow.

Optional: generate a browser trace only after the flow is stable:

```bash
reflex browser lua
```

Treat that output as a starting trace, then refactor it into a reusable script.

## Script Shape

The exact browser selectors vary by site. Reuse the validated selectors and URLs from the browser phase exactly.

```lua
local CAREERS_URL = "<careers-url>"
local TEMPLATE = "templates/jobs-template.xlsx"
local OUTPUT = "out/jobs.xlsx"

local function writeHeaders(sheet)
  sheet.setCell(1, 1, "Title")
  sheet.setCell(2, 1, "URL")
  sheet.setCell(3, 1, "Summary")
end

local function writeJob(sheet, row, job)
  sheet.setCell(1, row, job.title)
  sheet.setCell(2, row, job.url)
  sheet.setCell(3, row, job.summary)
end

local function collectJobs()
  local jobs = {}
  local browser = Sel.start()

  browser.navigate(CAREERS_URL)

  -- Reuse the exact validated selectors from the browser phase.
  -- Example shape:
  -- 1. clear overlay if needed
  -- 2. collect job URLs from the listing page
  -- 3. open each job URL
  -- 4. read title + summary with the validated selectors

  browser.close()
  return jobs
end

local jobs = collectJobs()

local workbook = Xlsx.open(TEMPLATE)
local sheet = workbook.sheet("Jobs")

writeHeaders(sheet)

for index, job in ipairs(jobs) do
  writeJob(sheet, index + 1, job)
end

workbook.save(OUTPUT)
workbook.close()
Xlsx.close()
```

## Why This Pattern Is Good

- It uses the browser to prove the live flow before writing any script.
- It keeps output planning small and explicit.
- It treats Lua as codified automation, not as a separate research phase.
- It carries validated selectors and page transitions upward instead of rediscovering them.

## Avoid These Mistakes

- Do not start with broad capability audits when `open` + `summary` can answer the next question.
- Do not treat browser, chaining, and scripting as competing options.
- Do not jump to `html` after the first weak `summary`; refine `summary` first.
- Do not restart discovery from scratch once the browser flow is already validated.
- Do not let `.xlsx` template constraints derail the automation flow; decide the template path and continue.
