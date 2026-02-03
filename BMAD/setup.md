# Mattermost Development Setup Guide

This guide will help you set up a local development environment for Mattermost.

## Prerequisites

### Required Software

- **Go**: Version 1.21 or higher (1.24.11 recommended)
- **Node.js**: Version 18.x or higher
- **npm**: Version 8.x or higher
- **Docker**: For running PostgreSQL, MySQL, and Elasticsearch
- **Docker Compose**: For orchestrating development services
- **Git**: For version control

### Operating System Support

Mattermost development is supported on:
- macOS (Intel and Apple Silicon)
- Linux (Ubuntu 20.04+, Debian, RHEL 8+)
- Windows 10/11 (with WSL2 recommended)

## Installation Steps

### 1. Clone the Repository

```bash
git clone https://github.com/mattermost/mattermost.git
cd mattermost
```

### 2. Install Go

**macOS (using Homebrew):**
```bash
brew install go@1.24
```

**Linux:**
```bash
wget https://go.dev/dl/go1.24.11.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.24.11.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
```

**Windows:**
Download and install from [https://go.dev/dl/](https://go.dev/dl/)

Verify installation:
```bash
go version
# Should output: go version go1.24.11 ...
```

### 3. Install Node.js and npm

**macOS (using Homebrew):**
```bash
brew install node@18
```

**Linux (using nvm):**
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 18
nvm use 18
```

**Windows:**
Download and install from [https://nodejs.org/](https://nodejs.org/)

Verify installation:
```bash
node --version  # Should be v18.x or higher
npm --version   # Should be 8.x or higher
```

### 4. Install Docker and Docker Compose

Follow the official Docker installation guide for your platform:
- [Docker Desktop for Mac](https://docs.docker.com/desktop/install/mac-install/)
- [Docker Desktop for Windows](https://docs.docker.com/desktop/install/windows-install/)
- [Docker Engine for Linux](https://docs.docker.com/engine/install/)

Verify installation:
```bash
docker --version
docker compose version
```

### 5. Set Up Go Workspace

```bash
cd server
make setup-go-work
```

This creates a `go.work` file that manages the Go modules for the project.

### 6. Install Backend Dependencies

```bash
cd server
go mod download
```

### 7. Install Frontend Dependencies

```bash
cd webapp/channels
npm install
```

### 8. Start Docker Services

From the `server` directory:

```bash
make start-docker
```

This starts:
- PostgreSQL database (default)
- Elasticsearch for search
- MinIO for file storage
- OpenLDAP (if enterprise features enabled)

**Note:** On Apple Silicon Macs, an Elasticsearch override is automatically applied.

### 9. Build the Application

**Build Server:**
```bash
cd server
make build
```

**Build Webapp:**
```bash
cd webapp/channels
npm run build
```

## Configuration

### Database Configuration

By default, Mattermost uses PostgreSQL. The connection string is configured in `server/config/config.json`.

**Default PostgreSQL connection:**
```
postgres://mmuser:mostest@localhost:5432/mattermost_test?sslmode=disable
```

**To use MySQL instead:**
```bash
# Edit server/config/config.json
"DriverName": "mysql",
"DataSource": "mmuser:mostest@tcp(localhost:3306)/mattermost_test?charset=utf8mb4,utf8&writeTimeout=30s"
```

### Environment Variables

Create a `.env` file in the `server` directory (optional):

```bash
# Server settings
MM_SERVICESETTINGS_SITEURL=http://localhost:8065
MM_SERVICESETTINGS_LISTENADDRESS=:8065

# Database
MM_SQLSETTINGS_DRIVERNAME=postgres
MM_SQLSETTINGS_DATASOURCE=postgres://mmuser:mostest@localhost:5432/mattermost_test?sslmode=disable

# File storage
MM_FILESETTINGS_DRIVERNAME=local
MM_FILESETTINGS_DIRECTORY=./data/

# Elasticsearch
MM_ELASTICSEARCHSETTINGS_CONNECTIONURL=http://localhost:9200
```

## Verification

### Test the Setup

**Run server tests:**
```bash
cd server
make test-server-quick
```

**Run webapp tests:**
```bash
cd webapp/channels
npm test
```

### Check Service Health

Verify Docker services are running:
```bash
docker compose ps
```

You should see:
- `postgres` - Running on port 5432
- `elasticsearch` - Running on port 9200
- `minio` - Running on port 9000

## Troubleshooting

### Port Conflicts

If ports 8065, 5432, 9200, or 9000 are already in use:

```bash
# Stop conflicting services or change ports in docker-compose.makefile.yml
make stop-docker
```

### Permission Issues

On Linux, if you encounter permission errors:

```bash
sudo chown -R $USER:$USER .
```

### Go Module Issues

If you see Go module errors:

```bash
cd server
go clean -modcache
go mod download
```

### Node Module Issues

If you encounter npm errors:

```bash
cd webapp/channels
rm -rf node_modules package-lock.json
npm install
```

### Apple Silicon (M1/M2) Issues

Elasticsearch may require Rosetta 2:

```bash
softwareupdate --install-rosetta
```

The Makefile automatically applies the M1 override for Elasticsearch.

## Next Steps

Once setup is complete, proceed to [usage.md](./usage.md) to learn how to run and develop Mattermost.

## Additional Resources

- [Official Developer Setup Guide](https://developers.mattermost.com/contribute/server/developer-setup)
- [Mattermost Developer Documentation](https://developers.mattermost.com/)
- [Troubleshooting Guide](https://docs.mattermost.com/install/troubleshooting.html)
