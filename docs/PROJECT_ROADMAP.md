# Project Roadmap

## Project Goal

Maintain and improve the Simple Video Download Helper browser extension with a focus on a reliable development workflow, security, maintainability, and preserving the extension's intended functionality.

The extension source and its tooling currently live in `video-url-parser/`. The repository root provides convenience npm scripts that delegate into that directory.

## Roadmap Principles

- Work in small, reviewable steps.
- Audit before changing dependencies, architecture, permissions, or behavior.
- Prioritize security improvements that address a clearly identified risk.
- Preserve the parser's dependency-free runtime unless a change is explicitly agreed upon.
- Verify changes locally before considering a task complete.
- Record important decisions and update this roadmap as the project evolves.

## Initial Repository Observations

These are preliminary facts from the first read-only inspection. They are not a completed security audit, and no dependencies or runtime configuration have been changed.

- The repository root `package.json` has no dependencies. Its `postinstall` script runs `npm --prefix video-url-parser install`, while the root npm scripts delegate other tasks to that subproject.
- The root `package-lock.json` is lockfile version 3 and contains no dependency package entries.
- `video-url-parser/package.json` declares an empty `dependencies` object and 10 direct `devDependencies`. The extension's npm tooling is therefore development-only according to the manifest.
- `video-url-parser/package-lock.json` is lockfile version 3 and records 363 package entries, all marked as development dependencies.
- Node.js version declarations are inconsistent in age: `.nvmrc` selects Node 20, both package manifests allow Node.js >=18, and CI tests Node 18, 20, and 22.
- As of 2026-11-17, Node.js 18 and 20 are End-of-Life. Node.js 22 and 24 are in Maintenance LTS, while Node.js 26 is in Active LTS. The current Node 20 local default should therefore be reconsidered before setup is finalized.
- Dependabot is configured for weekly npm updates in both the root and `video-url-parser/` directories, plus GitHub Actions updates.
- CI currently separates lint/format checks from tests/build and runs with read-only repository contents permission.

**Not yet verified:** No local dependency installation, `npm audit`, outdated-package check, test run, lint, formatting check, or build has been performed as part of this inspection. The lockfile contents alone do not establish whether the dependency tree has known vulnerabilities.

## Architecture Findings from Initial Code Review

The following points were confirmed by reading the repository files on 2026-11-17. They describe the current implementation; they are not all approved design decisions.

- `video-url-parser/` is the actual extension project root. The repository root is a wrapper for documentation/configuration and npm scripts that delegate to the subproject.
- The extension's original JavaScript source is in `video-url-parser/js/`.
- `video-url-parser/webpack.config.js` sets `js/getPagesSource.js` as the Webpack entry point and outputs `dist/dist.min.js`. The generated `dist/` directory is not currently present in the tracked repository and should be produced by the build.
- `video-url-parser/manifest.json` declares `js/background.js` both as the Manifest V3 service worker and as a content script on almost all HTTP(S) pages.
- `js/background.js` is also loaded by the popup. It branches on the execution context: popup/extension page code injects jQuery and the parser bundle into the active tab; service-worker code manages the toolbar badge and context menus; content-script execution does not have the APIs needed for the popup injection path.
- The context menu's direct video/audio action uses Chrome's `downloads.download` API. The "Find videos on this page" action opens an external parser website and places the current page URL in its query string.
- The parser bundle is built from source; future parser changes should target readable files under `js/`, not hand-edit generated minified output.
- The current design has a likely unnecessary broad content-script registration, but its removal must be verified with regression testing before it is treated as safe.
- The extension also has external parser/API and online merge integrations outside `background.js`. Trace each integration before making privacy claims or removing host permissions.
- The extension detects media URLs and hands downloadable URLs to browser download functionality. HLS playlist parsing and actual complete stream downloading/merging are distinct concerns; do not assume the current implementation is a complete local HLS downloader.

## Agreed Working and Testing Approach

- Use GitHub Desktop to clone the repository locally; do not run `git init` because this repository already has Git history.
- Make project changes on a dedicated working branch rather than directly on the default branch.
- Work from the repository root for the convenience npm scripts. The root scripts delegate to `video-url-parser/`.
- After building, the unpacked extension directory to load in Chrome is `video-url-parser/`, because it contains the extension's `manifest.json` and runtime assets.
- Use Chrome's `chrome://extensions/` page with Developer mode and **Load unpacked** for local testing. After rebuilding, use the extension's reload control to load the new build.
- Do not assume a build, test, or local installation has succeeded until the project owner reports the result.

