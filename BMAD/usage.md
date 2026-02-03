# Mattermost Usage Guide

This guide covers how to run, develop, and use Mattermost in your local development environment.

## Running Mattermost

### Quick Start

**Run both server and webapp:**
```bash
cd server
make run
```

This command:
1. Starts Docker services (PostgreSQL, Elasticsearch, MinIO)
2. Builds and runs the Go server on `http://localhost:8065`
3. Starts the webpack dev server for the React webapp

### Running Components Separately

**Server only:**
```bash
cd server
make run-server
```

**Webapp only:**
```bash
cd server
make run-client
```

**Or from webapp directory:**
```bash
cd webapp/channels
npm run run
```

### Stopping Services

**Stop server:**
```bash
cd server
make stop-server
```

**Stop webapp:**
```bash
cd server
make stop-client
```

**Stop everything (including Docker):**
```bash
cd server
make stop
```

## Development Workflow

### Making Code Changes

#### Backend (Go) Development

1. **Make changes** to Go files in `server/channels/`
2. **Restart the server** to see changes:
   ```bash
   make restart-server
   ```

The server will recompile and restart automatically.

**Hot reload with Air (optional):**
```bash
go install github.com/cosmtrek/air@latest
cd server
air
```

#### Frontend (React/TypeScript) Development

1. **Make changes** to files in `webapp/channels/src/`
2. **Webpack dev server** automatically hot-reloads changes
3. **No restart needed** - changes appear in browser immediately

### Running Tests

#### Backend Tests

**Quick tests (unit tests only):**
```bash
cd server
make test-server-quick
```

**Full test suite:**
```bash
cd server
make test-server
```

**Specific package:**
```bash
cd server
go test ./channels/app/...
```

**With coverage:**
```bash
cd server
ENABLE_COVERAGE=true make test-server
make cover  # Opens coverage report in browser
```

#### Frontend Tests

**Run all tests:**
```bash
cd webapp/channels
npm test
```

**Run tests in watch mode:**
```bash
cd webapp/channels
npm run test:watch
```

**Run specific test file:**
```bash
cd webapp/channels
npm test -- user_settings
```

### Debugging

#### Debug Server with Delve

**Interactive debugging:**
```bash
cd server
make debug-server
```

**Headless mode (for IDE integration):**
```bash
cd server
make debug-server-headless
```

Then connect your IDE debugger to `localhost:2345`.

**VS Code launch.json example:**
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Connect to Mattermost",
            "type": "go",
            "request": "attach",
            "mode": "remote",
            "remotePath": "${workspaceFolder}/server",
            "port": 2345,
            "host": "localhost"
        }
    ]
}
```

#### Debug Webapp

Use browser DevTools:
1. Open `http://localhost:8065` in Chrome/Firefox
2. Press `F12` to open DevTools
3. Source maps are enabled by default in development

