---
title: Functions
description: Reference for the PostgreSQL functions exposed by the pg_documentdb extension in v0.114-0, including CRUD, schema, index, diagnostics, user and role management.
---

# Functions

`pg_documentdb` exposes its public API as PostgreSQL functions in the `documentdb_api` schema. The MongoDB wire protocol commands handled by `pg_documentdb_gw` are implemented as thin wrappers over these functions, so you can call them directly from any PostgreSQL client (e.g. `psql`, `psycopg`, JDBC).

All BSON parameters are encoded as PostgreSQL `bson` values (provided by the `pg_documentdb_core` extension). For example, you can pass a literal BSON spec using the cast `'{ ... }'::documentdb_core.bson` in `psql`.

> **This page describes v0.114-0.** It has not yet been revised for the current release (v0.116-0); behavior added upstream after the v0.114-0 tag is not documented here.

> **Reading the signatures below.** Parameter names are the ones the extension actually declares in `pg_documentdb/sql/udfs/`, so they are safe to use in named-argument calls such as `p_database_name => 'mydb'`. They are unquoted identifiers, so PostgreSQL folds them to lower case — `\df documentdb_api.*` prints `commandspec` where the source writes `commandSpec`, and `bigint`/`boolean`/`double precision` where the source writes `int8`/`bool`/`float8`. Either spelling works in a call. Types are written unqualified: `bson` and `bsonsequence` live in `documentdb_core`. A function with **two or more** `OUT` parameters returns a `record` and should be called as `SELECT * FROM ...`; one with a single `OUT` parameter returns that parameter's type directly, so a plain `SELECT fn(...)` is fine. A few wire-protocol entry points are PostgreSQL `PROCEDURE`s and must be invoked with `CALL`; these are called out individually.

## CRUD Operations

Functions for creating, reading, updating, and deleting documents.

| Function | Description |
| --- | --- |
| `documentdb_api.insert(p_database_name text, p_insert bson, p_insert_documents bsonsequence DEFAULT NULL, p_transaction_id text DEFAULT NULL, OUT p_result bson, OUT p_success boolean)` | Executes a MongoDB `insert` command. |
| `documentdb_api.update(p_database_name text, p_update bson, p_insert_documents bsonsequence DEFAULT NULL, p_transaction_id text DEFAULT NULL, OUT p_result bson, OUT p_success boolean)` | Executes a MongoDB `update` command. |
| `documentdb_api.delete(p_database_name text, p_delete bson, p_insert_documents bsonsequence DEFAULT NULL, p_transaction_id text DEFAULT NULL, OUT p_result bson, OUT p_success boolean)` | Executes a MongoDB `delete` command. |
| `documentdb_api.find_and_modify(p_database_name text, p_message bson, p_transaction_id text DEFAULT NULL, OUT p_result bson, OUT p_success boolean)` | Executes a MongoDB `findAndModify` command. |
| `documentdb_api.insert_one(p_database_name text, p_collection_name text, p_document bson, p_transaction_id text DEFAULT NULL)` | Convenience wrapper that inserts a single document by database and collection name and returns the `insert` response. |
| `documentdb_api_internal.insert_one(p_collection_id bigint, p_shard_key_value bigint, p_document bson, p_transaction_id text)` | **Deprecated and non-functional.** The implementation is a stub that always raises `insert_one is deprecated and should not be called`. Use the public `documentdb_api.insert_one` above. |
| `CALL documentdb_api.bulkWrite(p_command bson, p_ops bsonsequence DEFAULT NULL, p_ns_info bsonsequence DEFAULT NULL, INOUT p_result bson DEFAULT NULL, INOUT p_success boolean DEFAULT NULL)` | **Command surface only — not implemented in v0.114-0.** The `PROCEDURE` exists (its SQL definition landed in v0.111-0) but every call raises `bulkWrite is not yet implemented`, and it also rejects being called inside a transaction. |
| `documentdb_api.find_cursor_first_page(database text, commandSpec bson, cursorId int8 DEFAULT 0, OUT cursorPage bson, OUT continuation bson, OUT persistConnection bool, OUT cursorId int8)` | Opens a cursor for a `find` command and returns its first page. Omit `cursorId` to have the server generate one. |
| `documentdb_api.aggregate_cursor_first_page(database text, commandSpec bson, cursorId int8 DEFAULT 0, OUT cursorPage bson, OUT continuation bson, OUT persistConnection bool, OUT cursorId int8)` | Opens a cursor for an `aggregate` command and returns its first page. Omit `cursorId` to have the server generate one. |
| `documentdb_api.list_collections_cursor_first_page(database text, commandSpec bson, cursorId int8 DEFAULT 0, OUT cursorPage bson, OUT continuation bson, OUT persistConnection bool, OUT cursorId int8)` | Returns the `listCollections` result. Despite the name this is always a single batch — see the note below. |
| `documentdb_api.list_indexes_cursor_first_page(database text, commandSpec bson, cursorId int8 DEFAULT 0, OUT cursorPage bson, OUT continuation bson, OUT persistConnection bool, OUT cursorId int8)` | Returns the `listIndexes` result. Always a single batch — see the note below. |
| `documentdb_api.cursor_get_more(database text, getMoreSpec bson, continuationSpec bson, OUT cursorPage bson, OUT continuation bson)` | Returns the next page for an existing cursor (`getMore`). |
| `documentdb_api.count_query(database text, countSpec bson, OUT document bson)` | Executes a MongoDB `count` command. |
| `documentdb_api.distinct_query(database text, distinctSpec bson, OUT document bson)` | Executes a MongoDB `distinct` command. |
| `documentdb_api.collection(p_database_name text, p_collection_name text, OUT shard_key_value bigint, OUT object_id bson, OUT document bson)` | Set-returning function that reads a collection's raw rows — see the note below for its two usage restrictions. |

