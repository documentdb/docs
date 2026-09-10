---
title: Visual Studio Code Quick Start
description: Get started with DocumentDB using the Visual Studio Code extension for a seamless development experience.
---

# VS Code Extension Quick Start

Get started with DocumentDB using the Visual Studio Code extension for a seamless development experience.

The extension can set up a local DocumentDB instance for you: it pulls the official image, starts the container, waits until the database accepts connections, and saves the connection. It never installs Docker or elevates privileges.

## Prerequisites

- Visual Studio Code installed
- Docker Desktop or Docker Engine installed and running, configured for Linux containers
- Basic familiarity with document databases

Docker must be reachable from the environment VS Code is running in. If you work in WSL, a dev container, an SSH remote, or Codespaces, Docker needs to be available there rather than only on your host machine. The setup wizard runs a readiness check and explains what to fix if it cannot reach Docker.

## Installing the Extension

1. Open VS Code
2. Navigate to the Extensions marketplace (Ctrl+Shift+X or Cmd+Shift+X)
3. Search for "DocumentDB for VS Code"
4. Click Install
5. Reload VS Code if prompted

You can also install it from the [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-documentdb).

## Set Up DocumentDB Local

This is the fastest way to get a database running. The wizard handles the container, the port, and the credentials, so you do not run any Docker commands yourself.

1. Open the setup wizard using any of these:
   - Select the DocumentDB icon in the primary sidebar, expand **Your own DocumentDB** in the Connections view, and select **Set up DocumentDB Local**.
   - Run **DocumentDB: Set up DocumentDB Local** from the Command Palette (Ctrl+Shift+P or Cmd+Shift+P).
   - Paste `vscode://ms-azuretools.vscode-documentdb/local` into your browser address bar and allow VS Code to open the link. This requires extension version 0.10.1 or later.

2. On the **Introduction** step, select **Continue**.

   > **Note:** Nothing is downloaded or created on your machine until you start the instance in the next step.

3. On the **Configure** step, review the defaults and select **Start DocumentDB Local**.

   The defaults give you an available port (starting at `10260`), generated credentials, the `latest` official image, and optional sample data. Expand the advanced options to set the port, the image tag, or the credentials yourself, or to skip the sample data.

4. Wait for setup to finish. The extension pulls the image if needed, creates a container named `vscode-documentdb-local` with a persistent volume, starts it, and waits until the database accepts connections.

5. Select **Open Connection** to reveal the saved connection in the Connections view, then expand it to browse databases and collections.

If you keep the sample data option, the extension loads the image's own dataset, so you have something to query immediately. On the `latest` image, that is [StoreData](https://documentdb.io/docs/documentdb-local/#built-in-sample-data): a `StoreData` database holding 41,505 `stores` documents and 2 `ratings` documents. Images before 0.117 seed a smaller `sampledb` database instead, so pin an older tag in the advanced options if you want that one.

### Managing the Instance

Right-click the DocumentDB Local entry in the Connections view to **Start**, **Stop**, **Restart**, or **Delete Container**, and to **Copy Connection String**, **Copy Password**, or **View Logs**. Stopping and starting preserves your data. Deleting the container also removes its data volume and the generated credentials, permanently.

## Alternative: Connect to a Container You Started Yourself

Use this if you already run DocumentDB Local outside VS Code, or you want to manage the container yourself.

