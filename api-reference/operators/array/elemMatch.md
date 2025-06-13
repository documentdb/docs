# $elemMatch

Matches documents that contain an array field with at least one element that matches all the specified query criteria.

## Syntax

```javascript
{ <field>: { $elemMatch: { <query1>, <query2>, ... } } }
```

## Description

The `$elemMatch` operator matches documents that contain an array field with at least one element that matches all the specified query criteria. It allows you to match multiple conditions against the same array element.

## Examples

### Basic Element Match

```javascript
// Sample document
{
  _id: 1,
  scores: [
    { subject: "math", score: 85 },
    { subject: "english", score: 92 },
    { subject: "science", score: 78 }
  ]
}

// Find documents with at least one score between 80 and 90
db.students.find({
  scores: {
    $elemMatch: {
      score: { $gte: 80, $lte: 90 }
    }
  }
})
```

### Multiple Conditions

```javascript
// Sample document
{
  _id: 2,
  results: [
    { product: "A", quantity: 5, price: 10 },
    { product: "B", quantity: 2, price: 20 },
    { product: "C", quantity: 7, price: 15 }
  ]
}

// Find documents with products that have quantity > 5 and price < 20
db.orders.find({
  results: {
    $elemMatch: {
      quantity: { $gt: 5 },
      price: { $lt: 20 }
    }
  }
})
```

### Nested Array Elements

```javascript
// Sample document
{
  _id: 3,
  surveys: [
    {
      responses: [
        { question: 1, answer: "yes", score: 1 },
        { question: 2, answer: "no", score: 0 }
      ],
      date: ISODate("2024-01-15")
    }
  ]
}

// Find documents with specific survey responses
db.feedback.find({
  "surveys.responses": {
    $elemMatch: {
      question: 1,
      answer: "yes",
      score: { $gt: 0 }
    }
  }
})
```

## Behavior

1. **Element Matching**
   - Matches a single array element against multiple criteria
   - All conditions must match the same element
   - Returns document if any element matches

2. **Query Conditions**
   - Supports comparison operators
   - Can use logical operators
   - Allows regular expressions

3. **Array Requirements**
   - Field must be an array
   - At least one element must match all criteria
   - Does not match non-array fields

## Use Cases

1. **Complex Data Validation**
   - Validating array elements
   - Checking multiple conditions
   - Finding specific patterns

2. **Search and Filter**
   - Advanced search functionality
   - Filtering array elements
   - Complex query conditions

3. **Data Analysis**
   - Finding specific data patterns
   - Analyzing array elements
   - Complex data queries

## Performance Considerations

1. **Index Usage**
   - Create compound indexes for multiple fields
   - Use multikey indexes for array fields
   - Consider index selectivity

2. **Query Optimization**
   - Place selective conditions first
   - Use appropriate array field indexes
   - Consider query patterns

3. **Memory Usage**
   - Monitor array sizes
   - Consider document size limits
   - Be mindful of complex queries

## Best Practices

1. **Query Design**
   - Use specific conditions
   - Combine with other operators when needed
   - Consider query readability

2. **Performance**
   - Create appropriate indexes
   - Monitor query performance
   - Optimize for common patterns

3. **Maintenance**
   - Document query patterns
   - Review index usage
   - Monitor system performance

## Related Operators

- [$all](all.md) - Matches arrays containing all specified elements
- [$size](size.md) - Matches arrays of specific length
- [$and](../query/and.md) - Joins query clauses with logical AND 