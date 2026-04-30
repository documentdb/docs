---
title: MongoDB migration guide
description: Learn how to assess and migrate supported MongoDB workloads to DocumentDB.
---

# MongoDB to DocumentDB migration guide

Use this guide to assess MongoDB workloads before you move them to DocumentDB.

## Overview

DocumentDB provides MongoDB wire protocol compatibility for supported commands and operators. Before you migrate, review the [API reference](../api-reference/) and test representative application queries against a DocumentDB environment.

## Migration checklist

1. Inventory the commands, operators, indexes, and driver options that your application uses.
2. Compare that inventory with the [API reference](../api-reference/).
3. Run integration tests against [`documentdb-local`](../documentdb-local/index.md) or a self-managed DocumentDB deployment.
4. Update connection strings to use the DocumentDB endpoint, TLS settings, and credentials for your environment.
5. Validate performance-sensitive queries and indexes with production-like data volumes.

## Common compatibility areas to test

| Area | What to validate |
| --- | --- |
| Commands and operators | Confirm that each command and operator used by your application is documented as supported. |
| Indexes | Recreate required indexes and validate query plans for critical paths. |
| Aggregation pipelines | Test complex pipelines, especially stages that depend on sorting, geospatial data, vector search, or window functions. |
| Authentication and TLS | Confirm that your driver connection string includes the required credentials and TLS options. |

## Next steps

- Start a local environment with [DocumentDB local](../documentdb-local/index.md).
- Connect an application with the [Node.js setup guide](nodejs-setup.md) or [Python setup guide](python-setup.md).
