---
layout: default
title: Transaction Engine
parent: System Architecture
nav_order: 3
description: "Fail-Closed Transactions: journal state machine, byte-for-byte backups, and post-write semantic verification."
---

# Transaction Engine & Fail-Closed Safety

Modifying serialized Unity YAML files is an operation fraught with risk. A single misplaced colon, truncated byte, or invalid indentation can corrupt a 200MB scene file, rendering it unopenable by Unity.

The **Transaction Engine** guarantees that no repair, import rollback, or component stripping operation can ever leave your project in a corrupt state.

---

## The Fail-Closed Model

Traditional file utilities operate on a **fail-open** model: they perform writes sequentially; if an error occurs midway through, whatever was already written remains on disk, leaving a partially corrupted project.

ImportShield operates strictly on a **fail-closed** model:
- No file is modified until byte backups of all targets are verified on disk.
- If any operation fails, times out, or triggers a verification discrepancy, the engine immediately and automatically rolls back all touched files to their verified originals.
- If an operation cannot be 100% verified, it is treated as a total failure.

---

## Transaction Lifecycle State Machine

Every modification executed by ImportShield passes through a formal finite state machine recorded in `<ProjectRoot>/ProjectDoctorData/Transactions/<id>/journal.json`:

```
               ┌───────────────────────┐
               │      Initiated        │
               └───────────┬───────────┘
                           │ 1. Create directory & copy byte originals
                           ▼
               ┌───────────────────────┐
               │       Prepared        │
               └───────────┬───────────┘
                           │ 2. Apply surgical line patches to disk
                           ▼
               ┌───────────────────────┐
               │        Written        │
               └───────────┬───────────┘
                           │ 3. Execute Semantic Verification
             ┌─────────────┴─────────────┐
 (Pass)      ▼                           ▼ (Fail)
┌─────────────────────────┐ ┌─────────────────────────┐
│        Committed        │ │       RolledBack        │
└─────────────────────────┘ └─────────────────────────┘
```

---

## 1. Atomic Byte Backups

Prior to touching a single file on disk:
1. ImportShield allocates a unique transaction ID (`yyyyMMdd_HHmmss_<guid>`).
2. It creates an isolated directory:
   ```
   <ProjectRoot>/ProjectDoctorData/Transactions/<id>/orig/
   ```
3. Target assets and their associated `.meta` files are copied into `orig/`.
4. The copied files are immediately read back and their SHA-256 hashes are verified against the active project files.
5. Only when byte parity is confirmed does the journal transition to the `Prepared` state.

---

## 2. Line-Level YAML Patching (`FilePatchEngine`)

When applying repairs, ImportShield does not re-serialize the entire file through a generic YAML generator:
- It targets only the specific lines where broken GUIDs, fileIDs, or properties reside.
- It preserves original file line endings (CRLF on Windows, LF on Unix/macOS).
- It preserves all comments, custom formatting, and non-target lines exactly as they existed.

---

## 3. Post-Write Semantic Verification

Immediately after writing patches to disk, ImportShield runs a thorough verification pass:
1. **Target Reference Resolution**: Parses the touched file using `YamlLineScanner` to ensure the replaced GUID/fileID matches the intended candidate.
2. **Untouched Line Invariance**: Compares all unpatched lines against the original backup to ensure no unintended formatting or data shifts occurred.
3. **No Secondary Breakages**: Confirms that the repair did not trigger new missing references elsewhere in the asset.

If any check fails:
- The engine halts execution immediately.
- The verified files in `orig/` are copied back to the project root.
- The journal status is recorded as `RolledBack`.
- A detailed error diagnostic is presented to the user.

---

## 4. 1-Click Manual Rollback

Every completed transaction remains preserved in your `History` tab. Even weeks later, you can select any transaction and click **Rollback Transaction**:
- Restores original files byte-for-byte.
- Automatically handles any manual edits made in the interim via the [**Conflicts Folder**]({{ site.baseurl }}/docs/features/quarantine-and-conflicts/).

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
