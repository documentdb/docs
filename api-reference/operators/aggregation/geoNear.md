# $geoNear

Returns documents ordered by their proximity to a specified geospatial point.

## Syntax

```javascript
{
  $geoNear: {
    near: <point>,
    distanceField: <string>,
    spherical: <boolean>,
    maxDistance: <number>,
    query: <document>,
    distanceMultiplier: <number>,
    includeLocs: <string>,
    uniqueDocs: <boolean>,
    minDistance: <number>,
    key: <string>
  }
}
```

## Parameters

| Parameter | Type | Description | Required |
|-----------|------|-------------|----------|
| `near` | Point or GeoJSON Point | The point to calculate distances from | Yes |
| `distanceField` | string | The field to store the calculated distance | Yes |
| `spherical` | boolean | Whether to use spherical geometry | No |
| `maxDistance` | number | Maximum distance in meters from point | No |
| `query` | document | Additional query conditions | No |
| `distanceMultiplier` | number | Factor to multiply distances by | No |
| `includeLocs` | string | Field to store the location used for calculation | No |
| `uniqueDocs` | boolean | Whether to return unique documents | No |
| `minDistance` | number | Minimum distance in meters from point | No |
| `key` | string | Index key to use for geospatial query | No |

## Description

The `$geoNear` stage performs a geospatial search on a collection and returns documents ordered by distance from a specified point. It must be the first stage in an aggregation pipeline and requires a geospatial index.

## Examples

### Basic Distance Search

```javascript
db.places.aggregate([
  {
    $geoNear: {
      near: { type: "Point", coordinates: [-73.98, 40.75] },
      distanceField: "distance",
      maxDistance: 5000,
      spherical: true
    }
  }
])
```

### Search with Additional Query Conditions

```javascript
db.restaurants.aggregate([
  {
    $geoNear: {
      near: { type: "Point", coordinates: [-73.98, 40.75] },
      distanceField: "distance",
      query: { cuisine: "Italian", rating: { $gte: 4 } },
      maxDistance: 2000,
      spherical: true
    }
  },
  {
    $project: {
      name: 1,
      distance: 1,
      rating: 1,
      cuisine: 1
    }
  }
])
```

### Distance Calculation with Custom Units

```javascript
db.locations.aggregate([
  {
    $geoNear: {
      near: { type: "Point", coordinates: [-73.98, 40.75] },
      distanceField: "distanceKm",
      distanceMultiplier: 0.001, // Convert meters to kilometers
      spherical: true,
      query: { type: "store" },
      includeLocs: "location"
    }
  },
  {
    $project: {
      name: 1,
      distanceKm: { $round: ["$distanceKm", 2] },
      location: 1
    }
  }
])
```

### Finding Locations Within Distance Range

```javascript
db.places.aggregate([
  {
    $geoNear: {
      near: { type: "Point", coordinates: [-73.98, 40.75] },
      distanceField: "distance",
      minDistance: 1000,
      maxDistance: 5000,
      spherical: true,
      query: {
        openingHours: { $exists: true },
        isActive: true
      }
    }
  },
  {
    $group: {
      _id: "$type",
      locations: { $push: { name: "$name", distance: "$distance" } },
      avgDistance: { $avg: "$distance" }
    }
  }
])
```

## Behavior

1. **Stage Position**
   - Must be the first stage in the pipeline
   - Only one `$geoNear` stage per pipeline

2. **Index Requirements**
   - Requires a geospatial index
   - For 2dsphere index: use `spherical: true`
   - For 2d index: use `spherical: false`

3. **Distance Calculation**
   - For 2dsphere: distances in meters
   - For 2d: distances in radians
   - Use `distanceMultiplier` to convert units

4. **Result Ordering**
   - Documents sorted by distance (nearest first)
   - Consistent with `$near` operator behavior

## Use Cases

1. **Location-Based Services**
   - Finding nearby points of interest
   - Delivery radius calculations
   - Store locator applications

2. **Geospatial Analytics**
   - Distance-based clustering
   - Coverage area analysis
   - Service area planning

3. **Mobile Applications**
   - Location-aware features
   - Proximity alerts
   - Geographic content filtering

## Performance Considerations

1. **Index Selection**
   - Use 2dsphere index for geographic coordinates
   - Use 2d index for planar coordinates
   - Ensure index covers query conditions

2. **Query Optimization**
   - Use `maxDistance` to limit search radius
   - Include selective query conditions
   - Consider index size and coverage

3. **Memory Usage**
   - Results must fit in memory
   - Use projection to limit field return
   - Consider pagination for large result sets

## Related Operators

- [$near](../query/near.md) - Geospatial query operator
- [$nearSphere](../query/nearSphere.md) - Spherical geospatial query
- [$geometry](../query/geometry.md) - Specifies GeoJSON geometry
- [$maxDistance](../query/maxDistance.md) - Maximum distance filter 