---
layout: default
title: Time Machine
parent: Features
nav_order: 10
description: "Time Machine: snapshot scrubber, historical graph rewind, and differential state comparison."
---

# Time Machine (Snapshot Scrubber)

Version control systems (such as Git) track text line changes, but they cannot tell you how your Unity serialized reference graph *evolved* across dozens of commits, package imports, and asset migrations.

The **Time Machine** module provides an interactive timeline scrubber allowing you to inspect, compare, and rewind reference graphs across historical project snapshots.

---

## The Historical Snapshot Archive

Every significant project milestone generates a compressed binary snapshot in `Library/ProjectDoctor/Snapshots/`:
- Baseline snapshots
- Pre-import freeze snapshots
- Post-repair transaction checkpoints
- Manual milestone captures

These snapshots record the full topology of GUIDs, fileIDs, script types, and cross-asset links at that specific instant.

---

## Operating the Time Machine

1. Open the **History** tab in the ImportShield Window:
   ```
   Tools ▸ ImportShield ▸ ImportShield Window ▸ History
   ```
2. The **Time Machine Scrubber** displays a chronological timeline of all recorded snapshots:
   - Date and UTC timestamp
   - Event trigger (`Pre-Import: FantasyProps`, `Baseline Scan`, `Repair Plan #4`)
   - Asset count and reference health status
3. Drag the scrubber or select any two snapshots on the timeline.
4. Click **Compare Snapshots (Diff)**.

---

## Differential State Comparison

Comparing two snapshots instantly renders a comparative delta matrix:

<div class="table-wrapper" markdown="block">

| Comparison Category | Description | Insight Provided |
|:---|:---|:---|
| **Added Assets** | Files and GUIDs present in Snapshot B but not Snapshot A. | Tracks package additions and new asset creations. |
| **Removed Assets** | Files and GUIDs present in Snapshot A but absent in Snapshot B. | Discovers assets deleted or moved outside the project. |
| **Modified References** | Assets whose internal fileID/GUID pointers changed. | Pinpoints exactly which commit or import altered a prefab's target. |
| **Resolved Breaks** | Broken references in Snapshot A that were healed in Snapshot B. | Confirms successful repair operations. |
| **Introduced Breaks** | Healthy references in Snapshot A that became broken in Snapshot B. | Identifies regressions introduced by asset reorganizations. |

</div>

---

## Historical Graph Rewind & Recovery

If an asset reorganization performed days ago resulted in elusive reference corruption that went unnoticed:
1. Locate the last known-good snapshot on the Time Machine scrubber.
2. Select the corrupted prefabs or scenes.
3. Click **Rewind Selected References to Snapshot**.
4. ImportShield reads the historical GUID mappings from the archive, generates a two-pane diff against the current files on disk, and applies a transactional repair restoring the historical links.

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
