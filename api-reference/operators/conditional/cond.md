# $cond

A ternary operator that evaluates a boolean expression and returns one of two specified values.

## Syntax

### Statement Format

```javascript
{
  $cond: {
    if: <boolean-expression>,
    then: <true-case>,
    else: <false-case>
  }
}
```

### Array Format

```javascript
{ $cond: [ <boolean-expression>, <true-case>, <false-case> ] }
```

## Description

The `$cond` operator evaluates a boolean expression and returns one value if the expression evaluates to true, and a different value if the expression evaluates to false. It functions similarly to an if-then-else statement in programming languages.

## Examples

### Basic Usage

```javascript
// Calculate discount based on quantity
db.orders.aggregate([
  {
    $project: {
      item: 1,
      discount: {
        $cond: {
          if: { $gt: ["$quantity", 100] },
          then: 0.20,
          else: 0.10
        }
      }
    }
  }
])
```

### Array Format

```javascript
// Classify temperature readings
db.weather.aggregate([
  {
    $project: {
      temperature: 1,
      classification: {
        $cond: [
          { $gt: ["$temperature", 25] },
          "hot",
          "cold"
        ]
      }
    }
  }
])
```

### Nested Conditions

```javascript
// Determine shipping method based on multiple conditions
db.orders.aggregate([
  {
    $project: {
      shippingMethod: {
        $cond: {
          if: { $gt: ["$weight", 100] },
          then: "freight",
          else: {
            $cond: {
              if: { $gt: ["$priority", 5] },
              then: "express",
              else: "standard"
            }
          }
        }
      }
    }
  }
])
```

## Behavior

1. **Expression Evaluation**
   - Boolean expression evaluation
   - Type conversion rules
   - Null handling

2. **Return Values**
   - Type consistency
   - Expression evaluation
   - Error handling

3. **Nesting Support**
   - Multiple conditions
   - Complex expressions
   - Operator combinations

## Use Cases

1. **Conditional Classification**
   ```javascript
   // Classify products by price range
   db.products.aggregate([
     {
       $project: {
         category: {
           $cond: {
             if: { $gte: ["$price", 1000] },
             then: "premium",
             else: "standard"
           }
         }
       }
     }
   ])
   ```

2. **Status Determination**
   ```javascript
   // Determine order status
   db.orders.aggregate([
     {
       $project: {
         status: {
           $cond: {
             if: { $eq: ["$paid", true] },
             then: "processing",
             else: "pending"
           }
         }
       }
     }
   ])
   ```

3. **Dynamic Calculations**
   ```javascript
   // Calculate dynamic fees
   db.transactions.aggregate([
     {
       $project: {
         fee: {
           $cond: {
             if: { $gt: ["$amount", 1000] },
             then: { $multiply: ["$amount", 0.02] },
             else: { $multiply: ["$amount", 0.03] }
           }
         }
       }
     }
   ])
   ```

## Performance Considerations

1. **Expression Complexity**
   - Evaluation overhead
   - Nested conditions
   - Expression optimization

2. **Data Types**
   - Type conversion costs
   - Consistent types
   - Null handling

3. **Aggregation Pipeline**
   - Stage position
   - Memory usage
   - Index utilization

## Best Practices

1. **Code Organization**
   - Use statement format for readability
   - Break down complex conditions
   - Document conditions

2. **Error Handling**
   - Handle null values
   - Validate input types
   - Provide default values

3. **Performance**
   - Minimize nesting
   - Use appropriate indexes
   - Consider alternatives

## Common Patterns

### Status Mapping

```javascript
// Map numeric status to text
{
  $project: {
    status: {
      $cond: {
        if: { $eq: ["$status", 1] },
        then: "active",
        else: "inactive"
      }
    }
  }
}
```

### Threshold-based Classification

```javascript
// Classify based on threshold
{
  $project: {
    risk: {
      $cond: {
        if: { $gt: ["$score", 75] },
        then: "low",
        else: "high"
      }
    }
  }
}
```

### Dynamic Field Selection

```javascript
// Select field based on condition
{
  $project: {
    value: {
      $cond: {
        if: { $eq: ["$useAlternate", true] },
        then: "$alternateField",
        else: "$defaultField"
      }
    }
  }
}
```

## Related Operators

- [$ifNull](ifnull.md) - Returns a specified value if the expression evaluates to null
- [$switch](switch.md) - Evaluates multiple conditions and returns a matching result
- [$expr](../aggregation/expr.md) - Allows use of aggregation expressions in query language 