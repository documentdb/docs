---
title: Configuration
description: Server GUCs and gateway environment variables for configuring DocumentDB, including the defaults introduced in v0.114-0.
---

# Configuration

DocumentDB is configured at two layers: the PostgreSQL extension (via GUCs in `postgresql.conf`) and the gateway process (via environment variables or a JSON config file). This page covers the most commonly adjusted settings, including behavior that changed in **v0.114-0**.

## Extension GUCs

`pg_documentdb` exposes tuning and feature-flag settings as PostgreSQL GUCs under the `documentdb.` prefix. Set them in `postgresql.conf`, per session with `SET`, or per role/database with `ALTER ROLE ... SET`.

### Schema validation

| GUC | Default | Description |
| --- | --- | --- |
| `documentdb.enableSchemaValidation` | `on` (since v0.114-0) | Enforces collection `$jsonSchema` validators on write operations. When `off`, a collection's validator is stored but not enforced. |

Collections with a `validator` are enforced on `insert`, `update`, `findAndModify`, and aggregation output stages (`$merge`, `$out`) when the collection's `validationLevel` is not `off` and its `validationAction` is `error`. (A `validationAction` of `warn` is not rejected on the write path, and a write that sets `bypassDocumentValidation` skips enforcement — see `documentdb.enableBypassDocumentValidation`.) Prior to v0.114-0 enforcement was opt-in; it is now enabled by default.

### Non-blocking unique index builds

| GUC | Default | Description |
| --- | --- | --- |
| `documentdb.enableNonBlockingUniqueIndexBuild` | `on` (since v0.114-0) | Builds unique ordered indexes without holding a long write lock, using `CREATE INDEX CONCURRENTLY` with post-processing to register the exclusion constraint and validate existing rows. |

When enabled, creating a unique ordered index on an existing collection no longer blocks concurrent writes for the duration of the build.

## Gateway configuration

The gateway (`pg_documentdb_gw`) reads its settings from a JSON configuration file and/or `DOCUMENTDB_*` environment variables. Environment variables override the JSON file, which makes them convenient for systemd-managed and container deployments. *(Environment-variable configuration added in v0.114-0.)*

> **Note:** The packaged gateway service (its systemd unit and `gateway.env` file) is not yet published as a release asset — the v0.114-0 GitHub release ships the PostgreSQL extension packages only. The `DOCUMENTDB_*` settings below apply to the `documentdb-gateway` binary (which the `documentdb-local` container image configures internally) and to downstream packaging that installs the systemd unit.

| Environment variable | Purpose |
| --- | --- |
| `DOCUMENTDB_PG_URL_FILE` | Path to a file containing the PostgreSQL connection URL, read at startup. For package-managed installs the URL must not contain a password — the gateway connects over a local Unix socket using peer/trust authentication. |
| `DOCUMENTDB_LISTEN_ADDR` | Address the gateway listens on, in `host:port` or `:port` form (for example `:10260`). |
| `DOCUMENTDB_TLS_CERT_FILE` | Path to the TLS certificate file. |
| `DOCUMENTDB_TLS_KEY_FILE` | Path to the TLS private key file. |
| `DOCUMENTDB_TLS_AUTO_GENERATE` | When `true`, auto-generate a self-signed certificate if no cert/key files are provided. |
| `DOCUMENTDB_TLS_STATE_DIR` | Directory where an auto-generated certificate/key is written and re-read on restart (defaults to `/var/lib/documentdb-gateway/tls`). |
| `DOCUMENTDB_LOG_LEVEL` | Log level for the gateway's tracing subscriber (for example `info`, `debug`). |

For systemd-managed installs these are typically set through the unit's `EnvironmentFile` (for example `gateway.env`).

### Connectivity check

The gateway binary provides a `check` subcommand (added in v0.114-0) that acts as a post-install connectivity probe:

```bash
documentdb-gateway check [--config <path>]
```

It connects to the configured PostgreSQL backend, runs the same startup validation as the service-start path, and reports the installed `documentdb` extension version. It exits `0` on success and `1` on failure (printing a human-readable message and a hint on stderr), which makes it suitable for post-install smoke tests and health checks.
