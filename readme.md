# DocumentDB Documentation

Welcome to the official documentation for [DocumentDB](https://github.com/documentdb/documentdb).

## Documentation Sections

- [Getting Started](getting-started/index.md) - Quick start guides and basic concepts
- [API Reference](api-reference/index.md) - Detailed API documentation
- [PostgreSQL API](postgres-api/index.md) - PostgreSQL-compatible API documentation
- [Architecture](architecture/index.md) - System architecture and design principles
- [documentdb-local](documentdb-local/index.md) - Detailed documentation of the documentdb-local container image
- [DocumentDB Kubernetes Operator](https://documentdb.io/documentdb-kubernetes-operator) - Documentation for the DocumentDB Kubernetes Operator

## Contributing

When contributing to this documentation repository, keep the following in mind:

### API Reference Pages

Landing pages for the `api-reference` folder are dynamically generated. There's no need to add `index.md` files or manually maintain lists of reference articles.

To customize the documentation:

- **Landing page descriptions**: Update the `_metadata.description.md` file in each folder to change the description shown on the landing page.
- **Metadata**: Modify the YAML front matter in each article to update metadata (SEO, category, type, etc.) for individual pages.

### Articles

Documentation articles are stored in the following folders:

- `/architecture`
- `/documentdb-local`
- `/getting-started`
- `/postgres-api`

Each folder contains Markdown files (*\*.md*) and a *navigation.yml* file. Markdown files require the following YAML front matter metadata:

| | Description |
| --- | --- |
| **`title`** | The title of the page rendered at the top of the article and in the browser tab/window. |
| **`description`** | The description of the page for SEO metadata and social media sharing. |

Articles **must** also be included in the *navigation.yml* file in order to appear in the navigation bar for the documentation site.
