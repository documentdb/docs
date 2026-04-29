---
title: Pre-built Packages
description: Download and install DocumentDB using the pre-built Linux packages and container image published with each release.
---

# Pre-built Packages

Download and install DocumentDB using the pre-built packages and container image published with each release.

## Latest Release

The current release is [`v0.110-0`](https://github.com/documentdb/documentdb/releases/tag/v0.110-0), published on 2026-04-22.

This release publishes pre-built Linux packages for PostgreSQL 16, 17, and 18, plus the `documentdb-local` container image. See the [v0.110-0 release notes](../release-notes/v0.110-0.md) for the full list of features, performance improvements, and bug fixes.

> `v0.110-0` publishes Linux packages only. macOS and Windows installers are not part of this release.

## Package Matrix

| Package family | Supported distributions | PostgreSQL versions | Architectures | Downloads |
| --- | --- | --- | --- | --- |
| DEB | Debian 11, Debian 12, Debian 13, Ubuntu 22.04, Ubuntu 24.04 | 16, 17, 18 | amd64, arm64 | [v0.110-0 release assets](https://github.com/documentdb/documentdb/releases/tag/v0.110-0) |
| RPM | RHEL 8, RHEL 9 | 16, 17, 18 | x86_64, aarch64 | [v0.110-0 release assets](https://github.com/documentdb/documentdb/releases/tag/v0.110-0) |

Choose the asset whose filename matches your Linux distribution, PostgreSQL major version, and CPU architecture. For example:

- `ubuntu24.04-postgresql-17-documentdb_0.110-0_amd64.deb`
- `rhel9-postgresql17-documentdb-0.110.0-1.el9.x86_64.rpm`

## Install a DEB Package

```bash
curl -LO https://github.com/documentdb/documentdb/releases/download/v0.110-0/ubuntu24.04-postgresql-17-documentdb_0.110-0_amd64.deb
sudo apt install ./ubuntu24.04-postgresql-17-documentdb_0.110-0_amd64.deb
```

## Install an RPM Package

```bash
curl -LO https://github.com/documentdb/documentdb/releases/download/v0.110-0/rhel9-postgresql17-documentdb-0.110.0-1.el9.x86_64.rpm
sudo dnf install ./rhel9-postgresql17-documentdb-0.110.0-1.el9.x86_64.rpm
```

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

> **Note:** Replace `<YOUR_USERNAME>` and `<YOUR_PASSWORD>` with your desired credentials. If you omit them, `documentdb-local` uses its built-in defaults.
>
> **Port Note:** Port `10260` is used by default in these instructions to avoid conflicts with other local database services. You can use port `27017` (the standard MongoDB port) or any other available port if you prefer. If you do, be sure to update the port number in both your `docker run` command and your connection string accordingly.

### Available image tags for v0.110-0

In addition to the rolling `:latest` tag, `v0.110-0` publishes one image per
PostgreSQL major version. Each tag is a multi-arch manifest covering
`linux/amd64` and `linux/arm64`:

| Tag | PostgreSQL version |
| --- | --- |
| `ghcr.io/documentdb/documentdb/documentdb-local:pg15-0.110.0` | 15 |
| `ghcr.io/documentdb/documentdb/documentdb-local:pg16-0.110.0` | 16 |
| `ghcr.io/documentdb/documentdb/documentdb-local:pg17-0.110.0` | 17 |
| `ghcr.io/documentdb/documentdb/documentdb-local:latest` | Same manifest as `pg17-0.110.0` |

Pin to a `pgNN-0.110.0` tag when you need a deterministic image for
reproducible builds, CI, or production. Use `:latest` for the easiest
local-development experience.
