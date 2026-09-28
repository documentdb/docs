---
title: Configuration
description: Server GUCs and gateway environment variables for configuring DocumentDB.
---

# Configuration

DocumentDB is configured at two layers: the PostgreSQL extension (via GUCs in `postgresql.conf`) and the gateway process (via environment variables or a JSON config file). This page covers the most commonly adjusted settings.

## Extension GUCs

`pg_documentdb` exposes tuning and feature-flag settings as PostgreSQL GUCs under the `documentdb.` prefix. Set them in `postgresql.conf`, per session with `SET`, or per role/database with `ALTER ROLE ... SET`.

### Schema validation

| GUC | Default | Description |
| --- | --- | --- |
| `documentdb.enableSchemaValidation` | `on` | Enforces collection `$jsonSchema` validators on write operations. When `off`, a collection's validator is stored but not enforced. |

Collections with a `validator` are enforced on `insert`, `update`, `findAndModify`, and aggregation output stages (`$merge`, `$out`) when the collection's `validationLevel` is not `off` and its `validationAction` is `error`. (A `validationAction` of `warn` is not rejected on the write path, and a write that sets `bypassDocumentValidation` skips enforcement — see `documentdb.enableBypassDocumentValidation`.)

### RUM index library

| GUC | Default | Description |
| --- | --- | --- |
| `documentdb.rum_library_load_option` | `require_documentdb_extended_rum` | Requires the DocumentDB extended RUM library on every supported PostgreSQL major version. Set this in `postgresql.conf` and restart PostgreSQL for a change to take effect. |

