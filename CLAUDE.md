# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository automates building and publishing Docker container images for Claude Code's development environment to GitHub Container Registry (GHCR). It monitors two upstream sources daily:
- Anthropic's official Claude Code `.devcontainer` configuration (via git submodule at `claude-code/`)
- New `@anthropic-ai/claude-code` npm package releases

Published image: `ghcr.io/pravorskyi/claude-code-devcontainer/devcontainer:latest`

## Architecture

**Two Dockerfiles exist — they serve different purposes:**
- `claude-code/.devcontainer/Dockerfile` — The **source of truth** for published images. This is the upstream Anthropic Dockerfile maintained via git submodule.
- `.devcontainer/Dockerfile` — Used only for developing THIS project locally. It extends the published GHCR image.

**CI/CD Workflows** (`.github/workflows/`):
- `check.yml` — Daily scheduled check (2am UTC) that detects upstream `.devcontainer` changes or new npm releases, then triggers the build workflow.
- `devcontainer.yml` — Builds and pushes the Docker image to GHCR with `latest` and version-specific tags (e.g., `claude-code-1.0.20`).
- `update-submodule.yml` — Updates the `claude-code/` submodule reference when upstream changes are detected.

**Firewall isolation** (`.devcontainer/init-firewall.sh`):
Network traffic is restricted to whitelisted domains only (GitHub IPs via API, npm registry, Anthropic services, VS Code updates). Uses iptables/ipset with dynamically-resolved IP ranges.

## Key Commands

This is an infrastructure/CI-CD project with no traditional build/test/lint commands. All automation runs via GitHub Actions workflows.

```bash
# Update the submodule to latest upstream
git submodule update --remote claude-code

# Check current Claude Code version on npm
npm view @anthropic-ai/claude-code version

# Build the devcontainer image locally (from the upstream Dockerfile)
docker build -f claude-code/.devcontainer/Dockerfile claude-code/.devcontainer/
```

## Git Workflow

- `master` is the main branch
- Feature work goes through `dev` branch with PRs into `master`
- The `claude-code/` directory is a git submodule — use `git submodule update --init` after cloning