**React DevTools:**
Install the [React Developer Tools](https://react.dev/learn/react-developer-tools) browser extension.

**Redux DevTools:**
Install the [Redux DevTools](https://github.com/reduxjs/redux-devtools) browser extension.

### Code Quality

#### Linting

**Backend (Go):**
```bash
cd server
make check-style
```

**Frontend (TypeScript/JavaScript):**
```bash
cd webapp/channels
npm run check
```

#### Formatting

**Backend:**
```bash
cd server
gofmt -w .
```

**Frontend:**
```bash
cd webapp/channels
npm run fix
```

## Accessing Mattermost

### Web Interface

Open your browser to:
```
http://localhost:8065
```

### First-Time Setup

1. **Create an account:**
   - Click "Create an account"
   - Enter email, username, and password
   - First user becomes system admin

2. **Create a team:**
   - Enter team name
   - Choose team URL

3. **Create channels:**
   - Click "+" next to "Channels"
   - Create public or private channels

### Test Data

**Load sample data:**
```bash
cd server
make test-data
```

This creates:
- System admin: `sysadmin` / `Sys@dmin-sample1`
- Regular user: `user-1` / `SampleUs@r-1`
- 60 sample users
- Multiple teams and channels
- Sample posts and messages

### Admin Console

Access the System Console at:
```
http://localhost:8065/admin_console
```

Login with system admin credentials to configure:
- Server settings
- Authentication
- Integrations
- Plugins
- User management

## Common Development Tasks

### Adding a New API Endpoint

1. **Define the model** in `server/channels/model/`
2. **Create store method** in `server/channels/store/sqlstore/`
3. **Add app logic** in `server/channels/app/`
4. **Create API handler** in `server/channels/api4/`
5. **Update OpenAPI spec** in `api/v4/`
6. **Write tests** for each layer

Example structure:
```
server/channels/
├── model/my_feature.go          # Data structures
├── store/sqlstore/my_feature_store.go  # Database access
├── app/my_feature.go            # Business logic
└── api4/my_feature.go           # HTTP handlers
```

### Adding a New React Component

1. **Create component** in `webapp/channels/src/components/`
2. **Create styles** in same directory (`.scss` file)
3. **Add Redux actions** in `webapp/channels/src/actions/`
4. **Add Redux reducers** in `webapp/channels/src/reducers/`
5. **Add selectors** in `webapp/channels/src/selectors/`
6. **Write tests** in `__tests__` directory

Example structure:
```
webapp/channels/src/
├── components/my_feature/
│   ├── index.tsx                # Component
│   ├── my_feature.scss          # Styles
│   └── __tests__/
│       └── my_feature.test.tsx  # Tests
├── actions/my_feature.ts        # Redux actions
├── reducers/my_feature.ts       # Redux reducer
└── selectors/my_feature.ts      # Selectors
```

### Working with Plugins

**Install a plugin:**
```bash
# Download plugin .tar.gz
# Upload via System Console > Plugins > Management
# Or use mmctl:
bin/mmctl plugin add plugin.tar.gz --local
```

**Develop a plugin:**
See [Building A Plugin](https://developers.mattermost.com/integrate/plugins/) in the developer docs.

### Database Migrations

**Create a new migration:**
```bash
cd server
make new-migration name=add_my_feature_table
```

This creates a new migration file in `server/channels/db/migrations/`.

**Run migrations:**
Migrations run automatically on server start.

### Working with mmctl (CLI)

**Build mmctl:**
```bash
cd server
make mmctl-build
```

**Use mmctl:**
```bash
# Connect to local server
bin/mmctl auth login http://localhost:8065 --name local --username admin --password admin

# List users
bin/mmctl user list

# Create a user
bin/mmctl user create --email user@example.com --username testuser --password Password1!

# Use local mode (no authentication needed)
bin/mmctl user list --local
```

## Performance Optimization

### Development Mode

Development builds include:
- Source maps
- Hot module replacement
- Verbose logging
- No minification

### Production Build

**Build for production:**
```bash
# Server
cd server
make build

# Webapp
cd webapp/channels
npm run build
```

Production builds:
- Minified JavaScript
- Optimized assets
- No source maps
- Compressed bundles

## Environment-Specific Configuration

### Local Development

Default configuration in `server/config/config.json`.

### Testing Environment

```bash
export MM_SQLSETTINGS_DATASOURCE="postgres://mmuser:mostest@localhost:5432/mattermost_test?sslmode=disable"
```

### Enable Debug Logging

```bash
export MM_LOGSETTINGS_CONSOLELEVEL=DEBUG
make run-server
```

## Troubleshooting

### Server Won't Start

**Check logs:**
```bash
tail -f server/mattermost.log
```

**Common issues:**
- Port 8065 already in use
- Database connection failed
- Missing configuration file

### Webapp Build Errors

**Clear cache and rebuild:**
```bash
cd webapp/channels
rm -rf node_modules dist
npm install
npm run build
```

### Database Issues

**Reset database:**
```bash
cd server
make stop-docker
make clean-docker
make start-docker
```

**Warning:** This deletes all data!

### Plugin Errors

**Check plugin logs:**
```bash
tail -f server/logs/mattermost.log | grep plugin
```

**Disable problematic plugin:**
```bash
bin/mmctl plugin disable plugin-id --local
```

## Next Steps

- Review [api-reference.md](./api-reference.md) for API integration
- See [architecture.md](./architecture.md) for architectural patterns
- Check [CONTRIBUTING.md](../CONTRIBUTING.md) for contribution guidelines

## Additional Resources

- [Developer Documentation](https://developers.mattermost.com/)
- [API Documentation](https://api.mattermost.com/)
- [Plugin Developer Guide](https://developers.mattermost.com/integrate/plugins/)
- [Community Server](https://community.mattermost.com/)
