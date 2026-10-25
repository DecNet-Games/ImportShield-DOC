---
layout: default
title: Blast Radius & Repair
parent: Features
nav_order: 3
description: "Change Investigator, Blast Radius Map, Evidence-Ranked Recovery Engine (R1–R7), and two-pane diff previews."
---

# Blast Radius & Reference Repair

When an asset import, package update, or script refactoring breaks serialized references in Unity, you need to know three things immediately: **what broke**, **why it broke**, and **how to fix it with verified proof**.

The **Change Investigator** and **Evidence-Ranked Recovery Engine** provide this exact transparency.

---

## The Blast Radius Map

The **Blast Radius Map** is a vector-rendered visual node graph drawn inside the ImportShield **Changes** tab using Unity's UI Toolkit `Painter2D` engine.

```
[ FantasyProps_v2.unitypackage ]
         │ (overwrites Weapon.cs with new GUID)
         ▼
    [ Weapon.cs ]
     ├──► [ Sword.prefab ] ──► [ Arena.unity ] (Scene)
     └──► [ Shield.prefab ] ──► [ Character_Hero.prefab ]
```

### Visual Anatomy
- **Epicenter (Orange/Red)**: The imported package or modified root asset that triggered the change.
- **Direct Casualties (Red)**: Scripts, models, or textures whose GUID or fileID was altered, deleted, or overwritten.
- **Propagated Breaks (Amber)**: Prefabs, ScriptableObjects, and Scenes whose serialized fields point to a direct casualty.
- **Healthy Assets (Green/Muted)**: Unaffected project assets.
- **Interactive Cascade-to-Green**: When you apply a repair plan, the node graph animates in real-time, watching affected nodes transition from red and amber back to crisp green as references resolve on disk.

---

## Why References Break: Cause Attribution (C1–C8)

ImportShield never shows generic "Reference Missing" errors. Every broken reference is analyzed against an 8-point diagnostic taxonomy:

<div class="table-wrapper" markdown="block">

| Code | Cause Category | Diagnostic Explanation |
|:---:|:---|:---|
| **C1** | Target File Deleted | The referenced file was removed from the filesystem by the import or user. |
| **C2** | Target File Replaced with New GUID | An asset exists at the expected path, but its `.meta` file was overwritten with a new GUID. |
| **C3** | Script Class Renamed or Removed | The `.cs` source file exists, but the class name inside was renamed or moved to another namespace. |
| **C4** | Assembly Dependency Changed | The referenced assembly definition (`.asmdef`) was re-scoped, missing, or renamed. |
| **C5** | Prefab Variant Base Modified | A nested prefab variant's parent GUID was altered or stripped. |
| **C6** | Script Blocked by Compile Error | The script exists and is valid, but compiler errors in *other* scripts prevent Unity from loading its type. Marked as **Blocked**, never **Missing**. |
| **C7** | Unity Version Upgrade Serialization | The serialization structure changed between major Unity editor versions. |
| **C8** | Unknown / Complex Cause | Complex edge cases (< 5% of breaks); flagged for detailed manual review. |

</div>

---

## Evidence-Ranked Recovery Engine (R1–R7)

When repairing broken references, ImportShield rejects crude guessing. It evaluates 7 hierarchical recovery strategies, assigning an honest confidence score and plain-English explanation to each candidate:

### Strategy Taxonomy

1. **R1: Moved / Reimported Asset (Certain, 100%)**
   An asset with an identical content hash and identical filename exists at another path in the project.
2. **R2: Content Hash & Type Match (High, 96%)**
   Content hash and serialized asset type match an existing file, even if the filename differs slightly.
3. **R3: Path Match with New GUID (High, 92%)**
   An asset of identical type sits at the exact original path, but has a freshly generated GUID (classic package re-import scenario).
4. **R4: Script Class & Attribute Match (High, 90–94%)**
   Identifies renamed C# classes using namespace inspection, `[MovedFrom("OldNamespace")]`, or `[FormerlySerializedAs("OldClass")]` attributes.
5. **R5: Git History Pickaxe (High, 88%)**
   Queries local Git commit logs (if available) to identify the original file path and name associated with a deleted GUID.
6. **R6: Normalized Name Similarity (Medium, ≤60%)**
   Uses normalized Levenshtein distance to find similarly named assets within the same directory hierarchy.
7. **R7: Persistent Rule Book Match (Certain, 100%)**
   Matches an explicit mapping previously confirmed by the developer and stored in `ProjectDoctorData/rules.json`.

---

## Confidence Bands

Candidates are categorized into four strict confidence tiers:

- **Certain (100%)**: Mathematically verifiable matches (R1, R7). Safe for one-click bulk repair.
- **High (90–99%)**: Strongly verified through metadata, attributes, or content hashes (R2, R3, R4, R5). Safe for one-click bulk repair.
- **Medium (60–89%)**: Heuristic suggestions (R6). Displayed with yellow warning badges; requires developer review before applying.
- **Low (<60%)**: Weak correlation. Displayed for informational purposes only; **never** included in bulk repair actions.

---

## Two-Pane Line Diff Preview

Before a single byte on disk is altered, ImportShield presents an interactive **Diff Viewer**:

- **Left Pane (Current Disk State)**: Displays the exact line in your `.unity`, `.prefab`, or `.asset` file with the broken GUID / fileID highlighted in red.
- **Right Pane (Proposed Repair)**: Displays the surgical line replacement with the candidate GUID / fileID highlighted in green.
- **Line Ending & Formatting Integrity**: Preserves original line endings (CRLF vs LF), spacing, YAML indentation, and surrounding comments.

When you click **Apply Plan**, ImportShield executes the change inside a fail-closed transaction. If any discrepancy is detected during post-write verification, the transaction instantly rolls back to the verified backup.

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
