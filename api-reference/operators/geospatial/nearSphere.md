# $nearSphere

The `$nearSphere` operator returns geospatial objects in proximity to a point on a sphere, sorted by distance on a spherical surface.

## Syntax

```javascript
{
  <location field>: {
    $nearSphere: {
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

The `$nearSphere` operator:
- Returns points ordered by spherical distance from a specified point
- Uses spherical geometry for distance calculations
- Requires a geospatial index
- Can specify maximum and minimum distances
- Results are automatically sorted by distance
- More accurate for Earth-surface calculations than `$near`

## Examples

### Basic Spherical Query

```javascript
// Find places near a point using spherical geometry
db.places.find({
   location: {
      $nearSphere: {
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
// Find places between 1km and 5km away on a sphere
db.places.find({
   location: {
      $nearSphere: {
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
// Find open restaurants within 2km on Earth's surface
db.restaurants.find({
   location: {
      $nearSphere: {
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
- Uses spherical geometry for distance calculations
- Maximum of 100 results by default
- More accurate for global distance calculations

## Performance Considerations

- Requires a geospatial index
- Slightly slower than `$near` due to spherical calculations
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
- Slightly slower than `$near`
- Not suitable for queries that cross the 180th meridian
- Must use valid longitude/latitude coordinates

## Related Operators

- [$near](near.md) - Selects points near a point using flat geometry
- [$maxDistance](maxDistance.md) - Specifies maximum distance
- [$minDistance](minDistance.md) - Specifies minimum distance
- [$geometry](geometry.md) - Specifies a GeoJSON geometry 