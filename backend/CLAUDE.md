# CLAUDE.md - Backend

This file provides specific guidance to Claude Code when working with the FastAPI backend in this directory.

## Backend Overview

This is a FastAPI backend with:
- **Framework**: FastAPI with SQLModel ORM
- **Database**: PostgreSQL
- **Authentication**: JWT with refresh tokens
- **Testing**: Pytest with coverage
- **Package Management**: uv (Python package manager)

## Key Commands

```bash
# Install dependencies
uv sync

# Run development server
fastapi dev app/main.py

# Run tests
bash ./scripts/test.sh

# Run linting
bash ./scripts/lint.sh

# Format code
bash ./scripts/format.sh

# Access database shell
docker compose exec db psql -U postgres -d app
```

## Code Structure

```
backend/
├── app/
│   ├── main.py              # FastAPI application entry point
│   ├── models.py            # SQLModel database models
│   ├── crud.py              # Database operations (Create, Read, Update, Delete)
│   ├── api/
│   │   ├── deps.py          # Dependency injection (auth, DB sessions)
│   │   ├── main.py          # API router aggregation
│   │   └── routes/          # API endpoints by resource
│   │       ├── users.py     # User management endpoints
│   │       ├── items.py     # Item CRUD endpoints
│   │       ├── login.py     # Authentication endpoints
│   │       └── utils.py     # Utility endpoints (health, test email)
│   ├── core/
│   │   ├── config.py        # Settings management with Pydantic
│   │   ├── db.py            # Database connection and session
│   │   └── security.py      # Password hashing, JWT handling
│   └── tests/               # Test suite
│       ├── api/             # API endpoint tests
│       ├── crud/            # CRUD operation tests
│       └── utils/           # Test utilities and fixtures
├── alembic/                 # Database migrations
├── scripts/                 # Utility scripts
└── pyproject.toml          # Project dependencies and configuration
```

## Development Workflow

### Adding a New API Endpoint

1. **Create the model** (if needed) in `app/models.py`:
```python
class NewModel(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    # Add fields here
    owner_id: int | None = Field(default=None, foreign_key="user.id", nullable=False)
```

2. **Add CRUD operations** in `app/crud.py`:
```python
def create_new_model(*, session: Session, new_model_in: NewModelCreate, owner_id: int) -> NewModel:
    db_obj = NewModel.model_validate(new_model_in, update={"owner_id": owner_id})
    session.add(db_obj)
    session.commit()
    session.refresh(db_obj)
    return db_obj
```

3. **Create API route** in `app/api/routes/new_model.py`:
```python
from fastapi import APIRouter, Depends
from app.api.deps import CurrentUser, SessionDep

router = APIRouter()

@router.post("/", response_model=NewModelPublic)
def create_new_model(
    *,
    session: SessionDep,
    current_user: CurrentUser,
    new_model_in: NewModelCreate,
) -> Any:
    """Create new model."""
    return crud.create_new_model(session=session, new_model_in=new_model_in, owner_id=current_user.id)
```

4. **Register router** in `app/api/main.py`:
```python
from app.api.routes import new_model
api_router.include_router(new_model.router, prefix="/new-models", tags=["new-models"])
```

5. **Write tests** in `app/tests/api/test_new_model.py`

6. **Create migration**:
```bash
alembic revision --autogenerate -m "Add new model"
alembic upgrade head
```

### Testing Guidelines

- Always write tests for new endpoints
- Use the provided fixtures in `app/tests/utils/`
- Test both success and error cases
- Check authentication and authorization

Example test:
```python
def test_create_item(
    client: TestClient, superuser_token_headers: dict[str, str]
) -> None:
    data = {"title": "Foo", "description": "Fighters"}
    response = client.post(
        f"{settings.API_V1_STR}/items/",
        headers=superuser_token_headers,
        json=data,
    )
    assert response.status_code == 200
    content = response.json()
    assert content["title"] == data["title"]
```

### Database Operations

- **Never modify the database schema directly** - always use Alembic migrations
- **Use SQLModel relationships** for related data
- **Apply row-level security** - users should only access their own data
- **Use transactions** for operations that modify multiple tables

### Security Best Practices

1. **Always validate user permissions** in endpoints
2. **Use dependency injection** for current user (`CurrentUser`)
3. **Hash passwords** with bcrypt (handled by `security.py`)
4. **Validate JWT tokens** on every request
5. **Use environment variables** for secrets (never hardcode)

### Performance Considerations

- Use pagination for list endpoints (see `app/api/routes/items.py`)
- Implement query optimization with SQLModel's relationship loading
- Use database indexes for frequently queried fields
- Consider caching for read-heavy endpoints

### Error Handling

The API uses consistent error responses:
- `400`: Bad Request (validation errors)
- `401`: Unauthorized (invalid/missing token)
- `403`: Forbidden (insufficient permissions)
- `404`: Not Found
- `422`: Unprocessable Entity (request validation)

Always use HTTPException with clear error messages:
```python
raise HTTPException(status_code=404, detail="Item not found")
```

### Environment Variables

Key environment variables (see `.env.example`):
- `DATABASE_URL`: PostgreSQL connection string
- `SECRET_KEY`: JWT signing key (generate with `openssl rand -hex 32`)
- `FIRST_SUPERUSER`: Initial admin email
- `FIRST_SUPERUSER_PASSWORD`: Initial admin password
- `BACKEND_CORS_ORIGINS`: Allowed CORS origins

### Common Issues & Solutions

1. **Import errors**: Ensure you're using absolute imports from `app`
2. **Migration conflicts**: Review auto-generated migrations before applying
3. **Test failures**: Check if test database needs migration
4. **CORS errors**: Update `BACKEND_CORS_ORIGINS` in `.env`

### Code Style

- Follow PEP 8 (enforced by ruff)
- Use type hints everywhere
- Keep functions small and focused
- Write descriptive docstrings for API endpoints
- Use meaningful variable names

### Debugging Tips

- FastAPI's automatic `/docs` endpoint for testing
- Use `print()` or `logging` for debugging (visible in Docker logs)
- SQLModel's `.model_dump()` for inspecting model data
- Check `docker compose logs backend` for errors