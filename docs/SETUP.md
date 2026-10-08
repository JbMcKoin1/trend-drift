# Setup

The remaining setup work, as a checklist. Phase 0 (repo and Claude configuration) is done; see [ROADMAP.md](ROADMAP.md).

Placeholders: `<cpanel-user>` is the cPanel username, `<server-host>` is the server hostname from the Namecheap welcome email, and `<your-email>` is the address for notifications (not on trend-drift.com). Confirm every document root in cPanel → Domains before using it.

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

**Deadline: October 20, 2026.** Done 2026-10-07.

**Why this approach:** the current Namecheap PositiveSSL Multi-Domain certificate can't be automated, because the Namecheap SSL plugin doesn't support multi-domain certs. It would need a manual reissue about every 200 days, and that interval keeps shrinking as maximum certificate lifetimes are reduced. acme.sh with Let's Encrypt renews automatically.

> This phase changes SSL configuration and covers wholesomepup.com, both of which `CLAUDE.md` puts off-limits to Claude. The user ran every command; Claude explained each one first.

**Retire the Namecheap SSL plugin first** (cPanel → Namecheap SSL). With "HTTPS by default" on, it auto-issues certs for new domains, which would fight acme.sh. See [DECISIONS.md](DECISIONS.md).

- [x] **Switch off "HTTPS by default"** so the plugin stops issuing certs for new domains.
- Staging can't be removed from the plugin (its only option is Reinstall, which must not be used), and its free cert can't be cancelled from the Namecheap dashboard. acme.sh's deploy simply replaces it. If the plugin reissues around **April 22, 2027**, both certs are valid, so staging keeps working, and acme.sh's next renewal (at most 60 days later) puts the Let's Encrypt cert back.
- [ ] **April 2027 check:** confirm staging is still serving the Let's Encrypt cert.

All remaining commands run on the server over `ssh trend-drift`. Completed 2026-10-07.

- [x] **Install acme.sh** (use a real address *not* on trend-drift.com; Let's Encrypt rejects `example.com`):
  ```sh
  curl https://get.acme.sh | sh -s email=<your-email>
  ```
  Installs to `~/.acme.sh/` and adds a daily cron job for renewals. The email is saved as `ACCOUNT_EMAIL` in `~/.acme.sh/account.conf`.
- [x] **Set Let's Encrypt as the CA:**
  ```sh
  ~/.acme.sh/acme.sh --set-default-ca --server letsencrypt
  ```
- [x] **First issuance via DNS validation, with a temporary cPanel API token.**
  Webroot validation can't work for the *first* cert: the server forces every `http://` request to `https://` (a server-level redirect, not in any `.htaccess`), and `https://www.*` lands on the server's default site until a cert covering `www` is installed. See [DECISIONS.md](DECISIONS.md).
  1. cPanel → Security → Manage API Tokens → create `acme_temp` **with an expiry date 1–2 days out**.
  2. Load it without saving it to shell history:
     ```sh
     export cPanel_Username=<cpanel-user> cPanel_Hostname=https://<server-host>:2083
     read -s -p "Token: " cPanel_Apitoken; echo; export cPanel_Apitoken
     ```
  3. Dry run against the Let's Encrypt practice server, then the real request:
     ```sh
     ~/.acme.sh/acme.sh --issue --test --dns dns_cpanel \
       -d trend-drift.com -d www.trend-drift.com -d staging.trend-drift.com \
       -d wholesomepup.com -d www.wholesomepup.com
     # then the same command with --force instead of --test
     ```
- [x] **Deploy it to cPanel with the `cpanel_uapi` hook** (runs `uapi` locally; no token needed). Don't set `DEPLOY_CPANEL_AUTO_ENABLED`:
  ```sh
  ~/.acme.sh/acme.sh --deploy -d trend-drift.com --ecc --deploy-hook cpanel_uapi
  ```
  Expect long `NcCustomHooks::SSL` / `RestApiClient.pm` warnings. They come from a Namecheap add-on and are harmless; look for `Successfully deployed certificate to 3 of 3 sites`.
- [x] **Confirmed** all five names serve the trusted Let's Encrypt cert (expires 2027-01-05).
- [x] **Switch renewals to webroot.** Re-issue with webroot so it becomes the saved method:
  ```sh
  ~/.acme.sh/acme.sh --issue --force \
    -d trend-drift.com         -w /home/<cpanel-user>/public_html \
    -d www.trend-drift.com     -w /home/<cpanel-user>/public_html \
    -d staging.trend-drift.com -w /home/<cpanel-user>/staging.trend-drift.com \
    -d wholesomepup.com        -w /home/<cpanel-user>/wholesomepup.com \
    -d www.wholesomepup.com    -w /home/<cpanel-user>/wholesomepup.com
  ```
  **Give one `-w` per `-d`.** acme.sh pairs webroots with domains *by position*, not by grouping, so three `-w` for five `-d` maps the wrong folders. (On 2026-10-07 this was fixed by editing `Le_Webroot` in `~/.acme.sh/trend-drift.com_ecc/trend-drift.com.conf` to five entries in `Le_Domain`/`Le_Alt` order.)
- [x] **Proved webroot works** (Let's Encrypt skipped the check because of cached validations): put a test file in each `.well-known/acme-challenge/`, fetched it over `http://` through all five names (redirects to `https://` and reaches the right folder), then deleted it.
- [x] **Removed the token:** `sed -i '/^SAVED_cPanel_/d' ~/.acme.sh/account.conf` (the remaining `USER_PATH` line mentioning cpanel is harmless), revoked `acme_temp` in cPanel.
- [x] **Enable failure notifications** (level 1 = errors only; sends a test email):
  ```sh
  export MAIL_TO=<your-email>
  ~/.acme.sh/acme.sh --set-notify --notify-hook mail --notify-level 1
  ```
- [ ] **December 8, 2026 check:** the first automatic renewal (scheduled around Dec 7) succeeded, using webroot. Confirm the cert expiry has moved past Jan 5, 2027.
- [ ] **Optional:** delete the old PositiveSSL cert from cPanel → SSL/TLS → Certificates. It's no longer installed and expires Oct 20.

**Renewal depends on:** `.well-known/acme-challenge/` being reachable in all three document roots, and HTTPS staying set up for all five names. Deploys must never delete or block `.well-known/`.

## Phase 3: Tooling exploration

Tracked in [ROADMAP.md](ROADMAP.md#phase-3-tooling-exploration).

## Phase 4: Repo and docs scaffolding (complete)

Done as part of Claude configuration (Phase 0).

- [x] Repo cloned, project docs, `CLAUDE.md`, `.gitignore`, and `.claude/settings.json` committed

## Phase 5: Deploy pipeline

- [ ] Write a deploy script: build, then sync `dist/` over `ssh trend-drift` into `public_html/finance/` (production) and the matching folder on staging, never the document root itself
- [ ] Exclude `.well-known/` from every sync and delete step
- [ ] Check for server-side `.htaccess` files (HTTPS redirects) in each document root, and make sure deploys preserve them
- [ ] Deploy to staging, verify, then promote to production

## Phase 6: v1 build

Tracked in [ROADMAP.md](ROADMAP.md#phase-6--v1-personal-finance-helper). Each slice is built from a spec in [specs/](specs/).
