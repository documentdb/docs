# $minDistance

The `$minDistance` operator specifies the minimum distance to limit the results of a geospatial query.

## Syntax

```javascript
{
  $near: {
    $geometry: {
      type: "Point",
      coordinates: [ <longitude>, <latitude> ]
    },
    $minDistance: <distance in meters>
  }
}
```

## Description

The `$minDistance` operator:
- Specifies the minimum distance for `$near` and `$nearSphere` queries
- Distance is measured in meters when using GeoJSON points
- Used to exclude results too close to the query point
- Works with both 2dsphere and 2d indexes
- Must be used within a `$near` or `$nearSphere` operator

## Examples

### Basic Distance Query

```javascript
// Find places at least 1 kilometer away
db.places.find({
   location: {
      $near: {
         $geometry: {
            type: "Point",
            coordinates: [-73.93, 40.82]
         },
         $minDistance: 1000  // 1 kilometer in meters
      }
   }
})
```

### Combined with Maximum Distance

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

### With Additional Query Criteria

```javascript
// Find restaurants at least 500m away that are currently open
db.restaurants.find({
   location: {
      $near: {
         $geometry: {
            type: "Point",
            coordinates: [-0.127, 51.507]
         },
         $minDistance: 500
      }
   },
   isOpen: true,
   rating: { $gte: 4 }
})
```

## Behavior

- Distance is measured in meters when using GeoJSON
- Results are automatically sorted by distance (nearest first)
- Works with both spherical and flat geometries
- Must be a positive number
- Can be combined with `$maxDistance`
- Returns error if distance is invalid

## Performance Considerations

- Requires a geospatial index
- More efficient with smaller distance ranges
- Performance improves with more selective queries
- Index usage is optimal when combined with other query criteria
- Consider chunking large distance ranges

## Index Requirements

```javascript
// Create a 2dsphere index for GeoJSON points
db.collection.createIndex({ location: "2dsphere" })

// Or create a 2d index for legacy coordinates
db.collection.createIndex({ location: "2d" })
```

## Limitations

- Must be used with `$near` or `$nearSphere`
- Cannot be used with `$geoWithin`
- Distance must be positive
- May not be exact for legacy coordinates
- Cannot be greater than `$maxDistance` when both are used

## Related Operators

- [$maxDistance](maxDistance.md) - Specifies maximum distance
- [$near](near.md) - Selects points near a point
- [$nearSphere](nearSphere.md) - Selects points near a point on a sphere
- [$geometry](geometry.md) - Specifies a GeoJSON geometry 