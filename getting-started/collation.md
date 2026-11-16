---
title: Collation
description: Configure language-aware string comparisons, queries, and ordered indexes in DocumentDB 1.2.
---

# Configure collation

Collation controls how DocumentDB compares and sorts string values. Use it to match names without regard to case, distinguish or ignore accents, apply language-specific ordering, or sort digit sequences within strings numerically.

Collation changes comparison rules, not stored data. It does not rewrite text, rename fields, or convert strings to numbers. Field names remain case-sensitive, and regular expressions use their own matching options rather than collation rules.

This guide targets **DocumentDB 1.2**. It covers operation-level collation and supported ordered indexes. The `create` command in this release does not support setting a collection's default collation.

Start with [setup](#enable-collation), then work through [string comparisons](#compare-and-sort-strings), [comparison options](#choose-comparison-options), [aggregation](#count-deduplicate-and-group-values), [writes](#update-and-delete-matching-documents), and [indexes](#use-collated-indexes). The final sections cover [troubleshooting](#troubleshooting) and [release limitations](#limitations-in-12).

## Prerequisites

- A running DocumentDB 1.2 instance with its extensions updated to `1.2-0`. See the [release notes](release-notes-1.2.md).
- PostgreSQL administrator access to configure the database used by the gateway.
- A [shell connection](mongo-shell-quickstart.md) with permission to create collections and indexes and to read and write documents.
- An unused database for the examples. The examples create collections and modify sample documents; run them in order. Expected results describe the relevant fields rather than shell-specific output formatting.

## Enable collation

Connect through `psql` as a PostgreSQL administrator. Apply the following settings to the PostgreSQL database used by the gateway. The example uses `postgres`; replace that name if your gateway connects to a different PostgreSQL database.

```sql
ALTER DATABASE postgres SET documentdb_core.enableCollation = on;
ALTER DATABASE postgres SET documentdb.enableCollationWithNonUniqueOrderedIndexes = on;
ALTER DATABASE postgres SET documentdb.defaultUseCompositeOpClass = on;
```

The first setting enables operation-level collation. The second enables collation on ordered indexes, including the unique indexes described later. The third makes the regular index specifications in this guide create ordered indexes.

Database-level settings apply to **new PostgreSQL sessions**. Restart the gateway, or restart your local container without recreating its data volume, so its connection pools pick up the settings. A `SET` in your `psql` session does not configure separate gateway sessions. The PostgreSQL database name is not the logical document database name selected in the shell.

For collated unique indexes, also enable these settings before reconnecting the gateway:

```sql
ALTER DATABASE postgres SET documentdb.enable_collated_unique_indexes = on;
ALTER DATABASE postgres SET documentdb.enable_composite_unique_optional_key = on;
ALTER DATABASE postgres SET documentdb.enableCompositeUniqueHash = on;
ALTER DATABASE postgres SET documentdb.enable_unique_null_missing_equivalence = on;
ALTER DATABASE postgres SET documentdb.enable_unique_null_missing_semantics = on;
```

Keep the required `documentdb_extended_rum` extension installed. The settings serve different purposes:

| GUC | Purpose |
| --- | --- |
| `documentdb_core.enableCollation` | Enables operation-level collation for supported commands. |
| `documentdb.enableCollationWithNonUniqueOrderedIndexes` | Enables collation on ordered indexes, including collated unique indexes despite its name. |
| `documentdb.enable_collated_unique_indexes` | Enables the collated unique-index path. |
| `documentdb.enable_composite_unique_optional_key` | Enables the optional-key support required by collated unique indexes. |
| `documentdb.defaultUseCompositeOpClass` | Creates regular indexes as ordered indexes. Collated indexes require ordered indexes. |
| `documentdb.enableCompositeUniqueHash` | Selects the composite-hash unique-index implementation used by the collated unique-index examples. Keep it enabled. |
| `documentdb.enable_unique_null_missing_equivalence` | Applies null equivalence to missing paths during unique-index rechecks. |
| `documentdb.enable_unique_null_missing_semantics` | Enables the null and missing-value conflict checks used by unique indexes. |

Two other settings are useful to recognize when diagnosing an existing installation. Keep `documentdb.fail_on_collated_index_term_truncation` enabled to reject collated index terms that cannot be represented without truncation. Do not use `documentdb.skipFailOnCollation` as a substitute for enabling support: it can suppress an unsupported-collation error without applying the requested comparison rules.

### Verify the PostgreSQL configuration

Open a **new** `psql` connection to the gateway's PostgreSQL database and inspect the active settings:

```sql
SELECT name, setting
FROM pg_settings
WHERE lower(name) IN (
  'documentdb_core.enablecollation',
  'documentdb.enablecollationwithnonuniqueorderedindexes',
  'documentdb.enable_collated_unique_indexes',
  'documentdb.enable_composite_unique_optional_key',
  'documentdb.defaultusecompositeopclass',
  'documentdb.enablecompositeuniquehash',
  'documentdb.enable_unique_null_missing_equivalence',
  'documentdb.enable_unique_null_missing_semantics',
  'documentdb.fail_on_collated_index_term_truncation',
  'documentdb.skipfailoncollation'
)
ORDER BY name;
```

The eight setup settings should report `on`; the truncation guard should report `on`, and `skipFailOnCollation` should report `off`. These values describe that PostgreSQL connection. Existing gateway sessions, or a gateway role with its own overrides, can still have different settings. If a setting is absent, check the installed extension version and that the extension is loaded in the connection. See [Configuration](../postgres-api/configuration.md#collation) for more context.

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

To request binary comparison explicitly, use `{ locale: "simple" }`:

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

The labels are ordered as `"item1"`, `"item2"`, `"item10"`. This option compares digit sequences, not full signed or decimal numeric literals, and does not convert strings to numbers. Store numeric data as numbers when you need numeric semantics.

### Membership, ranges, and regular expressions

Equality, membership, and range predicates use the operation's comparison rules:

```javascript
db.collationContacts
  .find({ name: { $in: ["ada", "GRACE"] } }, { _id: 1 })
  .collation({ locale: "en", strength: 2 })
  .sort({ _id: 1 })
  .toArray();

db.collationContacts
  .find({ name: { $gte: "ada", $lt: "grace" } }, { _id: 1 })
  .collation({ locale: "en", strength: 2 })
  .sort({ _id: 1 })
  .toArray();
```

The membership query returns IDs `1`, `2`, and `3`. The range query returns IDs `1` and `2`.

Regular expressions do **not** inherit case or accent handling from collation:

```javascript
db.collationContacts
  .find({ name: { $regex: "^ada$" } }, { _id: 1 })
  .collation({ locale: "en", strength: 2 })
  .toArray();

db.collationContacts
  .find({ name: { $regex: "^ada$", $options: "i" } }, { _id: 1 })
  .sort({ _id: 1 })
  .toArray();
```

The first query returns no documents. The explicit `i` option makes the second query match IDs `1` and `2`. Use collated equality, not a regular expression, when you need the locale's equality rules.

You can also send the collation directly in a wire command, without a cursor helper:

```javascript
db.runCommand({
  find: "collationContacts",
  filter: { name: "ada" },
  projection: { _id: 1, name: 1 },
  sort: { _id: 1 },
  batchSize: 10,
  collation: { locale: "en", strength: 2 }
});
```

The response's `cursor.firstBatch` contains IDs `1` and `2`.

## Choose comparison options

A non-simple collation document requires a `locale`. Locale identifiers are case-sensitive, and supported regional variants are limited. For example, use `en_US`, not `en-US`; do not assume that every language tag is accepted. Keyword variants such as `de@collation=phonebook` also need application-specific validation; see [limitations](#limitations-in-12).

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

### Distinguish case while ignoring accents

Combine strength `1` with `caseLevel: true` when accents should be ignored but case should still matter:

```javascript
db.collationCafe
  .find({ name: "cafe" }, { _id: 1, name: 1 })
  .collation({ locale: "en", strength: 1, caseLevel: true })
  .sort({ _id: 1 })
  .toArray();
```

IDs `1` and `3` match. Uppercase `"CAF\u00c9"` does not.

`caseFirst` controls the order of case differences; it does not make an otherwise case-insensitive comparison case-sensitive:

```javascript
db.collationContacts
  .find({}, { _id: 0, name: 1 })
  .collation({ locale: "en", strength: 3, caseFirst: "upper" })
  .sort({ name: 1 })
  .toArray();

db.collationContacts
  .find({}, { _id: 0, name: 1 })
  .collation({ locale: "en", strength: 3, caseFirst: "lower" })
  .sort({ name: 1 })
  .toArray();
```

The orders are `"ADA"`, `"Ada"`, `"Grace"` and `"Ada"`, `"ADA"`, `"Grace"`.

### Ignore spaces or punctuation

Use `alternate: "shifted"` with `maxVariable` to choose which variable characters can be ignored at lower strengths:

```javascript
db.createCollection("collationCodes");
db.collationCodes.insertMany([
  { _id: 1, code: "ab" },
  { _id: 2, code: "a-b" },
  { _id: 3, code: "a b" }
]);

db.collationCodes.countDocuments(
  { code: "ab" },
  { collation: { locale: "en", strength: 3, alternate: "shifted", maxVariable: "punct" } }
);

db.collationCodes.countDocuments(
  { code: "ab" },
  { collation: { locale: "en", strength: 3, alternate: "shifted", maxVariable: "space" } }
);

db.collationCodes.countDocuments(
  { code: "ab" },
  { collation: { locale: "en", strength: 4, alternate: "shifted", maxVariable: "punct" } }
);
```

The counts are `3`, `2`, and `1`. With `punct`, both spaces and the hyphen are shifted. With `space`, the hyphen remains significant. At strength `4`, the shifted characters distinguish these values again.

### Apply locale-specific and Unicode rules

Turkish treats the dotted and dotless forms of `I` differently from English. Normalization is a separate option for handling Unicode representations:

```javascript
db.createCollection("collationUnicode");
db.collationUnicode.insertMany([
  { _id: 1, value: "caf\u00e9" },
  { _id: 2, value: "cafe\u0301" },
  { _id: 3, value: "I" },
  { _id: 4, value: "\u0131" },
  { _id: 5, value: "i" },
  { _id: 6, value: "\u0130" }
]);

db.collationUnicode
  .find({ value: "I" }, { _id: 1 })
  .collation({ locale: "tr", strength: 2 })
  .sort({ _id: 1 })
  .toArray();

db.collationUnicode
  .find({ value: "caf\u00e9" }, { _id: 1 })
  .collation({ locale: "en", strength: 3, normalization: true })
  .sort({ _id: 1 })
  .toArray();
```

The Turkish query matches IDs `3` and `4`. The normalization example matches IDs `1` and `2`, which store precomposed and decomposed accented text. Enabling normalization does not rewrite either stored value, and some canonically equivalent strings can compare equal without enabling it.

For reverse secondary accent ordering, set `backwards: true`:

```javascript
db.createCollection("collationFrench");
db.collationFrench.insertMany([
  { _id: 1, word: "cot\u00e9" },
  { _id: 2, word: "c\u00f4te" }
]);

db.collationFrench
  .find({}, { _id: 0, word: 1 })
  .collation({ locale: "fr", strength: 2, backwards: true })
  .sort({ word: 1 })
  .toArray();
```

The order is `"c\u00f4te"`, then `"cot\u00e9"`. Choose the locale and options that reflect your users' language rather than assuming one global case-folding rule.

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

String-valued `$min` and `$max` also use the operation's comparison rules:

```javascript
db.collationContacts.aggregate(
  [
    { $group: { _id: null, firstLabel: { $min: "$label" }, lastLabel: { $max: "$label" } } }
  ],
  { collation: { locale: "en", numericOrdering: true } }
).toArray();
```

The result has `firstLabel: "item1"` and `lastLabel: "item10"`.

Do not depend on the representative spelling of a grouped or distinct equivalence class, or on distinct result order. If a sort considers multiple strings equal, add a unique, collation-insensitive tie-breaker, such as a numeric `_id`, when you need deterministic ordering.

A pipeline uses one effective collation for its supported stages and nested pipelines, not a separate collation per field or stage. `$group` is supported, but not every accumulator supports non-simple collation; see [limitations](#limitations-in-12).

### Use comparison expressions and arrays

The pipeline's collation also reaches supported comparison expressions:

```javascript
db.collationContacts.aggregate(
  [
    { $match: { $expr: { $eq: ["$name", "ada"] } } },
    { $project: { _id: 1, matches: { $eq: ["$name", "ADA"] } } },
    { $sort: { _id: 1 } }
  ],
  { collation: { locale: "en", strength: 2 } }
).toArray();

db.createCollection("collationTags");
db.collationTags.insertOne({
  _id: 1,
  tags: ["Red", "RED", "blue"],
  labels: ["item10", "item2", "item1"]
});

db.collationTags
  .find({ tags: { $all: ["red", "BLUE"] } }, { _id: 1 })
  .collation({ locale: "en", strength: 2 })
  .toArray();

db.collationTags.aggregate(
  [
    {
      $project: {
        _id: 0,
        redTags: { $filter: { input: "$tags", as: "tag", cond: { $eq: ["$$tag", "red"] } } },
        uniqueTagCount: { $size: { $setUnion: ["$tags", []] } },
        sortedLabels: { $sortArray: { input: "$labels", sortBy: 1 } }
      }
    }
  ],
  { collation: { locale: "en", strength: 2, numericOrdering: true } }
).toArray();
```

The first pipeline returns IDs `1` and `2`, both with `matches: true`. The array query matches ID `1`. The final pipeline returns `redTags: ["Red", "RED"]`, `uniqueTagCount: 2`, and `sortedLabels: ["item1", "item2", "item10"]`. `$setUnion` here is an expression, not the unsupported `$group` accumulator `$addToSet`.

### Join collections with `$lookup`

Pass collation to `aggregate`, not inside `$lookup`:

```javascript
db.createCollection("collationOrders");
db.collationOrders.insertOne({ _id: 101, contactName: "ada" });

db.collationOrders.aggregate(
  [
    {
      $lookup: {
        from: "collationContacts",
        localField: "contactName",
        foreignField: "name",
        as: "contacts"
      }
    },
    { $project: { _id: 1, contactCount: { $size: "$contacts" } } }
  ],
  { collation: { locale: "en", strength: 2 } }
).toArray();
```

Order `101` has `contactCount: 2` because both `"Ada"` and `"ADA"` match. There is still only one operation-level collation for the pipeline. In 1.2, collated join conditions are evaluated without index pushdown, even if a matching collated index exists.

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

`findAndModify` also supports operation-level collation in 1.2. In raw batched `update` and `delete` commands, put `collation` inside each individual update or delete entry. In this release, a top-level `collation` field on these two commands is ignored without an error, so it does not apply the requested comparison rules. An `insert` command does not accept an operation-level collation override.

### Return a modified document

Use a numeric sort key to choose deterministically when several strings compare equal:

```javascript
db.createCollection("collationJobs");
db.collationJobs.insertMany([
  { _id: 1, owner: "Ada", state: "pending" },
  { _id: 2, owner: "ADA", state: "pending" }
]);

db.collationJobs.findAndModify({
  query: { owner: "ada", state: "pending" },
  sort: { _id: 1 },
  update: { $set: { state: "running" } },
  new: true,
  collation: { locale: "en", strength: 2 }
});
```

The returned document has `_id: 1` and `state: "running"`. Document `2` remains pending.

### Place collation in batched commands

Each update or delete entry carries its own collation:

```javascript
db.runCommand({
  update: "collationJobs",
  updates: [
    {
      q: { owner: "ada" },
      u: { $set: { state: "done" } },
      multi: true,
      collation: { locale: "en", strength: 2 }
    }
  ]
});

db.runCommand({
  delete: "collationJobs",
  deletes: [
    {
      q: { owner: "ADA", state: "done" },
      limit: 0,
      collation: { locale: "en", strength: 2 }
    }
  ]
});
```

The update matches both jobs; the delete removes both. A non-upserting update still leaves a field's original spelling unchanged unless the update explicitly changes that field.

### Upsert without normalizing stored values

Collation determines whether an existing document matches. It does not normalize strings when an upsert inserts a new document:

```javascript
db.createCollection("collationUpsertContacts");

db.collationUpsertContacts.updateOne(
  { name: "Ada" },
  { $set: { active: true } },
  { upsert: true, collation: { locale: "en", strength: 2 } }
);

db.collationUpsertContacts.updateOne(
  { name: "ADA" },
  { $set: { active: false } },
  { upsert: true, collation: { locale: "en", strength: 2 } }
);

db.collationUpsertContacts.find({}, { _id: 0, name: 1, active: 1 }).toArray();
```

The first operation inserts a document using the filter's spelling, `"Ada"`. The second matches that document instead of inserting another. The collection contains one document with `name: "Ada"` and `active: false`.

## Use collated indexes

Create an ordered index with the same comparison rules as your queries:

```javascript
db.collationContacts.createIndex(
  { name: 1 },
  { name: "name_en_ci", collation: { locale: "en", strength: 2 } }
);
```

For a query to use a collated index for string comparisons, specify the same locale identifier and comparison options used by the index. Do not rely on two differently named locales producing similar ordering. An index does not change the collection's comparison rules, so continue to specify the collation on each operation.

### Inspect an index and its query plan

```javascript
db.collationContacts.getIndexes();

db.collationContacts
  .find({ name: "ada" }, { _id: 1 })
  .collation({ locale: "en", strength: 2 })
  .hint("name_en_ci")
  .sort({ _id: 1 })
  .toArray();

db.collationContacts
  .find({ name: "ada" })
  .collation({ locale: "en", strength: 2 })
  .explain("queryPlanner");
```

The hinted query returns IDs `1` and `2`. Use `getIndexes()` to inspect the index's collation and `explain()` to inspect the selected plan. Explain output is engine-specific; do not expect PostgreSQL-backed plans to have the same shape as another server's output.

The planner can choose a collection scan for a small dataset. A hint is useful for this matching-index check, not a reason to force an index in every application query. Changing strength, locale, or other comparison options can make an index ineligible for string comparisons. A binary index is not a replacement for a matching collated index.

### Use compound indexes and multiple collations

An index has one collation for its string keys. For example:

```javascript
db.collationContacts.createIndex(
  { name: 1, _id: 1 },
  { name: "name_id_en_ci", collation: { locale: "en", strength: 2 } }
);

db.collationContacts
  .find({ name: "ada" }, { _id: 1 })
  .collation({ locale: "en", strength: 2 })
  .hint("name_id_en_ci")
  .sort({ _id: 1 })
  .toArray();

db.collationContacts.createIndex(
  { name: 1 },
  { name: "name_en_cs", collation: { locale: "en", strength: 3 } }
);
```

The compound-index query returns IDs `1` and `2`. The last command adds a separate case-sensitive index. Give indexes distinct names when using the same key pattern with different collations, and budget for their storage and write-maintenance cost. Collation does not turn the numeric `_id` into a string.

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

For a non-sparse, single-field unique index, missing and `null` values also participate in uniqueness:

```javascript
db.createCollection("collationOptionalContacts");
db.collationOptionalContacts.createIndex(
  { name: 1 },
  {
    name: "unique_optional_name_en_ci",
    unique: true,
    collation: { locale: "en", strength: 2 }
  }
);
db.collationOptionalContacts.insertOne({ _id: 1 });
```

The following insert fails with a duplicate-key error because the existing missing field and explicit `null` conflict:

```javascript
db.collationOptionalContacts.insertOne({ _id: 2, name: null });
```

This example uses the null and missing-value settings from [setup](#enable-collation). An empty string is not a missing value. Do not generalize the single-field example to sparse indexes or compound-key combinations.

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| A command reports that collation is not supported | Check the extension version and `documentdb_core.enableCollation` in new gateway PostgreSQL sessions, then check whether the command or pipeline stage is supported. |
| `createIndex.collation is not implemented yet` | Enable `documentdb.enableCollationWithNonUniqueOrderedIndexes` and reconnect the gateway. |
| `Collation is only supported with ordered (or composite) indexes` | Enable `documentdb.defaultUseCompositeOpClass` and reconnect the gateway, or set `enableOrderedIndex: true` on a supported index specification. |
| `Collation is not yet supported for unique indexes` | Enable `documentdb.enable_collated_unique_indexes` and reconnect the gateway. |
| `collated unique indexes are not supported` | Check `documentdb.enable_composite_unique_optional_key`, the installed extension version, and `documentdb_extended_rum`. Follow the complete unique-index setup, not just the non-unique-named setting. |
| Enabling a setting in `psql` has no effect on shell queries | Check that you configured the correct PostgreSQL database, reconnected gateway pools, and accounted for role/session overrides. |
| A raw update or delete succeeds but matches only exact spellings | Move `collation` from the top level into each `updates` or `deletes` entry. A top-level field is ignored in 1.2. |
| Case-insensitive equality works but a regular expression does not | Regex matching uses its own options, not the operation's collation. |
| An index is not selected | Compare the query and index collation, inspect `getIndexes()` and `explain()`, and consider dataset size and other predicates. |
| A unique-index build or write fails with a duplicate key | Check for values that become equal under the index's case, accent, punctuation, numeric, and null/missing rules. Do not assume binary-distinct strings remain unique. |
| An error says an index term requires truncation with a non-simple collation | Review the value and index design rather than disabling `documentdb.fail_on_collated_index_term_truncation`. |
| An error says `collation is not supported with the $addToSet accumulator in $group yet.` | This accumulator is not collation-aware in 1.2. Use a supported pipeline, or remove collation only if binary comparison is acceptable. |
| An error says `Field 'locale' is invalid` | Check the case and form of the locale identifier, for example `en_US`, not `en-US` or `EN`. |
| The shell reports that `.collation` is not a function | Use a full, current `mongosh` client with the cursor helper, or send an explicit `collation` field with a supported wire command. |

When checking a locale, test representative values from your application, including ties and accents. Do not use `skipFailOnCollation` to make errors disappear: success without the intended comparison rules is not a valid fix.

## Limitations in 1.2

| Area | Limitation |
| --- | --- |
| Collection defaults and views | The `create` command rejects a `collation` option. Do not use `documentdb.skipFailOnCollation` to enable it: that setting bypasses the rejection without implementing a collection default. |
| Locale variants | In DocumentDB 1.2.0, `find` with `{ locale: "de@collation=phonebook", strength: 1 }` does not match `"M\u00fcller"` when filtering for `"Mueller"`. This differs from native MongoDB 7.0 and 8.0.32. Validate locale variants against your application's language requirements. |
| Index types | Non-simple collation requires supported ordered indexes. Text, hashed, geospatial, and vector indexes do not support it. |
| Index pushdown | Collated `$expr` predicates and `$lookup` join conditions are not pushed down to indexes in 1.2. Ordinary collated-index support does not imply index support for these predicates. |
| Large indexed values | Writes and indexed reads reject collated values whose index terms would require truncation. |
| Aggregation stages | Non-simple collation is not supported with `$bucket`, `$bucketAuto`, `$sortByCount`, `$setWindowFields`, `$geoNear`, `$fill`, `$densify`, or `$merge`. |
| `$group` accumulators | Supported examples include `$sum`, `$avg`, `$min`, `$max`, and `$count`. In 1.2, unsupported cases include `$addToSet`, `$push`, `$mergeObjects`, `N`-valued accumulators, and `$top`/`$bottom`. Some sorted `$first`/`$last` pipelines are also unsupported. |

## Remove the sample collections

Run this only in the sample database after finishing the examples. It deletes these collections and their indexes, not other collections:

```javascript
for (const collectionName of [
  "collationContacts",
  "collationCafe",
  "collationCodes",
  "collationUnicode",
  "collationFrench",
  "collationTags",
  "collationOrders",
  "collationWriteContacts",
  "collationJobs",
  "collationUpsertContacts",
  "collationUniqueContacts",
  "collationOptionalContacts"
]) {
  db.getCollection(collectionName).drop();
}
```

## Related content

- [Configuration](../postgres-api/configuration.md#collation)
- [find](../api-reference/commands/query-and-write/find.md)
- [createIndexes](../api-reference/commands/indexing/createIndexes.md)
- [DocumentDB 1.2 release notes](release-notes-1.2.md)
