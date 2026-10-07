# Setup

The remaining setup work, as a checklist. Phase 0 (repo and Claude configuration) is done; see [ROADMAP.md](ROADMAP.md).

Placeholders: `<cpanel-user>` is the cPanel username, `<server-host>` is the server hostname from the Namecheap welcome email, and `<you@example.com>` is the address for notifications. Confirm every document root in cPanel → Domains before using it.

## Phase 1: SSH access

- [x] **Enable SSH:** cPanel → Manage Shell → enable SSH access.
- [x] **Create a key locally** (on your machine, not the server):
  ```sh
  ssh-keygen -t ed25519 -f ~/.ssh/trend_drift -C "trend-drift deploy"
  ```
  Use a passphrase. This creates `~/.ssh/trend_drift` (private, never shared) and `~/.ssh/trend_drift.pub` (public).
- [x] **Import and authorize the public key:** cPanel → SSH Access → Manage SSH Keys → Import Key. Paste the contents of `trend_drift.pub`, then click Manage → Authorize.
- [x] **Add a host entry** to `~/.ssh/config`:
  ```
  Host trend-drift
      HostName <server-host>
      User <cpanel-user>
      Port 21098
      IdentityFile ~/.ssh/trend_drift
      IdentitiesOnly yes
  ```
- [x] **Test the connection:** `ssh trend-drift`. It should log in without a password prompt (other than the key passphrase).
- [x] **Create the staging subdomain:** cPanel → Domains → create `staging.trend-drift.com` with "Share document root" **unticked** (otherwise staging serves the production folder).

**Gotcha:** the SSH `User` is the **cPanel** username (shown in the cPanel Terminal prompt), not the Namecheap account login. Using the wrong one gives `Permission denied (publickey)` and a fallback password prompt.

Document roots (relative to `/home/<cpanel-user>/`):

| Domain | Document root |
|--------|---------------|
| trend-drift.com | `public_html` |
| staging.trend-drift.com | `staging.trend-drift.com` |
| wholesomepup.com | `wholesomepup.com` |

## Phase 2: SSL automation

**Deadline: October 20, 2026.**

**Why this approach:** the current Namecheap PositiveSSL Multi-Domain certificate can't be automated, because the Namecheap SSL plugin doesn't support multi-domain certs. It would need a manual reissue about every 200 days, and that interval keeps shrinking as maximum certificate lifetimes are reduced. acme.sh with Let's Encrypt renews automatically.

> This phase changes SSL configuration and covers wholesomepup.com, both of which `CLAUDE.md` puts off-limits to Claude by default. Do these steps yourself, or explicitly authorize Claude for this phase.

All commands run on the server over `ssh trend-drift`.

- [ ] **Install acme.sh:**
  ```sh
  curl https://get.acme.sh | sh -s email=<you@example.com>
  ```
  Installs to `~/.acme.sh/` and adds a daily cron job for renewals.
- [ ] **Set Let's Encrypt as the CA:**
  ```sh
  ~/.acme.sh/acme.sh --set-default-ca --server letsencrypt
  ```
- [ ] **Issue one cert for all five names, using webroot validation.** Each `-w` sets the webroot for the `-d` names that follow it:
  ```sh
  ~/.acme.sh/acme.sh --issue \
    -w /home/<cpanel-user>/public_html             -d trend-drift.com -d www.trend-drift.com \
    -w /home/<cpanel-user>/staging.trend-drift.com -d staging.trend-drift.com \
    -w /home/<cpanel-user>/wholesomepup.com        -d wholesomepup.com -d www.wholesomepup.com
  ```
  All five names must already resolve to this server, and each webroot must serve `/.well-known/acme-challenge/`.
- [ ] **Deploy it to cPanel with the `cpanel_uapi` hook:**
  ```sh
  export DEPLOY_CPANEL_AUTO_ENABLED=1
  ~/.acme.sh/acme.sh --deploy -d trend-drift.com --deploy-hook cpanel_uapi
  ```
  Auto mode installs the cert on every domain it covers. acme.sh saves the hook and reruns it after each renewal.
- [ ] **Confirm in a browser** that all five names show the new Let's Encrypt cert.
- [ ] **Enable failure notifications:**
  ```sh
  export MAIL_TO=<you@example.com>
  ~/.acme.sh/acme.sh --set-notify --notify-hook mail --notify-level 1
  ```
  Level 1 sends email only on errors.
- [ ] **Force one test renewal** to prove the full cycle works, including the deploy hook:
  ```sh
  ~/.acme.sh/acme.sh --renew -d trend-drift.com --force
  ```
- [ ] **Remove the old PositiveSSL cert** from cPanel once the new one is confirmed on all five names.

## Phase 3: Tooling exploration

- [ ] Scaffold Vite + React
- [ ] Prototype Papa Parse on sample (anonymized) bank CSVs
- [ ] Choose a chart library (Chart.js or alternative)
- [ ] Set up the test runner and `tests/fixtures/`

## Phase 4: Repo and docs scaffolding (complete)

Done as part of Claude configuration (Phase 0).

- [x] Repo cloned, project docs, `CLAUDE.md`, `.gitignore`, and `.claude/settings.json` committed

## Phase 5: Deploy pipeline

- [ ] Write a deploy script: build, then sync `dist/` to the server over `ssh trend-drift`
- [ ] Exclude `.well-known/` from every sync and delete step
- [ ] Deploy to staging, verify, then promote to production

## Phase 6: v1 build

Build the five v1 slices in [ROADMAP.md](ROADMAP.md), each from a spec in [specs/](specs/):

- [ ] CSV drop and raw view
- [ ] Column mapping
- [ ] Categorization
- [ ] Transfer detection
- [ ] Monthly dashboard
