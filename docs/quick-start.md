---
layout: default
title: Quick Start
nav_order: 4
description: "The 60-second quick start guide for ImportShield: installation, preflight, blast-radius investigation, and 1-click rollback."
permalink: /docs/quick-start/
---

# ImportShield — Quick Start Guide (60-Second Setup)

Welcome to **ImportShield** — the commercial-grade impulse-buy safety net and reference repair suite for Unity.

---

## 1. Installation & Zero-Configuration Activation

ImportShield activates automatically upon package installation.

- Open **Tools ▸ ImportShield ▸ ImportShield Window** (or press the **🟢 ImportShield** pill in the top-right of your Scene View).
- ImportShield automatically indexes your project and establishes a pristine **Baseline Snapshot**.
- You are now **Protected** 🟢.

---

## 2. Before You Import: Package Preflight

Never import an untrusted `.unitypackage` blindly again!

1. Go to **Tools ▸ ImportShield ▸ Preflight Package...**.
2. Drag and drop any `.unitypackage` into the drop zone.
3. Inspect the instant **GREEN / AMBER / RED** safety verdict:
   - **Will Overwrite**: Files that replace your existing project assets.
   - **GUID Collisions**: Duplicated metadata that would corrupt references.
   - **Duplicate Classes**: Script names (e.g. `PlayerController.cs`) that would cause compile errors.
4. Click **Import Safely** — ImportShield takes an atomic pre-import snapshot, creates file backups, and runs the import.

---

## 3. After an Import: The Change Report & Blast Radius Map

If an import replaces meta files or breaks references:

1. ImportShield automatically detects the import settle event.
2. Open the **Changes** tab:
   - Inspect the **Blast Radius Map**: Visual node graph radiating outward from the imported package to affected scenes and prefabs.
   - Separate **New Breaks** from **Pre-existing Breaks** (ImportShield never blames an import for old damage).
3. Click any broken reference to inspect its **Evidence Card**:
   - Understand **WHY** it broke (Cause C1–C8).
   - Review ranked **Fix Candidates** with honest confidence scores (Certain, High, Medium, Low).
4. Click **⚡ Fix All High+ Confidence (Preview)**:
   - Review the two-pane diff in the **Diff Viewer**.
   - Click **Apply Plan** — watch the Blast Radius Map cascade to green!
5. Need to undo everything? Click **↩ Undo Import** to restore all overwritten and deleted files.

---

## 4. Healing Pink Shaders (Material Doctor)

Upgrading to URP or imported materials looking neon magenta?

1. Open the **Materials** tab in the ImportShield Window.
2. Inspect all materials with missing or legacy Built-in shaders.
3. Click **Migrate to URP Lit (Preview)** to swap the shader and automatically map legacy properties (`_MainTex` &rarr; `_BaseMap`, `_Color` &rarr; `_BaseColor`, `_Glossiness` &rarr; `_Smoothness`).

---

## 5. Cleaning Dead Scripts (Component Stripper)

1. Open the **Stripper** tab.
2. Review all prefabs and scenes with `Missing (MonoBehaviour)` components.
3. Click **Strip Missing (Preview)** — safely cleans GameObject component arrays with journal backup and 1-click rollback.

---

## 6. Safety & Rollback Guarantee

Every repair is journaled to `<ProjectRoot>/ProjectDoctorData/Transactions/`:

- Go to the **History** tab at any time to inspect past repairs.
- Click **Rollback Transaction** to restore byte-identical originals, even after restarting Unity!

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