> **The four `*_cursor_first_page` functions are `STRICT`.** Passing an explicit `NULL` for `cursorId` (which a prepared-statement client may do when it cannot omit a positional argument) returns a single all-`NULL` row with no error and no cursor. Omit the argument instead of binding `NULL`.

> **`list_collections_cursor_first_page` and `list_indexes_cursor_first_page` ignore `cursorId`.** Both hardcode a single-batch query, so `OUT cursorId` always comes back `0` and `OUT continuation` always `NULL`; the result cannot be resumed with `cursor_get_more`. Only `find` and `aggregate` produce resumable cursors.

> **`documentdb_api.collection` only works as a FROM-clause table function with literal names.** A planner hook rewrites `SELECT * FROM documentdb_api.collection('mydb', 'users')` into a scan of the backing table. If either name argument is not a constant (a column reference, a correlated subquery, a PL/pgSQL variable) the rewrite is skipped and the direct call raises `Collection function should be only used in a FROM clause`. Naming a collection that does not exist is fine — it rewrites to an empty scan and returns zero rows.

### Write procedures

`insert` and `update` are also exposed as `PROCEDURE`s. `delete` has no procedure form in v0.114-0.

| Procedure | Description |
| --- | --- |
| `CALL documentdb_api.insert_bulk(p_database_name text, p_insert bson, p_insert_documents bsonsequence DEFAULT NULL, p_transaction_id text DEFAULT NULL, INOUT p_result bson DEFAULT NULL, INOUT p_success boolean DEFAULT NULL)` | Inserts documents, committing after each sub-batch rather than as one unit. |
| `CALL documentdb_api.update_bulk(p_database_name text, p_update bson, p_insert_documents bsonsequence DEFAULT NULL, p_transaction_id text DEFAULT NULL, INOUT p_result bson DEFAULT NULL, INOUT p_success boolean DEFAULT NULL)` | `update` equivalent of `insert_bulk`. |
| `CALL documentdb_api.insert_txn_proc(p_database_name text, p_insert bson, p_insert_documents bsonsequence DEFAULT NULL, p_transaction_id text DEFAULT NULL, INOUT p_result bson DEFAULT NULL, INOUT p_success boolean DEFAULT NULL)` | Inserts documents without opening a subtransaction per statement. |
| `CALL documentdb_api.update_txn_proc(p_database_name text, p_update bson, p_insert_documents bsonsequence DEFAULT NULL, p_transaction_id text DEFAULT NULL, INOUT p_result bson DEFAULT NULL, INOUT p_success boolean DEFAULT NULL)` | `update` equivalent of `insert_txn_proc`. |

