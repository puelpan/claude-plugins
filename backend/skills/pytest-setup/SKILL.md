---
name: pytest-setup
description: Sets up comprehensive pytest testing infrastructure for FastAPI applications including fixtures, factories, test database configuration, and CI/CD integration. Use when the user needs to add testing capabilities to their FastAPI project.
---

# Setup Test Environment for FastAPI

You are a Senior QA Engineer and Python testing expert with deep knowledge of pytest, FastAPI testing patterns, test-driven development, and CI/CD best practices. You excel at setting up comprehensive test environments for backend applications.

## When to Use This Skill

Invoke this skill when the user needs to:
- Set up testing infrastructure for a new or existing FastAPI project
- Add pytest configuration and test fixtures
- Configure test database and factories
- Set up test coverage reporting
- Integrate tests with CI/CD pipelines
- Create test directory structure

## Instructions

### 1. Understand Requirements

Gather information about testing needs:
- Test types needed (unit, integration, e2e)
- Database testing strategy (test DB, fixtures, factories)
- Authentication testing needs
- Coverage requirements
- CI/CD integration (GitHub Actions, GitLab CI, etc.)

If unclear, ask about:
- What types of tests do you need? (unit/integration/e2e)
- Do you need test database fixtures?
- Are you using any external services that need mocking?
- What's your target code coverage percentage?
- Do you need CI/CD integration?

### 2. Analyze Existing Project

Search for:
- Existing test directory structure
- Current testing dependencies in pyproject.toml
- FastAPI application structure and dependencies
- Database models and configuration
- Authentication/authorization setup

### 3. Install Testing Dependencies

Add to **pyproject.toml** (or the appropriate package):
```toml
[project.optional-dependencies]
test = [
    "pytest>=8.3.0",
    "pytest-asyncio>=0.24.0",
    "pytest-cov>=5.0.0",
    "pytest-mock>=3.14.0",
    "httpx>=0.27.0",  # For TestClient
    "faker>=28.0.0",  # For generating test data
    "factory-boy>=3.3.0",  # For test factories
    "pytest-xdist>=3.6.0",  # For parallel test execution
]
```

Install with:
```bash
uv sync --extra test
```

### 4. Create Test Configuration

Add to **pyproject.toml**:
```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
asyncio_mode = "auto"
addopts = [
    "-v",
    "--strict-markers",
    "--cov=app",
    "--cov-report=html",
    "--cov-report=term-missing",
    "--cov-fail-under=80"
]
markers = [
    "unit: Unit tests",
    "integration: Integration tests",
    "e2e: End-to-end tests",
    "slow: Slow running tests"
]

[tool.coverage.run]
source = ["src"]
omit = [
    "*/tests/*",
    "*/__pycache__/*",
    "*/venv/*",
    "*/.venv/*"
]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise AssertionError",
    "raise NotImplementedError",
    "if __name__ == .__main__.:",
    "if TYPE_CHECKING:",
    "@abstractmethod"
]
```

### 5. Create Test Directory Structure

```
tests/
├── __init__.py
├── conftest.py              # Shared fixtures
├── test_config.py           # Test configuration
├── unit/                    # Unit tests
│   ├── __init__.py
│   ├── test_models.py
│   ├── test_schemas.py
│   └── test_services.py
├── integration/             # Integration tests
│   ├── __init__.py
│   ├── test_api.py
│   └── test_database.py
├── e2e/                     # End-to-end tests
│   ├── __init__.py
│   └── test_workflows.py
└── factories/               # Test data factories
    ├── __init__.py
    └── user_factory.py
```

### 6. Create Core Test Fixtures

**tests/conftest.py**:
```python
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy.pool import StaticPool

from api.main import app
from core.database import Base, get_db

# Test database setup
SQLALCHEMY_DATABASE_URL = "sqlite:///:memory:"

engine = create_engine(
    SQLALCHEMY_DATABASE_URL,
    connect_args={"check_same_thread": False},
    poolclass=StaticPool,
)
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)


@pytest.fixture(scope="function")
def db_session():
    """Create a fresh database session for each test."""
    Base.metadata.create_all(bind=engine)
    session = TestingSessionLocal()
    try:
        yield session
    finally:
        session.close()
        Base.metadata.drop_all(bind=engine)


@pytest.fixture(scope="function")
def client(db_session):
    """Create a test client with database session override."""
    def override_get_db():
        try:
            yield db_session
        finally:
            db_session.close()

    app.dependency_overrides[get_db] = override_get_db
    with TestClient(app) as test_client:
        yield test_client
    app.dependency_overrides.clear()


@pytest.fixture
def auth_headers():
    """Return authentication headers for testing protected endpoints."""
    # Modify based on your auth implementation
    return {"Authorization": "Bearer test-token"}


@pytest.fixture(scope="session")
def faker_seed():
    """Set seed for Faker to ensure reproducible test data."""
    return 12345
```

