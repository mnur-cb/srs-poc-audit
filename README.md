# NPM Audit Automated Remediation Workflow

## Executive Summary

This workflow provides a controlled, production-safe system for
automatically resolving `npm audit` vulnerabilities across all services
in the repository.

For each service, it:

1.  Removes existing `overrides` first when present, then installs dependencies (`npm install`); otherwise uses deterministic install (`npm ci`)
2.  Detects vulnerabilities
3.  Applies safe patch/minor fixes
4.  Applies force fixes only for `devDependencies`
5.  Injects dynamic overrides only for eligible transitive dependencies
6.  Blocks dependency spec downgrades by restoring original versions
7.  Validates changes using `npm run test` when a test script exists
8.  Automatically creates a Pull Request with a factual change summary
9.  Auto-merges once CI checks pass

Aggressive force fixes are restricted to `devDependencies`. Override
injection is skipped for direct production and direct dev dependencies.

------------------------------------------------------------------------

## Usage Example

Create a workflow in your repository (for example,
`.github/workflows/security-automation.yml`) and call this reusable
workflow:

```yaml
name: Security Automation

on:
  schedule:
    - cron: '0 8 * * *'
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write

jobs:
  security-fix:
    uses: mnur-cb/srs-poc-audit/.github/workflows/npm-audit-reusable.yml@main
    with:
      jira_story: SAT-258381
      enable_auto_merge: false
    secrets: inherit
```

`jira_story` is optional. If provided, it is used as a prefix in the PR
title and as the PR branch name.

------------------------------------------------------------------------

## Architecture Overview

### Remediation Flow Diagram

 <img src="./assets/mermaid-diagram.png" alt="Example image" width="600">

------------------------------------------------------------------------

## Dependency Protection Rules

| Action                         | Direct Production Dependencies | Direct Dev Dependencies |
|:-------------------------------|:-------------------------------|:------------------------|
| Safe Fix                       | ✅ Allowed                     | ✅ Allowed              |
| Force Fix                      | ❌ Blocked                     | ✅ Allowed              |
| Dynamic Overrides              | ❌ Blocked                     | ❌ Blocked              |
| Test Validation                | ✅ If test script exists       | ✅ If test script exists|
| Downgrade Guard (spec version) | ✅ Blocked                     | ✅ Blocked              |

------------------------------------------------------------------------

## Design Principles

-   Deterministic installs using `npm ci` when no existing `overrides` are present
-   Existing `overrides` are removed before remediation starts
-   No lockfile deletion
-   No override of direct production dependencies
-   No direct dependency downgrade commits (including overrides)
-   No repository pollution (temp files isolated)
-   Automatic test validation before commit
-   Transparent, factual PR summaries per service

------------------------------------------------------------------------

## Remediation Hierarchy

1.  Apply safe fixes
2.  Apply force fixes (dev-only)
3.  Apply dynamic overrides (dev-only)
4.  Roll back detected dependency downgrades
5.  Validate via tests when available
6.  Revert if unsafe
7.  Commit only validated changes

This ensures maximum automation with minimal production risk.
