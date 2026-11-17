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

- The repository root `package.json` has no dependencies. Its npm scripts now delegate to `video-url-parser/` using `cd video-url-parser && npm ...`; this replaced the earlier `npm --prefix` delegation after it produced an npm configuration warning. The root `postinstall` script uses the same directory-navigation approach.
- The root `package-lock.json` is lockfile version 3 and contains no dependency package entries.
- `video-url-parser/package.json` declares an empty `dependencies` object and 10 direct `devDependencies`. The extension's npm tooling is therefore development-only according to the manifest.
- `video-url-parser/package-lock.json` is lockfile version 3 and records 363 package entries, all marked as development dependencies.
- Node.js is now standardized on `24.18.0` locally: `.nvmrc` selects `24.18.0`, both package manifests require `>=24.18.0 <25`, and CI runs on Node 24.x. This is an agreed project runtime policy.
- Dependabot is configured for weekly npm updates in both the root and `video-url-parser/` directories, plus GitHub Actions updates.
- CI currently separates lint/format checks from tests/build and runs with read-only repository contents permission.

**Verification update (2026-11-17):** The project owner confirmed Node.js `v24.18.0` and npm `12.2.0`. ESLint, Prettier, all 310 tests, coverage, and the production build passed before the Node-version configuration update. After the Node 24 configuration was pulled, `npm run check` reached `format:check` and stopped because `package.json` has a Prettier formatting warning; tests and build were therefore not rerun in that latest command. The owner has explicitly said occasional Prettier warnings are non-blocking and will be handled periodically. A dedicated dependency/outdated/security review is still pending; do not infer a clean audit from the lockfile or earlier root-level audit output.

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
- Make approved project changes directly on `master` unless the project owner explicitly requests a separate branch. Do not create a branch without telling the owner first.
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
- [x] Review all npm scripts and understand which directory owns each tool; simplify root delegation to use `cd video-url-parser && npm ...`.
- [ ] Inventory production and development dependencies.
- [ ] Identify lifecycle scripts and transitive dependency considerations.
- [x] Review and standardize the Node.js version policy: `.nvmrc` is `24.18.0`, both package manifests require `>=24.18.0 <25`, and CI uses Node 24.x.
- [x] Inspect the existing CI workflow and automated dependency update configuration; CI has been aligned to Node 24.x.
- [ ] Check whether the documented setup matches the actual repository configuration.

### 1.2 Security and Maintenance Review

- [ ] Check installed dependencies for known vulnerabilities.
- [ ] Review outdated dependencies and determine which updates are necessary.
- [ ] Review dependency update strategy and lockfile practices.
- [x] Investigate the `global-ignore-file` warning and simplify root npm script delegation; the warning disappeared in the subsequent root-level check.
- [ ] Review build and packaging scripts for reproducibility and unintended files.
- [ ] Identify opportunities to strengthen static checks and CI security.
- [ ] Record findings, risks, and recommended actions before changing tooling.

### 1.3 Development Environment Decisions

- [x] Agree on the supported Node.js version: Node.js 24, with `.nvmrc` pinned to `24.18.0`.
- [ ] Agree on the package manager and lockfile strategy.
- [x] Keep root-level install delegation, using `cd video-url-parser && npm install` rather than `npm --prefix`.
- [ ] Agree on dependency update and audit practices.
- [ ] Agree on the required local validation commands.
- [ ] Document the final setup instructions.

### 1.4 Apply and Verify Approved Changes

- [x] Apply the agreed tooling and configuration changes: root npm delegation and Node 24 runtime/CI configuration.
- [ ] Update lockfiles where necessary.
- [ ] Run dependency installation from a clean state.
- [ ] Run tests, linting, formatting checks, coverage, and production build after the Node 24 configuration update. Note: Prettier warnings are non-blocking for this project owner; if `npm run check` stops at formatting, run the remaining test/coverage/build commands separately.
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

### 4.1 English-Only Localization Cleanup

**Status:** Planned

**Objective:** Simplify the extension for personal use by retaining English and removing unused translations.

- [ ] Keep the English UI translation as the single supported language.
- [ ] Keep the English Chrome extension metadata locale and remove other locale directories.
- [ ] Remove the language selector from the popup if it is only used for switching translations.
- [ ] Simplify or remove translation-loading and language-switching logic that is no longer needed.
- [ ] Update localization tests to validate the English-only setup instead of requiring 25+ languages.
- [ ] Check the manifest and all locale references before deleting translation assets.
- [ ] Run the full project check and production build after the cleanup.

**Scope note:** This is a planned cleanup, not authorization to remove files immediately. Inspect the current localization wiring first and preserve English labels, descriptions, and popup behavior.

## Decision Log

Record significant project decisions here, including the date, context, decision, and consequences.

| Date | Decision | Status |
| --- | --- | --- |
| 2026-11-17 | Begin with development environment and security baseline before feature work. | Agreed |
| 2026-11-17 | Plan to simplify localization to English only for personal use; inspect and update UI wiring, Chrome metadata, and tests before removing other locales. | Planned |
| 2026-11-17 | Standardize the project runtime on Node.js 24: `.nvmrc` `24.18.0`, package engine range `>=24.18.0 <25`, and CI Node 24.x. | Implemented; local version verified |
| 2026-11-17 | Replace root `npm --prefix` script delegation with directory navigation after investigating the npm configuration warning. | Implemented; warning absent in subsequent root check |
| 2026-11-17 | Treat occasional Prettier formatting warnings as non-blocking; the project owner will format periodically. | Agreed |

## Notes

- This roadmap is an evolving plan, not a commitment to implement every item.
- Proposed changes remain pending until discussed and agreed upon.
- Mark tasks complete only after implementation and verification.
