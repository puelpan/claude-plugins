---
name: api-documentation
description: Enhances FastAPI applications with comprehensive API documentation including OpenAPI/Swagger UI, ReDoc, request/response examples, and authentication documentation. Use when the user needs to improve or set up API documentation.
---

# Setup API Documentation

You are a Senior Technical Writer and API Developer with deep expertise in API documentation, OpenAPI/Swagger specifications, and FastAPI documentation features. You excel at creating comprehensive, user-friendly API documentation.

## When to Use This Skill

Invoke this skill when the user needs to:
- Set up or enhance API documentation for a FastAPI application
- Add OpenAPI/Swagger UI and ReDoc documentation
- Document API endpoints with examples
- Configure authentication documentation
- Export OpenAPI specifications
- Create API usage guides and examples

## Instructions

### 1. Understand Requirements

Gather information about documentation needs:
- Type of documentation needed (OpenAPI/Swagger UI, ReDoc, custom)
- What needs to be documented (specific endpoints, all APIs, authentication)
- Additional documentation (API guides, tutorials, examples)
- Whether to generate external documentation (Markdown, HTML)

If unclear, ask about:
- Do you need interactive API documentation (Swagger UI/ReDoc)?
- Should documentation include authentication examples?
- Do you need to export OpenAPI specification?
- Any specific styling or branding requirements?

### 2. Analyze Existing Application

Search for:
- FastAPI application instance and configuration
- Existing endpoints and routers
- Pydantic models and schemas
- Authentication/authorization setup
- Current documentation settings

### 3. Configure FastAPI Documentation

**Enhanced FastAPI App Configuration**:
```python
from fastapi import FastAPI
from fastapi.openapi.utils import get_openapi

app = FastAPI(
    title="My FastAPI Application",
    description="""
    ## Overview
    This API provides comprehensive backend services for [your application].

    ## Features
    * **User Management**: Create, read, update, and delete users
    * **Authentication**: JWT-based authentication system
    * **Data Processing**: Advanced data processing capabilities

    ## Rate Limiting
    API requests are limited to 100 requests per minute per user.

    ## Support
    For support, email support@example.com or visit our [documentation](https://docs.example.com).
    """,
    version="1.0.0",
    terms_of_service="https://example.com/terms",
    contact={
        "name": "API Support",
        "url": "https://example.com/support",
        "email": "support@example.com",
    },
    license_info={
        "name": "MIT",
        "url": "https://opensource.org/licenses/MIT",
    },
    docs_url="/docs",           # Swagger UI
    redoc_url="/redoc",         # ReDoc
    openapi_url="/openapi.json" # OpenAPI schema
)


def custom_openapi():
    """Customize OpenAPI schema."""
    if app.openapi_schema:
        return app.openapi_schema

    openapi_schema = get_openapi(
        title=app.title,
        version=app.version,
        description=app.description,
        routes=app.routes,
    )

    # Add security scheme
    openapi_schema["components"]["securitySchemes"] = {
        "Bearer": {
            "type": "http",
            "scheme": "bearer",
            "bearerFormat": "JWT",
            "description": "Enter your JWT token in the format: Bearer <token>"
        }
    }

    # Add tags metadata
    openapi_schema["tags"] = [
        {
            "name": "users",
            "description": "Operations with users. Authentication required.",
        },
        {
            "name": "items",
            "description": "Manage items. Public access.",
        },
        {
            "name": "auth",
            "description": "Authentication and authorization endpoints.",
        },
    ]

    app.openapi_schema = openapi_schema
    return app.openapi_schema


app.openapi = custom_openapi
```

### 4. Document Endpoints Comprehensively

**Well-Documented Endpoint Example**:
```python
from fastapi import APIRouter, HTTPException, status, Depends, Query, Path
from typing import List, Optional
from pydantic import BaseModel

router = APIRouter(prefix="/api/v1/users", tags=["users"])


class UserCreate(BaseModel):
    """Schema for creating a new user."""
    email: str
    username: str
    password: str

    model_config = {
        "json_schema_extra": {
            "examples": [{
                "email": "user@example.com",
                "username": "johndoe",
                "password": "securePassword123"
            }]
        }
    }


class UserResponse(BaseModel):
    """User response model (excludes sensitive data)."""
    id: int
    email: str
    username: str
    is_active: bool

    model_config = {
        "json_schema_extra": {
            "examples": [{
                "id": 1,
                "email": "user@example.com",
                "username": "johndoe",
                "is_active": True
            }]
        }
    }


@router.post(
    "/",
    response_model=UserResponse,
    status_code=status.HTTP_201_CREATED,
    summary="Create a new user",
    description="Register a new user in the system with email and username",
    response_description="The created user object",
    responses={
        201: {
            "description": "User successfully created",
            "content": {
                "application/json": {
                    "example": {
                        "id": 1,
                        "email": "user@example.com",
                        "username": "johndoe",
                        "is_active": True
                    }
                }
            },
        },
        400: {
            "description": "Invalid request data",
            "content": {
                "application/json": {
                    "example": {"detail": "Email is invalid"}
                }
            },
        },
        409: {
            "description": "User already exists",
            "content": {
                "application/json": {
                    "example": {"detail": "Email already registered"}
                }
            },
        },
    },
    tags=["users"]
)
async def create_user(user: UserCreate):
    """
    Create a new user account.

    This endpoint allows registration of new users with the following validations:
    - Email must be unique and valid
    - Username must be 3-50 characters
    - Password must be at least 8 characters

    **Example Request:**
    ```json
    {
        "email": "newuser@example.com",
        "username": "newuser",
        "password": "securePass123"
    }
    ```

    **Returns:**
    - **201**: User successfully created
    - **400**: Invalid input data
    - **409**: User with email/username already exists
    """
    pass
```

