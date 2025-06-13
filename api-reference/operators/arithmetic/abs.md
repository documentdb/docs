# $abs

Returns the absolute value of a number.

## Syntax

```javascript
{ $abs: <number> }
```

## Description

The `$abs` operator returns the absolute (positive) value of a number. It accepts any valid expression that resolves to a number.

## Examples

### Basic Usage

```javascript
db.numbers.aggregate([
  {
    $project: {
      number: 1,
      absoluteValue: { $abs: "$number" }
    }
  }
])
```

Input documents:
```javascript
{ "_id": 1, "number": -5 }
{ "_id": 2, "number": 7 }
{ "_id": 3, "number": -2.5 }
```

Output documents:
```javascript
{ "_id": 1, "number": -5, "absoluteValue": 5 }
{ "_id": 2, "number": 7, "absoluteValue": 7 }
{ "_id": 3, "number": -2.5, "absoluteValue": 2.5 }
```

### Using with Expressions

```javascript
db.sales.aggregate([
  {
    $project: {
      item: 1,
      difference: { $subtract: ["$target", "$actual"] },
      absoluteDifference: { $abs: { $subtract: ["$target", "$actual"] } },
      percentageDeviation: {
        $multiply: [
          {
            $divide: [
              { $abs: { $subtract: ["$target", "$actual"] } },
              "$target"
            ]
          },
          100
        ]
      }
    }
  }
])
```

### Filtering with Absolute Values

```javascript
db.measurements.aggregate([
  {
    $match: {
      $expr: {
        $lt: [
          { $abs: { $subtract: ["$reading", "$baseline"] } },
          10
        ]
      }
    }
  },
  {
    $project: {
      reading: 1,
      baseline: 1,
      deviation: { $abs: { $subtract: ["$reading", "$baseline"] } }
    }
  }
])
```

### Statistical Calculations

```javascript
db.samples.aggregate([
  {
    $group: {
      _id: "$experimentId",
      avgDeviation: {
        $avg: {
          $abs: { $subtract: ["$value", "$expectedValue"] }
        }
      },
      maxDeviation: {
        $max: {
          $abs: { $subtract: ["$value", "$expectedValue"] }
        }
      }
    }
  }
])
```

## Behavior

1. **Input Types**
   - Numbers: Returns absolute value
   - Null: Returns null
   - Missing: Returns null
   - Non-numeric: Throws an error

2. **Precision**
   - Maintains decimal precision
   - Preserves numeric type (integer/decimal)

3. **Error Cases**
   - Non-numeric values cause errors
   - NaN input results in NaN output

## Use Cases

1. **Financial Analysis**
   - Calculating price variations
   - Analyzing profit/loss margins
   - Computing absolute differences

2. **Scientific Measurements**
   - Error margin calculations
   - Deviation analysis
   - Tolerance checking

3. **Data Processing**
   - Signal processing
   - Noise reduction
   - Outlier detection

4. **Statistical Analysis**
   - Mean absolute deviation
   - Absolute error calculations
   - Range calculations

## Examples in Aggregation Pipeline

### Filtering by Absolute Difference

```javascript
db.stocks.aggregate([
  {
    $match: {
      $expr: {
        $lt: [
          { $abs: { $subtract: ["$currentPrice", "$movingAverage"] } },
          { $multiply: ["$movingAverage", 0.05] }  // 5% threshold
        ]
      }
    }
  }
])
```

### Computing Average Absolute Error

```javascript
db.predictions.aggregate([
  {
    $group: {
      _id: "$modelId",
      meanAbsoluteError: {
        $avg: {
          $abs: { $subtract: ["$predicted", "$actual"] }
        }
      }
    }
  },
  {
    $sort: { meanAbsoluteError: 1 }
  }
])
```

## Performance Considerations

1. **Optimization**
   - Use indexes on original fields when filtering
   - Consider pre-computing absolute values for frequent queries
   - Combine with other arithmetic operations efficiently

2. **Memory Usage**
   - Minimal memory overhead
   - Efficient for large datasets
   - No additional storage requirements

## Related Operators

- [$subtract](subtract.md) - Subtracts two numbers
- [$multiply](multiply.md) - Multiplies numbers
- [$cmp](../comparison/cmp.md) - Compares two values
- [$avg](../accumulator/avg.md) - Calculates average value 