# $match

Filters the documents to pass only those that match the specified condition(s).

## Syntax

```javascript
{ $match: { <query> } }
```

The `<query>` syntax is identical to the query syntax for the `find()` method.

## Description

The `$match` stage filters documents to allow only matching documents to pass unmodified into the next pipeline stage. `$match` uses standard MongoDB query operators and can take advantage of indexes if placed at the beginning of the pipeline.

## Examples

### Basic Match

```javascript
db.articles.aggregate([
  {
    $match: {
      status: "published",
      views: { $gt: 100 }
    }
  }
])
```

### Match with Multiple Conditions

```javascript
db.sales.aggregate([
  {
    $match: {
      $and: [
        { date: { $gte: new Date("2024-01-01") } },
        { status: "completed" },
        { "customer.type": "premium" },
        { total: { $gt: 100 } }
      ]
    }
  },
  {
    $group: {
      _id: "$customer.country",
      totalSales: { $sum: "$total" }
    }
  }
])
```

### Match with Regular Expression

```javascript
db.products.aggregate([
  {
    $match: {
      name: { $regex: /^iPhone/, $options: "i" },
      category: "Electronics",
      inStock: true
    }
  },
  {
    $project: {
      name: 1,
      price: 1,
      specifications: 1
    }
  }
])
```

### Match with Array Elements

```javascript
db.inventory.aggregate([
  {
    $match: {
      tags: { $all: ["electronics", "mobile"] },
      "specs.color": { $in: ["black", "silver"] },
      quantity: { $exists: true, $gt: 0 }
    }
  },
  {
    $sort: { price: -1 }
  }
])
```

### Match with Geospatial Query

```javascript
db.restaurants.aggregate([
  {
    $match: {
      location: {
        $geoWithin: {
          $centerSphere: [[-73.93, 40.82], 5 / 3963.2]
        }
      },
      cuisine: "Italian",
      rating: { $gte: 4 }
    }
  },
  {
    $project: {
      name: 1,
      address: 1,
      rating: 1,
      distance: 1
    }
  }
])
```

## Query Operators

### Comparison
- `$eq`: Equals
- `$gt`: Greater than
- `$gte`: Greater than or equal
- `$lt`: Less than
- `$lte`: Less than or equal
- `$ne`: Not equal
- `$in`: In array
- `$nin`: Not in array

### Logical
- `$and`: Logical AND
- `$or`: Logical OR
- `$not`: Logical NOT
- `$nor`: Logical NOR

### Element
- `$exists`: Field exists
- `$type`: BSON type

### Array
- `$all`: All elements match
- `$elemMatch`: Element matches
- `$size`: Array size

### Evaluation
- `$regex`: Regular expression
- `$text`: Text search
- `$where`: JavaScript expression

## Behavior

1. **Pipeline Position**
   - Can be used multiple times in a pipeline
   - Most efficient when used early
   - Takes advantage of indexes when first

2. **Query Optimization**
   - Uses same query engine as `find()`
   - Benefits from existing indexes
   - Supports query planner hints

3. **Memory Usage**
   - Streams documents through pipeline
   - Does not require significant memory
   - Efficient for large datasets

## Use Cases

1. **Data Filtering**
   - Removing unwanted documents
   - Selecting specific categories
   - Date range filtering

2. **Complex Queries**
   - Multi-condition matching
   - Pattern matching
   - Geospatial queries

3. **Performance Optimization**
   - Reducing pipeline load
   - Index utilization
   - Early data reduction

## Performance Considerations

1. **Index Usage**
   - Place `$match` early in pipeline
   - Use indexed fields in conditions
   - Consider compound indexes

2. **Query Structure**
   - Use selective conditions first
   - Combine related conditions
   - Avoid unnecessary complexity

3. **Memory Efficiency**
   - Filter early to reduce data flow
   - Use covered queries when possible
   - Consider batch processing

## Best Practices

1. **Pipeline Optimization**
   - Place `$match` before `$group` and `$project`
   - Use `$match` to reduce documents early
   - Combine multiple `$match` stages when possible

2. **Query Design**
   - Use appropriate operators
   - Structure queries for index usage
   - Consider query selectivity

3. **Performance**
   - Monitor query performance
   - Use explain plans
   - Optimize indexes for common queries

## Related Operators

- [$project](project.md) - Reshapes documents
- [$group](group.md) - Groups documents
- [$sort](sort.md) - Sorts documents
- [$limit](limit.md) - Limits number of documents 