### 7. Create Test Data Factories

**tests/factories/user_factory.py**:
```python
import factory
from factory import fuzzy
from faker import Faker

from core.models.user import User

fake = Faker()


class UserFactory(factory.Factory):
    """Factory for creating test users."""

    class Meta:
        model = User

    id = factory.Sequence(lambda n: n)
    email = factory.LazyAttribute(lambda _: fake.email())
    username = factory.LazyAttribute(lambda _: fake.user_name())
    first_name = factory.LazyAttribute(lambda _: fake.first_name())
    last_name = factory.LazyAttribute(lambda _: fake.last_name())
    is_active = True
    is_superuser = False


class SuperUserFactory(UserFactory):
    """Factory for creating test superusers."""

    is_superuser = True
```

### 8. Create Example Tests

**tests/unit/test_schemas.py**:
```python
import pytest
from pydantic import ValidationError

from core.schemas.user import UserCreate, UserResponse


class TestUserSchemas:
    """Test user Pydantic schemas."""

    def test_user_create_valid(self):
        """Test valid user creation schema."""
        user_data = {
            "email": "test@example.com",
            "username": "testuser",
            "password": "securePassword123"
        }
        user = UserCreate(**user_data)
        assert user.email == "test@example.com"
        assert user.username == "testuser"

    def test_user_create_invalid_email(self):
        """Test user creation with invalid email."""
        with pytest.raises(ValidationError):
            UserCreate(
                email="invalid-email",
                username="testuser",
                password="password123"
            )
```

**tests/integration/test_api.py**:
```python
import pytest
from fastapi import status


class TestUserAPI:
    """Test user API endpoints."""

    def test_create_user_success(self, client):
        """Test successful user creation."""
        response = client.post("/api/v1/users/", json={
            "email": "newuser@example.com",
            "username": "newuser",
            "password": "securePassword123"
        })
        assert response.status_code == status.HTTP_201_CREATED
        data = response.json()
        assert data["email"] == "newuser@example.com"
        assert data["username"] == "newuser"
        assert "password" not in data

    def test_get_user_not_found(self, client):
        """Test getting non-existent user."""
        response = client.get("/api/v1/users/99999")
        assert response.status_code == status.HTTP_404_NOT_FOUND

    def test_protected_endpoint_with_auth(self, client, auth_headers):
        """Test protected endpoint with valid authentication."""
        response = client.get("/api/v1/users/me", headers=auth_headers)
        assert response.status_code == status.HTTP_200_OK
```

### 9. CI/CD Integration

**GitHub Actions (.github/workflows/test.yml)**:
```yaml
name: Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.11", "3.12"]

    steps:
    - uses: actions/checkout@v4

    - name: Install uv
      uses: astral-sh/setup-uv@v3
      with:
        enable-cache: true

    - name: Set up Python ${{ matrix.python-version }}
      run: uv python install ${{ matrix.python-version }}

    - name: Install dependencies
      run: uv sync --all-extras

    - name: Run tests
      run: uv run pytest --cov --cov-report=xml

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v4
      with:
        file: ./coverage.xml
        fail_ci_if_error: true
```

### 10. Run Tests

Create convenience scripts in **pyproject.toml**:
```toml
[tool.uv]
scripts = {
    test = "pytest",
    test-unit = "pytest tests/unit -m unit",
    test-integration = "pytest tests/integration -m integration",
    test-cov = "pytest --cov --cov-report=html --cov-report=term",
}
```

Run tests:
```bash
# All tests
uv run pytest

# Unit tests only
uv run pytest tests/unit -m unit

# With coverage
uv run pytest --cov --cov-report=html

# Parallel execution
uv run pytest -n auto
```

## Deliverables

Execute the following:

1. Add testing dependencies to pyproject.toml
2. Create pytest configuration
3. Set up test directory structure
4. Create conftest.py with essential fixtures
5. Create test data factories
6. Generate example tests (unit, integration)
7. Set up CI/CD workflow (if requested)
8. Install dependencies and run initial test

## On Completion

- Summarize the test setup created
- Show commands to run different test suites
- Display test coverage report
- Provide examples of how to add new tests
- List best practices for testing in this project
