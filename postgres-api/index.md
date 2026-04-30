---
title: Components
description: Learn about the pg_documentdb_core, pg_documentdb, and pg_documentdb_gw components that enable BSON support, document operations, and MongoDB wire protocol access.
---

# Components

The DocumentDB implementation consists of PostgreSQL extensions plus a Rust gateway that work together to provide MongoDB-compatible document database functionality on PostgreSQL.

## pg_documentdb_core

pg_documentdb_core is a PostgreSQL extension that introduces BSON datatype support and operations for native Postgres. This core component is essential for enabling document-oriented NoSQL capabilities within a PostgreSQL environment. It provides the foundational data structures and functions required to handle BSON data types, which are crucial for performing CRUD operations on documents.

### Key Features

- **BSON Datatype Support:** Adds BSON (Binary JSON) datatype to PostgreSQL, allowing for efficient storage and manipulation of JSON-like documents.

- **Native Operations:** Implements native PostgreSQL operations for BSON data, ensuring seamless integration and performance.

- **Extensibility:** Serves as the core building block for additional functionalities and extensions within the DocumentDB ecosystem.

## pg_documentdb

pg_documentdb is the public API extension for DocumentDB, providing CRUD functionality on documents stored in the database. This component leverages pg_documentdb_core and exposes functions through schemas such as `documentdb_api`, `documentdb_api_catalog`, and `documentdb_api_internal`.

### Key Features

- **CRUD Operations:** Provides a rich set of APIs for creating, reading, updating, and deleting documents.

- **Advanced Queries:** Supports complex queries, including full-text searches, geospatial queries, and vector embeddings.

- **Integration:** Works with pg_documentdb_core to deliver document management capabilities.

## pg_documentdb_gw

pg_documentdb_gw is the Rust gateway that accepts MongoDB wire protocol connections and dispatches commands to the PostgreSQL extension functions.

### Usage

To use DocumentDB through PostgreSQL, install the `documentdb_core` and `documentdb` extensions. To connect with MongoDB drivers, run the `pg_documentdb_gw` gateway in front of those extensions.
