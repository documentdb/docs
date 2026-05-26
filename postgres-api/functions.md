---
title: Functions
description: Reference for the PostgreSQL functions exposed by the pg_documentdb extension, including CRUD, schema, diagnostics, user and role management.
---

# Functions

`pg_documentdb` exposes its public API as PostgreSQL functions in the `documentdb_api` schema. The MongoDB wire protocol commands handled by `pg_documentdb_gw` are implemented as thin wrappers over these functions, so you can call them directly from any PostgreSQL client (e.g. `psql`, `psycopg`, JDBC).

All BSON parameters are encoded as PostgreSQL `bson` values (provided by the `pg_documentdb_core` extension). For example, you can pass a literal BSON spec using the cast `'{ ... }'::documentdb_core.bson` in `psql`.

> The function signatures below show the public `documentdb_api` surface (or `documentdb_api_internal` where noted) as defined in `pg_documentdb/sql/udfs/`. A few wire-protocol entry points are implemented as PostgreSQL `PROCEDURE`s (and must be invoked with `CALL`); these are called out individually.

## CRUD Operations

Functions for creating, reading, updating, and deleting documents.

| Function | Description |
| --- | --- |
| `documentdb_api.insert(database text, spec bson, documents bsonsequence DEFAULT NULL, transaction_id text DEFAULT NULL)` | Executes a MongoDB `insert` command. |
| `documentdb_api_internal.insert_one(collection_id bigint, shard_key_value bigint, document bson, transaction_id text)` | Internal helper to insert a single document into a known collection. |
| `documentdb_api.update(database text, spec bson, updates bsonsequence DEFAULT NULL, transaction_id text DEFAULT NULL)` | Executes a MongoDB `update` command. |
| `documentdb_api.delete(database text, spec bson, deletes bsonsequence DEFAULT NULL, transaction_id text DEFAULT NULL)` | Executes a MongoDB `delete` command. |
| `documentdb_api.find_and_modify(database text, spec bson, transaction_id text DEFAULT NULL)` | Executes a MongoDB `findAndModify` command. |
| `CALL documentdb_api.bulkWrite(command bson, ops bsonsequence DEFAULT NULL, ns_info bsonsequence DEFAULT NULL, INOUT result bson, INOUT success boolean)` | Executes a MongoDB `bulkWrite` command (added in v0.111-0). Implemented as a `PROCEDURE`. |
| `documentdb_api.find_cursor_first_page(database text, spec bson, cursor_id int8)` | Opens a cursor for a `find` command and returns its first page. |
| `documentdb_api.aggregate_cursor_first_page(database text, spec bson, cursor_id int8)` | Opens a cursor for an `aggregate` command and returns its first page. |
| `documentdb_api.list_collections_cursor_first_page(database text, spec bson, cursor_id int8)` | Opens a cursor for a `listCollections` command. |
| `documentdb_api.list_indexes_cursor_first_page(database text, spec bson, cursor_id int8)` | Opens a cursor for a `listIndexes` command. |
| `documentdb_api.cursor_get_more(database text, cursor_spec bson, continuation bson)` | Returns the next page for an existing cursor (`getMore`). |
| `documentdb_api.count_query(database text, spec bson)` | Executes a MongoDB `count` command. |
| `documentdb_api.distinct_query(database text, spec bson)` | Executes a MongoDB `distinct` command. |

## Collection and Database Management

Functions for managing collections, views, databases, and sharding.

