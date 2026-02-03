---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
inputDocuments: ['docs/project-context.md', 'README.md']
workflowType: 'architecture'
project_name: 'mattermost'
user_name: 'BMAD Bot'
date: '2026-02-04'
lastStep: 8
status: 'complete'
completedAt: '2026-02-04'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**
- **Core Messaging:** Users interact via a React-based SPA; Server handles message routing, storage, and retrieval.
- **User Management:** Robust user authentication and permission system (as seen in `server/channels/api4/role.go`).
- **Data Persistence:** Persistent storage of users, posts, channels, and teams in SQL databases (PostgreSQL/MySQL).

**Non-Functional Requirements:**
- **Scalability:** Layered monolith and support for search engines (Elasticsearch) suggest high scalability goals.
- **Database Agnosticism:** Must support multiple SQL dialects (Postgres, MySQL).
- **Maintainability:** Clear separation of API, App, and Store layers.
- **Type Safety:** Strong typing in backend (Go) and frontend (TypeScript).

**Scale & Complexity:**
- **Primary domain:** Full-Stack (Collaboration Platform)
- **Complexity level:** High/Enterprise
- **Estimated architectural components:** 3 (Server, Webapp, API) + Database + Search Engine

### Technical Constraints & Dependencies

- **Language Locking:** Backend strictly Go 1.24+; Frontend strictly TypeScript/React 18.
- **Store Interface:** All database access *must* go through defining interfaces in `server/channels/store`, never raw SQL in logic layers.
- **API Standards:** Modifications must adhere to OpenAPI v4 specs in `server/channels/api4`.

### Cross-Cutting Concerns Identified

- **Permissions & RBAC:** handled centrally (logic in App/Store layers).
- **Real-time Updates:** (Implicit) Handling state synchronization across clients via Redux and backend events.
- **Data Migrations:** Managed via `golang-migrate`.

## Starter Template Evaluation (Brownfield Context)

### Primary Technology Domain

**Full-Stack Collaboration Platform** based on project requirements analysis.

### Starter Options Considered

**N/A - Existing Brownfield Project**
This project is an existing, mature codebase. We are documenting the *current* architectural foundation rather than selecting a new starter template.

### Selected Foundation: Existing Mattermost Codebase

**Rationale for Selection:**
Project is an active brownfield application. The "starter" is the current `master` branch state.

**Architectural Decisions Provided by Current Foundation:**

**Language & Runtime:**
- **Backend:** Go 1.24.11
- **Frontend:** TypeScript / React 18.2.0

**Styling Solution:**
- Hybrid approach: Bootstrap 3.4 legacy + MUI Material 5.11 modern components + Sass for custom overrides.

**Build Tooling:**
- **Backend:** `Makefile` driven build system.
- **Frontend:** Webpack for bundling, Babel for transpilation.

**Testing Framework:**
- **Backend:** Standard Go `testing` package + `testify`.
- **Frontend:** Jest + React Testing Library + Enzyme (legacy).

**Code Organization:**
- **Server:** Layered Monolith (`api4` -> `app` -> `store`).
- **Webapp:** Feature-folder/Type-based organization (`components`, `actions`, `reducers`, `selectors`).

**Development Experience:**
- `make run-server` for backend.
- `npm run run` for webpack dev server.

## Core Architectural Decisions (Brownfield Analysis)

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- **Language Lock:** Server must use Go 1.24.11. Webapp must use TypeScript + React 18.2.0.
- **Database Agnosticism:** Features must work on both PostgreSQL and MySQL.

### Data Architecture

- **Database:** PostgreSQL / MySQL supported via `jmoiron/sqlx`.
- **Search:** Elasticsearch 8.x / OpenSearch supported for full-text caching.
- **Migration Strategy:** Versioned migrations using `github.com/golang-migrate/migrate/v4`.

### Authentication & Security

- **Authentication:** Stateful Sessions + OAuth2 providers.
- **Authorization:** Role-Based Access Control (RBAC) enforced at API layer middleware.
- **Security:** CSRF protection tokens required for state-mutating requests.

### API & Communication Patterns

- **Style:** RESTful API.
- **Specification:** OpenAPI v4 (Swagger).
- **Versioning:** URL-versioned (`/api/v4/...`).
- **Real-time:** WebSocket events for live updates (e.g., typing, new posts).

### Frontend Architecture

