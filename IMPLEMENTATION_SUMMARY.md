# Implementation Summary

This document records the work completed to create a small static website, package it as a BusyBox container, publish it to GitHub Container Registry (GHCR), and keep publishing to the original package after the repository moved to an organization.

## 1. Initial static website and container

- Created static assets and Dockerfile
- The container serves plain HTTP. TLS and compression are intended to be handled by the surrounding infrastructure.
- Added `.github/workflows/publish.yml` to build and publish the container on pushes to the default `main` branch, with a manual `workflow_dispatch` trigger.

## 2. Initial package creation

- A first workflow run completed successfully. The package was initially published under the original personal namespace, `ghcr.io/nichwall/migration-testing`.

## 3. Repository move and package locations

The repository was moved to the organization `nichwall-old`. Its correct name is `migration-testing`, so the current repository and image path are:

- Repository: [`nichwall-old/migration-testing`](https://github.com/nichwall-old/migration-testing)
- Current image: `ghcr.io/nichwall-old/migration-testing`
- Original image: `ghcr.io/nichwall/migration-testing`

The original GHCR package remained under the personal account after the repository transfer. Its direct listing is [nichwall’s migration-testing package page](https://github.com/users/nichwall/packages/container/package/migration-testing). The repository’s old package link can return 404 because GitHub removes a package’s repository link during a transfer; the package listing remains under its original account.

## 4. Publishing to both package namespaces

- Updated the workflow so each commit publishes to both the current organization image and the original personal image.
- The current organization image is published using the repository-scoped `GITHUB_TOKEN`.
- The original personal package did not accept an anonymous push from the organization workflow. The package’s **Manage Actions access** selector did not offer organization-owned repositories, so a separate credential was used for that package.
- A repository secret named `GHCR_OLD_PACKAGE_TOKEN` was added to the `nichwall-old/migration-testing` repository. It contains a classic personal access token with GHCR package write access to `nichwall` account.
- The workflow now logs into GHCR as `nichwall` with that secret for the original package, then publishes the same source under `ghcr.io/nichwall/migration-testing`.
- Both destinations receive `latest` and the commit SHA tags. The workflow builds and pushes each destination in a separate step so each can use its appropriate credential.

## 5. Workflow and image verification

- The latest checked workflow run completed successfully for commit `50ddc378838dfceaf6bf2bc7d468d5561b2ec44a`. Both the organization publish step and original-package publish step succeeded. [View the run](https://github.com/nichwall-old/migration-testing/actions/runs/42856033809).
- Initially, the organization package could not be inspected anonymously. After GitHub CLI authentication was refreshed with `read:packages`, Docker logged into GHCR and pulled both `latest` images.
- The original image’s `latest` and commit-SHA tags resolved to the same manifest digest: `sha256:ee70f92713f93d88f0d1133e0e68936770226737570a96c2ace1625aa17cd4ee`.
- The organization image `latest` manifest digest was `sha256:39f59c379640efc37b88865b68863b52aa3ca8a81aedd389306af7a2f55cbc50`.
- The registry manifest digests differ, but both pulled images have the same local Docker image ID, `sha256:c9a449b96346aefcde46e75fc28d298f61a685e6ce0c095e9f86b95c66d3a9a8`, and the same reported image size, `4,424,746` bytes.
- Ran the organization image locally and confirmed it served the current page title, **“Fieldnotes — take both the short and medium way.”** The verification container was stopped after the check.

## Current publishing requirements

- Pushes to `main` and manual workflow runs publish to both GHCR namespaces.
- The repository must retain the `GHCR_OLD_PACKAGE_TOKEN` Actions secret for the original personal package.
- The workflow’s `GITHUB_TOKEN` needs `packages: write` for the organization package; that permission is already declared in the workflow.
