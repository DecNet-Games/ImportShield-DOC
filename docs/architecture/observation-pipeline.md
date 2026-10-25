---
layout: default
title: Observation Pipeline
parent: System Architecture
nav_order: 1
description: "The Observation Pipeline: ProjectScanner, YamlLineScanner, GuidIndex, and stat-cache acceleration."
---

# Observation Pipeline

The **Observation Pipeline** is responsible for reading the state of your project from disk and maintaining an accurate, in-memory representation of all assets, GUIDs, and reference relationships.

---

## Performance Challenges in Unity Asset Indexing

In large commercial projects containing tens of thousands of assets, naive scanning approaches fail quickly:
- Using `AssetDatabase.LoadAssetAtPath` forces Unity to instantiate every asset into memory, consuming tens of gigabytes of RAM.
- Using standard YAML DOM parsers creates millions of intermediate nodes, triggering severe garbage collection pauses.
- Re-reading all files on disk after minor changes introduces noticeable editor hitches.

ImportShield overcomes these bottlenecks through three tightly coupled systems: the **Stat-Cache**, the **YamlLineScanner**, and the **GuidIndex**.

---

## 1. The Stat-Cache Architecture

Before reading a file from disk, the `ProjectScanner` queries filesystem metadata via operating system stat calls:

```csharp
public readonly struct FileStatKey : IEquatable<FileStatKey>
{
    public string Path { get; }
    public long FileSizeBytes { get; }
    public long LastModifiedTimeUtcTicks { get; }
}
```

- When indexing begins, `ProjectScanner` iterates through `Assets/` and `Packages/`.
- For each file, it queries `(Path, FileSizeBytes, LastModifiedTimeUtcTicks)`.
- If an entry matching all three fields exists in the in-memory stat-cache, the file's pre-computed reference edges are reused immediately.
- Result: On incremental edits where only one or two files changed, 99.9% of project files bypass disk reads entirely, allowing full-project re-indexing in **under 500 milliseconds**.

---

## 2. Fast Streaming Tokenizer (`YamlLineScanner`)

When a file has been modified or newly created, `YamlLineScanner` processes the file as a raw text stream:

- **Zero Object Allocations**: Rather than constructing an AST or DOM, it processes lines sequentially using read-only string spans (`ReadOnlySpan<char>`).
- **Targeted Token Extraction**: It seeks specific YAML patterns:
  - Asset class IDs: `--- !u!114 &123456789`
  - Referenced GUIDs: `guid: ([a-f0-9]{32})`
  - Referenced fileIDs: `fileID: (-?[0-9]+)`
  - Script bindings: `m_Script: {fileID: 11500000, guid: ...}`
- **Position Tracking**: Records the exact zero-based byte offset and line number of every token. This eliminates the need for expensive string searches during patch generation.

---

## 3. The In-Memory `GuidIndex`

Extracted metadata is fed into a bi-directional lookup graph called `GuidIndex`:

```
┌────────────────────────────────────────────────────────┐
│ GuidIndex                                              │
├────────────────────────────────────────────────────────┤
│ GUID -> Asset Record (Path, Type, Hash, LastModified)  │
│ Path -> GUID                                           │
│ Target GUID -> List<ReferrerEdge> (Inbound references) │
│ Referrer GUID -> List<TargetEdge> (Outbound references)│
└────────────────────────────────────────────────────────┘
```

This bi-directional structure allows instantaneous queries:
- *"Who references this texture?"* (Immediate lookup via inbound edge index).
- *"What does this prefab depend on?"* (Immediate lookup via outbound edge index).
- *"Where did this GUID go?"* (Immediate lookup in the asset record table).

---

## 4. Compressed Binary Snapshot Storage

When a snapshot is saved, `SnapshotBinarySerializer` writes the graph using a binary layout compressed via `System.IO.Compression.GZipStream`:
- Strings are interned into a deduplicated string table.
- GUIDs are stored as raw 16-byte buffers rather than 32-character hex strings.
- 50,000 project references compress down to under 3 megabytes on disk.

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
