---
title: Architecture under the hood
description: Deep dive into DocumentDB's internal architecture, data structures, query processing, storage engine design, and distributed systems.
---

# Architecture under the hood

DocumentDB is a MongoDB-compatible open-source document database built on PostgreSQL. It combines PostgreSQL extensions for BSON storage and query execution with a gateway that accepts MongoDB wire protocol requests.

## Core components

| Component | Role |
| --- | --- |
| `pg_documentdb_core` | PostgreSQL extension that adds BSON data type support and core BSON operations. |
| `pg_documentdb` | PostgreSQL extension that exposes the DocumentDB API surface for document operations, indexing, aggregation, authentication, and metadata management. |
| `pg_documentdb_gw` | Gateway that accepts MongoDB wire protocol requests and translates them into DocumentDB API calls. |
| `documentdb-local` | Container image that packages the gateway, PostgreSQL, and DocumentDB extensions for local development and testing. |

## Request flow

1. A MongoDB driver or `mongosh` sends a request to the DocumentDB gateway.
2. The gateway parses the MongoDB wire protocol command and maps it to the matching DocumentDB API operation.
3. The PostgreSQL extensions execute the operation against BSON documents stored in PostgreSQL tables.
4. DocumentDB returns a MongoDB-compatible response to the client.

## Related documentation

- [DocumentDB local](../documentdb-local/index.md)
- [PostgreSQL API](../postgres-api/index.md)
- [API reference](../api-reference/)
