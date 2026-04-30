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
5aff734a3591   ghcr.io/documentdb/documentdb/documentdb-local:latest                             "/bin/bash -c '/home…"   5 seconds ago   Up 4 seconds   0.0.0.0:10260->10260/tcp, :::10260->10260/tcp                                                              optimistic_blackwell
```

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
| Specify the password for DocumentDB. | `--password [value]` | Overrides `PASSWORD` environment variable | STRING | `Admin100` | Password for DocumentDB. Set your own value when you start the container. |
| The port of the DocumentDB endpoint. | `--documentdb-port [value]` | Overrides `DOCUMENTDB_PORT` environment variable | INT | `10260` | The port needs to be published, for example, using `-p 10260:10260`. |
| Specify a directory for data. | `--data-path [value]` | Overrides `DATA_PATH` environment variable. | STRING | `/data` | For example, to set `/usr/documentdb/data` as data directory, add this option to `docker run` command: `--mount type=bind,source=./.local/data,target=/usr/documentdb/data` |
| Specify the owner. | `--owner [value]` | Overrides `OWNER` environment variable. | STRING | Current container user | Specify the owner for DocumentDB. |
| Specify whether to start the PostgreSQL server. | `--start-pg [value]` | Overrides `START_POSTGRESQL` environment variable | `true`, `false` | `true` | Specify whether to start the PostgreSQL server. |
| Specify whether to create a user. | `--create-user [value]` | Overrides `CREATE_USER` environment variable | `true`, `false` | `true` | Specify whether to create a user. |
| Specify the port for the PostgreSQL server. | `--pg-port [value]` | Overrides `POSTGRESQL_PORT` environment variable | INT | `9712` | Specify the port for the PostgreSQL server. |
| Specify whether to allow external connections to PostgreSQL. | `--allow-external-connections [value]` | Overrides `ALLOW_EXTERNAL_CONNECTIONS` environment variable | `true`, `false` | `false` | Specify whether to allow external connections to PostgreSQL. |
| Specify the path to a PEM certificate for securing traffic. | `--cert-path [value]` | Overrides `CERT_PATH` environment variable. | STRING | N/A | Mount this file into the container. For example, to set `/mycert.pem`, add this option to `docker run`: `--mount type=bind,source=./mycert.pem,target=/mycert.pem`. Must be used with `--key-file`. |
| Override the default private key with a PEM key file. | `--key-file [value]` | Overrides `KEY_FILE` environment variable. | STRING | N/A | Mount this file into the container. For example, to set `/mykey.key`, add this option to `docker run`: `--mount type=bind,source=./mykey.key,target=/mykey.key`. Must be used with `--cert-path`. |
| Specify a directory containing JavaScript initialization files. | `--init-data-path [value]` | Overrides `INIT_DATA_PATH` environment variable | STRING | `/init_doc_db.d` | Files are executed in alphabetical order with `mongosh`. Using this option also skips the built-in sample data. |
| Skip initialization with built-in sample data. | `--skip-init-data` | Overrides `SKIP_INIT_DATA` environment variable | `true`, `false` | `false` | Skip creation of the built-in `sampledb` data. |
| Enable telemetry data. | `--enable-telemetry [value]` | Overrides `ENABLE_TELEMETRY` environment variable | `true`, `false` | `false` | Enable telemetry data sent to the usage collector (Azure Application Insights). |
| Specify log verbosity. | `--log-level [value]` | Overrides `LOG_LEVEL` environment variable. | `quiet`, `error`, `warn`, `info`, `debug`, `trace` | `info` | The verbosity of logs that will be emitted. |


## Feature support

See the [DocumentDB API reference](https://documentdb.io/docs/reference) for currently supported features.


## Installing certificates 

By default, DocumentDB Local generates new self-signed certificates each time the container starts. To prevent certificate errors, install them on your local machine. The example below shows how to use this setup with `mongosh`.

### Get certificate

In a `bash` window, run the following to copy the certificate from the container to the local
host: 

```bash
docker cp docdb:/home/documentdb/gateway/pg_documentdb_gw/cert.pem ~/documentdb-cert.pem
```

### Use the certificate with mongosh

```bash
mongosh localhost:10260 -u demo -p test --authenticationMechanism SCRAM-SHA-256 --tls --tlsCAFile ~/documentdb-cert.pem
```

```output
Current Mongosh Log ID:	690ce1171181053c6edbf354
Connecting to:		mongodb://<credentials>@localhost:10260/?directConnection=true&serverSelectionTimeoutMS=2000&authMechanism=SCRAM-SHA-256&tls=true&tlsCAFile=%2FUsers%2Fgeeichbe%2Fdocumentdb-cert.pem&appName=mongosh+2.5.1
Using MongoDB:		7.0.0
Using Mongosh:		2.5.1
mongosh 2.5.9 is available for download: https://www.mongodb.com/try/download/shell

For mongosh info see: https://www.mongodb.com/docs/mongodb-shell/

[direct: mongos] test>
```


## Reporting issues

If you encounter issues with using this version of DocumentDB, open an issue in the GitHub repository (<https://github.com/documentdb/documentdb/issues>) and tag it with the label `documentdb-local`.
