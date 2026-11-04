# 🧹 GitHub Actions Account Janitor

An automated, serverless maintenance engine that dynamically cleans GitHub Actions caches and prunes historical workflow runs across all repositories owned by `@varunagarwal-pro`.

---

## 🎯 Purpose & Features

- **Storage Quota Protection:** Automatically deletes GitHub Actions caches across all repositories, maintaining **0 MB** cache quota usage so free tier accounts never exhaust storage limits.
- **Run History Hygiene:** Prunes older, completed historical workflow runs while strictly preserving the **single latest run** for each workflow.
- **In-Flight Safety:** Never touches runs with status `in_progress` or `queued`.
- **Dynamic Discovery:** Automatically discovers 100% of repositories (public, private, sovereign, forks, and newly created repositories) without requiring any manual configuration.
- **Zero Cost:** Hosted in a dedicated public repository to leverage **unlimited, free GitHub Actions runner minutes** without deducting from private quota pools.

---

## 🔒 Security Architecture (Zero-Code Access)

This repository operates under a strict **Least-Privilege Security Contract**:
1. **Fine-Grained PAT (v2):** Uses a scoped token with strictly **`Actions: Read & Write`** and **`Metadata: Read-Only`**.
2. **Zero Code Access:** The token has **NO** `Contents` permission. It is physically barred by GitHub's API from reading, cloning, or modifying source code in any repository.
3. **Zero Credential Access:** The token has **NO** `Secrets` permission. It cannot view, extract, or alter any repository secrets.
4. **Fork PR Isolation:** Secrets are never passed to external pull request workflows.
5. **Restricted Execution:** Workflows only trigger via internal schedule (`cron`) or authorized manual dispatch (`workflow_dispatch`).

---

## ⚙️ Workflow Schedule

- **Automated Cron:** Runs daily at `04:00 UTC` (9:30 AM IST).
- **Manual Trigger:** Supports `workflow_dispatch` with options for dry-run simulation or custom run retention counts.
