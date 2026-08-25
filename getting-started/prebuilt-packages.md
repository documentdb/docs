---
title: Pre-built Packages
description: Download and install DocumentDB using the pre-built Linux packages and container image published with each release.
---

# Pre-built Packages

Download and install DocumentDB using the pre-built packages and container image published with each release.

## Latest Release

The current release is [`v0.116-0`](https://github.com/documentdb/documentdb/releases/tag/v0.116-0), published on 2026-08-20. It also carries the changes prepared for `v0.115-0`, which was never published.

This release introduces a redesigned Linux packaging layout (see below), fixes a gateway connection-pool bug that evicted pools while they were still in use, fixes a backend crash on insert into a sharded collection from a pooled session on PostgreSQL 18.6 or newer, defaults TOAST compression to `lz4`, and stamps build provenance into the `documentdb-local` image.

> `v0.116-0` publishes Linux packages only. macOS and Windows installers are not part of this release.

## What ships in a release

Before `v0.116-0` a release published a single package per PostgreSQL major — the extension. It now publishes a set:

| Package | Role |
| --- | --- |
| `documentdb` (meta) + `documentdb-N` | Full stand-alone install. Pins PostgreSQL major N and its extension, and owns the systemd lifecycle. The meta package pins PostgreSQL 18. |
| `postgresql-N-documentdb` | The PostgreSQL extension for major N (files only). |
| `documentdb-gateway` | Wire-protocol runtime that serves the MongoDB-compatible endpoint. |
| `documentdb-postgresql-tools` | Administrator helpers: `documentdb-tune`, `documentdb-createcluster`, `documentdb-register-gateway`, `documentdb-gateway-admin`. |
| `documentdb-common` | Shared, PostgreSQL-agnostic payload: `documentdb-setup`, the systemd template units, helper scripts and sample data. |

Two version grammars are in play by design: on DEB the extension keeps the control-file form `0.116-0` while every other package uses `0.116.0`; on RPM everything is `0.116.0` (the extension's `-0` becomes the RPM release field, giving `0.116.0-1`).

## Package Matrix

`v0.116-0` builds and tests a first-party matrix of Ubuntu 24.04 and RHEL-compatible 9:

| Package family | Distributions | PostgreSQL versions | Architectures | Downloads |
| --- | --- | --- | --- | --- |
| DEB | Ubuntu 24.04 | 17, 18 | amd64, arm64 | [v0.116-0 release assets](https://github.com/documentdb/documentdb/releases/tag/v0.116-0) |
| RPM | RHEL-compatible 9 | 17, 18 | x86_64, aarch64 | [v0.116-0 release assets](https://github.com/documentdb/documentdb/releases/tag/v0.116-0) |

Other combinations — PostgreSQL 15/16, Debian 11/12/13, Ubuntu 22.04, RHEL-compatible 8 — are not built by first-party CI for this release. Build them on demand from the tag with the scripts in [`packaging/`](https://github.com/documentdb/documentdb/blob/main/packaging/README.md), or install the newest release that did build them from the package repository (see below). PostgreSQL 15 is extension-only: `documentdb-setup` and `documentdb-register-gateway` require PostgreSQL 16 or newer.

Choose the asset whose filename matches your distribution, PostgreSQL major version, and CPU architecture. For example:

- `ubuntu24.04-documentdb_0.116.0_all.deb`
- `ubuntu24.04-postgresql-18-documentdb_0.116-0_amd64.deb`
- `rhel9-postgresql18-documentdb-0.116.0-1.el9.x86_64.rpm`

Every release also ships `SHA256SUMS` and `manifest.txt`. Verify what you downloaded:

```bash
gh release download v0.116-0 -R documentdb/documentdb -D pkgs && cd pkgs && sha256sum -c SHA256SUMS
```

## Repository-backed install (recommended)

Installing from <https://documentdb.io> is easier than downloading assets, because the package manager resolves the dependencies between the packages for you. See [Package Installation](https://documentdb.io/packages) for the exact command for your distribution.

The repository also keeps the most recent package for distributions a given release did not build, so Ubuntu 22.04, Debian 11/12/13 and RHEL-compatible 8 continue to resolve the extension package from an earlier release.

## Install from downloaded assets

Needs the PostgreSQL upstream (PGDG) repository enabled first; RHEL-compatible hosts also need EPEL and CRB — see [Package Installation](https://documentdb.io/packages). If the host cannot reach PGDG either, see [Offline / air-gapped](#offline--air-gapped).

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

For arm64 swap `x86_64` → `aarch64`; for PostgreSQL 17 swap `18` → `17`. As with DEB, only the gateway and extension assets are architecture-specific; the other four are `noarch`.

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

If the target already has PostgreSQL and the PGDG extension dependencies (`postgresql-N-cron`, `-pgvector`, `-postgis-3`), the extension installs from one file — no gateway, no `documentdb-setup`:

```bash
sudo apt install ./ubuntu24.04-postgresql-18-documentdb_0.116-0_amd64.deb
```

## Offline / air-gapped

Release assets alone are not enough — DocumentDB also needs PostgreSQL, `pg_cron`, `pgvector` and PostGIS from PGDG. Stage the full dependency closure on a connected machine of the **same distro, release and architecture**, serve it to the target as a local repository, then install with one command. Commands: [Offline / air-gapped install](https://documentdb.io/docs/getting-started/packages/#offline-air-gapped-install).

> Stage with `apt-cache depends --recurse` / `dnf download --alldeps`. `apt-get install --download-only` and a bare `dnf download --resolve` skip whatever is already installed on the staging machine; the bundle looks complete and the target dies with `Depends: adduser but it is not installable`.

## Set up and connect

Installing the packages puts files on disk. The setup wizard creates the PostgreSQL instance, installs the extensions, bootstraps the admin user and starts the gateway:

```bash
sudo documentdb-setup --admin-user admin
```

It prompts for the admin password; for servers and CI pass `--admin-password-file <file>` or `--admin-password-stdin` together with `--yes`. Then connect (`mongosh` is not shipped by these packages):

```bash
mongosh 'mongodb://admin:<PASSWORD>@127.0.0.1:10260/mydb?tls=true&tlsAllowInvalidCertificates=true' \
        --eval 'db.runCommand({ping: 1})'
```

> The gateway binds all interfaces (`0.0.0.0:10260`) by default, even though the connect string above says `127.0.0.1`. Firewall the port and supply a real certificate before exposing it to a network.

## Docker Images

Use the official `documentdb-local` image from GHCR:

```bash
# Pull the published image
docker pull ghcr.io/documentdb/documentdb/documentdb-local:latest

# Tag the image for convenience
docker tag ghcr.io/documentdb/documentdb/documentdb-local:latest documentdb

# Run the container with your chosen username and password
docker run -dt -p 10260:10260 --name documentdb-container ghcr.io/documentdb/documentdb/documentdb-local:latest --username <YOUR_USERNAME> --password <YOUR_PASSWORD>
```

> **Note:** Replace `<YOUR_USERNAME>` and `<YOUR_PASSWORD>` with your desired credentials. You must set these when creating the container for authentication to work.
>
> **Port Note:** Port `10260` is used by default in these instructions to avoid conflicts with other local database services. You can use port `27017` (the standard MongoDB port) or any other available port if you prefer. If you do, be sure to update the port number in both your `docker run` command and your connection string accordingly.

`v0.116-0` publishes the following multi-architecture tags (linux/amd64 and linux/arm64) to GHCR:

- `ghcr.io/documentdb/documentdb/documentdb-local:pg15-0.116.0`
- `ghcr.io/documentdb/documentdb/documentdb-local:pg16-0.116.0`
- `ghcr.io/documentdb/documentdb/documentdb-local:pg17-0.116.0`
- `ghcr.io/documentdb/documentdb/documentdb-local:pg18-0.116.0`
- `ghcr.io/documentdb/documentdb/documentdb-local:latest` (currently aliases `pg17-0.116.0`)

Each image records the release it was built from in `/version.txt` and in its OCI labels:

```bash
docker run --rm --entrypoint cat ghcr.io/documentdb/documentdb/documentdb-local:pg18-0.116.0 /version.txt
```