> **None of these procedures may be called inside an explicit transaction block.** Each one raises `the insert procedure cannot be used in transactions. Please use the insert function instead` (`ERRCODE_DOCUMENTDB_OPERATIONNOTSUPPORTEDINTRANSACTION`) when it detects an open transaction. They differ from each other only in commit granularity, not in transaction support: `_bulk` commits per sub-batch, `_txn_proc` skips the per-statement subtransaction. Inside a transaction, use the plain `insert`/`update` functions.

> **The gateway does not use these by default.** Both are gated by gateway settings that default to `false` — `enableWriteProcedures` selects the `_txn_proc` variants and `enableWriteProceduresWithBatchCommit` selects the `_bulk` variants. Out of the box every wire-protocol write goes through `documentdb_api.insert` / `documentdb_api.update`.

## Collection and Database Management

Functions for managing collections, views, databases, and sharding.

| Function | Description |
| --- | --- |
| `documentdb_api.create_collection(p_database_name text, p_collection_name text)` | Creates an empty collection with no options. Returns `true` if it was created, `false` if it already existed. |
| `documentdb_api.create_collection_view(dbname text, createSpec bson)` | Backs the wire protocol `create` command: creates either a collection **or** a view from a full BSON spec. Accepts create options such as `viewOn`/`pipeline`, `validator`, and `validationLevel`, none of which the two-argument `create_collection` above can express. (`coll_mod` can change the same options on an existing collection.) `changeStreamPreAndPostImages` additionally requires the `documentdb.enablePreImages` GUC, which defaults to `off`. |
| `documentdb_api.drop_collection(p_database_name text, p_collection_name text, p_write_concern bson DEFAULT NULL, p_collection_uuid uuid DEFAULT NULL, p_track_changes bool DEFAULT true)` | Drops a collection. |
| `documentdb_api.rename_collection(p_database_name text, p_collection_name text, p_target_name text, p_drop_target bool DEFAULT false)` | Renames a collection using explicit arguments. |
| `documentdb_api.rename_collection(p_commandspec bson)` | Overload taking a wire-protocol `renameCollection` BSON spec. This is the form the gateway calls. |
| `documentdb_api.drop_database(p_database_name text, p_write_concern bson DEFAULT NULL)` | Drops a database and all of its collections. |
| `documentdb_api.list_databases(p_list_databases_spec bson)` | Returns information about all databases (added in v0.102-0). |
| `documentdb_api.shard_collection(p_database_name text, p_collection_name text, p_shard_key bson, p_is_reshard bool DEFAULT true)` | Shards or reshards a collection on the supplied key. This is the form the gateway calls. **Pass `p_is_reshard => false` when sharding for the first time** — see the notes below. |
| `documentdb_api.shard_collection(p_shard_key_spec bson)` | Single-argument overload taking the whole request as BSON. |
| `documentdb_api.reshard_collection(p_shard_key_spec bson)` | Re-shards an already-sharded collection. |
| `documentdb_api.unshard_collection(p_shard_key_spec bson)` | Removes the shard key from a sharded collection, returning it to a single unsharded table. |
| `documentdb_api.coll_mod(p_database_name text, p_collection_name text, p_spec bson)` | Executes a MongoDB `collMod` command. Changes collection options — `viewOn`, `pipeline`, `validator`, `expireAfterSeconds`, `changeStreamPreAndPostImages` — on an existing collection. |
| `documentdb_api.compact(p_spec bson)` | Compacts a collection's data and indexes (added in v0.104-0). Requires a GUC that is off by default — see the note below. |

> **Shard keys must be hashed.** Every value in the shard key document has to be the string `"hashed"`; anything else raises `only shard keys that use hashed are supported` (or `Shard key value provided is invalid` for a different string). A first-time shard therefore looks like `SELECT documentdb_api.shard_collection('mydb', 'users', '{ "value": "hashed" }'::documentdb_core.bson, false);`.

> **`shard_collection` defaults to resharding.** `p_is_reshard` defaults to `true`, so the natural three-argument call fails on a collection that is not sharded yet with `Collection <db>.<coll> is not sharded` (`ERRCODE_DOCUMENTDB_NAMESPACENOTSHARDED`). Pass `false` explicitly for an initial shard.

