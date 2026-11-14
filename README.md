# 🛰️ Sentinel-Orbit — Autonomous Fleet Upstream Sync & Storage Hygiene Satellite

[![Ecosystem](https://img.shields.io/badge/Ecosystem-Sentinel--Labs-blue?style=for-the-badge&logo=shield)](https://github.com/varunagarwal-pro)
[![Security](https://img.shields.io/badge/Security-Isolated%20PAT%20Fortress-red?style=for-the-badge&logo=securityscorecard)](https://github.com/varunagarwal-pro/Sentinel-Orbit)
[![Runner-Cost](https://img.shields.io/badge/Runner%20Cost-0%20Min%20(Free%20Tier)-brightgreen?style=for-the-badge&logo=githubactions)](https://github.com/varunagarwal-pro/Sentinel-Orbit/actions)
[![Fleet-Scope](https://img.shields.io/badge/Fleet%20Scope-47%20Repositories-orange?style=for-the-badge&logo=git)](https://github.com/varunagarwal-pro)
[![Upstream-Sync](https://img.shields.io/badge/Upstream%20Sync-41%20Enrolled%20Repos-blueviolet?style=for-the-badge&logo=gitkraken)](https://github.com/varunagarwal-pro/Sentinel-Orbit/actions/workflows/fleet-upstream-sync.yml)
[![Status](https://img.shields.io/badge/Status-Active%20Production-success?style=for-the-badge)](https://github.com/varunagarwal-pro/Sentinel-Orbit/actions)

> **📌 Document Authority:** Canonical architecture, operational runbook, and security boundary for **Sentinel-Orbit**.  
> **📅 Version:** 2.0.0 (Unified Twin-Engine Production Architecture)  
> **👤 Lead Maintainer:** Varun Agarwal (Sentinel Labs)

---

## 🌌 1. System Overview & Architectural Purpose

**`Sentinel-Orbit`** serves as the autonomous, serverless satellite orbiting the entire **Sentinel Labs** superproject ecosystem. Running inside GitHub Actions on a public repository, it leverages **100% free and unlimited public runner minutes** to execute two mission-critical automated pipelines across every repository under [`@varunagarwal-pro`](https://github.com/varunagarwal-pro):

1. **🛰️ Engine 1: Fleet Upstream Synchronization (`fleet-upstream-sync.yml`)**  
   Continuously monitors, mirrors, and stages incoming code from upstream open-source projects across 41 enrolled repositories with zero risk to local customizations.
2. **🧹 Engine 2: Account Storage & Workflow Run Pruner (`fleet-storage-prune.yml`)**  
   Enforces a strict 0 MB GitHub Actions cache ceiling and purges historical run backlogs across all 47 account repositories.

```
                                  ╔═════════════════════════════════════════════╗
                                  ║               SENTINEL-ORBIT                ║
                                  ║    (Autonomous Zero-Cost Fleet Satellite)   ║
                                  ╚═════════════════════════════════════════════╝
                                                         │
                                    [ 100% Free & Unlimited Actions Minutes ]
                                                         │
                   ┌─────────────────────────────────────┴─────────────────────────────────────┐
                   │                                                                           │
                   ▼                                                                           ▼
 ┌───────────────────────────────────────────┐               ┌───────────────────────────────────────────┐
 │ 🛰️ ENGINE 1: FLEET UPSTREAM SYNC          │               │ 🧹 ENGINE 2: STORAGE & RUN PRUNER         │
 │ • Daily Schedule: 02:17 UTC (07:47 IST)   │               │ • Trigger: Event-Chained (workflow_run)   │
 │ • 41 Enrolled Fleet Repositories          │──────────────►│ • Fallback Schedule: 04:00 UTC (09:30 IST)│
 │ • Tiers A, B, C & X Integration           │  (Auto-fires  │ • Dynamically Scans 100% Account Repos    │
 │ • Topic Branch: upstream-try-merge        │  on complete) │ • 0 MB Actions Cache Ceiling              │
 │ • Dispatches Alerts to @VarunOpsBot       │               │ • In-Flight Run Immunity Shield           │
 └───────────────────────────────────────────┘               └───────────────────────────────────────────┘
```

> [!IMPORTANT]
> **Zero Billable Runner Minute Impact:**  
> By deploying `Sentinel-Orbit` as a dedicated public satellite, GitHub Actions provides **unlimited, 100% free runner minutes**. Zero minutes are consumed from your private 2,000-minute monthly organization allowance, reserving private quotas exclusively for application builds, tests, and deployments.

---

## 🛰️ 2. Engine 1: Fleet Upstream Sync Architecture

The Upstream Sync Engine operates in accordance with the master governance policy defined in `Sentinel-Core/Docs/Upstream-Repo-Sync.md`. It dynamically checks out the Python fleet engine and `upstream/fleet.toml` from `Sentinel-Labs` using secret `FLEET_SYNC_PAT` and executes preflight AST linting and unit tests before orchestrating fleet changes.

### 🏢 Four-Tier Repository Governance Matrix

| Tier | Role | Count | Synchronization Strategy | Operational Guardrails |
| :---: | :--- | :---: | :--- | :--- |
| **Tier A** | **Pure Mirrors** | **32** | Fast-forward only directly to upstream HEAD when `ahead == 0`. | Zero local custom code. GitHub Actions are disabled on mirrors to prevent duplicate CI runs and noise. |
| **Tier B** | **Curated Downstreams** | **2** | Force-reset mirror branch to upstream (`upstream-mirror`). Working branch (`dev`/`main`) is never touched. | Triage issue tracks pending commits against a cryptographic SHA ledger (`upstream/ledgers/`). Ports applied by hand. |
| **Tier C** | **Maintained Custom Forks** | **6** | Trial merge (`git merge-tree`). Clean merges push to topic branch `upstream-try-merge` and open a PR. | Nothing automatic ever writes to working branches. Gate workflow must pass. Branch automatically deleted upon merge. |
| **Tier X** | **Frozen / Isolated** | **1** | Excluded from upstream automation (e.g. `Blokada-TV`). | Independent codebase; unconfirmed or archived upstream. |

### 🌿 The `upstream-try-merge` Lifecycle

For Tier C repositories (`Technitium-DnsServer`, `uno-launchpad-kit`, `firestack`, `Technitium-DNS-Companion`, `technitium-webui`, `nextcloud-integration`), incoming code is staged on a flat, dedicated topic branch:

```
upstream (remote) ────► upstream-mirror (1:1 pure mirror, reset-only)
                              │
                              ▼  (trial merge with working branch)
                         upstream-try-merge (clean staging branch)
                              │
                              ▼  (pull request + CI gate verification)
                         main / dev (your production branch with custom commits)
```

* **Automated Branch Cleanup:** As soon as an `upstream-try-merge` Pull Request is reviewed and merged into your working branch, the branch is **automatically deleted** from GitHub via engine-side API checks and native `delete-branch-on-merge` settings.
* **Alerting & Digest:** Results are reported in real-time to `@VarunOpsBot` via Telegram with explicit status badges (`UP-TO-DATE`, `SYNCED`, `PR-OPEN`, `CONFLICT`, `PENDING`).

---

## 🧹 3. Engine 2: Account Storage & Run Hygiene

The Storage & Run Pruner runs serverless across all **47 repositories** under [`@varunagarwal-pro`](https://github.com/varunagarwal-pro) using Python 3.11 with 64-bit multi-threaded rate-limited execution.

### 🎯 Core Capabilities:
* **0 MB Cache Ceiling:** Purges expired and stale GitHub Actions caches across all repositories, maintaining a clean storage quota.
* **Latest Run Preservation:** Groups historical runs by workflow ID and preserves the **single latest completed run** for every active workflow, cleanly deleting obsolete historical backlogs.
* **In-Flight Safety Shield:** Actively detects `in_progress` and `queued` runs and marks them **strictly immune** from deletion, ensuring ongoing CI/CD jobs are never disturbed.
* **Event-Chained Trigger:** Automatically executes immediately upon completion of the Upstream Sync Engine, ensuring newly generated sync run logs and build caches are swept without waiting.

---

## ⏱️ 4. Execution Schedules & Trigger Modes

| Pipeline | Workflow File | Trigger Types | Original Schedule (UTC) | Indian Standard Time (IST, UTC+5:30) |
| :--- | :--- | :--- | :--- | :--- |
| **Upstream Sync Engine** | `fleet-upstream-sync.yml` | Scheduled Cron + Manual Dispatch | `17 2 * * *` (Daily) | **07:47 AM IST (Daily)** |
| **Storage & Run Pruner** | `fleet-storage-prune.yml` | Event Chained (`workflow_run`) + Daily Cron + Manual Dispatch | `0 4 * * *` (Daily fallback) | **09:30 AM IST (Daily fallback)**<br>*(or immediately after sync)* |

### Manual Dispatch Parameters:

#### 1. Upstream Sync Engine:
* `mode`: `apply` (writes mirrors, opens PRs/issues) or `check` (read-only audit dry run).
* `repo`: Optional single repository name to target (leave blank to run the entire 41-repo fleet).

#### 2. Storage & Run Pruner:
* `dry_run`: `true` for simulated non-destructive audit, `false` for live purge.
* `keep_runs`: Number of completed runs to retain per workflow (default: `1`).

---

## 🔒 5. Security Boundary & Access Isolation

`Sentinel-Orbit` operates under strict least-privilege security controls:

* **Engine & Policy Isolation:** Source code for the fleet engine and `fleet.toml` live securely in the private `Sentinel-Labs` repository. `Sentinel-Orbit` checks them out into ephemeral runner memory using secret `FLEET_SYNC_PAT` and never retains copies in public storage.
* **Granular Secrets:**
  * `FLEET_SYNC_PAT`: Scoped Fine-Grained Personal Access Token with repository contents and pull request permissions.
  * `PAT_ORBIT` / `PAT_JANITOR`: Dedicated token scoped exclusively for Actions cache and run pruning.
  * `VARUN_OPS_BOT_TOKEN` & `TELEGRAM_CHAT_ID`: Encrypted bot credentials for Telegram dispatch.
* **External PR Isolation:** Workflows in this repository ignore external pull requests (`pull_request`), physically barring third-party code from accessing runner secrets.

---

## 🤝 6. Governance & Master Superproject Linkage

`Sentinel-Orbit` is an autonomous operational satellite to the master [**Sentinel-Labs**](https://github.com/varunagarwal-pro/Sentinel-Labs) ecosystem.

* **Master Upstream Specification:** `Sentinel-Core/Docs/Upstream-Repo-Sync.md`
* **Fleet Configuration:** `Sentinel-Labs/upstream/fleet.toml`
* **Fleet Engine Source:** `Sentinel-Labs/tools/upstream/`
* **Local Pruner Script:** `Sentinel-Labs/tools/prune_account_storage.py`
