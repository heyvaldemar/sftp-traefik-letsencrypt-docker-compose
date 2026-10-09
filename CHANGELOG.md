# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Security

- **`traefik:3.7` was rebuilt upstream**; the pin moved from `sha256:b588cb566045…` to `sha256:575fa15b1350…`. Same version, same tag, a rebuilt base image — the usual shape of a security fix in a base layer.

## [1.5.2] - 2026-10-07

### Security

- **`traefik:3.7` was rebuilt upstream**; the pin moved from `sha256:24841fe2de73…` to `sha256:b588cb566045…`. Same version, same tag, a rebuilt base image — the usual shape of a security fix in a base layer.
- **`atmoz/sftp:debian` was rebuilt upstream**; the pin moved from `sha256:77bbedb7f120…` to `sha256:f308cffe2724…`. Same version, same tag, a rebuilt base image — the usual shape of a security fix in a base layer.

## [1.5.1] - 2026-10-06

### Fixed

- **`update.sh` no longer stops without a word when a release adds a variable and no compose file requires one.** The search for `${VAR:?}` came back empty, and under `pipefail` that empty result ended the script with status 1 right after it listed the new variables.
- **`update.sh` stops on a `.env` it cannot read, before the checkout.** It used to fall through: every new required variable read as "not set", or, with none, the tree moved to the new tag and `docker compose up` failed on the permission. Now it names the file, its owner and mode, and changes nothing.

### Security

- **`atmoz/sftp:debian` was rebuilt upstream**; the pin moved from `sha256:75dcc29683ad…` to `sha256:77bbedb7f120…`. Same version, same tag, a rebuilt base image — the usual shape of a security fix in a base layer.

## [1.5.0] - 2026-09-26

### Added

