---
title: hello
description: The hello command returns information about the DocumentDB server, including topology role, wire protocol limits, and the DocumentDB engine version.
type: commands
category: diagnostic
---

# hello

The `hello` command returns a document describing the role of the DocumentDB server, along with connection limits, supported authentication mechanisms, and the version of the DocumentDB engine you are connected to. It's the recommended way to discover the DocumentDB version from a MongoDB client, since the `internal.documentdb_versions` field reports the underlying engine version rather than only the emulated MongoDB wire-protocol version.

The versions in `internal.documentdb_versions` come from the backend DocumentDB PostgreSQL extension, not from the gateway. The array reports two values: the installed SQL extension version (from the `pg_extension` catalog, in `major-minor` form such as `0.113-0`) followed by the loaded binary version (from `documentdb_api.binary_version()`, in dotted form such as `0.113.0`). In PostgreSQL, the installed extension version *is* the schema version — it tracks which SQL upgrade scripts have been applied — so the first value reflects the schema currently installed in the database. These two values normally describe the same release and differ only in formatting, but they can diverge during an upgrade — for example, when the shared library has been updated but `ALTER EXTENSION documentdb UPDATE` hasn't yet been run. A mismatch such as `["0.112-0", "0.113.0"]` indicates the SQL schema is behind the loaded binary.

## Syntax

The syntax for the `hello` command is as follows:

```javascript
db.runCommand({ hello: 1 })
```

The `mongosh` helper `db.hello()` runs the same command:

```javascript
db.hello()
```

## Return fields

The response includes the following fields. Fields marked as DocumentDB-specific are extensions to the standard MongoDB response.

| Field | Description |
| --- | --- |
| **`isWritablePrimary`** | `true` when the connected node accepts writes. |
| **`msg`** | Set to `isdbgrid`, indicating requests are served through the DocumentDB gateway. |
| **`maxBsonObjectSize`** | Maximum permitted size, in bytes, of a BSON document. |
| **`maxMessageSizeBytes`** | Maximum permitted size, in bytes, of a single message. |
| **`maxWriteBatchSize`** | Maximum number of write operations permitted in a single batch. |
| **`localTime`** | The server's current time, in UTC. |
| **`logicalSessionTimeoutMinutes`** | Minutes an idle logical session is retained before timing out. |
| **`minWireVersion`** / **`maxWireVersion`** | Range of the MongoDB wire protocol versions the server supports. |
| **`readOnly`** | `true` when the connection is read-only. |
| **`connectionId`** | Identifier of the current connection. |
| **`saslSupportedMechs`** | Authentication mechanisms the server supports, for example `SCRAM-SHA-256`. |
| **`internal.documentdb_versions`** | (DocumentDB-specific) Versions reported by the backend DocumentDB PostgreSQL extension: the installed SQL extension version — which in PostgreSQL is the schema version (`major-minor`, for example `0.113-0`) — followed by the loaded binary version (dotted, for example `0.113.0`). These normally match and diverge only during an upgrade. |
| **`ok`** | `1` when the command succeeds. |

## Examples

### Example 1: Inspect the server role and version

Run the `hello` command to view the full response:

```javascript
db.runCommand({ hello: 1 })
```

The server returns a document similar to the following:

```json
{
  "isWritablePrimary": true,
  "msg": "isdbgrid",
  "maxBsonObjectSize": 16777216,
  "maxMessageSizeBytes": 48000000,
  "maxWriteBatchSize": 25000,
  "localTime": 1783006410548,
  "logicalSessionTimeoutMinutes": 30,
  "minWireVersion": 0,
  "maxWireVersion": 21,
  "readOnly": false,
  "connectionId": 382703778,
  "saslSupportedMechs": [
    "SCRAM-SHA-256"
  ],
  "internal": {
    "documentdb_versions": [
      "0.113-0",
      "0.113.0"
    ],
    "kind": ""
  },
  "ok": 1
}
```

### Example 2: Determine the DocumentDB engine version

To read only the DocumentDB engine version, project the `internal.documentdb_versions` field:

```javascript
db.runCommand({ hello: 1 }).internal.documentdb_versions
```

The command returns an array with the installed SQL extension version followed by the loaded binary version:

```json
[ "0.113-0", "0.113.0" ]
```

The first value is the installed SQL extension version — the schema version, from the `pg_extension` catalog — and the second is the compiled binary version. They normally describe the same release and differ only in formatting.

> [!NOTE]
> The `buildInfo` command reports the emulated MongoDB wire-protocol version (for example, `7.0.0`), not the DocumentDB engine version. Use `hello` and read `internal.documentdb_versions` to determine the DocumentDB version.

## Related content

- [DocumentDB Local](https://documentdb.io/docs/documentdb-local/)
