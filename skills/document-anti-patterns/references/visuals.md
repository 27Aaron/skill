# Visuals and Image Slots

## Visual Excess

| # | Pattern | Bad | Fix |
|---|---------|-----|-----|
| 28 | More than 3 brand-color accents per page | Four different colored highlights on the same page | One accent color for emphasis; use weight or size for hierarchy |
| 29 | Chart with no insight title | Chart titled "Revenue by Quarter" | Title states the insight: "Revenue accelerated after Q2 price change" |
| 30 | Decorative chart that restates the text | Bar chart showing the same three numbers the paragraph just listed | If the text already communicates it, the chart must add a dimension (comparison, trend, distribution) |
| 31 | Icon or emoji as section marker | Sections led by decorative icons with no semantic value | Use typographic hierarchy (size, weight, spacing) instead |
| 32 | Label chrome over every heading | `01 · FEATURES` above each section of a page whose sections are not a sequence; `Design · Build · Ship` joining words that are not data fields | Number only real sequences (chapters, slides, steps); keep one eyebrow on the cover or hero at most; a middle dot only separates data fields on one line |
| 33 | Hand-placed chart geometry | Bar heights or arc endpoints typed into the SVG, so a bar and its label disagree | Write the numbers as a spec and generate the chart from it with code; change data in the spec, never in the SVG |

## Image Slots

| # | Pattern | Bad | Fix |
|---|---------|-----|-----|
| 34 | Image generated before slot | A polished 16:9 concept image is forced into a 3:2 feature panel and loses its focal point | Decide the slot first, then crop, pad, or generate to that ratio |
| 35 | Real screenshot redrawn as fake UI | A product screenshot is AI-redesigned and no longer matches the shipped app | Preserve real screenshots; use programmatic padding or split panels before redrawing |
| 36 | Mixed screenshot ratios in one group | Three gallery panels jump between tall, wide, and square crops | Normalize the frame and padding; keep the screenshot content intact inside it |
| 37 | Missing product image filled with atmosphere | A product page uses abstract texture because no screenshot was provided | Mark the material gap or omit the panel; do not substitute unrelated imagery |
