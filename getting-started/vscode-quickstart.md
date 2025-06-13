# VS Code Extension Quick Start

Get started with DocumentDB using the Visual Studio Code extension for a seamless development experience.

## Prerequisites

- Visual Studio Code installed
- Docker Desktop installed and running
- Basic familiarity with document databases
- Git installed (for cloning the repository)

## Installing the Extension

1. Open VS Code
2. Navigate to the Extensions marketplace (Ctrl+Shift+X or Cmd+Shift+X)
3. Search for "DocumentDB for VS Code"
4. Click Install
5. Reload VS Code if prompted

## Setting Up Your First Database

1. Creating a new DocumentDB instance
   ```bash
   docker pull ghcr.io/microsoft/documentdb/documentdb-local:latest
   docker tag ghcr.io/microsoft/documentdb/documentdb-local:latest documentdb
   docker run -dt -p 10260:10260 --name documentdb-container documentdb --username <YOUR_USERNAME> --password <YOUR_PASSWORD>
   docker image rm -f ghcr.io/microsoft/documentdb/documentdb-local:latest || echo "No existing documentdb image to remove"

   ```
   > **Note:** Replace `<YOUR_USERNAME>` and `<YOUR_PASSWORD>` with your desired credentials. You must set these when creating the container for authentication to work.
   > **Port Note:** Port `10260` is used by default in these instructions to avoid conflicts with other local database services. You can use port `27017` (the standard MongoDB port) or any other available port if you prefer. If you do, be sure to update the port number in both your `docker run` command and your connection string accordingly.

2. Connecting to your database
   - Click the DocumentDB icon in the VS Code sidebar
   - Click "Add New Connection"
   - On the navigation bar, click on "Connection String"
   - Paste your connection string:
     ```
     mongodb://<YOUR_USERNAME>:<YOUR_PASSWORD>@localhost:10260/?tls=true&tlsAllowInvalidCertificates=true&authMechanism=SCRAM-SHA-256
     ```

3. Creating your first database and collection
   - Click on the drop-down next to your local connection and select "Create Database..."
   - Enter database name and confirm
   - Click on the drop-down next to your created dataabse and select "Create Collection..."
   - Enter collection name and confirm
   - Repeat for every database and collection you wish to create under your connection

## Working with Documents

1. Creating documents
   - Use the Table View for quick data entry
   - Use the Tree View for hierarchical data exploration
   - Use the JSON View for detailed document structure
   ```json
   {
     "name": "Test Document",
     "type": "example",
     "created_at": new Date()
   }
   ```

2. Using the document explorer
   - Browse documents in multiple views:
     - Table View for quick insights
     - Tree View for hierarchical exploration
     - JSON View for detailed structure
   - Use smooth pagination for large datasets

## Import and Export

1. Importing data
   - Click on the "Import" button on each collection
   - Choose your JSON file
   - Confirm import

2. Exporting data
   - Export entire collections or query results using the "Export" button on each collection

## Debugging and Troubleshooting

1. Common issues and solutions

2. Using the extension logs

3. Getting support
   - Visit our [GitHub repository](https://github.com/microsoft/vscode-documentdb)
   - Join the community discussions on [Discord]()
   - Check documentation

## Next Steps

- Explore advanced querying capabilities in the [API Reference](../api-reference/index.md)
- Learn about indexing strategies in the [Architecture](../architecture/index.md) section
- Connect your application using one of our [Language Guides](python-setup.md) 