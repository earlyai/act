# Early Catch for Unit Test Generation

A composite GitHub Action that automatically generates unit tests for modified TypeScript files in pull requests using Early Catch CLI.

## Features

- Automatically detects modified TypeScript files in pull requests
- Excludes test files (files ending with `.spec.ts`, `.test.ts`, or in `__tests__/` directories)
- Generates tests using EarlyAI CLI
- Auto-commits generated test files back to the pull request branch
- Configurable branch for commits

## Usage

### Basic Usage

```yaml
name: Generate Tests

on:
  pull_request:
    types: [opened, synchronize]

permissions:
  contents: write
  pull-requests: read

jobs:
  generate-tests:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    container:
      image: ghcr.io/earlyai/early-github-action
    steps:
      - name: Checkout target repo
        uses: actions/checkout@v4
        with:
          ref: ${{ github.head_ref }}
          set-safe-directory: true

      - name: Install dependencies
        run: npm install

      - name: Generate tests with EarlyAI
        uses: earlyai/act@main
        with:
          early-secret-token: ${{ secrets.EARLY_SECRET_TOKEN }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `branch` | Branch to commit generated files to | No | `${{ github.head_ref }}` |
| `early-secret-token` | EarlyAI access token | Yes | - |

## Requirements

- This action must be run in a pull request context
- The repository must have the `EARLY_SECRET_TOKEN` secret configured
- The EarlyAI CLI must be available in the environment (typically via a Docker container or pre-installed)
- GitHub CLI (`gh`) must be available in the runner environment
- `jq` must be available for JSON processing
- **This action needs to run inside `ghcr.io/earlyai/early-github-action` container**
- The workflow must have the following permissions:
  - `contents: write` - to commit generated test files
  - `pull-requests: read` - to read PR information and files
- The GitHub CLI is automatically authenticated using the `GITHUB_TOKEN` environment variable (provided by GitHub Actions)

## Container Usage

When using the public action, it's recommended to use the official EarlyAI container:

```yaml
container:
  image: ghcr.io/earlyai/early-github-action
```

This container includes all the necessary dependencies including:
- EarlyAI CLI
- GitHub CLI (`gh`)
- `jq` for JSON processing
- Node.js and npm

## Prepare Steps

When using the public action, you should include these prepare steps:

1. **Checkout with safe directory**: Ensures the git directory is marked as safe
2. **Install dependencies**: Installs npm dependencies if your project uses them

```yaml
- name: Checkout target repo
  uses: actions/checkout@v4
  with:
    ref: ${{ github.head_ref }}
    set-safe-directory: true

- name: Install dependencies
  run: npm install
```

## Generated Files

The action will generate test files with the pattern `**/*.early.test.ts` and automatically commit them to the specified branch.

## Error Handling

- If no pull request context is found, the action will fail
- The EarlyAI CLI execution continues on error to allow partial test generation
- Generated files are only committed if they exist
- The commit step uses `exit 0` to prevent failures from stopping the workflow

## Limitations

- This is a composite action, so it runs in the same environment as the calling workflow
- The action requires GitHub CLI and `jq` to be available in the runner environment
- Timeout should be configured at the job level using `timeout-minutes`

## Example Workflow

See `example-workflow.yml` for a complete example of how to use this action in a workflow. 
