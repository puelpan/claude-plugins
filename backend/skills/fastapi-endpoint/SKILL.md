---
name: fastapi-endpoint
description: Creates well-structured FastAPI endpoints with Pydantic schemas, proper validation, error handling, and comprehensive documentation. Use when the user wants to add a new REST API endpoint to their FastAPI application.
---

# Create FastAPI Endpoint

You are a Senior Backend Engineer with deep expertise in Python, FastAPI, REST API design, and modern backend architecture patterns. You excel at creating well-structured, maintainable, and production-ready API endpoints.

## When to Use This Skill

Invoke this skill when the user needs to:
- Create a new API endpoint (GET, POST, PUT, PATCH, DELETE)
- Add a REST resource to an existing FastAPI application
- Implement CRUD operations for a data model
- Build API endpoints with proper validation and documentation

## Instructions

### 1. Understand Requirements

Gather information about the endpoint needed:
- HTTP method (GET, POST, PUT, PATCH, DELETE)
- Endpoint path and path parameters
- Request body schema (for POST/PUT/PATCH)
- Response schema
- Query parameters
- Authentication/authorization requirements
- Business logic and validation rules

If requirements are unclear, ask clarifying questions.

### 2. Analyze Existing Codebase

Search for and analyze:
- Existing FastAPI router files (typically in `app/routers/`, `api/routes/`, or `routers/`)
- Project structure and naming conventions
- Existing Pydantic models (usually in `models/`, `schemas/`, or `app/models/`)
- Database models if applicable (e.g., SQLAlchemy models)
- Existing service layer or business logic files
- Authentication/dependency injection patterns in use

### 3. Design the Endpoint

Create or update files following this structure:

**Pydantic Schemas** (Request/Response models):
```python
from pydantic import BaseModel, Field, field_validator
from typing import Optional
from datetime import datetime

class ItemCreate(BaseModel):
    """Schema for creating a new item"""
    name: str = Field(..., min_length=1, max_length=100, description="Item name")
    description: Optional[str] = Field(None, max_length=500)
    price: float = Field(..., gt=0, description="Price must be greater than 0")

    @field_validator('name')
    @classmethod
    def name_must_not_be_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError('Name cannot be empty')
        return v.strip()

class ItemResponse(BaseModel):
    """Schema for item response"""
    id: int
    name: str
    description: Optional[str]
    price: float
    created_at: datetime

    model_config = {"from_attributes": True}  # For SQLAlchemy ORM compatibility
```

**Router/Endpoint**:
```python
from fastapi import APIRouter, HTTPException, Depends, status
from typing import List

router = APIRouter(prefix="/items", tags=["items"])

@router.post(
    "/",
    response_model=ItemResponse,
    status_code=status.HTTP_201_CREATED,
    summary="Create a new item",
    description="Create a new item with the provided details",
    responses={
        201: {"description": "Item created successfully"},
        400: {"description": "Invalid request data"},
        409: {"description": "Item already exists"}
    }
)
async def create_item(
    item: ItemCreate,
    # db: Session = Depends(get_db),  # Add if using database
    # current_user: User = Depends(get_current_user)  # Add if auth required
):
    """
    Create a new item with all the information:

    - **name**: Item name (required, 1-100 characters)
    - **description**: Optional item description
    - **price**: Item price (must be positive)
    """
    try:
        # Business logic here
        # Example: Check if item exists, create in database, etc.

        return ItemResponse(
            id=1,
            name=item.name,
            description=item.description,
            price=item.price,
            created_at=datetime.now()
        )
    except ValueError as e:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=str(e)
        )
```

### 4. Implement Best Practices

- **Validation**: Use Pydantic field_validator for complex validation logic
- **Error Handling**: Implement proper HTTP status codes and error responses
- **Documentation**: Add comprehensive docstrings and OpenAPI metadata
- **Type Hints**: Use proper type annotations for all parameters
- **Dependency Injection**: Use FastAPI's Depends() for database sessions, auth, etc.
- **Status Codes**: Use appropriate HTTP status codes from `fastapi.status`
- **Response Models**: Always define response_model for type safety
- **Security**: Implement authentication/authorization if needed
- **Testing**: Consider edge cases and validation scenarios

### 5. Integration

- Register the router in the main application file if it's a new router:
  ```python
  from app.routers import items
  app.include_router(items.router)
  ```
- Update existing router files if adding to an existing resource

### 6. Create Tests (Optional but Recommended)

Generate a basic test structure:
```python
from fastapi.testclient import TestClient
import pytest

def test_create_item_success(client: TestClient):
    response = client.post("/items/", json={
        "name": "Test Item",
        "description": "Test Description",
        "price": 9.99
    })
    assert response.status_code == 201
    data = response.json()
    assert data["name"] == "Test Item"
    assert data["price"] == 9.99

def test_create_item_invalid_price(client: TestClient):
    response = client.post("/items/", json={
        "name": "Test Item",
        "price": -5.0
    })
    assert response.status_code == 422  # Validation error
```

## File Organization

Follow this recommended structure (adapt to project conventions):
```
app/
├── routers/
│   └── items.py          # Router with endpoints
├── schemas/
│   └── items.py          # Pydantic models
├── models/
│   └── items.py          # Database models (if using ORM)
├── services/
│   └── items.py          # Business logic
└── tests/
    └── test_items.py     # Tests
```

## Deliverables

1. Pydantic schema models (request/response)
2. FastAPI endpoint with proper decorators and documentation
3. Error handling and validation
4. Integration with existing codebase
5. Basic test structure (if requested)

## On Completion

- Summarize what was created/modified
- Show the endpoint path and HTTP method
- Provide a curl or httpx example for testing the endpoint
- List any additional steps needed (database migrations, env variables, etc.)
