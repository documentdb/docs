---
title: Collation
description: Configure language-aware string comparisons, queries, and ordered indexes in DocumentDB 1.2.
---

# Configure collation

Collation controls how DocumentDB compares and sorts string values. Use it to match names without regard to case, distinguish or ignore accents, apply language-specific ordering, or sort digit sequences within strings numerically.

Collation changes comparison rules, not stored data. It does not rewrite text, rename fields, or convert strings to numbers. Field names remain case-sensitive, and regular expressions use their own matching options rather than collation rules.

This guide targets **DocumentDB 1.2**, where collation is off by default. It covers operation-level collation and supported ordered indexes. The `create` command in this release does not support setting a collection's default collation.

## Prerequisites

- A running DocumentDB 1.2 instance with its extensions updated to `1.2-0`. See the [release notes](release-notes-1.2.md).
- PostgreSQL administrator access to configure the database used by the gateway.
- A [shell connection](mongo-shell-quickstart.md) with permission to create collections and indexes and to read and write documents.
- An unused database for the examples. The examples create collections and modify sample documents; run them in order.

## Enable collation

Connect through `psql` as a PostgreSQL administrator. Apply the following settings to the PostgreSQL database used by the gateway. The example uses `postgres`; replace that name if your gateway connects to a different PostgreSQL database.

```sql
ALTER DATABASE postgres SET documentdb_core.enableCollation = on;
ALTER DATABASE postgres SET documentdb.enableCollationWithNonUniqueOrderedIndexes = on;
```

The first setting enables operation-level collation. The second enables collation on ordered indexes, including the unique indexes described later.

Database defaults apply to **new PostgreSQL sessions**. Restart the gateway, or restart your local container without recreating its data volume, so its connection pools pick up the settings. A `SET` in your `psql` session does not configure separate gateway sessions. The PostgreSQL database name is not the logical document database name selected in the shell.

For collated unique indexes, also enable these settings before reconnecting the gateway:

```sql
ALTER DATABASE postgres SET documentdb.enable_collated_unique_indexes = on;
ALTER DATABASE postgres SET documentdb.enable_composite_unique_optional_key = on;
```

Keep the default ordered-index configuration and required `documentdb_extended_rum` extension. Leave `documentdb.enable_unique_null_missing_equivalence` and `documentdb.enable_unique_null_missing_semantics` enabled; both default to `on` in 1.2 and govern unique-index null and missing-value behavior.

