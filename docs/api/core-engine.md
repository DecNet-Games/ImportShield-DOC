---
layout: default
title: Core Engine API
parent: Public API
nav_order: 1
description: "Core Engine API reference: ProjectScanner, Differ, RecoveryEngine, and AutoDoctorEngine."
---

# Core Engine API Reference

The Core Engine provides the fundamental observation, diffing, and recovery algorithms that power ImportShield.

---

## 1. `ProjectScanner`

The `ProjectScanner` performs disk-level indexing of `.meta` files and serialized YAML assets (`.unity`, `.prefab`, `.asset`, `.mat`).

```csharp
using ProjectDoctor.Core.Parsing;
using ProjectDoctor.Core.Snapshots;

// Initialize the scanner
var scanner = new ProjectScanner();

// Scan project synchronously with a descriptive reason
string projectRoot = "C:/MyGameProject";
string scanReason = "ManualPreImportCheck";
string engineVersion = "2022.3.22f1";

Snapshot snapshot = scanner.ScanProject(projectRoot, scanReason, engineVersion);

Console.WriteLine($"Indexed {snapshot.Assets.Count} assets and {snapshot.ReferenceEdges.Count} references.");
```

### Key Methods & Properties
- `ScanProject(string projectRoot, string reason, string unityVersion)`: Runs full incremental index utilizing the internal stat-cache.
- `ScanSingleAsset(string projectRoot, string assetPath)`: Incrementally parses an individual file and updates reference edges.
- `GuidIndex`: In-memory bi-directional graph mapping GUIDs, fileIDs, and file paths.

---

## 2. `Differ`

The `Differ` compares two snapshots to produce a structured `ChangeSet` containing added, removed, and broken references with cause attribution.

```csharp
using ProjectDoctor.Core.Diff;

var differ = new Differ();
ChangeSet changes = differ.ComputeDiff(beforeSnapshot, afterSnapshot, "Import:PropsPack_v2");

Console.WriteLine($"New Breaks: {changes.NewBreaks.Count}");
Console.WriteLine($"Pre-existing Breaks: {changes.PreExistingBreaks.Count}");

foreach (BreakRecord b in changes.NewBreaks)
{
    Console.WriteLine($"Break in {b.ReferrerPath} (Line {b.LineNumber}) -> Cause: {b.Cause} ({b.CauseEvidence})");
}
```

### Key Properties of `ChangeSet`:
- `NewBreaks`: Read-only list of references broken by this specific event.
- `PreExistingBreaks`: Read-only list of references that were already broken prior to the event.
- `AddedAssets`: Newly introduced files and their GUIDs.
- `OverwrittenAssets`: Existing project files that were replaced by the import.

---

## 3. `RecoveryEngine`

The `RecoveryEngine` evaluates broken references against the 7-strategy recovery hierarchy (R1–R7) to discover candidate fixes.

```csharp
using ProjectDoctor.Core.Recovery;

var recovery = new RecoveryEngine();

// Find candidates for a specific broken reference
IReadOnlyList<FixCandidate> candidates = recovery.FindCandidates(breakRecord, currentSnapshot);

foreach (FixCandidate candidate in candidates)
{
    Console.WriteLine($"Candidate: {candidate.TargetGuid} | Band: {candidate.Band} | Score: {candidate.ConfidenceScore:P0}");
    Console.WriteLine($"Evidence: {candidate.PlainEnglishEvidence}");
}
```

### Confidence Bands:
- `ConfidenceBand.Certain` (100%): R1 (Moved/reimported), R7 (Rule book match).
- `ConfidenceBand.High` (90–99%): R2 (Content hash match), R3 (Path match new GUID), R4 (Script rename/attribute), R5 (Git history).
- `ConfidenceBand.Medium` (60–89%): R6 (Levenshtein name similarity).
- `ConfidenceBand.Low` (<60%): Weak correlation.

---

## 4. `AutoDoctorEngine`

The `AutoDoctorEngine` orchestrates end-to-end observation, diffing, candidate finding, and repair plan formulation into a single unified entry point.

```csharp
using ProjectDoctor.Core.AutoDoctor;

var engine = new AutoDoctorEngine(projectRoot);
AutoDoctorResult result = engine.AnalyzeProject();

if (result.HasActionableRepairs)
{
    Console.WriteLine($"Ready to repair {result.RecommendedRepairs.Count} high-confidence breaks.");
}
```

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
