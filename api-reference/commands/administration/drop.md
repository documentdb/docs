---
title: drop and dropDatabase
description: Remove collections or logical databases using DocumentDB wire commands.
type: commands
category: administration
---

# drop and dropDatabase

`drop` removes a collection; `dropDatabase` removes the current logical database. Both invalidate associated gateway cursors.

```javascript
db.retired_timetables.drop()
db.getSiblingDB("bookings_scratch").dropDatabase()
```

| Parameter | Description |
| --- | --- |
| **`drop`** | (Required) A string specifying the collection name. |
| **`dropDatabase`** | (Required) A command marker, conventionally `1`; the target database comes from `$db`, supplied by the driver. |

The default wire adapters forward the database and collection names, not optional drop specifications. Do not rely on `encryptedFields`, `collectionUUID`, or `writeConcern` to change the drop operation. The response includes `ok` and `dropped`.

See [wire protocol and common fields](../diagnostic/wire-protocol.md). Source: gateway `processor/data_description.rs`, `process_drop_collection` and `process_drop_database`, and `postgres/documentdb_data_client.rs`.
