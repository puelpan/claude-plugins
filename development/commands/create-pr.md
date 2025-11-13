# Create Pull Request

Create a new branch, commit changes, and submit a pull request.

## Behavior

- Creates a new branch based on current changes
- Analyzes changes and automatically splits into logical commits when
  appropriate
- Each commit focuses on a single logical change or feature
- Creates descriptive commit messages for each logical unit
- Pushes branch to remote
- Creates pull request with proper summary and test plan
- Do NOT add Claude co-authorship to PR description or commit messages
- Use Github CLI if necessary

## Guidelines for Automatic Commit Splitting

- Run the linter, formatting and type checks tool for this project before
  commiting your changes.
- Split commits by feature, component, or concern
- Keep related file changes together in the same commit
- Separate refactoring from feature additions
- Ensure each commit can be understood independently
- Multiple unrelated changes should be split into separate commits
- Do NOT add Claude co-authorship to PR description or commit messages
