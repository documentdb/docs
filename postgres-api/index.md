---
title: Components
description: Learn about pg_documentdb_core and pg_documentdb_api PostgreSQL extensions that enable BSON support and document operations in Postgres.
---

# Components

The DocumentDB implementation consists of two key PostgreSQL extensions that work together to provide MongoDB-compatible document database functionality within PostgreSQL.

## pg_documentdb_core

pg_documentdb_core is a PostgreSQL extension that introduces BSON datatype support and operations for native Postgres. This core component is essential for enabling document-oriented NoSQL capabilities within a PostgreSQL environment. It provides the foundational data structures and functions required to handle BSON data types, which are crucial for performing CRUD operations on documents.

### Key Features

- **BSON Datatype Support:** Adds BSON (Binary JSON) datatype to PostgreSQL, allowing for efficient storage and manipulation of JSON-like documents.

- **Native Operations:** Implements native PostgreSQL operations for BSON data, ensuring seamless integration and performance.

- **Extensibility:** Serves as the core building block for additional functionalities and extensions within the DocumentDB ecosystem.

## pg_documentdb_api

pg_documentdb_api is the public API surface for DocumentDB, providing CRUD functionality on documents stored in the database. This component leverages the capabilities of pg_documentdb_core to offer a comprehensive set of APIs for managing document data within PostgreSQL.

### Key Features

- **CRUD Operations:** Provides a rich set of APIs for creating, reading, updating, and deleting documents.

- **Advanced Queries:** Supports complex queries, including full-text searches, geospatial queries, and vector embeddings.

- **Integration:** Works seamlessly with pg_documentdb_core to deliver robust document management capabilities.

### Usage

To use pg_documentdb_api, you need to have pg_documentdb_core installed and configured in your PostgreSQL environment. Once set up, you can leverage the APIs provided by pg_documentdb_api to perform various document operations.
