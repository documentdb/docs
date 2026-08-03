# DocumentDB Documentation

Welcome to the official documentation for [DocumentDB](https://github.com/documentdb/documentdb).

## Documentation Sections

- [Getting Started](getting-started/index.md) - Quick start guides and basic concepts
- [API Reference](https://documentdb.io/docs/reference/) - Detailed API documentation
- [PostgreSQL API](postgres-api/index.md) - PostgreSQL-compatible API documentation
- [Architecture](architecture/index.md) - System architecture and design principles
- [documentdb-local](documentdb-local/index.md) - Detailed documentation of the documentdb-local container image
- [Kubernetes Operator](kubernetes-operator/index.md) - Running DocumentDB on Kubernetes, and where to find the operator's own documentation

## Contributing

When contributing to this documentation repository, keep the following in mind:

### API Reference Pages

Landing pages for the `api-reference` folder are dynamically generated. There's no need to add `index.md` files or manually maintain lists of reference articles.

To customize the documentation:

- **Landing page descriptions**: Update the `_metadata.description.md` file in each folder to change the description shown on the landing page.
- **Metadata**: Modify the YAML front matter in each article to update metadata (SEO, category, type, etc.) for individual pages.
