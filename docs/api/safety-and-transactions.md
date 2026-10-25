---
layout: default
title: Safety & Transactions API
parent: Public API
nav_order: 2
description: "Safety & Transactions API reference: TransactionEngine, FilePatchEngine, and QuarantineManager."
---

# Safety & Transactions API

The Safety and Transactions API handles disk-level line patching, verified byte backups, fail-closed rollbacks, and reversible quarantine.

---

## 1. `FilePatchEngine`

The `FilePatchEngine` generates and applies surgical line patches to serialized YAML assets without re-serializing whole files.

```csharp
using ProjectDoctor.Core.Repair;
using ProjectDoctor.Core.Recovery;
using ProjectDoctor.Core.Diff;

// Construct repair pairs (BreakRecord + chosen FixCandidate)
var pairs = new List<(BreakRecord breakRec, FixCandidate candidate)>
{
    (brokenReference, chosenCandidate)
};

// Generate a PatchPlan (calculates exact byte offsets and diffs)
PatchPlan plan = FilePatchEngine.CreatePlan(projectRoot, pairs);

foreach (FilePatch patch in plan.FilePatches)
{
    Console.WriteLine($"Patch for {patch.RelativePath}: {patch.LineReplacements.Count} lines modified.");
}

// Apply the plan within a fail-closed transaction
bool success = FilePatchEngine.ApplyPlan(projectRoot, plan);
if (!success)
{
    Console.WriteLine("Patch failed verification and was automatically rolled back!");
}
```

---

## 2. `TransactionEngine`

The `TransactionEngine` manages the creation, journaling, verification, and rollback of atomic operations.

```csharp
using ProjectDoctor.Core.Safety;

var transactionEngine = new TransactionEngine(projectRoot);

// Begin transaction
TransactionContext context = transactionEngine.BeginTransaction("RepairWeaponReferences");

try
{
    // 1. Stage backup files
    context.BackupFiles(new[] { "Assets/Scenes/Arena.unity", "Assets/Prefabs/Sword.prefab" });

    // 2. Perform file writes
    // ...

    // 3. Verify post-conditions
    bool verified = context.VerifyTargetReferences();
    if (!verified)
    {
        throw new InvalidOperationException("Post-write verification failed.");
    }

    // 4. Commit transaction
    context.Commit();
}
catch (Exception ex)
{
    Console.WriteLine($"Rolling back due to error: {ex.Message}");
    context.Rollback();
}
```

### Rollback via Transaction ID
You can roll back any previously committed transaction from code:
```csharp
bool rolledBack = transactionEngine.RollbackTransaction("20261025_143022_a1b2c3d4");
```

---

## 3. `QuarantineManager`

The `QuarantineManager` provides reversible file isolation outside `Assets/`.

```csharp
using ProjectDoctor.Core.Safety;

var quarantine = new QuarantineManager(projectRoot);

// Quarantine unreferenced files
var filesToQuarantine = new[] { "Assets/LegacyProps/Chair.prefab" };
string batchId = quarantine.QuarantineFiles(filesToQuarantine, "CleanupUnreferencedProps");

Console.WriteLine($"Quarantined to batch: {batchId}");

// Restore batch later
bool restored = quarantine.RestoreBatch(batchId);
Console.WriteLine($"Batch restoration status: {restored}");
```

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
