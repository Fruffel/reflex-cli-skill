# Books to Scrape Summary Flow

Use this example when an agent needs to:

- navigate a category site with `summary`
- use element refs (`@rN`) from summary for direct interaction
- avoid unnecessary `--session`
- extract repeated item links from a listing page
- open detail pages and summarize the first few results

This is an AI-oriented workflow example, not a shell script to hide execution. Run one command at a time, inspect each JSON response, then continue.

## What This Example Teaches

1. Start the scoped auto-session once.
2. Keep normal commands in auto-session mode without repeating `--session`.
3. Use `summary` to discover category links — prefer `ref` over `selector` when present.
4. Use refs directly for `click`, `attribute`, and other actions — no selector re-parsing needed.
5. Re-run `summary` after each navigation to get fresh refs (stale refs return `ELEMENT_STALE`).
6. If the first `summary` only shows consent/chat widgets, clear them and rerun `summary` before deeper discovery.
7. If `summary` does not surface item-level selectors on the first pass, fine-tune `summary` first (`-s`, `-C`, `-d`, count) and use `html` only as a last resort instead of guessing.
8. Open each book page and extract the product description with `text`.
9. End with `reflex browser session-kill`.

## Fiction Then Horror

Open the site and get a summary. The snapshot shows category links with refs like `[ref=@r10]` for Fiction and `[ref=@r33]` for Horror:

```bash
reflex browser start
reflex browser open https://books.toscrape.com/index.html
reflex browser summary 20 -i -c
```

Click Fiction using its ref directly — no selector needed:

```bash
reflex browser click "@r10"
```

After navigation, refs are invalidated. Re-run summary on the Fiction page to get fresh book refs:

```bash
reflex browser summary 20 -i -c
# snapshot shows: - link "Soumission" [ref=@r150], - link "Private Paris" [ref=@r151], …
```

Read titles and hrefs for the first four books using their refs:

```bash
reflex browser attribute "@r150" title
reflex browser attribute "@r150" href
reflex browser attribute "@r151" title
reflex browser attribute "@r151" href
reflex browser attribute "@r152" title
reflex browser attribute "@r152" href
reflex browser attribute "@r153" title
reflex browser attribute "@r153" href
```

Open each detail page and extract the description:

```bash
reflex browser open "https://books.toscrape.com/catalogue/soumission_998/index.html"
reflex browser text "css=#product_description + p"

reflex browser open "https://books.toscrape.com/catalogue/private-paris-private-10_958/index.html"
reflex browser text "css=#product_description + p"

reflex browser open "https://books.toscrape.com/catalogue/we-love-you-charlie-freeman_954/index.html"
reflex browser text "css=#product_description + p"

reflex browser open "https://books.toscrape.com/catalogue/thirst_946/index.html"
reflex browser text "css=#product_description + p"
```

Go back to the homepage and get a fresh summary to find the Horror ref:

```bash
reflex browser open https://books.toscrape.com/index.html
reflex browser summary 20 -i -c
# snapshot shows: - link "Horror" [ref=@r222]
```

Click Horror via ref and re-run summary on the Horror page:

```bash
reflex browser click "@r222"
reflex browser summary 20 -i -c
# snapshot shows: - link "Security" [ref=@r338], - link "Follow You Home" [ref=@r339], …
```

Read the first four Horror titles and hrefs via their refs:

```bash
reflex browser attribute "@r338" title
reflex browser attribute "@r338" href
reflex browser attribute "@r339" title
reflex browser attribute "@r339" href
reflex browser attribute "@r340" title
reflex browser attribute "@r340" href
reflex browser attribute "@r341" title
reflex browser attribute "@r341" href
```

Open each Horror detail page and extract the description:

```bash
reflex browser open "https://books.toscrape.com/catalogue/security_925/index.html"
reflex browser text "css=#product_description + p"

reflex browser open "https://books.toscrape.com/catalogue/follow-you-home_809/index.html"
reflex browser text "css=#product_description + p"

reflex browser open "https://books.toscrape.com/catalogue/the-loney_756/index.html"
reflex browser text "css=#product_description + p"

reflex browser open "https://books.toscrape.com/catalogue/pet-sematary_726/index.html"
reflex browser text "css=#product_description + p"
```

Cleanup:

```bash
reflex browser session-kill
```

## Why This Pattern Is Good

- It does not keep repeating `--session` in a single-flow task.
- It uses refs from `summary` targets and snapshot lines — no CSS/XPath parsing needed for discovered elements.
- It re-runs `summary` after each navigation to get fresh refs, never reusing stale ones.
- It uses field-specific reads (`attribute`, `text`) instead of generic DOM evaluation.
- It keeps each browser action observable and recoverable.

## Avoid These Mistakes

- Do not run `reflex browser --help` in the middle of the task to figure out the workflow; consult `skills/reflex-browser/SKILL.md`.
- Do not switch to explicit `--session` unless you intentionally need multiple concurrent sessions or a user-requested pinned id.
- Do not reuse refs across navigations — they are invalidated after any DOM-mutating action or navigation.
- Do not keep indexing new selectors after failures; rerun `summary` if page state changes or selectors go stale.
- Do not treat consent/banner/chat-only summary output as failure; clear the overlay and rerun `summary`.
- Do not jump to `html` after the first weak `summary`; fine-tune `summary` scope and flags first.
- Do not abandon a validated browser flow just because chaining or scripting is available; those layers extend the same flow.
- Do not prefer `eval` when `attribute`, `text`, `summary`, or `open` already cover the task.
