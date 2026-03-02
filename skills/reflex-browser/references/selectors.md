# Selector Strategy

## Goal

Generate selectors that are stable enough for automation and clear enough for recovery when pages change.

## Workflow

1. Inspect page state:
   - run `summary` first.
2. Ask selector helper:
   - run `selector_helper` with intent text.
3. Validate without mutation:
   - run `visible` or `wait` with candidate selector.
4. Perform action:
   - use `click`, `fill`, `type`, etc.
5. Re-evaluate after DOM/navigation change:
   - rerun `summary` and `selector_helper`.

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

## Recovery Pattern

On selector failure:

1. capture `summary`
2. run `selector_helper` with refined intent
3. probe candidate with `visible`/`wait`
4. retry action
