---
title: renameCollection
description: Rename a DocumentDB collection within the same logical database.
type: commands
category: administration
---

# renameCollection

Renames a collection within the same logical database.

```javascript
db.createCollection("flights_staging")
db.flights_staging.renameCollection("flights_active", false)
```

| Parameter | Description |
| --- | --- |
| **`renameCollection`** | (Required) A string specifying the source namespace, `database.collection`. |
| **`to`** | (Required) A string specifying the destination namespace in the same database. |
| **`dropTarget`** | (Optional) A boolean that drops an existing destination collection if `true`; defaults to `false`. |

Cross-database renaming and renaming a collection to itself are rejected. Shared `writeConcern` and `comment` do not alter renaming behavior.

See [wire protocol and common fields](../diagnostic/wire-protocol.md). Source: `pg_documentdb/src/commands/rename_collection.c`, `ParseRenameCollectionSpec`.