The package setup wizard and container startup handle the required extension setup. For a manually configured PostgreSQL instance, create both `documentdb` and `documentdb_extended_rum`; `CREATE EXTENSION documentdb CASCADE` does not create `documentdb_extended_rum` automatically. Follow the [package setup guidance](https://documentdb.io/docs/getting-started/packages/) before creating indexes.

Setting the option to `none` opts out of the library-load requirement.

### RUM vacuum pruning

| GUC | Default | Description |
| --- | --- | --- |
| `documentdb_rum.enable_targeted_posting_tree_pruning` | `on` | Vacuum takes a brief posting-tree root cleanup lock per deletion attempt instead of holding one for the whole pruning pass. |
| `documentdb_rum.prune_rum_empty_pages` | `on` | Vacuum removes empty entry-leaf pages from extended RUM indexes. |

### TOAST compression

`documentdb-tune`, `documentdb-setup` and the `documentdb-local` image write `default_toast_compression = 'lz4'`, which decompresses large documents faster than PostgreSQL's `pglz` default. It affects newly written values only. Override it with `documentdb-tune --toast-compression lz4|pglz|default` or the `DOCUMENTDB_TOAST_COMPRESSION` environment variable; `default` leaves PostgreSQL's own setting alone. An explicit `lz4` on a PostgreSQL build without lz4 support fails instead of being downgraded.

### Namespace validation

| GUC | Default | Description |
| --- | --- | --- |
| `documentdb.enable_null_collection_validation` | `on` | Rejects database and collection names that contain an embedded null character. |

### Non-blocking unique index builds

| GUC | Default | Description |
| --- | --- | --- |
| `documentdb.enableNonBlockingUniqueIndexBuild` | `on` | Builds unique ordered indexes without holding a long write lock, using `CREATE INDEX CONCURRENTLY` with post-processing to register the exclusion constraint and validate existing rows. |

When enabled, creating a unique ordered index on an existing collection does not block concurrent writes for the duration of the build.

### Off-by-default feature flags

These flags gate functionality that is otherwise silently unavailable — in each case the command still succeeds, so the symptom is a missing effect rather than an error.

| GUC | Default | Description |
| --- | --- | --- |
| `documentdb.enableCompactVacuumFull` | `off` | Allows `compact` with `mode: "full"`, the blocking `VACUUM FULL` that returns space to the operating system. While `off`, that mode returns `{ "ok": 1, "bytesFreed": 0 }` without doing any work. The default `standard` mode (a non-blocking `VACUUM`) runs regardless. |
| `documentdb.enablePreImages` | `off` | Allows the `changeStreamPreAndPostImages` collection option. While `off`, `create` and `collMod` reject that option. |
| `documentdb.indexBuildsScheduledOnBgWorker` | `off` | Drains the background index build queue from a PostgreSQL background worker instead of a pg_cron job. Leave `off` where pg_cron is configured and working; turn it on where pg_cron cannot run the job, otherwise queued index builds never start. |

### Nested `$lookup` ordering

| GUC | Default | Description |
| --- | --- | --- |
| `documentdb.force_nested_lookup_pipeline_after_join` | `off` | Runs nested `$lookup` stages after the parent equality join, so enrichment applies only to matched foreign documents. Unsupported query shapes may fail while it is `on`. |

### Role management flags

Unlike the flags above, these two produce an error rather than a missing effect, so a caller sees the failure immediately.

| GUC | Default | Description |
| --- | --- | --- |
| `documentdb.enableRoleCrud` | `off` | Enables role CRUD through the data plane, with custom roles persisted in a roles catalog. While `off`, `create_role`, `drop_role`, and `roles_info` each raise before doing any work, for example "The CreateRole command is currently unsupported." Note that `update_role` is not implemented in any case. |
| `documentdb.enableRolesAdminDBCheck` | `on` | Requires the wire-protocol role commands to be issued against the `admin` database, raising "CreateRole must be called from 'admin' database." otherwise. The user management commands are governed separately by `documentdb.enableUsersAdminDBCheck`, which is `off` by default. |

## Gateway configuration

The gateway (`pg_documentdb_gw`) reads its settings from a JSON configuration file and/or `DOCUMENTDB_*` environment variables. Environment variables override the JSON file, which makes them convenient for systemd-managed and container deployments.

> **Note:** The packaged gateway service — its systemd unit and per-major `gateway.env` file — is published as a release asset (`documentdb-gateway`, together with `documentdb-common`, which owns the unit templates). The `DOCUMENTDB_*` settings below apply to that packaged service, to the `documentdb-gateway` binary directly (which the `documentdb-local` container image configures internally), and to downstream packaging that installs its own unit. On a packaged install, `documentdb-setup` writes the managed block in `/etc/documentdb/local/<major>/gateway.env`; edits outside that block are preserved, but the managed block is rebuilt whenever the wizard re-runs.

| Environment variable | Purpose |
| --- | --- |
| `DOCUMENTDB_PG_URL_FILE` | Path to a file containing the PostgreSQL connection URL, read at startup. For package-managed installs the URL must not contain a password — the gateway connects over a local Unix socket using peer/trust authentication. |
| `DOCUMENTDB_LISTEN_ADDR` | Address the gateway listens on, in `host:port` or `:port` form (for example `:10260`). |
| `DOCUMENTDB_TLS_CERT_FILE` | Path to the TLS certificate file. |
| `DOCUMENTDB_TLS_KEY_FILE` | Path to the TLS private key file. |
| `DOCUMENTDB_TLS_AUTO_GENERATE` | When `true`, auto-generate a self-signed certificate if no cert/key files are provided. |
| `DOCUMENTDB_TLS_STATE_DIR` | Directory where an auto-generated certificate/key is written and re-read on restart (defaults to `/var/lib/documentdb-gateway/tls`). When that directory is not writable — as in the `documentdb-local` container image — the gateway falls back to a per-user state directory under `$HOME/.local/state` and logs the path it chose. |
| `DOCUMENTDB_LOG_LEVEL` | Log level for the gateway's tracing subscriber (for example `info`, `debug`). |

For systemd-managed installs these are typically set through the unit's `EnvironmentFile` (for example `gateway.env`).

### Connectivity check

The gateway binary provides a `check` subcommand that acts as a post-install connectivity probe:

```bash
documentdb-gateway check [--config <path>]
```

It connects to the configured PostgreSQL backend, runs the same startup validation as the service-start path, and reports the installed `documentdb` extension version. It exits `0` on success and `1` on failure (printing a human-readable message and a hint on stderr), which makes it suitable for post-install smoke tests and health checks.
