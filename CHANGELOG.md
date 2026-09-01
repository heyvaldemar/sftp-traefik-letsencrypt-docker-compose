# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

_(no unreleased changes yet)_

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
  hardcoded to `user1`/`user2` while the usernames were variables —
  renaming an account silently mounted the wrong home directory.

### Security

- **Credentials untracked from git.** The tracked `.env` carried
  generated-looking passwords for both accounts — rotate them if
  reused.

### Added

- **Deployment Verification workflow**: actionlint; Trivy scans of both
  pinned images; weekly digest-drift check; and a deploy-and-test job
  that performs a real SFTP login and directory listing through
  Traefik's TCP router.

[Unreleased]: https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/heyvaldemar/sftp-traefik-letsencrypt-docker-compose/releases/tag/v1.0.0
