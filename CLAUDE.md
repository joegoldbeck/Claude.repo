# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Full Stack FastAPI Template that combines:
- **Backend**: Python with FastAPI, SQLModel, PostgreSQL
- **Frontend**: React with TypeScript, Chakra UI, TanStack Router
- **Infrastructure**: Docker Compose, Traefik reverse proxy

## Running the Project

### Quick Start with Docker Compose

```bash
# Start all services with file watching (recommended)
docker compose watch
```

This starts all services with the following URLs:
- Frontend: http://localhost:5173
- Backend API: http://localhost:8000
- API Documentation (Swagger): http://localhost:8000/docs
- Database Admin (Adminer): http://localhost:8080
- Traefik Dashboard: http://localhost:8090
- MailCatcher (email testing): http://localhost:1080

**Note**: First startup may take a minute while the backend waits for the database and runs initial setup.

### Default Login Credentials

- **Email**: `admin@example.com`
- **Password**: `changethis`

These are configured in `.env` as:
- `FIRST_SUPERUSER=admin@example.com`
- `FIRST_SUPERUSER_PASSWORD=changethis`

The superuser is automatically created on first startup if it doesn't exist.

### Monitoring Services

```bash
# Check all logs
docker compose logs

# Check specific service logs
docker compose logs backend
docker compose logs frontend
```

### Mixed Local/Docker Development

You can run some services locally while keeping others in Docker:

```bash
# Stop frontend container and run locally
docker compose stop frontend
cd frontend
npm run dev

# OR stop backend container and run locally
docker compose stop backend
cd backend
fastapi dev app/main.py
```

## Key Commands

### Backend Development

```bash
# Navigate to backend directory first
cd backend

# Install dependencies
uv sync

# Run development server (local)
fastapi dev app/main.py

# Run tests
bash ./scripts/test.sh

# Lint code
bash ./scripts/lint.sh

# Format code
bash ./scripts/format.sh
```

### Frontend Development

```bash
# Navigate to frontend directory first
cd frontend

# Use correct Node version
fnm use  # or nvm use

# Install dependencies
npm install

# Run development server (local)
npm run dev

# Build for production
npm run build

# Lint and fix code
npm run lint

# Generate TypeScript client from backend API
npm run generate-client
```

### Docker Operations

```bash
# Start with file watching (recommended for development)
docker compose watch

# Start without file watching
docker compose up -d

# Access backend container
docker compose exec backend bash

# Run backend tests in container
docker compose exec backend bash scripts/tests-start.sh

# Stop specific service
docker compose stop frontend

# Stop everything and clean up
docker compose down -v --remove-orphans
```

### Database Operations

```bash
# Access backend container first
docker compose exec backend bash

# Create new migration
alembic revision --autogenerate -m "Description"

# Apply migrations
alembic upgrade head

# Run pre-start script (migrations + initial setup)
uv run bash scripts/prestart.sh
```

## Architecture & Code Structure

### Backend Architecture (`/backend`)

- **API Routes** (`app/api/routes/`): RESTful endpoints organized by resource
- **Models** (`app/models.py`): SQLModel database models with Pydantic validation
- **CRUD** (`app/crud.py`): Database operations layer
- **Core** (`app/core/`): Security, configuration, database connection
- **Tests** (`app/tests/`): Pytest-based test suite organized by module

Key patterns:
- Dependency injection for database sessions and current user
- JWT-based authentication with refresh tokens
- Row-level security through user ownership checks
- Automatic OpenAPI documentation generation

### Frontend Architecture (`/frontend`)

- **Routes** (`src/routes/`): File-based routing with TanStack Router
- **Components** (`src/components/`): Feature-based component organization
- **Client** (`src/client/`): Auto-generated TypeScript API client
- **Hooks** (`src/hooks/`): Custom React hooks for auth and data fetching

Key patterns:
- TanStack Query for server state management
- Chakra UI for consistent component styling
- Protected routes with authentication checks
- Optimistic UI updates with mutation hooks

### Testing Strategy

- **Backend**: Unit tests for CRUD operations, integration tests for API endpoints
- **Frontend**: E2E tests with Playwright covering critical user flows
- **Test Data**: Consistent test fixtures and factories

### Important Conventions

1. **API Client Generation**: Frontend TypeScript client must be regenerated when backend API changes
2. **Environment Variables**: Never commit `.env` files; use `.env.example` as template
3. **Database Migrations**: Always create Alembic migrations for model changes
4. **Type Safety**: Both backend (Pydantic) and frontend (TypeScript) enforce strict typing
5. **Authentication**: All API endpoints except auth routes require valid JWT token

### Pre-commit Setup

This project uses pre-commit hooks for code quality. Set it up once:

```bash
# Install pre-commit hooks
uv run pre-commit install

# Run manually on all files
uv run pre-commit run --all-files
```

Pre-commit will automatically run before each git commit to ensure code quality.

### Common Development Tasks

To add a new API endpoint:
1. Create route in `backend/app/api/routes/`
2. Add CRUD operations in `backend/app/crud.py` if needed
3. Update models in `backend/app/models.py` if needed
4. Write tests in `backend/app/tests/`
5. Regenerate frontend client: `./scripts/generate-client.sh`
6. Create corresponding frontend components and routes

To modify database schema:
1. Update SQLModel in `backend/app/models.py`
2. Generate migration: `alembic revision --autogenerate -m "Description"`
3. Review and apply migration: `alembic upgrade head`
4. Update related CRUD operations and tests

### Deployment Notes

- Production uses environment-specific `.env` files
- GitHub Actions handle CI/CD for staging and production
- Traefik manages HTTPS and routing in production
- Database backups configured via environment variables