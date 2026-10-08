# 01 — CSV drop and raw view

## Goal
A user can drop one or more bank CSV files on /finance and see the parsed rows in a table, entirely in the browser.

## Behavior
- Drop zone plus a "choose files" button; accepts multiple .csv files at once.
- Parsing happens in the browser (Papa Parse), off the main thread for large files, with a progress indicator.
- Handles: byte-order mark, quoted fields containing commas, junk lines above the header row, blank trailing rows.
- Shows each file's name, row count, and detected date range, then a scrollable table of raw rows with original headers.
- Unreadable files show a plain-language error naming the file; other files still load.
- No data is stored in this slice; nothing persists after reload.
- No network requests are made.

## Out of scope
Column mapping, categorization, storage, duplicate handling, charts. Skipping transactions dated on the import day happens at storage (spec 03), so the raw view still shows them.

## Acceptance criteria
- Every fixture in `tests/fixtures/` parses to the expected row count and headers (Vitest, logic tested without React).
- A file with junk lines above the header shows the correct header row.
- A synthetic 5-year file parses without freezing the page.
- With the CSP active, the browser network panel shows no requests after page load.
- Layout is usable on a phone-width screen.

## Open questions
- What does a Regions export look like exactly: headers, date format, sign convention, category column, any preamble lines? (Needed for fixtures.)
- Same questions for Wilson Bank & Trust.
- Background parsing under the CSP: Papa Parse's built-in worker mode loads from a `blob:` URL, which a strict CSP blocks. Either allow `worker-src blob:`, or run Papa Parse inside our own small worker file served from the site (proposed). Decide before implementing.
- How should the date range be detected before column mapping exists? (Proposal: guess the date column by format; mapping in spec 02 makes it authoritative.)
