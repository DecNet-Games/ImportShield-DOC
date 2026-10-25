---
layout: default
title: Provenance Separation & Causes
parent: System Architecture
nav_order: 2
description: "Provenance Separation (New vs Pre-existing Breaks) and the Cause Attribution taxonomy (C1–C8)."
---

# Provenance Separation & Cause Attribution

The most frustrating aspect of troubleshooting Unity project breakages is attributing responsibility. When an asset import finishes and twenty errors appear in the console, developers cannot easily discern whether the new package caused the breakages or whether those assets were already broken weeks ago.

ImportShield solves this through strict **Provenance Separation** and a deterministic **Cause Attribution Taxonomy (C1–C8)**.

---

## Provenance Separation: New vs Pre-existing Breaks

ImportShield never blames an import for pre-existing damage. When analyzing changes, the `Differ` engine compares three distinct sets of references:

1. **Baseline Snapshot ($S_0$)**: The established pristine state of the project.
2. **Pre-Import Freeze ($S_{\text{pre}}$)**: The state immediately before the current operation.
3. **Post-Import State ($S_{\text{post}}$)**: The current state on disk.

```
       All Broken References Detected in Post-Import State
                             │
       ┌─────────────────────┴─────────────────────┐
       ▼                                           ▼
[ New Breaks ]                             [ Pre-existing Breaks ]
Reference was VALID in S_pre               Reference was ALREADY BROKEN in S_pre
Broken by the current import event         Existed prior to the current event
Target of immediate automated repairs      Grouped separately; never blamed on import
```

### Why This Separation Is Crucial
- **Eliminates Developer Panic**: You only see breakages directly attributable to the package you just imported.
- **Accurate Rollback Scopes**: Rolling back an import only restores files modified by that specific import, without touching unrelated fixes or edits made elsewhere.
- **Precise CI Gates**: Continuous integration gates can be configured to fail *only* if `new-broken-refs > 0`, ignoring legacy technical debt in older project branches.

---

## Cause Attribution Taxonomy (C1–C8)

When a reference break is identified as a **New Break**, the engine executes an 8-stage diagnostic heuristic to determine exactly *why* the link broke:

<div class="table-wrapper" markdown="block">

| Code | Classification | Formal Diagnostic Rule | Example Scenario |
|:---:|:---|:---|:---|
| **C1** | Target File Deleted | Target GUID existed in $S_{\text{pre}}$, but its file was removed from the filesystem during the import. | An asset pack cleanup script removed a required texture. |
| **C2** | Overwritten with New GUID | A file exists at the expected path, but its `.meta` file contains a different GUID in $S_{\text{post}}$ than in $S_{\text{pre}}$. | An imported package overwrote a customized script `.meta` file. |
| **C3** | Class Renamed or Removed | The `.cs` source file exists on disk, but the class name or namespace matching the fileID was altered. | A script author renamed `PlayerMotor` to `CharacterMotor`. |
| **C4** | Assembly Dependency Changed | The referenced assembly definition (`.asmdef`) was deleted, re-scoped, or had a dependency reference removed. | A package moved internal classes into an isolated runtime assembly. |
| **C5** | Prefab Variant Base Modified | A prefab variant's parent GUID changed or was disconnected. | An updated package replaced a base model prefab with a new variant. |
| **C6** | Blocked by Compile Error | Target script source code is intact on disk, but compiler errors in *other* scripts prevent assembly generation. | A missing package dependency produces compiler errors that block unrelated scripts. |
| **C7** | Version Upgrade Serialization | Property structure or fileID changed due to Unity engine version upgrade. | Upgrading from Unity 2022.3 to Unity 6 altered a component's internal serialization layout. |
| **C8** | Unknown / Complex Cause | Reference cannot be attributed to C1–C7 heuristics. | Highly complex multi-asset circular dependencies (< 5% of cases). |

</div>

---

## The Value of Cause C6 ("Blocked, Not Missing")

In standard Unity, if a single script in your project contains a syntax error, Unity cannot compile scripts. As a result, *every other script component* in your open scenes displays a misleading yellow warning:
```
The referenced script on this Behaviour is missing!
```
Panicked developers frequently assume dozens of scripts were deleted or corrupted.

ImportShield explicitly tracks **C6: Blocked by Assembly Compile Error**:
- The inspector displays a blue **Blocked 🔵** status rather than a red **Missing 🔴** error.
- ImportShield informs you: *"This script is present on disk and valid, but cannot be loaded because another script has compiler errors."*
- Prevents accidental deletion or stripping of healthy components during compilation hitches.

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
