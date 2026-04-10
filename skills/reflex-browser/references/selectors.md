# Selector Strategy

## Goal

Generate selectors that are stable enough for automation and clear enough for recovery when pages change.

## Refs: Preferred Selector When Available

`summary` assigns short-lived element refs (`@r1`, `@r2`, …) to each interactive element. When a ref is present, use it as the selector — it resolves directly to the live DOM element and is more reliable than CSS or XPath.

```
# snapshot line:  - link "Fiction" [ref=@r10]
# target field:   { "selector": "css=a[href='...']", "ref": "@r10" }

reflex-browser click "@r10"              # preferred
reflex-browser click "css=a[href='...']" # fallback if ref absent
```

Ref lifecycle — refs are **invalidated** after any navigation or DOM-mutating action:

- `click` / `fill` / `type` / `enter` / `tab` that causes a DOM change or navigation
- `open` / `back` / `forward` / `refresh` / tab switches

Always re-run `summary` after such actions before using refs again. A stale ref returns `errorCode: ELEMENT_STALE` — re-run `summary` on that error.

## Workflow

1. Inspect page state:
   - run `summary` first.
   - do not jump to `eval` or `html` unless summary-based recovery is too weak.
2. Guard page identity before selectors:
   - run `url` and verify expected page context.
3. Ask summary for ranked hints:
   - run `summary -i -c` for normal interactive discovery.
   - if the first pass is too broad, add `-s "<selector>"` before changing tools.
   - add `-C` when the page uses non-semantic clickable UI.
   - add `-s "<selector>"` to narrow discovery to one container.
   - add `-d <n>` when large pages produce too much noise.
4. Validate without mutation:
   - run `visible` or `wait` with candidate selector.
5. Perform action:
   - use `click`, `fill`, `type`, etc.
6. Re-evaluate after DOM/navigation change:
   - rerun `summary` with the narrowest useful flags.

## Blocking Overlays

If `summary` returns only cookie/consent/chat widgets or similar overlays:

1. clear the blocking widget first with its `ref` or selector
2. rerun `summary`
3. continue selector discovery only after the overlay is gone
4. do not switch to `eval`, `html`, or external fetching while the overlay is still blocking the page

## Summary Refinement Order

When the first `summary` pass is weak, keep the same tool and change one thing at a time:

1. start with `summary 20 -i -c` — count 20 is enough for most pages
2. narrow scope with `-s`
3. add `-C` for cursor-driven or non-semantic controls
4. tune `-d` for noisy pages
5. only increase count beyond 20 when the page has many repeated items and you need targets for more of them
6. only then consider `html`, and only after 2 materially different `summary` passes still fail

## Preferred Selector Order

1. Stable IDs or test hooks:
   - `id=...`
   - `css=[data-testid='...']`
2. Stable semantic attributes:
   - `name=...`
   - `css=[aria-label='...']`
3. Scoped CSS with parent context:
   - form/dialog/container anchored selectors
4. XPath only when CSS is not expressive enough
5. Deep positional selectors (`nth-child`) as last resort

## Shadow DOM

Shadow DOM selectors are supported with `>>>` chains.

- Format: `css=<shadow-host> >>> css=<inner-target>`
- Nested example: `css=app-shell >>> css=app-page >>> css=button.save`
- Mixed example: `css=book-app >>> xpath=.//h2[.='Boxer en Brandon']`
- Keep the validated chain as-is. Do not flatten it into one CSS selector.
- Prefer `summary` refs when present; use manual `>>>` selectors when you need a durable selector string for later actions or scripts.

## Stabilization Tactics

- Add container context when there are duplicate elements.
- Prefer selectors tied to role/label semantics over class names from build pipelines.
- Avoid dynamic IDs/classes unless they are known stable.
- Use explicit waits for interactive transitions.

## Repeated Collections

When extracting rows/cards/items:

1. Locate the repeated parent element first (list row/card container).
2. Index the repeated parent (`... > li:nth-of-type(n)`), not an internal descendant unless that descendant is truly the sibling set.
3. Scope field selectors under each indexed parent (`title`, `price`, `rating`, etc).
4. Validate one mid-list selector before full loops (for example item 2 or 3) to catch wrong-level indexing early.

Example:

- Fragile: `css=article.product_pod:nth-of-type(2) h3 a`
- Better: `css=ol.row > li:nth-of-type(2) h3 a`

## Recovery Pattern

On selector failure:

1. capture `summary`
2. re-check with `url`
3. run `summary` again with refined flags and narrower scope
   - add `-s` to focus on the likely container
   - add `-C` if the page is using cursor-driven controls
   - tune `-d` or summary count if the page is still noisy
   - clear overlays first if summary is dominated by consent/chat widgets
4. probe candidate with `visible`/`wait`
5. retry action

Escalation depth:

- Start with `summary` for structural context and quick selector regeneration.
- Clear blocking overlays before escalating.
- Keep refining `summary` before switching tools.
- Use `html` only when `summary` still yields weak candidates after 2 materially different refinement passes plus retry/validation.

Circuit breaker:

- Retry once for transient timing.
- If the same intent fails twice, stop iterative extraction and re-discover with `summary` before continuing.
