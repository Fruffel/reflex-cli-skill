# Books to Scrape Summary Flow

Use this example when an agent needs to:

- navigate a category site with `summary`
- avoid unnecessary `--session`
- extract repeated item links from a listing page
- open detail pages and summarize the first few results

This is an AI-oriented workflow example, not a shell script to hide execution. Run one command at a time, inspect each JSON response, then continue.

## What This Example Teaches

1. Start the scoped auto-session once.
2. Keep normal commands in auto-session mode without repeating `--session`.
3. Use `summary` to discover category links first.
4. If `summary` does not surface item-level selectors, derive them from `html` evidence instead of guessing.
5. Open each book page and extract the product description with `text`.
6. End with `session-kill`.

## Fiction Then Horror

Open the site:

```bash
reflex-browser start
reflex-browser open https://books.toscrape.com/index.html
reflex-browser summary 20 -i -c
```

Open the Fiction category using the matching category link from `response.data.summary.targets[]`:

```bash
reflex-browser open "catalogue/category/books/fiction_10/index.html"
```

Derive the repeated-item selector from raw HTML evidence when `summary` does not provide item-level book selectors:

```bash
reflex-browser html
```

Relevant structure from the page HTML:

```html
<ol class="row">
  <li class="col-xs-6 col-sm-4 col-md-3 col-lg-3">
    <article class="product_pod">
      <h3>
        <a href="../../../soumission_998/index.html" title="Soumission"
          >Soumission</a
        >
      </h3>
    </article>
  </li>
</ol>
```

That proves:

- repeated collection container: `css=ol.row > li`
- repeated card body: `article.product_pod`
- title/detail link inside each card: `h3 a`
- book title is available in the anchor `title` attribute
- detail URL is available in the anchor `href` attribute

Read the first four book titles and hrefs from the listing page with the html-derived selector path:

```bash
reflex-browser attribute "css=ol.row > li:nth-of-type(1) article.product_pod h3 a" title
reflex-browser attribute "css=ol.row > li:nth-of-type(1) article.product_pod h3 a" href
reflex-browser attribute "css=ol.row > li:nth-of-type(2) article.product_pod h3 a" title
reflex-browser attribute "css=ol.row > li:nth-of-type(2) article.product_pod h3 a" href
reflex-browser attribute "css=ol.row > li:nth-of-type(3) article.product_pod h3 a" title
reflex-browser attribute "css=ol.row > li:nth-of-type(3) article.product_pod h3 a" href
reflex-browser attribute "css=ol.row > li:nth-of-type(4) article.product_pod h3 a" title
reflex-browser attribute "css=ol.row > li:nth-of-type(4) article.product_pod h3 a" href
```

Open each detail page and extract the description:

```bash
reflex-browser open "../../../soumission_998/index.html"
reflex-browser text "css=#product_description + p"

reflex-browser open "../../../private-paris-private-10_958/index.html"
reflex-browser text "css=#product_description + p"

reflex-browser open "../../../we-love-you-charlie-freeman_954/index.html"
reflex-browser text "css=#product_description + p"

reflex-browser open "../../../thirst_946/index.html"
reflex-browser text "css=#product_description + p"
```

Go back to the home page, discover Horror with `summary`, and repeat the same pattern:

```bash
reflex-browser open https://books.toscrape.com/index.html
reflex-browser summary 20 -i -c
reflex-browser open "catalogue/category/books/horror_31/index.html"
```

Read the first four Horror titles and hrefs:

```bash
reflex-browser attribute "css=ol.row > li:nth-of-type(1) article.product_pod h3 a" title
reflex-browser attribute "css=ol.row > li:nth-of-type(1) article.product_pod h3 a" href
reflex-browser attribute "css=ol.row > li:nth-of-type(2) article.product_pod h3 a" title
reflex-browser attribute "css=ol.row > li:nth-of-type(2) article.product_pod h3 a" href
reflex-browser attribute "css=ol.row > li:nth-of-type(3) article.product_pod h3 a" title
reflex-browser attribute "css=ol.row > li:nth-of-type(3) article.product_pod h3 a" href
reflex-browser attribute "css=ol.row > li:nth-of-type(4) article.product_pod h3 a" title
reflex-browser attribute "css=ol.row > li:nth-of-type(4) article.product_pod h3 a" href
```

Open each Horror detail page and extract the description:

```bash
reflex-browser open "../../../security_925/index.html"
reflex-browser text "css=#product_description + p"

reflex-browser open "../../../follow-you-home_809/index.html"
reflex-browser text "css=#product_description + p"

reflex-browser open "../../../the-loney_756/index.html"
reflex-browser text "css=#product_description + p"

reflex-browser open "../../../pet-sematary_726/index.html"
reflex-browser text "css=#product_description + p"
```

Cleanup:

```bash
reflex-browser session-kill
```

## Why This Pattern Is Good

- It does not keep repeating `--session` in a single-flow task.
- It uses `summary` for category discovery and falls back to `html` only when item-level selectors are not surfaced.
- It anchors repeated-item selectors at the list item level.
- It shows where the repeated-item selector actually came from, instead of relying on an unstated guess.
- It uses field-specific reads (`attribute`, `text`) instead of generic DOM evaluation.
- It keeps each browser action observable and recoverable.

## Avoid These Mistakes

- Do not run `reflex-browser --help` in the middle of the task to figure out the workflow; consult `skills/reflex-browser/SKILL.md`.
- Do not switch to explicit `--session` unless you intentionally need multiple concurrent sessions or a user-requested pinned id.
- Do not keep indexing new selectors after failures; rerun `summary` if page state changes or selectors go stale.
- Do not prefer `eval` when `attribute`, `text`, `summary`, or `open` already cover the task.
