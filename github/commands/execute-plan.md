# Excecute plan

Execute a plan from GitHub issue or comment: $ARGUMENTS

## Instructions

You are a senior software engineer tasked with implementing a plan based on a
GitHub issue. Please:

- Fetch the GitHub issue using the issue number or URL provided in the arguments
- Parse the plan from the issue description or comment.
- Create an execution strategy with concrete implementation steps
- Generate concrete code and implementation steps
- Create a new branch with a brief name using the conventional commit style as
  prefix (`feat/<branch-name>`, `fix/<branch-name>`, `docs/<branch-name>`, etc)

## On Completion

- Analyze changes and split them into logical commits when appropriate
- Commit and push your changes in a logical order when appropriate
- Each commit focuses on a single logical change or feature
- Creates descriptive commit messages for each logical unit
- Ensure each commit can be understood independently following the rules at
  @./commit.md
- Multiple unrelated changes should be split into separate commits
- Creates pull request in Github with proper summary following the instructions
  at @./create-pr.md
- Do NOT add Claude co-authorship to PR description or commit messages
- Use Github CLI if necessary
- Link the issue and the PR
- Update the issue tasks
