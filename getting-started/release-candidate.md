---
title: Try the 1.0 Release Candidate
description: Try DocumentDB 1.0 RC1 from release assets or container images. For testing only, with no upgrade path and no fixes to RC1. No date for 1.0 yet.
---

# Try the 1.0 release candidate

[DocumentDB v1.0-RC1](https://github.com/documentdb/documentdb/releases/tag/v1.0-RC1) is a preview of 1.0 for testing. Try your applications and workloads against it and [report what breaks](#report-issues) before 1.0 ships.

There is no date for 1.0 yet. A release candidate is meant to be close to what 1.0 ships, apart from fixes found during testing; 1.0 follows once the release candidates stop turning up blocking issues. The upgrade and support policy from 1.0 onward will be published with the final release.

> **For testing only.** Don't use a release candidate for production or for data you need to keep.
>
> - **No upgrade path.** Upgrades from 0.117 to an RC, between release candidates, and from an RC to 1.0 are not supported. Start from an empty instance or volume, and expect to discard it.
> - **No fixes to RC1.** Fixes go into the next release candidate or 1.0, not into RC1. Expect an RC2.
> - **Not the default.** GitHub marks the RC as a pre-release. The package repository, the container `latest` tag and GitHub's latest release stay on the current stable release, v0.117-0. You only get the RC by asking for it explicitly, as shown below.

## Container image

The RC images are tagged `pg15-1.0-rc1`, `pg16-1.0-rc1`, `pg17-1.0-rc1` and `pg18-1.0-rc1`, for Linux amd64 and arm64. Don't use the older `pgNN-1.0.0-rc1` and `pgNN-1.0.0` tags: they hold an earlier RC1 build, and `pgNN-1.0.0` will move to the final 1.0 build. Include the container's `Release Version` log line when you [report issues](#report-issues). Use a new volume, not one a 0.117 container has used:

```bash
docker run -dt -p 127.0.0.1:10260:10260 -v documentdb-rc1-data:/data --name docdb-rc1 \
  -e USERNAME="${DOCUMENTDB_USERNAME:?Set DOCUMENTDB_USERNAME first}" \
  -e PASSWORD="${DOCUMENTDB_PASSWORD:?Set DOCUMENTDB_PASSWORD first}" \
  ghcr.io/documentdb/documentdb/documentdb-local:pg17-1.0-rc1
```

Everything else in [DocumentDB Local](../documentdb-local/index.md) applies, with these differences:

- **Health check.** The first health check runs about 30 seconds after start. Wait until `docker inspect -f '{{.State.Health.Status}}' docdb-rc1` reports `healthy`.
- **A stale `postmaster.pid` refuses to start.** After `docker kill`, `docker rm -f` or a crash, the next container on that volume refuses to start; 0.117 removed the stale file automatically. Re-create the container once with `-e DOCUMENTDB_FORCE_REMOVE_STALE_POSTMASTER_PID=true`, then without it. Run `docker stop` before `docker rm` to avoid this. As in 0.117, a second container on a volume that is already in use exits immediately.
- **Known issue: init script errors don't stop startup.** In RC1, an error in an `--init-data-path` script is logged but the container still reports success and marks the volume as initialized. Check `docker logs` after the first start; after fixing the script, start from a new volume. Fixed in the next release candidate.
- **`--disable-extended-rum` is ignored**, apart from a deprecation warning.
- **`--toast-compression lz4|pglz|default`** is new and defaults to `lz4`.

To remove the test instance and its data:

```bash
docker stop docdb-rc1 && docker rm docdb-rc1 && docker volume rm documentdb-rc1-data
```

## Linux packages

The RC is not in the package repository. Use a clean, disposable Ubuntu 24.04 or RHEL-compatible 9 host that has never had DocumentDB installed and runs systemd (not WSL without systemd, a chroot or a plain container). Then run the installer with `--version`:

```sh
curl -fsSLo documentdb-install.sh https://documentdb.io/install.sh &&
sh documentdb-install.sh --version v1.0-RC1
```

It enables PGDG (plus EPEL and CRB on EL9), downloads this host's five RC1 packages from the [release](https://github.com/documentdb/documentdb/releases/tag/v1.0-RC1), verifies them against its `SHA256SUMS` and runs the setup wizard. It doesn't add the DocumentDB package repository, so the host never picks up a stable release by accident. PostgreSQL 18 is the default; add `--pg-major 17` for 17, or `--dry-run` to see the plan first. Without `--version`, the same installer sets up v0.117-0.

After an RC run, the installer refuses that host, even in stable mode. Use a fresh host for anything else.

To check what you got, run `dpkg-query -W 'documentdb*' '*-documentdb'` or `rpm -qa '*documentdb*'`. RC1 packages are version `1.0~rc1`, `documentdb-gateway --version` prints `1.0.0-rc1`, and the extension version is `1.0-0`.

### Manual installation

Enable PGDG first, but not the DocumentDB package repository. On Ubuntu 24.04:

```bash
sudo apt install -y postgresql-common
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh -y
```

On RHEL-compatible 9, also enable EPEL and CRB. On RHEL itself, enable CRB with `sudo subscription-manager repos --enable codeready-builder-for-rhel-9-$(arch)-rpms` instead of `dnf config-manager`.

```bash
sudo dnf install -y dnf-plugins-core https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
sudo dnf config-manager --set-enabled crb
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-$(uname -m)/pgdg-redhat-repo-latest.noarch.rpm
sudo dnf -qy module disable postgresql
```

Download and verify the release files:

```bash
base=https://github.com/documentdb/documentdb/releases/download/v1.0-RC1
mkdir pkgs-rc1 && cd pkgs-rc1
curl -fsSLO "$base/SHA256SUMS"
awk '{print $2}' SHA256SUMS | while read -r f; do curl -fsSLO "$base/$f"; done
sha256sum -c SHA256SUMS
```

Install the five packages for PostgreSQL 18. On Ubuntu 24.04:

```bash
arch=$(dpkg --print-architecture)
sudo apt install -y ./ubuntu24.04-documentdb-{18,common,postgresql-tools}_*_all.deb \
                    ./ubuntu24.04-documentdb-gateway_*_"$arch".deb \
                    ./ubuntu24.04-postgresql-18-documentdb_*_"$arch".deb
```

On RHEL-compatible 9:

```bash
sudo dnf install -y ./documentdb-{18,common,postgresql-tools}-[0-9]*.noarch.rpm \
                    ./documentdb-gateway-*.el9.$(uname -m).rpm \
                    ./rhel9-postgresql18-documentdb-*.el9.$(uname -m).rpm
```

For PostgreSQL 17, replace `18` with `17` in the first and last package names, and in `--pg-version` below. Then run the setup wizard:

```bash
sudo documentdb-setup --pg-version 18 --use-new-postgres-instance --admin-user admin
```

The wizard now sets `default_toast_compression = 'lz4'`. To keep another setting, run the wizard directly and pass it through `sudo`, for example `sudo DOCUMENTDB_TOAST_COMPRESSION=pglz documentdb-setup ...`; `default` leaves PostgreSQL's own setting alone. The installer doesn't forward this variable.

To remove the RC, uninstall its packages and discard the host or its data directories. Don't reuse them for a stable installation.

## Report issues

Open an issue in [documentdb/documentdb](https://github.com/documentdb/documentdb/issues) and include `v1.0-RC1`, your PostgreSQL major version, and either the package versions or the `Release Version` line from `docker logs docdb-rc1`, which names the build commit. The [release notes](https://github.com/documentdb/documentdb/releases/tag/v1.0-RC1) list what changed since 0.117.
