---
layout: default
title: Pre-Import Snapshots
parent: Features
nav_order: 1
description: "Reference Snapshot Engine: baseline snapshots, pre-import freezing, change detection, and incremental indexing."
---

# Pre-Import Snapshots

The **Reference Snapshot Engine** forms the immutable observation layer of ImportShield. It continuously monitors your project's serialized asset graph, recording lightweight, deterministic snapshots before any major operation touches your files.

---

## Why Pre-Import Snapshots Matter

Traditional package imports in Unity immediately unpack archive contents into your `Assets/` directory. If an imported asset overwrites an existing file with a conflicting `.meta` GUID or removes a script required by your scenes, the damage occurs immediately.

ImportShield intercepts this workflow:
1. When Unity signals an impending import or package installation, ImportShield triggers an immediate **Pre-Import Freeze**.
2. It captures a binary snapshot of all asset paths, GUIDs, fileIDs, and cross-asset references.
3. It verifies that existing project state is fully mapped before allowing the import to proceed.
4. Once the import finishes, it compares the before and after states to generate an instant, actionable **Change Report**.

---

## Baseline Snapshots vs Pre-Import Freezes

ImportShield maintains two distinct categories of snapshots:

### 1. Baseline Snapshot
The **Baseline Snapshot** represents your project's known-good, stable reference state.
- Generated automatically upon first launch or when you manually click **Establish Baseline**.
- Stored permanently in `Library/ProjectDoctor/Snapshots/baseline.bin`.
- Serves as the anchor for identifying long-term drift, pre-existing breakages, and CI gate thresholds.

### 2. Pre-Import Freeze Snapshot
A **Pre-Import Freeze Snapshot** is a temporary, high-frequency capture created immediately prior to an import event.
- Stored alongside transaction logs in `Library/ProjectDoctor/Snapshots/pre_import_<timestamp>.bin`.
- Captures the exact millisecond-accurate state before Unity unpacks `.unitypackage` files or UPM installs a package.
- Used by the Change Investigator to strictly isolate **New Breaks** caused by that specific import.

---

## High-Performance Incremental Indexing

Indexing large commercial Unity projects with tens of thousands of assets cannot afford to scan every file on every frame. ImportShield achieves sub-500ms re-indexing using a multi-tiered architecture:

### Stat-Cache Validation
Before opening any file from disk, ImportShield queries the operating system filesystem metadata:
- Path
- File length (bytes)
- `mtimeUtc` (last modified timestamp in UTC)

If all three parameters match the cached entry in memory, the file's reference graph is reused instantly without reading a single byte of disk content.

### Fast Stream Tokenizer (`YamlLineScanner`)
For files that have been modified or newly created:
- Instead of using heavyweight generic YAML DOM parsers that allocate millions of intermediate objects, ImportShield uses an optimized, zero-allocation line reader (`YamlLineScanner`).
- Extracts `guid: [a-f0-9]{32}` and `fileID: -?[0-9]+` tokens directly from the text stream.
- Preserves exact byte offsets and line numbers for surgical patch generation.

### Compact GZip Binary Serialization
Snapshots are persisted to disk using high-speed binary serialization paired with streaming GZip compression:
- A project with 25,000 assets and 100,000 reference edges serializes down to just a few megabytes on disk.
- Snapshots load into memory in under 80 milliseconds.

---

## The Change Report

When an import concludes, ImportShield calculates the delta between the pre-import freeze and the new post-import state. The resulting **Change Report** answers three critical questions:

1. **What assets were added, modified, or overwritten?** (Full file inventory with path and hash diffs).
2. **What reference breakages occurred?** (Separated into brand-new breaks caused by this import versus breaks that were already present).
3. **What is the overall blast radius?** (Directly feedable into the [**Blast Radius Map**]({{ site.baseurl }}/docs/features/blast-radius-and-repair/)).

```
┌────────────────────────────────────────────────────────┐
│ Change Report: FantasyProps_v2.unitypackage            │
├────────────────────────────────────────────────────────┤
│ Added Assets:        14                                │
│ Overwritten Assets:   2 (Weapon.cs, Door.prefab)       │
│ Deleted / Removed:    0                                │
│ Newly Broken Refs:    3 (Scenes/Arena.unity -> Weapon) │
│ Pre-Existing Breaks:  1 (Unrelated legacy test prefab) │
├────────────────────────────────────────────────────────┤
│ Status: ACTION REQUIRED (3 high-confidence fixes)      │
└────────────────────────────────────────────────────────┘
```

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