> **Sharding operations rewrite the whole collection.** `shard_collection`, `reshard_collection`, and `unshard_collection` each build a shadow table and re-insert every document, so plan for roughly double the collection's disk footprint for the duration and expect a runtime proportional to collection size. None of them warn about this.

> **`compact` requires `documentdb.enableCompactVacuumFull`, which defaults to `off`.** With the GUC off, `compact` returns `{ "ok": 1, "bytesFreed": 0 }` and reclaims nothing — no error, no warning. Turn the GUC on to actually run the (blocking) `VACUUM FULL`. `compact` also required a separate `documentdb.enableCompact` GUC when it was introduced in v0.104-0; that flag was removed in v0.109-0.

## Index Management

| Function | Description |
| --- | --- |
| `documentdb_api_internal.create_indexes_non_concurrently(p_database_name text, p_arg bson, p_skip_check_collection_create boolean DEFAULT false)` | Creates indexes in the foreground, blocking writes for the duration. With the default `p_skip_check_collection_create => false` this raises on any collection whose data table was not created in the current transaction, so a direct call on an existing collection needs `p_skip_check_collection_create => true`. The background path below calls it internally when it creates the collection itself. |
| `documentdb_api.create_indexes_background(p_database_name text, p_index_spec bson, OUT retval bson, OUT ok boolean, OUT requests bson)` | Validates an index spec and **queues** the build, then returns immediately — it does not wait for the index to be built. Present since the initial release; background builds became the working default in v0.104-0, when the drain job started being scheduled automatically. |
| `documentdb_api_internal.check_build_index_status(p_arg bson, OUT retval bson, OUT ok boolean, OUT complete boolean)` | Reports on builds queued by `create_indexes_background`. Takes the `requests` document returned by that call. |
| `CALL documentdb_api.drop_indexes(p_database_name text, p_arg bson, INOUT retval bson DEFAULT NULL)` | Drops indexes via the wire-protocol `dropIndexes` command. Implemented as a `PROCEDURE`. |

### Building an index from SQL

`createIndexes` is a two-call protocol. The gateway drives both halves for you; a direct SQL caller has to do it explicitly. Create the collection first — if `create_indexes_background` has to create it, the build runs inline instead and step 2 does not apply.

Step 1 — queue the build, then **commit**:

```sql
SELECT ok, requests FROM documentdb_api.create_indexes_background(
    'mydb',
    '{ "createIndexes": "users", "indexes": [ { "key": { "email": 1 }, "name": "email_1" } ] }'::documentdb_core.bson
);
COMMIT;
```

```text
 ok |                                     requests
----+----------------------------------------------------------------------------------
 t  | { "indexRequest" : { "cmdType" : "C", "ids" : [ { "$numberInt" : "42009" } ] } }
```

The commit is not optional. The request is queued by inserting into an ordinary table inside your transaction, and the worker that drains the queue runs in a different backend — until you commit, it cannot see the row, so the build never starts. Clients that do not autocommit (`psycopg`, JDBC) must commit explicitly here.

**The index is not usable yet.** Step 2 — pass that whole `requests` document to `check_build_index_status`, committing between attempts, and stop on `ok`:

```sql
SELECT ok, complete, retval FROM documentdb_api_internal.check_build_index_status(
    '{ "indexRequest" : { "cmdType" : "C", "ids" : [ { "$numberInt" : "42009" } ] } }'::documentdb_core.bson
);
```

**Check `ok` before `complete`.** A failed build also reports `complete = true` (with `ok = false` and the reason in `retval`), and so does a request that has simply left the queue — the implementation treats "not in the queue" as success, which is why dropping an index deliberately leaves its queue row behind. A loop that exits on `complete` alone will report success for a build that deadlocked. Poll until `ok` is false (fail) or `complete` is true with `ok` still true (success), and confirm with `listIndexes` if it matters.

Committing between attempts matters for the same reason it does in step 1, and additionally because holding a transaction open blocks the `CREATE INDEX CONCURRENTLY` the worker is trying to run. Upstream's test suite wraps all of this in a polling procedure, `documentdb_test_helpers.create_indexes_background`, which is worth reading as a reference implementation.

Two more behaviors are easy to miss:

