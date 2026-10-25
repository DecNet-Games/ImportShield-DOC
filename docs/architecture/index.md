---
layout: default
title: System Architecture
nav_order: 6
has_children: true
permalink: /docs/architecture/
description: "System Architecture of ImportShield: the Observation, Diagnosis, and Repair pipeline."
---

# System Architecture Overview

ImportShield is designed around strict engineering principles to deliver rock-solid stability, deterministic recovery, and sub-second execution across massive Unity projects.

Unlike conventional editor tools that manipulate high-level Unity objects in memory, ImportShield treats the **filesystem as the source of truth**, executing all operations through a disciplined three-stage pipeline: **Observation**, **Diagnosis**, and **Repair**.

---

## The Three-Stage Architecture Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│ 1. OBSERVATION STAGE                                        │
│ - Filesystem Stat-Cache (path, size, mtimeUtc)              │
│ - High-performance streaming YamlLineScanner                │
│ - Compact GuidIndex & GZip binary snapshot store            │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. DIAGNOSIS STAGE                                          │
│ - Strict Provenance Separation (New vs Pre-existing)        │
│ - Cause Attribution Taxonomy (C1–C8)                        │
│ - Evidence-Ranked Recovery Engine (R1–R7)                   │
│ - Confidence Band Classification (Certain, High, Med, Low)  │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. REPAIR & VERIFICATION STAGE                              │
│ - Two-Pane Line Diff Generator                              │
│ - Transaction Journaling ('Prepared' -> 'Committed')        │
│ - Atomic Byte-for-Byte File Backups                         │
│ - Line-level YAML Patching (CRLF/LF preservation)           │
│ - Post-Write Semantic Verification (Fail-Closed)            │
└─────────────────────────────────────────────────────────────┘
```

---

## Core Engineering Principles

### 1. Disk Is the Source of Truth
Unity's in-memory `SerializedObject` APIs can obscure formatting details, reorder fields, or inadvertently drop custom serialization flags. ImportShield reads and writes serialized YAML directly on disk. All line patches strictly preserve line endings (CRLF vs LF), custom whitespace, comments, and file headers.

### 2. Fail-Closed Verification
Every mutation executed by ImportShield must satisfy three post-conditions:
1. Target references resolve correctly on disk.
2. Zero new breaks or missing scripts are introduced.
3. Untouched lines remain byte-identical to their pre-repair state.
If any condition fails, the entire transaction automatically rolls back using the saved byte backups.

### 3. Reversible Transactions
Before modifying any asset, original files and their `.meta` files are copied to:
```
<ProjectRoot>/ProjectDoctorData/Transactions/<TransactionId>/orig/
```
Even if Unity crashes, power is lost, or the OS terminates the process, the transaction journal enables 1-click rollback upon restarting the editor.

### 4. Decoupled Core (`"noEngineReferences": true`)
The core algorithms of ImportShield (`ProjectDoctor.Core`) are completely decoupled from `UnityEngine.dll` and `UnityEditor.dll`. This guarantees that core parsing, diffing, and recovery logic can be thoroughly verified through pure .NET unit tests without the overhead or unpredictability of the Unity runtime.

---

## Architecture Subsections

Explore the underlying implementation details in the dedicated architecture guides:

- [**Observation Pipeline**]({{ site.baseurl }}/docs/architecture/observation-pipeline/): Discover how `ProjectScanner`, `YamlLineScanner`, and the stat-cache achieve sub-500ms re-indexing.
- [**Provenance Separation & Cause Attribution**]({{ site.baseurl }}/docs/architecture/provenance-separation/): Learn how the engine separates new breakages from pre-existing issues and categorizes causes (C1–C8).
- [**Transaction Engine & Fail-Closed Safety**]({{ site.baseurl }}/docs/architecture/transaction-engine/): Review the state machine, journal format, byte backup strategy, and post-write verification pipeline.
