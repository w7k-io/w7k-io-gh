# w7k-io GitHub Actions

Reusable GitHub Actions for w7k-io projects.

## Available Actions

### sync-lockfile

Auto-sync `package-lock.json` or `yarn.lock` when `package.json` is modified in a PR.

#### Usage

```yaml
name: 🔒 Auto-sync lockfile

on:
  pull_request_target:
    types: [opened, synchronize]
    paths:
      - 'package.json'
      # Or for monorepo: 'src/main/ui/package.json'

permissions:
  contents: write

jobs:
  sync-lockfile:
    runs-on: ubuntu-latest
    steps:
      - uses: w7k-io/w7k-io-gh/sync-lockfile@main
        with:
          token: ${{ secrets.NPM_GITHUB_TOKEN }}
          # Optional: working-directory: src/main/ui
          # Optional: package-manager: npm (auto-detected by default)
```

#### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `token` | GitHub token with push permissions (PAT recommended for Dependabot) | Yes | - |
| `working-directory` | Directory containing package.json | No | `.` |
| `package-manager` | `npm` or `yarn` (auto-detected if not specified) | No | auto |
| `node-version` | Node.js version to use | No | `22` |
