# w7k-io GitHub Actions

Reusable GitHub Actions for w7k-io projects.

## Available Actions

### sync-npm-lockfile

Auto-sync `package-lock.json` when `package.json` is modified in a PR (npm only).

```yaml
- uses: w7k-io/w7k-io-gh/sync-npm-lockfile@main
  with:
    token: ${{ secrets.NPM_GITHUB_TOKEN }}
    # Required as soon as you depend on a private package:
    npm-token: ${{ secrets.NPM_GITHUB_TOKEN }}
    # Optional:
    # working-directory: src/main/ui
    # node-version: '22'
```

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `token` | GitHub token with push permissions — **git only** | Yes | - |
| `npm-token` | Token for a private npm registry, exported as `NPM_GITHUB_TOKEN` | No | - |
| `working-directory` | Directory containing package.json | No | `.` |
| `node-version` | Node.js version | No | `24` |

> Keep `node-version` identical to the one your CI jobs run on. A lockfile
> regenerated under a different npm major can resolve dependencies differently
> from the one your tests validated.

> ⚠️ **`token` authenticates git, never npm.** As soon as your project depends on a
> private package, pass `npm-token` as well — otherwise npm resolves an empty variable
> in `.npmrc` and fails with `401 Unauthorized ... unauthenticated: User cannot be
> authenticated with the token provided`. The failure only shows up in CI: developer
> machines already carry a token in their `~/.npmrc`. Cf. JUNI-1285.

> 🔒 **Guard your workflow against fork PRs.** This action is meant to run on
> `pull_request_target`, which executes the PR's code *with* repository secrets. It runs
> npm with `--ignore-scripts` so a malicious `preinstall` cannot read them, but that is
> defence in depth — add the guard on your side too:
>
> ```yaml
> if: github.event.pull_request.head.repo.full_name == github.repository
> ```

---

### gitversion

Calculate semantic version using GitVersion 6. Embeds a default config so repos don't need a `GitVersion.yml` file.

```yaml
# Default: ContinuousDeployment mode (no GitVersion.yml needed)
- uses: w7k-io/w7k-io-gh/gitversion@main
  id: version

# ContinuousDelivery mode (for repos that create releases manually)
- uses: w7k-io/w7k-io-gh/gitversion@main
  id: version
  with:
    mode: delivery

# With a next-version baseline
- uses: w7k-io/w7k-io-gh/gitversion@main
  id: version
  with:
    next-version: '1.0.0'

# Custom config file (overrides mode/next-version)
- uses: w7k-io/w7k-io-gh/gitversion@main
  id: version
  with:
    config-file: GitVersion.yml

- run: echo "Version: ${{ steps.version.outputs.version }}"
```

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `mode` | `deployment` (ContinuousDeployment) or `delivery` (ContinuousDelivery + GitHubFlow) | No | `deployment` |
| `next-version` | Version baseline (e.g., `1.0.0`) | No | _(auto)_ |
| `config-file` | Path to custom config (overrides mode/next-version) | No | _(generated)_ |

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

### github-release

Create a GitHub release with structured release notes (Features / Bug Fixes / Other Changes). Optionally bumps version files and attaches assets.

```yaml
- uses: w7k-io/w7k-io-gh/github-release@main
  with:
    version: ${{ steps.version.outputs.version }}
    token: ${{ secrets.NPM_GITHUB_TOKEN }}
    release-prefix: 'MyApp'        # → "MyApp v1.2.3"
    assets: 'dist/*.zip'           # Optional release assets
    bump-version: 'true'           # Bump package.json/pom.xml/pyproject.toml
```

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | Version to release | Yes | - |
| `token` | GitHub token with push + release permissions | Yes | - |
| `assets` | Glob pattern for release assets | No | _(none)_ |
| `release-prefix` | Release name prefix (e.g., `Juni` → `Juni v1.2.3`) | No | _(none)_ |
| `bump-version` | Bump version files before releasing | No | `true` |
| `bump-files` | `auto`, `package.json`, `pom.xml`, or `both` | No | `auto` |
| `bump-working-directory` | Directory containing version files | No | `.` |

| Output | Description |
|--------|-------------|
| `release-url` | URL of the created release |
| `release-id` | ID of the created release |

Release notes are auto-generated from git log between the previous tag and HEAD, categorized by commit prefix:
- `feat:` / `✨` → Features
- `fix:` / `🐛` / `🩹` → Bug Fixes
- Everything else → Other Changes

---

### bump-version

Update version in package.json and/or pom.xml, then commit and push.

```yaml
- uses: w7k-io/w7k-io-gh/bump-version@main
  with:
    version: ${{ steps.version.outputs.version }}
    token: ${{ secrets.GITHUB_TOKEN }}
```

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | Version to set | Yes | - |
| `token` | GitHub token with push permissions | Yes | - |
| `working-directory` | Directory containing version files | No | `.` |
| `files` | `auto`, `package.json`, `pom.xml`, or `both` | No | `auto` |
| `commit-message` | Commit message (`{version}` placeholder) | No | `chore: bump version to {version}` |

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
    # Optional — overrides the workspace default:
    # node-version: '22'
```

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `node-version` | Node.js version | No | `24` |
| `npm-token` | NPM token (sets `NPM_GITHUB_TOKEN` env var) | No | - |

> The default tracks the **active Node LTS** for the whole workspace. Because
> consumers reference this action as `@main`, changing it takes effect in every
> repo immediately, with no PR in their own diff. A repo that needs to stay
> behind — during a migration, or because its runtime image is not ready — must
> pin `node-version:` explicitly on its side.

| Output | Description |
|--------|-------------|
| `node-version` | Installed Node.js version |
