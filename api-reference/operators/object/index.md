# Object Expression Operators

Object expression operators provide functionality for manipulating and transforming document fields and structures.

## Available Operators

| Operator | Description |
|----------|-------------|
| [$mergeObjects](mergeObjects.md) | Combines multiple documents into a single document |
| [$objectToArray](objectToArray.md) | Converts a document to an array of key-value pairs |
| [$setField](setField.md) | Sets the value of a field in a document |

## Common Use Cases

- Combining multiple documents
- Transforming document structures
- Dynamic field manipulation
- Document normalization
- Schema transformation

## Example Usage

```javascript
// Merging multiple documents
db.collection.aggregate([
  {
    $project: {
      combined: {
        $mergeObjects: [
          "$defaults",
          "$overrides",
          { status: "active" }
        ]
      }
    }
  }
])

// Converting object to array
db.collection.aggregate([
  {
    $project: {
      fields: {
        $objectToArray: "$document"
      }
    }
  }
])

// Setting field value
db.collection.aggregate([
  {
    $project: {
      modified: {
        $setField: {
          field: "status",
          input: "$document",
          value: "updated"
        }
      }
    }
  }
])
```

## Best Practices

1. **Document Structure**
   - Maintain consistent schemas
   - Handle missing fields
   - Consider field order

2. **Performance**
   - Minimize document size
   - Optimize field access
   - Use appropriate indexes

3. **Data Integrity**
   - Validate field values
   - Handle type conversion
   - Preserve required fields 