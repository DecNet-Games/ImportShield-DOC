---
layout: default
title: Public API
nav_order: 7
has_children: true
permalink: /docs/api/
description: "Public API reference and programmatic C# interface for ImportShield."
---

# Public API & Architecture

**ImportShield** provides a comprehensive, type-safe C# API that can be consumed directly by custom Unity editor extensions, build automation pipelines, or third-party tools.

---

## The `noEngineReferences` Architecture

The underlying core logic is housed in the `ProjectDoctor.Core` assembly (namespace `ProjectDoctor.Core`), which is explicitly compiled with:
```json
"noEngineReferences": true
```

### Why Decoupling Matters:
1. **Zero Runtime Engine Coupling**: The core engine does not import `UnityEngine` or `UnityEditor`. It operates purely on standard .NET abstractions (`System.IO`, `System.Text`, `ReadOnlySpan<char>`).
2. **Lightning-Fast Execution**: File parsing, diffing, graph operations, and patch calculations execute without triggering Unity engine serialization overhead.
3. **Headless & Cross-Platform Testing**: The core can be executed in standalone console runners or .NET test frameworks completely outside the Unity Editor.

---

## High-Level API Namespaces

<div class="table-wrapper" markdown="block">

| Namespace | Responsibility | Primary Entry Points |
|:---|:---|:---|
| `ProjectDoctor.Core.Parsing` | Disk YAML scanning, token extraction, stat-cache | `ProjectScanner`, `YamlLineScanner` |
| `ProjectDoctor.Core.Snapshots` | Binary snapshot serialization, baseline storage | `SnapshotStore`, `BaselineStore`, `SnapshotBinarySerializer` |
| `ProjectDoctor.Core.Diff` | Delta computation, provenance separation, cause attribution | `Differ`, `ChangeSet`, `BreakRecord` |
| `ProjectDoctor.Core.Recovery` | Multi-strategy candidate search, evidence scoring | `RecoveryEngine`, `FixCandidate`, `ConfidenceBand` |
| `ProjectDoctor.Core.Repair` | Line-level YAML patching, diff generation | `FilePatchEngine`, `PatchPlan`, `LineDiff` |
| `ProjectDoctor.Core.Safety` | Fail-closed transactions, byte backups, quarantine | `TransactionEngine`, `QuarantineManager`, `Journal` |
| `ProjectDoctor.Core.Shaders` | Pink shader diagnosis & URP property translation | `MaterialDoctor`, `PipelineMapping` |
| `ProjectDoctor.Cli` | Headless batchmode runner for CI/CD gates | `CommandLineRunner` |

</div>

---

## Detailed API Guides

Explore the focused API documentation pages:

- [**Core Engine API**]({{ site.baseurl }}/docs/api/core-engine/): Programmatic indexing, diffing, and recovery.
- [**Safety & Transactions API**]({{ site.baseurl }}/docs/api/safety-and-transactions/): Fail-closed transactions, byte-level patching, and quarantine.
- [**CLI & Automation API**]({{ site.baseurl }}/docs/api/cli-automation/): Headless execution interface for continuous integration.

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
