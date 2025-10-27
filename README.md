# Shared GitHub Workflows

A collection of reusable GitHub Actions workflows for Python projects.

## Available Workflows

### Python Tests (`python-test.yml`)

Runs tests, type checking, and linting for Python projects using Poetry.

```yaml
jobs:
  test:
    uses: zawadzkim/shared-gh-workflows/.github/workflows/python-test.yml@main
    with:
      python-version: '3.11'  # optional
      working-directory: '.'  # optional
      poetry-install-args: '--with dev'  # optional
```

### Python Release (`python-release.yml`)

Creates GitHub releases and publishes packages to PyPI when version tags don't exist.

```yaml
jobs:
  release:
    uses: zawadzkim/shared-gh-workflows/.github/workflows/python-release.yml@main
    with:
      working-directory: '.'  # optional
```

### MkDocs Deploy (`mkdocs-deploy.yml`)

Builds and deploys MkDocs documentation to GitHub Pages.

```yaml
jobs:
  docs:
    uses: zawadzkim/shared-gh-workflows/.github/workflows/mkdocs-deploy.yml@main
    with:
      working-directory: '.'  # optional
      python-version: '3.11'  # optional
      poetry-install-args: '--with docs'  # optional
```

## Usage Example

Here's how to use these workflows in your repository:

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main, dev ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    uses: zawadzkim/shared-gh-workflows/.github/workflows/python-test.yml@main

  docs:
    if: github.ref == 'refs/heads/main'
    uses: zawadzkim/shared-gh-workflows/.github/workflows/mkdocs-deploy.yml@main

  release:
    if: github.ref == 'refs/heads/main'
    needs: [test, docs]
    uses: zawadzkim/shared-gh-workflows/.github/workflows/python-release.yml@main
```

## Setup Requirements

1. Enable GitHub Pages in your repository settings
2. Set up PyPI trusted publishing for releases
3. Configure branch protection rules as needed