- **Framework:** React 18.2.0 (SPA).
- **State Management:** Redux (Global) + Redux Thunk (Async) + Redux Persist (Offline).
- **Styling:** CSS Modules / Sass + Bootstrap / MUI Material.
- **Routing:** React Router v5.

### Infrastructure & Deployment

- **Hosting:** Self-hosted monolithic binary or containerized (Docker).
- **Build Artifacts:** Single Go binary + Static Frontend Assets.

### Decision Impact Analysis

**Implementation Sequence:**
1.  **Strict Layering:** Feature logic goes in `app`, Data access in `store/sqlstore`.
2.  **API First:** Define `api4` routes before implementing frontend consumers.

**Cross-Component Dependencies:**
- **Frontend-Backend Contract:** Strongly coupled via API v4. Changes to API require updates to `mattermost-redux` client logic in webapp.

## Implementation Patterns & Consistency Rules (Brownfield Enforcement)

### Pattern Categories Defined

**Critical Conflict Points:**
AI Agents must adhere to existing Mattermost conventions to avoid codebase fragmentation.

### Naming Patterns

**Database Naming Conventions:**
- **Tables:** Plural, Case-Sensitive (e.g., `Users`, `Channels`).
- **Columns:** PascalCase in DB (often), mapped to struct fields.
- **Go Struct Tags:** MUST use `` `db:"FieldName"` ``.

**API Naming Conventions:**
- **Endpoints:** Plural, lowercase, kebab-case if multiple words (e.g., `/api/v4/users`, `/api/v4/teams/{team_id}/members`).
- **Parameters:** snake_case in URLs and JSON bodies (e.g., `user_id`, `team_id`).

**Code Naming Conventions:**
- **Go:** PascalCase for exported symbols, camelCase for internal.
- **TypeScript:** PascalCase for Components, camelCase for functions/vars.
- **Redux Actions:** SCREAMING_SNAKE_CASE for types (e.g., `RECEIVED_NEW_POST`).

### Structure Patterns

**Layered Monolith Rules (Server):**
1.  **API Layer (`api4/`)**: Only HTTP handling (auth check, parsing params, sending response). Calls App layer.
2.  **App Layer (`app/`)**: Business logic, permission logic, webhooks, validation. Calls Store layer.
3.  **Store Layer (`store/`)**: Interface definitions.
4.  **SqlStore (`store/sqlstore/`)**: Raw SQL implementation.

**Component Organization (Webapp):**
- **Atomic Design:** False. Components are organized by feature (`components/admin_console`) or type (`components/common`).
- **Styles:** SASS files co-located or in `sass/` directory.

### Format Patterns

**API Response Formats:**
- **Success:** Direct JSON object or list.
- **Error:** MUST return `model.AppError` JSON structure (`id`, `message`, `detailed_error`, `status_code`).

**Data Exchange Formats:**
- **JSON:** snake_case keys (e.g., `{"first_name": "Bob"}`).
- **Date/Time:** Unix timestamps (milliseconds) (`CreateAt: int64`).

### Process Patterns

**Error Handling:**
- **Backend:** `func (a *App) DoSomething() (*Result, *model.AppError)`. Never return native `error` for API-facing functions.
- **Frontend:** Redux store handles errors in `requests` slice.

**State Management:**
- **Global:** User/Channel/Team data in Redux implementation (`mattermost-redux`).
- **Local:** UI state (modals, form inputs) in React Component state.

### Enforcement Guidelines

**All AI Agents MUST:**
1.  **Check Context:** Before modifying `store` interfaces, verify strict adherence to `sqlx` patterns.
2.  **Follow Layering:** Never import `store` directly in `api4`. Always go through `app`.
3.  **Use AppError:** Never return standard Go errors to the client.

## Project Structure & Boundaries (Brownfield Map)

### Complete Project Directory Structure

```text
mattermost/
├── api/                    # OpenAPI Specifications
│   └── v4/                 # V4 API definitions
├── server/                 # Go Backend
│   ├── channels/
│   │   ├── api4/           # HTTP Handlers (REST Layer)
│   │   │   ├── user.go     # User-related endpoints
│   │   │   └── team.go     # Team-related endpoints
│   │   ├── app/            # Business Logic Layer
│   │   │   ├── server.go   # Server struct & initialization
│   │   │   └── authentication.go # Auth logic
│   │   ├── model/          # Data Structs & Validation
│   │   │   ├── user.go
│   │   │   └── client4.go
│   │   └── store/          # Data Access Layer
│   │       ├── store.go    # Interface definitions
│   │       └── sqlstore/   # SQL implementation
│   ├── go.mod              # Backend dependencies
│   └── Makefile            # Build scripts
└── webapp/                 # React Frontend
    └── channels/
        ├── src/
        │   ├── actions/    # Redux Actions
        │   ├── components/ # React Components (Feature-based)
        │   ├── reducers/   # Redux Reducers
        │   ├── selectors/  # Redux Selectors
        │   ├── store/      # Store configuration
        │   ├── utils/      # Helper functions
        │   └── index.tsx   # Entry point
        └── package.json    # Frontend dependencies
```

