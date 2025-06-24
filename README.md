<div align="center">
  <img src="assets/icon.svg" alt="CodeBoarding Logo" height="150" />
  
  # CodeBoarding [Diagram-First Documentation]
  
  [![GitHub Action](https://img.shields.io/badge/GitHub-Action-blue?logo=github-actions)](https://github.com/marketplace/actions/codeboarding-diagram-first-documentation)
</div>

Generates diagram-first visualizations of your codebase using static analysis and large language models.

## Usage

```yaml
name: Generate Documentation
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
    types: [opened, synchronize, reopened]

jobs:
  documentation:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Required to access branch history
        
      - name: Generate Documentation
        uses: codeboarding/codeboarding-ghaction@v1
        with:
          repository_url: ${{ github.server_url }}/${{ github.repository }}
          source_branch: ${{ github.head_ref || github.ref_name }}
          target_branch: ${{ github.base_ref || 'main' }}
          output_directory: 'docs'
          output_format: '.md'
          
      - name: Upload Documentation
        uses: actions/upload-artifact@v4
        with:
          name: documentation
          path: |
            docs/
            .codeboarding/
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `repository_url` | Repository URL for which documentation will be generated | Yes | - |
| `source_branch` | Source branch for comparison (typically the PR branch) | Yes | - |
| `target_branch` | Target branch for comparison (typically the base branch) | Yes | - |
| `output_directory` | Directory where documentation files will be saved | No | `docs` |
| `output_format` | Format for documentation files (either `.md` or `.rst`) | No | `.md` |

## Outputs

| Output | Description |
|--------|-------------|
| `markdown_files_created` | Number of documentation files created |
| `json_files_created` | Number of JSON files created |
| `output_directory` | Directory where documentation files were saved |
| `json_directory` | Directory where JSON files were saved (always `.codeboarding`) |
| `has_changes` | Whether any files were created or changed |

## How It Works

The action works by:

1. Analyzing the differences introduced in the source branch and putting the results in the target branch
2. Generating documentation files based on the latest version of the source branch
3. Outputting two types of files:
   - Documentation files (Markdown or RST) in the specified output directory
   - Metadata files in the `.codeboarding` directory

## License

MIT License - see [LICENSE](LICENSE) file for details.

# CodeBoarding GitHub Action

## Important: Timeout Configuration

For large repositories, the analysis can take 15-45 minutes. Make sure to configure appropriate timeouts in your workflow:

```yaml
jobs:
  generate-docs:
    runs-on: ubuntu-latest
    timeout-minutes: 60  # Set to 60+ minutes for large repositories
    steps:
      - uses: actions/checkout@v4
      - uses: your-username/codeboarding-ghaction@v1
        with:
          # your inputs here
```

## Timeout Guidelines

- **Small repositories** (<1k files): 10-15 minutes
- **Medium repositories** (1k-5k files): 20-30 minutes  
- **Large repositories** (5k+ files): 30-60 minutes
- **Very large repositories** (10k+ files): 45-90 minutes

If your workflow consistently times out, consider:
1. Increasing `timeout-minutes` to 90 or higher
2. Running the action on a schedule during off-peak hours
3. Analyzing specific branches with smaller diffs
