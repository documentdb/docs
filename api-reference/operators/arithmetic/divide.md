# $divide

Returns the result of dividing the first number by the second.

## Syntax

```javascript
{ $divide: [ <dividend>, <divisor> ] }
```

## Description

The `$divide` operator divides one number by another and returns the result. The operator accepts any valid expression that resolves to numbers.

## Examples

### Basic Division

```javascript
db.sales.aggregate([
  {
    $project: {
      item: 1,
      unitPrice: { $divide: ["$total", "$quantity"] }
    }
  }
])
```

Input documents:
```javascript
{ "_id": 1, "item": "widget", "total": 100, "quantity": 4 }
```

Output documents:
```javascript
{ "_id": 1, "item": "widget", "unitPrice": 25 }
```

### Percentage Calculations

```javascript
db.performance.aggregate([
  {
    $project: {
      metric: 1,
      actual: 1,
      target: 1,
      percentageAchieved: {
        $multiply: [
          { $divide: ["$actual", "$target"] },
          100
        ]
      }
    }
  }
])
```

### Rate Calculations

```javascript
db.transactions.aggregate([
  {
    $project: {
      period: 1,
      transactions: 1,
      hours: 1,
      // Calculate transactions per hour
      transactionRate: {
        $round: [
          { $divide: ["$transactions", "$hours"] },
          2
        ]
      },
      // Calculate average time per transaction in minutes
      avgTransactionTime: {
        $round: [
          {
            $multiply: [
              { $divide: ["$hours", "$transactions"] },
              60
            ]
          },
          1
        ]
      }
    }
  }
])
```

### Complex Financial Calculations

```javascript
db.investments.aggregate([
  {
    $project: {
      investment: 1,
      // Calculate compound interest rate
      effectiveRate: {
        $subtract: [
          {
            $pow: [
              { $add: [1, { $divide: ["$nominalRate", "$compoundingsPerYear"] }] },
              "$compoundingsPerYear"
            ]
          },
          1
        ]
      },
      // Calculate present value
      presentValue: {
        $divide: [
          "$futureValue",
          {
            $pow: [
              { $add: [1, "$discountRate"] },
              "$years"
            ]
          }
        ]
      }
    }
  }
])
```

## Behavior

1. **Input Types**
   - Numbers: Standard division
   - Integers: May result in decimal
   - Decimals: Maintains precision
   - Null: Returns null
   - Zero divisor: Returns error

2. **Precision**
   - Maintains decimal precision
   - Follows IEEE 754 standard
   - Handles large and small numbers

3. **Special Cases**
   - Division by zero: Error
   - Infinity: Returns Infinity
   - -Infinity: Returns -Infinity
   - NaN: Returns NaN

## Use Cases

1. **Financial Analysis**
   - Unit price calculations
   - Rate computations
   - Percentage changes
   - Ratio analysis

2. **Performance Metrics**
   - Rates and velocities
   - Efficiency calculations
   - Resource utilization
   - Productivity measures

3. **Statistical Analysis**
   - Average calculations
   - Normalization
   - Ratio computations
   - Distribution analysis

4. **Resource Planning**
   - Capacity utilization
   - Load balancing
   - Resource allocation
   - Efficiency metrics

## Examples in Aggregation Pipeline

### Weighted Averages

```javascript
db.scores.aggregate([
  {
    $project: {
      student: 1,
      weightedScore: {
        $divide: [
          {
            $add: [
              { $multiply: ["$exam", 0.6] },
              { $multiply: ["$assignments", 0.3] },
              { $multiply: ["$participation", 0.1] }
            ]
          },
          { $add: [0.6, 0.3, 0.1] }
        ]
      }
    }
  }
])
```

### Moving Averages

```javascript
db.stocks.aggregate([
  {
    $group: {
      _id: "$symbol",
      movingAverage: {
        $divide: [
          { $sum: "$price" },
          { $sum: 1 }
        ]
      },
      volatility: {
        $divide: [
          { $stdDevPop: "$price" },
          { $avg: "$price" }
        ]
      }
    }
  }
])
```

## Performance Considerations

1. **Optimization**
   - Use appropriate numeric types
   - Consider index usage for filtered results
   - Handle division by zero cases

2. **Memory Usage**
   - Minimal memory overhead
   - Efficient for large datasets
   - Consider precision requirements

3. **Error Prevention**
   - Validate divisor values
   - Handle null cases
   - Consider default values

## Best Practices

1. **Error Handling**
   - Check for zero divisors
   - Handle null values explicitly
   - Validate input ranges

2. **Precision Management**
   - Define precision requirements
   - Use appropriate rounding
   - Document precision expectations

3. **Type Safety**
   - Validate numeric types
   - Handle type conversion
   - Consider range limitations

## Related Operators

- [$multiply](multiply.md) - Multiplies numbers
- [$mod](mod.md) - Returns remainder of division
- [$round](round.md) - Rounds to specified decimal place
- [$avg](../accumulator/avg.md) - Calculates average value 