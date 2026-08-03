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

> This container writes its data to `/data` inside the container. Nothing is mounted there in the command above, so the database is discarded when the container is removed. See `--data-path` in the table below to persist it.

### Wait for the container to be ready

`docker ps` reports the container as `Up` well before DocumentDB can accept connections - PostgreSQL has to initialize, the extensions have to be set up, and the admin user has to be created first. Connecting too early fails with `MongoServerSelectionError` or `ECONNREFUSED`.

Wait for the ready banner in the logs before connecting:

```bash
docker logs -f docdb
```

```output
=== DocumentDB is ready ===
```

First start typically takes a few tens of seconds.

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
| Specify the username for DocumentDB. | `--username [value]` | Overrides `USERNAME` environment variable | STRING | `default_user` | Username for DocumentDB. |
| Specify the password for DocumentDB. | `--password [value]` | Overrides `PASSWORD` environment variable | STRING | `Admin100` | Password for DocumentDB. Always set this explicitly. The built-in default is well known, and anyone who can reach the published port can authenticate with it. |
| The port of the DocumentDB endpoint. | `--documentdb-port [value]` | Overrides `DOCUMENTDB_PORT` environment variable | INT | `10260` | The port needs to be published - for example, using `-p 10260:10260`. |
| Specify a directory for data. | `--data-path [value]` | Overrides `DATA_PATH` environment variable. | STRING | `/data` | Data is not persisted unless you mount a volume at this path - for example, `-v documentdb-data:/data`. To use a different directory, set the flag and the mount together: `--data-path /usr/documentdb/data` with `--mount type=bind,source=./.local/data,target=/usr/documentdb/data`. |
| Specify the owner. | `--owner [value]` | Overrides `OWNER` environment variable. | STRING | `documentdb` | Specify the owner for DocumentDB. |
| Specify whether to start the PostgreSQL server. | `--start-pg [value]` | Overrides `START_POSTGRESQL` environment variable | `true`, `false` | `true` | Specify whether to start the PostgreSQL server. |
| Specify whether to create a user. | `--create-user [value]` | Overrides `CREATE_USER` environment variable | `true`, `false` | `true` | Specify whether to create a user. |
| Specify the port for the PostgreSQL server. | `--pg-port [value]` | Overrides `POSTGRESQL_PORT` environment variable | INT | `9712` | Specify the port for the PostgreSQL server. |
| Specify whether to allow external connections to PostgreSQL. | `--allow-external-connections [value]` | Overrides `ALLOW_EXTERNAL_CONNECTIONS` environment variable | `true`, `false` | `false` | Opens the container's internal PostgreSQL server to all interfaces. This does not affect the gateway, which always listens on all interfaces on the DocumentDB port. |
| Specify the path to a certificate for securing traffic. | `--cert-path [value]` | Overrides `CERT_PATH` environment variable. | STRING | NA | PEM-format certificate. Must be set together with `--key-file` - setting only one of the two fails at startup. You need to mount this file into the container. For example, to set `/mycert.pem`, add this option to `docker run` command: `--mount type=bind,source=./mycert.pem,target=/mycert.pem`. |
| Override default key with key in key file. | `--key-file [value]` | Overrides `KEY_FILE` environment variable. | STRING | NA | PEM-format private key. Must be set together with `--cert-path` - setting only one of the two fails at startup. You need to mount this file into the container. For example, to set `/mykey.key`, add this option to `docker run` command: `--mount type=bind,source=./mykey.key,target=/mykey.key` |
| Set the TLS mode for client connections. | `--tlsMode [value]` | Overrides `TLS_MODE` environment variable | `disabled`, `allowTLS`, `requireTLS` | `allowTLS` | With `allowTLS` the gateway accepts both plain and TLS connections; `disabled` behaves the same way. `requireTLS` rejects plain connections, so every client must connect with `tls=true`. |
| Enable initialization with built-in sample data. | `--init-data [value]` | Overrides `INIT_DATA` environment variable | `true`, `false` | `false` | Seeded once per data volume, on a fresh volume. Re-create the volume to seed again. |
| Specify a directory of scripts for database initialization. | `--init-data-path [value]` | Overrides `INIT_DATA_PATH` environment variable | STRING | `/init_doc_db.d` | JavaScript files are executed in alphabetical order using `mongosh`, once per fresh data volume. Scripts should be idempotent - a failed run is not retried on restart. |
| Skip initialization with built-in sample data. | `--skip-init-data` | Overrides `SKIP_INIT_DATA` environment variable | N/A | N/A | Legacy alias for `--init-data false`. Does not affect `--init-data-path`. |
| Disable the use of extended RUM for indexes. | `--disable-extended-rum` | Overrides `DISABLE_EXTENDED_RUM` environment variable | N/A | `false` | Extended RUM is enabled by default. |
| Enable telemetry data. | `--enable-telemetry [value]` | Overrides `ENABLE_TELEMETRY` environment variable | `true`, `false` | `false` | Enable telemetry data sent to the usage collector. |
| Specify log verbosity. | `--log-level [value]` | Overrides `LOG_LEVEL` environment variable. | `quiet`, `error`, `warn`, `info`, `debug`, `trace` | `info` | The verbosity of logs that will be emitted. To set the gateway's own log level, use the `DOCUMENTDB_LOG_LEVEL` environment variable. |


## Feature support

Please refer to the [documentdb](https://documentdb.io/docs/) documentation for currently supported features.


## Installing certificates 

If you do not supply your own certificate with `--cert-path` and `--key-file`, DocumentDB Local generates a self-signed certificate on first start and reuses it on subsequent starts, so the certificate stays stable across restarts. To prevent certificate errors, install it on your local machine. The example below shows how to use this setup with `mongosh`.

### Get certificate

The gateway chooses where to store auto-generated TLS material based on which directories are writable, so pin the location with `DOCUMENTDB_TLS_STATE_DIR` when starting the container:

```bash
docker run -dt -p 10260:10260 --name docdb \
  -e DOCUMENTDB_TLS_STATE_DIR=/data/tls \
  ghcr.io/documentdb/documentdb/documentdb-local:latest --username demo --password test
```

In a `bash` window, run the following to copy the certificate from the container to the local
host: 

```bash
docker cp docdb:/data/tls/cert.pem ~/documentdb-cert.pem
```

If you did not set `DOCUMENTDB_TLS_STATE_DIR`, the gateway logs the path it chose on startup:

```bash
docker logs docdb | grep "TLS auto-gen"
```

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
