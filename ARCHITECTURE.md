# GitHub Actions Pipeline Architecture

## Workflow Overview

Developer
    ↓
Push Code
    ↓
GitHub Repository
    ↓
CI Pipeline Triggered
    ↓
Lint Checks
    ↓
Formatting Checks
    ↓
Validation
    ↓
Pass / Fail Result

## Components

### CI Workflow
Location:
.github/workflows/ci.yml

Purpose:
- Automated validation
- Continuous Integration

### PR Checks
Location:
.github/workflows/pr-checks.yml

Purpose:
- Pull Request Quality Gates
- Prevent bad code from merging
