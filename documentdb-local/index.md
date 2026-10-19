---
title: DocumentDB Local
description: Learn how to install and run DocumentDB Local using Docker for local development and testing. Includes setup instructions, configuration options, and certificate management.
---

# DocumentDB Local

DocumentDB Local provides a lightweight, containerized environment for developing and testing applications locally, including prototyping and integration testing.

The examples below use the PostgreSQL 17 image from release **1.0.0**. Images for PostgreSQL 15 to 18 are tagged `pg15-1.0.0` to `pg18-1.0.0`, for Linux amd64 and arm64; `latest` points to `pg17-1.0.0`. Use a versioned tag when you need a reproducible setup. The [1.0 release notes](../getting-started/release-notes-1.0.md) list what changed and the known issues.

> **Usage telemetry is on by default.** The 1.0 image reports its version, platform and CPU architecture to Scarf at startup and hourly. Turn it off with `-e DOCUMENTDB_USAGE_TELEMETRY=false`. See [Usage telemetry](#usage-telemetry).

## Prerequisites

- [Docker](https://www.docker.com/)

## Installation

Get the Docker container image using `docker pull`.

```bash
docker pull ghcr.io/documentdb/documentdb/documentdb-local:pg17-1.0.0
```

## Running

Choose the admin credentials first, in the same terminal (this works in bash and zsh):

```bash
printf 'DocumentDB username: '; IFS= read -r DOCUMENTDB_USERNAME
printf 'DocumentDB password: '; IFS= read -r -s DOCUMENTDB_PASSWORD; echo
export DOCUMENTDB_USERNAME DOCUMENTDB_PASSWORD
```

Then start the container with `docker run`, and use `docker ps` to check that it is running:

```bash
docker run -dt -p 127.0.0.1:10260:10260 --name docdb \
  -e USERNAME="${DOCUMENTDB_USERNAME:?Set DOCUMENTDB_USERNAME first}" \
  -e PASSWORD="${DOCUMENTDB_PASSWORD:?Set DOCUMENTDB_PASSWORD first}" \
  ghcr.io/documentdb/documentdb/documentdb-local:pg17-1.0.0

docker ps
```

```output
CONTAINER ID   IMAGE                                                         COMMAND                  CREATED         STATUS                            PORTS                        NAMES
5aff734a3591   ghcr.io/documentdb/documentdb/documentdb-local:pg17-1.0.0     "/bin/bash -c '/home…"   5 seconds ago   Up 4 seconds (health: starting)   127.0.0.1:10260->10260/tcp   docdb
```

> The exported variables are reused by the client examples below; if you open a new shell, set them again. The guards stop an empty value from falling through to the image's public default credentials, which the container still accepts but warns about loudly. The credentials are passed as environment variables because `--username` and `--password` arguments stay visible in the container's process list.
>
> The loopback binding makes the gateway reachable only from this host. To allow remote clients, change it to `-p 10260:10260` only after restricting the port with a firewall and configuring a certificate that remote clients can validate.
>
> This container writes its database to `/data`, which the image declares as a Docker volume. The command above mounts nothing there, so each `docker run` gets a fresh anonymous volume: the data does not survive re-creating the container, and the old volume is left behind on the host until you prune it. Mount a named volume - `-v documentdb-data:/data` - to persist it. See `--data-path` in the table below.

### Wait for the container to be ready

`docker ps` reports the container as `Up` well before DocumentDB can accept connections - PostgreSQL has to initialize, the extensions have to be set up, and the admin user has to be created first. Connecting too early fails with `MongoServerSelectionError` or `ECONNREFUSED`.

The entrypoint prints a ready banner after gateway startup and any requested data initialization finish. Wait for it before connecting:

```bash
until docker logs docdb 2>&1 | grep -q "=== DocumentDB is ready ==="; do sleep 2; done
```

First start typically takes a few tens of seconds. If the command has not returned after a couple of minutes, the container most likely exited during startup - interrupt it and check `docker ps -a` and `docker logs docdb` for the error.

> Use `docker logs docdb` rather than `docker logs -f docdb` to check readiness. The container streams the PostgreSQL, gateway, and entrypoint logs to stdout for its whole lifetime, so `-f` never returns.

### Connect with mongosh

> The DocumentDB gateway endpoint is available on port `10260` by default. To access this with `mongosh`, run:

```bash
mongosh localhost:10260 \
  -u "${DOCUMENTDB_USERNAME:?Set DOCUMENTDB_USERNAME first}" \
  -p \
  --authenticationMechanism SCRAM-SHA-256 \
  --tls --tlsAllowInvalidCertificates
```

```output
Current Mongosh Log ID:	690cdcb84e2e610f0f48e609
Connecting to:		mongodb://<credentials>@localhost:10260/?tls=true&tlsAllowInvalidCertificates=true&directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.5.1
Using MongoDB:		7.0.0
Using Mongosh:		2.5.1
mongosh 2.5.9 is available for download: https://www.mongodb.com/try/download/shell

For mongosh info see: https://www.mongodb.com/docs/mongodb-shell/

[direct: mongos] test>
```

## Docker commands

The following table summarizes the available Docker commands for configuring the emulator. This table details the corresponding arguments, environment variables, allowed values, default settings, and descriptions of each command.

| Requirement | Arg | Env | Allowed values | Default | Description |
|---|---|---|---|---|---|
| Print the settings to stdout from the container | `--help`, `-h` | N/A | N/A | N/A | Display information on available configuration |
| Specify the username for DocumentDB. | `--username [value]` | Overrides `USERNAME` environment variable | STRING | `default_user` | Username for DocumentDB. It may not be an internal DocumentDB role name, and it may not begin with `documentdb`, `citus`, `pg`, or `internal_role` (case-insensitive). The container rejects a reserved name and exits before starting anything. |
| Specify the password for DocumentDB. | `--password [value]` | Overrides `PASSWORD` environment variable | STRING | `Admin100` | Password for DocumentDB. Always set this explicitly. The built-in default is well known, and anyone who can reach the published port can authenticate with it. When the password is left at its default, the container prints a warning at startup; `DOCUMENTDB_ALLOW_DEFAULT_PASSWORD=true` silences it. Prefer the environment variable: a `--password` argument stays visible in the container's process list. A changed password on an existing data volume is ignored; the volume keeps the admin user it was initialized with. |
| The port of the DocumentDB endpoint. | `--documentdb-port [value]` | Overrides `DOCUMENTDB_PORT` environment variable | INT | `10260` | The port needs to be published. For local use, bind only to loopback - for example, `-p 127.0.0.1:10260:10260`. To use host port `27017` without changing the gateway port, publish `-p 127.0.0.1:27017:10260`; add `--documentdb-port 27017` only when changing the container-side port too. |
| Specify a directory for data. | `--data-path [value]` | Overrides `DATA_PATH` environment variable. | STRING | `/data` | Data is not persisted unless you mount a volume at this path - for example, `-v documentdb-data:/data`. To use a different directory, set the mount and the flag together, keeping in mind that they go on opposite sides of the image name: `-v` / `--mount` is a `docker run` option and comes before it, `--data-path` is a container argument and comes after it. See the example below the table. |
| Specify the owner. | `--owner [value]` | Overrides `OWNER` environment variable. | STRING | `documentdb` | The PostgreSQL role used to create the admin user. The cluster this image initializes has a single superuser role, `documentdb`, so leave this at the default: any other value fails with `role "<value>" does not exist` after PostgreSQL has already initialized, and the container exits. |
| Specify whether to start the PostgreSQL server. | `--start-pg [value]` | Overrides `START_POSTGRESQL` environment variable | `true`, `false` | `true` | Set this to `false` only when you are pointing the gateway at a PostgreSQL server you run yourself; the container then expects one to be reachable on `--pg-port`. |
| Specify whether to create a user. | `--create-user [value]` | Overrides `CREATE_USER` environment variable | `true`, `false` | `true` | With `false` the container starts the gateway without creating the admin user. Nothing can authenticate with `--username` / `--password` until you create a user yourself, and data initialization fails if you enabled it. |
| Specify the port for the PostgreSQL server. | `--pg-port [value]` | Overrides `POSTGRESQL_PORT` environment variable | INT | `9712` | Specify the port for the PostgreSQL server. |
| Specify whether to allow external connections to PostgreSQL. | `--allow-external-connections [value]` | Overrides `ALLOW_EXTERNAL_CONNECTIONS` environment variable | `true`, `false` | `false` | Opens the container's internal PostgreSQL server to all interfaces and adds permissive host-based authentication rules (`host all all 0.0.0.0/0 scram-sha-256` and `host all all ::0/0 scram-sha-256`), which let any role reach any database from any address with a password. It only changes configuration inside the container, so you also need to publish the PostgreSQL port - for example `-p 9712:9712` - to connect from the host. Ignored when `--start-pg false`. This does not affect the gateway, which always listens on all interfaces on the DocumentDB port. |
| Specify the path to a certificate for securing traffic. | `--cert-path [value]` | Overrides `CERT_PATH` environment variable. | STRING | NA | PEM-format certificate. Must be set together with `--key-file` - setting only one of the two fails at startup. You need to mount this file into the container. For example, to set `/mycert.pem`, add this option to `docker run` command: `--mount type=bind,source=./mycert.pem,target=/mycert.pem`. |
| Override default key with key in key file. | `--key-file [value]` | Overrides `KEY_FILE` environment variable. | STRING | NA | PEM-format private key. Must be set together with `--cert-path` - setting only one of the two fails at startup. You need to mount this file into the container. For example, to set `/mykey.key`, add this option to `docker run` command: `--mount type=bind,source=./mykey.key,target=/mykey.key` |
| Set the TLS mode for client connections. | `--tlsMode [value]` | Overrides `TLS_MODE` environment variable | `disabled`, `allowTLS`, `requireTLS` | `allowTLS` | With `allowTLS` the gateway accepts both plain and TLS connections; `disabled` behaves the same way. `requireTLS` rejects plain connections, so every client must connect with `tls=true`. |
| Enable initialization with built-in sample data. | `--init-data [value]` | Overrides `INIT_DATA` environment variable | `true`, `false` | `false` | Loads the `StoreData` dataset once per data volume: a marker file in the volume records that it ran, so it also loads into a volume that already holds data but was never seeded. Use a new volume to seed again; see [Built-in sample data](#built-in-sample-data). |
| Specify a directory of scripts for database initialization. | `--init-data-path [value]` | Overrides `INIT_DATA_PATH` environment variable | STRING | `/init_doc_db.d` | JavaScript files run alphabetically using `mongosh`, once per data volume (a marker file records the attempt, so they also run against an existing volume that never ran them), before the built-in sample data when both are enabled. A syntax or runtime error stops the container with an error. The run isn't repeated on restart, so fix the scripts and start from a fresh volume. |
| Skip initialization with built-in sample data. | `--skip-init-data` | Overrides `SKIP_INIT_DATA` environment variable | `true`, `false` (`SKIP_INIT_DATA` only - the flag itself takes no value) | N/A | Legacy alias for `--init-data false`. Note that `SKIP_INIT_DATA=false` does the opposite of the flag: with `INIT_DATA` unset it enables the built-in sample data. Does not affect `--init-data-path`. |
| Disable the use of extended RUM for indexes. | `--disable-extended-rum` | Overrides `DISABLE_EXTENDED_RUM` environment variable | N/A (takes no value) | N/A | **Deprecated and ignored**, apart from a warning at startup; it will be removed in a later release. `documentdb_extended_rum` is always enabled on a volume this image initializes. |
| Enable telemetry data. | `--enable-telemetry [value]` | Overrides `ENABLE_TELEMETRY` environment variable | `true`, `false` | `false` | The gateway's operational metrics, unrelated to [usage telemetry](#usage-telemetry). **Known issue:** the value is validated at startup but no telemetry is currently emitted - the gateway's metrics and tracing exporters are disabled in this image, and an invalid value only serves to abort startup. |
| Specify log verbosity. | `--log-level [value]` | Overrides `LOG_LEVEL` environment variable. | `quiet`, `error`, `warn`, `info`, `debug`, `trace` | `info` | **Known issue:** the value is validated at startup but does not currently change what the container logs. To change the gateway's own verbosity, set the `DOCUMENTDB_LOG_LEVEL` environment variable instead; it takes a tracing filter such as `info` or `debug` (`quiet` is not one of its values). |
| Turn usage telemetry on or off. | `--usage-telemetry [value]` | Overrides `DOCUMENTDB_USAGE_TELEMETRY` environment variable | `true`, `false` (also `1`/`0`, `yes`/`no`, `on`/`off`, case-insensitive) | `true` | Reports the image version, platform and CPU architecture at startup and hourly. An unrecognized value is treated as `false` with a warning, so an opt-out can never stop the container from starting. See [Usage telemetry](#usage-telemetry). |
| Turn usage telemetry off. | `--disable-usage-telemetry` | N/A | N/A (takes no value) | N/A | Same as `--usage-telemetry false`. |
| Set TOAST compression. | `--toast-compression [value]` | Overrides `DOCUMENTDB_TOAST_COMPRESSION` environment variable | `lz4`, `pglz`, `default` | `lz4` | Compression for large values PostgreSQL stores out of line (TOAST). `lz4` decompresses large documents faster, `pglz` is PostgreSQL's built-in method, and `default` leaves the server's own setting alone. Affects newly written values only. Ignored, with a warning, when `--start-pg false`. |

> `--skip-init-data`, `--disable-usage-telemetry` and `--disable-extended-rum` are the only options that take no value. Passing one anyway - for example `--disable-usage-telemetry false` - is rejected as an unknown option and the container exits.

A complete `docker run` showing where each kind of option goes - Docker options before the image name, container arguments after it. This is the command from the **Running** section above with a persistent volume and sample data added, so remove that container first with `docker rm -f docdb`:

```bash
docker run -dt \
  -p 127.0.0.1:10260:10260 \
  -v documentdb-data:/data \
  --name docdb \
  -e USERNAME="${DOCUMENTDB_USERNAME:?Set DOCUMENTDB_USERNAME first}" \
  -e PASSWORD="${DOCUMENTDB_PASSWORD:?Set DOCUMENTDB_PASSWORD first}" \
  ghcr.io/documentdb/documentdb/documentdb-local:pg17-1.0.0 \
  --init-data true
```

## Usage telemetry

The 1.0 image sends usage data by default. The native packages send nothing.

At startup and then hourly, the image reports its version, platform and CPU architecture to [Scarf](https://scarf.sh). It never sends database, collection or user names, queries, documents or credentials. Scarf does see the source IP address of each request, and the privacy statement says that address shouldn't be assumed to be anonymous once received. A `[TELEMETRY]` notice in the container log says when it is on.

To turn it off, add any of these before the image name:

```bash
-e DOCUMENTDB_USAGE_TELEMETRY=false
-e NO_ANALYTICS=1
-e DO_NOT_TRACK=1
```

or `--disable-usage-telemetry` (or `--usage-telemetry false`) after it. `NO_ANALYTICS` and `DO_NOT_TRACK` win over any setting that turns it on. To send the reports to your own collector instead, set `DOCUMENTDB_USAGE_TELEMETRY_ENDPOINT`.

In CI, turn it off explicitly with `-e DOCUMENTDB_USAGE_TELEMETRY=false`. The image skips telemetry only when `CI`, `GITHUB_ACTIONS` or `TF_BUILD` is set to a non-empty value in the container's own environment, and `docker run` doesn't pass the runner's variables through: a container started on a CI runner still reports unless you forward one, for example `-e GITHUB_ACTIONS`. `--enable-telemetry` is a separate setting for the gateway's own metrics and doesn't affect this.

The [privacy statement](https://github.com/documentdb/documentdb/blob/v1.0-0/documentdb-local/PRIVACY.md) has the details. The image also includes it at `/home/documentdb/PRIVACY.md`.

## Built-in sample data

The built-in sample data is the `StoreData` dataset. Loading it is opt-in: add `--init-data true` to the container arguments, as in the example above. If you also give scripts in `--init-data-path`, both run: your scripts first, then the sample data, each tracked by its own marker in the volume.

| Collection | Documents |
|---|---:|
| `StoreData.stores` | 41,505 |
| `StoreData.ratings` | 2 |

The bundled Extended JSON preserves BSON Binary, Date, and Timestamp values. After the ready banner, inspect the collections with `mongosh`:

```javascript
db.getSiblingDB("StoreData").stores.countDocuments({})   // 41505
db.getSiblingDB("StoreData").ratings.countDocuments({})  // 2
```

Seeding is one-shot per data volume; use a new, empty volume to seed again. A direct loader rerun tolerates duplicate keys rather than duplicating documents. See the [versioned sample-data guide](https://github.com/documentdb/documentdb/tree/v1.0-0/documentdb-local/sample-data) for manual loader instructions.

## Container lifecycle

- **Health check.** The image has a Docker health check. `docker inspect -f '{{.State.Health.Status}}' docdb` reports `healthy` once DocumentDB accepts connections.
- **Stale `postmaster.pid`.** After `docker kill`, `docker rm -f`, an out-of-memory kill or a host crash, the next start removes the stale file, but only when the volume's lock shows a 1.0 container wrote it. Otherwise it refuses to start, because an older container may still be using the volume. Once nothing else uses it, re-create the container once with `-e DOCUMENTDB_FORCE_REMOVE_STALE_POSTMASTER_PID=true`, then without it (`docker start` can't add the variable). Don't share a volume between hosts. A second container on a volume that is already in use exits immediately.
- **PostgreSQL loss stops the container.** If PostgreSQL inside the container stops, the gateway fails requests straight away and the container exits after 20 seconds (`DOCUMENTDB_POSTMASTER_EXIT_TIMEOUT`), so a restart policy can bring it back.

## Upgrading the image

A newer image on an existing data volume doesn't update the DocumentDB extensions in it ([#773](https://github.com/documentdb/documentdb/issues/773)), so the volume keeps the old schema under the new binaries. Upgrading a volume in place is not supported in 1.0. To move data from a 0.117 container to 1.0, export it with `mongodump`, start a 1.0 container on a new volume, and load it with `mongorestore`. The PostgreSQL major version of the image must also match the one that initialized the volume.

If you do start a 1.0 image on a volume an older container used, the 1.0 image refuses to start while the old container's `postmaster.pid` is still there; see [Container lifecycle](#container-lifecycle) for the override and removing it afterward. For 0.116 and 0.117, stop the old container with `docker stop` first, which removes the file; after `docker kill`, `docker rm -f` or a crash it stays. Images 0.114 and earlier don't normally stop PostgreSQL cleanly, so their volumes usually keep the file.

## Feature support

Please refer to the [documentdb](https://documentdb.io/docs/) documentation for currently supported features.


## Installing certificates 

If you do not supply your own certificate with `--cert-path` and `--key-file`, DocumentDB Local generates a self-signed certificate on first start and reuses it on subsequent starts of the same container, so `docker stop` / `docker start` keeps it stable. Removing and re-creating the container generates a new certificate unless you persist the directory it is stored in. The generated certificate is valid for 365 days and is not renewed automatically - re-create the container, or delete `cert.pem` from the state directory shown below, to generate a fresh one.

To validate the certificate instead of skipping validation with `tlsAllowInvalidCertificates=true`, copy it out of the container and point `mongosh` at it.

### Get certificate

The gateway picks its TLS state directory from the first writable candidate. In this image that resolves to a path under the container user's home directory, so no extra options are needed when starting the container. In a `bash` window, copy the certificate from the container to the local host:

```bash
docker cp docdb:/home/documentdb/.local/state/documentdb-gateway/tls/cert.pem ~/documentdb-cert.pem
```

The gateway logs the path it actually chose on startup. Check there first if the copy reports `No such container:path`:

```bash
docker logs docdb | grep "TLS auto-gen"
```

To keep the same certificate across re-creating the container, pin the location with `DOCUMENTDB_TLS_STATE_DIR` and put it inside the data volume. This replaces the container you started earlier, so run `docker rm -f docdb` first:

```bash
docker run -dt \
  -p 127.0.0.1:10260:10260 \
  -v documentdb-data:/data \
  -e DOCUMENTDB_TLS_STATE_DIR=/data/tls \
  --name docdb \
  -e USERNAME="${DOCUMENTDB_USERNAME:?Set DOCUMENTDB_USERNAME first}" \
  -e PASSWORD="${DOCUMENTDB_PASSWORD:?Set DOCUMENTDB_PASSWORD first}" \
  ghcr.io/documentdb/documentdb/documentdb-local:pg17-1.0.0
```

Point it inside the data directory rather than at a volume of its own: a separate volume is created root-owned, and the gateway - which runs as an unprivileged user - cannot write its key there. The trade-off is that the key is included in any backup of the data volume. If the entrypoint ever has to repair the data directory's ownership (it does when the owner doesn't match, or with `DOCUMENTDB_FORCE_OWNERSHIP_REPAIR=true`), it runs `chmod -R 750` over the directory, which leaves the private key group-readable rather than owner-only.

### Use the certificate with mongosh

```bash
mongosh localhost:10260 \
  -u "${DOCUMENTDB_USERNAME:?Set DOCUMENTDB_USERNAME first}" \
  -p \
  --authenticationMechanism SCRAM-SHA-256 \
  --tls --tlsCAFile ~/documentdb-cert.pem
```

```output
Current Mongosh Log ID:	690ce1171181053c6edbf354
Connecting to:		mongodb://<credentials>@localhost:10260/?directConnection=true&serverSelectionTimeoutMS=2000&authMechanism=SCRAM-SHA-256&tls=true&tlsCAFile=%2Fhome%2Fuser%2Fdocumentdb-cert.pem&appName=mongosh+2.5.1
Using MongoDB:		7.0.0
Using Mongosh:		2.5.1
mongosh 2.5.9 is available for download: https://www.mongodb.com/try/download/shell

For mongosh info see: https://www.mongodb.com/docs/mongodb-shell/

[direct: mongos] test>
```


## Beyond local development

DocumentDB Local runs a single container on one machine, with no replication and no failover, which is what makes it convenient for development and testing. For other ways to run DocumentDB:

- [Kubernetes Operator](https://documentdb.io/docs/kubernetes-operator/) - run DocumentDB as a replicated service, with automatic failover, backup and restore, and rolling upgrades.
- [Pre-built packages](https://documentdb.io/docs/getting-started/packages/) - add the DocumentDB extension to a PostgreSQL server you already run.


## Reporting issues

If you encounter issues with using this version of DocumentDB, open an issue in the GitHub repository (<https://github.com/documentdb/documentdb/issues>) and tag it with the label `documentdb-local`.
