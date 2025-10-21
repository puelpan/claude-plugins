# Claude Code Plugin Marketplace

A curated collection of Claude Code plugins designed to streamline development
workflows with enhanced Git, GitHub, and project management capabilities.

## Available Plugins

### Development Plugin

Provides slash commands to ease the development process with GitHub integration:

- `/create-plan` - Generate structured development plans
- `/execute-plan` - Execute planned development tasks
- `/commit` - Smart commit creation with context
- `/create-pr` - Create pull requests with AI-generated descriptions
- `/pr-review` - Review pull requests with detailed analysis
- `/five-whys` - Root cause analysis using the Five Whys technique

### Backend Plugin

Comprehensive toolkit for Python FastAPI backend development with modern tooling:

- `/new-endpoint` - Create new FastAPI endpoints with proper structure and best practices
- `/setup-project` - Initialize a FastAPI project with uv and workspace configuration
- `/setup-tests` - Set up complete testing infrastructure with pytest
- `/add-migration` - Create and manage database migrations with Alembic
- `/add-docs` - Generate comprehensive API documentation with OpenAPI/Swagger

## Installation

### Add Marketplace

```bash
/plugin marketplace add puelpan/claude-plugins
```

### Browse Available Plugins

```bash
/plugin marketplace list
```

### Install a Plugin

```bash
/plugin install development
/plugin install backend
```

## Usage

Once installed, plugins provide custom slash commands that can be invoked
directly in Claude Code:

### Development Plugin Commands

```bash
/create-plan    # Start planning your feature
/commit         # Create a contextual commit
/create-pr      # Generate and create a PR
/five-whys      # Perform root cause analysis
```

### Backend Plugin Commands

```bash
# Initialize a new FastAPI project
/setup-project my-api-project

# Create a new API endpoint
/new-endpoint POST /users - create a new user

# Set up testing infrastructure
/setup-tests

# Create a database migration
/add-migration create users table

# Generate API documentation
/add-docs
```

## Contributing

This is a personal marketplace, but feel free to fork and create your own plugin
collections.
