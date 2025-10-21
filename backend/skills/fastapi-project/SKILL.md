---
name: fastapi-project
description: Initializes a complete FastAPI project with uv package manager, workspace configuration, proper directory structure, and modern Python tooling. Use when the user wants to set up a new FastAPI project or create a monorepo workspace structure.
---

# Setup FastAPI Project with UV and Workspaces

You are a Senior DevOps Engineer and Python expert with deep knowledge of modern Python tooling, project structure, and dependency management using uv. You excel at setting up production-ready Python projects with proper workspace organization.

## When to Use This Skill

Invoke this skill when the user needs to:
- Initialize a new FastAPI project from scratch
- Set up a monorepo with multiple Python packages
- Configure uv workspace for a FastAPI application
- Create a production-ready FastAPI project structure
- Set up development tooling and configuration

## Instructions

### 1. Understand Requirements

Gather information about the project:
- Project name (default: "fastapi-app")
- Workspace structure (monorepo vs single project)
- Additional services/packages needed
- Database requirements (PostgreSQL, MySQL, SQLite, etc.)
- Authentication needs
- Additional features (Celery, Redis, etc.)

If unclear, ask about:
- Do you need a monorepo with multiple services/packages?
- What database will you use?
- Do you need authentication/authorization?
- Do you need background tasks (Celery/Redis)?
- Any specific external APIs or integrations?

### 2. Initialize UV Project

Run the following commands to set up the project:

```bash
# Install uv if not available
# curl -LsSf https://astral.sh/uv/install.sh | sh

# Create new project
uv init $PROJECT_NAME
cd $PROJECT_NAME

# Initialize Python version
uv python install 3.12  # or specified version
uv python pin 3.12
```

### 3. Configure Workspace (if needed)

For monorepo/multi-package projects, create a workspace structure:

**Root pyproject.toml** (workspace configuration):
```toml
[project]
name = "my-fastapi-workspace"
version = "0.1.0"
description = "FastAPI workspace with multiple packages"
requires-python = ">=3.12"

[tool.uv.workspace]
members = [
    "packages/api",
    "packages/core",
    "packages/shared"
]

[tool.uv.sources]
# Local package dependencies
api = { workspace = true }
core = { workspace = true }
shared = { workspace = true }
```

**Workspace Structure**:
```
my-project/
├── pyproject.toml          # Workspace root
├── .python-version         # Python version
├── packages/
│   ├── api/                # Main FastAPI application
│   │   ├── pyproject.toml
│   │   └── src/
│   │       └── api/
│   │           └── main.py
│   ├── core/               # Business logic & services
│   │   ├── pyproject.toml
│   │   └── src/
│   │       └── core/
│   ├── shared/             # Shared utilities
│   │   ├── pyproject.toml
│   │   └── src/
│   │       └── shared/
│   └── workers/            # Background workers (optional)
│       ├── pyproject.toml
│       └── src/
│           └── workers/
├── tests/
└── .gitignore
```

### 4. Create Package Structure

For each package in the workspace, create a pyproject.toml:

**packages/api/pyproject.toml**:
```toml
[project]
name = "api"
version = "0.1.0"
description = "FastAPI main application"
requires-python = ">=3.12"
dependencies = [
    "fastapi>=0.115.0",
    "uvicorn[standard]>=0.30.0",
    "pydantic>=2.9.0",
    "pydantic-settings>=2.5.0",
    "python-multipart>=0.0.9",
    "core",  # Local workspace dependency
    "shared",  # Local workspace dependency
]

[project.optional-dependencies]
dev = [
    "pytest>=8.3.0",
    "pytest-asyncio>=0.24.0",
    "httpx>=0.27.0",
    "pytest-cov>=5.0.0",
    "ruff>=0.6.0",
    "mypy>=1.11.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/api"]
```

**packages/core/pyproject.toml**:
```toml
[project]
name = "core"
version = "0.1.0"
description = "Core business logic and services"
requires-python = ">=3.12"
dependencies = [
    "sqlalchemy>=2.0.0",
    "alembic>=1.13.0",
    "pydantic>=2.9.0",
    "shared",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/core"]
```

### 5. Create FastAPI Application Structure

