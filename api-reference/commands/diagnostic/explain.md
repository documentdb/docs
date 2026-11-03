---
title: explain
description: Explain supported DocumentDB read commands and select diagnostic verbosity.
type: commands
category: diagnostic
---

# explain

Returns planning information for `find`, `aggregate`, `count`, or `distinct`. The default gateway does not explain write commands.

```javascript
db.flights.find({ status: "Delayed" }).explain("executionStats")
```

| Parameter | Description |
| --- | --- |
| **`explain`** | (Required) A document specifying the nested read command, with the same options as its command page. |
| **`verbosity`** | (Optional) A string selecting `queryPlanner` (default), `executionStats`, or `allPlansExecution`. DocumentDB also recognizes `allShardsQueryPlan` and `allShardsExecution` for distributed diagnostics. |

`allPlansExecution` provides detailed execution plans, including cursor-based planning details, alongside execution statistics. Use it to investigate how a cursor query is planned.

Execution-statistics modes execute the underlying read to collect statistics. An unrecognized verbosity string follows the implementation's default mode rather than selecting a new diagnostic mode. `aggregate` also accepts `explain: true` directly.

See [wire protocol and common fields](wire-protocol.md). Source: gateway `explain/mod.rs`, `process_explain` and `Verbosity`.