See [Configuration](../postgres-api/configuration.md#collation) for the settings and their defaults.

## Compare and sort strings

Create a collection without a default collation:

```javascript
db.createCollection("collationContacts");

db.collationContacts.insertMany([
  { _id: 1, name: "Ada", label: "item2" },
  { _id: 2, name: "ADA", label: "item10" },
  { _id: 3, name: "Grace", label: "item1" }
]);
```

Use English comparison rules at strength `2` to ignore case while retaining accent distinctions:

```javascript
db.collationContacts
  .find({ name: "ada" }, { _id: 1, name: 1 })
  .collation({ locale: "en", strength: 2 })
  .sort({ _id: 1 })
  .toArray();
```

The result contains documents `1` and `2`. Their stored names remain `"Ada"` and `"ADA"`. Without the collation, the lowercase filter matches neither name in this collection.

To request binary comparison explicitly, use `{ locale: "simple" }` without additional comparison options:

```javascript
db.collationContacts
  .find({ name: "Ada" }, { _id: 1, name: 1 })
  .collation({ locale: "simple" })
  .toArray();
```

Only document `1` matches. For natural ordering of digit sequences within strings, use `numericOrdering`:

```javascript
db.collationContacts
  .find({}, { _id: 0, label: 1 })
  .collation({ locale: "en", numericOrdering: true })
  .sort({ label: 1 })
  .toArray();
```

The labels are ordered as `"item1"`, `"item2"`, `"item10"`. This option does not convert strings to numeric values.

### Choose comparison options

A non-simple collation document requires a `locale`. Locale identifiers are case-sensitive, and supported regional variants are limited. For example, use `en_US`, not `en-US`; do not assume that every language tag is accepted.

| Option | Accepted values | Default for `en` | Purpose |
| --- | --- | --- | --- |
| `locale` | Supported locale identifier, such as `en`, `fr`, `de`, or `tr`, or `simple`. | Required. | Selects language-specific rules or binary comparison. |
| `strength` | Integer from `1` to `5`. | `3`. | Selects the comparison level. |
| `caseLevel` | Boolean. | `false`. | Adds case distinctions, including at strengths `1` and `2`. |
| `caseFirst` | `off`, `upper`, or `lower`. | `off`. | Controls case ordering when case is distinguished. |
| `numericOrdering` | Boolean. | `false`. | Compares digit sequences in strings numerically. |
| `alternate` | `non-ignorable` or `shifted`. | `non-ignorable`. | Controls whether spaces and punctuation contribute at lower comparison levels. |
| `maxVariable` | `punct` or `space`. | `punct`. | Selects characters shifted when `alternate` is `shifted`. |
| `backwards` | Boolean. | `false`. | Enables reverse secondary weighting for accent ordering. |
| `normalization` | Boolean. | `false`. | Enables Unicode normalization checks during comparison. |

For English with the other default options, strength `1` ignores case and accents, strength `2` distinguishes accents but ignores case, and strength `3` distinguishes both. Other locales can tailor ordering, especially case and accent ordering. Strengths `4` and `5` add quaternary and identical comparison levels; they are not substitutes for `simple`.

For example, vary the strength while keeping the filter unchanged:

```javascript
db.createCollection("collationCafe");

db.collationCafe.insertMany([
  { _id: 1, name: "cafe" },
  { _id: 2, name: "CAF\u00c9" },
  { _id: 3, name: "caf\u00e9" }
]);

db.collationCafe.countDocuments(
  { name: "caf\u00e9" },
  { collation: { locale: "en", strength: 1 } }
);

db.collationCafe.countDocuments(
  { name: "caf\u00e9" },
  { collation: { locale: "en", strength: 2 } }
);

db.collationCafe.countDocuments(
  { name: "caf\u00e9" },
  { collation: { locale: "en", strength: 3 } }
);
```

The counts are `3`, `2`, and `1`, respectively. The Unicode escapes represent uppercase and lowercase accented `e`.

## Count, deduplicate, and group values

Supported `find`, `count`, `distinct`, and `aggregate` operations use their supplied collation for supported string comparisons.

```javascript
db.collationContacts.countDocuments(
  { name: "ada" },
  { collation: { locale: "en", strength: 2 } }
);

db.collationContacts.distinct(
  "name",
  {},
  { collation: { locale: "en", strength: 2 } }
);

db.collationContacts.aggregate(
  [
    { $group: { _id: "$name", contacts: { $sum: 1 } } },
    { $sort: { contacts: -1 } }
  ],
  { collation: { locale: "en", strength: 2 } }
).toArray();
```

The count is `2`. The distinct result contains `"Grace"` and one representative of the `"Ada"`/`"ADA"` equivalence class. The aggregation returns two groups, with counts of `2` and `1`.

Do not depend on the representative spelling of a grouped or distinct equivalence class, or on distinct result order. If a sort considers multiple strings equal, add a unique, collation-insensitive tie-breaker, such as a numeric `_id`, when you need deterministic ordering.

A pipeline uses one effective collation for its supported stages and nested pipelines, not a separate collation per field or stage. `$group` is supported, but not every accumulator supports non-simple collation; see [limitations](#limitations-in-12).

## Update and delete matching documents

Use a separate collection so the writes do not change the earlier query results:

```javascript
db.createCollection("collationWriteContacts");

db.collationWriteContacts.insertMany([
  { _id: 1, name: "Ada", status: "pending" },
  { _id: 2, name: "ADA", status: "pending" },
  { _id: 3, name: "Grace", status: "pending" }
]);

db.collationWriteContacts.updateMany(
  { name: "ada" },
  { $set: { status: "reviewed" } },
  { collation: { locale: "en", strength: 2 } }
);

db.collationWriteContacts.deleteMany(
  { name: "ada" },
  { collation: { locale: "en", strength: 2 } }
);
```

The update matches and modifies two documents. The delete removes those two documents, leaving Grace.

`findAndModify` also supports operation-level collation in 1.2. In raw batched `update` and `delete` commands, put `collation` inside each individual update or delete entry, not at the top level. An `insert` command does not accept an operation-level collation override.

## Use collated indexes

Create an ordered index with the same comparison rules as your queries:

```javascript
db.collationContacts.createIndex(
  { name: 1 },
  { name: "name_en_ci", collation: { locale: "en", strength: 2 } }
);
```

A query's collation must match the index's comparison rules for that index to support string comparisons. An index does not change the collection's comparison rules, so continue to specify the collation on each operation.

### Enforce case-insensitive uniqueness

With the additional unique-index settings [enabled](#enable-collation), create a unique ordered index:

```javascript
db.createCollection("collationUniqueContacts");

db.collationUniqueContacts.createIndex(
  { name: 1 },
  {
    name: "unique_name_en_ci",
    unique: true,
    collation: { locale: "en", strength: 2 }
  }
);

db.collationUniqueContacts.insertOne({ _id: 1, name: "Ada" });
```

The following insert fails with a duplicate-key error because `"ADA"` equals the existing `"Ada"` under the index's collation:

```javascript
db.collationUniqueContacts.insertOne({ _id: 2, name: "ADA" });
```

The index enforces its own comparison rules on every write. A different collation on a query or update does not change those uniqueness rules. Existing data must satisfy the constraint before a unique index can be built.

## Limitations in 1.2

| Area | Limitation |
| --- | --- |
| Collection defaults and views | The `create` command rejects a `collation` option. Do not use `documentdb.skipFailOnCollation` to enable it: that setting bypasses the rejection without implementing a collection default. |
| Index types | Non-simple collation requires supported ordered indexes. Text, hashed, geospatial, and vector indexes do not support it. |
| Large indexed values | Writes and indexed reads reject collated values whose index terms would require truncation. |
| Aggregation stages | Non-simple collation is not supported with `$bucket`, `$bucketAuto`, `$sortByCount`, `$setWindowFields`, `$geoNear`, `$fill`, `$densify`, or `$merge`. |
| `$group` accumulators | Supported examples include `$sum`, `$avg`, `$min`, `$max`, and `$count`. In 1.2, unsupported cases include `$addToSet`, `$push`, `$mergeObjects`, `N`-valued accumulators, and `$top`/`$bottom`. Some sorted `$first`/`$last` pipelines are also unsupported. |

## Related content

- [Configuration](../postgres-api/configuration.md#collation)
- [find](../api-reference/commands/query-and-write/find.md)
- [createIndexes](../api-reference/commands/indexing/createIndexes.md)
- [DocumentDB 1.2 release notes](release-notes-1.2.md)
