# NPM Audit Automated Remediation Workflow

## Executive Summary

This workflow provides a controlled, production-safe system for
automatically resolving `npm audit` vulnerabilities across all services
in the repository.

For each service, it:

1.  Runs a deterministic install (`npm ci`)
2.  Detects vulnerabilities
3.  Applies safe patch/minor fixes
4.  Applies force fixes only for `devDependencies`
5.  Injects dynamic overrides for vulnerable transitive dev dependencies
6.  Validates changes using `npm run test`
7.  Automatically creates a Pull Request with a detailed change summary
8.  Auto-merges once CI checks pass

Aggressive fixes (force + overrides) are restricted to development
dependencies only, ensuring runtime stability.

------------------------------------------------------------------------

## Architecture Overview

### Remediation Flow Diagram

 <img src="./assets/mermaid-diagram.png" alt="Example image" width="600">

------------------------------------------------------------------------

## Dependency Protection Rules

| Action          | Production Dependencies | Dev Dependencies |
|:----------------|:------------------------|:-----------------|
| Safe Fix        | ✅ Allowed              | ✅ Allowed       |
| Force Fix       | ❌ Blocked              | ✅ Allowed       |
| Overrides       | ❌ Blocked              | ✅ Allowed       |
| Test Validation | ✅ Required             | ✅ Required      |

------------------------------------------------------------------------

## Design Principles

-   Deterministic installs using `npm ci`
-   No lockfile deletion
-   No override of direct production dependencies
-   No repository pollution (temp files isolated)
-   Automatic test validation before commit
-   Transparent PR summaries per service

------------------------------------------------------------------------

## Remediation Hierarchy

1.  Apply safe fixes
2.  Apply force fixes (dev-only)
3.  Apply dynamic overrides (dev-only)
4.  Validate via tests
5.  Revert if unsafe
6.  Commit only validated changes

This ensures maximum automation with minimal production risk.
