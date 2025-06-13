# $objectToArray

Converts a document to an array of key-value pairs.

## Syntax

```javascript
{ $objectToArray: <object> }
```

## Description

The `$objectToArray` operator converts a document object into an array where each element is a document containing two fields:
- `k`: The field name from the original document
- `v`: The field value from the original document

## Examples

### Basic Conversion

```javascript
db.collection.aggregate([
  {
    $project: {
      asArray: {
        $objectToArray: {
          item: "apple",
          price: 5.99,
          quantity: 10
        }
      }
    }
  }
])
```

Result:
```javascript
{
  asArray: [
    { k: "item", v: "apple" },
    { k: "price", v: 5.99 },
    { k: "quantity", v: 10 }
  ]
}
```

### Converting Document Fields

```javascript
db.inventory.aggregate([
  {
    $project: {
      item: 1,
      specs: { $objectToArray: "$specifications" }
    }
  }
])
```

### Processing Dynamic Fields

```javascript
db.config.aggregate([
  {
    $project: {
      settings: { $objectToArray: "$options" }
    }
  },
  {
    $unwind: "$settings"
  },
  {
    $match: {
      "settings.v": { $type: "string" }
    }
  }
])
```

## Behavior

1. **Output Format**
   - Array of k-v documents
   - Preserves field order
   - Maintains value types
   - Handles nested objects

2. **Type Handling**
   - Accepts any valid document
   - Preserves BSON types
   - Handles arrays as values
   - Processes nested documents

3. **Special Cases**
   - Empty objects
   - Null values
   - Missing fields
   - Invalid inputs

## Use Cases

1. **Dynamic Field Processing**
   - Handling variable schemas
   - Field name analysis
   - Value type validation
   - Schema discovery

2. **Data Transformation**
   - Document normalization
   - Field extraction
   - Format conversion
   - Data migration

3. **Analysis**
   - Field usage statistics
   - Value distribution
   - Schema validation
   - Data profiling

## Best Practices

1. **Performance**
   - Consider result size
   - Handle large documents
   - Plan memory usage
   - Use appropriate indexes

2. **Data Handling**
   - Validate input objects
   - Handle missing data
   - Consider field order
   - Process nested structures

3. **Error Management**
   - Check input types
   - Handle edge cases
   - Validate output
   - Plan error recovery

## Related Operators

- [$mergeObjects](mergeObjects.md) - Combines multiple documents into a single document
- [$setField](setField.md) - Sets the value of a field in a document 