**packages/api/src/api/main.py**:
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from contextlib import asynccontextmanager

# Import routers
# from api.routers import users, items

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    print("Starting up...")
    yield
    # Shutdown
    print("Shutting down...")

app = FastAPI(
    title="My FastAPI Application",
    description="API built with FastAPI and UV",
    version="0.1.0",
    lifespan=lifespan
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Configure appropriately for production
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Health check
@app.get("/health", tags=["health"])
async def health_check():
    return {"status": "healthy"}

# Include routers
# app.include_router(users.router, prefix="/api/v1")
# app.include_router(items.router, prefix="/api/v1")

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

**packages/api/src/api/config.py** (Settings):
```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    # Application
    app_name: str = "FastAPI Application"
    debug: bool = False

    # Database
    database_url: str = "sqlite:///./app.db"

    # Security
    secret_key: str = "change-me-in-production"

    # CORS
    cors_origins: list[str] = ["http://localhost:3000"]

    model_config = SettingsConfigDict(
        env_file=".env",
        case_sensitive=False
    )

settings = Settings()
```

### 6. Directory Structure Template

Create the complete directory structure:
```
packages/api/src/api/
├── __init__.py
├── main.py                 # FastAPI app
├── config.py               # Settings
├── routers/                # API routes
│   ├── __init__.py
│   └── health.py
├── dependencies.py         # Shared dependencies
└── middleware.py           # Custom middleware

packages/core/src/core/
├── __init__.py
├── models/                 # Database models
│   ├── __init__.py
│   └── base.py
├── schemas/                # Pydantic schemas
│   ├── __init__.py
│   └── base.py
├── services/               # Business logic
│   ├── __init__.py
│   └── base.py
├── repositories/           # Data access layer
│   ├── __init__.py
│   └── base.py
└── database.py             # Database connection

packages/shared/src/shared/
├── __init__.py
├── utils/
│   ├── __init__.py
│   └── helpers.py
└── exceptions.py           # Custom exceptions
```

### 7. Install Dependencies

```bash
# From workspace root
uv sync --all-packages

# Or install specific package with dev dependencies
uv sync --package api --extra dev
```

### 8. Create Configuration Files

**.env**:
```env
APP_NAME="My FastAPI App"
DEBUG=True
DATABASE_URL="sqlite:///./app.db"
SECRET_KEY="your-secret-key-here"
```

**.gitignore**:
```
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
.venv/
ENV/
.uv/

# Testing
.pytest_cache/
.coverage
htmlcov/

# IDEs
.vscode/
.idea/
*.swp
*.swo

# Environment
.env
.env.local

# Database
*.db
*.sqlite
*.sqlite3

# UV
.uv/
uv.lock
```

**README.md**:
```markdown
# FastAPI Project

## Setup

1. Install UV:
   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```

2. Install dependencies:
   ```bash
   uv sync --all-packages
   ```

3. Create `.env` file from `.env.example`

4. Run the application:
   ```bash
   uv run --package api uvicorn api.main:app --reload
   ```

## Development

- Run tests: `uv run pytest`
- Format code: `uv run ruff format .`
- Lint: `uv run ruff check .`
- Type check: `uv run mypy .`
```

### 9. Add Development Scripts

Add to root **pyproject.toml**:
```toml
[tool.uv]
dev-dependencies = [
    "ruff>=0.6.0",
    "mypy>=1.11.0",
    "pytest>=8.3.0",
    "pytest-asyncio>=0.24.0",
    "pytest-cov>=5.0.0",
]

[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "B", "ANN"]
ignore = ["ANN101", "ANN102"]

[tool.mypy]
python_version = "3.12"
strict = true
warn_return_any = true
warn_unused_configs = true
```

## Deliverables

Execute the following:

1. Create project structure with all directories
2. Generate all configuration files (pyproject.toml, .env, .gitignore)
3. Create FastAPI application skeleton
4. Set up workspace if multi-package structure requested
5. Initialize uv and install dependencies
6. Create README with setup instructions
7. Verify the setup by running the application

## On Completion

- Provide summary of created structure
- Show commands to run the application
- List next steps (database setup, add endpoints, etc.)
- Provide example curl command to test the health endpoint
