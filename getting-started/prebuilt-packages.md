---
title: Pre-built Packages
description: Install DocumentDB from the package repository, or from the release assets and container image published with each release.
---

# Pre-built Packages

**Easiest path:** install from the package repository — the package manager resolves the dependency graph for you. See [Linux Packages Quick Start](https://documentdb.io/docs/getting-started/packages/) for the exact command for your distribution, or the [Package Finder](https://documentdb.io/packages).

This page covers the **release assets** instead: what each release publishes, and how to install from downloaded files.

## Install from downloaded assets

Enable the PostgreSQL upstream (PGDG) repository first; RHEL-compatible hosts also need EPEL and CRB — see [Package Installation](https://documentdb.io/packages).

Pass all six files for your platform to **one** command. `apt` and `dnf` resolve dependencies only from repository indexes, so the meta package alone fails with `documentdb : Depends: documentdb-18 (>= 0.116.0) but it is not installable`.

### DEB (Ubuntu 24.04, PostgreSQL 18, amd64)

For arm64 swap `amd64` → `arm64`; for PostgreSQL 17 swap `18` → `17`. Only the gateway and extension assets are arch-specific — the other four are `_all.deb`.

```bash
sudo apt install ./ubuntu24.04-documentdb_0.116.0_all.deb \
                 ./ubuntu24.04-documentdb-18_0.116.0_all.deb \
                 ./ubuntu24.04-documentdb-common_0.116.0_all.deb \
                 ./ubuntu24.04-documentdb-postgresql-tools_0.116.0_all.deb \
                 ./ubuntu24.04-documentdb-gateway_0.116.0_amd64.deb \
                 ./ubuntu24.04-postgresql-18-documentdb_0.116-0_amd64.deb
```

### RPM (RHEL-compatible 9, PostgreSQL 18, x86_64)

For arm64 swap `x86_64` → `aarch64`; for PostgreSQL 17 swap `18` → `17`. Only the gateway and extension assets are arch-specific; the other four are `noarch`.

```bash
sudo dnf install ./documentdb-0.116.0-1.noarch.rpm \
                 ./documentdb-18-0.116.0-1.noarch.rpm \
                 ./documentdb-common-0.116.0-1.noarch.rpm \
                 ./documentdb-postgresql-tools-0.116.0-1.noarch.rpm \
                 ./documentdb-gateway-0.116.0-1.el9.x86_64.rpm \
                 ./rhel9-postgresql18-documentdb-0.116.0-1.el9.x86_64.rpm
```

The `ubuntu24.04-` and `rhel9-` filename prefixes disambiguate release assets; they are not part of the package name.

### Extension only, from a single file

If the host already has PostgreSQL and the PGDG extension dependencies (`postgresql-N-cron`, `-pgvector`, `-postgis-3`), the extension installs from one file — no gateway, no `documentdb-setup`:

```bash
sudo apt install ./ubuntu24.04-postgresql-18-documentdb_0.116-0_amd64.deb
```

### Offline / air-gapped

Release assets alone are not enough — DocumentDB also needs PostgreSQL, `pg_cron`, `pgvector` and PostGIS from PGDG. Stage the full dependency closure on a connected machine of the **same distro, release and architecture**, serve it to the target as a local repository, then install with one command. Commands: [Offline / air-gapped install](https://documentdb.io/docs/getting-started/packages/#offline-air-gapped-install).

> Stage with `apt-cache depends --recurse` / `dnf download --alldeps`. `apt-get install --download-only` and a bare `dnf download --resolve` skip whatever is already installed on the staging machine; the bundle looks complete and the target dies with `Depends: adduser but it is not installable`.

## Set up and connect

Installing the packages puts files on disk. The setup wizard creates the PostgreSQL instance, installs the extensions, bootstraps the admin user and starts the gateway:

```bash
sudo documentdb-setup --admin-user admin
```

It prompts for the admin password; for servers and CI pass `--admin-password-file <file>` or `--admin-password-stdin` together with `--yes`. Then connect (`mongosh` is not shipped by these packages). A bare `-p` makes `mongosh` prompt, so pass the password inline in scripts — a non-interactive shell otherwise sends an empty one and fails with the unhelpful `MongoServerError: Invalid key`:

```bash
mongosh localhost:10260 -u admin -p '<PASSWORD>' --authenticationMechanism SCRAM-SHA-256 \
        --tls --tlsAllowInvalidCertificates --eval 'db.runCommand({ping: 1})'
```

> The gateway binds all interfaces (`0.0.0.0:10260`) by default, even though it is reached at `127.0.0.1` above. Firewall the port and supply a real certificate before exposing it to a network.

## What each release publishes

[`v0.116-0`](https://github.com/documentdb/documentdb/releases/tag/v0.116-0) (2026-08-20) — Linux packages only; no macOS or Windows installers. It also carries the unpublished `v0.115-0` changes. Fix list: [release notes](https://github.com/documentdb/documentdb/releases/tag/v0.116-0).

Since `v0.116-0` a release publishes a package set rather than a lone extension:

| Package | Role |
| --- | --- |
| `documentdb` (meta) + `documentdb-N` | Full stand-alone install. Pins PostgreSQL major N and its extension, and owns the systemd lifecycle. The meta package pins PostgreSQL 18. |
| `postgresql-N-documentdb` | The PostgreSQL extension for major N (files only). |
| `documentdb-gateway` | Wire-protocol runtime that serves the MongoDB-compatible endpoint. |
| `documentdb-postgresql-tools` | Administrator helpers: `documentdb-tune`, `documentdb-createcluster`, `documentdb-register-gateway`, `documentdb-gateway-admin`. |
| `documentdb-common` | Shared, PostgreSQL-agnostic payload: `documentdb-setup`, the systemd template units, helper scripts and sample data. |

First-party CI builds and tests Ubuntu 24.04 (DEB) and RHEL-compatible 9 (RPM), on PostgreSQL 17 and 18, for both architectures:

| Family | Architectures | Asset name |
| --- | --- | --- |
| DEB | amd64, arm64 | `ubuntu24.04-postgresql-18-documentdb_0.116-0_amd64.deb` |
| RPM | x86_64, aarch64 | `rhel9-postgresql18-documentdb-0.116.0-1.el9.x86_64.rpm` |

Note the two version grammars: on DEB the extension keeps `0.116-0` while every other package uses `0.116.0`; on RPM everything is `0.116.0-1`.

Everything else — PostgreSQL 15/16, Debian 11/12/13, Ubuntu 22.04, RHEL-compatible 8 — is not built by first-party CI or hosted by documentdb.io for this release. Starting with v0.116, packages from earlier releases are not carried forward to make those combinations appear current. This also withdraws the older PostgreSQL 16 extension packages previously served for Ubuntu 24.04 and RHEL-compatible 9.

Existing installations keep running, but receive no package updates and cannot reinstall those packages from documentdb.io. Empty signed metadata remains at retired repository URLs so package-manager refreshes do not break unrelated operations. Remove the repository configuration on a host that will not move to the current matrix:

```bash
# Debian / Ubuntu
sudo rm -f /etc/apt/sources.list.d/documentdb.list
sudo apt update

# RHEL-compatible
sudo rm -f /etc/yum.repos.d/documentdb.repo
sudo dnf clean all
```

Community builds for other targets are welcome. Check out the matching source tag and follow its [`packaging/` guide](https://github.com/documentdb/documentdb/blob/v0.116-0/packaging/README.md): `build_packages.sh` builds the extension, `gateway/build_gateway_packages.sh` builds the gateway, and `build_extra_packages.sh` builds the common, tools, stand-alone, and meta packages. PostgreSQL 15 remains extension-only because `documentdb-setup` requires PostgreSQL 16 or newer.

Every release also ships `SHA256SUMS` and `manifest.txt`:

```bash
gh release download v0.116-0 -R documentdb/documentdb -D pkgs && cd pkgs && sha256sum -c SHA256SUMS
```

## Container image

```bash
docker run -dt -p 10260:10260 --name documentdb-container \
  ghcr.io/documentdb/documentdb/documentdb-local:latest \
  --username <YOUR_USERNAME> --password <YOUR_PASSWORD>
```

Credentials must be set at create time or authentication will not work. Port `10260` avoids clashing with a local MongoDB; if you prefer `27017`, change both the `-p` flag and your connection string.

`v0.116-0` publishes these multi-architecture tags (linux/amd64 and linux/arm64):

- `ghcr.io/documentdb/documentdb/documentdb-local:pg15-0.116.0`
- `ghcr.io/documentdb/documentdb/documentdb-local:pg16-0.116.0`
- `ghcr.io/documentdb/documentdb/documentdb-local:pg17-0.116.0`
- `ghcr.io/documentdb/documentdb/documentdb-local:pg18-0.116.0`
- `ghcr.io/documentdb/documentdb/documentdb-local:latest` (currently aliases `pg17-0.116.0`)

Each image records the release it was built from in `/version.txt` and in its OCI labels:

```bash
docker run --rm --entrypoint cat ghcr.io/documentdb/documentdb/documentdb-local:pg18-0.116.0 /version.txt
```
