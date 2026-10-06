# CLAUDE.md

## Project

trend-drift.com is an analysis platform. Its first product is a **personal finance helper**: users drop in bank CSV files and see monthly cash flow, spending by category, savings rate, and month-over-month trends.

All CSV processing happens **in the browser**. The site is static files, and no financial data ever reaches our server.

More detail: [docs/PRODUCT.md](docs/PRODUCT.md), [docs/ROADMAP.md](docs/ROADMAP.md), [docs/DECISIONS.md](docs/DECISIONS.md).

## Stack (planned, not yet installed)

- Vite + React
- Papa Parse for CSV parsing
- Chart.js (or similar) for charts
- Hosting: Namecheap Stellar shared hosting (cPanel), static files only
- Deploys: a script that pushes files over SSH (port 21098), to staging first, then production
- SSL: acme.sh / Let's Encrypt, managed on the server

## Rules

- **Never commit real financial data.** Test CSVs must be anonymized and live in `tests/fixtures/`. `.gitignore` blocks `.csv` files everywhere else.
- **Never upload or send user financial data anywhere.** All processing stays client-side. That means no analytics payloads, error reports, or API calls that contain transaction data.
- **Deploys must never delete or overwrite the `.well-known` folder** on the server, because acme.sh uses it to renew certificates. Exclude it from any sync or delete step.
- **Never change DNS, email, or SSL configuration.** cPanel email is live on this domain.
- **Deploy to staging before production.**
- **After finishing a feature,** update `docs/ROADMAP.md` and `docs/DECISIONS.md`.
- **Feature specs live in `docs/specs/`.** Implement from the spec, and flag anything ambiguous instead of guessing.

## Out of scope

wholesomepup.com is on the same hosting account. Don't touch it.
