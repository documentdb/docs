---
title: Azure setup
description: Learn deployment options for running open-source DocumentDB on Azure.
---

# Run DocumentDB on Azure

You can run open-source DocumentDB on Azure infrastructure for development, testing, or self-managed deployments.

## Choose a deployment option

| Option | Use when |
| --- | --- |
| [`documentdb-local`](../documentdb-local/index.md) on an Azure virtual machine | You want the fastest way to test DocumentDB on Azure infrastructure. |
| [Pre-built Linux packages](prebuilt-packages.md) on an Azure virtual machine | You want a self-managed DocumentDB deployment with explicit control over PostgreSQL and extension installation. |
| Azure Cosmos DB for MongoDB | You want an Azure managed MongoDB-compatible database service. This service is separate from the open-source DocumentDB project. |

## Avoid product confusion

Open-source DocumentDB isn't the same product as Azure Cosmos DB for MongoDB. If you need to deploy open-source DocumentDB, use the `documentdb-local` image or the Linux packages. If you need a fully managed Azure database service, evaluate [Azure Cosmos DB for MongoDB](https://learn.microsoft.com/azure/cosmos-db/mongodb/introduction).
