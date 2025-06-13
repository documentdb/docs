# Arithmetic Expression Operators

Arithmetic expression operators perform mathematical operations on values and return the result. These operators can be used in aggregation pipeline stages like `$project` and `$match` to perform calculations.

## Available Operators

| Operator | Description |
|----------|-------------|
| [$abs](abs.md) | Returns the absolute value of a number. |
| [$add](add.md) | Adds numbers together or adds numbers and dates. |
| [$ceil](ceil.md) | Returns the smallest integer greater than or equal to the specified number. |
| [$divide](divide.md) | Returns the result of dividing the first number by the second. |
| [$exp](exp.md) | Raises e to the specified exponent. |
| [$floor](floor.md) | Returns the largest integer less than or equal to the specified number. |
| [$ln](ln.md) | Calculates the natural logarithm ln (e-based) of a number. |
| [$log](log.md) | Calculates the log of a number in the specified base. |
| [$log10](log10.md) | Calculates the log base 10 of a number. |

## Common Use Cases

1. **Financial Calculations**
   - Computing discounts and markups
   - Calculating interest rates
   - Currency conversions
   - Tax computations

2. **Statistical Analysis**
   - Computing averages and deviations
   - Normalizing data
   - Scaling values
   - Logarithmic transformations

3. **Scientific Computations**
   - Unit conversions
   - Physical calculations
   - Mathematical modeling
   - Data normalization

## Example Pipeline

```javascript
db.sales.aggregate([
  {
    $project: {
      item: 1,
      originalPrice: 1,
      discount: 1,
      // Calculate final price with discount
      finalPrice: {
        $subtract: [
          "$originalPrice",
          { $multiply: ["$originalPrice", { $divide: ["$discount", 100] }] }
        ]
      },
      // Round up to nearest dollar
      roundedPrice: {
        $ceil: {
          $subtract: [
            "$originalPrice",
            { $multiply: ["$originalPrice", { $divide: ["$discount", 100] }] }
          ]
        }
      },
      // Calculate percentage of original price
      percentageOfOriginal: {
        $multiply: [
          { $divide: [
              { $subtract: [
                "$originalPrice",
                { $multiply: ["$originalPrice", { $divide: ["$discount", 100] }] }
              ]},
              "$originalPrice"
            ]
          },
          100
        ]
      }
    }
  }
])
```

## Best Practices

1. **Type Handling**
   - Ensure operands are of the correct type
   - Handle null values appropriately
   - Consider type conversion implications

2. **Precision**
   - Be aware of floating-point arithmetic limitations
   - Use appropriate rounding functions
   - Consider decimal precision requirements

3. **Performance**
   - Combine operations when possible
   - Use efficient calculation methods
   - Consider index usage for filtered calculations

4. **Error Handling**
   - Handle division by zero
   - Check for overflow conditions
   - Validate input ranges

## Operator Combinations

Arithmetic operators can be combined to create complex calculations:

```javascript
db.measurements.aggregate([
  {
    $project: {
      // Convert to logarithmic scale and round
      logValue: { $ceil: { $log10: "$value" } },
      
      // Calculate percentage difference
      percentDiff: {
        $multiply: [
          { $divide: [
              { $subtract: ["$current", "$previous"] },
              "$previous"
            ]
          },
          100
        ]
      },
      
      // Normalize values to 0-1 range
      normalized: {
        $divide: [
          { $subtract: ["$value", "$min"] },
          { $subtract: ["$max", "$min"] }
        ]
      }
    }
  }
])
```

## Type Conversion

When using arithmetic operators, be aware of automatic type conversion:

- Numbers are automatically converted to decimals for calculations
- Dates can be used in addition and subtraction
- Strings that represent numbers are not automatically converted
- Null values in calculations typically result in null

## Error Conditions

Common error conditions to handle:

- Division by zero
- Logarithm of zero or negative numbers
- Overflow in exponential operations
- Invalid numeric strings
- Missing or null values

## Related Operators

- [Comparison Operators](../comparison/index.md)
- [Type Conversion Operators](../type/index.md)
- [Date Operators](../date/index.md)
- [Conditional Operators](../conditional/index.md) 