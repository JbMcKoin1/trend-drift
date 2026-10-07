# Decisions

A dated log of project decisions. Add new entries at the bottom, and don't rewrite old ones. If a decision is reversed, add a new entry that refers back to the old one.

| Date | Decision | Reason |
|------|----------|--------|
| 2026-10-06 | All CSV processing happens client-side in the browser; no financial data is uploaded or stored on the server. | Protects user privacy and removes the need for a backend. |
| 2026-10-06 | Host as static files on Namecheap Stellar shared hosting (cPanel). | Hosting is already paid for, and static files keep running costs near zero. |
| 2026-10-06 | SSL via Let's Encrypt using acme.sh on the server. | Free, automated certificate renewal that works on cPanel shared hosting. |
| 2026-10-06 | Deploys go to a staging subdomain first, then production. | Catches broken builds before real users see them. |
| 2026-10-06 | Real bank exports live in `~/finance-data/`, outside the repo, and Claude Code is denied read access to that folder. | Keeps real financial data out of git and away from AI tooling. Only anonymized fixtures go in `tests/fixtures/`. |
| 2026-10-07 | acme.sh manages certificates for all five names, including staging. The Namecheap SSL plugin is retired: "HTTPS by default" is switched off, and acme.sh's deploy replaces the plugin's staging cert (the plugin can't release staging). | One renewal system avoids two tools overwriting each other's certificates. |
| 2026-10-07 | First certificate issued with DNS validation using a short-lived cPanel API token; renewals use webroot validation, and the token was removed and revoked. | A server-level HTTP-to-HTTPS redirect blocks webroot validation for names without a cert yet. Keeping a full-access token on the server permanently would turn a website compromise into an account takeover. |
