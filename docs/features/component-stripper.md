---
layout: default
title: Component Stripper
parent: Features
nav_order: 8
description: "Missing Component Stripper: safely removing dead MonoBehaviour m_Script slots without corrupting sibling components."
---

# Missing Component Stripper

Renaming a script, deleting a deprecated class, or importing a package with missing dependencies frequently leaves behind yellow warnings in Unity:
```
The referenced script on this Behaviour is missing!
```
In the inspector, these appear as disabled component headers displaying **`Missing (MonoBehaviour)`**.

The **Missing Component Stripper** safely scrubs dead component references from scenes and prefabs without corrupting adjacent components or scene hierarchy data.

---

## The Danger of Naive Script Removal

In Unity's YAML serialization format, a GameObject references its attached components via an array of fileID pointers:

```yaml
--- !u!1 &123456789
GameObject:
  m_ObjectHideFlags: 0
  m_Component:
  - component: {fileID: 123456790}  # Transform
  - component: {fileID: 123456791}  # Dead MonoBehaviour (Missing)
  - component: {fileID: 123456792}  # BoxCollider
--- !u!114 &123456791
MonoBehaviour:
  m_ObjectHideFlags: 0
  m_Script: {fileID: 0}            # Dead script reference
```

If an editor utility carelessly deletes the `MonoBehaviour` YAML block without updating the parent `GameObject`'s `m_Component` array:
- The GameObject retains a dangling fileID pointer to a non-existent component.
- The next time Unity opens the scene or saves the prefab, the serialization graph becomes corrupted.
- Sibling components (such as Colliders, Rigidbodies, or custom scripts) can shift array positions, detaching event callbacks and breaking inspector assignments.

---

## The ImportShield Surgical Stripping Flow

ImportShield performs a two-pass atomic cleanup:

```
[ Scan Scene / Prefab YAML ]
             │
             ▼
┌──────────────────────────────────────────────┐
│ Identify Dead MonoBehaviour Blocks           │
│ - Locate 'MonoBehaviour:' with 'm_Script: 0' │
│ - Record target fileID (e.g. 123456791)      │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│ Prune GameObject Component Pointer Array     │
│ - Locate parent GameObject block             │
│ - Remove '- component: {fileID: 123456791}'  │
│ - Re-index remaining array elements cleanly  │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│ Remove Dangling MonoBehaviour Block          │
│ - Remove serialized component data block     │
│ - Preserve all sibling components untouched  │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│ Fail-Closed Verification                     │
│ - Verify sibling components resolve intact   │
│ - Confirm zero new compiler/engine warnings  │
└──────────────────────────────────────────────┘
```

---

## Operating the Component Stripper

1. Open the **Stripper** tab in the ImportShield Window:
   ```
   Tools ▸ ImportShield ▸ ImportShield Window ▸ Stripper
   ```
2. Click **Scan Missing Scripts**.
3. ImportShield surveys all scenes and prefabs listed in your project assets.
4. Review the detected missing components, grouped by asset path and GameObject hierarchy name.
5. Click **Strip Missing (Preview)**:
   - The Diff Viewer opens, showing the exact YAML lines to be removed.
   - Sibling component pointers are shown intact with zero formatting drift.
6. Click **Apply Stripping Plan**:
   - The original assets are backed up to `ProjectDoctorData/Transactions/`.
   - The surgical stripping is applied to disk.
   - Unity reloads the assets; all missing script warnings disappear cleanly.
7. Need to revert? Click **Rollback Transaction** in the **History** tab to restore original files at any time.

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
