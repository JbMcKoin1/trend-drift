# Chart library (open decision)

Status: undecided. Pick by building the hardest v1 chart (monthly income vs. spending) in two candidates and comparing by eye.

## Hard requirements
- Good-looking, readable defaults (ease of use extends to charts)
- Dark mode and reasonable accessibility
- No external requests: no CDN scripts or remote fonts (CSP blocks them)
- Works under a strict CSP (check for inline-style injection)
- Sits behind a thin wrapper so it can be swapped later

## Candidates

**Chart.js** (named in the original plan)
- Pros: mature and widely used; standard bar/line/donut charts work well; draws to canvas, so large datasets stay fast.
- Cons: needs a React wrapper (react-chartjs-2); canvas output is harder to make accessible and to style with CSS.

**Recharts**
- Pros: built for React, so it fits the app naturally; SVG output is easy to style.
- Cons: awkward for unusual or custom charts; larger bundle; can slow down with very many data points.

**Observable Plot**
- Pros: small; excellent defaults for trend-over-time data, which is most of this app; concise code.
- Cons: not React-specific (needs a small wrapper); less interactivity out of the box; smaller ecosystem.

## Next step
Prototype Chart.js against Observable Plot, record the choice in DECISIONS.md.
