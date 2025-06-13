# $all

Matches arrays that contain all elements specified in the query.

## Syntax

```javascript
{ <field>: { $all: [ <value1>, <value2>, ... ] } }
```

## Description

The `$all` operator selects documents where the value of a field is an array that contains all the specified elements. The order of the elements does not matter.

## Examples

### Basic Array Match

```javascript
// Sample document
{
  _id: 1,
  tags: ["red", "green", "blue"]
}

// Query for documents where tags contains both "red" and "blue"
db.items.find({ tags: { $all: ["red", "blue"] } })
```

### Matching Arrays with Embedded Documents

```javascript
// Sample document
{
  _id: 2,
  specifications: [
    { color: "red", size: "large" },
    { color: "blue", size: "medium" }
  ]
}

// Query using $elemMatch within $all
db.products.find({
  specifications: {
    $all: [
      { $elemMatch: { color: "red", size: "large" } },
      { $elemMatch: { color: "blue", size: "medium" } }
    ]
  }
})
```

### Combining with Other Operators

```javascript
// Find active products with all required features
db.products.find({
  status: "active",
  features: {
    $all: ["wifi", "bluetooth"],
    $size: 2  // Exactly these two features
  }
})
```

## Behavior

1. **Array Matching**
   - Matches arrays containing all specified elements
   - Order of elements is not important
   - Can match nested array elements

2. **Type Handling**
   - Elements must match exactly (type and value)
   - Can use regular expressions and subdocuments
   - Supports `$elemMatch` for complex matching

3. **Empty Arrays**
   - `$all: []` matches any array
   - Empty array in document matches if query elements exist

## Use Cases

1. **Feature Requirements**
   - Finding products with specific features
   - Matching documents with required tags
   - Filtering by multiple criteria

2. **Document Classification**
   - Categorizing documents by attributes
   - Finding items with complete sets
   - Matching multiple conditions

3. **Data Validation**
   - Ensuring all required elements exist
   - Verifying data completeness
   - Checking for multiple conditions

## Performance Considerations

1. **Index Usage**
   - Create indexes on array fields
   - Use compound indexes for combined queries
   - Consider index selectivity

2. **Query Optimization**
   - Place most selective conditions first
   - Use appropriate array size limits
   - Consider alternative query patterns

3. **Memory Impact**
   - Monitor array sizes in documents
   - Be mindful of large array comparisons
   - Consider pagination for large result sets

## Related Operators

- [$elemMatch](elemMatch.md) - Matches array elements meeting multiple criteria
- [$size](size.md) - Matches arrays of specific length
- [$in](../query/in.md) - Matches any of the specified values 