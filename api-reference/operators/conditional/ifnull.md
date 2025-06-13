# $ifNull

Returns either the non-null result of the first expression or the result of the second expression.

## Syntax

```javascript
{ $ifNull: [ <expression1>, <expression2> ] }
```

## Description

The `$ifNull` operator evaluates the first expression. If the result is not null, it returns that result; if the result is null, it returns the result of the second expression. This operator is particularly useful for handling missing fields or null values in documents.

## Examples

### Basic Usage

```javascript
// Use default value for missing field
db.users.aggregate([
  {
    $project: {
      name: 1,
      status: { $ifNull: ["$status", "active"] }
    }
  }
])
```

### Multiple Expressions

```javascript
// Check multiple fields for non-null value
db.contacts.aggregate([
  {
    $project: {
      preferredContact: {
        $ifNull: [
          "$email",
          "$phone",
          "$address",
          "no contact info"
        ]
      }
    }
  }
])
```

### Nested Fields

```javascript
// Handle nested document fields
db.profiles.aggregate([
  {
    $project: {
      settings: {
        theme: {
          $ifNull: ["$settings.theme", "default"]
        }
      }
    }
  }
])
```

## Behavior

1. **Null Handling**
   - Evaluates null values
   - Handles missing fields
   - Array of expressions

2. **Expression Evaluation**
   - Sequential evaluation
   - Type preservation
   - Error handling

3. **Return Values**
   - First non-null value
   - Default value handling
   - Type consistency

## Use Cases

1. **Default Values**
   ```javascript
   // Set default configuration
   db.settings.aggregate([
     {
       $project: {
         timeout: {
           $ifNull: ["$timeout", 30]
         }
       }
     }
   ])
   ```

2. **Data Cleaning**
   ```javascript
   // Clean missing data
   db.records.aggregate([
     {
       $project: {
         value: {
           $ifNull: ["$value", 0]
         }
       }
     }
   ])
   ```

3. **Fallback Values**
   ```javascript
   // Handle multiple possible fields
   db.users.aggregate([
     {
       $project: {
         displayName: {
           $ifNull: ["$nickname", "$firstName", "$username"]
         }
       }
     }
   ])
   ```

## Performance Considerations

1. **Expression Evaluation**
   - Sequential processing
   - Short-circuit evaluation
   - Expression complexity

2. **Memory Usage**
   - Value storage
   - Expression caching
   - Pipeline optimization

3. **Query Optimization**
   - Index usage
   - Field presence
   - Expression order

## Best Practices

1. **Default Values**
   - Choose meaningful defaults
   - Consider type consistency
   - Document defaults

2. **Error Handling**
   - Validate input types
   - Handle edge cases
   - Provide fallbacks

3. **Code Organization**
   - Keep expressions simple
   - Use meaningful names
   - Document assumptions

## Common Patterns

### Field Coalescing

```javascript
// Choose first non-null value
{
  $project: {
    contact: {
      $ifNull: ["$email", "$phone", "$address", "unknown"]
    }
  }
}
```

### Default Configuration

```javascript
// Set default settings
{
  $project: {
    settings: {
      timeout: { $ifNull: ["$settings.timeout", 60] },
      retries: { $ifNull: ["$settings.retries", 3] }
    }
  }
}
```

### Data Normalization

```javascript
// Normalize missing values
{
  $project: {
    metrics: {
      count: { $ifNull: ["$metrics.count", 0] },
      average: { $ifNull: ["$metrics.average", 0] }
    }
  }
}
```

## Related Operators

- [$cond](cond.md) - A ternary operator for conditional logic
- [$switch](switch.md) - Evaluates multiple conditions
- [$coalesce](../aggregation/coalesce.md) - Returns first non-null value from multiple expressions 