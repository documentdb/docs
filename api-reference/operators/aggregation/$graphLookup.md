---
title: $graphLookup
description: The $graphLookup stage in the aggregation pipeline performs a recursive search on a collection to traverse graph or hierarchical data.
type: operators
category: aggregation
---

# $graphLookup

The `$graphLookup` stage in the aggregation pipeline performs a recursive search on a collection, following relationships between documents to traverse graph or hierarchical data structures. It is useful for finding connected documents across multiple levels, such as organizational hierarchies, category trees, or social networks.

## Syntax

```javascript
{
  $graphLookup: {
    from: <collection>,
    startWith: <expression>,
    connectFromField: <string>,
    connectToField: <string>,
    as: <string>,
    maxDepth: <number>,
    depthField: <string>,
    restrictSearchWithMatch: <document>
  }
}
```

## Parameters

| Parameter | Description |
| --- | --- |
| **`from`** | Required. The target collection to perform the recursive search on. |
| **`startWith`** | Required. An expression that specifies the value to start the recursive search from. |
| **`connectFromField`** | Required. The field in each document that contains the value to recursively match against `connectToField`. |
| **`connectToField`** | Required. The field in the `from` collection documents to match against `connectFromField`. |
| **`as`** | Required. The name of the array field to add to the input documents, containing all documents found during the recursive search. |
| **`maxDepth`** | Optional. A non-negative integer specifying the maximum recursion depth. A value of `0` means only the direct connections are returned. |
| **`depthField`** | Optional. The name of a field to add to each output document indicating the recursion depth at which the document was found. |
| **`restrictSearchWithMatch`** | Optional. A filter expression that limits the documents considered during the recursive search. |

## Limitations

- The `from` collection cannot be a sharded collection.

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

Let's also say we have a `categories` collection representing a hierarchy of product categories:

```json
{ "_id": "Beverages", "parentCategory": null }
{ "_id": "Alcoholic Beverages", "parentCategory": "Beverages" }
{ "_id": "Wine", "parentCategory": "Alcoholic Beverages" }
{ "_id": "Organic Wine", "parentCategory": "Wine" }
{ "_id": "White Wine", "parentCategory": "Wine" }
{ "_id": "Sparkling Wine", "parentCategory": "Wine" }
{ "_id": "Spirits", "parentCategory": "Alcoholic Beverages" }
{ "_id": "Whiskey", "parentCategory": "Spirits" }
{ "_id": "Vodka", "parentCategory": "Spirits" }
{ "_id": "Rum", "parentCategory": "Spirits" }
{ "_id": "Brandy", "parentCategory": "Spirits" }
{ "_id": "Bitters", "parentCategory": "Spirits" }
```

### Example 1: Traversing a category hierarchy

To find all ancestor categories for "Organic Wine" by traversing the hierarchy upward:

```javascript
db.categories.aggregate([
  {
    $match: { _id: "Organic Wine" }
  },
  {
    $graphLookup: {
      from: "categories",
      startWith: "$parentCategory",
      connectFromField: "parentCategory",
      connectToField: "_id",
      as: "ancestors",
      depthField: "depth"
    }
  }
])
```

This query returns the following result:

```json
[
  {
    "_id": "Organic Wine",
    "parentCategory": "Wine",
    "ancestors": [
      {
        "_id": "Wine",
        "parentCategory": "Alcoholic Beverages",
        "depth": 0
      },
      {
        "_id": "Alcoholic Beverages",
        "parentCategory": "Beverages",
        "depth": 1
      },
      {
        "_id": "Beverages",
        "parentCategory": null,
        "depth": 2
      }
    ]
  }
]
```

### Example 2: Finding all sub-categories with a depth limit

To find all sub-categories under "Alcoholic Beverages" up to 1 level deep:

```javascript
db.categories.aggregate([
  {
    $match: { _id: "Alcoholic Beverages" }
  },
  {
    $graphLookup: {
      from: "categories",
      startWith: "$_id",
      connectFromField: "_id",
      connectToField: "parentCategory",
      as: "subCategories",
      maxDepth: 1,
      depthField: "level"
    }
  },
  {
    $project: {
      _id: 1,
      "subCategories._id": 1,
      "subCategories.level": 1
    }
  }
])
```

This query returns the following result:

```json
[
  {
    "_id": "Alcoholic Beverages",
    "subCategories": [
      { "_id": "Wine", "level": 0 },
      { "_id": "Spirits", "level": 0 },
      { "_id": "Organic Wine", "level": 1 },
      { "_id": "White Wine", "level": 1 },
      { "_id": "Sparkling Wine", "level": 1 },
      { "_id": "Whiskey", "level": 1 },
      { "_id": "Vodka", "level": 1 },
      { "_id": "Rum", "level": 1 },
      { "_id": "Brandy", "level": 1 },
      { "_id": "Bitters", "level": 1 }
    ]
  }
]
```

### Example 3: Recursive lookup with a filter

To find all ancestor categories for "Whiskey" but only include categories that have a non-null parent (excluding the root):

```javascript
db.categories.aggregate([
  {
    $match: { _id: "Whiskey" }
  },
  {
    $graphLookup: {
      from: "categories",
      startWith: "$parentCategory",
      connectFromField: "parentCategory",
      connectToField: "_id",
      as: "ancestors",
      depthField: "depth",
      restrictSearchWithMatch: {
        parentCategory: { $ne: null }
      }
    }
  }
])
```

This query returns the following result:

```json
[
  {
    "_id": "Whiskey",
    "parentCategory": "Spirits",
    "ancestors": [
      {
        "_id": "Spirits",
        "parentCategory": "Alcoholic Beverages",
        "depth": 0
      },
      {
        "_id": "Alcoholic Beverages",
        "parentCategory": "Beverages",
        "depth": 1
      }
    ]
  }
]
```
