# 08 – CI/CD

This repo uses a **lean, docs-first CI** that keeps the project clean and recruiter-friendly:

- **GitHub Actions** for fast checks on every PR and push
- **markdownlint** for style and structure
- **pre-commit** hooks for local consistency (same checks before commits)
- **Release tagging** with semantic versions (`vMAJOR.MINOR.PATCH`)
- **Protected `main`** (only via PR + green checks)

> Scope: This is a sanitized case study (no client code/data). CI focuses on documentation quality, repo hygiene, and reproducibility.

---

## 1) Branch & PR policy

- Default branch: `main`
- Flow: `feature/*` or `chore/*` → PR → checks must pass → squash-merge
- Required checks:
  - `CI / markdown-and-format (pull_request)`
  - `CI / pre-commit (pull_request)`
- No direct pushes to `main`.

---

## 2) GitHub Actions (CI)

Two lightweight workflows run on PRs and pushes:

### a) Markdown & format

- Validates all `*.md` using **markdownlint-cli2**
- Fails on broken headings, list spacing, trailing spaces, etc.

<details>
<summary>Reference YAML (minimal)</summary>

```yaml
name: CI / markdown-and-format

on:
  pull_request:
  push:
    branches: [ main ]

jobs:
  mdlint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "lts/*"
      - name: Install markdownlint-cli2
        run: npm i -g markdownlint-cli2
      - name: Run markdownlint
        run: markdownlint-cli2 "**/*.md" --config .markdownlint.jsonc
