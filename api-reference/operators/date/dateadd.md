# $dateAdd

Adds a specified amount of time to a date.

## Syntax

```javascript
{
  $dateAdd: {
    startDate: <expression>,
    unit: <unit>,
    amount: <number>,
    timezone: <timezone>  // Optional
  }
}
```

## Description

The `$dateAdd` operator adds a specified number of time units to a date. The operator takes a start date, a unit of time, and the number of units to add. Optionally, you can specify a timezone for the calculation.

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

## Examples

### Basic Usage

```javascript
// Add 3 days to a date
db.events.aggregate([
  {
    $project: {
      futureDate: {
        $dateAdd: {
          startDate: "$date",
          unit: "day",
          amount: 3
        }
      }
    }
  }
])
```

### With Timezone

```javascript
// Add 2 hours in specific timezone
db.appointments.aggregate([
  {
    $project: {
      adjustedTime: {
        $dateAdd: {
          startDate: "$scheduledTime",
          unit: "hour",
          amount: 2,
          timezone: "America/New_York"
        }
      }
    }
  }
])
```

### Multiple Units

```javascript
// Add multiple time units sequentially
db.tasks.aggregate([
  {
    $project: {
      dueDate: {
        $dateAdd: {
          startDate: {
            $dateAdd: {
              startDate: "$createdAt",
              unit: "day",
              amount: 30
            }
          },
          unit: "hour",
          amount: 12
        }
      }
    }
  }
])
```

## Behavior

1. **Date Handling**
   - Timezone awareness
   - DST handling
   - Date normalization

2. **Unit Processing**
   - Unit conversion
   - Boundary handling
   - Overflow behavior

3. **Error Handling**
   - Invalid dates
   - Invalid units
   - Range validation

## Use Cases

1. **Due Date Calculation**
   ```javascript
   // Calculate due date for tasks
   db.tasks.aggregate([
     {
       $project: {
         dueDate: {
           $dateAdd: {
             startDate: "$assignedDate",
             unit: "day",
             amount: "$durationDays"
           }
         }
       }
     }
   ])
   ```

2. **Scheduling**
   ```javascript
   // Schedule future appointments
   db.appointments.aggregate([
     {
       $project: {
         nextAppointment: {
           $dateAdd: {
             startDate: "$lastVisit",
             unit: "month",
             amount: 6
           }
         }
       }
     }
   ])
   ```

3. **Expiration Calculation**
   ```javascript
   // Calculate expiration dates
   db.subscriptions.aggregate([
     {
       $project: {
         expiryDate: {
           $dateAdd: {
             startDate: "$startDate",
             unit: "year",
             amount: 1
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

2. **Date Arithmetic**
   - Unit conversion efficiency
   - Date normalization
   - Boundary checks

3. **Expression Optimization**
   - Compound operations
   - Index usage
   - Memory usage

## Best Practices

1. **Timezone Handling**
   - Specify timezone when needed
   - Consider DST impact
   - Document timezone assumptions

2. **Unit Selection**
   - Choose appropriate units
   - Consider boundary cases
   - Handle overflows

3. **Error Handling**
   - Validate input dates
   - Handle invalid results
   - Provide defaults

## Common Patterns

### Subscription Period

```javascript
// Calculate subscription end date
{
  $project: {
    endDate: {
      $dateAdd: {
        startDate: "$subscriptionStart",
        unit: "month",
        amount: "$subscriptionLength"
      }
    }
  }
}
```

### Task Scheduling

```javascript
// Schedule task deadline
{
  $project: {
    deadline: {
      $dateAdd: {
        startDate: "$createdAt",
        unit: "day",
        amount: "$estimatedDays"
      }
    }
  }
}
```

### Session Expiry

```javascript
// Calculate session expiration
{
  $project: {
    expiresAt: {
      $dateAdd: {
        startDate: "$loginTime",
        unit: "minute",
        amount: 30
      }
    }
  }
}
```

## Related Operators

- [$dateDiff](datediff.md) - Calculates the difference between two dates
- [$dateFromParts](datefromparts.md) - Constructs a date from parts
- [$dateToString](../string/datetostring.md) - Converts a date to a formatted string 