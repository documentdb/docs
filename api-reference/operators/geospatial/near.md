# $near

The `$near` operator returns geospatial objects in proximity to a point, sorted by distance.

## Syntax

```javascript
{
  <location field>: {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [ <longitude>, <latitude> ]
      },
      $maxDistance: <distance in meters>,
      $minDistance: <distance in meters>
    }
  }
}
```

## Description

The `$near` operator:
- Returns points ordered by distance from a specified point
- Supports both GeoJSON points and legacy coordinates
- Requires a geospatial index
- Can specify maximum and minimum distances
- Results are automatically sorted by distance

## Examples

### Basic Proximity Query

```javascript
// Find places near a point
db.places.find({
   location: {
      $near: {
         $geometry: {
            type: "Point",
            coordinates: [-73.93, 40.82]
         },
         $maxDistance: 5000  // 5 kilometers
      }
   }
})
```

### With Distance Range

```javascript
// Find places between 1km and 5km away
db.places.find({
   location: {
      $near: {
         $geometry: {
            type: "Point",
            coordinates: [-122.27, 37.80]
         },
         $minDistance: 1000,  // 1 kilometer
         $maxDistance: 5000   // 5 kilometers
      }
   }
})
```

### With Additional Filters

```javascript
// Find open restaurants within 2km, sorted by distance
db.restaurants.find({
   location: {
      $near: {
         $geometry: {
            type: "Point",
            coordinates: [-0.127, 51.507]
         },
         $maxDistance: 2000
      }
   },
   isOpen: true,
   cuisine: "Italian"
})
```

## Behavior

- Results are automatically sorted by distance (nearest first)
- Requires a geospatial index
- Returns error if no suitable index exists
- Distances are in meters when using GeoJSON
- Can be used with both 2dsphere and 2d indexes
- Maximum of 100 results by default

## Performance Considerations

- Requires a geospatial index
- Performance depends on:
  - Number of results
  - Distance range
  - Additional query criteria
- More efficient with smaller distances
- Consider using limit() for large result sets
- Index usage is optimal with GeoJSON points

## Index Requirements

```javascript
// Create a 2dsphere index for GeoJSON points
db.collection.createIndex({ location: "2dsphere" })

// Or create a 2d index for legacy coordinates
db.collection.createIndex({ location: "2d" })
```

## Limitations

- Maximum of 100 results by default
- Requires a geospatial index
- Cannot use hint() with geospatial queries
- Distance calculations may be approximate
- Not suitable for queries that cross the 180th meridian

## Related Operators

- [$nearSphere](nearSphere.md) - Selects points near a point on a sphere
- [$maxDistance](maxDistance.md) - Specifies maximum distance
- [$minDistance](minDistance.md) - Specifies minimum distance
- [$geometry](geometry.md) - Specifies a GeoJSON geometry 