- **A malformed index spec does not raise a SQL error.** `create_indexes_background` returns `ok = false` with the reason in `retval` and an empty `requests`, so a script that only watches for exceptions will report success on a rejected build. (A `NULL` spec or a read-only server does raise.)
- **Something has to drain the queue.** By default that is a pg_cron job running `documentdb_api_internal.build_index_concurrently`, scheduled by `documentdb_api_internal.schedule_background_index_build_jobs`; setting `documentdb.indexBuildsScheduledOnBgWorker` moves the work to a background worker instead. If neither is running — for example when `cron.use_background_workers` is `off` and pg_cron cannot authenticate against a hardened `pg_hba.conf` — queued builds never start, and anything polling for completion (including the gateway) waits indefinitely.

## Diagnostic and Administration Commands

Functions backing diagnostic and administrative wire protocol commands.

| Function | Description |
| --- | --- |
| `documentdb_api.coll_stats(p_database_name text, p_collection_name text, p_scale float8 DEFAULT 1)` | Returns storage and index statistics for a collection (`collStats`). |
| `documentdb_api.db_stats(p_database_name text, p_scale float8 DEFAULT 1, p_freestorage bool DEFAULT false)` | Returns storage statistics for a database (`dbStats`). |
| `documentdb_api.validate(database text, validateSpec bson, OUT document bson)` | Validates a collection's indexes (`validate`). |
| `documentdb_api.connection_status(p_spec bson)` | Returns authentication and role information for the current connection (`connectionStatus`, added in v0.105-0). |
| `documentdb_api.kill_op(p_command_spec bson)` | Cancels a running operation by op id (`killOp`, added in v0.109-0). |
| `documentdb_api.current_op_command(p_spec bson, OUT document bson)` | Backs the `currentOp` wire-protocol command (added in v0.102-0). The `$currentOp` aggregation stage uses the internal `documentdb_api_internal.current_op_aggregation(p_spec bson, OUT document bson)` set-returning helper, which has existed since v0.101-0. |

## User Management

Functions for creating, updating, and managing database users. Backed by the wire protocol `createUser`, `dropUser`, `updateUser`, and `usersInfo` commands.

| Function | Description |
| --- | --- |
| `documentdb_api.create_user(p_spec bson)` | Creates a new user. |
| `documentdb_api.drop_user(p_spec bson)` | Drops an existing user. |
| `documentdb_api.update_user(p_spec bson)` | Updates an existing user (password, roles, etc.). |
| `documentdb_api.users_info(p_spec bson)` | Returns information about one or more users. |

## Role Management

All four role functions were added in v0.106-0, together with wire-protocol support for `createRole`. Support for the `dropRole` and `rolesInfo` commands followed in v0.108-0. `updateRole` is routed by the gateway to `documentdb_api.update_role`.

All of them are gated behind `documentdb.enableRoleCrud`, added in v0.108-0 and off by default, so on a stock build `create_role`, `drop_role`, and `roles_info` raise "The CreateRole command is currently unsupported." and its equivalents before doing any work. The wire-protocol commands are additionally required to run against the `admin` database (`documentdb.enableRolesAdminDBCheck`, on by default since v0.109-0); the user management functions above are not.

| Function | Description |
| --- | --- |
| `documentdb_api.create_role(p_spec bson)` | Creates a new role. |
| `documentdb_api.drop_role(p_spec bson)` | Drops an existing role. |
| `documentdb_api.update_role(p_spec bson)` | Not implemented. The function body is a bare `ereport(ERROR)`, so every call raises "UpdateRole command is not supported in preview." regardless of the spec or of `enableRoleCrud`. |
| `documentdb_api.roles_info(p_spec bson)` | Returns information about one or more roles. |

## Utility Functions

| Function | Description |
| --- | --- |
| `documentdb_api.binary_version()` | Returns the running binary version of the `pg_documentdb` shared library. |
| `documentdb_api.binary_extended_version()` | Returns the extended binary version, including build metadata. |

## Example

To call any of these directly from `psql`:

```sql
SELECT * FROM documentdb_api.insert(
    'mydb',
    '{ "insert": "users", "documents": [ { "name": "alice" } ] }'::documentdb_core.bson
);
```

Because `insert` declares two `OUT` parameters, `SELECT *` returns them as the named columns `p_result` and `p_success`. Dropping the `*` would collapse them into a single composite value.

For more complete examples (cursors, aggregation, sharding) see the [API Reference](https://documentdb.io/docs/reference/).
