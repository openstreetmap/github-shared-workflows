# GitHub Reusable Workflows

This repo contains shared reusable GitHub Action Workflows which are called by other repos.

Workflows are in folder `.github/workflows/`

## Container build and publish

[Container Build and Publish](.github/workflows/container-build-publish.yml)
uses Docker's GitHub Builder to build `linux/amd64` and `linux/arm64` on native
runners and publish a single multi-platform image. Pull requests build both
platforms without publishing. Published images include OCI metadata, BuildKit
provenance, and an SPDX SBOM for each platform's final image. Docker's GitHub
Builder signs the attestations using GitHub OIDC, not the final multi-platform
image index. The workflow does not delete image versions.

The workflow publishes to `ghcr.io/<owner>/<repository>` by default. Give the
calling job `contents: read`, `packages: write` (to publish using `GITHUB_TOKEN`),
and `id-token: write` (to sign attestations). Update existing callers before
rolling out the shared workflow with signing enabled. Use `dockerfile` and
`context` to select files from the triggering commit, or `image_name` to publish
under a different GHCR image name.

Branch builds publish a branch tag. Only default-branch builds update `latest`,
including scheduled rebuilds. A non-default branch named `latest` publishes as
`branch-latest` instead. Stable SemVer tags such as `v1.2.3` publish `1.2.3`,
`1.2`, and `1`; prereleases publish just their full version. The workflow
disables Docker's automatic `latest` tagging so releases never move that tag.