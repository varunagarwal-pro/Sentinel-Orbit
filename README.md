# 🛰️ Sentinel Orbit — Autonomous Fleet Storage & Workflow Hygiene Engine

[![Ecosystem](https://img.shields.io/badge/Ecosystem-Sentinel--Labs-blue?style=for-the-badge&logo=shield)](https://github.com/varunagarwal-pro)
[![Security](https://img.shields.io/badge/Security-Zero--Code%20Access%20Fortress-red?style=for-the-badge&logo=securityscorecard)](https://github.com/varunagarwal-pro/Sentinel-Orbit)
[![Runner-Cost](https://img.shields.io/badge/Runner%20Cost-0%20Min%20(Free%20Tier)-brightgreen?style=for-the-badge&logo=githubactions)](https://github.com/varunagarwal-pro/Sentinel-Orbit/actions)
[![Fleet-Scope](https://img.shields.io/badge/Fleet%20Scope-45%2B%20Repositories-orange?style=for-the-badge&logo=git)](https://github.com/varunagarwal-pro)
[![Status](https://img.shields.io/badge/Status-Active%20Production-success?style=for-the-badge)](https://github.com/varunagarwal-pro/Sentinel-Orbit/actions)

> **📌 Document Authority:** Canonical architecture, security boundary, and operational guide for **Sentinel-Orbit**.  
> **📅 Version:** 1.0.0 (Production Release)  
> **👤 Lead Maintainer:** Varun Agarwal (Sentinel Labs)

---

## 🌌 1. System Overview & Architectural Purpose

**`Sentinel-Orbit`** is the autonomous, zero-cost maintenance satellite orbiting the entire **Sentinel Labs** multi-cloud and repository ecosystem. It runs serverless inside GitHub Actions, executing daily hygiene and storage quota enforcement across every repository owned by [`@varunagarwal-pro`](https://github.com/varunagarwal-pro).

```
                                  ╔═════════════════════════════════════════╗
                                  ║             SENTINEL-ORBIT              ║
                                  ║   (Autonomous Public Fleet Satellite)   ║
                                  ╚═════════════════════════════════════════╝
                                                       │
                           [ 100% Free & Unlimited Actions Runner Minutes ]
                                                       │
                  ┌────────────────────────────────────┼────────────────────────────────────┐
                  │                                    │                                    │
                  ▼                                    ▼                                    ▼
    ┌───────────────────────────┐        ┌───────────────────────────┐        ┌───────────────────────────┐
    │    DYNAMIC FLEET DISCOVERY│        │   SURGICAL RUN HYGIENE    │        │  CACHE QUOTA ENFORCEMENT  │
    │  • Dynamic API Pagination │        │  • In-Flight Run Immune   │        │  • 0 MB Cache Footprint   │
    │  • Public & Private Repos │        │  • Preserves Latest Run   │        │  • Daily Eviction Cycle   │
    │  • 45+ Repos Auto-Detected│        │  • Historical Run Pruner  │        │  • Prevents Quota Lockout │
    └───────────────────────────┘        └───────────────────────────┘        └───────────────────────────┘
```

> [!IMPORTANT]
> **Zero Runner Minute Deductions:**  
> By deploying `Sentinel-Orbit` as a standalone public repository, GitHub Actions provides **unlimited, 100% free runner execution minutes**. Zero minutes are consumed from your account's private 2,000-minute monthly allocation, reserving all private runner quotas exclusively for production builds, Docker packaging, and application testing.

---

## 🎯 2. Core Operational Capabilities

| Capability | Technical Implementation | Operational Guarantee |
|---|---|---|
| **Zero-Configuration Discovery** | Queries `GET /user/repos?per_page=100&affiliation=owner` with full automatic pagination. | Any new repository, fork, or experiment created in the account is automatically discovered and cleaned on the next run with **zero manual registration**. |
| **Actions Cache Zero-Quota** | Calls `GET /repos/{owner}/{repo}/actions/caches` followed by targeted `DELETE` operations. | Maintains a **0 MB** cache footprint across all repositories, preventing GitHub Actions storage quota exhaustion on free accounts. |
| **Surgical Run Preservation** | Groups historical runs by `workflow_id`, sorting chronologically descending (`created_at`). | Strictly preserves the **single latest completed run** for every active workflow. Historical backlogs are pruned cleanly. |
| **In-Flight Safety Shield** | Inspects run lifecycle status flags (`in_progress`, `queued`). | Any running build or pending queue item is **strictly immune** from deletion. Active deployments are never disrupted. |
| **Abuse-Rate Pacing** | Token-bucket pacing (`0.08s` sleep between mutations) with backoff. | Operates well below GitHub secondary API rate limits, avoiding HTTP 429 throttling during multi-hundred run purges. |
| **Rich Audit Telemetry** | Emits live Markdown dashboards directly into `$GITHUB_STEP_SUMMARY`. | Provides full transparency with a collapsible per-repository audit table showing freed MBs, purged caches, and pruned runs. |

---

## 🔒 3. Zero-Code-Access Security Fortress

`Sentinel-Orbit` is architected under the **Principle of Least Privilege**. It operates using a Fine-Grained Personal Access Token (PAT v2) named **`PAT_JANITOR`** (or `PAT_ORBIT`) configured with mathematical isolation against source code access:

```
┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│                           SENTINEL-ORBIT SECURITY ACCESS CONTRACT                           │
└─────────────────────────────────────────────────────────────────────────────────────────────┘
  ├── ✅ Actions (Read & Write)     ➔  Required to list and delete workflow runs and Actions caches
  ├── ✅ Metadata (Read-Only)       ➔  Mandatory GitHub API gateway resource routing
  ├── ❌ Contents (NO ACCESS)       ➔  BLOCKED: Cannot read, clone, view, or write repository code
  ├── ❌ Secrets (NO ACCESS)        ➔  BLOCKED: Cannot view, access, or modify repository secrets
  ├── ❌ Administration (NO ACCESS) ➔  BLOCKED: Cannot delete, rename, or modify repository settings
  └── ❌ Pull Requests (NO ACCESS)  ➔  BLOCKED: Zero interaction with developer pull requests
```

> [!NOTE]
> **External Fork Isolation:**  
> In accordance with GitHub enterprise security standards, workflows running in this repository do not listen to external pull requests (`pull_request`). GitHub physically bars external fork PRs from accessing repository secrets. Only authorized scheduled crons and authenticated owner dispatches can trigger `Sentinel-Orbit`.

---

## ⚙️ 4. Execution Schedule & Manual Controls

`Sentinel-Orbit` runs on an automated daily clock and supports on-demand manual triggers via `workflow_dispatch`:

```yaml
on:
  schedule:
    # Daily at 04:00 UTC (9:30 AM IST) - 1 hour after upstream sync
    - cron: '0 4 * * *'
  workflow_dispatch:
    inputs:
      dry_run:
        description: 'Simulate audit without mutating remote resources'
        type: boolean
        default: false
      keep_runs:
        description: 'Number of latest completed runs to preserve per workflow'
        type: number
        default: 1
```

### Manual Trigger Instructions:
1. Navigate to the [**Sentinel-Orbit Actions Tab**](https://github.com/varunagarwal-pro/Sentinel-Orbit/actions).
2. Select **`🛰️ Sentinel-Orbit: Fleet Storage & Run Pruner`**.
3. Click **"Run workflow"**.
4. *(Optional)* Check `dry_run` for a non-destructive simulation or change `keep_runs` to retain more historical runs.

---

## 📊 5. Verified Deployment Metrics

During initial deployment and live verification, `Sentinel-Orbit` purged the existing historical backlog across the account:

- **Total Repositories Processed:** 45 repositories
- **Total Actions Caches Purged:** 26 caches
- **Account Cache Storage Freed:** **1,387.58 MB (~1.36 GB)** $\rightarrow$ Reclaimed to **0 MB**
- **Historical Workflow Runs Purged:** 644 historical runs
- **Active Workflow Runs Preserved:** 34 latest runs
- **Runner Execution Duration:** 36 seconds
- **Actions Minutes Incurred:** **0 minutes** (100% Free)

---

## 🤝 6. Governance & Submodule Matrix

`Sentinel-Orbit` functions as an operational satellite to the master [**Sentinel-Labs**](https://github.com/varunagarwal-pro/Sentinel-Labs) ecosystem. Its core script is mirrored into the centralized tools directory at `tools/prune_account_storage.py` for local and fleet orchestration.