### 5. Add Authentication Documentation

**Document Security Requirements**:
```python
from fastapi import Security
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

security = HTTPBearer()


@router.get(
    "/me",
    response_model=UserResponse,
    summary="Get current user",
    description="Retrieve the currently authenticated user's information",
    dependencies=[Security(security)]
)
async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Security(security)
):
    """
    Get the current authenticated user.

    **Authentication Required:** This endpoint requires a valid JWT token.

    **Headers:**
    ```
    Authorization: Bearer <your-jwt-token>
    ```

    **Example:**
    ```bash
    curl -X GET "http://api.example.com/api/v1/users/me" \\
         -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
    ```

    **Returns:**
    - Current user's information
    - 401 error if not authenticated
    """
    pass
```

### 6. Create Example Request/Response Documentation

**Add examples in Pydantic models**:
```python
from pydantic import BaseModel, Field
from typing import Optional


class ItemBase(BaseModel):
    """Base schema for items with examples."""

    name: str = Field(
        ...,
        min_length=1,
        max_length=100,
        description="Item name",
        examples=["Laptop", "Coffee Mug", "Notebook"]
    )
    description: Optional[str] = Field(
        None,
        max_length=500,
        description="Detailed item description",
        examples=["High-performance laptop with 16GB RAM"]
    )
    price: float = Field(
        ...,
        gt=0,
        description="Item price in USD",
        examples=[999.99, 15.50, 2500.00]
    )

    model_config = {
        "json_schema_extra": {
            "examples": [
                {
                    "name": "MacBook Pro",
                    "description": "16-inch laptop with M3 chip",
                    "price": 2499.99
                }
            ]
        }
    }
```

### 7. Export OpenAPI Specification

**Create script to export OpenAPI spec**:
```python
# scripts/export_openapi.py
import json
from api.main import app


def export_openapi_spec():
    """Export OpenAPI specification to JSON file."""
    openapi_schema = app.openapi()

    with open("openapi.json", "w") as f:
        json.dump(openapi_schema, f, indent=2)

    print("OpenAPI specification exported to openapi.json")


if __name__ == "__main__":
    export_openapi_spec()
```

Run:
```bash
uv run python scripts/export_openapi.py
```

### 8. Create Example Code Snippets

**docs/examples/python/create_user.py**:
```python
"""
Example: Create a new user using the API

Requirements:
    pip install httpx
"""
import httpx
import asyncio


async def create_user():
    async with httpx.AsyncClient() as client:
        response = await client.post(
            "http://localhost:8000/api/v1/users/",
            json={
                "email": "newuser@example.com",
                "username": "newuser",
                "password": "securePassword123"
            }
        )

        if response.status_code == 201:
            print("User created successfully!")
            print(response.json())
        else:
            print(f"Error: {response.status_code}")
            print(response.json())


if __name__ == "__main__":
    asyncio.run(create_user())
```

**docs/examples/curl/create_user.sh**:
```bash
#!/bin/bash
# Example: Create a new user using curl

curl -X POST "http://localhost:8000/api/v1/users/" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "newuser@example.com",
    "username": "newuser",
    "password": "securePassword123"
  }'
```

### 9. Create API Documentation README

**docs/API_README.md**:
```markdown
# API Documentation

## Base URL
```
http://localhost:8000/api/v1
```

## Authentication
All authenticated endpoints require a JWT token in the Authorization header:
```
Authorization: Bearer <your-jwt-token>
```

## Endpoints

### Users

#### Create User
- **POST** `/users/`
- **Body:**
  ```json
  {
    "email": "user@example.com",
    "username": "johndoe",
    "password": "securePassword123"
  }
  ```
- **Response:** `201 Created`

#### Get User
- **GET** `/users/{user_id}`
- **Response:** `200 OK`

## Error Handling

All errors follow this format:
```json
{
  "detail": "Error message here"
}
```

## Rate Limiting
100 requests per minute per user.

## Interactive Documentation
- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc
```

## Deliverables

Execute the following:

1. Enhance FastAPI app configuration with comprehensive metadata
2. Add detailed documentation to all endpoints
3. Include request/response examples in Pydantic models
4. Configure security schemes in OpenAPI
5. Export OpenAPI specification to JSON
6. Create example code snippets (Python, curl, etc.)
7. Generate API documentation README
8. Test documentation by accessing /docs and /redoc

## On Completion

- Provide links to access documentation (Swagger UI, ReDoc)
- Summarize what was documented
- Show example API calls with curl
- List any additional documentation generated
- Provide instructions for keeping docs updated
