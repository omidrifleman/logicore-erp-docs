# First-time GitHub publish

Local repo is ready at `logicore-erp-docs/` (commit `1cbfe75`).

## One-time setup

```powershell
# 1. Authenticate GitHub CLI
gh auth login

# 2. Create public repo and push
cd C:\Users\Omirax\Desktop\logicore-erp-docs
gh repo create logicore-erp-docs --public `
  --description "Public LogiCore ERP documentation (specs, design references)" `
  --source=. --remote=origin --push
```

If the repo already exists on GitHub:

```powershell
cd C:\Users\Omirax\Desktop\logicore-erp-docs
git push -u origin main
```

## Ongoing sync (from frontend repo)

```powershell
cd C:\Users\Omirax\Desktop\Shipping
.\scripts\sync-docs.ps1 -Checkpoint "speckit-plan"
```
