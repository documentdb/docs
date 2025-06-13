# $size

Matches arrays with a specific number of elements.

## Syntax

```javascript
{ <field>: { $size: <number> } }
```

## Description

The `$size` operator matches any array with the specified number of elements. It provides a way to query documents based on the length of array fields.

## Examples

### Basic Size Match

```javascript
// Sample document
{
  _id: 1,
  tags: ["red", "green", "blue"]
}

// Find documents where tags array has exactly 3 elements
db.items.find({ tags: { $size: 3 } })
```

### Combining with Other Operators

```javascript
// Sample document
{
  _id: 2,
  categories: ["electronics", "mobile"],
  reviews: [
    { rating: 4, text: "Good product" },
    { rating: 5, text: "Excellent" }
  ]
}

// Find documents with exactly 2 categories and at least one 5-star review
db.products.find({
  categories: { $size: 2 },
  reviews: {
    $elemMatch: { rating: 5 }
  }
})
```

### Nested Arrays

```javascript
// Sample document
{
  _id: 3,
  name: "Course A",
  modules: [
    {
      name: "Module 1",
      lessons: ["1.1", "1.2", "1.3"]
    },
    {
      name: "Module 2",
      lessons: ["2.1", "2.2"]
    }
  ]
}

// Find courses where a module has exactly 2 lessons
db.courses.find({
  "modules.lessons": { $size: 2 }
})
```

## Behavior

1. **Size Matching**
   - Matches exact array length only
   - Cannot use comparison operators with $size
   - Must be a non-negative integer

2. **Field Requirements**
   - Field must be an array
   - Does not match non-array fields
   - Null or missing fields don't match

3. **Limitations**
   - Cannot compare sizes (e.g., greater than)
   - Cannot use ranges
   - Must use exact numbers

## Use Cases

1. **Data Validation**
   - Verifying array completeness
   - Checking required elements
   - Validating data structure

2. **Content Management**
   - Finding complete sets
   - Managing fixed-size collections
   - Validating content structure

3. **Query Filtering**
   - Filtering by array length
   - Finding specific patterns
   - Identifying data anomalies

## Performance Considerations

1. **Query Optimization**
   - No direct index support for $size
   - Consider alternative query patterns
   - Use appropriate array indexes

2. **Memory Usage**
   - Efficient for size checks
   - No additional memory overhead
   - Fast array length comparison

3. **Scaling**
   - Works well with large collections
   - Consistent performance
   - No performance degradation

## Best Practices

1. **Query Design**
   - Use exact size matches
   - Combine with other operators
   - Consider query patterns

2. **Alternative Approaches**
   - Store array length in separate field
   - Use computed fields
   - Consider denormalization

3. **Maintenance**
   - Monitor query performance
   - Review usage patterns
   - Update indexes as needed

## Workarounds for Size Comparisons

Since `$size` only matches exact numbers, here are some workarounds for size comparisons:

### Greater Than

```javascript
// Add a size field during updates
db.collection.update({},
  { $set: { arraySize: { $size: "$arrayField" } } }
)

// Query using the size field
db.collection.find({ arraySize: { $gt: 5 } })
```

### Range Queries

```javascript
// Using $where (not recommended for production)
db.collection.find({
  $where: "this.arrayField.length >= 5 && this.arrayField.length <= 10"
})
```

## Related Operators

- [$all](all.md) - Matches arrays containing all specified elements
- [$elemMatch](elemMatch.md) - Matches array elements meeting multiple criteria
- [$exists](../query/exists.md) - Matches documents with existing fields 