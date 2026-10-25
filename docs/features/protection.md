---
layout: default
title: Protection Watchers
parent: Features
nav_order: 5
description: "Background protection watchers: ImportGuardHooks, CrashRecoveryWatcher, DurableVerificationQueue, and ReloadProbe."
---

# Protection Watchers & Crash Resilience

ImportShield doesn't just wait for you to click buttons—it integrates directly with the Unity Editor's event bus and domain lifecycle to safeguard your project in real-time.

---

## 1. ImportGuardHooks

`ImportGuardHooks` connects into Unity's `AssetDatabase` callback pipeline to intercept destructive operations before they reach your files.

- **Package Import Interception (`importPackageStarted`)**: When an import begins, `ImportGuardHooks` pauses execution for milliseconds, triggers a [**Pre-Import Freeze**]({{ site.baseurl }}/docs/features/pre-import-snapshots/), and establishes byte backups of any assets slated for overwriting.
- **Import Settle Detection (`importPackageCompleted` / `importPackageCancelled`)**: Detects when Unity finishes extracting the package and recompiling scripts. It immediately triggers an incremental scan, calculates the delta, and pops open the Change Report if new breakages are discovered.
- **UPM Dependency Resolution**: Monitors `Packages/manifest.json` for changes made via the Unity Package Manager, ensuring package updates are snapshotted with equal fidelity.

---

## 2. CrashRecoveryWatcher

Hardware crashes, blue screens, sudden power loss, or forced Unity kills (`Task Manager`) can strike at any moment. If an interruption occurs halfway through patching a 100MB scene file, standard tools leave your scene corrupt and unreadable.

`CrashRecoveryWatcher` operates on an atomic, state-driven journal:

```
[ Transaction Initiated ]
          │
          ▼
Write 'journal.json' with Status = "Prepared"
Verify target byte backups exist in ProjectDoctorData/
          │
          ▼
Apply disk-level YAML patch
          │
          ▼
Verify patched file on disk (Post-Condition)
          │
          ▼
Update 'journal.json' with Status = "Committed"
```

### The Crash Recovery Sequence
1. Whenever the Unity Editor boots up or reloads its AppDomain, `CrashRecoveryWatcher` runs as a high-priority `[InitializeOnLoad]` routine.
2. It inspects `<ProjectRoot>/ProjectDoctorData/Transactions/` for any journal files whose status is **not** marked `Committed` (e.g. status is `Prepared`).
3. If an uncommitted journal is found, ImportShield instantly surfaces a modal recovery prompt:
   ```
   ⚠️ Interrupted Transaction Detected
   A previous repair transaction was interrupted during editor shutdown.
   Would you like to complete verification or roll back to the verified backup?
   [ Rollback to Backup ]  [ Re-verify & Commit ]
   ```
4. Clicking **Rollback to Backup** restores the verified original bytes immediately, guaranteeing that your scene or prefab is never left in a corrupted state.

---

## 3. DurableVerificationQueue

When repairing dozens of scenes and prefabs across a massive project, file I/O operations must often span across Unity script compilation boundaries and domain reloads.

`DurableVerificationQueue` provides a persistent work queue:
- Persisted to disk in `ProjectDoctorData/verification_queue.json`.
- Survives script recompilation cycles and assembly unloads.
- Guarantees that every single touched file undergoes post-patch validation before the overall transaction is certified as clean.

---

## 4. ReloadProbe

Unity's script compilation cycle reloads the C# AppDomain, wiping in-memory state. `ReloadProbe` ensures seamless state continuity across domain reloads:

- Detects when an assembly compilation was triggered by an import or repair.
- Probes script compilation results: if an import introduces fatal compile errors, `ReloadProbe` halts subsequent dependent actions and surfaces the exact compile error diagnostic to the user.
- Distinguishes between scripts that are temporarily unloading versus scripts that were genuinely deleted or renamed.

---

## Zero Performance Overhead

All protection watchers are engineered for zero runtime interference:
- Zero background polling loops.
- Event-driven execution hooked exclusively into native Editor events.
- Negligible idle memory footprint (< 4MB).

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
