# CLAUDE.md

## Project

trend-drift.com is an analysis platform. Its first product is a **personal finance helper**: users drop in bank CSV files and see monthly cash flow, spending by category, savings rate, and month-over-month trends.

The finance helper lives at **trend-drift.com/finance**; the site root is reserved for a future platform landing page. All CSV processing happens **in the browser**, transactions are saved in the browser (IndexedDB), and the app makes **no network requests after loading**. The site is static files, and no financial data ever reaches our server.

More detail: [docs/PRODUCT.md](docs/PRODUCT.md), [docs/ROADMAP.md](docs/ROADMAP.md), [docs/DECISIONS.md](docs/DECISIONS.md), [docs/SETUP.md](docs/SETUP.md), [docs/OPEN-QUESTIONS.md](docs/OPEN-QUESTIONS.md), and feature specs in [docs/specs/](docs/specs/).

## Stack (planned, not yet installed)

- Vite + React
- Papa Parse for CSV parsing
- Charts: undecided, see [docs/research/chart-library.md](docs/research/chart-library.md)
- Vitest for tests; IndexedDB for in-browser storage
- Hosting: Namecheap Stellar shared hosting (cPanel), static files only
- Deploys: a script that pushes files over SSH (port 21098), to staging first, then production
- SSL: acme.sh / Let's Encrypt, managed on the server

## Rules

- **Never commit real financial data.** Test CSVs must be anonymized and live in `tests/fixtures/`. `.gitignore` blocks `.csv` files everywhere else.
- **Never upload or send user financial data anywhere.** All processing stays client-side. That means no analytics payloads, error reports, or API calls that contain transaction data.
- **No network requests after page load, and no analytics.** A strict Content Security Policy enforces this. Self-host every asset (fonts, scripts, icons); never load from a CDN. Don't loosen the CSP without a DECISIONS.md entry approving that specific exception.
- **Keep core logic out of React.** Parsing, categorization, duplicate handling, and month roll-ups live in plain modules tested against fixtures.
- **Namespace browser storage keys per tool** (for example `finance:`), since all tools share one origin.
- **Deploys must never delete or overwrite the `.well-known` folder** on the server, because acme.sh uses it to renew certificates. Exclude it from any sync or delete step.
- **Deploys touch only the app's own folder** (`/finance`). Never sync or delete at the site root: it holds the current site and the `.htaccess` file.
- **Never change DNS, email, or SSL configuration.** cPanel email is live on this domain.
- **Deploy to staging before production.**
- **After finishing a feature,** update `docs/ROADMAP.md` and `docs/DECISIONS.md`.
- **Feature specs live in `docs/specs/`.** Implement from the spec, and flag anything ambiguous instead of guessing.

## Out of scope

wholesomepup.com is on the same hosting account. Don't touch it.
