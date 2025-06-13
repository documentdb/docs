# $switch

Evaluates a series of case expressions and returns a matching result.

## Syntax

```javascript
{
  $switch: {
    branches: [
      { case: <expression>, then: <value> },
      { case: <expression>, then: <value> },
      ...
    ],
    default: <value>  // Optional
  }
}
```

## Description

The `$switch` operator evaluates a series of case expressions in order until it finds an expression that evaluates to true, then returns the corresponding `then` value. If no case expression evaluates to true and a default value is specified, it returns the default value. If no case matches and there is no default, it returns an error.

## Examples

### Basic Usage

```javascript
// Classify temperature ranges
db.weather.aggregate([
  {
    $project: {
      temperature: 1,
      description: {
        $switch: {
          branches: [
            { case: { $gt: ["$temperature", 30] }, then: "hot" },
            { case: { $gt: ["$temperature", 20] }, then: "warm" },
            { case: { $gt: ["$temperature", 10] }, then: "mild" }
          ],
          default: "cold"
        }
      }
    }
  }
])
```

### Multiple Conditions

```javascript
// Determine shipping cost based on weight and distance
db.shipments.aggregate([
  {
    $project: {
      cost: {
        $switch: {
          branches: [
            {
              case: {
                $and: [
                  { $lte: ["$weight", 5] },
                  { $lte: ["$distance", 100] }
                ]
              },
              then: 10
            },
            {
              case: {
                $and: [
                  { $lte: ["$weight", 5] },
                  { $gt: ["$distance", 100] }
                ]
              },
              then: 20
            },
            {
              case: { $gt: ["$weight", 5] },
              then: 50
            }
          ],
          default: 30
        }
      }
    }
  }
])
```

### Complex Expressions

```javascript
// Calculate discounts based on multiple factors
db.orders.aggregate([
  {
    $project: {
      discount: {
        $switch: {
          branches: [
            {
              case: { $gt: ["$totalAmount", 1000] },
              then: { $multiply: ["$totalAmount", 0.2] }
            },
            {
              case: { $gt: ["$itemCount", 10] },
              then: { $multiply: ["$totalAmount", 0.15] }
            },
            {
              case: { $eq: ["$customerType", "premium"] },
              then: { $multiply: ["$totalAmount", 0.1] }
            }
          ],
          default: 0
        }
      }
    }
  }
])
```

## Behavior

1. **Case Evaluation**
   - Sequential evaluation
   - Short-circuit behavior
   - Boolean expressions

2. **Branch Selection**
   - First match wins
   - Default handling
   - Error conditions

3. **Expression Types**
   - Complex conditions
   - Value computation
   - Type consistency

## Use Cases

1. **Status Classification**
   ```javascript
   // Classify order status
   db.orders.aggregate([
     {
       $project: {
         status: {
           $switch: {
             branches: [
               { case: { $eq: ["$paid", false] }, then: "pending" },
               { case: { $eq: ["$shipped", false] }, then: "processing" },
               { case: { $eq: ["$delivered", false] }, then: "shipped" }
             ],
             default: "completed"
           }
         }
       }
     }
   ])
   ```

2. **Price Calculation**
   ```javascript
   // Calculate tiered pricing
   db.products.aggregate([
     {
       $project: {
         price: {
           $switch: {
             branches: [
               { case: { $gte: ["$quantity", 100] }, then: { $multiply: ["$basePrice", 0.7] } },
               { case: { $gte: ["$quantity", 50] }, then: { $multiply: ["$basePrice", 0.8] } },
               { case: { $gte: ["$quantity", 20] }, then: { $multiply: ["$basePrice", 0.9] } }
             ],
             default: "$basePrice"
           }
         }
       }
     }
   ])
   ```

3. **Risk Assessment**
   ```javascript
   // Evaluate risk levels
   db.transactions.aggregate([
     {
       $project: {
         riskLevel: {
           $switch: {
             branches: [
               { case: { $gt: ["$amount", 10000] }, then: "high" },
               { case: { $gt: ["$frequency", 10] }, then: "medium" },
               { case: { $eq: ["$verified", false] }, then: "elevated" }
             ],
             default: "low"
           }
         }
       }
     }
   ])
   ```

## Performance Considerations

1. **Branch Order**
   - Most common cases first
   - Evaluation complexity
   - Short-circuit optimization

2. **Expression Complexity**
   - Condition evaluation
   - Value computation
   - Memory usage

3. **Default Values**
   - Error handling
   - Type consistency
   - Fallback strategy

## Best Practices

1. **Code Organization**
   - Order cases logically
   - Use meaningful names
   - Document conditions

2. **Error Handling**
   - Provide default values
   - Validate inputs
   - Handle edge cases

3. **Performance**
   - Optimize case order
   - Minimize complexity
   - Consider alternatives

## Common Patterns

### Status Mapping

```javascript
// Map status codes to descriptions
{
  $project: {
    status: {
      $switch: {
        branches: [
          { case: { $eq: ["$statusCode", 1] }, then: "pending" },
          { case: { $eq: ["$statusCode", 2] }, then: "processing" },
          { case: { $eq: ["$statusCode", 3] }, then: "completed" }
        ],
        default: "unknown"
      }
    }
  }
}
```

### Grade Assignment

```javascript
// Assign letter grades
{
  $project: {
    grade: {
      $switch: {
        branches: [
          { case: { $gte: ["$score", 90] }, then: "A" },
          { case: { $gte: ["$score", 80] }, then: "B" },
          { case: { $gte: ["$score", 70] }, then: "C" },
          { case: { $gte: ["$score", 60] }, then: "D" }
        ],
        default: "F"
      }
    }
  }
}
```

### Service Level

```javascript
// Determine service level
{
  $project: {
    serviceLevel: {
      $switch: {
        branches: [
          { case: { $gte: ["$usage", 1000] }, then: "enterprise" },
          { case: { $gte: ["$usage", 500] }, then: "business" },
          { case: { $gte: ["$usage", 100] }, then: "professional" }
        ],
        default: "basic"
      }
    }
  }
}
```

## Related Operators

- [$cond](cond.md) - A simpler ternary operator for conditional logic
- [$ifNull](ifnull.md) - Returns a value if the expression is null
- [$expr](../aggregation/expr.md) - Allows use of aggregation expressions in query language 