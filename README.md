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
