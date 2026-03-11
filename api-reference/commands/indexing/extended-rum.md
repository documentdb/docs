---
title: Extended RUM
description: Extended RUM is the default index access method for DocumentDB on PostgreSQL 18 and later, providing enhanced query and scan capabilities over the standard RUM index.
type: commands
category: indexing
---

# Extended RUM

Extended RUM (`documentdb_extended_rum`) is a fork of the [RUM](https://github.com/postgrespro/rum) index access method that extends it with capabilities specific to DocumentDB's document-oriented workloads. Starting with DocumentDB **v0.108**, Extended RUM is the **default** index access method on PostgreSQL 18+ setups.

## Overview

DocumentDB uses RUM indexes internally for document indexing and query execution. Extended RUM builds on the standard RUM access method with improvements to the query path while keeping the same on-disk storage layout. This means:

- Indexes created with standard RUM and Extended RUM are **storage-compatible** — they have the same layout on disk.
- All changes in Extended RUM are limited to the query and scan paths. Existing indexes do not need to be rebuilt.
- Extended RUM is loaded as a separate PostgreSQL extension (`documentdb_extended_rum`) alongside the core DocumentDB extension.

## Default behavior by PostgreSQL version

| PostgreSQL version | Default RUM behavior | GUC default |
|---|---|---|
| PG 16, PG 17 | Standard RUM | `none` |
| PG 18+ | Extended RUM (**required**) | `require_documentdb_extended_rum` |

On PostgreSQL 18 and later, DocumentDB defaults to `require_documentdb_extended_rum`, meaning the Extended RUM library must be loaded for the server to function correctly. On earlier PostgreSQL versions, the standard RUM access method is used unless explicitly overridden.

## Configuration

### `documentdb.rum_library_load_option`

This GUC controls how the RUM library is loaded. It is a **postmaster-level** setting, meaning it requires a server restart to take effect.

| Value | Behavior |
|---|---|
| `none` | Use the standard RUM access method. Default on PG 16/17. |
| `prefer_documentdb_extended_rum` | Use Extended RUM if available; fall back to standard RUM otherwise. |
| `require_documentdb_extended_rum` | Require Extended RUM; fail to start if it is not available. Default on PG 18+. |

### `documentdb.alternate_index_handler_name`

When Extended RUM is enabled, this is set to `extended_rum` to route index operations through the Extended RUM access method.

## DocumentDB Local

When running DocumentDB Local via Docker, Extended RUM is **enabled by default**.

To disable Extended RUM, pass the `--disable-extended-rum` flag:

```bash
docker run -dt -p 10260:10260 --name docdb \
  ghcr.io/documentdb/documentdb/documentdb-local:latest \
  --username demo --password test \
  --disable-extended-rum
```

You can also set the `DISABLE_EXTENDED_RUM=true` environment variable:

```bash
docker run -dt -p 10260:10260 --name docdb \
  -e DISABLE_EXTENDED_RUM=true \
  ghcr.io/documentdb/documentdb/documentdb-local:latest \
  --username demo --password test
```

## Related features

### Ordered indexes

DocumentDB v0.109 enabled **ordered indexes by default**. Ordered indexes use a composite operator class that can improve sort and range query performance. This behavior works alongside Extended RUM and can be controlled independently:

- Per-index: specify `"storageEngine": {"enableOrderedIndex": false}` when creating an index.
- Server-wide: set the `documentdb.defaultUseCompositeOpClass` GUC to `off`.

## Version history

| Version | Change |
|---|---|
| v0.104 | `rum_enable_index_scan` enabled by default. |
| v0.106 | Internal Extended RUM extension added. |
| v0.108 | Extended RUM becomes the default on PG 18+. RUM dependency becomes implicit via the shared library. |
| v0.109 | Ordered indexes enabled by default (`defaultUseCompositeOpClass`). |
