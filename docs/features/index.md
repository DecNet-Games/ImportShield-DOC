---
layout: default
title: Features
nav_order: 5
has_children: true
permalink: /docs/features/
description: "Comprehensive feature matrix and guide to ImportShield's modules and protection engines."
---

# Feature Matrix & Capabilities

ImportShield delivers an integrated suite of twelve specialized diagnostic, recovery, and protection modules designed to guarantee project stability and eliminate import anxiety.

---

## Complete Feature Matrix

<div class="table-wrapper" markdown="block">

| Module | Core Functionality | Primary Trigger / UI Location | Key Safety Mechanism |
|:---|:---|:---|:---|
| [**Pre-Import Snapshots**]({{ site.baseurl }}/docs/features/pre-import-snapshots/) | Atomic reference graph freeze & incremental indexer | Automatic on package import / `Tools ▸ ImportShield` | GZip binary snapshots with `(path, size, mtime)` stat cache |
| [**Package Preflight**]({{ site.baseurl }}/docs/features/package-preflight/) | Sandboxed `.unitypackage` scan with GREEN / AMBER / RED verdict | `Tools ▸ ImportShield ▸ Preflight Package...` | Evaluates archive tar headers before disk modification |
| [**Blast Radius & Repair**]({{ site.baseurl }}/docs/features/blast-radius-and-repair/) | Vector node visualizer & evidence-ranked repair engine | **Changes** tab | Multi-strategy confidence scoring (R1–R7) & two-pane line diffs |
| [**Quarantine & Conflicts**]({{ site.baseurl }}/docs/features/quarantine-and-conflicts/) | Zero-silent-deletion storage & post-import edit preservation | `ProjectDoctorData/Quarantine` & `Conflicts` | Isolates files outside `Assets/` with intact GUIDs; protects user work |
| [**Protection Watchers**]({{ site.baseurl }}/docs/features/protection/) | Background event listeners & crash recovery monitors | Automatic runtime lifecycle | `ImportGuardHooks`, `CrashRecoveryWatcher`, `DurableVerificationQueue` |
| [**Safe Cleanup Engine**]({{ site.baseurl }}/docs/features/cleanup/) | Reachability analysis, duplicate merger & bloat detection | **Cleanup** tab | String-literal scanner protects `Resources.Load` & Addressables |
| [**Material Doctor**]({{ site.baseurl }}/docs/features/material-doctor/) | Pink shader healing & URP Lit property translation | **Materials** tab | Automated property mapping (`_MainTex` &rarr; `_BaseMap`, etc.) |
| [**Missing Component Stripper**]({{ site.baseurl }}/docs/features/component-stripper/) | Dead `MonoBehaviour` removal without sibling corruption | **Stripper** tab | Surgical YAML array pruning with pre-strip transaction backups |
| [**Audit Certificate**]({{ site.baseurl }}/docs/features/audit-certificate/) | Health reporting & SHA-256 digital fingerprinting | **Reports** tab | Formal cryptographic attestation of project reference integrity |
| [**Time Machine**]({{ site.baseurl }}/docs/features/time-machine/) | Snapshot scrubber & historical graph rewind | **History** tab | Instant comparison between any two historical project states |
| [**Command Line Interface (CLI)**]({{ site.baseurl }}/docs/features/cli/) | Headless batchmode runner for CI/CD gates | Terminal / CI pipeline scripts | Configurable threshold assertions with exit codes `0`, `1`, `2` |

</div>

---

## Operational Architecture: Observation, Diagnosis, Repair

All ImportShield modules operate across three strictly enforced architectural boundaries:

```
┌─────────────────────────────────────────────────────────────┐
│ 1. OBSERVATION                                              │
│ - Fast YamlLineScanner parses token streams                 │
│ - Stat-cache eliminates redundant I/O                       │
│ - In-memory GuidIndex maps GUID/fileID references           │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. DIAGNOSIS                                                │
│ - Strict Provenance Separation (New Breaks vs Pre-existing) │
│ - Cause Attribution taxonomy (C1–C8)                        │
│ - Evidence scoring & confidence categorization (R1–R7)      │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. REPAIR & VERIFICATION                                    │
│ - Atomic byte backup of targets to ProjectDoctorData/       │
│ - Journal state recorded as 'Prepared'                      │
│ - Line-level patch applied with preserved line-endings      │
│ - Immediate post-patch verification (Fail-Closed on error)  │
└─────────────────────────────────────────────────────────────┘
```

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*

---

## Detailed Feature Guides

Dive into the dedicated guides below to master each individual module:

- [**Pre-Import Snapshots**]({{ site.baseurl }}/docs/features/pre-import-snapshots/)
- [**Package Preflight**]({{ site.baseurl }}/docs/features/package-preflight/)
- [**Blast Radius & Repair**]({{ site.baseurl }}/docs/features/blast-radius-and-repair/)
- [**Quarantine & Conflicts**]({{ site.baseurl }}/docs/features/quarantine-and-conflicts/)
- [**Protection Watchers**]({{ site.baseurl }}/docs/features/protection/)
- [**Safe Cleanup Engine**]({{ site.baseurl }}/docs/features/cleanup/)
- [**Material Doctor**]({{ site.baseurl }}/docs/features/material-doctor/)
- [**Component Stripper**]({{ site.baseurl }}/docs/features/component-stripper/)
- [**Audit Certificate**]({{ site.baseurl }}/docs/features/audit-certificate/)
- [**Time Machine**]({{ site.baseurl }}/docs/features/time-machine/)
- [**Command Line Interface (CLI)**]({{ site.baseurl }}/docs/features/cli/)
