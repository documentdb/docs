# $facet

Processes multiple aggregation pipelines within a single stage on the same set of input documents.

## Syntax

```javascript
{
  $facet: {
    <outputField1>: [ <stage1>, <stage2>, ... ],
    <outputField2>: [ <stage1>, <stage2>, ... ],
    ...
  }
}
```

## Description

The `$facet` operator allows you to create multi-faceted aggregations that characterize data across multiple dimensions (or facets) within a single aggregation stage. Each facet can have its own pipeline, and all pipelines operate on the same input documents.

## Examples

### Basic Faceted Search

```javascript
db.products.aggregate([
  {
    $facet: {
      "categoryCounts": [
        { $group: { _id: "$category", count: { $sum: 1 } } }
      ],
      "priceTiers": [
        { $bucket: {
            groupBy: "$price",
            boundaries: [0, 50, 100, 200, 500],
            default: "500+",
            output: { count: { $sum: 1 } }
          }
        }
      ],
      "topRated": [
        { $match: { rating: { $gte: 4.5 } } },
        { $limit: 5 },
        { $project: { name: 1, rating: 1, _id: 0 } }
      ]
    }
  }
])
```

### Analytics Dashboard

```javascript
db.sales.aggregate([
  {
    $facet: {
      "dailyRevenue": [
        { $group: {
            _id: { $dateToString: { format: "%Y-%m-%d", date: "$date" } },
            revenue: { $sum: "$amount" }
          }
        },
        { $sort: { _id: -1 } },
        { $limit: 30 }
      ],
      "topProducts": [
        { $group: {
            _id: "$product",
            totalSales: { $sum: "$amount" }
          }
        },
        { $sort: { totalSales: -1 } },
        { $limit: 5 }
      ],
      "customerStats": [
        { $group: {
            _id: "$customerId",
            purchases: { $sum: 1 },
            totalSpent: { $sum: "$amount" }
          }
        },
        { $group: {
            _id: null,
            avgPurchasesPerCustomer: { $avg: "$purchases" },
            avgSpentPerCustomer: { $avg: "$totalSpent" }
          }
        }
      ]
    }
  }
])
```

### Geospatial Analysis

```javascript
db.locations.aggregate([
  {
    $facet: {
      "nearbyLocations": [
        { $geoNear: {
            near: { type: "Point", coordinates: [-73.98, 40.75] },
            distanceField: "distance",
            maxDistance: 5000,
            spherical: true
          }
        },
        { $limit: 10 }
      ],
      "locationsByType": [
        { $group: { _id: "$type", count: { $sum: 1 } } }
      ],
      "popularHours": [
        { $unwind: "$visitTimes" },
        { $group: {
            _id: "$visitTimes.hour",
            avgVisitors: { $avg: "$visitTimes.visitors" }
          }
        },
        { $sort: { avgVisitors: -1 } }
      ]
    }
  }
])
```

## Behavior

- Each facet operates independently on the same input documents
- Output is a single document with fields for each facet
- Each facet field contains an array of results from its pipeline
- Cannot use `$out`, `$merge`, or `$facet` stages within a facet pipeline
- Memory limit applies to the combined memory used by all facets

## Use Cases

1. **Multi-dimensional Analytics**
   - Calculating metrics across different dimensions
   - Generating dashboard data
   - Creating report summaries

2. **Search Interfaces**
   - Faceted search implementations
   - Category and filter counts
   - Search result statistics

3. **Data Profiling**
   - Analyzing data distributions
   - Computing summary statistics
   - Quality metrics calculation

4. **Real-time Analytics**
   - Concurrent metric calculations
   - Performance monitoring
   - User behavior analysis

## Performance Considerations

1. **Memory Usage**
   - Each facet pipeline maintains its own memory buffer
   - Total memory usage is the sum of all facet pipelines
   - Consider using `$limit` or `$sample` to reduce memory usage

2. **Optimization**
   - Place `$match` stages early in facet pipelines
   - Use indexes effectively in each facet
   - Consider breaking very complex facets into separate aggregations

3. **Parallelization**
   - Facets can be processed in parallel
   - Balance between number of facets and system resources
   - Consider hardware capabilities when designing facets

## Related Operators

- [$bucket](bucket.md) - Categorizes documents into groups
- [$group](group.md) - Groups documents by specified expression
- [$match](match.md) - Filters documents in the pipeline
- [$sort](sort.md) - Sorts documents in the pipeline 