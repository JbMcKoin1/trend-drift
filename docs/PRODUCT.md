# Product

## What trend-drift is

trend-drift.com is an analysis platform: a set of simple tools that turn raw data people already have into clear trends over time. Each tool is a static web app that runs entirely in the browser.

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

1. **Privacy first.** Financial data never leaves the user's device. All parsing, categorizing, and charting happens in the browser. There is no account to link, no upload, and no server-side storage. Any saved settings (column mappings, category rules) live in the browser's local storage.
2. **Very easy to use.** Dropping in a file should be enough to get a useful result. Setup steps such as mapping columns or fixing categories should be short, and the app remembers them so they only happen once per bank.
3. **Nearly free to run.** The site is static files on shared hosting. There are no backend servers, databases, or paid APIs to run.

### Known hard problems

- **Bank CSV formats vary.** Column names, date formats, and sign conventions differ by bank. The app needs a column-mapping step that's remembered for each bank.
- **Categorization.** Keyword rules give a starting point, and user corrections override them and are remembered.
- **Transfer detection.** Moving money between your own accounts (checking to savings, card payments) must not be counted twice as both spending and income.
