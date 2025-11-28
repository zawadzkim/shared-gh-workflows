# Shared GitHub Workflows

A collection of reusable GitHub Actions workflows for Python projects using Poetry and Pixi.

## Available Workflows

### Poetry-based Workflows

#### Python Tests (`python-test.yml`)

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

#### Python Release (`python-release.yml`)

Creates GitHub releases and publishes packages to PyPI when version tags don't exist.

```yaml
jobs:
  release:
    uses: zawadzkim/shared-gh-workflows/.github/workflows/python-release.yml@main
    with:
      working-directory: '.'  # optional
      publish: true  # optional
    secrets:
      PYPI_TOKEN: ${{ secrets.PYPI_TOKEN }}  # required if publish=true
```

#### MkDocs Deploy (`mkdocs-deploy.yml`)

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

### Pixi-based Workflows

#### Pixi Tests (`pixi-test.yml`)

Runs tests, linting, and builds packages for Python projects using Pixi.

```yaml
jobs:
  test:
    uses: zawadzkim/shared-gh-workflows/.github/workflows/pixi-test.yml@pixi
    with:
      pixi-version: 'v0.59.0'  # optional
      working-directory: '.'  # optional
      environment: 'dev'  # optional
      test-command: 'pytest tests/ -v'  # optional
      lint-command: 'ruff check .'  # optional
      coverage: true  # optional
      upload-artifacts: true  # optional
```

#### Pixi Deployment (`pixi-deploy.yml`)

Complete deployment workflow for Pixi projects including testing, building, releasing, and PyPI publishing.

```yaml
jobs:
  deploy:
    uses: zawadzkim/shared-gh-workflows/.github/workflows/pixi-deploy.yml@pixi
    with:
      pixi-version: 'v0.59.0'  # optional
      working-directory: '.'  # optional
      environment: 'dev'  # optional
      pypi-environment: 'pypi'  # optional
      deploy-docs: false  # optional
      docs-environment: 'docs'  # optional
      docs-build-command: 'mkdocs build'  # optional
      skip-existing-versions: true  # optional
    secrets:
      PYPI_TOKEN: ${{ secrets.PYPI_TOKEN }}  # required
```

#### Pixi Documentation (`pixi-docs.yml`)

Builds and optionally deploys documentation for Pixi projects.

```yaml
jobs:
  docs:
    uses: zawadzkim/shared-gh-workflows/.github/workflows/pixi-docs.yml@pixi
    with:
      pixi-version: 'v0.59.0'  # optional
      working-directory: '.'  # optional
      docs-environment: 'docs'  # optional
      docs-build-command: 'mkdocs build'  # optional
      deploy: false  # optional
```

## Usage Examples

### Poetry Project Example

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
    with:
      publish: true
    secrets:
      PYPI_TOKEN: ${{ secrets.PYPI_TOKEN }}
```

### Pixi Project Example

```yaml
# .github/workflows/cd.yml
name: Continuous Deployment

on:
  push:
    branches: [main]
    paths-ignore:
      - "docs/**"
      - "*.md"
  workflow_dispatch:

permissions:
  contents: write
  packages: write
  pages: write
  id-token: write

jobs:
  deploy:
    uses: zawadzkim/shared-gh-workflows/.github/workflows/pixi-deploy.yml@pixi
    with:
      deploy-docs: true
      docs-environment: 'docs'
      docs-build-command: 'mkdocs build'
    secrets:
      PYPI_TOKEN: ${{ secrets.PYPI_TOKEN }}
```

```yaml
# .github/workflows/ci.yml
name: Continuous Integration

on:
  push:
    branches: [dev]
    paths-ignore:
      - "docs/**"
      - "*.md"
  pull_request:
    branches: [main, dev]
    paths-ignore:
      - "docs/**"
      - "*.md"

jobs:
  test:
    uses: zawadzkim/shared-gh-workflows/.github/workflows/pixi-test.yml@pixi
    with:
      environment: 'dev'
      test-command: 'pytest tests/ -v --cov=mypackage --cov-report=term --cov-report=xml'
      lint-command: 'ruff check .'
      coverage: true
```

## Setup Requirements

### For Poetry Projects
1. Enable GitHub Pages in your repository settings
2. Set up PyPI trusted publishing for releases
3. Configure branch protection rules as needed

### For Pixi Projects
1. Ensure your `pyproject.toml` has proper project metadata
2. Set up PyPI trusted publishing or provide `PYPI_TOKEN` secret
3. Configure Pixi environments in `pixi.toml`:
   - `dev` environment with test dependencies
   - `docs` environment with documentation dependencies (if using docs)
4. Enable GitHub Pages if deploying documentation
5. Set up proper permissions in workflow files

## Pixi Environment Setup

Example `pixi.toml` configuration:

```toml
[project]
name = "my-package"
version = "0.1.0"
# ... other project metadata

[environments]
dev = ["test", "lint", "build"]
docs = ["doc"]

[dependencies]
python = ">=3.10"

[feature.test.dependencies]
pytest = "*"
pytest-cov = "*"

[feature.lint.dependencies]
ruff = "*"

[feature.build.dependencies]
build = "*"

[feature.doc.dependencies]
mkdocs = "*"
mkdocs-material = "*"
```