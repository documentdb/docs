---
title: validate
description: Validate DocumentDB collection metadata and indexes with supported validation modes.
type: commands
category: diagnostic
---

# validate

Validates a collection and returns validation information. Views are not supported.

```javascript
db.flights.validate()
```

| Parameter | Description |
| --- | --- |
| **`validate`** | (Required) A string specifying the collection name. |
| **`repair`** | (Optional) A boolean-like value that repairs baseline collection access privileges; not a general-purpose BSON or index repair facility. |

`repair` defaults to `false`. Collection-owner authorization is required.

See [wire protocol and common fields](wire-protocol.md). Source: `pg_documentdb/src/commands/validate.c`, `command_validate`.
