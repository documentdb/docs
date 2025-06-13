# $dateFromString

Converts a date string to a date object.

## Syntax

```javascript
{
  $dateFromString: {
    dateString: <dateString>,
    format: <formatString>,     // Optional
    timezone: <timezone>,       // Optional
    onError: <expression>,      // Optional
    onNull: <expression>        // Optional
  }
}
```

## Description

The `$dateFromString` operator converts a date string to a date object according to the specified format. If no format is specified, it expects the date string to be in ISO-8601 format.

### Format Specifiers

- `%Y` - Year (4 digits)
- `%m` - Month (01-12)
- `%d` - Day of month (01-31)
- `%H` - Hour in 24-hour format (00-23)
- `%M` - Minute (00-59)
- `%S` - Second (00-59)
- `%L` - Millisecond (000-999)
- `%z` - UTC offset
- `%Z` - Timezone name

## Examples

### Basic Usage

```javascript
// Parse ISO date string
db.events.aggregate([
  {
    $project: {
      date: {
        $dateFromString: {
          dateString: "2024-03-15T14:30:00Z"
        }
      }
    }
  }
])
```

### Custom Format

```javascript
// Parse custom formatted date
db.logs.aggregate([
  {
    $project: {
      timestamp: {
        $dateFromString: {
          dateString: "15/03/2024 14:30:45",
          format: "%d/%m/%Y %H:%M:%S"
        }
      }
    }
  }
])
```

### With Error Handling

```javascript
// Parse date with fallback
db.records.aggregate([
  {
    $project: {
      date: {
        $dateFromString: {
          dateString: "$dateField",
          onError: ISODate("2024-01-01T00:00:00Z"),
          onNull: ISODate("2024-01-01T00:00:00Z")
        }
      }
    }
  }
])
```

## Behavior

1. **String Parsing**
   - Format matching
   - Validation rules
   - Default handling

2. **Timezone Processing**
   - Timezone conversion
   - DST handling
   - UTC normalization

3. **Error Handling**
   - Invalid formats
   - Null values
   - Fallback values

## Use Cases

1. **Data Import**
   ```javascript
   // Parse imported date strings
   db.imports.aggregate([
     {
       $project: {
         parsedDate: {
           $dateFromString: {
             dateString: "$rawDate",
             format: "%Y-%m-%d"
           }
         }
       }
     }
   ])
   ```

2. **Log Processing**
   ```javascript
   // Parse log timestamps
   db.logs.aggregate([
     {
       $project: {
         timestamp: {
           $dateFromString: {
             dateString: "$logTime",
             format: "%Y%m%d%H%M%S"
           }
         }
       }
     }
   ])
   ```

3. **Data Cleanup**
   ```javascript
   // Clean and normalize dates
   db.records.aggregate([
     {
       $project: {
         normalizedDate: {
           $dateFromString: {
             dateString: "$dateString",
             onError: "$$REMOVE"
           }
         }
       }
     }
   ])
   ```

## Performance Considerations

1. **Format Parsing**
   - Pattern complexity
   - String length
   - Format validation

2. **Error Handling**
   - Validation cost
   - Fallback processing
   - Exception handling

3. **Timezone Operations**
   - Conversion overhead
   - DST calculations
   - Cache utilization

## Best Practices

1. **Format Specification**
   - Use clear formats
   - Document patterns
   - Handle variations

2. **Error Handling**
   - Provide fallbacks
   - Validate inputs
   - Log errors

3. **Timezone Management**
   - Specify when needed
   - Handle DST
   - Document assumptions

## Common Patterns

### Standard Date Parsing

```javascript
// Parse standard date format
{
  $project: {
    date: {
      $dateFromString: {
        dateString: "$isoDate"
      }
    }
  }
}
```

### Custom Format Parsing

```javascript
// Parse custom date format
{
  $project: {
    timestamp: {
      $dateFromString: {
        dateString: "$customDate",
        format: "%Y-%m-%d %H:%M:%S",
        timezone: "UTC"
      }
    }
  }
}
```

### Error Handling

```javascript
// Handle invalid dates
{
  $project: {
    validDate: {
      $dateFromString: {
        dateString: "$inputDate",
        onError: { $const: null },
        onNull: { $const: null }
      }
    }
  }
}
```

## Related Operators

- [$dateToString](../string/datetostring.md) - Converts a date to a formatted string
- [$dateFromParts](datefromparts.md) - Constructs a date from individual parts
- [$dateToParts](../aggregation/datetoparts.md) - Breaks down a date into its component parts 