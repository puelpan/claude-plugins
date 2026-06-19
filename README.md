# Claude Code Plugin Marketplace

A curated collection of Claude Code plugins designed to streamline development
workflows with enhanced Git, GitHub, and project management capabilities.

## Available Plugins

### Development Plugin

Eases the development process with GitHub integration through slash commands
and skills.

#### Commands

Explicitly invoked with a slash command:

- `/create-plan` - Generate structured development plans
- `/execute-plan` - Execute planned development tasks
- `/pr-review` - Review pull requests with detailed analysis
- `/five-whys` - Root cause analysis using the Five Whys technique

#### Skills

Invoked explicitly with `/name`, or automatically when your request matches
their purpose:

- `/commit` - Create a conventional-style commit from the current changes
- `/create-pr` - Create a branch, split logical commits, and open a PR
- `/linter` - Run the project linter and fix the reported issues
- `/web-automation` - Browser automation assistant with Chrome DevTools MCP for
  creating and debugging web scrapers
- `/c4-diagrams` - Generate C4 model architecture diagrams as PlantUML source,
  derived from the codebase or a written description

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
```

## Usage

Once installed, plugins provide custom slash commands that can be invoked
directly in Claude Code:

```bash
/create-plan    # Start planning your feature
/commit         # Create a contextual commit
/create-pr      # Generate and create a PR
```

## Contributing

This is a personal marketplace, but feel free to fork and create your own plugin
collections.
