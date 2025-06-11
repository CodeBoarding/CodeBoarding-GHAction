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

jobs:
  documentation:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Generate Documentation
        uses: your-username/CodeBoarding-GHAction@v1
        with:
          repository_url: ${{ github.server_url }}/${{ github.repository }}
          output_directory: '.codeboarding'
          
      - name: Upload Documentation
        uses: actions/upload-artifact@v4
        with:
          name: documentation
          path: .codeboarding/
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `repository_url` | Repository URL for which documentation will be generated | Yes | - |
| `output_directory` | Directory where documentation files will be saved | No | `.codeboarding` |

## Outputs

| Output | Description |
|--------|-------------|
| `files_created` | Number of files created |
| `output_directory` | Directory where files were saved |
| `has_changes` | Whether any files were created or changed |

## License

MIT License - see [LICENSE](LICENSE) file for details.