| Function | Description |
| --- | --- |
| `documentdb_api.create_collection(database text, collection text)` | Creates a new collection in the given database. |
| `documentdb_api.create_collection_view(database text, spec bson)` | Creates a view backed by an aggregation pipeline. |
| `documentdb_api.drop_collection(database text, collection text, write_concern bson DEFAULT NULL, collection_uuid uuid DEFAULT NULL, track_changes bool DEFAULT true)` | Drops a collection. |
| `documentdb_api.rename_collection(database text, collection text, target_name text, drop_target bool DEFAULT false)` | Renames a collection. |
| `documentdb_api.drop_database(database text, write_concern bson DEFAULT NULL)` | Drops a database and all of its collections. |
| `documentdb_api.list_databases(spec bson)` | Returns information about all databases (added in v0.102-0). |
| `documentdb_api.shard_collection(database text, collection text, shard_key bson, is_reshard bool DEFAULT true)` | Shards or reshards a collection on the supplied key. |
| `documentdb_api.reshard_collection(shard_key_spec bson)` | Re-shards a collection from a wire-protocol BSON spec. |
| `documentdb_api.coll_mod(database text, collection text, spec bson)` | Executes a MongoDB `collMod` command. |
| `documentdb_api.compact(spec bson)` | Compacts a collection's data and indexes (added in v0.104-0; requires the `documentdb.enableCompact` GUC in versions prior to v0.109-0). |

## Index Management

| Function | Description |
| --- | --- |
| `documentdb_api_internal.create_indexes_non_concurrently(database text, spec bson, skip_check_collection_create bool DEFAULT false)` | Creates one or more indexes in the foreground. Lives in the internal schema. |
| `documentdb_api_internal.create_index_background(database text, spec bson)` | Schedules background index builds via the index queue (the default since v0.104-0). |
| `CALL documentdb_api.drop_indexes(database text, spec bson, INOUT retval bson DEFAULT NULL)` | Drops indexes via the wire-protocol `dropIndexes` command. Implemented as a `PROCEDURE`. |

## Diagnostic and Administration Commands

Functions backing diagnostic and administrative wire protocol commands.

| Function | Description |
| --- | --- |
| `documentdb_api.coll_stats(database text, collection text, scale float8 DEFAULT 1)` | Returns storage and index statistics for a collection (`collStats`). |
| `documentdb_api.db_stats(database text, scale float8 DEFAULT 1, free_storage bool DEFAULT false)` | Returns storage statistics for a database (`dbStats`). |
| `documentdb_api.validate(database text, validate_spec bson)` | Validates a collection's indexes (`validate`). |
| `documentdb_api.connection_status(spec bson)` | Returns authentication and role information for the current connection (`connectionStatus`, added in v0.105-0). |
| `documentdb_api.kill_op(command_spec bson)` | Cancels a running operation by op id (`killOp`, added in v0.109-0). |
| `documentdb_api.current_op_command(spec bson)` | Backs the `currentOp` wire-protocol command (added in v0.101-0). The `$currentOp` aggregation stage uses the internal `documentdb_api_internal.current_op_aggregation(spec bson)` set-returning helper. |

## User Management

Functions for creating, updating, and managing database users. Backed by the wire protocol `createUser`, `dropUser`, `updateUser`, and `usersInfo` commands.

| Function | Description |
| --- | --- |
| `documentdb_api.create_user(spec bson)` | Creates a new user. |
| `documentdb_api.drop_user(spec bson)` | Drops an existing user. |
| `documentdb_api.update_user(spec bson)` | Updates an existing user (password, roles, etc.). |
| `documentdb_api.users_info(spec bson)` | Returns information about one or more users. |

## Role Management

Role management is supported since v0.106-0 (`createRole`, `rolesInfo`) and v0.108-0 (`dropRole`).

| Function | Description |
| --- | --- |
| `documentdb_api.create_role(spec bson)` | Creates a new role. |
| `documentdb_api.drop_role(spec bson)` | Drops an existing role. |
| `documentdb_api.update_role(spec bson)` | Updates an existing role's privileges or inherited roles. |
| `documentdb_api.roles_info(spec bson)` | Returns information about one or more roles. |

## Utility Functions

| Function | Description |
| --- | --- |
| `documentdb_api.binary_version()` | Returns the running binary version of the `pg_documentdb` shared library. |
| `documentdb_api.binary_extended_version()` | Returns the extended binary version, including build metadata. |

## Example

To call any of these directly from `psql`:

```sql
SELECT documentdb_api.insert(
    'mydb',
    '{ "insert": "users", "documents": [ { "name": "alice" } ] }'::documentdb_core.bson
);
```

For more complete examples (cursors, aggregation, sharding) see the [API Reference](https://documentdb.io/docs/api-reference).
