---
title: MongoDB shell quick start
description: Learn how to connect to DocumentDB with mongosh and run basic CRUD, aggregation, and vector search operations.
---

# MongoDB shell quick start

Use `mongosh` to connect to a local DocumentDB instance and run common MongoDB Query Language commands.

## Prerequisites

- [Docker](https://www.docker.com/products/docker-desktop)
- [`mongosh`](https://www.mongodb.com/docs/mongodb-shell/install/)
- A running `documentdb-local` container

## Start DocumentDB local

Run the `documentdb-local` container with a username and password:

```bash
docker run -dt -p 10260:10260 --name documentdb-container \
  ghcr.io/documentdb/documentdb/documentdb-local:latest \
  --username demo \
  --password test
```

The DocumentDB gateway listens on port `10260` by default.

## Connect with mongosh

Use TLS when you connect to `documentdb-local`. For quick local testing, you can allow the self-signed certificate that the container generates:

```bash
mongosh "mongodb://demo:test@localhost:10260/?tls=true&tlsAllowInvalidCertificates=true"
```

After the shell connects, switch to a database:

```javascript
use quickstart
```

## Insert documents

```javascript
db.stores.insertMany([
  {
    name: "Fourth Coffee",
    category: "coffee",
    totalSales: 1200,
    location: { type: "Point", coordinates: [-122.33, 47.61] },
    embedding: [0.12, 0.25, 0.33]
  },
  {
    name: "Tailspin Toys",
    category: "retail",
    totalSales: 850,
    location: { type: "Point", coordinates: [-122.31, 47.60] },
    embedding: [0.10, 0.20, 0.30]
  }
])
```

## Query documents

```javascript
db.stores.find({ category: "coffee" })
```

Project only selected fields:

```javascript
db.stores.find(
  { totalSales: { $gte: 500 } },
  { _id: 0, name: 1, totalSales: 1 }
)
```

## Update documents

```javascript
db.stores.updateOne(
  { name: "Fourth Coffee" },
  { $set: { status: "active" } }
)
```

## Aggregate documents

```javascript
db.stores.aggregate([
  {
    $group: {
      _id: "$category",
      totalSales: { $sum: "$totalSales" },
      storeCount: { $sum: 1 }
    }
  },
  { $sort: { totalSales: -1 } }
])
```

## Run vector search

`$vectorSearch` is an aggregation stage in DocumentDB. It must be the first stage in the aggregation pipeline and requires compatible vector data and indexing.

```javascript
db.stores.aggregate([
  {
    $vectorSearch: {
      queryVector: [0.1, 0.2, 0.3],
      path: "embedding",
      numCandidates: 100,
      limit: 10
    }
  }
])
```

## Delete documents

```javascript
db.stores.deleteOne({ name: "Tailspin Toys" })
```

## Next steps

- Review the [API reference](../api-reference/) for supported commands and operators.
- Learn how to use DocumentDB from [Node.js](nodejs-setup.md) or [Python](python-setup.md).
