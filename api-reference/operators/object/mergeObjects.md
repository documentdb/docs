# $mergeObjects

Combines multiple documents into a single document.

## Syntax

```javascript
{ $mergeObjects: [ <document1>, <document2>, ... ] }
```

## Description

The `$mergeObjects` operator combines multiple documents into a single document. When there are duplicate field names, the operator takes the value from the last document that contains the field.

## Examples

### Basic Object Merge

```javascript
db.collection.aggregate([
  {
    $project: {
      combined: {
        $mergeObjects: [
          { status: "pending", type: "basic" },
          { status: "active", priority: "high" }
        ]
      }
    }
  }
])
```

Result:
```javascript
{
  combined: {
    status: "active",    // Takes the last value
    type: "basic",
    priority: "high"
  }
}
```

### Merging with Document Fields

```javascript
db.orders.aggregate([
  {
    $project: {
      result: {
        $mergeObjects: [
          "$defaults",
          "$overrides",
          {
            lastModified: new Date(),
            status: "processed"
          }
        ]
      }
    }
  }
])
```

### Merging Array of Objects

```javascript
db.config.aggregate([
  {
    $project: {
      settings: {
        $reduce: {
          input: "$configObjects",
          initialValue: {},
          in: {
            $mergeObjects: ["$$value", "$$this"]
          }
        }
      }
    }
  }
])
```

## Behavior

1. **Field Resolution**
   - Last value wins for duplicate fields
   - Preserves field order from input
   - Null values overwrite existing fields
   - Missing fields are ignored

2. **Type Handling**
   - Accepts any valid document
   - Preserves field types
   - Handles nested objects
   - Arrays are treated as atomic values

3. **Error Cases**
   - Non-document arguments
   - Invalid BSON types
   - Maximum document size exceeded

## Use Cases

1. **Configuration Management**
   - Merging default and custom settings
   - Building configuration objects
   - Applying overrides
   - Environment-specific settings

2. **Document Transformation**
   - Combining partial updates
   - Normalizing document structure
   - Applying templates
   - Data migration

3. **Dynamic Object Creation**
   - Building complex objects
   - Conditional field inclusion
   - Template instantiation
   - Document composition

## Best Practices

1. **Document Structure**
   - Plan field precedence
   - Document merge order
   - Handle conflicts
   - Validate results

2. **Performance**
   - Minimize merge depth
   - Control document size
   - Consider field count
   - Monitor memory usage

3. **Error Handling**
   - Validate input documents
   - Handle missing fields
   - Check size limits
   - Consider fallbacks

## Related Operators

- [$objectToArray](objectToArray.md) - Converts a document to an array of key-value pairs
- [$setField](setField.md) - Sets the value of a field in a document 