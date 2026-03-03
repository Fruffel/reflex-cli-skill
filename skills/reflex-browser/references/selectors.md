# Selector Strategy

## Goal

Generate selectors that are stable enough for automation and clear enough for recovery when pages change.

## Workflow

1. Inspect page state:
   - run `summary` first.
2. Guard page identity before selectors:
   - run `url` and verify expected page context.
3. Ask summary for ranked hints:
   - run `summary --intent "<intent>"`.
4. Validate without mutation:
   - run `visible` or `wait` with candidate selector.
5. Perform action:
   - use `click`, `fill`, `type`, etc.
6. Re-evaluate after DOM/navigation change:
   - rerun `summary --intent`.

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
3. run `summary --intent` with refined intent
4. probe candidate with `visible`/`wait`
5. retry action

Escalation depth:

- Start with `summary` for structural context and quick selector regeneration.
- Use `html` only when `summary --intent` yields weak candidates after retry/validation.

Circuit breaker:

- Retry once for transient timing.
- If the same intent fails twice, stop iterative extraction and re-discover with `summary --intent` before continuing.
