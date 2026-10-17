---
title: $changeStream
description: The $changeStream stage opens a change stream cursor in MongoDB. DocumentDB does not support it.
type: operators
category: aggregation
---

# $changeStream

> [!IMPORTANT]
> DocumentDB does not support `$changeStream`. Any pipeline that uses it, including `db.collection.watch()`, fails with `Stage $changeStream is not supported yet in native pipeline` (`CommandNotSupported`, code 115).

In MongoDB, the `$changeStream` aggregation stage opens a cursor that reports inserts, updates, deletes, and other changes as they happen. The syntax below is listed for reference only.

## Syntax

```javascript
{
  $changeStream: {
    allChangesForCluster: <boolean>,
    fullDocument: <string>,
    fullDocumentBeforeChange: <string>,
    resumeAfter: <ResumeToken>,
    startAfter: <ResumeToken>,
    startAtOperationTime: <Timestamp>,
    showExpandedEvents: <boolean>
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`allChangesForCluster`** | Optional. Boolean. If true, returns changes for the entire cluster. Default is false. |
| **`fullDocument`** | Optional. String. Determines what to return for update operations. Options: 'default', 'updateLookup', 'whenAvailable', 'required'. |
| **`fullDocumentBeforeChange`** | Optional. String. Returns the preimage of the document. Options: "off", "whenAvailable", "required". |
| **`resumeAfter`** | Optional. Resume token to resume change stream after a specific event. |
| **`startAfter`** | Optional. Resume token to start change stream after a specific event. |
| **`startAtOperationTime`** | Optional. timestamp for starting change stream from a specific time. |
| **`showExpandedEvents`** | Optional. Boolean. Include another change stream events. Default is false. |

## Example

This pipeline is rejected:

```javascript
db.stores.aggregate([
  {
    $changeStream: {
      fullDocument: "updateLookup"
    }
  }
])
```

```
Stage $changeStream is not supported yet in native pipeline
```
