# Project Context: Mattermost

## Platform Overview
Mattermost is an open-source, self-hosted collaboration platform. The project follows a layered monolithic architecture for the server and a single-page application (SPA) architecture for the web client. It is designed for high scalability, supporting large organizations with thousands of users.

## Architecture

### Backend (Server)
The server operates as a **Layered Monolith** with clear separation of concerns:
- **API Layer (`server/channels/api4`)**: Handles HTTP requests, authentication, and routing using `gorilla/mux`. It exposes a comprehensive RESTful API.
- **App Layer (`server/channels/app`)**: Contains the core business logic. It acts as the bridge between the API and the Store.
- **Store Layer (`server/channels/store`)**: Defines data access interfaces.
- **SQL Store (`server/channels/store/sqlstore`)**: Implements the Store interfaces for PostgreSQL and MySQL using `jmoiron/sqlx` and `database/sql`.

### Frontend (Webapp)
The web application is a **Component-based SPA** built with React and TypeScript:
- **State Management**: Uses **Redux** for global state management, with `redux-thunk` for asynchronous actions and `redux-persist` for local storage persistence.
- **Routing**: Uses `react-router-dom` (v5) for client-side navigation.
- **Styling**: Mix of **Bootstrap 3.4**, **MUI Material**, and custom **Sass** styles.
- **Build System**: Uses **Webpack** for bundling and asset management.

### API
- The API is defined using **OpenAPI v4** specifications allowing for consistent client generation and documentation.

## Technology Stack

| Component | Technology | Key Libraries/Tools |
|-----------|------------|---------------------|
| **Server** | Go 1.24+ | `gorilla/mux` (Routing), `sqlx` (DB), `golang-migrate` (Migrations), `elastic/go-elasticsearch` (Search) |
| **Database** | SQL | PostgreSQL, MySQL |
| **Webapp** | TypeScript, React 18 | `redux`, `react-redux`, `bootstrap`, `@mui/material`, `webpack`, `jest` |
| **API** | REST / OpenAPI | Swagger/Redoc |

## Key Directories & Files

### Server (`server/`)
- `channels/api4/`: API route definitions and handlers.
- `channels/app/`: Core application logic (e.g., `server.go`, service layers).
- `channels/store/`: `Store` interface definitions.
- `channels/store/sqlstore/`: SQL implementations of stores (e.g., `user_store.go`, `post_store.go`).
- `channels/model/`: Data models and struct definitions.

### Webapp (`webapp/`)
- `channels/src/components/`: React UI components.
- `channels/src/stores/`: Redux store configuration (`redux_store.tsx`).
- `channels/src/actions/`: Redux action creators.
- `channels/src/reducers/`: Redux reducers (combined in `reducers/index.ts`).
- `channels/src/selectors/`: Redux state selectors.
- `channels/src/utils/`: Utility functions.
- `channels/package.json`: Frontend dependencies and scripts.

### API (`api/`)
- `v4/`: OpenAPI specifications for the V4 API.

## Implementation Patterns & Rules

### Database Access
- **Never** write raw SQL in the API or App layers.
- All database interactions must go through the **Store Interfaces** defined in `server/channels/store`.
- Use the `SqlStore` implementation for all SQL queries (`server/channels/store/sqlstore`).
- Support both **PostgreSQL** (primary) and **MySQL**. Use compatible SQL syntax or specific adapters where necessary.

### API Development
- New endpoints should be versioned (e.g., API v4) and placed in `server/channels/api4`.
- Endpoints must check for permissions using the `Session` and `Context`.
- Use `AppError` for standardized error reporting.

### Frontend Development
- **Redux**: Use for global state (users, channels, posts).
- **Components**: Prefer functional components with Hooks.
- **Styling**: Use existing utility classes from Bootstrap or defined CSS variables where possible to maintain consistency.
- **Testing**: Write tests using `jest` and React Testing Library.

## Build & Run
- **Server**: `make run-server` (or `go run ./cmd/mattermost`).
- **Webapp**: `npm run run` (starts Webpack dev server).
