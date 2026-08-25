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

`apt` and `dnf` resolve dependencies only from repository indexes, so a dependency on a bare local file is unresolvable and **installing the meta package on its own fails**:

```text
documentdb : Depends: documentdb-18 (>= 0.116.0) but it is not installable
```

This is not a defect in the packages — the same thing happens to any local `.deb` or `.rpm` whose dependencies are not in an enabled repository. Pass the whole set for your platform to one command.

This still assumes the host can reach PGDG for PostgreSQL itself. If it cannot, see [Offline / air-gapped install](#offline--air-gapped-install), which stages the whole dependency closure and restores single-command installs.

### DEB (Ubuntu 24.04, PostgreSQL 18, amd64)

For arm64 swap `amd64` → `arm64`; for PostgreSQL 17 swap `18` → `17`. Only the gateway and extension assets carry an architecture — the other four are `_all.deb` and are the same file on both.

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

Both families need the PostgreSQL upstream (PGDG) repository enabled first, and RHEL-compatible hosts additionally need EPEL and CRB. See [Package Installation](https://documentdb.io/packages) for those prerequisites.

### Extension only, from a single file

If the target already has PostgreSQL and the PGDG extension dependencies (`postgresql-N-cron`, `-pgvector`, `-postgis-3`), the extension package installs from one file, with no gateway and no `documentdb-setup`:

```bash
sudo apt install ./ubuntu24.04-postgresql-18-documentdb_0.116-0_amd64.deb
```

## Offline / air-gapped install

An air-gapped host needs more than the DocumentDB assets: the packages depend on PostgreSQL itself and on `pg_cron`, `pgvector` and PostGIS from PGDG, none of which are on the release page. Stage the whole dependency closure on a connected machine first.

Run the staging step on a machine with the **same distribution, release and architecture** as the target — the closure is specific to all three.

### Stage the bundle (connected machine)

Configure the repositories exactly as for an online install, then download the closure and index it:

```bash
# Debian / Ubuntu
mapfile -t PKGS < <(apt-cache depends --recurse --no-recommends --no-suggests \
    --no-conflicts --no-breaks --no-replaces --no-enhances documentdb-18 \
  | grep '^[a-zA-Z0-9]' | sort -u)
mkdir -p bundle && cd bundle
apt-get download "${PKGS[@]}"
dpkg-scanpackages . /dev/null > Packages && gzip -k Packages
```

```bash
# RHEL-compatible
sudo dnf install -y dnf-plugins-core createrepo_c
mkdir -p bundle
sudo dnf download --resolve --alldeps --destdir bundle documentdb-18
createrepo_c bundle
```

> **Use the full-closure flags, not `--download-only`.** `apt-get install --download-only` and a bare `dnf download --resolve` skip anything already installed on the staging machine. The bundle then looks complete but fails on a clean target with errors like `Depends: adduser but it is not installable`. `apt-cache depends --recurse` and `dnf download --alldeps` ignore local install state, which is what you want.

Expect roughly 200 packages / 200 MB for the DEB closure and 270 packages / 170 MB for the RPM closure, dominated by PostGIS and its GDAL dependencies.

Two warnings from the DEB staging step are expected and harmless: `Download is performed unsandboxed as root`, and a long `dpkg-scanpackages: warning: Packages in archive but missing from override file` list, which is just an artifact of passing `/dev/null` as the override file.

### Install from the bundle (air-gapped target)

Copy `bundle/` across, point the package manager at it, and install with one command — the local index gives you full dependency resolution, so you do not have to name the packages individually or order them correctly:

```bash
# Debian / Ubuntu
echo "deb [trusted=yes] file:/path/to/bundle ./" \
  | sudo tee /etc/apt/sources.list.d/documentdb-offline.list
sudo apt-get update
sudo apt install documentdb-18
```

```bash
# RHEL-compatible
printf '%s\n' '[documentdb-offline]' 'name=DocumentDB offline bundle' \
  'baseurl=file:///path/to/bundle' 'enabled=1' 'gpgcheck=0' \
  | sudo tee /etc/yum.repos.d/documentdb-offline.repo
sudo dnf install documentdb-18
```

`[trusted=yes]` and `gpgcheck=0` tell the package manager to accept the local directory, which has no repository signature of its own. The upstream signatures were already verified when the bundle was staged; verify the transfer itself with `sha256sum` if the bundle crosses an untrusted boundary.

Then continue with [Set up and connect](#set-up-and-connect) as normal — `documentdb-setup` needs no network.

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
