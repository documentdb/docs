---
title: $changeStream
description: The $changeStream stage is registered in DocumentDB but is not available in the open-source v0.110-0 runtime unless a downstream extension enables the change-stream hook.
type: operators
category: aggregation
---

# $changeStream

The `$changeStream` aggregation stage is registered in the `v0.110-0` source, but the open-source runtime does not enable change streams by default. In the open-source path, `IsChangeStreamFeatureAvailableAndCompatible()` returns `false` unless a downstream extension installs the change-stream compatibility hook, so attempts to run `$changeStream` return a command-not-supported error.

## Availability

| Runtime | Status |
| --- | --- |
| Open-source DocumentDB `v0.110-0` | Not available by default. |
| Downstream builds that provide a change-stream hook | Availability and behavior depend on that downstream implementation. |

## Source-backed validation rules

When a runtime enables the change-stream hook, the core pipeline parser applies these rules before dispatching to the change-stream aggregation function:

| Rule | Behavior |
| --- | --- |
| Stage position | `$changeStream` must be the first stage in the aggregation pipeline. |
| Views | `$changeStream` cannot run on views. |
| Nested pipelines | `$changeStream` is prohibited inside a `$unionWith` subpipeline. |
| Following stages | Only `$match`, `$project`, `$addFields`, `$replaceRoot`, `$replaceWith`, `$set`, `$unset`, and `$redact` are permitted after `$changeStream`. |

## Syntax

```javascript
db.collection.aggregate([
  { $changeStream: { } }
])
```

Do not rely on this stage in open-source DocumentDB `v0.110-0` unless your runtime explicitly documents change-stream support.
