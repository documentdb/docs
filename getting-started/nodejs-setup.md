---
title: Node.js Setup Guide
description: Learn how to set up and use DocumentDB with Node.js using the official MongoDB Node.js driver.
---

# Node.js Setup Guide

Learn how to set up and use DocumentDB with Node.js using the official MongoDB Node.js driver.

## Prerequisites

- Node.js 20.19 or later (required by the current `mongodb` driver)
- npm or yarn package manager
- DocumentDB installed and running
- Docker installed (if set up is not completed yet)
- Basic Node.js knowledge

## Project Setup (skip if already done)

Before connecting from Node.js, make sure you have a running DocumentDB instance using Docker:

   ```bash
   # Pull the latest DocumentDB Docker image
   docker pull ghcr.io/documentdb/documentdb/documentdb-local:latest

   # Tag the image for convenience
   docker tag ghcr.io/documentdb/documentdb/documentdb-local:latest documentdb

   read -r -p 'DocumentDB username: ' DOCUMENTDB_USERNAME
   read -r -s -p 'DocumentDB password: ' DOCUMENTDB_PASSWORD
   printf '\n'
   export DOCUMENTDB_USERNAME DOCUMENTDB_PASSWORD

   # Run the container with your chosen username and password
   if docker run -dt -p 127.0.0.1:10260:10260 --name documentdb-container documentdb \
     --username "${DOCUMENTDB_USERNAME:?DocumentDB username cannot be empty}" \
     --password "${DOCUMENTDB_PASSWORD:?DocumentDB password cannot be empty}"; then
     docker image rm -f ghcr.io/documentdb/documentdb/documentdb-local:latest
   fi
   ```
> **Note:** During the transition to the Linux Foundation, Docker images may still be hosted on Microsoft's container registry. These will be migrated to the new DocumentDB organization as the transition completes.
>
> **Note:** The prompts export the credentials for the Node.js example below. The guards reject empty values so the container cannot fall through to the public `default_user` / `Admin100` defaults. If you skip this Docker setup or open a new shell, set both environment variables before running Node.js.
>
> **Readiness Note:** `docker ps` reports the container as `Up` before DocumentDB can accept connections. Wait for the ready banner first: `until docker logs documentdb-container 2>&1 | grep -q "=== DocumentDB is ready ==="; do sleep 2; done`
>
> **Network Note:** The example binds the gateway only to the local host. Expose it to other machines only after adding firewall rules and a certificate those clients can validate.
>
> **Port Note:** To use host port `27017` while leaving the gateway on its default container port, publish `-p 127.0.0.1:27017:10260` and connect to `localhost:27017`. To change the gateway's internal port too, add `--documentdb-port 27017` after the image name and publish that container port.

## Installation

1. Creating a new Node.js project
   ```bash
   mkdir my-documentdb-app
   cd my-documentdb-app
   npm init -y
   ```

2. Installing the MongoDB driver
   ```bash
   npm install mongodb
   ```

## Connecting to DocumentDB

DocumentDB Local accepts TLS connections on the gateway port and requires authentication. Connect with the username and password you set when starting the container, and because the container uses a self-signed certificate, the simplest local setup skips certificate validation with `tlsAllowInvalidCertificates=true` (in production, provide the gateway certificate instead).

The code reads the same `DOCUMENTDB_USERNAME` and `DOCUMENTDB_PASSWORD` values exported during Docker setup and raises a clear error if either is missing.

```javascript
const { MongoClient } = require('mongodb');

const username = process.env.DOCUMENTDB_USERNAME;
const password = process.env.DOCUMENTDB_PASSWORD;
if (!username || !password) {
  throw new Error('Set DOCUMENTDB_USERNAME and DOCUMENTDB_PASSWORD before connecting');
}

const client = new MongoClient(
  'mongodb://localhost:10260/?authSource=admin&tls=true&tlsAllowInvalidCertificates=true&directConnection=true',
  {
    auth: { username, password },
  },
);

async function main() {
  await client.connect();
  const db = client.db('your_database');
  console.log('connected');
  await client.close();
}

main().catch((error) => {
  console.error('Connection error:', error);
  process.exit(1);
});
```

## Basic Operations

Replace the earlier `main()` function and its call with the example below. The
operations all run inside `main()`, after `const db = client.db(...)`.
`await` is only valid inside an `async` function, and `db` only exists in that scope —
running these at the top level of a file gives `ReferenceError: db is not defined`.

```javascript
async function main() {
  await client.connect();
  const db = client.db('your_database');
  const users = db.collection('users');

  await users.insertOne({ name: 'John Doe', email: 'john@example.com', createdAt: new Date() });
  await users.insertMany([
    { name: 'Jane Smith', email: 'jane@example.com' },
    { name: 'Bob Johnson', email: 'bob@example.com' },
  ]);

  await users.updateOne({ name: 'John Doe' }, { $set: { status: 'active' } });
  console.log(await users.findOne({ name: 'John Doe' }));
  console.log(await users.countDocuments());

  await users.deleteOne({ name: 'Bob Johnson' });

  await client.close();
}

main().catch((error) => {
  console.error(error);
  process.exit(1);
});
```

## Beyond CRUD

Aggregation pipelines, vector search, geospatial queries, and change streams use the
same syntax as the MongoDB shell. See the
[Mongo Shell Quick Start](https://documentdb.io/docs/getting-started/mongo-shell-quickstart/)
for worked examples, and the [API reference](https://documentdb.io/docs/reference/)
for the supported operator set.

## Next Steps

- Explore advanced features
- Learn about indexing strategies
- Build your first application 
