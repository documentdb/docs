# $bsonSize

Returns the size of a document or value in bytes when converted to BSON.

## Syntax

```javascript
{ $bsonSize: <expression> }
```

## Description

The `$bsonSize` operator calculates the size in bytes of a document or value when converted to BSON format. This includes the overhead of BSON encoding, making it useful for estimating storage requirements and monitoring document sizes.

## Examples

### Basic Usage

```javascript
// Calculate document size
db.collection.aggregate([
  {
    $project: {
      documentSize: {
        $bsonSize: "$$ROOT"
      }
    }
  }
])
```

### Field Size Analysis

```javascript
// Analyze size of specific fields
db.collection.aggregate([
  {
    $project: {
      field: 1,
      fieldSize: {
        $bsonSize: "$field"
      }
    }
  }
])
```

### Size-based Filtering

```javascript
// Find large documents
db.collection.aggregate([
  {
    $match: {
      $expr: {
        $gt: [
          { $bsonSize: "$$ROOT" },
          16 * 1024  // 16KB
        ]
      }
    }
  }
])
```

## Behavior

1. **BSON Encoding**
   - Type overhead
   - Field names
   - Nested documents

2. **Size Calculation**
   - Complete document
   - Individual fields
   - Array elements

3. **Special Cases**
   - Null handling
   - Empty documents
   - Maximum size

## Use Cases

1. **Storage Analysis**
   ```javascript
   // Analyze collection storage
   db.collection.aggregate([
     {
       $group: {
         _id: "$type",
         avgSize: {
           $avg: { $bsonSize: "$$ROOT" }
         },
         maxSize: {
           $max: { $bsonSize: "$$ROOT" }
         }
       }
     }
   ])
   ```

2. **Document Size Monitoring**
   ```javascript
   // Monitor document growth
   db.collection.aggregate([
     {
       $project: {
         _id: 1,
         currentSize: { $bsonSize: "$$ROOT" },
         exceedsLimit: {
           $gt: [
             { $bsonSize: "$$ROOT" },
             "$sizeLimit"
           ]
         }
       }
     }
   ])
   ```

3. **Optimization Analysis**
   ```javascript
   // Analyze field size impact
   db.collection.aggregate([
     {
       $project: {
         totalSize: { $bsonSize: "$$ROOT" },
         dataSize: { $bsonSize: "$data" },
         metadataSize: {
           $subtract: [
             { $bsonSize: "$$ROOT" },
             { $bsonSize: "$data" }
           ]
         }
       }
     }
   ])
   ```

## Performance Considerations

1. **Computation Cost**
   - BSON encoding
   - Size calculation
   - Document traversal

2. **Memory Usage**
   - Document loading
   - Buffer allocation
   - Temporary storage

3. **Query Optimization**
   - Index usage
   - Projection impact
   - Pipeline position

## Best Practices

1. **Size Monitoring**
   - Regular checks
   - Threshold alerts
   - Trend analysis

2. **Optimization**
   - Field selection
   - Document structure
   - Storage efficiency

3. **Error Prevention**
   - Size limits
   - Growth monitoring
   - Validation checks

## Common Patterns

### Size Distribution

```javascript
// Analyze size distribution
{
  $bucket: {
    groupBy: { $bsonSize: "$$ROOT" },
    boundaries: [0, 1024, 16384, 32768],
    default: "large",
    output: {
      count: { $sum: 1 },
      docs: { $push: "$_id" }
    }
  }
}
```

### Storage Optimization

```javascript
// Identify optimization candidates
{
  $project: {
    _id: 1,
    totalSize: { $bsonSize: "$$ROOT" },
    dataRatio: {
      $divide: [
        { $bsonSize: "$data" },
        { $bsonSize: "$$ROOT" }
      ]
    }
  }
}
```

### Capacity Planning

```javascript
// Estimate storage requirements
{
  $group: {
    _id: "$category",
    totalSize: {
      $sum: { $bsonSize: "$$ROOT" }
    },
    avgSize: {
      $avg: { $bsonSize: "$$ROOT" }
    },
    count: { $sum: 1 }
  }
}
```

## Related Operators

- [$binarySize](binarysize.md) - Returns the size of a binary string in bytes
- [$strLenBytes](../string/strlenbytes.md) - Returns the length of a string in bytes
- [$size](../array/size.md) - Returns the number of elements in an array 