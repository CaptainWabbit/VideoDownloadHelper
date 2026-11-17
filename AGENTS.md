# AGENTS.md

## Project Overview

This repository contains the Simple Video Download Helper browser extension.

- The browser extension and its development tooling live in `video-url-parser/`.
- The repository root contains project documentation, repository configuration, and npm scripts that delegate to the extension project.
- The extension uses Chrome Extension Manifest V3.
- The parser is intended to remain dependency-free at runtime. Development and test dependencies are separate from the shipped extension code.

## Confirmed Extension Architecture

The actual browser extension and its development tooling live in `video-url-parser/`. Treat that directory as the extension project root.

Key source files and responsibilities:

- `manifest.json`: Manifest V3 configuration, extension permissions, popup, service worker, and content-script registration.
- `main.html`: Popup UI.
- `js/popup.js`: Popup behavior.
- `js/background.js`: Currently reused in multiple contexts: the Manifest V3 service worker, a broadly registered content script, and an extension page script loaded by the popup. Its behavior is gated by whether `window` exists.
- `js/getPagesSource.js`: Webpack entry point for the media-detection/parser bundle.
- `js/parsevideo.js`: Video parsing logic.
- `js/video.js`: Popup-side media URL handling and download-related behavior.
- `js/functions.js` and `js/constants.js`: Shared utilities and constants.
- `webpack.config.js`: Bundles `js/getPagesSource.js` into `dist/dist.min.js`.
- `dist/dist.min.js`: Generated parser bundle; it is build output, not the primary source to edit. The current repository does not track this generated directory.

The root `package.json` is a convenience wrapper. Its scripts delegate to `video-url-parser/`; its `postinstall` script installs that subproject's development dependencies. The extension's own `package.json` and lockfile are inside `video-url-parser/`.

### Current background-script behavior

Based on the inspected `js/background.js` and `manifest.json`:

- In the popup/extension-page context, `background.js` queries the active tab and injects jQuery 3.4.1 and the generated parser bundle into eligible HTTP(S) pages.
- In the service-worker context, it updates the toolbar badge from parser messages, clears the badge when a tab starts loading, and creates right-click context menus.
- The direct video/audio context-menu action calls Chrome's `downloads.download` API with the selected media URL.
- The "Find videos on this page" context-menu action opens an external parser website and includes the current page URL in its query string.
- The same file is registered as a content script on nearly all HTTP(S) pages. In that context, the popup injection path is not available; the file currently adds a runtime message listener but does not perform the main parser injection.

These are observations of the current implementation, not a final architecture recommendation. Verify behavior before removing or relocating any responsibilities.

### Privacy and external-service observations

- The context-menu external-parser action sends the current page URL to the external website when the user selects that action.
- Other external parser, optional VIP API, and online HLS-merge integrations exist elsewhere in the extension and must be reviewed in their owning source files.
- Do not describe the extension as entirely local until all network requests and external integrations have been traced.
- No malicious behavior was identified in the inspected `background.js`, but reviewing one file is not a complete security audit.

## Working Agreement

- The repository owner is the project lead and makes final decisions about scope, architecture, dependencies, and behavior.
- Before making implementation changes, inspect the relevant existing code and documentation.
- Prefer small, focused changes that are easy to review.
- Do not modify unrelated files or perform broad refactors without an agreed reason.
- Do not upgrade dependencies, change supported runtime versions, or alter browser permissions without first documenting the rationale and discussing the impact.
- Do not introduce runtime dependencies into the parser without explicit approval.
- Preserve existing functionality unless a change has been agreed upon.
- Never commit secrets, credentials, personal data, local environment files, or generated build artifacts.

## Security Priorities

Treat this as a browser extension with access to user browsing contexts. Security decisions must account for both the extension's source code and its build/release toolchain.

When reviewing changes, pay particular attention to:

- Least-privilege extension permissions and host permissions.
- Content-script and background/service-worker message validation.
- Safe handling of URLs and page-provided data.
- Avoiding unsafe DOM insertion and script execution.
- Dependency provenance, vulnerabilities, lifecycle scripts, and lockfile integrity.
- Build output and packaging contents.
- Privacy: avoid collecting, transmitting, or retaining user data unless explicitly required and documented.

Do not claim a security improvement without identifying the risk it addresses and how it is verified.

## Development Workflow

The agreed collaboration workflow is:

1. Inspect the repository and make focused changes on the GitHub repository.
2. Report the exact files changed and summarize the changes.
3. The project owner fetches/pulls the changes locally using GitHub Desktop or the VS Code terminal.
4. The project owner reviews and tests the changes locally.
5. Discuss results and decide the next step before proceeding.

Do not assume local commands have been run or tests have passed unless their results are available.

## Documentation

- Keep the project roadmap in `docs/PROJECT_ROADMAP.md`.
- Update documentation when an agreed change affects development setup, architecture, security practices, or project workflow.
- Distinguish confirmed facts from proposals and pending decisions.
- Keep roadmap items actionable and mark their status accurately.

## Repository Conventions

- Follow the existing formatting, linting, testing, and build conventions unless a change to those conventions has been agreed upon.
- Inspect the relevant package manifest and scripts before changing dependencies or tooling.
- Keep `package-lock.json` consistent with any approved `package.json` changes.
- Run the narrowest relevant checks first, then the project's full check script when appropriate.
- Do not edit generated files unless the task specifically requires it.