- **Traefik's timeouts on the HTTPS entry point can be set from `.env`.**
  `TRAEFIK_READ_TIMEOUT`, `TRAEFIK_WRITE_TIMEOUT` and `TRAEFIK_IDLE_TIMEOUT`
  default to Traefik's own values (60s, 0s, 180s), so nothing changes unless
  you set them. Traefik reads its static configuration from one source, here
  the command in the compose file, and an override file can only replace that
  command whole; a variable is the way to tune it and keep taking updates.
  The same change was asked for in the [Keycloak template](https://github.com/heyvaldemar/keycloak-traefik-letsencrypt-docker-compose), and every template in the fleet gets it at once.

## [1.4.5] - 2026-09-26

### Security

- **`atmoz/sftp:debian` was rebuilt upstream**; the pin moved from `sha256:2b7fa66f4aa7…` to `sha256:75dcc29683ad…`. Same version, same tag, a rebuilt base image — the usual shape of a security fix in a base layer.

## [1.4.4] - 2026-09-23

### Changed

- **The freshness check has its own workflow, Pin Freshness.** It ran inside Deployment Verification, whose badge is the one at the top of this README. Across the fleet, nine red runs in ten were a pin one version behind - which the fleet's triage moves within the day - and a reader cannot tell that from a stack that does not boot. The badge now says whether the stack boots. The job itself is unchanged.

### Security

- **`atmoz/sftp:debian` was rebuilt upstream**; the pin moved from `sha256:2b314c149154…` to `sha256:2b7fa66f4aa7…`. Same version, same tag, a rebuilt base image — the usual shape of a security fix in a base layer.

## [1.4.3] - 2026-09-21

### Security

- **`traefik:3.7` was rebuilt upstream**; the pin moved from `sha256:1c32e7c36820…` to `sha256:24841fe2de73…`. Same version, same tag, a rebuilt base image — the usual shape of a security fix in a base layer.

## [1.4.2] - 2026-09-18

### Security

- **`traefik:3.7` was rebuilt upstream**; the pin moved from `sha256:f86a2cab1b5c…` to `sha256:1c32e7c36820…`. Same version, same tag, a rebuilt base image — the usual shape of a security fix in a base layer.
- **`atmoz/sftp:debian` was rebuilt upstream**; the pin moved from `sha256:a5a0081d3538…` to `sha256:2b314c149154…`. Same version, same tag, a rebuilt base image — the usual shape of a security fix in a base layer.

## [1.4.1] - 2026-09-07

### Changed

- **`update.sh` names any new required variable before it moves.** An update can add a required variable; `docker compose up` used to stop on it after the checkout, with the tree already on the new tag. The script now lists the variables that appeared in `.env.example` since your version and refuses, before anything has moved, when a required one is not in your `.env`. Names only, never values.

 as before. A deployment that sets neither is unchanged. The
  freshness job, the Trivy matrix and the fleet digest automation resolve
  the nested default before reading a pin. Needs Docker Compose v2.5 or
  newer (2022): v2.0 to v2.4 leave the inner `${...}` unexpanded and
  `docker compose up` fails with an invalid reference instead of
  deploying something unexpected.

## [1.3.0] - 2026-09-02

### Security

- **Container hardening.** Every service runs with
  `security_opt: no-new-privileges:true` (no privilege escalation via
  setuid binaries even if a process escapes its initial capability
  set). Infrastructure containers (the reverse proxy, databases,
  caches, backups) drop every Linux capability and add back only what
  their entrypoints need (bind :80/:443, chown a data directory, drop to
  the service user). Application containers keep the default capability
  set: upstream images assume it, and a wrong guess there is a boot loop
  in production, not a hardening win. CI boots the stack under these
  settings on every push.

## [1.2.0] - 2026-09-02

### Added

- **Resource limits on every service, as `.env`-overridable defaults.**
  Each service now carries memory and CPU limits plus reservations
  (`<SERVICE>_MEMORY_LIMIT`, `_CPU_LIMIT`, `_MEMORY_RESERVATION`,
  `_CPU_RESERVATION`, defaults listed in `.env.example`). Set any of
  them in `.env` and the override survives every `git pull`. The
  defaults are what CI boots the stack under, so they are known to be
  enough for a fresh install; raise a limit if a service is OOM-killed
  under your real load (`docker inspect` shows `OOMKilled=true`).

## [1.1.0] - 2026-09-02

### Added

- **`update.sh`**: unattended updates to the newest tagged release,
  and nothing else: a tag is cut only after CI has booted the pinned
  images and passed the smoke tests, so "update to the latest tag" means
  "update to a combination a machine has already run". It refuses to
  cross a major version on its own (`--allow-major` after reading the
  notes), refuses a checkout with local modifications, and supports
  `--dry-run`. Put it on a cron timer for hands-off minor/patch updates.

## [1.0.0] - 2026-09-01

First semver release. Brings this template to the fleet standard established
in [keycloak-traefik-letsencrypt-docker-compose](https://github.com/heyvaldemar/keycloak-traefik-letsencrypt-docker-compose)
v1.2.0.

### Changed

- **Traefik 3.2 → 3.7** (3.2's Docker client cannot talk to Docker
  Engine 29); the `atmoz/sftp:debian` floating tag is now digest-pinned.
  Both pins live in the compose `x-images` block.

### Fixed

- **Host data paths follow the account names**: the bind mounts were
  hardcoded to `user1`/`user2` while the usernames were variables:
  renaming an account silently mounted the wrong home directory.

### Security

- **Credentials untracked from git.** The tracked `.env` carried
  generated-looking passwords for both accounts. Rotate them if
  reused.

### Added

- **Deployment Verification workflow**: actionlint; Trivy scans of both
  pinned images; weekly digest-drift check; and a deploy-and-test job
  that performs a real SFTP login and directory listing through
  Traefik's TCP router.

[Unreleased]: https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/compare/v1.5.2...HEAD
[1.5.2]: https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/compare/v1.5.1...v1.5.2
[1.5.1]: https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/compare/v1.5.0...v1.5.1
[1.5.0]: https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/compare/v1.4.5...v1.5.0
[1.4.5]: https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/compare/v1.4.4...v1.4.5
[1.4.4]: https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/compare/v1.4.3...v1.4.4
[1.4.3]: https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/compare/v1.4.2...v1.4.3
[1.4.2]: https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/compare/v1.4.1...v1.4.2
[1.4.1]: https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/compare/v1.4.0...v1.4.1
[1.4.0]: https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/compare/v1.3.0...v1.4.0
[1.3.0]: https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/compare/v1.2.0...v1.3.0
[1.2.0]: https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/releases/tag/v1.0.0
