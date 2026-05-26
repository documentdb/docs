---
title: MongoDB Shell Quick Start
description: Get started with DocumentDB using the MongoDB shell (mongosh) for a familiar MongoDB-compatible experience.
---

# MongoDB Shell Quick Start

Get started with DocumentDB using the MongoDB shell (`mongosh`) for a familiar MongoDB-compatible experience.

## Prerequisites

- [MongoDB Shell (`mongosh`)](https://www.mongodb.com/try/download/shell) installed
- [Docker Desktop](https://www.docker.com/) installed and running
- Basic MongoDB knowledge

## Setting up DocumentDB locally

Pull the latest `documentdb-local` image and start the container. DocumentDB Local listens on port `10260` by default and requires the username and password to be set on first run.

```bash
# Pull the latest DocumentDB Docker image
docker pull ghcr.io/documentdb/documentdb/documentdb-local:latest

# Tag the image for convenience
docker tag ghcr.io/documentdb/documentdb/documentdb-local:latest documentdb

# Run the container with your chosen username and password
docker run -dt -p 10260:10260 --name documentdb-container documentdb --username <YOUR_USERNAME> --password <YOUR_PASSWORD>
```

> **Note:** Replace `<YOUR_USERNAME>` and `<YOUR_PASSWORD>` with your desired credentials. These must be set when creating the container for authentication to work.
>
> **Port note:** Port `10260` is used by default to avoid conflicts with other local database services. You can use port `27017` (the standard MongoDB port) or any other available port — update the port in the `docker run` command and your connection string accordingly.

Confirm the container is running:

```bash
docker ps
```

## Connecting to DocumentDB

DocumentDB Local terminates TLS on the gateway port. The container generates a new self-signed certificate on each start, so the simplest local connection skips certificate validation with `tlsAllowInvalidCertificates=true`.

```bash
mongosh "mongodb://<YOUR_USERNAME>:<YOUR_PASSWORD>@localhost:10260/?tls=true&tlsAllowInvalidCertificates=true"
```

For instructions on installing the generated certificate so you can validate it normally, see [DocumentDB Local](https://documentdb.io/docs/documentdb-local).

## Basic Operations

### Create a database and collection

```javascript
// Switch to (or create) a database
use mydb

// Create a collection
db.createCollection("users")
```

### Insert documents

```javascript
// Insert a single document
db.users.insertOne({
  name: "John Doe",
  email: "john@example.com",
  created_at: new Date()
})

// Insert multiple documents
db.users.insertMany([
  { name: "Jane Smith", email: "jane@example.com" },
  { name: "Bob Johnson", email: "bob@example.com" }
])
```

### Query documents

```javascript
// Find all documents
db.users.find()

// Find with a filter
db.users.find({ name: "John Doe" })

// Find with a projection
db.users.find({}, { name: 1, email: 1, _id: 0 })

// Combine multiple operators
db.users.find({
  $and: [
    { created_at: { $gte: new Date("2026-01-01") } },
    { email: { $regex: "@example.com$" } }
  ]
})
```

### Update documents

```javascript
// Update a single document
db.users.updateOne(
  { name: "John Doe" },
  { $set: { status: "active" } }
)

// Update many documents
db.users.updateMany(
  { email: { $regex: "@example.com$" } },
  { $set: { domain: "example.com" } }
)
```

### Delete documents

```javascript
// Delete a single document
db.users.deleteOne({ name: "John Doe" })

// Delete many documents
db.users.deleteMany({ status: "inactive" })
```

## Working with Indexes

DocumentDB supports many MongoDB-compatible index types, including single-field, compound, multi-key, partial, unique, text, geospatial, and vector indexes.

```javascript
// Single field index
db.users.createIndex({ email: 1 })

// Compound index
db.users.createIndex({ name: 1, email: 1 })

// Unique index
db.users.createIndex({ email: 1 }, { unique: true })

// Text index
db.articles.createIndex({ content: "text" })

// Geospatial (2dsphere) index
db.places.createIndex({ location: "2dsphere" })

// Partial index
db.orders.createIndex(
  { orderDate: 1 },
  { partialFilterExpression: { status: "active" } }
)
```

To create a vector index on an embedding field, use the `cosmosSearchOptions` index spec accepted by the DocumentDB gateway:

```javascript
db.products.createIndex(
  { embedding: "cosmosSearch" },
  {
    name: "vectorIndex",
    cosmosSearchOptions: {
      kind: "vector-ivf",
      numLists: 100,
      similarity: "COS",
      dimensions: 384
    }
  }
)
```

## Aggregation Pipelines

```javascript
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $group: {
      _id: "$customer",
      total: { $sum: "$amount" },
      count: { $sum: 1 }
  }},
  { $sort: { total: -1 } }
])
```

DocumentDB also supports stages such as `$lookup`, `$unwind`, `$facet`, `$bucket`, `$bucketAuto`, and many others. See the [API Reference](https://documentdb.io/docs/api-reference) for the full list.

## Vector Search

```javascript
db.products.aggregate([
  {
    $search: {
      cosmosSearch: {
        vector: [0.1, 0.2, 0.3],
        path: "embedding",
        k: 10
      }
    }
  }
])
```

## Geospatial Queries

```javascript
db.places.find({
  location: {
    $near: {
      $geometry: { type: "Point", coordinates: [-73.9667, 40.78] },
      $maxDistance: 1000
    }
  }
})
```

## Diagnostics and Administration

DocumentDB ships with a number of MongoDB-compatible administrative commands:

```javascript
// Build info and server info
db.runCommand({ buildInfo: 1 })
db.runCommand({ hello: 1 })

// Database and collection statistics
db.stats()
db.users.stats()

// List databases (admin DB)
db.adminCommand({ listDatabases: 1 })

// Inspect or kill currently running operations
db.currentOp()
db.killOp(<opid>)

// Validate a collection
db.users.validate()

// Compact a collection
db.runCommand({ compact: "users" })
```

User and role management commands are also supported:

```javascript
// Users
db.runCommand({ createUser: "alice", pwd: "secret", roles: [ { role: "readWrite", db: "mydb" } ] })
db.runCommand({ usersInfo: 1 })

// Roles
db.runCommand({ createRole: "appWriter", privileges: [], roles: [ "readWrite" ] })
db.runCommand({ rolesInfo: 1 })
```

## Best Practices

- **Connection pooling:** reuse a single `mongosh` connection per session.
- **Indexes:** create indexes that match your most frequent query and sort patterns. Use `db.collection.getIndexes()` to inspect existing indexes.
- **Explain plans:** prefix queries with `.explain("executionStats")` to inspect how DocumentDB plans and executes them.
- **TLS:** in production, always provide the gateway certificate via `--tlsCAFile` rather than disabling validation.

## Next Steps

- Browse the [API Reference](https://documentdb.io/docs/api-reference) for the full list of supported commands, operators, and aggregation stages.
- Connect from your application using the [Python](https://documentdb.io/docs/getting-started/python-setup) or [Node.js](https://documentdb.io/docs/getting-started/nodejs-setup) setup guides.
- Use the [Visual Studio Code extension](https://documentdb.io/docs/getting-started/vscode-extension-guide) for a GUI experience over the same gateway.
