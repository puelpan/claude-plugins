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

Comprehensive toolkit for Python FastAPI backend development with modern tooling. This plugin provides **Skills** that Claude automatically invokes when you're working on backend tasks:

- **fastapi-endpoint** - Creates FastAPI endpoints with Pydantic schemas, validation, and documentation
- **fastapi-project** - Initializes FastAPI projects with uv, workspace configuration, and proper structure
- **pytest-setup** - Sets up complete testing infrastructure with pytest, fixtures, and factories
- **alembic-migration** - Creates and manages database migrations with Alembic
- **api-documentation** - Generates comprehensive API documentation with OpenAPI/Swagger

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

### Development Plugin (Slash Commands)

The development plugin provides slash commands that you invoke directly:

```bash
/create-plan    # Start planning your feature
/commit         # Create a contextual commit
/create-pr      # Generate and create a PR
/five-whys      # Perform root cause analysis
```

### Backend Plugin (Skills)

The backend plugin provides **Skills** that Claude automatically uses when relevant. Simply describe what you want to do, and Claude will invoke the appropriate skill:

**Examples:**
- "Create a new POST endpoint for user registration" → Uses `fastapi-endpoint` skill
- "Set up a new FastAPI project with uv" → Uses `fastapi-project` skill
- "Add pytest testing to my project" → Uses `pytest-setup` skill
- "Create a migration to add an email column" → Uses `alembic-migration` skill
- "Improve my API documentation" → Uses `api-documentation` skill

Skills are model-invoked, meaning Claude decides when to use them based on your request and the skill's description. You don't need to remember specific command names!

## Contributing

This is a personal marketplace, but feel free to fork and create your own plugin
collections.
