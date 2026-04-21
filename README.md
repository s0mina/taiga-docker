# Taiga Infrastructure Fork

This is a fork of `taigaio/taiga-docker` with ARM64 support and automated multi-arch image builds.

## Why this fork exists

Official Taiga Docker images are x86-only. The Hydroxide cluster runs on ARM64 nodes, so images need to be rebuilt for ARM. This fork handles that automatically via GitHub Actions.

## Changes from upstream

| Repo | Change | Reason |
|------|--------|--------|
| taiga-back | `gosu` → `runuser` in Dockerfile + entrypoint.sh + async_entrypoint.sh | gosu downloads x86 binary; runuser is built into util-linux and works on all arches |
| taiga-protected | `gosu` → `runuser` in Dockerfile + entrypoint.sh | Same as above |
| taiga-front | No changes | Upstream already multi-arch (nginx:alpine + pre-built JS) |
| taiga-events | No changes | Upstream already multi-arch (node:16-alpine) |
| taiga-docker | Added `.github/workflows/build.yml` | Multi-arch build pipeline |

## How the build works

1. Push to `stable` triggers `.github/workflows/build.yml`
2. For each of the 4 component repos, the workflow:
   - Checks out the `stable` branch
   - Sets up QEMU for ARM emulation
   - Builds the Dockerfile for `linux/amd64` and `linux/arm64`
   - Pushes to `ghcr.io/s0mina/<repo>:latest` and `:<sha>`
3. Build time is ~20-30 minutes per component due to QEMU emulation

## Image locations

- `ghcr.io/s0mina/taiga-back:latest`
- `ghcr.io/s0mina/taiga-front:latest`
- `ghcr.io/s0mina/taiga-events:latest`
- `ghcr.io/s0mina/taiga-protected:latest`

## Triggering a rebuild

- **Auto:** push any change to the `stable` branch of any of the 4 repos or `taiga-docker`
- **Manual:** Actions tab → Build Taiga Images → Run workflow → select `stable`

## When transferred to Hydroxide-RBWR org

In `.github/workflows/build.yml`:
- Change `ORG: s0mina` → `ORG: hydroxide-rbwr`
- Change all 4 `repo: s0mina/taiga-*` → `repo: Hydroxide-RBWR/taiga-*`

## Deployment

See `Hydroxide-Infra/jobs/taiga*.nomad.hcl` for the Nomad deployment specs (to be added).
