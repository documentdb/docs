# $elemMatch (projection)

Projects the first array element that matches the specified condition.

## Syntax

```javascript
{ <field>: { $elemMatch: { <condition> } } }
```

## Description

The `$elemMatch` projection operator returns the first element in an array that matches the specified condition. This is different from the `$elemMatch` query operator, which selects documents containing an array element that matches the condition.

## Examples

### Basic Array Element Matching

```javascript
// Sample document
{
  _id: 1,
  scores: [
    { subject: "math", score: 80 },
    { subject: "english", score: 90 },
    { subject: "science", score: 95 }
  ]
}

// Query with projection
db.students.find(
  {},
  {
    scores: {
      $elemMatch: { score: { $gt: 85 } }
    }
  }
)

// Result
{
  _id: 1,
  scores: [
    { subject: "english", score: 90 }
  ]
}
```

### Multiple Conditions

```javascript
db.inventory.find(
  {},
  {
    items: {
      $elemMatch: {
        price: { $lt: 100 },
        inStock: true
      }
    }
  }
)
```

### With Array of Primitives

```javascript
// Sample document
{
  _id: 1,
  numbers: [1, 4, 6, 8, 12, 15]
}

// Query with projection
db.numbers.find(
  {},
  {
    numbers: {
      $elemMatch: { $gt: 10 }
    }
  }
)

// Result
{
  _id: 1,
  numbers: [12]
}
```

## Behavior

1. **Matching**
   - Returns first matching element
   - Preserves array structure
   - Maintains element order
   - Returns null if no match

2. **Array Types**
   - Works with array of documents
   - Works with array of primitives
   - Preserves element structure
   - Maintains data types

3. **Field Handling**
   - Includes _id by default
   - Other fields need explicit inclusion
   - Supports multiple array fields
   - Preserves array field name

## Use Cases

1. **Data Filtering**
   - Selective array element retrieval
   - Condition-based projection
   - Data sampling
   - Result limiting

2. **Performance Optimization**
   - Reducing result size
   - Network bandwidth optimization
   - Memory usage control
   - Query efficiency

3. **Data Analysis**
   - Element inspection
   - Condition validation
   - Sample extraction
   - Pattern matching

## Best Practices

1. **Query Design**
   - Use specific conditions
   - Consider array size
   - Plan field inclusion
   - Optimize matching criteria

2. **Performance**
   - Use appropriate indexes
   - Minimize result size
   - Consider query selectivity
   - Monitor execution time

3. **Data Handling**
   - Validate array fields
   - Handle empty arrays
   - Consider null values
   - Plan error cases

## Related Operators

- [$slice](slice.md) - Projects a subset of elements from an array
- [$meta](meta.md) - Projects metadata from document queries 