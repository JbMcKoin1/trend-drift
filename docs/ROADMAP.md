# Roadmap

Status: `[ ]` not started · `[~]` in progress · `[x]` done

## Setup

**Next up:** Phase 3 (tooling exploration).

Unresolved decisions are tracked in [OPEN-QUESTIONS.md](OPEN-QUESTIONS.md).

### Phase 0: Claude configuration
- [x] Clone the repo locally
- [x] Scaffold project docs, `CLAUDE.md`, and `.gitignore`
- [x] Set up the Claude Project
- [x] Add `.claude/settings.json` deny rules (`.env` files, `~/.ssh/`, `~/finance-data/`)

### Phase 1: Server access
- [x] Confirm SSH access to Namecheap Stellar (port 21098) with key-based auth
- [x] Create the staging subdomain and confirm its document root
- [x] Confirm the production document root for trend-drift.com

### Phase 2: SSL automation
- [x] acme.sh issues one Let's Encrypt cert for all five names and installs it via cPanel (details in [SETUP.md](SETUP.md))
- [x] Record which paths renewal depends on (`.well-known/`) so deploys never touch them
- [ ] Confirm the first automatic renewal (around December 7, 2026)

### Phase 3: Tooling exploration
- [ ] Collect the Regions and Wilson Bank & Trust export formats (spec 01 open questions) and build synthetic fixtures
- [ ] Scaffold Vite + React, served under `/finance/`
- [ ] Set up Vitest and `tests/fixtures/`
- [ ] Prototype Papa Parse on the fixtures, including background parsing under the CSP
- [ ] Set up the strict CSP and self-hosted assets; confirm no network requests after load
- [ ] Choose a chart library: prototype Chart.js vs. Observable Plot ([research](research/chart-library.md))

### Phase 4: Repo and docs scaffolding
- [x] Complete, done as part of Phase 0 (Claude configuration)

### Phase 5: Deploy pipeline
- [ ] Write a deploy script: build, then sync `dist/` over SSH into the `finance/` folder only
- [ ] Exclude `.well-known/` from every sync and delete step, and never sync or delete at the site root
- [ ] Serve the CSP and check it on staging
- [ ] Deploy to staging, verify, then promote to production

## Phase 6 — v1: Personal finance helper

v1 covers a single account (where the user's direct deposit lands). Each slice gets a spec in `docs/specs/` before implementation.

1. [ ] **CSV drop and raw view.** Drag and drop one or more CSV files, parse them in the browser, and show the raw rows in a table.
2. [ ] **Column mapping.** Map date, description, amount (or debit/credit), and the bank's category column if present. Detect the bank by header shape and remember the mapping per bank. Regions first, then Wilson Bank & Trust.
3. [ ] **Storage and months.** Save transactions in the browser, split them into months, merge overlapping files by date coverage, flag partial months, skip transactions dated on the import day, and add "Forget my data."
4. [ ] **Categorization.** User rules, then bank categories mapped to a standard set, then built-in starter rules. Corrections become remembered rules.
5. [ ] **Export and re-import.** Export full state to a versioned file; re-import merges through the same date-coverage logic.
6. [ ] **Monthly dashboard.** Monthly cash flow, spending by category, savings rate, and month-over-month trends, built so quarter and year roll-ups can be added later.
7. [ ] **Sample data.** A "Try with sample data" button using synthetic fixtures.

## Later ideas

- Quarter and year views
- Transfer detection across multiple accounts
- Recurring charge detection (subscriptions, bills)
- Optional AI-assisted categorization (top tier, opt-in, requires a deliberate CSP exception; must respect the privacy principle in `docs/PRODUCT.md`)
- Pricing tiers
