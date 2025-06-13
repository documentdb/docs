# $comment

Adds a comment to a query expression to help identify queries in logs and profiles.

## Syntax

```javascript
{ $comment: <string> }
```

## Description

The `$comment` operator allows you to add a comment to your query. The comment appears in the following:
- Database profiler output
- mongod log messages
- Explain plan output
- System.profile collection entries

## Examples

### Basic Comment Usage

```javascript
// Adding a comment to a find query
db.users.find(
  {
    status: "active",
    $comment: "Query to find active users"
  }
)
```

### Comment in Aggregation Pipeline

```javascript
db.orders.aggregate([
  {
    $match: {
      $comment: "Filter orders from last month",
      date: {
        $gte: new Date("2024-01-01"),
        $lt: new Date("2024-02-01")
      }
    }
  },
  {
    $group: {
      $comment: "Group by product category",
      _id: "$category",
      total: { $sum: "$amount" }
    }
  }
])
```

### Comment with Complex Query

```javascript
db.inventory.find({
  $comment: "Find items below reorder point",
  $and: [
    { quantity: { $lt: "$reorderPoint" } },
    { status: "active" },
    { discontinued: false }
  ]
})
```

## Behavior

1. **Comment Placement**
   - Can be used in any query operation
   - Multiple comments allowed in aggregation pipeline
   - No effect on query results

2. **Visibility**
   - Appears in profiler output
   - Visible in explain plans
   - Logged in diagnostic logs

3. **Performance Impact**
   - Minimal overhead
   - No impact on query execution
   - No impact on results

## Use Cases

1. **Query Identification**
   - Tracking specific queries
   - Debugging complex operations
   - Performance monitoring

2. **Documentation**
   - Explaining query purpose
   - Noting business rules
   - Marking query versions

3. **Maintenance**
   - Identifying query sources
   - Tracking query changes
   - Auditing query usage

## Best Practices

1. **Comment Content**
   - Use descriptive comments
   - Include relevant context
   - Add version information
   - Reference related tickets/issues

2. **Usage Guidelines**
   - Comment complex queries
   - Include business logic explanation
   - Document assumptions
   - Note performance considerations

3. **Maintenance**
   - Keep comments up to date
   - Remove obsolete comments
   - Update as queries change

## Related Operators

None - The `$comment` operator is a standalone utility operator. 