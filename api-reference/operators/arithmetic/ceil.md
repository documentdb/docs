# $ceil

Returns the smallest integer greater than or equal to the specified number.

## Syntax

```javascript
{ $ceil: <number> }
```

## Description

The `$ceil` operator rounds a number up to the nearest integer. It returns the smallest integer greater than or equal to the specified number.

## Examples

### Basic Usage

```javascript
db.samples.aggregate([
  {
    $project: {
      original: 1,
      rounded: { $ceil: "$value" }
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
{ "_id": 1, "original": 7.8, "rounded": 8 }
{ "_id": 2, "original": -3.4, "rounded": -3 }
{ "_id": 3, "original": 4.0, "rounded": 4 }
```

### Rounding in Calculations

```javascript
db.orders.aggregate([
  {
    $project: {
      item: 1,
      quantity: 1,
      unitPrice: 1,
      // Round up total to nearest dollar
      totalCost: {
        $ceil: {
          $multiply: ["$quantity", "$unitPrice"]
        }
      }
    }
  }
])
```

### Inventory Planning

```javascript
db.inventory.aggregate([
  {
    $project: {
      product: 1,
      demand: 1,
      // Calculate boxes needed (each box holds 12 units)
      boxesNeeded: {
        $ceil: {
          $divide: ["$demand", 12]
        }
      },
      // Calculate storage space needed (each box takes 2.3 cubic feet)
      storageNeeded: {
        $ceil: {
          $multiply: [
            { $divide: ["$demand", 12] },
            2.3
          ]
        }
      }
    }
  }
])
```

### Time Calculations

```javascript
db.tasks.aggregate([
  {
    $project: {
      task: 1,
      duration: 1,
      // Calculate full hours needed
      billableHours: {
        $ceil: {
          $divide: ["$duration", 60]  // duration in minutes
        }
      },
      // Calculate full days needed
      workDays: {
        $ceil: {
          $divide: ["$duration", 480]  // duration in minutes, 8-hour workday
        }
      }
    }
  }
])
```

## Behavior

1. **Input Types**
   - Numbers: Rounds up to nearest integer
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
   - Rounding up prices
   - Currency conversions
   - Tax calculations
   - Budget planning

2. **Inventory Management**
   - Container calculations
   - Storage space planning
   - Order quantity rounding
   - Packaging requirements

3. **Time Management**
   - Billable hours calculation
   - Work period planning
   - Schedule blocking
   - Duration rounding

4. **Resource Allocation**
   - Capacity planning
   - Resource distribution
   - Quota calculations
   - Batch processing

## Examples in Aggregation Pipeline

### Resource Planning

```javascript
db.projects.aggregate([
  {
    $project: {
      project: 1,
      resources: {
        workers: {
          $ceil: {
            $multiply: [
              { $divide: ["$totalHours", 40] },  // 40-hour work week
              1.2  // 20% buffer
            ]
          }
        },
        equipment: {
          $ceil: {
            $multiply: ["$baseEquipment", 1.5]  // 50% extra for redundancy
          }
        }
      }
    }
  }
])
```

### Pricing Tiers

```javascript
db.products.aggregate([
  {
    $project: {
      product: 1,
      basePrice: 1,
      // Round up to nearest price point
      retailPrice: {
        $multiply: [
          {
            $ceil: {
              $divide: ["$basePrice", 4.99]
            }
          },
          4.99
        ]
      }
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

- [$floor](floor.md) - Rounds down to nearest integer
- [$round](round.md) - Rounds to specified decimal place
- [$trunc](trunc.md) - Truncates to specified decimal place
- [$multiply](multiply.md) - Multiplies numbers 