# Pre-built Packages

Download and install DocumentDB using our pre-built packages for various platforms.

## Available Versions

### Latest Stable Release (v1.0.0)

| Platform | Package | SHA256 | Size |
|----------|---------|--------|------|
| Linux (x86_64) | [.deb](https://github.com/microsoft/documentdb/releases/download/v1.0.0/documentdb-1.0.0-linux-x86_64.deb) [.rpm](https://github.com/microsoft/documentdb/releases/download/v1.0.0/documentdb-1.0.0-linux-x86_64.rpm) | `3a2d5fe7d1bba...` | 45.2 MB |
| macOS (Intel) | [.dmg](https://github.com/microsoft/documentdb/releases/download/v1.0.0/documentdb-1.0.0-darwin-x86_64.dmg) | `8b4c2e3f9acd...` | 42.8 MB |
| macOS (Apple Silicon) | [.dmg](https://github.com/microsoft/documentdb/releases/download/v1.0.0/documentdb-1.0.0-darwin-arm64.dmg) | `1f9e4d2c7b3a...` | 41.9 MB |
| Windows | [.msi](https://github.com/microsoft/documentdb/releases/download/v1.0.0/documentdb-1.0.0-windows-x86_64.msi) | `5c7b9f8e2d4a...` | 48.6 MB |

### Development Release (v1.1.0-beta)

| Version | Release Date | Linux (x86_64) | macOS (Intel) | macOS (ARM) | Windows | Docker |
|---------|-------------|----------------|---------------|-------------|---------|---------|
| v1.1.0-beta.3 | 2025-04-01 | [deb](https://github.com/microsoft/documentdb/releases/download/v1.1.0-beta.3/documentdb-1.1.0-beta.3-linux-x86_64.deb) [rpm](https://github.com/microsoft/documentdb/releases/download/v1.1.0-beta.3/documentdb-1.1.0-beta.3-linux-x86_64.rpm) | [dmg](https://github.com/microsoft/documentdb/releases/download/v1.1.0-beta.3/documentdb-1.1.0-beta.3-darwin-x86_64.dmg) | [dmg](https://github.com/microsoft/documentdb/releases/download/v1.1.0-beta.3/documentdb-1.1.0-beta.3-darwin-arm64.dmg) | [msi](https://github.com/microsoft/documentdb/releases/download/v1.1.0-beta.3/documentdb-1.1.0-beta.3-windows-x86_64.msi) | `v1.1.0-beta.3` |
| v1.1.0-beta.2 | 2025-03-15 | [deb](https://github.com/microsoft/documentdb/releases/download/v1.1.0-beta.2/documentdb-1.1.0-beta.2-linux-x86_64.deb) [rpm](https://github.com/microsoft/documentdb/releases/download/v1.1.0-beta.2/documentdb-1.1.0-beta.2-linux-x86_64.rpm) | [dmg](https://github.com/microsoft/documentdb/releases/download/v1.1.0-beta.2/documentdb-1.1.0-beta.2-darwin-x86_64.dmg) | [dmg](https://github.com/microsoft/documentdb/releases/download/v1.1.0-beta.2/documentdb-1.1.0-beta.2-darwin-arm64.dmg) | [msi](https://github.com/microsoft/documentdb/releases/download/v1.1.0-beta.2/documentdb-1.1.0-beta.2-windows-x86_64.msi) | `v1.1.0-beta.2` |
| v1.1.0-beta.1 | 2025-02-28 | [deb](https://github.com/microsoft/documentdb/releases/download/v1.1.0-beta.1/documentdb-1.1.0-beta.1-linux-x86_64.deb) [rpm](https://github.com/microsoft/documentdb/releases/download/v1.1.0-beta.1/documentdb-1.1.0-beta.1-linux-x86_64.rpm) | [dmg](https://github.com/microsoft/documentdb/releases/download/v1.1.0-beta.1/documentdb-1.1.0-beta.1-darwin-x86_64.dmg) | [dmg](https://github.com/microsoft/documentdb/releases/download/v1.1.0-beta.1/documentdb-1.1.0-beta.1-darwin-arm64.dmg) | [msi](https://github.com/microsoft/documentdb/releases/download/v1.1.0-beta.1/documentdb-1.1.0-beta.1-windows-x86_64.msi) | `v1.1.0-beta.1` |

## Installation Instructions

### Linux

1. Using package managers

   #### Ubuntu/Debian (apt)
   ```bash
   # Add DocumentDB repository
   curl -fsSL https://packages.microsoft.com/keys/documentdb-archive-keyring.gpg | sudo gpg --dearmor -o /usr/share/keyrings/documentdb-archive-keyring.gpg
   echo "deb [signed-by=/usr/share/keyrings/documentdb-archive-keyring.gpg] https://packages.microsoft.com/repos/documentdb stable main" | sudo tee /etc/apt/sources.list.d/documentdb.list
   
   # Update package list and install
   sudo apt update
   sudo apt install -y documentdb
   ```

   #### RHEL/CentOS (yum)
   ```bash
   # Add DocumentDB repository
   sudo rpm --import https://packages.microsoft.com/keys/documentdb-archive-keyring.gpg
   sudo yum-config-manager --add-repo https://packages.microsoft.com/repos/documentdb
   
   # Install DocumentDB
   sudo yum install -y documentdb
   ```

   #### Fedora (dnf)
   ```bash
   # Add DocumentDB repository
   sudo rpm --import https://packages.microsoft.com/keys/documentdb-archive-keyring.gpg
   sudo dnf config-manager --add-repo https://packages.microsoft.com/repos/documentdb
   
   # Install DocumentDB
   sudo dnf install -y documentdb
   ```

2. Manual installation
   ```bash
   # Download the package
   wget https://github.com/microsoft/documentdb/releases/download/v1.0.0/documentdb-1.0.0-linux-x86_64.deb
   
   # Verify checksum
   echo "3a2d5fe7d1bba..." documentdb-1.0.0-linux-x86_64.deb | sha256sum --check
   
   # Install dependencies
   sudo apt install -y ./documentdb-1.0.0-linux-x86_64.deb
   
   # Start DocumentDB service
   sudo systemctl enable --now documentdb
   ```

### macOS

1. Using Homebrew
   ```bash
   # Add DocumentDB tap
   brew tap microsoft/documentdb
   
   # Install DocumentDB
   brew install documentdb
   
   # Start DocumentDB service
   brew services start documentdb
   ```

2. Manual installation
   ```bash
   # Download the package
   curl -O https://github.com/microsoft/documentdb/releases/download/v1.0.0/documentdb-1.0.0-darwin-x86_64.dmg
   
   # Verify checksum
   echo "8b4c2e3f9acd..." documentdb-1.0.0-darwin-x86_64.dmg | shasum -a 256 --check
   
   # Mount and install
   hdiutil attach documentdb-1.0.0-darwin-x86_64.dmg
   sudo installer -pkg /Volumes/DocumentDB/DocumentDB.pkg -target /
   
   # Start DocumentDB service
   sudo launchctl load /Library/LaunchDaemons/com.microsoft.documentdb.plist
   ```

### Windows

1. Using installer
   ```powershell
   # Download MSI
   Invoke-WebRequest -Uri "https://github.com/microsoft/documentdb/releases/download/v1.0.0/documentdb-1.0.0-windows-x86_64.msi" -OutFile "documentdb.msi"
   
   # Verify checksum
   if ((Get-FileHash documentdb.msi -Algorithm SHA256).Hash -eq "5c7b9f8e2d4a...") {
       # Install silently
       Start-Process msiexec.exe -ArgumentList "/i documentdb.msi /quiet /norestart" -Wait
       # Start service
       Start-Service DocumentDB
   }
   ```

2. Manual installation
   ```powershell
   # Extract ZIP archive
   Expand-Archive documentdb-1.0.0-windows-x86_64.zip -DestinationPath C:\Program Files\DocumentDB
   
   # Add to PATH
   $env:Path += ";C:\Program Files\DocumentDB\bin"
   [Environment]::SetEnvironmentVariable("Path", $env:Path, [EnvironmentVariableTarget]::Machine)
   
   # Install and start service
   documentdb-service.exe --install
   Start-Service DocumentDB
   ```

## Docker Images

1. Official images
   ```bash
   # Pull latest stable version
   docker pull ghcr.io/microsoft/documentdb/documentdb-local:latest

   # Tag the image for convenience
   docker tag ghcr.io/microsoft/documentdb/documentdb-local:latest documentdb

   # Run the container with your chosen username and password
   docker run -dt -p 10260:10260 --name documentdb-container documentdb --username <YOUR_USERNAME> --password <YOUR_PASSWORD>
   docker image rm -f ghcr.io/microsoft/documentdb/documentdb-local:latest || echo "No existing documentdb image to remove"
   ```
   > **Note:** Replace `<YOUR_USERNAME>` and `<YOUR_PASSWORD>` with your desired credentials. You must set these when creating the container for authentication to work.
   > 
   > **Port Note:** Port `10260` is used by default in these instructions to avoid conflicts with other local database services. You can use port `27017` (the standard MongoDB port) or any other available port if you prefer. If you do, be sure to update the port number in both your `docker run` command and your connection string accordingly.

2. Version-specific tags
   ```bash
   # Pull specific version
   docker pull microsoft/documentdb:1.0.0
   
   # Run development version
   docker pull microsoft/documentdb:1.1.0-beta.3
   ```

## Verifying Installation

1. Version check
   ```bash
   documentdb --version
   ```

2. Server status
   ```bash
   # Check service status
   sudo systemctl status documentdb  # Linux
   brew services info documentdb     # macOS
   Get-Service DocumentDB           # Windows
   
   # Check server logs
   sudo journalctl -u documentdb    # Linux
   tail -f /usr/local/var/log/documentdb/documentdb.log  # macOS
   Get-Content "C:\ProgramData\DocumentDB\logs\documentdb.log"  # Windows
   ```

3. Connection test
   ```bash
   # Using MongoDB shell
   mongosh "mongodb://localhost:27017"
   
   # Using Python
   python3 -c "from pymongo import MongoClient; print(MongoClient().server_info())"
   ```

## Configuration

1. Default configuration locations
   - Linux: `/etc/documentdb/documentdb.conf`
   - macOS: `/usr/local/etc/documentdb/documentdb.conf`
   - Windows: `C:\ProgramData\DocumentDB\config\documentdb.conf`

2. Custom configuration
   ```yaml
   # documentdb.conf
   storage:
     dbPath: /var/lib/documentdb
     engine: wiredTiger
   
   systemLog:
     destination: file
     path: /var/log/documentdb/documentdb.log
     logAppend: true
   
   net:
     port: 27017
     bindIp: 0.0.0.0
   
   security:
     authorization: enabled
   ```

3. Environment variables
   ```bash
   # Common configuration options
   export DOCUMENTDB_PORT=27017
   export DOCUMENTDB_DBPATH=/var/lib/documentdb
   export DOCUMENTDB_LOGPATH=/var/log/documentdb/documentdb.log
   export DOCUMENTDB_BIND_IP=0.0.0.0
   ```

## Upgrading

1. Upgrade paths
   ```bash
   # Linux (apt)
   sudo apt update
   sudo apt upgrade documentdb
   
   # macOS (Homebrew)
   brew upgrade documentdb
   
   # Windows (PowerShell)
   # Download new MSI and run installer
   ```

2. Backup procedures
   ```bash
   # Create backup
   documentdb-dump --out /path/to/backup
   
   # Archive backup
   tar czf backup-$(date +%Y%m%d).tar.gz /path/to/backup
   ```

3. Post-upgrade verification
   ```bash
   # Check version
   documentdb --version
   
   # Verify data integrity
   documentdb-validate --dbpath /var/lib/documentdb
   
   # Check logs for errors
   tail -f /var/log/documentdb/documentdb.log
   ```

## Troubleshooting

1. Common installation issues
   - Permission errors: Ensure proper file permissions and ownership
   - Port conflicts: Check if port 27017 is already in use
   - Missing dependencies: Install required system libraries
   - Service startup failures: Check system logs for errors

2. Version compatibility
   | Client Driver | Minimum Version | Maximum Version |
   |--------------|-----------------|-----------------|
   | PyMongo      | 4.0.0          | Latest          |
   | Node.js      | 4.0.0          | Latest          |
   | mongosh      | 1.0.0          | Latest          |

3. System requirements
   | Component | Minimum | Recommended |
   |-----------|---------|-------------|
   | CPU       | 2 cores | 4+ cores    |
   | RAM       | 4 GB    | 8+ GB       |
   | Disk      | 10 GB   | 50+ GB SSD  |
   | OS        | See supported versions below |

   Supported Operating Systems:
   - Linux: Ubuntu 20.04+, RHEL 8+, CentOS 8+
   - macOS: 11.0+ (Big Sur)
   - Windows: Windows 10/11, Windows Server 2019+

## Support Matrix

| Version | Release Date | End of Support | PostgreSQL Version |
|---------|-------------|----------------|-------------------|
| 1.0.0   | 2025-01-23  | 2027-01-23    | 15.x             |
| 1.1.0-beta| 2025-03-15 | -             | 16.x             | 