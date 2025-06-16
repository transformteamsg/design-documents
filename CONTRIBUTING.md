# Contributing to Design Documents

This guide explains how to contribute to our design documents through the Request for Comments (RFC) and Architecture Decision Records (ADR) process.

## Request for Comments (RFC) Process

For significant architectural decisions or major feature proposals, we follow a Request for Comments (RFC) process:

1. **Create an RFC Issue:**

   - Use the RFC template when creating a new GitHub Issue
   - Follow the standard RFC format (see [ADR 0001](adr/0001-standardise-rfc-and-adr-templates.md) for template)
   - Include a clear timeline for discussion and voting

2. **Discussion Phase:**

   - Engage with the team through comments on the RFC issue
   - Address questions and incorporate feedback
   - The discussion phase typically lasts 1-2 weeks

3. **Voting Phase:**

   - Author initiates voting by posting a comment
   - Team members vote using 👍 (approve) or 👎 (reject) reactions
   - Voting typically lasts 3 days or until consensus is reached

4. **Decision:**
   - If approved, the RFC is converted into an Architecture Decision Record (ADR)
   - If rejected, the issue is closed but can be reopened if new arguments arise

## Architecture Decision Records (ADRs)

When an RFC is accepted, it is documented as an ADR:

1. **Location:**

   - Engineering-wide decisions: `/adr/`
   - Product-specific decisions: `/product-foo/`

2. **Format:**
   - Follow the standard ADR template (see [ADR 0001](adr/0001-standardise-rfc-and-adr-templates.md))
   - Use 4-digit numbering (e.g., `0001-standardise-rfc-and-adr-templates.md`)
   - Include clear status tracking (Accepted, Superseded, or Deprecated)

## Formatting, Tooling, and Style Guidelines

To ensure consistency and quality in our ADR and RFC documents, we use automated formatting tools and enforce style guidelines. This requires Node.js and some dependencies.

**Prerequisites:**

- **Node.js:** Version 22 or 23. (Use a version manager like [fnm](https://github.com/Schniz/fnm) or [nvm](https://github.com/nvm-sh/nvm).)
- **pnpm:** For managing dependencies. Install via Homebrew: `brew install pnpm`, or see the [pnpm installation guide](https://pnpm.io/installation).

**Setup Steps:**

1. Clone your fork of the repository.
2. Install dependencies with:
   ```sh
   pnpm install
   ```

**Formatting and Style:**

- We use [Prettier](https://prettier.io/) for formatting all documentation files.
- **`lint-staged`** and **`husky`** are used to automatically format files on commit, ensuring code quality.
  - **Pre-commit:** Runs `lint-staged` (which executes `prettier`) on staged files.
- To manually format all documentation files, run:
  ```sh
  pnpm format
  ```

## Code of Conduct

We expect all contributors and participants to adhere to our [Code of Conduct](CODE_OF_CONDUCT.md). Please ensure you are familiar with its contents and follow the guidelines for reporting any violations.
