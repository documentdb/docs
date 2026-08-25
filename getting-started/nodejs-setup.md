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

   # Run the container with your chosen username and password
   docker run -dt -p 10260:10260 --name documentdb-container documentdb --username <YOUR_USERNAME> --password <YOUR_PASSWORD>
   docker image rm -f ghcr.io/documentdb/documentdb/documentdb-local:latest
   ```
> **Note:** During the transition to the Linux Foundation, Docker images may still be hosted on Microsoft's container registry. These will be migrated to the new DocumentDB organization as the transition completes.
>
> **Note:** Replace `<YOUR_USERNAME>` and `<YOUR_PASSWORD>` with your desired credentials. Always set them explicitly: if you omit them the container starts with the built-in `default_user` / `Admin100`, which are public and let anyone who can reach the published port authenticate as the admin user.
>
> **Readiness Note:** `docker ps` reports the container as `Up` before DocumentDB can accept connections. Wait for the ready banner first: `until docker logs documentdb-container 2>&1 | grep -q "=== DocumentDB is ready ==="; do sleep 2; done`
>
> **Port Note:** Port `10260` is used by default in these instructions to avoid conflicts with other local database services. You can use port `27017` (the standard MongoDB port) or any other available port if you prefer. If you do, be sure to update the port number in both your `docker run` command and your connection string accordingly.

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

```javascript
const { MongoClient } = require('mongodb');

const uri = 'mongodb://<YOUR_USERNAME>:<YOUR_PASSWORD>@localhost:10260/?tls=true&tlsAllowInvalidCertificates=true';
const client = new MongoClient(uri);

async function main() {
  await client.connect();
  const db = client.db('your_database');
  console.log('connected');
  return db;
}

main().catch((error) => {
  console.error('Connection error:', error);
  process.exit(1);
});
```

## Basic Operations

The operations below all run inside `main()`, after `const db = client.db(...)` above.
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

Aggregation pipelines, vector search, geospatial queries and change streams use the
same syntax as the MongoDB shell. See the
[Mongo Shell Quick Start](https://documentdb.io/docs/getting-started/mongo-shell-quickstart/)
for worked examples, and the [API reference](https://documentdb.io/docs/api-reference/)
for the supported operator set.

## Next Steps

- Explore advanced features
- Learn about indexing strategies
- Build your first application 
