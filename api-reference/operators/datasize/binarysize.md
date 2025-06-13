# $binarySize

Returns the size of a binary string in bytes.

## Syntax

```javascript
{ $binarySize: <expression> }
```

## Description

The `$binarySize` operator calculates the size in bytes of a binary string. The operator accepts any expression that evaluates to a binary data type and returns the number of bytes in that binary string.

## Examples

### Basic Usage

```javascript
// Calculate size of binary field
db.files.aggregate([
  {
    $project: {
      filename: 1,
      fileSize: {
        $binarySize: "$content"
      }
    }
  }
])
```

### With Conditional Logic

```javascript
// Check file size threshold
db.files.aggregate([
  {
    $project: {
      filename: 1,
      isLarge: {
        $gt: [
          { $binarySize: "$content" },
          1024 * 1024  // 1MB
        ]
      }
    }
  }
])
```

### Size-based Filtering

```javascript
// Filter documents by binary size
db.files.aggregate([
  {
    $match: {
      $expr: {
        $lt: [
          { $binarySize: "$data" },
          1024  // 1KB
        ]
      }
    }
  }
])
```

## Behavior

1. **Input Handling**
   - Binary data types
   - Null values
   - Invalid types

2. **Size Calculation**
   - Byte counting
   - Zero handling
   - Maximum size

3. **Error Handling**
   - Type validation
   - Null processing
   - Invalid input

## Use Cases

1. **File Size Analysis**
   ```javascript
   // Analyze file sizes
   db.files.aggregate([
     {
       $project: {
         filename: 1,
         sizeInKB: {
           $divide: [
             { $binarySize: "$content" },
             1024
           ]
         }
       }
     }
   ])
   ```

2. **Storage Management**
   ```javascript
   // Group files by size range
   db.files.aggregate([
     {
       $project: {
         sizeRange: {
           $switch: {
             branches: [
               {
                 case: { $lt: [{ $binarySize: "$data" }, 1024] },
                 then: "small"
               },
               {
                 case: { $lt: [{ $binarySize: "$data" }, 1024 * 1024] },
                 then: "medium"
               }
             ],
             default: "large"
           }
         }
       }
     }
   ])
   ```

3. **Quota Management**
   ```javascript
   // Check quota usage
   db.storage.aggregate([
     {
       $project: {
         isOverQuota: {
           $gt: [
             { $sum: { $binarySize: "$files" } },
             "$quotaLimit"
           ]
         }
       }
     }
   ])
   ```

## Performance Considerations

1. **Memory Usage**
   - Size calculation
   - Buffer handling
   - Memory limits

2. **Computation Cost**
   - Direct size access
   - No decompression
   - No validation

3. **Query Optimization**
   - Index usage
   - Projection optimization
   - Aggregation pipeline

## Best Practices

1. **Type Handling**
   - Validate input types
   - Handle null values
   - Check constraints

2. **Size Management**
   - Use appropriate units
   - Consider thresholds
   - Handle limits

3. **Error Handling**
   - Provide defaults
   - Log errors
   - Handle edge cases

## Common Patterns

### Size Classification

```javascript
// Classify by size
{
  $project: {
    category: {
      $switch: {
        branches: [
          {
            case: { $lt: [{ $binarySize: "$data" }, 100 * 1024] },
            then: "small"
          },
          {
            case: { $lt: [{ $binarySize: "$data" }, 1024 * 1024] },
            then: "medium"
          }
        ],
        default: "large"
      }
    }
  }
}
```

### Storage Analysis

```javascript
// Analyze storage usage
{
  $group: {
    _id: "$type",
    totalSize: {
      $sum: { $binarySize: "$content" }
    },
    avgSize: {
      $avg: { $binarySize: "$content" }
    }
  }
}
```

### Quota Enforcement

```javascript
// Check quota limits
{
  $match: {
    $expr: {
      $lt: [
        { $binarySize: "$data" },
        "$userQuota"
      ]
    }
  }
}
```

## Related Operators

- [$bsonSize](bsonsize.md) - Returns the size of a document in BSON format
- [$strLenBytes](../string/strlenbytes.md) - Returns the length of a string in bytes
- [$size](../array/size.md) - Returns the number of elements in an array 