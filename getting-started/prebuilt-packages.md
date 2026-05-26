---
title: Pre-built Packages
description: Download and install DocumentDB using the pre-built Linux packages and container image published with each release.
---

# Pre-built Packages

Download and install DocumentDB using the pre-built packages and container image published with each release.

## Latest Release

The current release is [`v0.112-0`](https://github.com/documentdb/documentdb/releases/tag/v0.112-0), published on 2026-05-26.

This release eliminates subquery migration for many `$group` aggregations, adds index pushdown for `$group` when `_id` is a multi-field document expression, expands collation support for non-unique ordered indexes (`$lt`/`$lte`/`$ne`/`$not` combinations), migrates streaming query execution off the SPI cursor path, and ships several crash fixes for `BSONFIRSTN`/`BSONLASTN`/`BSONMAXN`/`BSONMINN` on empty sharded collections.

> `v0.112-0` publishes Linux packages only. macOS and Windows installers are not part of this release.

## Package Matrix

| Package family | Supported distributions | PostgreSQL versions | Architectures | Downloads |
| --- | --- | --- | --- | --- |
| DEB | Debian 11, Debian 12, Debian 13, Ubuntu 22.04, Ubuntu 24.04 | 16, 17, 18 | amd64, arm64 | [v0.112-0 release assets](https://github.com/documentdb/documentdb/releases/tag/v0.112-0) |
| RPM | RHEL 8, RHEL 9 | 16, 17, 18 | x86_64, aarch64 | [v0.112-0 release assets](https://github.com/documentdb/documentdb/releases/tag/v0.112-0) |

Choose the asset whose filename matches your Linux distribution, PostgreSQL major version, and CPU architecture. For example:

- `ubuntu24.04-postgresql-17-documentdb_0.112-0_amd64.deb`
- `rhel9-postgresql17-documentdb-0.112.0-1.el9.x86_64.rpm`

## Install a DEB Package

```bash
curl -LO https://github.com/documentdb/documentdb/releases/download/v0.112-0/ubuntu24.04-postgresql-17-documentdb_0.112-0_amd64.deb
sudo apt install ./ubuntu24.04-postgresql-17-documentdb_0.112-0_amd64.deb
```

## Install an RPM Package

```bash
curl -LO https://github.com/documentdb/documentdb/releases/download/v0.112-0/rhel9-postgresql17-documentdb-0.112.0-1.el9.x86_64.rpm
sudo dnf install ./rhel9-postgresql17-documentdb-0.112.0-1.el9.x86_64.rpm
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

> **Note:** Replace `<YOUR_USERNAME>` and `<YOUR_PASSWORD>` with your desired credentials. You must set these when creating the container for authentication to work.
>
> **Port Note:** Port `10260` is used by default in these instructions to avoid conflicts with other local database services. You can use port `27017` (the standard MongoDB port) or any other available port if you prefer. If you do, be sure to update the port number in both your `docker run` command and your connection string accordingly.

`v0.112-0` publishes the following multi-architecture tags (linux/amd64 and linux/arm64) to GHCR:

- `ghcr.io/documentdb/documentdb/documentdb-local:pg15-0.112.0`
- `ghcr.io/documentdb/documentdb/documentdb-local:pg16-0.112.0`
- `ghcr.io/documentdb/documentdb/documentdb-local:pg17-0.112.0`
- `ghcr.io/documentdb/documentdb/documentdb-local:latest` (currently aliases `pg17-0.112.0`)
