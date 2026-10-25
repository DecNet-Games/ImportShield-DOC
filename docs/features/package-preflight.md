---
layout: default
title: Package Preflight
parent: Features
nav_order: 2
description: "Package Preflight inspection: sandboxed .unitypackage scanning, safety verdicts, overwrite maps, and GUID collision detection."
---

# Package Preflight

**Package Preflight** allows you to inspect and evaluate any `.unitypackage` before a single byte is extracted into your project's `Assets/` directory.

---

## Why Preflight Before Importing?

When you double-click a `.unitypackage` or drag it into Unity, the standard engine dialog displays a simple list of filenames with checkboxes. It does **not** inform you:
- Whether an imported script shares a class name with one of your existing scripts (which triggers catastrophic compilation errors).
- Whether an imported asset's `.meta` file uses a GUID that collides with an existing asset in your project.
- Whether an imported asset will silently overwrite customized prefabs or textures in your project.
- Whether the package targets a different render pipeline or requires missing assembly definitions.

Package Preflight inspects the internal TAR archive and `.meta` files inside the package in an isolated sandbox, giving you complete clarity before committing changes to disk.

---

## Using Package Preflight

1. In the Unity Editor, open:
   ```
   Tools ▸ ImportShield ▸ Preflight Package...
   ```
2. Drag and drop any `.unitypackage` file into the designated drop zone (or click **Browse** to select an archive from disk).
3. ImportShield unpacks the archive table of contents into memory and runs a deep comparative analysis against your current project index.
4. An immediate safety verdict is rendered alongside an interactive inspection table.

---

## Safety Verdict Levels

Every scanned package receives a definitive, color-coded safety verdict:

### 🟢 GREEN: Safe to Import
- Zero file overwrites detected.
- Zero GUID collisions with existing assets.
- Zero duplicate class or namespace declarations.
- No pipeline incompatibilities identified.
- **Action**: Safe to proceed with standard import or **Import Safely**.

### 🟡 AMBER: Warnings Detected
- Overwriting non-critical assets (e.g. sample scenes or third-party documentation).
- Minor shader pipeline mismatches (e.g. Standard shader materials that can be healed by Material Doctor).
- Assembly definition references that may require additional packages.
- **Action**: Review the highlighted warnings; proceed with **Import Safely** to ensure automatic rollback backups are established.

### 🔴 RED: Severe Conflicts Detected
- **Direct Asset Overwrites**: The package contains files that will overwrite customized project assets at identical paths.
- **GUID Collisions**: The package contains `.meta` files whose GUIDs match existing, unrelated assets in your project, threatening massive reference cross-linking.
- **Duplicate Class Names**: The package declares C# classes (e.g. `PlayerController`, `GameManager`, `CameraFollow`) that already exist in your assemblies, which will immediately break project compilation.
- **Action**: Deselect conflicting files or use ImportShield's targeted import recommendations.

---

## Conflict Detection Categories

<div class="table-wrapper" markdown="block">

| Conflict Type | Risk Level | What It Means | ImportShield Safeguard |
|:---|:---|:---|:---|
| **Will-Overwrite** | 🔴 Critical | The package contains a file whose path matches an existing asset in `Assets/`. | Shows exact side-by-side path comparison; backs up existing asset before allowing import. |
| **GUID Collision** | 🔴 Critical | The package asset uses a GUID already assigned to a different asset in your project. | Highlights both assets; warns that Unity's native importer would cross-corrupt references. |
| **Duplicate Class** | 🔴 Critical | A script inside the archive defines a C# class or struct name already present in your project. | Identifies the colliding `.cs` files and class identifiers before compilation failure occurs. |
| **Asmdef Mismatch** | 🟡 Warning | Package requires an assembly definition (`.asmdef`) not referenced by your project settings. | Lists missing assembly dependencies and target platforms. |
| **Pipeline Mismatch**| 🟡 Warning | Package materials use Built-in shaders in a project configured for Universal Render Pipeline (URP). | Flags materials for post-import repair using [**Material Doctor**]({{ site.baseurl }}/docs/features/material-doctor/). |

</div>

---

## The "Import Safely" Workflow

Instead of clicking Unity's standard import button, click **Import Safely** directly within the Package Preflight window:

1. **Atomic Pre-Import Freeze**: ImportShield takes an instantaneous binary snapshot of the project reference graph.
2. **Pre-Import Byte Backup**: Any existing assets flagged under "Will-Overwrite" have their current files and `.meta` files copied to `<ProjectRoot>/ProjectDoctorData/Transactions/<id>/orig/`.
3. **Execution**: The package is extracted into the project under controlled observation.
4. **Post-Import Verification**: ImportShield immediately calculates the diff, surfaces the [**Blast Radius Map**]({{ site.baseurl }}/docs/features/blast-radius-and-repair/), and prepares evidence-ranked repairs.
5. **Undo Import Ready**: If the import causes unexpected breakage, the **Undo Import** button is immediately available to restore overwritten or modified files byte-for-byte.

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