## Milestone 1 — Development Environment and Security Baseline

**Status:** In progress

**Objective:** Understand the current development toolchain and establish a secure, reproducible environment before making broader code changes.

### 1.1 Repository and Tooling Audit

- [ ] Inspect the root `package.json` and `package-lock.json`.
- [ ] Inspect `video-url-parser/package.json` and its lockfile, if present.
- [ ] Review all npm scripts and understand which directory owns each tool.
- [ ] Inventory production and development dependencies.
- [ ] Identify lifecycle scripts and transitive dependency considerations.
- [ ] Review the current Node.js version policy, including `.nvmrc` and `engines`.
- [ ] Inspect the existing CI workflow and automated dependency update configuration.
- [ ] Check whether the documented setup matches the actual repository configuration.

### 1.2 Security and Maintenance Review

- [ ] Check installed dependencies for known vulnerabilities.
- [ ] Review outdated dependencies and determine which updates are necessary.
- [ ] Review dependency update strategy and lockfile practices.
- [ ] Evaluate npm configuration and install behavior, including lifecycle scripts.
- [ ] Review build and packaging scripts for reproducibility and unintended files.
- [ ] Identify opportunities to strengthen static checks and CI security.
- [ ] Record findings, risks, and recommended actions before changing tooling.

### 1.3 Development Environment Decisions

- [ ] Agree on the supported Node.js version.
- [ ] Agree on the package manager and lockfile strategy.
- [ ] Decide whether root-level install delegation should remain.
- [ ] Agree on dependency update and audit practices.
- [ ] Agree on the required local validation commands.
- [ ] Document the final setup instructions.

### 1.4 Apply and Verify Approved Changes

- [ ] Apply the agreed tooling and configuration changes.
- [ ] Update lockfiles where necessary.
- [ ] Run dependency installation from a clean state.
- [ ] Run tests, linting, formatting checks, coverage, and production build.
- [ ] Verify the generated extension package and unpacked extension.
- [ ] Review the final diff and document the outcome.

**Milestone completion criteria:**

- The development environment and supported Node.js version are documented.
- Dependency ownership and installation behavior are understood.
- Security and maintenance risks have been reviewed.
- Approved changes are implemented without unexplained dependency or architecture changes.
- Relevant quality checks and the extension build pass locally.
- The project owner has reviewed the changes.

## Milestone 2 — Extension Architecture and Security Review

**Objective:** Understand the extension's runtime architecture and identify security-sensitive code paths.

- [ ] Map the extension entry points, popup, content scripts, and background/service-worker behavior.
- [ ] Review `manifest.json` permissions and host access.
- [ ] Trace message passing between extension contexts.
- [ ] Review URL parsing, validation, and page-provided input handling.
- [ ] Review DOM operations and HTML insertion for injection risks.
- [ ] Review network requests and data handling.
- [ ] Document the architecture and prioritize verified findings.

## Milestone 3 — Reliability and Test Coverage

**Objective:** Improve confidence in existing behavior and future changes.

- [ ] Review existing test coverage and identify important gaps.
- [ ] Add regression tests for confirmed bugs and security-sensitive behavior.
- [ ] Improve test fixtures for supported websites where appropriate.
- [ ] Verify build and packaging behavior in CI.
- [ ] Establish practical regression checks for future changes.

## Milestone 4 — Product Improvements

**Objective:** Plan and implement user-facing improvements after the development and security baseline is established.

- [ ] Review current extension behavior and known limitations.
- [ ] Gather and prioritize proposed improvements.
- [ ] Agree on scope and acceptance criteria for each change.
- [ ] Implement changes incrementally with appropriate tests and documentation.

## Decision Log

Record significant project decisions here, including the date, context, decision, and consequences.

| Date | Decision | Status |
| --- | --- | --- |
| 2026-11-17 | Begin with development environment and security baseline before feature work. | Agreed |

## Notes

- This roadmap is an evolving plan, not a commitment to implement every item.
- Proposed changes remain pending until discussed and agreed upon.
- Mark tasks complete only after implementation and verification.
