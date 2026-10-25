---
layout: default
title: Quarantine & Conflicts
parent: Features
nav_order: 4
description: "Reversible quarantine, zero-silent-deletion policy, and the Conflicts folder for preserving manual edits."
---

# Quarantine & Conflicts Management

Data safety is the foundational promise of ImportShield. Traditional cleanup and repair utilities often rely on irreversible permanent deletions. ImportShield enforces two non-negotiable safety architectures: **Reversible Quarantine** and the **Conflicts Protection Folder**.

---

## The Zero-Silent-Deletion Policy

Most Unity asset cleaners operate under a dangerous assumption: *"If an asset has no static references in scenes or prefabs, it is safe to delete."*

In modern game engines, this assumption frequently leads to corrupted projects:
- Assets loaded dynamically via `Resources.Load("Path")` or string-based Addressable keys lack static YAML references.
- Assets loaded via reflection or custom serialized data systems appear unreferenced to basic scanners.
- Permanent deletion (`File.Delete`) wipes the asset and its `.meta` file, permanently destroying its GUID.

**ImportShield never permanently deletes an asset without explicit quarantine.**

---

## Reversible Quarantine Architecture

When cleaning unreferenced assets or removing conflicting files during an import rollback:

```
[ Assets/LegacyProps/OldChair.prefab ]
                 │
                 │ (Quarantine action executed)
                 ▼
[ ProjectRoot/ProjectDoctorData/Quarantine/2026-10-25_143022/ ]
  ├── manifest.json
  ├── OldChair.prefab
  └── OldChair.prefab.meta
```

1. **Moved Outside `Assets/`**: Files and their corresponding `.meta` files are relocated to `<ProjectRoot>/ProjectDoctorData/Quarantine/<batch_id>/`.
2. **Metadata Preserved Intact**: The original GUID, file creation timestamps, and relative project paths are preserved inside an accompanying `manifest.json`.
3. **Unity Re-index**: Unity's asset pipeline cleanly unloads the file from memory without metadata corruption.
4. **1-Click Instant Restoration**: If you realize weeks later that a quarantined asset was needed for a seasonal event or dynamic script, open the **Quarantine Manager** and click **Restore**. The asset and its exact GUID return to their original project path immediately.

---

## The Conflicts Protection Folder

A frequent hazard during asset imports and rollbacks is the "interleaved edit" problem:
1. You import an updated package (`v2.0`).
2. You spend twenty minutes manually customizing one of the newly imported scripts or shaders.
3. You discover that other parts of the package broke references, and you click **↩ Undo Import**.

In a naive undo system, your twenty minutes of custom work would be permanently wiped out because the undo engine blindly restores the pre-import state.

### How ImportShield Protects Your Edits
When executing an **Undo Import** or rolling back a transaction:
1. ImportShield hashes the file currently on disk and compares it against:
   - The original pre-import backup.
   - The package-extracted version.
2. If the current hash does not match *either* version, ImportShield detects that **you made manual edits** after the import occurred.
3. Rather than overwriting your work, ImportShield relocates your modified file to:
   ```
   <ProjectRoot>/ProjectDoctorData/Conflicts/<timestamp>/<relativePath>
   ```
4. A prominent notification banner alerts you to the conflict, providing the exact path where your modified work is preserved safely.

---

## Safety Guarantees Summary

- **No Recycle Bin Reliance**: Files are moved to structured project quarantine folders rather than system trash cans that might be emptied automatically.
- **Survives `Library/` Deletion**: Because `ProjectDoctorData/` is stored at the project root outside `Library/`, deleting your `Library/` cache to fix editor issues will never compromise your quarantined assets or conflict backups.
- **Disk-Level Isolation**: Quarantined files are completely invisible to Unity's Asset Database and compiler, eliminating compile errors and memory overhead while preserving recovery safety.

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
