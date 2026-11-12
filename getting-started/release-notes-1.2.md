---
title: DocumentDB 1.2 Release Notes
description: What's new in DocumentDB 1.2, where to get it, the container changes that can stop an existing setup from starting, and how to upgrade from 1.0.
---

# DocumentDB 1.2 release notes

[DocumentDB v1.2-0](https://github.com/documentdb/documentdb/releases/tag/v1.2-0) is the release after 1.0. It is built from the same commit as v1.2-RC4. There was no 1.1 release; its changes ship in 1.2. The [changelog](https://github.com/documentdb/documentdb/blob/v1.2-0/CHANGELOG.md) lists the engine changes.

## Where to get it

| Channel | 1.2 version | Platforms |
| --- | --- | --- |
| [Linux packages](prebuilt-packages.md) | packages `1.2.0`, extension `1.2-0` | Ubuntu 24.04 and RHEL-compatible 9, PostgreSQL 17 and 18, amd64 and arm64 |
| [Container image](../documentdb-local/index.md) | `pg15-1.2.0` to `pg18-1.2.0`; `latest` and `1.2` point to `pg17-1.2.0` | Linux amd64 and arm64 |

The SQL extension version is `1.2-0`, and `hello` reports it in `internal.documentdb_versions`.

## What's new since 1.0

- **Collation** now works with `distinct`, `findAndModify` and `update`, including per-update collation in multi-document updates and upserts. An explicit `simple` collation is treated as no collation. Values that would have to be truncated in a collated index are rejected on writes and indexed reads. Collation is still off by default; see [Collation](../postgres-api/configuration.md#collation).
- **Users and roles.** `dropUser` and `updateUser` reject user names of 64 bytes or more, so an overlong name can no longer drop or change the user whose name is its 63-byte prefix ([#734](https://github.com/documentdb/documentdb/issues/734)). `rolesInfo`, `usersInfo` and `connectionStatus` return custom-role metadata; admin and root users see every custom role, other users the roles they inherit.
- **`$dateTrunc` with `unit: "day"`** uses PostgreSQL's `date_bin`, so bins are exact calendar days aligned to the reference date and respect time zone boundaries. Strides and results outside PostgreSQL's range are rejected. Set `documentdb.enable_date_trunc_day_with_pg_bin` to `off` for the old behavior.
- **Unique indexes.** Descending unique indexes now enforce null and missing conflicts, including on compound keys and collated indexes (`documentdb.enable_unique_null_missing_semantics`, on by default).
- **`collMod`** can rename an index when its key pattern and current name both match.
- **`bson_build_document`** accepts `timestamptz` values and writes them as BSON dates with millisecond precision.
- **Query planning.** `$elemMatch` predicates and the inner side of a `$lookup` are estimated from collected statistics, and dynamic cursor queries without planner statistics skip parallel plans. Field pruning before `$unwind` and order-by pushdown for `$group` over multi-key composite indexes are on by default.
- **Fixes** for crashes and wrong results when a correlated `$lookup` rescans a `$densify` stage, wrong sort and group results when a null equality falls between compound-index sort keys, and RUM leaf splits that could overflow with large index terms.
- **Retired settings.** The `enableSkipDottedFieldIndexTerms` and `enable_ordered_operator_scans` settings are removed; their behavior is now always on.

## Container image changes

Two of these can stop an existing `docker run` command or Compose file from starting:

- **A password is required.** Without `PASSWORD` or `--password` the container exits at startup instead of creating the admin user with the public default. `DOCUMENTDB_ALLOW_DEFAULT_PASSWORD=true` restores that default for short-lived local evaluation. With `--create-user false` a password is needed only to load data. On an existing volume the admin user keeps the password it was created with.
- **`TLS_MODE` takes one value.** An empty `TLS_MODE`, or a list such as `allowTLS,requireTLS`, is rejected.

Also in the image:

- `--password` no longer stays in the container's process list. Both `--password` and `PASSWORD` are still visible to `docker inspect`.
- `--log-level` sets the gateway's log level; `quiet` still shows errors.
- A newer image updates the extensions on an existing data volume at start.

## Upgrading

- **Packages:** 1.0 installations upgrade in place: upgrade the packages, then run `sudo documentdb-setup --pg-version <major> --yes`, which restarts PostgreSQL once and updates the extensions, keeping the admin users. See [Upgrading](https://documentdb.io/docs/linux-packages#upgrading).
- **Containers:** start the 1.2 image on the 1.0 volume; it updates the extensions at start. Going back to 1.0 on that volume isn't supported. See [Upgrading the image](../documentdb-local/index.md#upgrading-the-image).
- **Release candidates:** hosts and volumes that ran a release candidate have no supported upgrade path.

## Report issues

Open an issue in [documentdb/documentdb](https://github.com/documentdb/documentdb/issues) with your DocumentDB and PostgreSQL versions, and either the package versions or the `Release Version` line from `docker logs`.
