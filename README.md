# CodeBoarding GitHub Action

A composite GitHub Action that fetches documentation files from a CodeBoarding service and saves them as markdown files in your repository.

## Features

- 🚀 **Simple Integration**: Easy to use composite action
- 📁 **Flexible Output**: Configurable output directory (default: `.codeboarding`)
- 🔄 **Smart Processing**: Automatically adds `.md` extension if needed
- 📊 **Detailed Outputs**: Provides information about created files and changes
- 🛡️ **Error Handling**: Robust error handling for API calls and JSON parsing

## How it works

1. The action determines the repository URL (current repo or provided)
2. Calls your CodeBoarding endpoint with the repository URL as a query parameter
3. Receives a JSON response with files and their content
4. Creates/updates files in the specified output directory with `.md` extension
5. Provides outputs that can be used by subsequent workflow steps

## Usage

### Basic Usage

```yaml
- name: Fetch CodeBoarding Documentation
  uses: your-username/codeboarding-action@v1
  with:
    endpoint_url: 'https://your-api-endpoint.com/generate_docs'
```

### Advanced Usage

```yaml
- name: Fetch CodeBoarding Documentation
  id: codeboarding
  uses: your-username/codeboarding-action@v1
  with:
    endpoint_url: ${{ vars.CODEBOARDING_ENDPOINT }}
    repository_url: 'https://github.com/owner/repo'  # Optional: defaults to current repo
    output_directory: 'docs/generated'              # Optional: defaults to .codeboarding

- name: Check results
  run: |
    echo "Files created: ${{ steps.codeboarding.outputs.files_created }}"
    echo "Has changes: ${{ steps.codeboarding.outputs.has_changes }}"
```

### Complete Workflow with PR Creation

```yaml
name: Update Documentation

on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM
  workflow_dispatch:

jobs:
  update-docs:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Fetch Documentation
        id: docs
        uses: your-username/codeboarding-action@v1
        with:
          endpoint_url: ${{ vars.CODEBOARDING_ENDPOINT }}
      
      - name: Create Pull Request
        if: steps.docs.outputs.has_changes == 'true'
        uses: peter-evans/create-pull-request@v5
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          commit-message: "docs: update codeboarding documentation"
          title: "📚 Documentation Update"
          body: |
            Updated ${{ steps.docs.outputs.files_created }} documentation files.
          branch: docs/update
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `endpoint_url` | URL of the CodeBoarding service endpoint | Yes | - |
| `repository_url` | Repository URL to analyze (defaults to current repo) | No | Current repository |
| `output_directory` | Directory where files will be saved | No | `.codeboarding` |

## Outputs

| Output | Description |
|--------|-------------|
| `files_created` | Number of files created/updated |
| `output_directory` | Directory where files were saved |
| `has_changes` | Whether any files were created or changed (`true`/`false`) |

## Setup

### 1. Configure the endpoint URL

You have several options to configure the endpoint URL:

**Option A: Using repository variables (recommended)**
1. Go to your repository Settings > Secrets and variables > Actions
2. In the "Variables" tab, create a new variable:
   - Name: `CODEBOARDING_ENDPOINT`
   - Value: `https://your-api-endpoint.com/generate_docs` (replace with your actual endpoint)

**Option B: Pass directly in workflow**
```yaml
- uses: your-username/codeboarding-action@v1
  with:
    endpoint_url: 'https://your-api-endpoint.com/generate_docs'
```

### 2. Expected API Response Format

Your endpoint should return a JSON response in this format:

```json
{
  "repository": "owner/repo-name",
  "files": {
    "getting-started": "# Getting Started\n\nThis is the content...",
    "api-reference": "# API Reference\n\nAPI documentation...",
    "troubleshooting": "# Troubleshooting\n\nCommon issues..."
  }
}
```

- `repository`: The repository name (informational)
- `files`: A dictionary where keys are filenames and values are the markdown content

### 2. Expected API Response Format

Your endpoint should return a JSON response in this format:

```json
{
  "repository": "owner/repo-name",
  "files": {
    "getting-started": "# Getting Started\n\nThis is the content...",
    "api-reference": "# API Reference\n\nAPI documentation...",
    "troubleshooting": "# Troubleshooting\n\nCommon issues..."
  }
}
```

- `repository`: The repository name (informational, optional)
- `files`: A dictionary where keys are filenames and values are the markdown content

### 3. Permissions

When using this action in workflows that create PRs or commits, ensure your workflow has the necessary permissions:

```yaml
jobs:
  your-job:
    permissions:
      contents: write        # For committing files
      pull-requests: write   # For creating PRs
```

## Design Philosophy

This composite action follows the **single responsibility principle**:

- ✅ **Does one thing well**: Fetches and saves documentation files
- ✅ **Flexible output**: You decide what to do with the files (commit, PR, etc.)
- ✅ **Composable**: Can be combined with other actions easily
- ✅ **Testable**: Clear inputs and outputs make testing straightforward

## Examples

### Example 1: Save to Custom Directory

```yaml
- uses: your-username/codeboarding-action@v1
  with:
    endpoint_url: ${{ vars.CODEBOARDING_ENDPOINT }}
    output_directory: 'documentation/auto-generated'
```

### Example 2: Analyze Different Repository

```yaml
- uses: your-username/codeboarding-action@v1
  with:
    endpoint_url: ${{ vars.CODEBOARDING_ENDPOINT }}
    repository_url: 'https://github.com/other-org/other-repo'
```

### Example 3: Conditional Processing

```yaml
- name: Generate docs
  id: docs
  uses: your-username/codeboarding-action@v1
  with:
    endpoint_url: ${{ vars.CODEBOARDING_ENDPOINT }}

- name: Only proceed if files were created
  if: steps.docs.outputs.has_changes == 'true'
  run: |
    echo "Processing ${{ steps.docs.outputs.files_created }} new files..."
    # Your custom logic here
```

## Troubleshooting

### Common Issues

1. **API endpoint returns non-200 status**
   - Check that your endpoint URL is correct
   - Verify the endpoint is accessible from GitHub Actions runners
   - Check endpoint logs for errors

2. **Invalid JSON response**
   - Ensure your endpoint returns valid JSON
   - Check that the response includes the expected `files` key

3. **No files created**
   - Verify the API response contains a `files` object with content
   - Check the action logs for debugging information

### Debug Mode

The action includes extensive logging. Check the action logs in your workflow runs for detailed information about:
- API request and response
- JSON parsing results
- File creation process
- `contents: write` - to create commits
- `pull-requests: write` - to create pull requests

## Usage

### Automatic execution
- The action runs daily at 2 AM UTC
- Scheduled runs can be configured by modifying the cron expression in the workflow

### Manual execution
1. Go to Actions tab in your repository
2. Select "Documentation Update" workflow
3. Click "Run workflow"

## File handling

- Files are created in the `.codeboarding` directory
- All files automatically get `.md` extension if not already present
- Existing files are overwritten with new content
- Only creates a PR if there are actual changes

## Pull Request behavior

- PR title: "doc update"
- Branch name: `docs/codeboarding-update`
- Automatically deletes the branch after PR is merged/closed
- Only creates PR if there are changes to commit

## Troubleshooting

### API call fails
- Check that your endpoint URL is correct
- Verify the endpoint is accessible from GitHub Actions runners
- Check the Actions logs for the exact error message

### No PR created
- This means no changes were detected
- The API response might be identical to existing files
- Check the "Check for changes" step in the workflow logs

### Invalid JSON response
- Verify your endpoint returns valid JSON
- Check the response format matches the expected structure
- Review the API response in the workflow logs
