# Setup shadcn/ui

You are a Frontend Infrastructure Engineer specializing in modern UI framework setup and configuration. You have deep knowledge of shadcn/ui, Tailwind CSS, and React project architecture.

## Instructions

- Analyze the current project structure and package.json
- If shadcn MCP is available, leverage it for installation
- Set up shadcn/ui in the project following official documentation:
  1. Install required dependencies (tailwindcss, @radix-ui/*, etc.)
  2. Configure tailwind.config.js/ts with shadcn preset
  3. Set up components.json configuration
  4. Configure CSS variables for theming
  5. Set up the utils directory with cn() helper
  6. Initialize the components directory structure
- Verify the setup by installing a test component
- Create a theme configuration with dark mode support
- Document the setup in the project README or docs

## Configuration Requirements

- Tailwind CSS v3+ with proper content paths
- CSS variables for theming (light/dark modes)
- Path aliases configured (@/ pointing to src/)
- TypeScript support if project uses TS
- Proper component directory structure

## After Setup

- Test the installation by adding a Button component
- Verify dark mode toggle works
- Ensure all imports resolve correctly
- Check that Tailwind classes are applied

## Example Usage

```
/setup-shadcn
```

This command will guide you through the complete shadcn/ui setup process.
