---
title: $bucketAuto
description: The $bucketAuto stage in the aggregation pipeline categorizes documents into a specified number of evenly distributed buckets.
type: operators
category: aggregation
---

# $bucketAuto

The `$bucketAuto` stage in the aggregation pipeline categorizes documents into a specified number of evenly distributed buckets. Unlike `$bucket`, which requires manually defined boundaries, `$bucketAuto` automatically determines the bucket boundaries to evenly distribute documents across the specified number of buckets. This stage is useful for creating histograms and distributing data into groups without knowing the exact range of values.

## Syntax

```javascript
{
  $bucketAuto: {
    groupBy: <expression>,
    buckets: <number>,
    output: {
      <outputField1>: { <accumulator1>: <expression1> },
      ...
    },
    granularity: <string>
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`groupBy`** | Required. An expression to group documents by. The expression must resolve to a value that can be compared and sorted. |
| **`buckets`** | Required. A positive integer specifying the number of buckets to create. |
| **`output`** | Optional. An object specifying computed fields for each bucket using accumulator expressions such as `$sum`, `$avg`, `$min`, `$max`, etc. |
| **`granularity`** | Optional. A string specifying the preferred number series to use for bucket boundaries. Supported values include `"R5"`, `"R10"`, `"R20"`, `"R40"`, `"R80"`, `"1-2-5"`, `"E6"`, `"E12"`, `"E24"`, `"E48"`, `"E96"`, `"E192"`, and `"POWERSOF2"`. |

## Examples

Consider this sample document from the stores collection.

```json
{
    "_id": "0fcc0bf0-ed18-4ab8-b558-9848e18058f4",
    "name": "First Up Consultants | Beverage Shop - Satterfieldmouth",
    "location": {
        "lat": -89.2384,
        "lon": -46.4012
    },
    "staff": {
        "totalStaff": {
            "fullTime": 8,
            "partTime": 20
        }
    },
    "sales": {
        "totalSales": 75670,
        "salesByCategory": [
            {
                "categoryName": "Wine Accessories",
                "totalSales": 34440
            },
            {
                "categoryName": "Bitters",
                "totalSales": 39496
            },
            {
                "categoryName": "Rum",
                "totalSales": 1734
            }
        ]
    },
    "promotionEvents": [
        {
            "eventName": "Unbeatable Bargain Bash",
            "promotionalDates": {
                "startDate": {
                    "Year": 2024,
                    "Month": 6,
                    "Day": 23
                },
                "endDate": {
                    "Year": 2024,
                    "Month": 7,
                    "Day": 2
                }
            },
            "discounts": [
                {
                    "categoryName": "Whiskey",
                    "discountPercentage": 7
                },
                {
                    "categoryName": "Bitters",
                    "discountPercentage": 15
                },
                {
                    "categoryName": "Brandy",
                    "discountPercentage": 8
                },
                {
                    "categoryName": "Sports Drinks",
                    "discountPercentage": 22
                },
                {
                    "categoryName": "Vodka",
                    "discountPercentage": 19
                }
            ]
        },
        {
            "eventName": "Steal of a Deal Days",
            "promotionalDates": {
                "startDate": {
                    "Year": 2024,
                    "Month": 9,
                    "Day": 21
                },
                "endDate": {
                    "Year": 2024,
                    "Month": 9,
                    "Day": 29
                }
            },
            "discounts": [
                {
                    "categoryName": "Organic Wine",
                    "discountPercentage": 19
                },
                {
                    "categoryName": "White Wine",
                    "discountPercentage": 20
                },
                {
                    "categoryName": "Sparkling Wine",
                    "discountPercentage": 19
                },
                {
                    "categoryName": "Whiskey",
                    "discountPercentage": 17
                },
                {
                    "categoryName": "Vodka",
                    "discountPercentage": 23
                }
            ]
        }
    ]
}
```

### Example 1: Distributing stores into sales buckets

To automatically distribute stores into 4 buckets based on total sales:

```javascript
db.stores.aggregate([
  {
    $bucketAuto: {
      groupBy: "$sales.totalSales",
      buckets: 4,
      output: {
        count: { $sum: 1 },
        avgSales: { $avg: "$sales.totalSales" },
        stores: { $push: "$name" }
      }
    }
  }
])
```

This query returns the following results:

```json
[
  {
    "_id": { "min": 0, "max": 18750 },
    "count": 10382,
    "avgSales": 9245.12,
    "stores": [
      "Fourth Coffee | Storage Solution Gallery - Port Camilla",
      "Contoso, Ltd. | Book Store - Lake Myron"
    ]
  },
  {
    "_id": { "min": 18750, "max": 37500 },
    "count": 10374,
    "avgSales": 28012.87,
    "stores": [
      "First Up Consultants | Bed and Bath Center - South Amir",
      "Boulder Innovations | Home Security Place - Ankundingburgh"
    ]
  },
  {
    "_id": { "min": 37500, "max": 56250 },
    "count": 10376,
    "avgSales": 46891.33,
    "stores": [
      "Lakeshore Retail | DJ Equipment Stop - Port Cecile"
    ]
  },
  {
    "_id": { "min": 56250, "max": 75670 },
    "count": 10374,
    "avgSales": 65982.45,
    "stores": [
      "First Up Consultants | Beverage Shop - Satterfieldmouth"
    ]
  }
]
```

### Example 2: Bucketing staff counts with granularity

To distribute stores into 3 buckets based on full-time staff count using the `"R5"` preferred number granularity:

```javascript
db.stores.aggregate([
  {
    $bucketAuto: {
      groupBy: "$staff.totalStaff.fullTime",
      buckets: 3,
      output: {
        count: { $sum: 1 },
        minStaff: { $min: "$staff.totalStaff.fullTime" },
        maxStaff: { $max: "$staff.totalStaff.fullTime" }
      },
      granularity: "R5"
    }
  }
])
```

This query returns the following results:

```json
[
  {
    "_id": { "min": 1, "max": 6.3 },
    "count": 13846,
    "minStaff": 1,
    "maxStaff": 6
  },
  {
    "_id": { "min": 6.3, "max": 16 },
    "count": 15214,
    "minStaff": 7,
    "maxStaff": 15
  },
  {
    "_id": { "min": 16, "max": 25 },
    "count": 12446,
    "minStaff": 16,
    "maxStaff": 25
  }
]
```

### Example 3: Creating a sales category distribution

To unwind the sales categories and distribute individual category sales into 3 automatically sized buckets:

```javascript
db.stores.aggregate([
  {
    $unwind: "$sales.salesByCategory"
  },
  {
    $bucketAuto: {
      groupBy: "$sales.salesByCategory.totalSales",
      buckets: 3,
      output: {
        count: { $sum: 1 },
        avgCategorySales: { $avg: "$sales.salesByCategory.totalSales" },
        categories: { $addToSet: "$sales.salesByCategory.categoryName" }
      }
    }
  }
])
```

This query returns the following results:

```json
[
  {
    "_id": { "min": 100, "max": 15000 },
    "count": 20720,
    "avgCategorySales": 7234.56,
    "categories": [
      "Rum",
      "Wine Accessories",
      "Storage Boxes",
      "Children's Books"
    ]
  },
  {
    "_id": { "min": 15000, "max": 30000 },
    "count": 20715,
    "avgCategorySales": 22487.91,
    "categories": [
      "DJ Speakers",
      "Turntables",
      "Bath Accessories"
    ]
  },
  {
    "_id": { "min": 30000, "max": 49999 },
    "count": 20715,
    "avgCategorySales": 39124.33,
    "categories": [
      "Bitters",
      "Mattress Toppers",
      "Science Fiction"
    ]
  }
]
```
