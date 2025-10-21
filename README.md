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

React page creation with TanStack Router, TanStack Form, and shadcn/ui:

- `create-page` skill - Expert guidance for creating pages with TanStack Router, TanStack Form, and shadcn/ui components

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

### React Frontend Skills

To use the `create-page` skill, invoke it in your conversation with Claude Code. The skill provides expert guidance for creating React pages using TanStack Router, TanStack Form, and shadcn/ui components.

## Contributing

This is a personal marketplace, but feel free to fork and create your own plugin
collections.
