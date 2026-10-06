# Roadmap

Status: `[ ]` not started · `[~]` in progress · `[x]` done

## Setup

### Phase 1: Server access
- [ ] Confirm SSH access to Namecheap Stellar (port 21098) with key-based auth
- [ ] Create the staging subdomain and confirm its document root
- [ ] Confirm the production document root for trend-drift.com

### Phase 2: SSL automation
- [ ] Confirm acme.sh is issuing and auto-renewing Let's Encrypt certificates for the production and staging domains
- [ ] Record which paths renewal depends on (`.well-known/`) so deploys never touch them

### Phase 3: Tooling exploration
- [ ] Scaffold Vite + React
- [ ] Prototype Papa Parse on sample (anonymized) bank CSVs
- [ ] Choose a chart library (Chart.js or alternative)
- [ ] Set up the test runner and `tests/fixtures/`

### Phase 4: Deploy pipeline
- [ ] Write a deploy script: build, then sync `dist/` over SSH
- [ ] Exclude `.well-known/` from every sync and delete step
- [ ] Deploy to staging, verify, then promote to production

## v1: Personal finance helper

Each slice gets a spec in `docs/specs/` before implementation.

1. [ ] **CSV drop and raw view.** Drag and drop one or more CSV files, parse them in the browser, and show the raw rows in a table.
2. [ ] **Column mapping.** Map date, description, and amount (or debit/credit) columns. Detect the bank by header shape and remember the mapping per bank in local storage.
3. [ ] **Categorization.** Apply keyword rules to assign categories. The user can correct a category, and corrections become remembered rules.
4. [ ] **Transfer detection.** Find matching opposite-sign amounts across accounts within a short date window, mark them as transfers, and exclude them from income and spending totals.
5. [ ] **Monthly dashboard.** Show monthly cash flow, spending by category, savings rate, and month-over-month trends.

## Later ideas

- Recurring charge detection (subscriptions, bills)
- Optional AI-assisted categorization (opt-in, must respect the privacy principle in `docs/PRODUCT.md`)
