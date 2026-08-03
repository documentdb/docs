---
title: DocumentDB Local
description: Learn how to install and run DocumentDB Local using Docker for local development and testing. Includes setup instructions, configuration options, and certificate management.
---

# DocumentDB Local

DocumentDB Local provides a lightweight, containerized environment for developing and testing applications locally, including prototyping and integration testing.

## Prerequisites

- [Docker](https://www.docker.com/)

## Installation

Get the Docker container image using `docker pull`.

```bash
docker pull ghcr.io/documentdb/documentdb/documentdb-local:latest

```

## Running

To run the container, use `docker run`. Afterwards, use `docker ps` to validate that the container is running.

```bash
docker run -dt -p 10260:10260 --name docdb ghcr.io/documentdb/documentdb/documentdb-local:latest --username demo --password test


docker ps
```

```output
CONTAINER ID   IMAGE                                                                             COMMAND                  CREATED         STATUS         PORTS                                                                                                      NAMES
5aff734a3591   ghcr.io/documentdb/documentdb/documentdb-local:latest                             "/bin/bash -c '/home…"   5 seconds ago   Up 4 seconds   0.0.0.0:10260->10260/tcp, :::10260->10260/tcp                                                              docdb
```

> This container writes its database to `/data`, which the image declares as a Docker volume. The command above mounts nothing there, so each `docker run` gets a fresh anonymous volume: the data does not survive re-creating the container, and the old volume is left behind on the host until you prune it. Mount a named volume - `-v documentdb-data:/data` - to persist it. See `--data-path` in the table below.

### Wait for the container to be ready

`docker ps` reports the container as `Up` well before DocumentDB can accept connections - PostgreSQL has to initialize, the extensions have to be set up, and the admin user has to be created first. Connecting too early fails with `MongoServerSelectionError` or `ECONNREFUSED`.

The entrypoint prints a ready banner once the gateway is accepting connections. Wait for it before connecting:

```bash
timeout 180 bash -c 'until docker logs docdb 2>&1 | grep -q "=== DocumentDB is ready ==="; do sleep 2; done' \
  && echo "DocumentDB is ready"
```

First start typically takes a few tens of seconds. If the wait times out, the container most likely exited during startup - check `docker ps -a` and `docker logs docdb` for the error.

> Use `docker logs docdb` rather than `docker logs -f docdb` to check readiness. The container streams the PostgreSQL, gateway, and entrypoint logs to stdout for its whole lifetime, so `-f` never returns.

### Connect with mongosh

> The DocumentDB gateway endpoint is available on port `10260` by default. To access this with `mongosh`, run:

```bash
mongosh "mongodb://demo:test@localhost:10260/?tls=true&tlsAllowInvalidCertificates=true"
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
| Specify the password for DocumentDB. | `--password [value]` | Overrides `PASSWORD` environment variable | STRING | `Admin100` | Password for DocumentDB. Always set this explicitly. The built-in default is well known, and anyone who can reach the published port can authenticate with it. |
| The port of the DocumentDB endpoint. | `--documentdb-port [value]` | Overrides `DOCUMENTDB_PORT` environment variable | INT | `10260` | The port needs to be published - for example, using `-p 10260:10260`. |
| Specify a directory for data. | `--data-path [value]` | Overrides `DATA_PATH` environment variable. | STRING | `/data` | Data is not persisted unless you mount a volume at this path - for example, `-v documentdb-data:/data`. To use a different directory, set the mount and the flag together, keeping in mind that they go on opposite sides of the image name: `-v` / `--mount` is a `docker run` option and comes before it, `--data-path` is a container argument and comes after it. See the example below the table. |
| Specify the owner. | `--owner [value]` | Overrides `OWNER` environment variable. | STRING | `documentdb` | The PostgreSQL role used to create the admin user. This image ships only the `documentdb` superuser, so leave it at the default: any other value fails with `role "<value>" does not exist` after PostgreSQL has already initialized, and the container exits. |
| Specify whether to start the PostgreSQL server. | `--start-pg [value]` | Overrides `START_POSTGRESQL` environment variable | `true`, `false` | `true` | Specify whether to start the PostgreSQL server. |
| Specify whether to create a user. | `--create-user [value]` | Overrides `CREATE_USER` environment variable | `true`, `false` | `true` | Specify whether to create a user. |
| Specify the port for the PostgreSQL server. | `--pg-port [value]` | Overrides `POSTGRESQL_PORT` environment variable | INT | `9712` | Specify the port for the PostgreSQL server. |
| Specify whether to allow external connections to PostgreSQL. | `--allow-external-connections [value]` | Overrides `ALLOW_EXTERNAL_CONNECTIONS` environment variable | `true`, `false` | `false` | Opens the container's internal PostgreSQL server to all interfaces and adds a permissive host-based authentication rule (`host all all 0.0.0.0/0 scram-sha-256`), which lets any role reach any database from any address with a password. It only changes configuration inside the container, so you also need to publish the PostgreSQL port - for example `-p 9712:9712` - to connect from the host. Ignored when `--start-pg false`. This does not affect the gateway, which always listens on all interfaces on the DocumentDB port. |
| Specify the path to a certificate for securing traffic. | `--cert-path [value]` | Overrides `CERT_PATH` environment variable. | STRING | NA | PEM-format certificate. Must be set together with `--key-file` - setting only one of the two fails at startup. You need to mount this file into the container. For example, to set `/mycert.pem`, add this option to `docker run` command: `--mount type=bind,source=./mycert.pem,target=/mycert.pem`. |
| Override default key with key in key file. | `--key-file [value]` | Overrides `KEY_FILE` environment variable. | STRING | NA | PEM-format private key. Must be set together with `--cert-path` - setting only one of the two fails at startup. You need to mount this file into the container. For example, to set `/mykey.key`, add this option to `docker run` command: `--mount type=bind,source=./mykey.key,target=/mykey.key` |
| Set the TLS mode for client connections. | `--tlsMode [value]` | Overrides `TLS_MODE` environment variable | `disabled`, `allowTLS`, `requireTLS` | `allowTLS` | With `allowTLS` the gateway accepts both plain and TLS connections; `disabled` behaves the same way. `requireTLS` rejects plain connections, so every client must connect with `tls=true`. |
| Enable initialization with built-in sample data. | `--init-data [value]` | Overrides `INIT_DATA` environment variable | `true`, `false` | `false` | Seeded once per data volume, on a fresh volume. Re-create the volume to seed again. |
| Specify a directory of scripts for database initialization. | `--init-data-path [value]` | Overrides `INIT_DATA_PATH` environment variable | STRING | `/init_doc_db.d` | JavaScript files are executed in alphabetical order using `mongosh`, once per fresh data volume. Scripts should be idempotent - a failed run is not retried on restart. |
| Skip initialization with built-in sample data. | `--skip-init-data` | Overrides `SKIP_INIT_DATA` environment variable | `true`, `false` (`SKIP_INIT_DATA` only - the flag itself takes no value) | N/A | Legacy alias for `--init-data false`. Note that `SKIP_INIT_DATA=false` does the opposite of the flag: with `INIT_DATA` unset it enables the built-in sample data. Does not affect `--init-data-path`. |
| Disable the use of extended RUM for indexes. | `--disable-extended-rum` | Overrides `DISABLE_EXTENDED_RUM` environment variable | N/A (takes no value) | N/A | Extended RUM is enabled by default. **Known issue:** this flag does not currently disable it - the container still starts with `documentdb_extended_rum` configured. |
| Enable telemetry data. | `--enable-telemetry [value]` | Overrides `ENABLE_TELEMETRY` environment variable | `true`, `false` | `false` | **Known issue:** the value is validated at startup but no telemetry is currently emitted - the gateway's metrics and tracing exporters are disabled in this image, and an invalid value only serves to abort startup. |
| Specify log verbosity. | `--log-level [value]` | Overrides `LOG_LEVEL` environment variable. | `quiet`, `error`, `warn`, `info`, `debug`, `trace` | `info` | **Known issue:** the value is validated at startup but does not currently change what the container logs. To change the gateway's own verbosity, set the `DOCUMENTDB_LOG_LEVEL` environment variable instead; it takes a tracing filter such as `info` or `debug` (`quiet` is not one of its values). |

> `--skip-init-data` and `--disable-extended-rum` are the only options that take no value. Passing one anyway - for example `--disable-extended-rum false` - leaves the container spinning in its argument parser: it produces no logs, never becomes ready, and never exits.

A complete `docker run` showing where each kind of option goes - Docker options before the image name, container arguments after it. This is the command from [Running](#running) with a persistent volume and sample data added, so remove that container first with `docker rm -f docdb`:

```bash
docker run -dt \
  -p 10260:10260 \
  -v documentdb-data:/data \
  --name docdb \
  ghcr.io/documentdb/documentdb/documentdb-local:latest \
  --username demo --password test --init-data true
```


## Feature support

Please refer to the [documentdb](https://documentdb.io/docs/) documentation for currently supported features.


## Installing certificates 

If you do not supply your own certificate with `--cert-path` and `--key-file`, DocumentDB Local generates a self-signed certificate on first start and reuses it on subsequent starts of the same container, so `docker stop` / `docker start` keeps it stable. Removing and re-creating the container generates a new certificate unless you persist the directory it is stored in. The generated certificate is valid for 365 days and is not renewed automatically - re-create the container, or delete the files below, to generate a fresh one.

To validate the certificate instead of skipping validation with `tlsAllowInvalidCertificates=true`, copy it out of the container and point `mongosh` at it.

### Get certificate

The gateway picks its TLS state directory from the first writable candidate. In this image that resolves to a path under the container user's home directory, so no extra options are needed on the `docker run` above. In a `bash` window, copy the certificate from the container to the local host:

```bash
docker cp docdb:/home/documentdb/.local/state/documentdb-gateway/tls/cert.pem ~/documentdb-cert.pem
```

The gateway logs the path it actually chose on startup. Check there first if the copy reports `No such container:path`:

```bash
docker logs docdb | grep "TLS auto-gen"
```

To pin the location yourself - for example on a mounted volume, so the certificate survives re-creating the container - set `DOCUMENTDB_TLS_STATE_DIR` when you first start the container, and point it somewhere outside `--data-path`. The entrypoint runs `chmod -R 750` over the data directory on every start, which would loosen the private key's permissions on each restart.

### Use the certificate with mongosh

```bash
mongosh localhost:10260 -u demo -p test --authenticationMechanism SCRAM-SHA-256 --tls --tlsCAFile ~/documentdb-cert.pem
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


## Reporting issues

If you encounter issues with using this version of DocumentDB, open an issue in the GitHub repository (<https://github.com/documentdb/documentdb/issues>) and tag it with the label `documentdb-local`.
