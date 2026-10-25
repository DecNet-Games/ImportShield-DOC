---
layout: default
title: Frequently Asked Questions
nav_order: 9
description: "Frequently Asked Questions (FAQ) regarding ImportShield for Unity."
permalink: /docs/faq/
---

# Frequently Asked Questions (FAQ)

Find clear answers to common questions about ImportShield's architecture, safety model, and daily usage.

---

### Q1: What makes ImportShield different from traditional asset cleaners?
**A:** Traditional cleaners merely scan for unreferenced files and permanently delete them from disk, risking silent deletion of assets referenced dynamically (`Resources.Load`, Addressables, or reflection). 

ImportShield is primarily an **import safety net and reference repair suite**: it creates semantic snapshots before every import, attributes why references broke, previews repairs with line-level diffs, and provides 1-click transactional rollback. When cleaning, it **quarantines** rather than deletes, allowing complete restoration with preserved GUIDs at any time.

---

### Q2: Does ImportShield require Git or version control to work?
**A:** No. While ImportShield can optionally inspect Git history to recover old GUID names and paths, its core snapshot and rollback engine is completely independent of source control. It creates byte backups in `<ProjectRoot>/ProjectDoctorData/` before every mutation, allowing game jam developers and solo developers without VCS to undo imports and repairs safely.

---

### Q3: What happens if Unity crashes or power is lost during a repair?
**A:** ImportShield uses an atomic journal file (`journal.json`). Before mutating any asset, the journal status is marked `Prepared` and original bytes are verified on disk. If interrupted by a power failure or crash, the `CrashRecoveryWatcher` detects incomplete transactions upon restarting Unity and offers 1-click rollback or recovery.

---

### Q4: Can ImportShield recover deleted C# script source code?
**A:** No. ImportShield repairs serialized references between scenes, prefabs, ScriptableObjects, and scripts. It does not synthesize or recover deleted C# code bodies. If a script class was renamed or moved to another namespace, ImportShield identifies the rename (Cause C3 / Strategy R4) and updates referrers to point to the new class name.

---

### Q5: Does ImportShield send telemetry or make network calls?
**A:** Never. ImportShield is 100% offline and private. There are zero outbound HTTP/HTTPS requests, zero telemetry pings, and zero analytics trackers.

---

### Q6: What Unity versions and render pipelines are supported?
**A:** ImportShield is tested on Unity **2022.3 LTS** and **Unity 6**. It supports the Built-in Render Pipeline, Universal Render Pipeline (URP), and High Definition Render Pipeline (HDRP). The UI is built with UI Toolkit and fully adapts to both Light and Dark editor skins.

---

### Q7: Where are snapshots and backups stored?
**A:** Ephemeral snapshots are stored in `Library/ProjectDoctor/Snapshots/`. Irreplaceable transaction backups and quarantine archives are stored in `<ProjectRoot>/ProjectDoctorData/` outside `Library/`, ensuring they survive accidental `Library/` folder deletions.

---

### Q8: Will ImportShield add code or bloat to my built game?
**A:** No. ImportShield is an **editor-only tool**. Its runtime dependencies are strictly zero, and all code is compiled within editor and CLI assembly definitions with `"noEngineReferences": true`. Zero bytes of ImportShield code are ever bundled into standalone player builds.

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
