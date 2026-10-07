# Decisions

A dated log of project decisions. Add new entries at the bottom, and don't rewrite old ones. If a decision is reversed, add a new entry that refers back to the old one.

| Date | Decision | Reason |
|------|----------|--------|
| 2026-10-06 | All CSV processing happens client-side in the browser; no financial data is uploaded or stored on the server. | Protects user privacy and removes the need for a backend. |
| 2026-10-06 | Host as static files on Namecheap Stellar shared hosting (cPanel). | Hosting is already paid for, and static files keep running costs near zero. |
| 2026-10-06 | SSL via Let's Encrypt using acme.sh on the server. | Free, automated certificate renewal that works on cPanel shared hosting. |
| 2026-10-06 | Deploys go to a staging subdomain first, then production. | Catches broken builds before real users see them. |
| 2026-10-06 | Real bank exports live in `~/finance-data/`, outside the repo, and Claude Code is denied read access to that folder. | Keeps real financial data out of git and away from AI tooling. Only anonymized fixtures go in `tests/fixtures/`. |
