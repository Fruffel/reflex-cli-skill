# Example: Fiction then Horror (Stateless Commands)

## User Prompt

`$reflex-browser Go to https://books.toscrape.com/index.html - fiction - give a summary of the first 4 books and then go to horror and give a summary of those first 4 books`

## Execution Rules Applied

1. One logical session id reused across commands.
2. One command per process invocation.
3. Sequential execution: inspect each JSON response before next step.

## Command Sequence

```bash
reflex-browser start
reflex-browser open "https://books.toscrape.com/index.html" --session <sessionId>
reflex-browser click "a[href*='fiction_10']" --session <sessionId>
reflex-browser wait "css=section > div > ol.row > li:nth-of-type(1) article.product_pod" 8000 --session <sessionId>
reflex-browser text "css=section > div > ol.row > li:nth-of-type(1) h3 a" --session <sessionId>
reflex-browser text "css=section > div > ol.row > li:nth-of-type(2) h3 a" --session <sessionId>
reflex-browser text "css=section > div > ol.row > li:nth-of-type(3) h3 a" --session <sessionId>
reflex-browser text "css=section > div > ol.row > li:nth-of-type(4) h3 a" --session <sessionId>
reflex-browser text "css=section > div > ol.row > li:nth-of-type(1) .price_color" --session <sessionId>
reflex-browser text "css=section > div > ol.row > li:nth-of-type(2) .price_color" --session <sessionId>
reflex-browser text "css=section > div > ol.row > li:nth-of-type(3) .price_color" --session <sessionId>
reflex-browser text "css=section > div > ol.row > li:nth-of-type(4) .price_color" --session <sessionId>
reflex-browser open "https://books.toscrape.com/catalogue/category/books/horror_31/index.html" --session <sessionId>
reflex-browser wait "css=section > div > ol.row > li:nth-of-type(1) article.product_pod" 8000 --session <sessionId>
reflex-browser text "css=section > div > ol.row > li:nth-of-type(1) h3 a" --session <sessionId>
reflex-browser text "css=section > div > ol.row > li:nth-of-type(2) h3 a" --session <sessionId>
reflex-browser text "css=section > div > ol.row > li:nth-of-type(3) h3 a" --session <sessionId>
reflex-browser text "css=section > div > ol.row > li:nth-of-type(4) h3 a" --session <sessionId>
reflex-browser text "css=section > div > ol.row > li:nth-of-type(1) .price_color" --session <sessionId>
reflex-browser text "css=section > div > ol.row > li:nth-of-type(2) .price_color" --session <sessionId>
reflex-browser text "css=section > div > ol.row > li:nth-of-type(3) .price_color" --session <sessionId>
reflex-browser text "css=section > div > ol.row > li:nth-of-type(4) .price_color" --session <sessionId>
reflex-browser session-kill <sessionId> --session <sessionId>
```

## Observed Session

- Example run returned: `session-dcbe75da`

## Result Summary

### Fiction (first 4)

1. `Soumission` - `£50.10`
2. `Private Paris (Private #10)` - `£47.61`
3. `We Love You, Charlie ...` - `£50.27`
4. `Thirst` - `£17.27`

### Horror (first 4)

1. `Security` - `£39.25`
2. `Follow You Home` - `£21.36`
3. `The Loney` - `£23.40`
4. `Pet Sematary` - `£10.56`
