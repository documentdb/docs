---
title: Components
description: Learn about pg_documentdb_core, pg_documentdb, and pg_documentdb_gw — the PostgreSQL extensions that enable BSON support, document operations, and MongoDB wire protocol compatibility in Postgres.
---

# Components

The DocumentDB implementation consists of three PostgreSQL extensions that work together to provide MongoDB-compatible document database functionality on top of PostgreSQL.

## pg_documentdb_core

`pg_documentdb_core` is a PostgreSQL extension that introduces BSON datatype support and operations for native Postgres. This core component is essential for enabling document-oriented NoSQL capabilities within a PostgreSQL environment. It provides the foundational data structures and functions required to handle BSON data types, which are crucial for performing CRUD operations on documents.

### Key Features

- **BSON Datatype Support:** Adds BSON (Binary JSON) datatype to PostgreSQL, allowing for efficient storage and manipulation of JSON-like documents.

- **Native Operations:** Implements native PostgreSQL operations for BSON data, ensuring seamless integration and performance.

- **Extensibility:** Serves as the core building block for additional functionalities and extensions within the DocumentDB ecosystem.

## pg_documentdb

`pg_documentdb` is the public API surface for DocumentDB, providing CRUD functionality on documents stored in the database. This component leverages the capabilities of `pg_documentdb_core` to offer a comprehensive set of APIs for managing document data within PostgreSQL.

### Key Features

- **CRUD Operations:** Provides a rich set of APIs for creating, reading, updating, and deleting documents.

- **Advanced Queries:** Supports complex queries, including full-text searches, geospatial queries, vector search, and aggregation pipelines.

- **Index Management:** Single-field, compound, multi-key, partial, unique, TTL, text, geospatial, and vector indexes.

- **User and Role Management:** Built-in commands for managing users, roles, and SCRAM authentication.

- **Integration:** Works seamlessly with `pg_documentdb_core` to deliver robust document management capabilities.

### Usage

To use `pg_documentdb`, you need to have `pg_documentdb_core` installed and configured in your PostgreSQL environment. Once set up, you can leverage the APIs provided by `pg_documentdb` to perform document operations from any PostgreSQL client. For the full list of callable functions, see [Functions](functions.md).

## pg_documentdb_gw

`pg_documentdb_gw` is the gateway protocol translation layer that converts the user's MongoDB wire protocol commands into PostgreSQL queries against the `pg_documentdb` API. It is what allows MongoDB-compatible drivers and tools (`mongosh`, PyMongo, the official MongoDB Node.js driver, the VS Code extension, and others) to talk to a DocumentDB instance unmodified.

### Key Features

- **MongoDB Wire Protocol:** Parses MongoDB wire protocol messages (`OP_MSG`, `OP_QUERY`, `OP_INSERT`, etc.) and dispatches them to the corresponding `pg_documentdb` SQL functions.

- **Authentication:** Supports SCRAM-SHA-256 and Plain authentication (including EntraId token-based Plain Auth introduced in v0.106-0).

- **TLS Termination:** Terminates TLS on the gateway port (default `10260`), allowing drivers to connect over the standard MongoDB-style `mongodb://` connection string with `tls=true`.

- **Cursor and Session Management:** Translates `getMore`, `killCursors`, `killSessions`, and `killOp` commands to the corresponding backend operations.

- **Deployment Options:** Can be run as a standalone process (the default in the `documentdb-local` container) or, since v0.108-0, as a PostgreSQL background worker via the `pg_documentdb_gw_host` extension.
