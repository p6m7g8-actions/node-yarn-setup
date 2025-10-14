# Copilot Coding Agent Onboarding

This repository defines a **composite GitHub Action** that installs and configures **Node.js** and **Yarn** on GitHub-hosted runners.  
It provides a reproducible environment setup for Node-based projects that require deterministic toolchain versions.

---

## 1. High-Level Details

- **Type:** GitHub Action (composite)  
- **Primary entry point:** `action.yml`  
- **Languages:** YAML + Bash  
- **Runtime:** Ubuntu 24.04 (`ubuntu-latest`)  
- **Purpose:** Automates Node.js + Yarn environment setup for any dependent CI workflow.  

### What the Action Does
1. Downloads and installs a specific version of Node.js.  
2. Adds Node binaries to the system `PATH`.  
3. Enables Corepack and activates Yarn.  
4. Restores cached `node_modules` to speed up builds.  
5. Verifies tool installations (Node, npm, Yarn, Corepack).  
6. Runs dependency installation via `yarn install`.  
7. Caches dependencies after installation.  

### Configurable Inputs
| Input | Description | Default |
|-------|--------------|----------|
| `node-version` | Node.js version to install | `24.9.0` |

This Action is designed to be used by other repositories to ensure consistent Node + Yarn environments across builds.

---

## 2. Build & Validation Instructions

> There is no compilation step for this repository. CI validation is handled via GitHub Actions workflows.

### Bootstrap (local validation)
To reproduce the Action locally:

```bash
NODE_VERSION=24.9.0

curl -fsSL "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.xz" -o "node-v${NODE_VERSION}-linux-x64.tar.xz"
sudo tar -xJf "node-v${NODE_VERSION}-linux-x64.tar.xz" -C /usr/local
sudo rm -f "node-v${NODE_VERSION}-linux-x64.tar.xz"
export PATH="/usr/local/node-v${NODE_VERSION}-linux-x64/bin:$PATH"

sudo corepack enable
sudo corepack prepare yarn@latest --activate

yarn -v && node -v && npm -v && corepack --version
```

**Preconditions:**  
- Ubuntu 24.04 or later.  
- `sudo` available (GitHub-hosted runners satisfy this).  

**Postconditions:**  
- `node`, `npm`, `corepack`, and `yarn` are available in `$PATH`.  
- Yarn can install dependencies from a `package.json`.

### Always do this before running builds
- Always install Node.js before enabling Corepack.  
- Always enable Corepack before activating Yarn.  
- Always ensure `$GITHUB_PATH` includes the Node bin directory.  

### Common Issues and Fixes
| Problem | Cause | Fix |
|----------|--------|-----|
| `curl` 404 | Invalid Node version | Use a released version like 24.9.0 |
| `corepack: not found` | Missing Node | Ensure Node installation completes successfully |
| `yarn install` fails | Missing `package.json` | Ensure the consumer repo defines a Node project |

### Validation
To validate this action inside a consuming repository:
```yaml
- uses: p6m7g8-actions/node-yarn-setup@main
  with:
    node-version: "24.9.0"
```
Then run:
```yaml
- run: yarn install && yarn test
```
This confirms Node, Yarn, and dependency resolution are functioning.

---

## 3. Project Layout and Automation

```
action.yml                   # Composite action definition
.github/workflows/
  ├── build.yml              # Required CI workflow (no-op pass)
  ├── auto-queue.yml         # Adds successful PRs to Merge Queue
  ├── auto-approve.yml       # Auto-approves trusted PRs or authors
  ├── pr-labeler.yml         # Adds 'contribution/core' and 'auto-merge' labels
  └── pull-request-lint.yml  # Enforces Conventional Commit PR titles
.vscode/settings.json        # Editor and YAML formatting defaults
LICENSE
README.md
```

### Continuous Integration Checks
- **Build / build:** Must always pass; acts as a required check for Merge Queue.  
- **PR Title Lint:** Ensures PR titles use allowed types (`chore`, `ci`, `docs`, `feat`, `fix`, `major`, `perf`, `refactor`, `revert`, `style`, `test`).  
- **PR Labeler & Auto-approve:** Labels PRs based on author and state.  
- **Auto-queue:** After a successful **Build**, queues the PR for merge using `gh pr merge`.

### How to Replicate the CI Locally
```bash
act pull_request
```
This runs the `build.yml` workflow locally using the [act](https://github.com/nektos/act) CLI.

### Hidden Dependencies
- Requires Corepack (bundled with Node >=16).  
- Requires network access to fetch the Node tarball and Yarn packages.  
- Requires `sudo` privileges to extract into `/usr/local`.

---

## 4. Guidelines for Copilot Coding Agents

- Treat this repository as a **toolchain bootstrapper**, not an application.  
- Preserve the step sequence: *install Node → enable Corepack → prepare Yarn → verify → install dependencies*.  
- Do not modify step names, job IDs, or workflow names without also updating dependent workflows (`auto-queue.yml`).  
- Keep YAML syntax compliant with GitHub Actions (YAML 1.2).  
- When adding inputs, ensure they have defaults to avoid breaking existing consumers.  
- Prefer stable, released versions for Node and Yarn — avoid nightly builds.  
- Trust this document. Only search the repository if information here is incomplete or demonstrably wrong.

---

This document provides complete bootstrap, validation, and structural knowledge for coding agents working in this repository.
