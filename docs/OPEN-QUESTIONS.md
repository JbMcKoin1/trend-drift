# Open questions

Decisions still waiting on a discussion. When one is settled, record it in [DECISIONS.md](DECISIONS.md), update the affected docs, and delete it here.

## 1. Background parsing under the CSP (blocks spec 01)

Parsing a multi-year CSV on the page itself freezes it; a **Web Worker** (a background helper in the browser) keeps the page responsive. Papa Parse's built-in worker mode (`worker: true`) creates its worker from a temporary `blob:` address, which the planned strict CSP refuses. Spec 01 requires both "no freezing" and the strict CSP, so one of these must give:

- **A. Allow `worker-src blob:` in the CSP.** Small real risk (only code already on the page can create a `blob:` address, and network requests stay blocked), but it's an exception to a no-exceptions policy and needs a DECISIONS.md entry.
- **B. Our own worker file (proposed).** About 30–50 lines: receives the file, runs Papa Parse in chunks, posts progress and rows back. Vite builds it as a same-origin file, so the CSP needs no exception. Papa Parse still does all the parsing. Maintenance is low: Web Workers and Vite's worker support are stable, and Papa Parse updates drop in. The same worker can later run duplicate merging and month roll-ups on multi-year history.

See [specs/01-csv-drop-and-raw-view.md](specs/01-csv-drop-and-raw-view.md).

## 2. What's in the v1 dashboard?

The 2026-10-07 scope decision says v1 "shows monthly cash flow and spending by category." [PRODUCT.md](PRODUCT.md) and ROADMAP slice 6 also list **savings rate** and **month-over-month trends**. Are those two in v1, or later?

## 3. Keep exported data files out of git (spec 05)

The export file (full state: transactions, rules, mappings) contains real financial data. Once spec 05 fixes its file extension or name pattern, add it to `.gitignore` the same way `.csv` is blocked, so an export saved inside the project folder can't be committed.

## 4. Import-day rule: exported yesterday, dropped today

The rule skips transactions dated on the day of **import** (the drop date). A file **exported** yesterday but dropped today keeps yesterday's rows, even if that day was incomplete when exported. Accept this, or also skip the file's last date when it's the export day? (Bank CSVs rarely state their export date, so the second option may not be detectable.)

## Also open, inside specs

- Spec 01: exact Regions and Wilson Bank & Trust export formats (headers, date format, sign convention, category column, preamble lines), and how to detect the date range before column mapping exists.
