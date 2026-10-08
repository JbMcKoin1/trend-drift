# Product

## What trend-drift is

trend-drift.com is an analysis platform: a set of simple tools that turn raw data people already have into clear trends over time. Each tool is a static web app that runs entirely in the browser and lives at its own path, starting with trend-drift.com/finance. The site root is reserved for a platform landing page.

## Who it's for

People who want answers from their own data without setting up spreadsheets, linking accounts to third-party services, or paying for a subscription.

## First product: personal finance helper

### Purpose

Take someone from **zero to one** in visibility into their monthly finances. They export CSV files from their bank, drop them on the page, and see:

- **Monthly cash flow:** income vs. spending per month
- **Spending by category:** where the money goes
- **Savings rate:** share of income kept each month
- **Month-over-month trends:** what's going up or down

### Principles

1. **Privacy first.** Financial data never leaves the user's device. All parsing, categorizing, and charting happens in the browser, and the app makes no network requests after loading (enforced by a Content Security Policy). There is no account to link, no upload, no analytics, and no server-side storage. Transactions and settings are saved in the browser so history builds up between visits; users can wipe everything with one control, or export their data to a file and re-import it later.
2. **Very easy to use.** Dropping in a file should be enough to get a useful result. Setup steps such as mapping columns or fixing categories should be short, and the app remembers them so they only happen once per bank. A "Try with sample data" button shows the payoff before anyone has to find a bank export. Desktop-first; phones must work, not necessarily look polished.
3. **Nearly free to run.** The site is static files on shared hosting. There are no backend servers, databases, or paid APIs to run.

### Known hard problems

- **Bank CSV formats vary.** Column names, date formats, and sign conventions differ by bank. The app needs a column-mapping step that's remembered for each bank.
- **Categorization.** User rules come first, then the bank's own category (mapped to our standard set), then built-in starter rules. User corrections become remembered rules.
- **Overlapping exports.** Bank exports usually cover ~90 days and overlap each other, and re-imported exports overlap everything. Data must be merged by date coverage so nothing is counted twice and genuine same-day duplicates are kept. Transactions dated on the day of import are never imported, because that day may still be incomplete.
- **Transfer detection (after v1).** Moving money between your own accounts (checking to savings, card payments) must not be counted twice as both spending and income. Deferred because v1 handles a single account.
