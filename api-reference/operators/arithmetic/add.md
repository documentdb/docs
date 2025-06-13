# $add

Adds numbers together or adds numbers and dates.

## Syntax

```javascript
{ $add: [ <expression1>, <expression2>, ... ] }
```

## Description

The `$add` operator adds numbers and/or dates together. It can:
- Add multiple numbers together
- Add milliseconds to a date
- Add multiple numbers to a date

## Examples

### Adding Numbers

```javascript
db.sales.aggregate([
  {
    $project: {
      item: 1,
      total: { $add: ["$price", "$tax", "$shipping"] }
    }
  }
])
```

Input documents:
```javascript
{ "_id": 1, "item": "book", "price": 20, "tax": 1.6, "shipping": 5 }
```

Output documents:
```javascript
{ "_id": 1, "item": "book", "total": 26.6 }
```

### Adding to Dates

```javascript
db.events.aggregate([
  {
    $project: {
      event: 1,
      startDate: 1,
      endDate: {
        $add: ["$startDate", { $multiply: ["$durationHours", 3600000] }]  // Convert hours to milliseconds
      }
    }
  }
])
```

### Complex Calculations

```javascript
db.orders.aggregate([
  {
    $project: {
      order: 1,
      // Calculate total with weighted tax rates
      totalWithTax: {
        $add: [
          "$subtotal",
          { $multiply: ["$localTaxRate", "$subtotal"] },
          { $multiply: ["$stateTaxRate", "$subtotal"] },
          "$shipping"
        ]
      },
      // Calculate delivery date
      estimatedDelivery: {
        $add: [
          "$orderDate",
          { $multiply: ["$processingDays", 24 * 60 * 60 * 1000] }  // Convert days to milliseconds
        ]
      }
    }
  }
])
```

### Array Sum with $reduce

```javascript
db.products.aggregate([
  {
    $project: {
      item: 1,
      prices: 1,
      totalCost: {
        $reduce: {
          input: "$prices",
          initialValue: 0,
          in: { $add: ["$$value", "$$this"] }
        }
      }
    }
  }
])
```

## Behavior

1. **Number Addition**
   - Adds multiple numeric values
   - Maintains decimal precision
   - Handles integer and decimal operands

2. **Date Addition**
   - Adds milliseconds to dates
   - Returns new Date object
   - Handles timezone correctly

3. **Type Handling**
   - Numbers: Standard addition
   - Dates: Adds milliseconds
   - Mixed: Date + Number = Date
   - Null: Results in null

4. **Error Cases**
   - Non-numeric/non-date values
   - Invalid date values
   - Overflow conditions

## Use Cases

1. **Financial Calculations**
   - Total price computation
   - Tax calculations
   - Fee aggregation
   - Budget summaries

2. **Date Operations**
   - Delivery date calculation
   - Event scheduling
   - Timeline generation
   - Duration computation

3. **Inventory Management**
   - Stock level updates
   - Reorder calculations
   - Capacity planning

4. **Analytics**
   - Running totals
   - Cumulative metrics
   - Growth calculations

## Examples in Aggregation Pipeline

### Running Total

```javascript
db.transactions.aggregate([
  {
    $sort: { date: 1 }
  },
  {
    $group: {
      _id: "$accountId",
      runningBalance: {
        $reduce: {
          input: "$amounts",
          initialValue: 0,
          in: { $add: ["$$value", "$$this"] }
        }
      },
      transactions: { $push: "$$ROOT" }
    }
  }
])
```

### Complex Date Calculations

```javascript
db.shipments.aggregate([
  {
    $project: {
      shipment: 1,
      estimatedArrival: {
        $add: [
          "$departureDate",
          { $multiply: ["$transitDays", 24 * 60 * 60 * 1000] },  // Transit time
          { $multiply: ["$processingDays", 24 * 60 * 60 * 1000] },  // Processing time
          { $multiply: ["$customsClearance", 24 * 60 * 60 * 1000] }  // Customs time
        ]
      }
    }
  }
])
```

## Performance Considerations

1. **Optimization**
   - Use compound operations when possible
   - Consider indexing for filtered calculations
   - Pre-calculate static values

2. **Memory Usage**
   - Efficient for simple additions
   - Consider chunking for large arrays
   - Monitor for date range limits

3. **Date Calculations**
   - Be aware of timezone implications
   - Handle DST transitions
   - Consider date validation

## Best Practices

1. **Type Safety**
   - Validate input types
   - Handle null values explicitly
   - Check for overflow conditions

2. **Date Operations**
   - Use consistent time units
   - Account for timezone differences
   - Validate date ranges

3. **Numeric Precision**
   - Consider decimal precision requirements
   - Handle rounding appropriately
   - Document precision expectations

## Related Operators

- [$subtract](subtract.md) - Subtracts two values
- [$multiply](multiply.md) - Multiplies numbers
- [$divide](divide.md) - Divides numbers
- [$sum](../accumulator/sum.md) - Sums numeric values 