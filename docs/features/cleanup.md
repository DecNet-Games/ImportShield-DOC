---
layout: default
title: Safe Cleanup Engine
parent: Features
nav_order: 6
description: "Safe Cleanup Engine: reachability analysis, duplicate asset merging, build-bloat detection, and dynamic reference scanning."
---

# Safe Cleanup Engine

Over years of development, Unity projects accumulate gigabytes of duplicate textures, abandoned models, test audio files, and orphaned prefabs. The **Safe Cleanup Engine** slashes project size and eliminates bloat while honoring our strict safety boundaries.

---

## The Reachability Analysis Architecture

Traditional cleaners look at individual files in isolation. ImportShield calculates complete transitive reachability roots:

```
[ Root Assets ]
├── Build Scenes (EditorBuildSettings.scenes)
├── Resources Folders (Assets/**/Resources/*)
├── StreamingAssets (Assets/StreamingAssets/*)
└── Addressable Asset Groups (Settings & Entries)
          │
          ▼ (Transitive YAML Reference Traversal)
┌──────────────────────────────────────────────┐
│ Reachable Asset Graph                        │
│ - Textures referenced by active Materials    │
│ - Scripts referenced by prefabs in scenes    │
│ - Audio clips bound to audio sources         │
└──────────────────────────────────────────────┘
          │
          ▼
┌──────────────────────────────────────────────┐
│ Unreferenced Asset Candidates                │
│ - Scanned by Dynamic Reference Scanner       │
└──────────────────────────────────────────────┘
```

---

## Dynamic Reference Scanner (String-Literal Protection)

The single greatest danger of automated asset deletion is breaking code that loads assets dynamically at runtime:

```csharp
// Example runtime dynamic load
var prefab = Resources.Load<GameObject>("Enemies/BossDragon");
var handle = Addressables.LoadAssetAsync<AudioClip>("Music/VictoryTheme");
```

A standard dependency crawler sees zero static YAML references to `BossDragon.prefab` or `VictoryTheme.mp3` and deletes them. When players reach the boss battle in production, the game crashes.

### The ImportShield Safeguard
Before flagging any asset as unused:
1. The **Dynamic Reference Scanner** extracts all string literals from all C# script files in your project.
2. It tests asset filenames, relative paths, and folder names against this string-literal index.
3. If a match is found, the asset is tagged with the **Possibly Dynamic** badge.
4. **Safety Rule**: Assets tagged as *Possibly Dynamic* are automatically excluded from bulk-quarantine actions. They must be manually reviewed and confirmed by the developer before moving to quarantine.

---

## Duplicate Asset Merger

Game development teams often inadvertently duplicate identical assets across different subdirectories (e.g. `UI/Icons/icon_coin.png` and `Store/Textures/icon_coin.png`).

ImportShield’s **Duplicate Merger**:
1. Groups all project files by their SHA-256 content hashes.
2. Detects duplicate asset files regardless of their filenames or folder paths.
3. Allows you to designate the canonical "master" asset.
4. Rewrites all scenes, prefabs, and materials referencing the duplicate copies to point to the canonical asset's GUID.
5. Safely relocates the redundant copies into [**Reversible Quarantine**]({{ site.baseurl }}/docs/features/quarantine-and-conflicts/).

---

## Build Bloat Engine

The **Build Bloat Engine** inspects texture dimensions, compression formats, and uncompressed audio clips:
- Identifies oversized textures (e.g. 4096&times;4096 textures used on minor background props).
- Detects uncompressed audio files consuming excessive storage.
- Flags large meshes with excessive polygon density or unused animation channels.

All cleanup recommendations are presented in a transparent table with estimated project size reductions.

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