### Architectural Boundaries

**API Boundaries:**
- **External:** All external access via `server/channels/api4`.
- **Internal:** `server/channels/app` acts as the service boundary. API handlers MUST call `app` methods, never `store` directly.

**Component Boundaries:**
- **Frontend-Backend:** API v4 over HTTP/WebSocket.
- **Redux-React:** Functional Logic in Actions/Reducers; View Logic in Components. Connect via `useSelector`/`useDispatch` (or legacy `connect`).

**Data Boundaries:**
- **Database:** Accessed ONLY via `server/channels/store` interfaces.
- **Models:** `server/channels/model` defines the "wire" format and DB struct format.

### Requirements to Structure Mapping

**Feature/Epic Mapping:**
- **User Management**
    - API: `server/channels/api4/user.go`
    - Logic: `server/channels/app/user.go`
    - DB: `server/channels/store/sqlstore/user_store.go`
    - Frontend: `webapp/channels/src/components/user_settings/`

**Cross-Cutting Concerns:**
- **Authentication:** `server/channels/api4/context.go` (middleware) & `server/channels/app/authentication.go`.

### Integration Points

**Internal Communication:**
- **Server:** Direct function calls (`app -> store`).
- **Webapp:** Redux Actions dispatch -> Thunk (API Call) -> Reducer update -> Component mutation.

## Architecture Validation Results

### Coherence Validation ✅

**Decision Compatibility:**
- **Go + React:** Standard industry pattern, fully compatible via REST/JSON.
- **Redux:** Fits the complex state requirements of a messaging app perfectly.
- **SQL Stores:** Abstraction layers in `store/sqlstore` correctly handle the dual-database requirement.

**Pattern Consistency:**
- Naming conventions mapped to existing code ensure new code blends in seamlessly.
- Layering rules (`api` -> `app` -> `store`) prevent circular dependencies and maintain clean separation.

### Requirements Coverage Validation ✅

**Core Functionality:**
- **Messaging:** Covered by API/WebSocket architecture.
- **Persistence:** Covered by `sqlstore` patterns.
- **Real-time:** Implicitly covered by WebSocket architecture.

**Non-Functional Requirements:**
- **Scalability:** Layered monolith key to horizontal scaling.
- **Maintainability:** Strict folder structure and naming rules enforce order.

### Implementation Readiness Validation ✅

**Decision Completeness:**
- Technology versions verified (Go 1.24+, React 18).
- Critical implementation paths are clear.

**Structure Completeness:**
- Full directory tree mapped.
- Separation of concerns clearly delineated.

### Architecture Completeness Checklist

**✅ Requirements Analysis**
- [x] Project context thoroughly analyzed (Brownfield)
- [x] Scale and complexity assessed (High/Enterprise)

**✅ Architectural Decisions**
- [x] Critical decisions documented (Go/React/SQL)
- [x] Integration patterns defined (Layered Monolith)

**✅ Implementation Patterns**
- [x] Naming conventions established (Snake/Pascal/Camel case rules)
- [x] Structure patterns defined (Feature-based vs Layer-based)

**✅ Project Structure**
- [x] Complete directory structure mapped
- [x] Component boundaries established

### Architecture Readiness Assessment

**Overall Status:** READY FOR IMPLEMENTATION

**Confidence Level:** HIGH (Based on proven existing codebase)

**Key Strengths:**
- Mature, well-tested architecture.
- Clear separation of concerns.
- Strong typing on both ends (Go/TS).

### Implementation Handoff

**AI Agent Guidelines:**
- **Strictly follow the Layered Monolith pattern.** Do not bypass the `app` layer.
- **Adhere to the Naming Conventions.** Check existing files if unsure whether to use `user_id` or `UserId`.
- **Use the provided patterns.** Don't invent new error handling or state management approaches.

**First Implementation Priority:**
- Use the Project Context and Architecture Document as the "Source of Truth" for all future code modifications.
