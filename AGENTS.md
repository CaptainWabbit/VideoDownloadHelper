# AGENTS.md

## Project Overview

This repository contains the Simple Video Download Helper browser extension.

- The browser extension and its development tooling live in `video-url-parser/`.
- The repository root contains project documentation, repository configuration, and npm scripts that delegate to the extension project.
- The extension uses Chrome Extension Manifest V3.
- The parser is intended to remain dependency-free at runtime. Development and test dependencies are separate from the shipped extension code.

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
