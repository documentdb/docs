# $meta

Projects metadata from document queries.

## Syntax

```javascript
{ <field>: { $meta: <metaDataKeyword> } }
```

Valid metadata keywords:
- `"textScore"` - Returns the score associated with a text search
- `"indexKey"` - Returns the index key used to perform the query
- `"sortKey"` - Returns the key used to sort the document

## Description

The `$meta` operator projects metadata about the document in query results. It's commonly used with text search to access relevance scores and with index operations to understand query execution.

## Examples

### Text Search Score

```javascript
// Perform text search and include score
db.articles.find(
  { $text: { $search: "mongodb database" } },
  {
    title: 1,
    score: { $meta: "textScore" }
  }
).sort({ score: { $meta: "textScore" } })
```

### Index Key Information

```javascript
// Get index key used for query
db.collection.find(
  { status: "active" },
  {
    indexKey: { $meta: "indexKey" }
  }
)
```

### Sort Key Information

```javascript
// Get sort key information
db.orders.find(
  {},
  {
    sortKey: { $meta: "sortKey" }
  }
).sort({ date: -1 })
```

## Behavior

1. **Text Search**
   - Returns relevance score
   - Score range 0 to 1
   - Higher score means better match
   - Requires text index

2. **Index Operations**
   - Shows index usage
   - Helps query optimization
   - Provides execution insight
   - Diagnostic information

3. **Sort Operations**
   - Reveals sort criteria
   - Shows sort direction
   - Helps understand ordering
   - Performance analysis

## Use Cases

1. **Text Search**
   - Relevance ranking
   - Content matching
   - Search optimization
   - Result sorting

2. **Query Optimization**
   - Index usage analysis
   - Query performance
   - Execution planning
   - Index selection

3. **Debugging**
   - Query troubleshooting
   - Performance analysis
   - Index verification
   - Sort validation

## Best Practices

1. **Text Search**
   - Create appropriate text indexes
   - Consider language settings
   - Plan scoring weights
   - Handle multiple fields

2. **Performance**
   - Monitor metadata overhead
   - Use selective projection
   - Consider result size
   - Optimize query patterns

3. **Index Usage**
   - Analyze index selection
   - Monitor index efficiency
   - Plan index strategy
   - Review query plans

## Related Operators

- [$text](../query/text.md) - Performs text search
- [$sort](../aggregation/sort.md) - Sorts documents in the pipeline 