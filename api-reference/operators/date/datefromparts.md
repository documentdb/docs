# $dateFromParts

Constructs a date object from the specified parts.

## Syntax

```javascript
{
  $dateFromParts: {
    year: <year>,
    month: <month>,          // Optional, defaults to 1
    day: <day>,             // Optional, defaults to 1
    hour: <hour>,           // Optional, defaults to 0
    minute: <minute>,       // Optional, defaults to 0
    second: <second>,       // Optional, defaults to 0
    millisecond: <ms>,      // Optional, defaults to 0
    timezone: <timezone>    // Optional
  }
}
```

## Description

The `$dateFromParts` operator constructs a date object from individual components such as year, month, day, etc. All parts except year are optional and will default to their minimum values if not specified.

### Component Ranges

- `year`: Any integer
- `month`: 1-12
- `day`: 1-31 (depending on month)
- `hour`: 0-23
- `minute`: 0-59
- `second`: 0-59
- `millisecond`: 0-999

## Examples

### Basic Usage

```javascript
// Create a date from year, month, and day
db.events.aggregate([
  {
    $project: {
      eventDate: {
        $dateFromParts: {
          year: 2024,
          month: 3,
          day: 15
        }
      }
    }
  }
])
```

### With Time Components

```javascript
// Create a precise timestamp
db.logs.aggregate([
  {
    $project: {
      timestamp: {
        $dateFromParts: {
          year: 2024,
          month: 3,
          day: 15,
          hour: 14,
          minute: 30,
          second: 45,
          millisecond: 500
        }
      }
    }
  }
])
```

### With Timezone

```javascript
// Create a date in specific timezone
db.appointments.aggregate([
  {
    $project: {
      appointmentTime: {
        $dateFromParts: {
          year: 2024,
          month: 3,
          day: 15,
          hour: 9,
          minute: 0,
          timezone: "America/New_York"
        }
      }
    }
  }
])
```

## Behavior

1. **Component Handling**
   - Default values
   - Range validation
   - Overflow handling

2. **Timezone Processing**
   - Timezone conversion
   - DST handling
   - UTC conversion

3. **Error Handling**
   - Invalid components
   - Out of range values
   - Invalid timezone

## Use Cases

1. **Date Construction**
   ```javascript
   // Create event dates
   db.events.aggregate([
     {
       $project: {
         startDate: {
           $dateFromParts: {
             year: "$eventYear",
             month: "$eventMonth",
             day: "$eventDay"
           }
         }
       }
     }
   ])
   ```

2. **Schedule Generation**
   ```javascript
   // Generate appointment slots
   db.schedule.aggregate([
     {
       $project: {
         slot: {
           $dateFromParts: {
             year: "$year",
             month: "$month",
             day: "$day",
             hour: "$hour",
             timezone: "$timezone"
           }
         }
       }
     }
   ])
   ```

3. **Data Normalization**
   ```javascript
   // Normalize date components
   db.records.aggregate([
     {
       $project: {
         normalizedDate: {
           $dateFromParts: {
             year: "$year",
             month: { $ifNull: ["$month", 1] },
             day: { $ifNull: ["$day", 1] }
           }
         }
       }
     }
   ])
   ```

## Performance Considerations

1. **Component Processing**
   - Validation overhead
   - Default handling
   - Type conversion

2. **Timezone Operations**
   - Timezone lookup
   - DST calculation
   - UTC conversion

3. **Error Handling**
   - Range checking
   - Validation cost
   - Error propagation

## Best Practices

1. **Input Validation**
   - Validate component ranges
   - Handle missing values
   - Check timezone validity

2. **Default Values**
   - Use meaningful defaults
   - Document assumptions
   - Handle edge cases

3. **Error Handling**
   - Provide fallbacks
   - Log invalid inputs
   - Handle exceptions

## Common Patterns

### Date Normalization

```javascript
// Normalize partial dates
{
  $project: {
    normalizedDate: {
      $dateFromParts: {
        year: "$year",
        month: { $ifNull: ["$month", 1] },
        day: { $ifNull: ["$day", 1] }
      }
    }
  }
}
```

### Schedule Generation

```javascript
// Generate time slots
{
  $project: {
    timeSlot: {
      $dateFromParts: {
        year: "$year",
        month: "$month",
        day: "$day",
        hour: "$slotHour",
        minute: 0,
        timezone: "UTC"
      }
    }
  }
}
```

### Date Composition

```javascript
// Compose date from separate fields
{
  $project: {
    fullDate: {
      $dateFromParts: {
        year: "$metadata.year",
        month: "$metadata.month",
        day: "$metadata.day",
        hour: "$time.hour",
        minute: "$time.minute"
      }
    }
  }
}
```

## Related Operators

- [$dateFromString](datefromstring.md) - Converts a date string to a date object
- [$dateToParts](../aggregation/datetoparts.md) - Breaks down a date into its component parts
- [$dateToString](../string/datetostring.md) - Converts a date to a formatted string 