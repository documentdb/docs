---
title: DocumentDB 1.0 Release Notes
description: What's new in DocumentDB 1.0, where to get it, how to upgrade from 0.117, and the known issues that ship with it.
---

# DocumentDB 1.0 release notes

[DocumentDB v1.0-0](https://github.com/documentdb/documentdb/releases/tag/v1.0-0) was released on **2026-10-19**. It is built from the same commit as the second release candidate, v1.0-RC2. The [changelog](https://github.com/documentdb/documentdb/blob/v1.0-0/CHANGELOG.md) lists every change.

## Where to get it

| Channel | 1.0 version | Platforms |
| --- | --- | --- |
| [Linux packages](prebuilt-packages.md) | packages `1.0.0`, extension `1.0-0` | Ubuntu 24.04 and RHEL-compatible 9, PostgreSQL 17 and 18, amd64 and arm64 |
| [Container image](../documentdb-local/index.md) | `pg15-1.0.0` to `pg18-1.0.0`; `latest` is `pg17-1.0.0` | Linux amd64 and arm64 |
| [Kubernetes operator](../kubernetes-operator/index.md) | operator `1.0.0`, which runs engine `1.0-0` | Kubernetes 1.35+, or 1.33-1.34 with the `ImageVolume` feature gate |

The SQL extension version is `1.0-0` everywhere, and `hello` reports it in `internal.documentdb_versions`.

## What's new since 0.117

- **Usage telemetry in the container image.** The `documentdb-local` image reports its version, platform and CPU architecture to Scarf at startup and hourly, on by default. Native packages send nothing. See [Usage telemetry](../documentdb-local/index.md#usage-telemetry) to turn it off.
- **Security fixes.** `find` and `aggregate` with `batchSize: 0` now check that the caller can read the collection. `$merge` and `$out` now need insert or update permission on the target collection, not just read. `createUser` and `updateUser` reject passwords that fail SASLprep.
- **Collation** now works with `count` and the `$stdDevPop`, `$stdDevSamp`, `$median` and `$percentile` accumulators, and applies to the symbol type. Collation is still off by default; see [Known issues](#known-issues).
- **Custom roles.** With `documentdb.enableRoleCrud` on (off by default), `createRole` and `dropRole` keep custom roles in a roles catalog. `updateRole` is still unsupported.
- **`compact`** reports `bytesFreed` as the measured change in on-disk size.
- **Gateway.** The gateway runs on its new v2 runtime by default. It now returns retryable or specific error codes for backend connection and pool failures instead of `InternalError`, and exits with an error instead of panicking when PostgreSQL is unreachable at startup.
- **Crash and correctness fixes**, including `$getField`/`$setField`/`$unsetField` on system variables, `$firstN`/`$lastN` without a sort, `$documents` inside `$facet`, `find`/`count`/`distinct` on views that end in `$group`, `$near`/`$geoNear` ordering on `2dsphere` indexes, overlapping `dropIndexes` deadlocks, and several RUM index scan and vacuum fixes.
- **Container behavior.** An error in an `--init-data-path` script now stops the container. A stale `postmaster.pid` left by an unclean stop is removed on the next start, and the container exits if PostgreSQL inside it stops. TOAST compression defaults to `lz4`, and `--disable-extended-rum` is deprecated and ignored. See [Container lifecycle](../documentdb-local/index.md#container-lifecycle).
- **Packages.** The setup wizard sets `default_toast_compression = 'lz4'`; pass `DOCUMENTDB_TOAST_COMPRESSION=pglz` or `default` through `sudo` to keep another setting.

## Upgrading

- **Packages:** 0.116 and 0.117 installations can be upgraded in place: upgrade the packages, then update the extensions. See [Upgrading to 1.0](prebuilt-packages.md#upgrading-to-10).
- **Containers:** a 1.0 image on an existing volume doesn't update the extensions. Move the data to a new volume with `mongodump` and `mongorestore`. See [Upgrading the image](../documentdb-local/index.md#upgrading-the-image).
- **Release candidates:** hosts and volumes that ran an RC can't be upgraded. Start 1.0 on a fresh host or a new volume.

## Known issues

- **The setup admin can't create or drop users** ([#752](https://github.com/documentdb/documentdb/issues/752)). `createUser` from the admin user created at setup fails with `User is not authorized to perform this action`. On a package install, manage users on the host instead:

  ```bash
  printf 'Password for app: '; stty -echo; read -r APP_PASSWORD; stty echo; echo
  printf '%s' "$APP_PASSWORD" | sudo documentdb-gateway-admin create-user --username app \
    --password-stdin --roles '[{"role":"readAnyDatabase","db":"admin"}]'
  sudo documentdb-gateway-admin drop-user --username app
  ```

- **A newer container image doesn't upgrade an existing volume** ([#773](https://github.com/documentdb/documentdb/issues/773)). The image never runs `ALTER EXTENSION ... UPDATE`, and a changed `--password` on an existing volume is ignored.
- **`updateOne` with `let` can miss a match.** An update with `multi: false` that filters on fields other than `_id` through `let` variables can report no match when PostgreSQL runs it with a generic plan. This is rare with default settings; setting `plan_cache_mode = force_custom_plan` in the PostgreSQL configuration avoids it.
- **An expensive `$replaceAll` can outlive `maxTimeMS`.** A very costly `$replaceAll` on a large string can keep running after `maxTimeMS` expires and can't be cancelled.
- **Collation is off by default.** `find`, `aggregate`, `count` and `delete` with a `collation` fail as unsupported unless `documentdb_core.enableCollation` is on. `distinct`, `findAndModify` and `update` don't support `collation` at all.
- **Container options that do nothing.** `--log-level` and `--enable-telemetry` are validated but have no effect, and a `--data-path` that contains a space fails.
- **Open packaging and container issues:**
  - [#771](https://github.com/documentdb/documentdb/issues/771): package remove, purge, upgrade and rerun paths that can leave the stack down or stuck.
  - [#759](https://github.com/documentdb/documentdb/issues/759): container init, runtime and quick-start gaps.
  - [#758](https://github.com/documentdb/documentdb/issues/758): container settings that aren't secure by default.
  - [#735](https://github.com/documentdb/documentdb/issues/735): gateway state ownership isn't preserved across an RPM erase and reinstall.
  - [#685](https://github.com/documentdb/documentdb/issues/685): a `--password` argument is readable in the container's process list; pass `-e PASSWORD` instead.
  - [#389](https://github.com/documentdb/documentdb/issues/389): the Debian packages lack a copyright file.

## Report issues

Open an issue in [documentdb/documentdb](https://github.com/documentdb/documentdb/issues) with your DocumentDB and PostgreSQL versions, and either the package versions or the `Release Version` line from `docker logs`.
