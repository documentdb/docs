# Index Advisor in DocumentDB (Public Preview)
Index Advisor is a built-in performance tuning assistant for **DocumentDB**. It helps you diagnose slow queries, understand query execution behavior, and recommend optimized index strategies to improve performance.
By analyzing your query structure along with collection and index statistics, Index Advisor generates clear, data-driven recommendations—accompanied by readable explanations that describe why a specific index would help.

## Prerequisites

To use Index Advisor, you must have:

* An active **DocumentDB instance** or an [**Azure DocumentDB cluster**](https://aka.ms/tryvcore).
* The [**DocumentDB for VS Code extension**](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-documentdb) installed.
* The [**GitHub Copilot**](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot) extension installed.
* A **valid GitHub Copilot subscription**.

If no valid Copilot subscription is detected, the extension will raise the following error:
> `GitHub Copilot is not available. Please install the GitHub Copilot extension and ensure you have an active subscription.`

## Key Benefits
* **Identify performance bottlenecks** and inefficient queries.
* **Receive actionable index recommendations** prioritized by performance impact.
* **Understand why an index matters** through clear, plain-English explanations.
* **Apply index recommendations instantly** within the extension.
* **Compare before-and-after performance** automatically once the index is created.

## How It Works
1. Open a **Find**, **Aggregate**, or **Count** query in the [DocumentDB for VS Code extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-documentdb) extension.
2. Go to the **Query Insights** tab and run your query.
3. Index Advisor collects and analyzes the query execution plan and statistics—either from the connected cluster (Standard Mode) or from preloaded data (Preload Mode).
4. All literal query values (for example, emails, numbers, or text) are replaced with `<value>` placeholders before being sent for analysis.
5. A language model (GitHub Copilot) examines the **sanitized** plan and statistics to recommend optimal indexes.
6. You can **apply** a recommendation directly; the extension creates the index and reruns the query to update performance metrics.


## Using Index Advisor in VS Code (Standard Mode)

Use Standard Mode when you can connect to a live cluster—ideal for real-time tuning directly inside VS Code.

1. **Open** a query (Find/Aggregate/Count) in the DocumentDB for VS Code extension.
2. Navigate to the **Query Insights** tab.
3. **Run** your query. The panel displays key performance indicators such as Execution Time, Documents Returned, Keys Examined, and Documents Examined.
4. Review the **Query Statistics** and **Execution Plan** summaries.

:::image type="content" source="media/query-statisics.png" alt-text="Screenshot of the query-statisics.":::

5. Explore the **Optimization Opportunities** list. Each recommendation includes a human-readable explanation and a suggested index definition.

:::image type="content" source="media/optimization-opportunities.png" alt-text="Screenshot of the optimization-opportunities.":::

6. Click **Apply** to create the recommended index.
7. After index creation, Index Advisor **re-runs the analysis** and updates metrics so you can compare performance improvements.

Index creation runs asynchronously in the background. Once complete, the panel automatically refreshes with updated results.

## Supported Index Scenarios
Index Advisor currently supports recommendations for the following query and indexing scenarios:

| Scenario                                          | Description                                                                                                                             |
| ------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| **Equality / Range Query**                        | Handles simple equality or range filters (for example, `field = value` or `field > value`).                                             |
| **Compound Filter / Covered Query / Lookup Join** | Analyzes queries that involve multiple filter conditions or joins that can be optimized with compound or covered indexes for Find Queries.               |
| **Composite Index**                               | Suggests multi-field (composite) indexes to support complex Find queries                                                   |
| **Sort Only / Filter + Sort**                     | Identifies when a sort operation can be improved or covered by an index.                                                                |
| **Filter + Sort / Index Pushdown**                | Recommends index structures that allow filtering and sorting to be handled efficiently within the index layer, reducing document scans. |
| **Low-selectivity field**                         | Supported for Find queries; will suggest a hidden index.                           |
| **Existing index coverage**                       | Supported for Find queries; if an index already exists, no new index is suggested. |

If your query scenario falls outside these patterns, please **file an ICM** with the DocumentDB team. The team will be happy to assist and review your specific use case.

## Privacy and Data Handling

Index Advisor is designed to help you safely optimize queries while protecting your data privacy.

### Data Collected by the Extension

Depending on the mode, Index Advisor may access:

* **Query execution plan** – structure and performance metrics only.
* **Collection statistics** – document count, sizes, index sizes, and number of indexes.
* **Index statistics** – index names, key patterns, and usage counts.
* **Cluster metadata** – whether the cluster runs on Azure and its API type.

### Sanitization Process

Before data is sent for analysis:

* All **literal values** in filters and execution plans are replaced with `<value>`.
* Field names and query operators (for example, `email`, `$gt`, `$in`) are **preserved** for context.
* The entire execution plan is recursively sanitized, ensuring no sensitive data remains.
* Numeric performance metrics such as `nReturned` or `executionTimeMillis` are retained because they contain no customer data.

**Result:** Only query structure and performance characteristics are shared—never your actual data.

## Example of Sanitization

**Before (not sent):**

```json
{
  "filter": {
    "email": "john.doe@example.com",
    "age": { "$gt": 25 }
  }
}
```

**After (sent):**

```json
{
  "filter": {
    "email": "<value>",
    "age": { "$gt": "<value>" }
  }
}
```

The model can recognize the query pattern but cannot infer or access your real data values.


## Limitations

* **Regional availability:** Index Advisor is currently available only in the **United States** and **Canada** regions.
* **Index management:** While Index Advisor recommends new indexes, **dropping indexes is not recommended** through the extension at this time.
* **Scenario coverage:** Only the supported scenarios listed above are optimized in this release. For other query types, please file an ICM with our team.
* **Data sensitivity:** Database and collection names are treated as metadata, but organizations should still review internal data classification policies.

## Compliance and Data Protection

* The current implementation (v1.0) is designed to **minimize exposure of personal or sensitive data**.
* Only sanitized structural and statistical data are analyzed.
* No sample documents or literal query values are transmitted.
* Metadata is limited to information required for contextual understanding.
* Any new features involving unsanitized data will be reviewed through formal Microsoft privacy and compliance processes.

## Best Practices

* Follow your organization’s **data governance policies** when exporting or sharing statistics.
* Review index recommendations before applying them to ensure they align with your workload and cost requirements.
* Avoid manually dropping indexes without reviewing dependencies or consulting with the Azure DocumentDB team.
* If your query patterns aren’t supported, **file an ICM** for guidance and support.

## Conclusion

Index Advisor provides clear, actionable insights to optimize query performance in DocumentDB—while maintaining strong privacy protections through comprehensive data sanitization.
It supports the most common query and indexing patterns, helps you safely implement performance improvements, and is continuously evolving to deliver more advanced, privacy-conscious optimization capabilities.
