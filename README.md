# GitHub Actions CI/CD Pipeline

![CI](https://github.com/shivaram-soma/github-actions-pipeline/actions/workflows/ci.yml/badge.svg)
![Docker](https://img.shields.io/badge/docker-ready-blue?logo=docker)
![GitHub Actions](https://img.shields.io/badge/github%20actions-automated-brightgreen?logo=githubactions)

A hand-crafted GitHub Actions CI/CD pipeline that triggers on every push to `main`, builds a Docker image tagged with the commit SHA, spins up a multi-container smoke-test environment, and verifies live endpoints — catching broken builds before they ship.

## What the Pipeline Does

```
push to main
    │
    ├─ 1. Checkout code
    ├─ 2. docker build -t flask-app:<SHA> .
    ├─ 3. Start Redis + App on isolated network
    ├─ 4. Smoke test /health and /count
    ├─ 5. Dump container logs on failure
    └─ 6. Cleanup
```

## Why Commit-SHA Tagging?

Using `flask-app:${{ github.sha }}` instead of `:latest`:
- Every build is uniquely identified and traceable to a commit
- Safe to run multiple builds in parallel without tag collisions
- Makes rollbacks trivial — you know exactly which image maps to which commit

## Pipeline Features

| Feature | Implementation |
|---------|---------------|
| Trigger | `push` and `pull_request` to `main` |
| Build traceability | Docker image tagged with `github.sha` |
| Network isolation | Custom bridge network `smoke-net` |
| Race condition prevention | `sleep 8` after container start |
| Debuggability | `if: failure()` logs dump |
| Guaranteed cleanup | `if: always()` teardown step |

## File Structure

```
github-actions-pipeline/
├── .github/
│   └── workflows/
│       ├── ci.yml          # Main CI pipeline (build + smoke test)
│       └── pr-checks.yml   # PR quality gates (lint, security scan)
├── .gitignore
└── README.md
```

## How to Use This Pipeline in Your Own Repo

1. Copy `.github/workflows/ci.yml` into your repository
2. Update the Docker build context if your `Dockerfile` is not at root
3. Adjust the smoke-test URLs to match your app's endpoints
4. Push to `main` and watch the **Actions** tab

## Try Breaking It

Push a commit with a syntax error in `app.py`:

```python
# Intentional error
def index(
    return "broken"
```

Watch the Actions tab go red → read the container logs step → fix → push → green.
This is the whole point: the pipeline catches what unit tests can't (a broken `CMD` in Dockerfile).

## Extending the Pipeline

```yaml
# Add to ci.yml — security scanning with Trivy
- name: Scan image for vulnerabilities
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: flask-app:${{ github.sha }}
    severity: CRITICAL,HIGH
    exit-code: 1
```

```yaml
# Add deploy step after smoke test passes
- name: Deploy to production
  if: github.ref == 'refs/heads/main'
  run: curl -X POST ${{ secrets.DEPLOY_HOOK }}
```
