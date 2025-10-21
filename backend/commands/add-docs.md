# Setup API Documentation

You are a Senior Technical Writer and API Developer with deep expertise in API documentation, OpenAPI/Swagger specifications, and FastAPI documentation features. You excel at creating comprehensive, user-friendly API documentation.

## Instructions

Enhance or set up API documentation for a FastAPI application based on $ARGUMENTS.

### 1. Understand Requirements

Parse requirements from $ARGUMENTS:
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
    """
    Schema for creating a new user.

    Attributes:
        email: User's email address (must be unique)
        username: User's username (3-50 characters)
        password: User's password (minimum 8 characters)
    """
    email: str
    username: str
    password: str

    class Config:
        json_schema_extra = {
            "example": {
                "email": "user@example.com",
                "username": "johndoe",
                "password": "securePassword123"
            }
        }


class UserResponse(BaseModel):
    """User response model (excludes sensitive data)."""
    id: int
    email: str
    username: str
    is_active: bool

    class Config:
        json_schema_extra = {
            "example": {
                "id": 1,
                "email": "user@example.com",
                "username": "johndoe",
                "is_active": True
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
    # Implementation here
    pass


@router.get(
    "/{user_id}",
    response_model=UserResponse,
    summary="Get user by ID",
    description="Retrieve a specific user by their unique identifier",
    responses={
        200: {"description": "User found"},
        404: {"description": "User not found"},
    }
)
async def get_user(
    user_id: int = Path(..., description="The unique identifier of the user", ge=1)
):
    """
    Retrieve a user by ID.

    **Path Parameters:**
    - **user_id**: User's unique identifier (must be positive integer)

    **Returns:**
    - User object if found
    - 404 error if user doesn't exist
    """
    pass


@router.get(
    "/",
    response_model=List[UserResponse],
    summary="List all users",
    description="Retrieve a paginated list of all users",
)
async def list_users(
    skip: int = Query(0, ge=0, description="Number of records to skip"),
    limit: int = Query(100, ge=1, le=1000, description="Maximum number of records to return"),
    active_only: Optional[bool] = Query(None, description="Filter by active status")
):
    """
    List all users with pagination.

    **Query Parameters:**
    - **skip**: Number of records to skip (default: 0)
    - **limit**: Maximum records to return (default: 100, max: 1000)
    - **active_only**: Filter by active users only

    **Example Request:**
    ```
    GET /api/v1/users?skip=0&limit=10&active_only=true
    ```

    **Returns:**
    List of user objects matching the criteria
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

**Add request/response examples in Pydantic models**:
```python
from pydantic import BaseModel, Field
from typing import Optional
from datetime import datetime


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

    class Config:
        json_schema_extra = {
            "examples": [
                {
                    "name": "MacBook Pro",
                    "description": "16-inch laptop with M3 chip",
                    "price": 2499.99
                },
                {
                    "name": "Coffee Mug",
                    "description": "Ceramic mug with company logo",
                    "price": 12.99
                }
            ]
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

### 8. Add API Documentation Page

**Create custom documentation page** (optional):
```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles
from fastapi.responses import HTMLResponse

app = FastAPI()

# Mount static files for custom docs
app.mount("/static", StaticFiles(directory="static"), name="static")


@app.get("/api-guide", response_class=HTMLResponse)
async def api_guide():
    """Serve custom API documentation page."""
    return """
    <!DOCTYPE html>
    <html>
    <head>
        <title>API Documentation</title>
        <link rel="stylesheet" href="/static/docs.css">
    </head>
    <body>
        <h1>API Documentation</h1>
        <p>Welcome to our API documentation.</p>
        <ul>
            <li><a href="/docs">Interactive API Docs (Swagger UI)</a></li>
            <li><a href="/redoc">Alternative Docs (ReDoc)</a></li>
            <li><a href="/openapi.json">OpenAPI Specification</a></li>
        </ul>
    </body>
    </html>
    """
```

### 9. Generate Markdown Documentation

Install and use tools to generate markdown docs:
```bash
# Install openapi-generator or similar tool
npm install -g @openapitools/openapi-generator-cli

# Generate markdown docs
openapi-generator-cli generate \
  -i openapi.json \
  -g markdown \
  -o docs/api
```

### 10. Add Example Code Snippets

**Create examples directory**:
```
docs/
├── examples/
│   ├── python/
│   │   ├── create_user.py
│   │   └── authenticate.py
│   ├── javascript/
│   │   ├── create_user.js
│   │   └── authenticate.js
│   └── curl/
│       ├── create_user.sh
│       └── authenticate.sh
```

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

### 11. Add API Versioning Documentation

Document API versions clearly:
```python
from fastapi import APIRouter

# V1 API
router_v1 = APIRouter(prefix="/api/v1")

# V2 API with changes
router_v2 = APIRouter(prefix="/api/v2")

app.include_router(router_v1, tags=["v1"])
app.include_router(router_v2, tags=["v2"])
```

### 12. Create README for API

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
