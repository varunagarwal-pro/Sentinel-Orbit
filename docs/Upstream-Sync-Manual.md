# 📖 Fleet Upstream Synchronization Manual — Repository Enrollment & Tier Management Guide

[![Ecosystem](https://img.shields.io/badge/Ecosystem-Sentinel--Labs-blue?style=for-the-badge&logo=shield)](https://github.com/varunagarwal-pro)
[![Architecture](https://img.shields.io/badge/Architecture-Twin--Engine%20Satellite-purple?style=for-the-badge&logo=gitkraken)](https://github.com/varunagarwal-pro/Sentinel-Orbit)
[![Governance](https://img.shields.io/badge/Governance-Tiered%20Upstream%20Sync-success?style=for-the-badge)](https://github.com/varunagarwal-pro/Sentinel-Core)
[![Version](https://img.shields.io/badge/Version-1.0.0-informational?style=for-the-badge)](https://github.com/varunagarwal-pro)

> **📌 Target Systems:** Sentinel-Labs Superproject & Sentinel-Orbit Satellite  
> **📅 Version:** 1.0.0 (Production Operational Standard)  
> **👤 Lead Maintainer:** Varun Agarwal (Sentinel Labs)

---

## 🧭 1. Architectural Philosophy & The Private/Public Split

The Sentinel-Labs upstream synchronization architecture uses a deliberate **two-repository boundary design**:

```
╔═════════════════════════════════════════╗       Checkout over PAT       ╔═════════════════════════════════════════╗
║         SENTINEL-LABS (Private)         ║ ────────────────────────────► ║        SENTINEL-ORBIT (Public)          ║
║  • upstream/fleet.toml (Source of Truth)║                               ║  • .github/workflows/                   ║
║  • tools/upstream/ (Sync Engine & Lint) ║                               ║    - fleet-upstream-sync.yml            ║
║  • upstream/ledgers/ (Audit History)    ║                               ║    - fleet-storage-prune.yml            ║
║  • Holds Private Topology & Decisions   ║                               ║  • 100% Free Unlimited Runner Minutes   ║
╚═════════════════════════════════════════╝                               ╚═════════════════════════════════════════╝
```

### Why this split exists:
1. **Privacy & Security:** The master list of tracked projects, custom forks, proprietary downstreams, and cryptographic patch decisions remains **strictly confidential** inside `Sentinel-Labs`.
2. **Zero Runner Minute Cost:** The execution runner runs serverless inside the public `Sentinel-Orbit` repository, consuming **0 minutes** from your private 2,000-minute monthly GitHub Actions allowance.

> [!IMPORTANT]
> **The Golden Rule of Upstream Sync:**  
> *Nothing automatic ever writes directly to a customized working branch.* Upstream code is either fast-forwarded on pure mirrors (Tier A), isolated on disposable trial branches with pull requests (Tier C), or triaged manually via cryptographic SHA ledgers (Tier B).

---

## 🗂️ 2. Which Tier Should You Choose?

Before adding a repository, determine its tier using this decision tree:

```
                          Is this repository a fork?
                                     │
                    ┌────────────────┴────────────────┐
                   YES                                NO
                    │                                 │
    Do you have any custom code,                      ▼
    patches, or branding changes?             [ Tier X / Standalone ]
                    │                         (Excluded from automated sync)
            ┌───────┴───────┐
            NO             YES
            │               │
            ▼               ▼
      [ Tier A ]   Can upstream changes be safely
    (Pure Mirror)  trial-merged via automated PRs?
                            │
                    ┌───────┴───────┐
                   YES              NO
                    │               │
                    ▼               ▼
              [ Tier C ]       [ Tier B ]
          (Maintained PRs)  (Curated Downstream)
```

---

## 🚀 3. Tier A: Pure Mirrors (100% Vanilla Forks)

### When to use:
* You forked an open-source project strictly for backup, personal hosting, reference, or distribution.
* You have **zero custom commits** on the repository.
* *Examples:* `zigzag-embroidery`, `IDM-Download-Manager`, `SmartTube`, `AdGuardHome`, `mitmproxy`.

---

### Step-by-Step Enrollment:

#### Step 1: Fork or Create the Repository on GitHub
Create the fork under your account: `https://github.com/varunagarwal-pro/<your-fork-name>`.

#### Step 2: Disable Inherited GitHub Actions (Mandatory)
Upstream repositories often have heavy CI/CD workflows (Pages deployments, release builders, test suites) that will fail in your fork or burn actions time. Disable Actions on the fork via GitHub CLI:

```powershell
gh api -X PUT repos/varunagarwal-pro/<your-fork-name>/actions/permissions -F enabled=false
```

#### Step 3: Enable Auto-Deletion of Head Branches (If Public)
Ensure merged branches are cleanly removed:

```powershell
gh repo edit varunagarwal-pro/<your-fork-name> --delete-branch-on-merge
```

#### Step 4: Add Entry to `upstream/fleet.toml` in `Sentinel-Labs`
Open `D:\Sentinel-Labs\upstream\fleet.toml` and append your repository under `# Tier A`:

```toml
[[repo]]
name = "<your-fork-name>"
tier = "A"
upstream = "<upstream-owner>/<upstream-repo>"
fork_branch = "main"   # Set to "master" if upstream's default branch is master
```

*Example for `zigzag-embroidery`:*
```toml
[[repo]]
name = "zigzag-embroidery"
tier = "A"
upstream = "simpsoka/zigzag"
fork_branch = "main"
```

#### Step 5: Validate Policy & Commit
Run the fleet linter locally to verify TOML syntax:

```powershell
python tools/upstream/cli.py lint .
```

If it prints `lint clean`, commit and push to `Sentinel-Labs`:

```powershell
git add upstream/fleet.toml
git commit -m "feat(fleet): enroll <your-fork-name> as Tier A pure mirror"
git push origin main
```

---

## 🛠️ 4. Tier C: Maintained Custom Forks (Automated PR Staging)

### When to use:
* You maintain custom changes, configs, or features on `main`, `master`, or `dev`.
* You want upstream changes to be **trial-merged automatically**, tested in CI, and delivered to you as a **clean Pull Request**.
* *Examples:* `Technitium-DnsServer`, `uno-launchpad-kit`, `firestack`, `Technitium-DNS-Companion`, `technitium-webui`.

---

### How Tier C Works:
1. Every night, the engine performs a clean `git merge-tree` trial merge between upstream's HEAD and your working branch.
2. If clean, it pushes to topic branch **`upstream-try-merge`** and opens/refreshes a **single rolling Pull Request** into your working branch.
3. Your repository's gate workflow builds and tests the PR in isolation.
4. You review the PR diff and click **Merge**.
5. Once merged, the branch `upstream-try-merge` is **automatically deleted** from GitHub.
6. If there is a merge conflict, the engine opens an **Issue** listing the exact conflicting files and hotspots, keeping the workflow green.

---

### Step-by-Step Enrollment:

#### Step 1: Ensure Gate Workflow Exists in Target Repository
The target fork must contain `.github/workflows/upstream-gate.yml` on its working branch. A template is available at `upstream/templates/technitium-webui-upstream-gate.yml`:

```yaml
name: Upstream gate

on:
  pull_request:
    branches: [main]   # Or [master] / [dev]

permissions:
  contents: read

jobs:
  gate:
    if: startsWith(github.head_ref, 'upstream-try-merge') || startsWith(github.head_ref, 'sync/upstream-')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build & Test
        run: |
          # Add your project build/test command here (e.g. dotnet build, npm test, etc.)
          echo "Gate validation passed"
```

#### Step 2: Enable Branch Auto-Deletion
```powershell
gh repo edit varunagarwal-pro/<your-fork-name> --delete-branch-on-merge
```

#### Step 3: Add Entry to `upstream/fleet.toml` in `Sentinel-Labs`
Add the configuration under `# Tier C`:

```toml
[[repo]]
name = "<your-fork-name>"
tier = "C"
upstream = "<upstream-owner>/<upstream-repo>"
upstream_ref = "main"            # Upstream's primary branch
working_branch = "main"          # Your local working branch (main, master, or dev)
mirror_branch = "upstream-mirror"
gate_workflow = "upstream-gate.yml"
cadence_days = 7                 # (Optional) Minimum days between PRs
```

#### Step 4: Validate & Commit
```powershell
python tools/upstream/cli.py lint .
git add upstream/fleet.toml
git commit -m "feat(fleet): enroll <your-fork-name> as Tier C maintained fork"
git push origin main
```

---

## 🏛️ 5. Tier B: Curated Downstreams (Cryptographic Ledger Triage)

### When to use:
* Core production proprietary applications with extensive divergence from upstream.
* Automated merging is strictly forbidden; upstream features or security patches are **cherry-picked or ported by hand**.
* *Examples:* `AstraQuant` (downstream of `marketcalls/openalgo`), `StreamShield` (downstream of `celzero/rethink-app`).

---

### How Tier B Works:
1. The engine force-resets an isolated `upstream-mirror` branch to upstream's HEAD. Your working code is **never touched**.
2. Upstream commits are compared against a **cryptographic SHA ledger** in `upstream/ledgers/<Repo>.jsonl`.
3. Unreviewed commits are reported in a centralized GitHub **Triage Issue** with classified tags (`protected-path`, `likely-drop`, `security`).
4. You decide whether to **port**, **drop**, or **defer** each commit.

---

### Step-by-Step Enrollment:

#### Step 1: Create the Ledger File
Create an empty JSONL ledger file in `Sentinel-Labs`:
`upstream/ledgers/<Repo>.jsonl`

#### Step 2: Add Entry to `upstream/fleet.toml`
```toml
[[repo]]
name = "<Repo>"
tier = "B"
upstream = "<upstream-owner>/<upstream-repo>"
working_branch = "dev"
mirror_branch = "upstream-mirror"
cadence_days = 7
```

#### Step 3: Triage Commands Runbook
Use the CLI to inspect and record decisions:

```powershell
# 1. View all pending upstream commits
python tools/upstream/cli.py ledger pending <Repo>

# 2. Mark a commit as ported
python tools/upstream/cli.py ledger port <Repo> <commit-sha> --reason "Ported to module X"

# 3. Mark a commit as dropped (irrelevant upstream feature)
python tools/upstream/cli.py ledger drop <Repo> <commit-sha> --reason "Not applicable to our architecture"

# 4. Advance baseline after a major review cycle
python tools/upstream/cli.py ledger advance-baseline <Repo> <commit-sha>
```

---

## ⚡ 6. Operational Verification & Testing Cheat Sheet

### Local Commands (Inside `Sentinel-Labs`):

| Goal | Command |
| :--- | :--- |
| **Lint Policy & Syntax** | `python tools/upstream/cli.py lint .` |
| **Run Unit Tests (33 tests)** | `python -m pytest tools/upstream/tests -q` |
| **Dry-Run Audit on a Single Repo** | `python tools/upstream/cli.py fleet --mode check --repo <Repo>` |
| **Full Dry-Run on Entire Fleet** | `python tools/upstream/cli.py fleet --mode check` |
| **Apply Sync on Single Repo Locally**| `python tools/upstream/cli.py fleet --mode apply --repo <Repo>` |

### Remote GitHub Actions Commands (`Sentinel-Orbit`):

| Goal | Command |
| :--- | :--- |
| **Trigger Full Fleet Sync** | `gh workflow run fleet-upstream-sync.yml -R varunagarwal-pro/Sentinel-Orbit` |
| **Trigger Single Repo Sync** | `gh workflow run fleet-upstream-sync.yml -R varunagarwal-pro/Sentinel-Orbit -f repo=<Repo>` |
| **Trigger Read-Only Audit** | `gh workflow run fleet-upstream-sync.yml -R varunagarwal-pro/Sentinel-Orbit -f mode=check` |
| **View Latest Run Status** | `gh run list --workflow=fleet-upstream-sync.yml -R varunagarwal-pro/Sentinel-Orbit --limit 1` |
| **Watch Live Run Progress** | `gh run watch -R varunagarwal-pro/Sentinel-Orbit` |

---

## 📋 7. Summary Enrollment Checklist

```markdown
### New Fork Enrollment Checklist:
- [ ] 1. Forked repository created under @varunagarwal-pro.
- [ ] 2. Actions disabled on fork (for Tier A pure mirrors).
- [ ] 3. `delete-branch-on-merge` enabled in repository settings.
- [ ] 4. Target entry added to `upstream/fleet.toml` in Sentinel-Labs.
- [ ] 5. Verified with `python tools/upstream/cli.py lint .` (must output `lint clean`).
- [ ] 6. Committed and pushed to `Sentinel-Labs:main`.
- [ ] 7. (Optional) Dispatched manual test run in `Sentinel-Orbit`.
```
