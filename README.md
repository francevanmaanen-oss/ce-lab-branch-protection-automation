# Lab M5.04 - Branch Protection & PR Automation

## What This Repository Demonstrates

### Branch Protection
- Direct pushes to `main` are blocked
- PRs require status checks to pass before merge
- At least one approving review is required

### CI Pipeline (`.github/workflows/ci.yml`)
- `terraform fmt -check` — enforces consistent formatting
- `terraform init` — validates provider configuration
- `terraform validate` — checks syntax and internal consistency

### Auto-Labeler (`.github/workflows/labeler.yml`)
- `.tf` files → `terraform` label
- `.md` files → `docs` label
- `.github/workflows/` → `ci` label

### PR Checklist Bot (`.github/workflows/pr-checklist.yml`)
- Posts a structured checklist on every new PR
- Author and reviewer sections

### CODEOWNERS (`.github/CODEOWNERS`)
- Terraform files require platform team review
- Workflow files require DevOps lead review

## Key Learnings
- Branch protection prevents accidental direct pushes
- Required status checks enforce code quality gates
- CODEOWNERS automates reviewer assignment
- Auto-labeling improves PR triage at scale

## Errors Encountered & How We Fixed Them

### 1. Branch Protection API — Invalid Request (HTTP 422)
**Problem:** The `gh api` command used `--field` to pass nested JSON objects for
`required_status_checks` and `required_pull_request_reviews`. The GitHub API received
these as plain strings instead of objects, causing a schema validation failure.

**Fix:** Replaced `--field` with `--input -` and passed the full request body as a
JSON heredoc, which correctly serializes nested objects:
```bash
gh api repos/{owner}/{repo}/branches/main/protection \
  --method PUT \
  --header "Content-Type: application/json" \
  --input - <<'EOF'
{ ... }
EOF
```

### 2. Terraform Format Check Failing in CI (Exit Code 3)
**Problem:** The CI workflow failed at the `terraform fmt -check -recursive` step with
exit code 3, meaning one or more `.tf` files had formatting that didn't match
Terraform's canonical style. This was caused by inconsistent indentation introduced
when appending the DynamoDB resource block using a `cat` heredoc.

**Fix:** Ran `terraform fmt -recursive` locally to auto-correct all formatting, then
committed and pushed the result. The CI re-ran and the format check passed.
```bash
terraform fmt -recursive
git add .
git commit -m "Fix Terraform formatting"
git push
```