1. Creating a new DocumentDB instance

   **Bash**

   ```bash
   docker pull ghcr.io/documentdb/documentdb/documentdb-local:latest
   docker tag ghcr.io/documentdb/documentdb/documentdb-local:latest documentdb
   read -r -p 'DocumentDB username: ' DOCUMENTDB_USERNAME
   read -r -s -p 'DocumentDB password: ' DOCUMENTDB_PASSWORD
   printf '\n'
   export DOCUMENTDB_USERNAME DOCUMENTDB_PASSWORD
   if docker run -dt -p 127.0.0.1:10260:10260 --name documentdb-container documentdb --username "${DOCUMENTDB_USERNAME:?DocumentDB username cannot be empty}" --password "${DOCUMENTDB_PASSWORD:?DocumentDB password cannot be empty}"; then
     docker image rm -f ghcr.io/documentdb/documentdb/documentdb-local:latest || echo "No existing documentdb image to remove"
   fi
   ```

   **PowerShell**

   ```powershell
   docker pull ghcr.io/documentdb/documentdb/documentdb-local:latest
   docker tag ghcr.io/documentdb/documentdb/documentdb-local:latest documentdb
   $env:DOCUMENTDB_USERNAME = Read-Host 'DocumentDB username'
   $securePassword = Read-Host 'DocumentDB password' -AsSecureString
   $env:DOCUMENTDB_PASSWORD = [System.Net.NetworkCredential]::new('', $securePassword).Password
   if ([string]::IsNullOrWhiteSpace($env:DOCUMENTDB_USERNAME) -or [string]::IsNullOrWhiteSpace($env:DOCUMENTDB_PASSWORD)) {
       throw 'DocumentDB credentials cannot be empty'
   } else {
       docker run -dt -p 127.0.0.1:10260:10260 --name documentdb-container documentdb --username "$env:DOCUMENTDB_USERNAME" --password "$env:DOCUMENTDB_PASSWORD"
       if ($LASTEXITCODE -eq 0) {
           docker image rm -f ghcr.io/documentdb/documentdb/documentdb-local:latest
           if ($LASTEXITCODE -ne 0) { echo "No existing documentdb image to remove" }
       }
   }
   ```

   > **Note:** During the transition to the Linux Foundation, Docker images may still be hosted on Microsoft's container registry. These will be migrated to the new DocumentDB organization as the transition completes.
   >
   > **Note:** Both versions prompt for credentials and reject empty values so the container cannot fall through to the public `default_user` / `Admin100` defaults. Enter the same values when the extension asks for the local connection credentials.
   >
   > **Network Note:** The example binds the gateway only to the local host. Expose it to other machines only after adding firewall rules and a certificate those clients can validate.
   >
   > **Port Note:** To use host port `27017` while leaving the gateway on its default container port, publish `-p 127.0.0.1:27017:10260` and enter `27017` in the extension. To change the gateway's internal port too, add `--documentdb-port 27017` after the image name and publish that container port.

2. Connecting to your database
   - Locate and select the DocumentDB icon in the primary VS Code sidebar on the left-hand side.
   - Add a new connection to your DocumentDB:
     - In the DocumentDB Connections area, locate and expand the **DocumentDB Local** node.
     - Select the **New Local Connection** option.
     - Confirm the port (default value `10260`), username, password, and choose the **Disable TLS/SSL** option.
     - **Note:** TLS/SSL can be enabled, but this walkthrough skips those steps for simplicity.
     - A new DocumentDB Local entry will be added and listed in your DocumentDB Connections area.

## Creating Databases and Collections

1. Click on the drop-down next to your connection and select "Create Database..."
2. Enter database name and confirm
3. Click on the drop-down next to your created database and select "Create Collection..."
4. Enter collection name and confirm
5. Repeat for every database and collection you wish to create under your connection

## Working with Documents

1. Creating documents
   - Use the Table View for quick data entry
   - Use the Tree View for hierarchical data exploration
   - Use the JSON View for detailed document structure
   ```json
   {
     "name": "Test Document",
     "type": "example",
     "created_at": { "$date": "2026-08-25T00:00:00Z" }
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

1. The browser link does not open setup

   Confirm the extension is installed and up to date, then run **DocumentDB: Set up DocumentDB Local** from the Command Palette instead. The deep link requires version 0.10.1 or later.

2. Setup reports that Docker is not reachable

   The wizard names the specific problem, such as Docker not running, or Docker configured for Windows containers rather than Linux containers. Fix what it reports and select **Continue setup**. Nothing has been created on your machine at that point.

3. Using the extension logs

   Right-click the DocumentDB Local entry in the Connections view and select **View Logs** to follow the container's output. Extension logs are in the Output panel under the DocumentDB channel.

4. Getting support
   - Visit our [GitHub repository](https://github.com/microsoft/vscode-documentdb)
   - Join the community on [Discord](https://discord.gg/vH7bYu524D)
   - Read the [extension user manual](https://microsoft.github.io/vscode-documentdb/user-manual/local-quick-start)

## Next Steps

- Explore advanced querying capabilities in the [API Reference](https://documentdb.io/docs/reference/)
- Connect your application using the [Python Setup for DocumentDB](https://documentdb.io/docs/getting-started/python-setup/) 
