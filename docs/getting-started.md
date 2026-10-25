---
layout: default
title: Getting Started
nav_order: 2
description: "Core mental model, 3-step onboarding, baseline snapshots, and protected status in ImportShield."
permalink: /docs/getting-started/
---

# Getting Started with ImportShield

ImportShield transforms how you manage asset imports, package upgrades, and serialized references in Unity. Instead of treating imports as destructive, irreversible gambles, ImportShield wraps your project in an automated flight recorder and transactional safety net.

---

## The Mental Model

Traditional Unity workflows treat the filesystem as an unprotected workspace. When an import overwrites existing assets or introduces duplicate GUIDs, Unity's internal asset database mutates immediately, often leaving broken object pointers (`m_Script: {fileID: 0}` or null serialized references).

ImportShield operates under a strict **Observation &rarr; Diagnosis &rarr; Repair** pipeline founded on disk-level truth:

```
[ Package Import / Event ]
          │
          ▼
┌──────────────────────────┐
│   Pre-Import Freeze      │  <-- Atomic snapshot & byte backup
└─────────┬────────────────┘
          │ (Import executes)
          ▼
┌──────────────────────────┐
│   Observation Pipeline   │  <-- Incremental YAML parser & Stat-cache
└─────────┬────────────────┘
          │
          ▼
┌──────────────────────────┐
│   Change Investigator    │  <-- Provenance separation (New vs Pre-existing)
└─────────┬────────────────┘  <-- Blast-radius graph generation
          │
          ▼
┌──────────────────────────┐
│   Evidence-Ranked Engine │  <-- Multi-strategy confidence scoring (R1–R7)
└─────────┬────────────────┘
          │
          ▼
┌──────────────────────────┐
│   Fail-Closed Repair     │  <-- Two-pane line diff preview
└──────────────────────────┘  <-- Journaled byte-patching & rollback
```

1. **Snapshots are Fast and Incremental**: Stored in `Library/ProjectDoctor/Snapshots/`, snapshots use compressed binary serialization and an internal stat cache (`path, size, mtimeUtc`) to index tens of thousands of assets in sub-second time.
2. **Backups Live in Project Storage**: Rollback data and irreplaceable originals are stored in `<ProjectRoot>/ProjectDoctorData/Transactions/`, completely separate from `Library/`. They survive `Library/` deletion and editor restarts.
3. **Repairs are Previewed and Reversible**: No file is ever modified blindly. You inspect exact line-level diffs, and every transaction can be rolled back with a single click.

---

## 3-Step Onboarding

Setting up ImportShield requires zero configuration files, zero external daemons, and zero database setup.

### Step 1: Open the ImportShield Window
From the Unity Editor top menu, select:
```
Tools ▸ ImportShield ▸ ImportShield Window
```
You can also click the **🟢 ImportShield** indicator pill located in the top-right toolbar overlay of the Scene View.

### Step 2: Automatic Baseline Scan
Upon its first launch, ImportShield automatically performs an initial baseline index of your project:
- Reads all `.meta` files and serialized YAML assets (`.unity`, `.prefab`, `.asset`, `.mat`).
- Maps all GUIDs, fileIDs, script types, and dependency relationships.
- Compresses and persists this baseline state into `Library/ProjectDoctor/Snapshots/`.

### Step 3: Verify "Protected" Status
Look at the status indicator in the top header of the ImportShield Window or the Scene View overlay. When the baseline snapshot is established, the indicator illuminates green:

```
🟢 Status: Protected | Baseline: #1 (2,418 assets indexed)
```

**Congratulations!** Your project is now actively protected. From this point forward:
- Any `.unitypackage` import automatically triggers a pre-import freeze.
- Any reference breakage triggers an automatic Change Report.
- Any manual modification can be checked against your baseline.

---

## Understanding Protection States

The status header communicates the health and monitoring state of your project:

| Status | Visual | Description | Action Needed |
|:---|:---|:---|:---|
| **Protected** | 🟢 Green | Pristine baseline snapshot active; import hooks engaged; rollback journal ready. | None. Safe to import or modify assets. |
| **Indexing** | 🟡 Amber | Scanner is actively parsing changed YAML files or building an incremental snapshot. | Wait brief moments (< 1s typical) for indexing to complete. |
| **Needs Attention** | 🔴 Red | Unresolved breakages detected following a recent import or script rename. | Open the **Changes** tab to review the Blast Radius Map and preview repairs. |
| **Recovery Active** | 🔵 Blue | CrashRecoveryWatcher detected an interrupted transaction upon editor boot. | Click **Review Journal** to complete verification or perform 1-click rollback. |

---

## Next Steps

- Proceed to [**Installation**]({{ site.baseurl }}/docs/installation/) to review package distribution details and prerequisites.
- Run through the [**Quick Start (60 Seconds)**]({{ site.baseurl }}/docs/quick-start/) to test your setup with a real import.
- Explore the [**Package Preflight**]({{ site.baseurl }}/docs/features/package-preflight/) guide to test packages before importing.
