# Commands

## Installation

```bash
# Add Biograph to your bench
bench get-app https://github.com/Tacten/biograph

# Install on a site
bench --site your-site.local install-app healthcare

# Run migrations after updates
bench --site your-site.local migrate
```

## Frontend Build

```bash
# Build patient portal (from repo root)
yarn build

# Or directly
cd patient_portal && yarn build

# Dev server (hot reload)
cd patient_portal && yarn dev
```

## Testing

```bash
# Run all tests (parallel)
bench --site test_site run-parallel-tests --app healthcare
```

## Linting

```bash
# Run all linters (ruff + semgrep)
pre-commit run --all-files
```

## Common Bench Commands

```bash
# Start development server
bench start

# Clear cache
bench --site your-site.local clear-cache

# Rebuild assets
bench build --app healthcare

# Open console
bench --site your-site.local console

# Export fixtures
bench --site your-site.local export-fixtures --app healthcare
```

## CI/CD Workflows

| Workflow | Trigger | What it does |
|----------|---------|-------------|
| `ci.yml` | PRs + daily cron | Server tests on MariaDB 11.8, Python 3.14, Node 24 |
| `linters.yml` | All PRs | Ruff + Semgrep code quality checks |
| `codeql.yml` | PRs | CodeQL security analysis |
| `docs_checker.yml` | PRs | Documentation validation |
| `generate-pot-file.yml` | On demand | Translation template generation |
| `initiate_release.yml` | Manual | Release initiation |
| `on_release.yml` | Release published | Release actions |
| `release_notes.yml` | On demand | Release notes generation |
| `semantic-commits.yml` | PRs | Commit message validation |

## Test Environment Setup (CI)

```bash
# CI uses this helper script:
.github/helper/install.sh

# It does:
# 1. Clone frappe, erpnext, payments apps
# 2. bench init with specific frappe branch
# 3. Install all apps on test_site
# 4. Set developer_mode and test runner flags
```
