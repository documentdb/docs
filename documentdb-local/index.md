# DocumentDB-local
DocumentDB is available as a Docker container. It supports running on a wide variety of processors and operating systems and make it easy to try out and test DocumentDB.


## Prerequisites

- [Docker](https://www.docker.com/)

## Installation

Get the Docker container image using `docker pull`. The container image is published to the Github container registry as `ghcr.io/documentdb/documentdb/documentdb-local:latest`.

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

> The DocumenttDB Mongo protocol gateway endpoint is typically available on port `10260`. To access this with `mongosh` run:

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

To be continued...
## Feature support

Please refer to the documentdb documentation for currently supported features,


## Limitations

In addition to features not yet supported or not planned, the following list includes current limitations of docuemntdb-lcoal:

To be added.


## Installing certificates 

DocumentDB-local by default will generate new selef signed certificates each tome you start the container. To avoid certificate errors these can be installed on the local host. The example belw will show how to use this with `mongosh`.

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

## Use in continuous integration workflow
TBD

## Reporting issues

If you encounter issues with using this version of DoucmenttDB, open an issue in the GitHub repository (<https://github.com/documentdb/documentdb/issues>) and tag it with the label `documentdb-local`.