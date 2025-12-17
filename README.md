# w7k-io GitHub Actions

Reusable GitHub Actions for w7k-io projects.

## Available Actions

### sync-lockfile

Auto-sync `package-lock.json` when `package.json` is modified in a PR.

```yaml
- uses: w7k-io/w7k-io-gh/sync-lockfile@main
  with:
    token: ${{ secrets.NPM_GITHUB_TOKEN }}
    # Optional:
    # working-directory: src/main/ui
    # node-version: '22'
```

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `token` | GitHub token with push permissions | Yes | - |
| `working-directory` | Directory containing package.json | No | `.` |
| `node-version` | Node.js version | No | `22` |

---

### gitversion

Calculate semantic version using GitVersion and optionally update package.json.

```yaml
- uses: w7k-io/w7k-io-gh/gitversion@main
  id: version
  with:
    config-file: GitVersion.yml
    update-package-json: 'true'
    upload-artifact: 'true'

- run: echo "Version: ${{ steps.version.outputs.version }}"
```

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `config-file` | Path to GitVersion config | No | `GitVersion.yml` |
| `update-package-json` | Update package.json version | No | `true` |
| `upload-artifact` | Upload package.json as artifact | No | `true` |

| Output | Description |
|--------|-------------|
| `version` | Full semantic version (e.g., 1.2.3) |
| `major` | Major version number |
| `minor` | Minor version number |
| `patch` | Patch version number |
| `prerelease` | Pre-release tag |
| `branch` | Branch name |
| `sha` | Commit SHA |

---

### setup-node-npm

Setup Node.js and set `NPM_GITHUB_TOKEN` env var for private registry authentication.

Requires a `.npmrc` in your repo with:
```
@w7k-io:registry=https://npm.pkg.github.com/
//npm.pkg.github.com/:_authToken=${NPM_GITHUB_TOKEN}
```

```yaml
- uses: w7k-io/w7k-io-gh/setup-node-npm@main
  with:
    npm-token: ${{ secrets.NPM_GITHUB_TOKEN }}
```

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `node-version` | Node.js version | No | `22` |
| `npm-token` | NPM token (sets `NPM_GITHUB_TOKEN` env var) | No | - |

| Output | Description |
|--------|-------------|
| `node-version` | Installed Node.js version |
