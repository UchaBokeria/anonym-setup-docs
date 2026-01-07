# Web UI Configuration

## Overview

The Web UI (WUI) provides a browser-based interface for configuring environment variables and managing SSL certificates. It serves as an alternative to manually editing `.env` files and placing certificate files in directories. The WUI includes three main sections: Configuration, Certificates, and References.

## Prerequisites

The `.wui` configuration file is already present in the repository and ready to use. You can modify this file if you need to change the port or host binding.

Default `.wui` settings:
- PORT: `:3030`
- HOST: `localhost`
- GOENV: `production`
- OUTPUT: `.env`

## Starting the Web UI

Run the WUI binary from the project root:

```terminal
./wui
```

The binary will start a web server on the configured port. If HOST is set to `localhost`, the binary will automatically detect and display your public IP address for remote browser access.

## Accessing the Interface

Once the WUI is running:

- **Default access**: `http://localhost:3030`
- **Remote access**: If HOST=localhost, use the public IP displayed by the binary
- **Network access**: If HOST=0.0.0.0, accessible from any network interface at `http://[server-ip]:3030`

## Web UI Features

The WUI interface is organized into three main tabs accessible from the top navigation.

### Configuration Tab

The Configuration tab provides a comprehensive form-based interface for all environment variables:

**Service Categories**:
- Application - General application settings
- LDAP / Active Directory - Authentication configuration
- PostgreSQL - Database connection settings
- PgBouncer - Connection pooling configuration
- S3 Minio - Object storage settings
- AI Services - OpenAI and Azure Language Service configuration
- Next.js Frontend - Frontend application settings

**Features**:
- All fields are pre-filled with default values
- Required fields are marked with a red asterisk (*)
- Real-time form validation
- Smart error handling: If required fields are missing, an error toast notification appears with a clickable link
- Clicking the error notification automatically scrolls to the missing field with an animated border highlight
- Click "Save" button to generate or update the `.env` file in the project root
- Each save operation overwrites the existing `.env` file with current form values

**Workflow**:
1. Navigate through service tabs to configure each component
2. Fill in required fields (marked with *)
3. Click "Save" to generate `.env` file
4. If validation fails, click the error notification to jump to the problematic field
5. After successful save, proceed with Docker deployment

### Certificates Tab

The Certificates tab provides visual SSL/TLS configuration management:

**Features**:
- Choose SSL/TLS options for different services
- Upload custom certificates through the web interface
- Alternative to manually placing certificate files in `./certs/` directories
- Visual configuration for Nginx, Database, PgBouncer, and LDAP certificates

**Use Cases**:
- Upload production certificates without command-line file operations
- Configure Let's Encrypt settings
- Manage self-signed certificate options
- Centralized certificate management interface

### References Tab

The References tab provides a searchable encyclopedia of all configuration variables:

**Features**:
- Complete list of all environment variables with descriptions
- Search functionality by name, description, category, or current value
- Color-coded categories for easy identification
- Displays current values from your `.env` file
- Click any variable name to navigate directly to its field in the Configuration tab with highlight animation

**Use Cases**:
- Quick reference when you need to find a specific variable
- Search for variables by functionality (e.g., search "SSL" to find all SSL-related settings)
- Verify current configuration values
- Navigate quickly to specific fields for editing

## WUI Configuration File

The `.wui` file in the project root controls WUI behavior:

```
PORT=:3030          # Port for WUI web interface (e.g., :3030, :8080)
HOST=localhost      # Host binding (localhost or 0.0.0.0 for all interfaces)
GOENV=production    # WUI environment mode (production/dev)
OUTPUT=.env         # Output file path for generated configuration
```

**Configuration Options**:

**PORT**
- Web server port for the WUI interface
- Format: `:PORT_NUMBER` (e.g., `:3030`, `:8080`)
- Change if port 3030 is already in use
- Default: `:3030`

**HOST**
- Host binding for the web server
- `localhost`: Only accessible from local machine (binary shows public IP for remote access)
- `0.0.0.0`: Accessible from any network interface
- Default: `localhost`

**GOENV**
- Environment mode for the WUI application
- Values: `production`, `dev`
- Default: `production`

**OUTPUT**
- Path where the generated `.env` file will be written
- Relative to WUI binary location
- Default: `.env`

**Note**: When HOST=localhost, the `./wui` binary automatically detects and displays your public IP address so you can access the interface from a browser on another machine.

