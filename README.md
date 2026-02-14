# Claude Code Devcontainer

Anthropic ships a `.devcontainer` with [Claude Code](https://github.com/anthropics/claude-code) but no image. So you have to build it yourself, every time. You build it Monday. Claude Code updates Tuesday. `.devcontainer` changes Wednesday. By Thursday you're questioning your life choices. Fuck that. I automated it.

This repo builds it for you daily and pushes to GHCR. Zero modifications.

## Usage

Use as a base image in your own Dockerfile:

```dockerfile
FROM ghcr.io/pravorskyi/claude-code-devcontainer/devcontainer:latest

# Example: add Python tooling
USER root
RUN apt-get update && apt-get install -y --no-install-recommends \
    python3-dev build-essential python3-venv \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

ENV VIRTUAL_ENV=/opt/venv
COPY requirements.txt /opt
RUN python3 -m venv "$VIRTUAL_ENV" && \
    . "$VIRTUAL_ENV/bin/activate" && \
    pip install --no-cache-dir -r /opt/requirements.txt
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

USER node
```

Or in `devcontainer.json` directly, if you don't need to extend the image and don't use Claude Code's `.devcontainer` as your main devcontainer:

```json
{
  "name": "Claude Code",
  "image": "ghcr.io/pravorskyi/claude-code-devcontainer/devcontainer:latest"
}
```

Pin to a specific Claude Code version instead of `latest`:

```
ghcr.io/pravorskyi/claude-code-devcontainer/devcontainer:claude-code-1.0.20
```

## How it works

A daily CI job checks for two things:
- New Claude Code releases on npm
- Changes to `.devcontainer` in the upstream repo

When either is detected, it builds and pushes the image to GHCR.

## Why trust this

The [`claude-code`](https://github.com/anthropics/claude-code) submodule points to the official Anthropic repo. The [Dockerfile](https://github.com/anthropics/claude-code/tree/main/.devcontainer) comes from there, unmodified. The [CI workflows](.github/workflows/) are short and readable. Every image is labeled with the exact source commit it was built from.

You can verify: `docker inspect ghcr.io/pravorskyi/claude-code-devcontainer/devcontainer:latest` and check `org.opencontainers.image.source` and `org.opencontainers.image.revision`.

## Why not trust this

This is a personal repo by some random dude (hi!) on the internet. GitHub Actions could be modified to inject anything into the image between builds. The submodule proves the source, not the output. If this matters to you, fork the repo and build your own image — the CI is all there.
