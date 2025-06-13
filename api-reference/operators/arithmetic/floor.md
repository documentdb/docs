# $floor

Returns the largest integer less than or equal to the specified number.

## Syntax

```javascript
{ $floor: <number> }
```

## Description

The `$floor` operator rounds a number down to the nearest integer. It returns the largest integer less than or equal to the specified number.

## Examples

### Basic Usage

```javascript
db.samples.aggregate([
  {
    $project: {
      original: 1,
      rounded: { $floor: "$value" }
    }
  }
])
```

Input documents:
```javascript
{ "_id": 1, "value": 7.8 }
{ "_id": 2, "value": -3.4 }
{ "_id": 3, "value": 4.0 }
```

Output documents:
```javascript
{ "_id": 1, "original": 7.8, "rounded": 7 }
{ "_id": 2, "original": -3.4, "rounded": -4 }
{ "_id": 3, "original": 4.0, "rounded": 4 }
```

### Financial Calculations

```javascript
db.transactions.aggregate([
  {
    $project: {
      amount: 1,
      // Floor to nearest dollar
      wholeDollars: { $floor: "$amount" },
      // Calculate cents
      cents: {
        $multiply: [
          {
            $subtract: [
              "$amount",
              { $floor: "$amount" }
            ]
          },
          100
        ]
      }
    }
  }
])
```

### Time Calculations

```javascript
db.activities.aggregate([
  {
    $project: {
      activity: 1,
      durationMinutes: 1,
      // Calculate complete hours
      completeHours: {
        $floor: { $divide: ["$durationMinutes", 60] }
      },
      // Calculate remaining minutes
      remainingMinutes: {
        $subtract: [
          "$durationMinutes",
          { $multiply: [
              { $floor: { $divide: ["$durationMinutes", 60] } },
              60
            ]
          }
        ]
      }
    }
  }
])
```

### Resource Allocation

```javascript
db.containers.aggregate([
  {
    $project: {
      items: 1,
      containerCapacity: 1,
      // Calculate full containers needed
      fullContainers: {
        $floor: { $divide: ["$items", "$containerCapacity"] }
      },
      // Calculate remaining items
      remainingItems: {
        $mod: ["$items", "$containerCapacity"]
      },
      // Calculate utilization percentage
      utilization: {
        $multiply: [
          {
            $divide: [
              "$items",
              { $multiply: [
                  { $floor: { $divide: ["$items", "$containerCapacity"] } },
                  "$containerCapacity"
                ]
              }
            ]
          },
          100
        ]
      }
    }
  }
])
```

## Behavior

1. **Input Types**
   - Numbers: Rounds down to nearest integer
   - Integers: Returns same number
   - Null: Returns null
   - Missing: Returns null
   - Non-numeric: Throws an error

2. **Precision**
   - Always returns integer
   - Maintains numeric type (integer)
   - Handles positive and negative numbers

3. **Special Cases**
   - Infinity: Returns Infinity
   - -Infinity: Returns -Infinity
   - NaN: Returns NaN

## Use Cases

1. **Financial Calculations**
   - Currency rounding
   - Price calculations
   - Budget allocations
   - Tax computations

2. **Resource Management**
   - Container packing
   - Storage allocation
   - Batch processing
   - Inventory management

3. **Time Calculations**
   - Duration rounding
   - Schedule planning
   - Time slot allocation
   - Period calculations

4. **Data Processing**
   - Range binning
   - Histogram creation
   - Data grouping
   - Boundary calculations

## Examples in Aggregation Pipeline

### Histogram Buckets

```javascript
db.measurements.aggregate([
  {
    $group: {
      _id: {
        bucket: {
          $floor: { $divide: ["$value", 10] }  // 10-unit buckets
        }
      },
      count: { $sum: 1 },
      min: { $min: "$value" },
      max: { $max: "$value" }
    }
  },
  {
    $project: {
      bucketRange: {
        start: { $multiply: ["$_id.bucket", 10] },
        end: { $multiply: [{ $add: ["$_id.bucket", 1] }, 10] }
      },
      count: 1,
      min: 1,
      max: 1
    }
  }
])
```

### Time Slot Allocation

```javascript
db.events.aggregate([
  {
    $project: {
      event: 1,
      startTime: 1,
      // Floor to nearest hour
      timeSlot: {
        $floor: {
          $divide: [
            { $subtract: ["$startTime", new Date("2024-01-01")] },
            3600000  // milliseconds in an hour
          ]
        }
      }
    }
  },
  {
    $group: {
      _id: "$timeSlot",
      events: { $push: "$event" },
      count: { $sum: 1 }
    }
  }
])
```

## Performance Considerations

1. **Optimization**
   - Simple operation with minimal overhead
   - Can be combined with other arithmetic operations
   - Consider indexing original fields for filtering

2. **Memory Usage**
   - Constant memory usage
   - No additional storage requirements
   - Efficient for large datasets

## Best Practices

1. **Type Handling**
   - Validate input types
   - Handle null values explicitly
   - Consider default values when appropriate

2. **Rounding Strategy**
   - Document rounding behavior
   - Consider business rules
   - Be consistent across application

3. **Error Handling**
   - Handle non-numeric inputs
   - Validate range constraints
   - Consider overflow conditions

## Related Operators

- [$ceil](ceil.md) - Rounds up to nearest integer
- [$round](round.md) - Rounds to specified decimal place
- [$trunc](trunc.md) - Truncates to specified decimal place
- [$mod](mod.md) - Returns remainder of division 