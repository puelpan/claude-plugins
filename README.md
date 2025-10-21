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

### React Frontend Plugin

React and frontend development tools with MCP integration for modern UI frameworks:

- `/add-component` - Create new React components with shadcn/ui integration
- `/setup-shadcn` - Set up shadcn/ui in your React project
- `/add-page` - Create new pages with proper routing and SEO
- `/optimize-component` - Optimize React components for performance
- `/add-form` - Create forms with validation (React Hook Form + Zod)
- `/setup-react-project` - Initialize a new React project with best practices

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
/plugin install react-frontend
```

## Usage

Once installed, plugins provide custom slash commands that can be invoked
directly in Claude Code:

### Development Commands
```bash
/create-plan    # Start planning your feature
/commit         # Create a contextual commit
/create-pr      # Generate and create a PR
```

### React Frontend Commands
```bash
/setup-react-project    # Initialize a new React project
/setup-shadcn          # Set up shadcn/ui
/add-component         # Create a new component
/add-page             # Create a new page
/add-form             # Create a form with validation
/optimize-component    # Optimize component performance
```

## Contributing

This is a personal marketplace, but feel free to fork and create your own plugin
collections.
