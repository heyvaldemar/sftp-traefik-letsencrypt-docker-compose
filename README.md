# SFTP + Traefik + Let's Encrypt — Docker Compose

[![Deployment Verification](https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/actions/workflows/deployment-verification.yml/badge.svg?branch=main)](https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/actions/workflows/deployment-verification.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This repository deploys an **SFTP server** ([atmoz/sftp](https://github.com/atmoz/sftp)) behind **Traefik's TCP router**, with the Traefik dashboard on HTTPS via **Let's Encrypt**. Two chrooted accounts are provisioned from `.env`; their data lands on the host under `/srv/sftpusers/<user>`.

## Getting started

```bash
# 1. Clone
git clone https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose
cd sftp-traefik-letsencrypt-docker-compose

# 2. Create the two Docker networks the stack expects
docker network create traefik-network
docker network create sftp-network

# 3. Create the host data directories
sudo mkdir -p /srv/sftpusers/user1 /srv/sftpusers/user2

# 4. Copy the environment template and fill in required values
cp .env.example .env
$EDITOR .env

# 5. Deploy
docker compose -f sftp-traefik-letsencrypt-docker-compose.yml -p sftp up -d
```

Connect with any SFTP client:

```bash
sftp -P 2222 user1@sftp.example.com
```

### What success looks like

```bash
docker compose -f sftp-traefik-letsencrypt-docker-compose.yml -p sftp ps
sftp -P 2222 user1@YOUR_SERVER   # prompts for the password from .env
```

### Common first-deploy issues

- **Connection refused.** The `SFTP_PORT` (default 2222) must be open in your firewall; Traefik publishes it directly.
- **Login works but uploads fail.** Upload into the `data` subdirectory — the chroot home itself is read-only by design (atmoz/sftp requirement).
- **Networks not found.** Step 2 was skipped.

## Supply chain trust

Two images — [`traefik`](https://hub.docker.com/_/traefik) and [`atmoz/sftp`](https://hub.docker.com/r/atmoz/sftp) — pinned by digest as interpolation defaults in the compose `x-images` block (atmoz/sftp publishes no semver tags, so the `debian` tag is pinned to an exact digest). `git pull` alone delivers the tested combination.

The weekly `check-pin-freshness` CI job re-resolves each pin against its registry; GitHub Actions are pinned by commit SHA and Dependabot keeps those fresh.

## Production checklist

- [ ] **Strong passwords** — 24+ random characters per account; regenerate the Traefik dashboard hash.
- [ ] **Prefer SSH keys** for real workloads — atmoz/sftp supports mounting public keys per user; password auth is the lowest bar.
- [ ] **Back up `/srv/sftpusers/`** — that's where all uploaded data lives.
- [ ] **Watch the upstream image** — atmoz/sftp moves slowly; the weekly digest check tells you when a rebuild lands.

## Unattended updates

Releases are the update channel: a tag is cut only after CI has built the pinned images, booted the full stack, and passed the smoke tests. `update.sh` moves a deployment to the newest tag and nothing else:

```bash
./update.sh --dry-run   # show what would be applied
./update.sh             # update within the current major and redeploy
```

Put it on a timer for hands-off minor/patch updates:

```bash
# crontab -e
17 5 * * *  /opt/sftp-traefik-letsencrypt-docker-compose/update.sh >> /var/log/sftp-update.log 2>&1
```

The script refuses to cross a MAJOR template version on its own — majors are breaking by definition and their release notes exist to be read. After reading them, `./update.sh --allow-major` performs the jump. It also refuses to touch a checkout with local modifications: your customization belongs in `.env`, which updates never overwrite.

This is deliberately a host-side script and not a container in the stack: an in-stack updater needs the Docker socket (root on the host) and turns "someone pushed to a repo" into "someone deployed to your machine" with no operator in the loop. A cron job under your own user updates only to tagged, CI-verified states and leaves the trust boundary where it was.

## Resource limits

Every service carries memory and CPU limits plus reservations as compose-level defaults — the same values CI boots the stack under. Override any of them in `.env` (the knobs and their defaults are listed in `.env.example`, e.g. `TRAEFIK_MEMORY_LIMIT=512m`) and the override survives every `git pull`. If a service is OOM-killed under real load, `docker inspect <container> --format '{{.State.OOMKilled}}'` says so; raise its `_MEMORY_LIMIT` and recreate.

## Testing

The [Deployment Verification](https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/actions/workflows/deployment-verification.yml?query=branch%3Amain) workflow runs on every push, pull request, and every Monday at 06:00 UTC: actionlint, Trivy scans of both pinned images, the weekly digest check, and a deploy-and-test job that performs a real SFTP login and directory listing through Traefik's TCP router with an ephemeral password.

## Security Notes

- Credentials are read from `.env` at deploy time; `.env` is gitignored and compose fails fast on missing required variables.
- **Pre-rotation advisory.** Releases before v1.0.0 (2026-09-01) shipped a tracked `.env` with generated-looking account passwords. Rotate them if your deployment reused them.
- Each account is chrooted to its own home; users cannot see each other's data.

---

## About the maintainer

<div align="center">

**Maintained by [Vladimir Mikhalev](https://github.com/heyvaldemar)** — Docker Captain · IBM Champion · AWS Community Builder

[YouTube](https://www.youtube.com/channel/UCf85kQ0u1sYTTTyKVpxrlyQ?sub_confirmation=1) · [Blog](https://heyvaldemar.com) · [LinkedIn](https://www.linkedin.com/in/heyvaldemar/)

</div>
