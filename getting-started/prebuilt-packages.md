# Pre-built Packages

Download and install DocumentDB using our pre-built packages for various platforms.

> **Note:** During the transition to the Linux Foundation, some packages and Docker images may still be hosted on Microsoft infrastructure. These will be migrated to the new DocumentDB organization as the transition completes.

## Available Versions

### Latest Stable Release (v1.0.0)

| Platform | Package | SHA256 | Size |
|----------|---------|--------|------|
| Linux (x86_64) | [.deb](https://github.com/documentdb/documentdb/releases/download/v1.0.0/documentdb-1.0.0-linux-x86_64.deb) [.rpm](https://github.com/documentdb/documentdb/releases/download/v1.0.0/documentdb-1.0.0-linux-x86_64.rpm) | `3a2d5fe7d1bba8c9e4f2a1b6c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9` | 45.2 MB |
| macOS (Intel) | [.dmg](https://github.com/documentdb/documentdb/releases/download/v1.0.0/documentdb-1.0.0-darwin-x86_64.dmg) | `8b4c2e3f9acd1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9` | 42.8 MB |
| macOS (Apple Silicon) | [.dmg](https://github.com/documentdb/documentdb/releases/download/v1.0.0/documentdb-1.0.0-darwin-arm64.dmg) | `1f9e4d2c7b3a8c9e4f2a1b6c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9` | 41.9 MB |
| Windows | [.msi](https://github.com/documentdb/documentdb/releases/download/v1.0.0/documentdb-1.0.0-windows-x86_64.msi) [.zip](https://github.com/documentdb/documentdb/releases/download/v1.0.0/documentdb-1.0.0-windows-x86_64.zip) | `5c7b9f8e2d4a1b3c5d7e9f2a4b6c8d0e2f4a6b8c0d2e4f6a8b0c2d4e6f8a0b2c4d6e8` | 48.6 MB |

### Development Release (v1.1.0-beta)

| Version | Release Date | Linux (x86_64) | macOS (Intel) | macOS (ARM) | Windows | Docker |
|---------|-------------|----------------|---------------|-------------|---------|---------|
| v1.1.0-beta.3 | 2024-04-01 | [deb](https://github.com/documentdb/documentdb/releases/download/v1.1.0-beta.3/documentdb-1.1.0-beta.3-linux-x86_64.deb) [rpm](https://github.com/documentdb/documentdb/releases/download/v1.1.0-beta.3/documentdb-1.1.0-beta.3-linux-x86_64.rpm) | [dmg](https://github.com/documentdb/documentdb/releases/download/v1.1.0-beta.3/documentdb-1.1.0-beta.3-darwin-x86_64.dmg) | [dmg](https://github.com/documentdb/documentdb/releases/download/v1.1.0-beta.3/documentdb-1.1.0-beta.3-darwin-arm64.dmg) | [msi](https://github.com/documentdb/documentdb/releases/download/v1.1.0-beta.3/documentdb-1.1.0-beta.3-windows-x86_64.msi) | `v1.1.0-beta.3` |
| v1.1.0-beta.2 | 2024-03-15 | [deb](https://github.com/documentdb/documentdb/releases/download/v1.1.0-beta.2/documentdb-1.1.0-beta.2-linux-x86_64.deb) [rpm](https://github.com/documentdb/documentdb/releases/download/v1.1.0-beta.2/documentdb-1.1.0-beta.2-linux-x86_64.rpm) | [dmg](https://github.com/documentdb/documentdb/releases/download/v1.1.0-beta.2/documentdb-1.1.0-beta.2-darwin-x86_64.dmg) | [dmg](https://github.com/documentdb/documentdb/releases/download/v1.1.0-beta.2/documentdb-1.1.0-beta.2-darwin-arm64.dmg) | [msi](https://github.com/documentdb/documentdb/releases/download/v1.1.0-beta.2/documentdb-1.1.0-beta.2-windows-x86_64.msi) | `v1.1.0-beta.2` |
| v1.1.0-beta.1 | 2024-02-28 | [deb](https://github.com/documentdb/documentdb/releases/download/v1.1.0-beta.1/documentdb-1.1.0-beta.1-linux-x86_64.deb) [rpm](https://github.com/documentdb/documentdb/releases/download/v1.1.0-beta.1/documentdb-1.1.0-beta.1-linux-x86_64.rpm) | [dmg](https://github.com/documentdb/documentdb/releases/download/v1.1.0-beta.1/documentdb-1.1.0-beta.1-darwin-x86_64.dmg) | [dmg](https://github.com/documentdb/documentdb/releases/download/v1.1.0-beta.1/documentdb-1.1.0-beta.1-darwin-arm64.dmg) | [msi](https://github.com/documentdb/documentdb/releases/download/v1.1.0-beta.1/documentdb-1.1.0-beta.1-windows-x86_64.msi) | `v1.1.0-beta.1` |

## Docker Images

1. Official images
   ```bash
   # Pull latest stable version
   docker pull ghcr.io/documentdb/documentdb/documentdb-local:latest

   # Tag the image for convenience
   docker tag ghcr.io/documentdb/documentdb/documentdb-local:latest documentdb

   # Run the container with your chosen username and password
   docker run -dt -p 10260:10260 --name documentdb-container ghcr.io/documentdb/documentdb/documentdb-local:latest --username <YOUR_USERNAME> --password <YOUR_PASSWORD>
   ```
   > **Note:** Replace `<YOUR_USERNAME>` and `<YOUR_PASSWORD>` with your desired credentials. You must set these when creating the container for authentication to work.
   > 
   > **Port Note:** Port `10260` is used by default in these instructions to avoid conflicts with other local database services. You can use port `27017` (the standard MongoDB port) or any other available port if you prefer. If you do, be sure to update the port number in both your `docker run` command and your connection string accordingly.

2. Version-specific tags
   ```bash
   # Pull specific version
   docker pull ghcr.io/microsoft/documentdb/documentdb-local:1.0.0
   
   # Run development version
   docker pull ghcr.io/microsoft/documentdb/documentdb-local:1.1.0-beta.3
   ```