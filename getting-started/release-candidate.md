---
title: Try the 1.0 Release Candidate
description: Try DocumentDB 1.0 RC1 from release assets or container images. For testing only, with no upgrade path and no maintenance.
---

# Try the 1.0 release candidate

[DocumentDB v1.0-RC1](https://github.com/documentdb/documentdb/releases/tag/v1.0-RC1) is a preview of 1.0 for testing. Try your applications and workloads against it and [report what breaks](#report-issues) before 1.0 ships.

> **For testing only.** Don't use a release candidate for production or for data you need to keep.
>
> - **No upgrade path.** Upgrades from 0.117 to the RC, and from the RC to 1.0, are not supported. Start from an empty instance or volume, and expect to discard it.
> - **No maintenance.** The RC receives no fixes or security updates. Fixes ship in 1.0.
> - **Not the default.** The package repository and the container `latest` tag stay on the current stable release, v0.117-0. You only get the RC by asking for it explicitly, as shown below.

## Container image

The RC images are tagged `pg15-1.0.0`, `pg16-1.0.0`, `pg17-1.0.0` and `pg18-1.0.0`, for Linux amd64 and arm64. These tags hold RC1 despite the final version number, so the 1.0 release may replace them; include the image digest when you [report issues](#report-issues). Use a new volume, not one a 0.117 container has used:

```bash
docker run -dt -p 127.0.0.1:10260:10260 -v documentdb-rc1-data:/data --name docdb-rc1 \
  -e USERNAME="${DOCUMENTDB_USERNAME:?Set DOCUMENTDB_USERNAME first}" \
  -e PASSWORD="${DOCUMENTDB_PASSWORD:?Set DOCUMENTDB_PASSWORD first}" \
  ghcr.io/documentdb/documentdb/documentdb-local:pg17-1.0.0
```

Everything else in [DocumentDB Local](../documentdb-local/index.md) applies, with these differences:

- **Health check.** A new volume is usually ready within seconds. Wait until `docker inspect -f '{{.State.Health.Status}}' docdb-rc1` reports `healthy`.
- **One container per volume.** A second container on the same volume exits immediately. After `docker kill`, `docker rm -f` or a crash, the next container also refuses to start because of a stale `postmaster.pid`. Re-create it once with `-e DOCUMENTDB_FORCE_REMOVE_STALE_POSTMASTER_PID=true`, then without it. Run `docker stop` before `docker rm` to avoid this.
- **Init script errors don't stop startup.** Errors in `--init-data-path` scripts are logged, and the volume is still marked as initialized. Check `docker logs` after the first start.
- **`--disable-extended-rum` is ignored**, apart from a deprecation warning.
- **`--toast-compression lz4|pglz|default`** is new and defaults to `lz4`.

To remove the test instance and its data:

```bash
docker stop docdb-rc1 && docker rm docdb-rc1 && docker volume rm documentdb-rc1-data
```

## Linux packages

The RC is not in the package repository. Install it from its release assets on a clean, disposable Ubuntu 24.04 or RHEL-compatible 9 host that has never had DocumentDB installed. Enable PGDG first, plus EPEL and CRB on EL9; see [Pre-built Packages](prebuilt-packages.md).

```bash
gh release download v1.0-RC1 -R documentdb/documentdb -D pkgs-rc1 && cd pkgs-rc1 && sha256sum -c SHA256SUMS
```

Ubuntu 24.04, PostgreSQL 18, amd64:

```bash
sudo apt install ./ubuntu24.04-documentdb-18_1.0.0_all.deb \
                 ./ubuntu24.04-documentdb-common_1.0.0_all.deb \
                 ./ubuntu24.04-documentdb-postgresql-tools_1.0.0_all.deb \
                 ./ubuntu24.04-documentdb-gateway_1.0.0_amd64.deb \
                 ./ubuntu24.04-postgresql-18-documentdb_1.0-0_amd64.deb
```

RHEL-compatible 9, PostgreSQL 18, x86_64:

```bash
sudo dnf install ./documentdb-18-1.0.0-1.noarch.rpm \
                 ./documentdb-common-1.0.0-1.noarch.rpm \
                 ./documentdb-postgresql-tools-1.0.0-1.noarch.rpm \
                 ./documentdb-gateway-1.0.0-1.el9.x86_64.rpm \
                 ./rhel9-postgresql18-documentdb-1.0.0-1.el9.x86_64.rpm
```

For arm64, replace `amd64` with `arm64` or `x86_64` with `aarch64`. For PostgreSQL 17, use the `17` files instead of the two PostgreSQL-specific `18` files. Then run the [setup wizard](prebuilt-packages.md#2-run-the-setup-wizard) as usual.

The wizard now sets `default_toast_compression = 'lz4'`. To keep another setting, pass it through `sudo` when you run the wizard, for example `sudo DOCUMENTDB_TOAST_COMPRESSION=pglz documentdb-setup ...`; `default` leaves PostgreSQL's own setting alone.

The release also publishes `install.sh`, but it installs from the package repository, so it sets up v0.117-0, not the RC.

To remove the RC, uninstall its packages and discard the host or its data directories. Don't reuse them for a stable installation.

## Report issues

Open an issue in [documentdb/documentdb](https://github.com/documentdb/documentdb/issues) and include `v1.0-RC1`, your PostgreSQL major version, and either the package file names or the image digest from `docker inspect -f '{{.Image}}' docdb-rc1`. The [release notes](https://github.com/documentdb/documentdb/releases/tag/v1.0-RC1) list what changed since 0.117.
