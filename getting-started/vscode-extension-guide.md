# DocumentDB for VS Code Extension

The [DocumentDB for VS Code extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-documentdb) is a powerful, open-source GUI that helps you browse, manage, and query DocumentDB and MongoDB databases across any cloud, hybrid, or local environment.

## Overview

DocumentDB for VS Code provides a developer-centric experience with minimal setup, offering universal support for both DocumentDB and MongoDB databases. Whether you're working with cloud-based, hybrid cloud, on-premises, or local instances, this extension provides the tools you need for efficient database management.

## Key Features

### Universal DocumentDB and MongoDB Support

- **Flexible Connections**: Use connection strings or browse your cloud providers
- **Cross-Platform Service Discovery**: Connect to DocumentDB and MongoDB instances hosted with your provider
- **Wide Compatibility**: Full support for all DocumentDB and MongoDB databases

### Developer-Centric Experience

- **Multiple Data Views**: Inspect collections using Table, Tree, or JSON layouts with built-in pagination
- **Query Editing**: Execute find queries with syntax highlighting, auto-completion, and field name suggestions
- **Document Management**: Create, edit, and delete documents directly from VS Code
- **Data Import/Export**: Quickly import JSON files or export documents, query results, or collections

## Installation

### Prerequisites

- [Visual Studio Code](https://code.visualstudio.com/) installed
- [Docker Desktop](https://www.docker.com/products/docker-desktop) (for running local DocumentDB instances)
- Basic familiarity with document databases
- MongoDB Shell (optional, for advanced commands)

### Installation Steps

1. **Open VS Code**
2. **Navigate to Extensions** (Ctrl+Shift+X or Cmd+Shift+X)
3. **Search for "DocumentDB for VS Code"**
4. **Click Install**
5. **Reload VS Code** if prompted

### Alternative Installation

Use the command palette:
1. Open VS Code Quick Open (Ctrl+P)
2. Paste: `ext install ms-azuretools.vscode-documentdb`
3. Press Enter

## Getting Started

1. **Start a local DocumentDB instance using Docker:**

   ```bash
   docker pull ghcr.io/documentdb/documentdb/documentdb-local:latest
   docker tag ghcr.io/documentdb/documentdb/documentdb-local:latest documentdb
   docker run -dt -p 10260:10260 --name documentdb-container documentdb --username admin --password password123
   docker image rm -f ghcr.io/documentdb/documentdb/documentdb-local:latest
   ```

   > **Note:** We're using port `10260` to avoid conflicts with other local database services. You can use port `27017` (the standard MongoDB port) if you prefer.

2. **Connect to DocumentDB using the VS Code extension:**
   - Open VS Code and click the DocumentDB icon in the sidebar
   - Click "Add New Connection"
   - Select "Connection String" and paste:
     ```
     mongodb://admin:password123@localhost:10260/?tls=true&tlsAllowInvalidCertificates=true&authMechanism=SCRAM-SHA-256
     ```
   - Click "Enter" when prompted for your username and password

## Core Features

### Database and Collection Management

#### Browsing Structure
- **Database View**: See all databases in your connection
- **Collection View**: Browse collections within each database
- **Document View**: Explore individual documents

#### Creating Resources
```javascript
// Create a new database
// Right-click in the database explorer and select "Create Database"

// Create a new collection
// Right-click on a database and select "Create Collection"

// Create a new document
// Right-click on a collection and select "Create Document"
```

### Data Views

#### Table View
- **Grid Layout**: View documents in a spreadsheet-like format
- **Sortable Columns**: Click column headers to sort data
- **Filtering**: Use the filter bar to search for specific values
- **Pagination**: Navigate through large datasets

#### Tree View
- **Hierarchical Display**: See document structure as a tree
- **Expandable Nodes**: Click to expand/collapse nested objects
- **Field Navigation**: Easily navigate complex document structures

#### JSON View
- **Raw JSON**: View documents in their native JSON format
- **Syntax Highlighting**: Color-coded JSON for better readability
- **Formatting**: Automatically formatted JSON display

### Query Editor

#### Basic Queries
```javascript
// Find all documents
db.collection.find({})

// Find documents with filters
db.collection.find({ status: "active" })

// Find documents with complex filters
db.collection.find({
  age: { $gte: 18 },
  status: { $in: ["active", "pending"] }
})
```

#### Aggregation Pipelines
```javascript
// Basic aggregation
db.collection.aggregate([
  { $match: { status: "active" } },
  { $group: { _id: "$category", count: { $sum: 1 } } },
  { $sort: { count: -1 } }
])

// Complex aggregation with multiple stages
db.sales.aggregate([
  { $match: { date: { $gte: new Date("2024-01-01") } } },
  { $lookup: { from: "products", localField: "productId", foreignField: "_id", as: "product" } },
  { $unwind: "$product" },
  { $group: { _id: "$product.category", totalSales: { $sum: "$amount" } } }
])
```

#### Query Features
- **Auto-completion**: Field names and operators are suggested as you type
- **Syntax Highlighting**: MongoDB query syntax is color-coded
- **Error Detection**: Invalid queries are highlighted
- **Query History**: Previous queries are saved for reuse

### Document Management

#### Creating Documents
```javascript
// Create a new document
{
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30,
  "created_at": new Date(2024-11-16),
  "tags": ["user", "active"]
}
```

#### Editing Documents
- **Inline Editing**: Click on values to edit them directly
- **JSON Editor**: Use the JSON view for complex edits
- **Validation**: Automatic validation of JSON syntax
- **Undo/Redo**: Support for editing operations

#### Deleting Documents
- **Single Document**: Right-click and select "Delete Document"
- **Bulk Operations**: Select multiple documents for deletion
- **Confirmation**: Confirmation dialog to prevent accidental deletions

### Data Import/Export

#### Importing Data
1. **JSON Files**: Import documents from JSON files
2. **CSV Files**: Import tabular data (with field mapping)
3. **Bulk Import**: Import large datasets efficiently

#### Exporting Data
1. **Single Documents**: Export individual documents
2. **Query Results**: Export filtered query results
3. **Collections**: Export entire collections
4. **Formats**: Export as JSON, CSV, or BSON

## Advanced Features

### MongoDB Scrapbooks

#### Creating Scrapbooks
```javascript
// Create a new scrapbook file (.mongo)
// This allows you to save and reuse queries

// Example scrapbook content
db.users.find({ status: "active" }).limit(10)

// You can include multiple queries
db.users.countDocuments({ status: "active" })

db.users.aggregate([
  { $match: { status: "active" } },
  { $group: { _id: "$department", count: { $sum: 1 } } }
])
```

#### Running Scrapbooks
- **Execute All**: Run all queries in the scrapbook
- **Execute Selection**: Run only selected queries
- **Step-by-Step**: Execute queries one at a time

### Index Management

#### Viewing Indexes
```javascript
// View all indexes on a collection
db.collection.getIndexes()
```

#### Creating Indexes
```javascript
// Create a single field index
db.collection.createIndex({ "email": 1 })

// Create a compound index
db.collection.createIndex({ "lastName": 1, "firstName": 1 })

// Create a unique index
db.collection.createIndex({ "email": 1 }, { unique: true })

// Create a geospatial index
db.collection.createIndex({ "location": "2dsphere" })
```

#### Managing Indexes
- **View Index Details**: See index specifications and usage statistics
- **Drop Indexes**: Remove unnecessary indexes
- **Index Analysis**: Understand index usage patterns

### Performance Monitoring

#### Query Performance
```javascript
// Analyze query performance
db.collection.find({ email: "user@example.com" }).explain("executionStats")
```

#### System Statistics
```javascript
// Get database statistics
db.stats()

// Get collection statistics
db.collection.stats()

// Get server status
db.runCommand({ serverStatus: 1 })
```

## Migration Support

### MongoDB to DocumentDB Migration

The VS Code extension is particularly useful for migrating from MongoDB to DocumentDB:

#### Pre-Migration Analysis
1. **Schema Exploration**: Use the extension to understand your MongoDB schema
2. **Data Volume Assessment**: Check collection sizes and document counts
3. **Index Analysis**: Review existing indexes and their usage

#### Migration Process
1. **Dual Connections**: Connect to both MongoDB and DocumentDB instances
2. **Data Comparison**: Use the extension to compare data between systems
3. **Validation**: Verify data integrity after migration

#### Post-Migration Validation
1. **Query Testing**: Test your application queries in DocumentDB
2. **Performance Monitoring**: Compare query performance between systems
3. **Data Verification**: Ensure all data migrated correctly

## Best Practices

### Connection Management

1. **Use Connection Strings**: Store connection strings securely
2. **Test Connections**: Verify connectivity before performing operations
3. **Monitor Performance**: Keep an eye on query performance

### Query Optimization

1. **Use Indexes**: Create appropriate indexes for your queries
2. **Limit Results**: Use `.limit()` for large result sets
3. **Project Fields**: Use projection to return only needed fields

### Data Management

1. **Backup Regularly**: Export important data regularly
2. **Validate Data**: Check data integrity after operations
3. **Use Transactions**: Use transactions for multi-document operations

## Troubleshooting

### Common Issues

#### Connection Problems
- **Authentication**: Verify username, password, and authentication mechanism
- **Network**: Check firewall settings and network connectivity
- **SSL/TLS**: Ensure SSL certificates are valid

#### Performance Issues
- **Indexes**: Review and optimize your index strategy
- **Query Patterns**: Analyze slow queries and optimize them
- **Connection Pooling**: Use appropriate connection pool settings

#### Data Issues
- **JSON Syntax**: Validate JSON syntax in documents
- **Data Types**: Ensure data types are compatible
- **Size Limits**: Check for document size limitations

### Getting Help

1. **Extension Documentation**: Check the extension's built-in help
2. **Community Support**: Join the [Discord community](https://discord.gg/vH7bYu524D)
3. **GitHub Issues**: Report bugs on the [extension repository](https://github.com/microsoft/vscode-documentdb)

## Integration with Other Tools

### Version Control
- **Git Integration**: VS Code's Git integration works with your database scripts
- **Scrapbook Versioning**: Version control your MongoDB scrapbooks
- **Configuration Management**: Store connection configurations in version control

### Development Workflow
- **Local Development**: Use local DocumentDB instances for development
- **Staging Environment**: Connect to staging databases for testing
- **Production Monitoring**: Monitor production databases safely

## Next Steps

- Explore [MongoDB Migration Guide](mongodb-migration-guide.md) for detailed migration steps
- Learn about [DocumentDB Features](../postgres-api/index.md) for advanced capabilities
- Join our [Discord community](https://discord.gg/vH7bYu524D) for support and discussions
- Report issues and contribute on [GitHub](https://github.com/documentdb/documentdb)
