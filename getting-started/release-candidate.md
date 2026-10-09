---
title: Try the 1.0 Release Candidate
description: Try DocumentDB 1.0 RC2 from release assets or container images. For testing only, with no upgrade path and no fixes to RC2. No date for 1.0 yet.
---

# Try the 1.0 release candidate

[DocumentDB v1.0-RC2](https://github.com/documentdb/documentdb/releases/tag/v1.0-RC2) is a preview of 1.0 for testing. Try your applications and workloads against it and [report what breaks](#report-issues) before 1.0 ships.

There is no date for 1.0 yet. A release candidate is meant to be close to what 1.0 ships, apart from fixes found during testing; 1.0 follows once the release candidates stop turning up blocking issues. The upgrade and support policy from 1.0 onward will be published with the final release.

> **For testing only.** Don't use a release candidate for production or for data you need to keep.
>
> - **No upgrade path.** Upgrades from 0.117 to an RC, between release candidates, and from an RC to 1.0 are not supported. Start from an empty instance or volume, and expect to discard it.
> - **No fixes to RC2.** Fixes go into the next release candidate or 1.0, not into RC2.
> - **Not the default.** GitHub marks the RC as a pre-release. The package repository, the container `latest` tag and GitHub's latest release stay on the current stable release, v0.117-0. You only get the RC by asking for it explicitly, as shown below.

## Changes since RC1

- **Usage telemetry in the container image.** The RC2 image sends anonymous usage data by default; native packages send nothing. See [Container image](#container-image).
- **Gateway v2 runtime.** The gateway now runs on its v2 runtime by default.
- **Init script errors stop startup.** RC1's known issue is fixed: an error in an `--init-data-path` script now stops the container with an error. The run isn't retried on restart, so fix the script and start from a new volume.
- **Permission fixes.** `find` and `aggregate` with `batchSize: 0` now check that the caller can read the collection, and `$merge` and `$out` now need insert or update permission on the target collection, not just read.

The [changelog](https://github.com/documentdb/documentdb/blob/v1.0-RC2/CHANGELOG.md) lists every change. There is no upgrade from RC1: if you tested RC1, start RC2 on a fresh host or a new volume.

## Container image

The RC images are tagged `pg15-1.0-rc2`, `pg16-1.0-rc2`, `pg17-1.0-rc2` and `pg18-1.0-rc2`, for Linux amd64 and arm64. Include the container's `Release Version` log line when you [report issues](#report-issues). Use a new volume, not one a 0.117 or RC1 container has used:

```bash
docker run -dt -p 127.0.0.1:10260:10260 -v documentdb-rc2-data:/data --name docdb-rc2 \
  -e USERNAME="${DOCUMENTDB_USERNAME:?Set DOCUMENTDB_USERNAME first}" \
  -e PASSWORD="${DOCUMENTDB_PASSWORD:?Set DOCUMENTDB_PASSWORD first}" \
  ghcr.io/documentdb/documentdb/documentdb-local:pg17-1.0-rc2
```

Everything else in [DocumentDB Local](../documentdb-local/index.md) applies, with these differences:

- **Usage telemetry is on by default.** At startup and then hourly, the image reports its version, platform and CPU architecture to Scarf, and nothing about your data, queries or credentials. Turn it off with `-e DOCUMENTDB_USAGE_TELEMETRY=false`, `-e NO_ANALYTICS=1` or `-e DO_NOT_TRACK=1`, or pass `--disable-usage-telemetry` after the image name. See [Usage telemetry](../documentdb-local/index.md#usage-telemetry).
- **Health check.** The first health check runs about 30 seconds after start. Wait until `docker inspect -f '{{.State.Health.Status}}' docdb-rc2` reports `healthy`.
- **Stale `postmaster.pid`.** After `docker kill`, `docker rm -f` or a crash, the next start removes the stale file, as 0.117 did, but only when the volume's lock shows an RC2 container wrote it. Otherwise it refuses to start: re-create the container once with `-e DOCUMENTDB_FORCE_REMOVE_STALE_POSTMASTER_PID=true`, then without it. Don't share a volume between hosts. As in 0.117, a second container on a volume that is already in use exits immediately.
- **PostgreSQL loss stops the container.** If PostgreSQL inside the container stops, the gateway fails requests straight away and the container exits after 20 seconds (`DOCUMENTDB_POSTMASTER_EXIT_TIMEOUT`).
- **`--disable-extended-rum` is ignored**, apart from a deprecation warning.
- **`--toast-compression lz4|pglz|default`** is new and defaults to `lz4`.

To remove the test instance and its data:

```bash
docker stop docdb-rc2 && docker rm docdb-rc2 && docker volume rm documentdb-rc2-data
```

## Linux packages

The RC is not in the package repository. Use a clean, disposable Ubuntu 24.04 or RHEL-compatible 9 host that has never had DocumentDB installed and runs systemd (not WSL without systemd, a chroot or a plain container). Then run the installer with `--version`:

```sh
curl -fsSLo documentdb-install.sh https://documentdb.io/install.sh &&
sh documentdb-install.sh --version v1.0-RC2
```

It enables PGDG (plus EPEL and CRB on EL9), downloads this host's five RC2 packages from the [release](https://github.com/documentdb/documentdb/releases/tag/v1.0-RC2), verifies them against its `SHA256SUMS` and runs the setup wizard. It doesn't add the DocumentDB package repository, so the host never picks up a stable release by accident. PostgreSQL 18 is the default; add `--pg-major 17` for 17, or `--dry-run` to see the plan first. Without `--version`, the same installer sets up v0.117-0.

After an RC run, the installer refuses that host, even in stable mode or for a later RC, so an RC1 host can't take RC2. Use a fresh host for anything else.

To check what you got, run `dpkg-query -W 'documentdb*' '*-documentdb'` or `rpm -qa '*documentdb*'`. RC2 packages are version `1.0~rc2`, `documentdb-gateway --version` prints `1.0.0-rc2`, and the extension version is `1.0-0`.

### Manual installation

Enable PGDG first, but not the DocumentDB package repository. On Ubuntu 24.04:

```bash
sudo apt update &&
sudo apt install -y postgresql-common gnupg &&
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh -y
```

On RHEL-compatible 9, also enable EPEL and CRB. Enable CRB on Rocky Linux, AlmaLinux or CentOS Stream 9:

```bash
sudo dnf install -y dnf-plugins-core &&
sudo dnf config-manager --set-enabled crb
```

Or on RHEL 9:

```bash
sudo subscription-manager repos --enable "codeready-builder-for-rhel-9-$(arch)-rpms"
```

Then enable EPEL and PGDG:

```bash
sudo dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm &&
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-$(uname -m)/pgdg-redhat-repo-latest.noarch.rpm &&
sudo dnf -qy module disable postgresql
```

Download and verify the release files:

```bash
base=https://github.com/documentdb/documentdb/releases/download/v1.0-RC2
mkdir pkgs-rc2 && cd pkgs-rc2 &&
curl -fsSLO "$base/SHA256SUMS" &&
awk '{print $2}' SHA256SUMS | xargs -I{} curl -fsSLO "$base/{}" &&
sha256sum -c SHA256SUMS
```

Continue only if every file reports `OK`. The install commands below check again and stop on any mismatch.

Install the five packages for PostgreSQL 18. On Ubuntu 24.04:

```bash
arch=$(dpkg --print-architecture)
sha256sum -c --quiet SHA256SUMS &&
sudo apt install -y ./ubuntu24.04-documentdb-{18,common,postgresql-tools}_*_all.deb \
                    ./ubuntu24.04-documentdb-gateway_*_"$arch".deb \
                    ./ubuntu24.04-postgresql-18-documentdb_*_"$arch".deb
```

On RHEL-compatible 9:

```bash
sha256sum -c --quiet SHA256SUMS &&
sudo dnf install -y ./documentdb-{18,common,postgresql-tools}-[0-9]*.noarch.rpm \
                    ./documentdb-gateway-*.el9.$(uname -m).rpm \
                    ./rhel9-postgresql18-documentdb-*.el9.$(uname -m).rpm
```

For PostgreSQL 17, replace `18` with `17` in the first and last package names, and in `--pg-version` below.

The setup wizard binds the gateway on all interfaces on port 10260, with a self-signed TLS certificate. On a host reachable from other networks, block port 10260 with the host firewall first. Then run the wizard:

```bash
sudo documentdb-setup --pg-version 18 --use-new-postgres-instance --admin-user admin
```

The wizard now sets `default_toast_compression = 'lz4'`. To keep another setting, run the wizard directly and pass it through `sudo`, for example `sudo DOCUMENTDB_TOAST_COMPRESSION=pglz documentdb-setup ...`; `default` leaves PostgreSQL's own setting alone. The installer doesn't forward this variable.

To remove the RC, uninstall its packages and discard the host or its data directories. Don't reuse them for a stable installation.

## Known issues

- **`distinct` or `count` on a grouped view can crash the server.** `distinct` on a view whose pipeline ends in `$group`, or `count` with a query on such a view, crashes the PostgreSQL backend. PostgreSQL restarts and disconnects every other session.
- **An expensive `$replaceAll` can outlive `maxTimeMS`.** A very costly `$replaceAll` on a large string can keep running after `maxTimeMS` expires and can't be cancelled.
- **`updateOne` can miss a match.** With a filter on a field other than `_id`, `updateOne` can report no match when PostgreSQL runs the statement with a generic plan. This is rare with default settings. Setting `plan_cache_mode = force_custom_plan` in the PostgreSQL configuration avoids it.
- **The setup admin can't create or drop users.** The admin user created at setup can't run `createUser` or `dropUser` over the MongoDB protocol. On a package install, manage users on the host instead:

  ```bash
  sudo documentdb-gateway-admin create-user --username app --password-file app.pw \
    --roles '[{"role":"readAnyDatabase","db":"admin"}]'
  sudo documentdb-gateway-admin drop-user --username app
  ```

- **Container options.** `--log-level` and `--enable-telemetry` have no effect, and a `--data-path` that contains a space fails.
- **Collation is off by default.** `find`, `aggregate` and `count` with a `collation` fail as unsupported unless `documentdb_core.enableCollation` is on.

## Report issues

Open an issue in [documentdb/documentdb](https://github.com/documentdb/documentdb/issues) and include `v1.0-RC2`, your PostgreSQL major version, and either the package versions or the `Release Version` line from `docker logs docdb-rc2`, which names the build commit. The [release notes](https://github.com/documentdb/documentdb/releases/tag/v1.0-RC2) list what changed since 0.117.
