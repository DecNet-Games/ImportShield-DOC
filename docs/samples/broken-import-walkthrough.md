---
layout: default
title: Broken Import Walkthrough
parent: Samples Overview
nav_order: 1
description: "Step-by-step interactive 3-stage walkthrough demonstrating ImportShield on the Broken Import Demo."
---

# Broken Import Demo Walkthrough

This interactive 3-stage walkthrough demonstrates how ImportShield detects, explains, and heals real-world project breakages using the bundled **Broken Import Demo** and `FantasyProps_v2.unitypackage`.

---

## Scenario Overview

You have a functioning game project containing combat prefabs (`Sword.prefab`, `Shield.prefab`) and a combat scene (`Arena.unity`). You purchase a new asset pack from the Asset Store (`FantasyProps_v2.unitypackage`) promising higher-fidelity weapons. 

Unknown to you, the author of `FantasyProps_v2` included a script named `Weapon.cs` with a **newly generated GUID**, overwriting your project's existing script `.meta` file and breaking references across your prefabs and scenes.

Here is how ImportShield handles this situation.

---

## Stage 1: Baseline & Package Preflight

### 1. Establish Baseline Protection
- Open `Assets/Samples/ImportShield/<version>/BrokenImportDemo/Scenes/Arena.unity`.
- Open **Tools ▸ ImportShield ▸ ImportShield Window**.
- Confirm that the status indicator in the top header is **🟢 Protected**.

### 2. Preflight `FantasyProps_v2.unitypackage`
- Navigate to: **Tools ▸ ImportShield ▸ Preflight Package...**.
- Locate `FantasyProps_v2.unitypackage` inside the sample directory:
  ```
  Assets/Samples/ImportShield/<version>/BrokenImportDemo/FantasyProps_v2.unitypackage
  ```
- Drag the `.unitypackage` into the Preflight drop zone.
- Notice the immediate **🔴 RED: Conflicts Detected** safety verdict:
  - **Will Overwrite**: `Weapon.cs` (matches existing project path).
  - **GUID Collision**: `Weapon.cs.meta` uses GUID `b1c2...` while your project uses `a0f9...`.
  - **Duplicate Class**: `Weapon` class declared in both packages.
- Click **Import Safely**. ImportShield automatically captures a Pre-Import Freeze snapshot, byte-backs up the existing `Weapon.cs`, and allows Unity to extract the package.

---

## Stage 2: Change Investigation & Blast Radius Map

As soon as Unity finishes extracting the package:

1. ImportShield immediately detects the settle event and opens the **Changes** tab.
2. Inspect the **Blast Radius Map**:
   - The root epicenter shows `FantasyProps_v2.unitypackage`.
   - Radiating outward, `Weapon.cs` is highlighted in red as a direct casualty.
   - Connected nodes `Sword.prefab`, `Shield.prefab`, and `Arena.unity` are highlighted in amber.
3. Review the **Provenance Separation**:
   - **New Breaks**: 3 (Directly caused by `FantasyProps_v2`).
   - **Pre-existing Breaks**: 0.
4. Click on the broken `Sword.prefab` reference to open its **Evidence Card**:
   - **Why It Broke**: `Cause C2: Target file overwritten at same path with new GUID`.
   - **Fix Candidates**:
     - Candidate 1: `R3: Path Match with New GUID (High, 92%)`.
     - Candidate 2: `R4: Script Class Match 'Weapon' (High, 94%)`.

---

## Stage 3: Two-Pane Diff, Repair & Rollback

### 1. Preview the Line Diff
- Click **⚡ Fix All High+ Confidence (Preview)**.
- The **Diff Viewer** modal opens:
  - Left pane displays the broken line in `Sword.prefab`: `m_Script: {fileID: 11500000, guid: a0f9...}`.
  - Right pane displays the surgical candidate replacement: `m_Script: {fileID: 11500000, guid: b1c2...}`.
  - Sibling components and line endings remain completely intact.

### 2. Apply the Plan & Cascade to Green
- Click **Apply Plan**.
- ImportShield executes the line patch inside a fail-closed transaction.
- In the Scene View and Changes tab, watch the **Cascade-to-Green**:
  - The red and amber nodes on the Blast Radius Map transition dynamically to green.
  - The inspector on `Sword.prefab` and `Arena.unity` immediately reconnects the `Weapon` component without any manual dragging!

### 3. Verify 1-Click "Undo Import"
Want to revert the entire experiment as if the import never happened?
- In the ImportShield Window, click **↩ Undo Import**.
- ImportShield restores the original `Weapon.cs` and its original `.meta` GUID from the transaction backup.
- Unity re-indexes the clean originals. Your project is restored byte-for-byte to its pristine pre-import baseline!

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
