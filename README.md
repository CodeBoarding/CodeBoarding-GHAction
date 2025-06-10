# CodeBoarding GitHub Action

This repository contains a GitHub Action that automatically fetches documentation files from a codeboarding service and creates pull requests with the updated content.

## How it works

1. The action gets the current repository URL
2. Calls your endpoint with the repository URL as a query parameter
3. Receives a JSON response with files and their content
4. Creates/updates files in the `.codeboarding` directory with `.md` extension
5. Creates a pull request with the changes (if any)

## Setup

### 1. Configure the endpoint URL

You have two options to configure the endpoint URL:

**Option A: Using repository variables (recommended)**
1. Go to your repository Settings > Secrets and variables > Actions
2. In the "Variables" tab, create a new variable:
   - Name: `CODEBOARDING_ENDPOINT`
   - Value: `https://your-api-endpoint.com/docs` (replace with your actual endpoint)

**Option B: Edit the workflow file**
Replace the line in `.github/workflows/doc-update.yml`:
```yaml
ENDPOINT_URL="${{ vars.CODEBOARDING_ENDPOINT || 'https://your-api-endpoint.com/docs' }}"
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

### 3. Permissions

The workflow requires the following permissions (already configured):
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
