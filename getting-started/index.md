---
title: Getting Started
description: DocumentDB is an open-source document database platform built on PostgreSQL. It offers developers a fully permissive, open-source platform for document data stores.
---

# Introduction to DocumentDB

DocumentDB is an open-source document database platform built on PostgreSQL. It offers developers a fully permissive, open-source platform for document data stores.

## What is DocumentDB?

DocumentDB provides a NoSQL datastore implemented using PostgreSQL, giving developers complete visibility into the architecture and implementation of the engine. It's designed to offer:

- Full compatibility with MongoDB wire protocol through the `pg_documentdb_api` layer
- Native BSON document support via the `pg_documentdb_core` PostgreSQL extension
- Advanced indexing capabilities including single field, multi-key, compound, text, and geospatial indexes
- Vector search functionality powered by the `pg_vector` PostgreSQL extension
- Enterprise-grade security with SCRAM authentication
- Full PostgreSQL compatibility for advanced SQL operations

## Key Features

- **PostgreSQL Foundation**: Built on the powerful PostgreSQL engine, allowing you to leverage both document and relational capabilities
- **Open Source**: Released under the MIT license with no restrictions on usage, modification, or distribution
- **Document Database Standard**: First implementation towards creating an open standard for document databases, similar to ANSI SQL for relational databases
- **Cloud Ready**: Supports multi-cloud deployments across major cloud providers with native integration in Azure Cosmos DB
- **Developer Friendly**: Rich ecosystem of tools and extensions, including VS Code integration and MongoDB compatibility

## Architecture Components

DocumentDB consists of two primary components:

1. **pg_documentdb_core**: A custom PostgreSQL extension that provides:
   - Native BSON data type support in PostgreSQL
   - Field-level indexing at any nesting depth
   - Multi-key, compound, text, and geospatial indexing
   - Vector search capabilities for AI/ML applications
   - SCRAM authentication mechanism

2. **pg_documentdb_api**: The data plane implementing:
   - CRUD operations with MongoDB compatibility
   - Advanced query functionality
   - Index management and optimization
   - Protocol translation layer for MongoDB wire protocol

## Common Use Cases

- **Modern Web Applications**: Store and query JSON documents with MongoDB compatibility
- **AI/ML Applications**: Leverage vector search for similarity matching and embeddings
- **Hybrid Data Models**: Combine document and relational data in the same database
- **Migration Path**: Easy transition from existing MongoDB workloads
- **Local Development**: Full-featured local instance for development and testing

## Getting Started Options

Choose the getting started guide that best fits your needs:

### Quick Start Guides
- [VS Code Extension Quick Start](https://documentdb.io/docs/getting-started/vscode-quickstart) - Recommended for developers new to DocumentDB
- [VS Code Extension Guide](https://documentdb.io/docs/getting-started/vscode-extension-guide) - Comprehensive guide to the VS Code extension

### Language-Specific Guides
- [Python Setup Guide](https://documentdb.io/docs/getting-started/python-setup) - Using DocumentDB with Python applications
- [Node.js Setup Guide](https://documentdb.io/docs/getting-started/nodejs-setup) - Using DocumentDB with Node.js applications

### Deployment Options
- [Pre-built Packages](https://documentdb.io/docs/getting-started/prebuilt-packages) - Download and install ready-to-use packages

## Community and Support

- **Discord Community**: Join our [Discord server](https://discord.gg/vH7bYu524D) for:
  - Community syncs and office hours
  - Real-time technical support
  - Migration assistance
  - Best practices sharing
- GitHub Repository: [documentdb/documentdb](https://github.com/documentdb/documentdb)
- Documentation: Comprehensive guides and API references
- Issue Tracking: Report bugs and request features on GitHub

## Next Steps

After choosing your preferred getting started path:
- Explore our [API Reference](https://documentdb.io/docs/reference) for detailed documentation
- Join our community to contribute and get support 
