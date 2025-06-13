# $dateDiff

Calculates the difference between two dates.

## Syntax

```javascript
{
  $dateDiff: {
    startDate: <expression>,
    endDate: <expression>,
    unit: <unit>,
    timezone: <timezone>,  // Optional
    startOfWeek: <day>     // Optional
  }
}
```

## Description

The `$dateDiff` operator calculates the difference between two dates in the specified unit of time. The operator returns the difference as a number, representing complete units between the dates.

### Available Units

- `year`
- `quarter`
- `month`
- `week`
- `day`
- `hour`
- `minute`
- `second`
- `millisecond`

### Start of Week Options

- `sunday`
- `monday`
- `tuesday`
- `wednesday`
- `thursday`
- `friday`
- `saturday`

## Examples

### Basic Usage

```javascript
// Calculate days between dates
db.events.aggregate([
  {
    $project: {
      daysBetween: {
        $dateDiff: {
          startDate: "$startDate",
          endDate: "$endDate",
          unit: "day"
        }
      }
    }
  }
])
```

### With Timezone

```javascript
// Calculate hours with timezone consideration
db.shifts.aggregate([
  {
    $project: {
      hoursWorked: {
        $dateDiff: {
          startDate: "$clockIn",
          endDate: "$clockOut",
          unit: "hour",
          timezone: "America/New_York"
        }
      }
    }
  }
])
```

### Custom Week Start

```javascript
// Calculate weeks with Monday as start
db.projects.aggregate([
  {
    $project: {
      weeksDuration: {
        $dateDiff: {
          startDate: "$projectStart",
          endDate: "$projectEnd",
          unit: "week",
          startOfWeek: "monday"
        }
      }
    }
  }
])
```

## Behavior

1. **Date Handling**
   - Complete unit counting
   - Timezone awareness
   - DST handling

2. **Unit Processing**
   - Boundary alignment
   - Partial unit handling
   - Week start handling

3. **Error Handling**
   - Invalid dates
   - Invalid units
   - Range validation

## Use Cases

1. **Age Calculation**
   ```javascript
   // Calculate age in years
   db.users.aggregate([
     {
       $project: {
         age: {
           $dateDiff: {
             startDate: "$birthDate",
             endDate: "$$NOW",
             unit: "year"
           }
         }
       }
     }
   ])
   ```

2. **Duration Tracking**
   ```javascript
   // Calculate task duration
   db.tasks.aggregate([
     {
       $project: {
         duration: {
           $dateDiff: {
             startDate: "$startTime",
             endDate: "$completionTime",
             unit: "hour"
           }
         }
       }
     }
   ])
   ```

3. **Billing Periods**
   ```javascript
   // Calculate billing cycles
   db.subscriptions.aggregate([
     {
       $project: {
         billingCycles: {
           $dateDiff: {
             startDate: "$subscriptionStart",
             endDate: "$currentDate",
             unit: "month"
           }
         }
       }
     }
   ])
   ```

## Performance Considerations

1. **Timezone Operations**
   - Timezone conversion cost
   - DST calculation overhead
   - Cache utilization

2. **Unit Calculations**
   - Boundary alignment cost
   - Unit conversion efficiency
   - Precision handling

3. **Expression Optimization**
   - Index usage
   - Memory usage
   - Computation cost

## Best Practices

1. **Unit Selection**
   - Choose appropriate units
   - Consider precision needs
   - Handle partial units

2. **Timezone Handling**
   - Specify timezone when needed
   - Consider DST impact
   - Document timezone assumptions

3. **Error Handling**
   - Validate input dates
   - Handle invalid results
   - Provide defaults

## Common Patterns

### Time Elapsed

```javascript
// Calculate time since creation
{
  $project: {
    ageInDays: {
      $dateDiff: {
        startDate: "$createdAt",
        endDate: "$$NOW",
        unit: "day"
      }
    }
  }
}
```

### Service Duration

```javascript
// Calculate service duration
{
  $project: {
    monthsOfService: {
      $dateDiff: {
        startDate: "$joinDate",
        endDate: "$currentDate",
        unit: "month"
      }
    }
  }
}
```

### Payment Intervals

```javascript
// Calculate payment periods
{
  $project: {
    paymentCycles: {
      $dateDiff: {
        startDate: "$lastPayment",
        endDate: "$nextPayment",
        unit: "month"
      }
    }
  }
}
```

## Related Operators

- [$dateAdd](dateadd.md) - Adds a specified amount of time to a date
- [$dateFromParts](datefromparts.md) - Constructs a date from parts
- [$dateToString](../string/datetostring.md) - Converts a date to a formatted string 