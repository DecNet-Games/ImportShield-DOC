---
layout: default
title: Audit Certificate & Health
parent: Features
nav_order: 9
description: "Project Health Reports, SHA-256 digital fingerprints, truthful scope audit certificates, and diagnostic bundles."
---

# Audit Certificate & Project Health Reports

When preparing a commercial Unity game for launch, milestone delivery to a publisher, or transfer between development teams, proving project integrity is paramount.

ImportShield generates cryptographic **Project Health Reports** and **Audit Certificates** verifying that your serialized reference graph is mathematically sound.

---

## The Audit Certificate

An **Audit Certificate** is a cryptographically hashed document confirming that every serialized asset reference in your project resolves to a valid target.

### Cryptographic Attestation
1. ImportShield crawls all serialized assets (`.unity`, `.prefab`, `.asset`, `.mat`, `.meta`).
2. Every asset path, file size, and reference target is ingested into a deterministic Merkle-tree hash structure.
3. The root hash is signed using SHA-256, generating a unique **Digital Fingerprint** representing the exact reference state of the project.
4. Any subsequent change to a file, GUID, or reference target will alter this fingerprint, providing an immutable audit trail for QA teams and publishers.

```
┌────────────────────────────────────────────────────────┐
│ ImportShield AUDIT CERTIFICATE                       │
├────────────────────────────────────────────────────────┤
│ Project Name:        SpaceExplorer_Game                │
│ Engine Version:      Unity 2022.3.22f1                 │
│ Total Assets Scanned: 4,129                            │
│ Total Reference Edges: 38,412                          │
│ Broken References:   0                                 │
│ Missing Scripts:     0                                 │
│ Digital Fingerprint: 9e8a7c2b...4f3a1d6e (SHA-256)     │
│ Audit Verdict:       CERTIFIED HEALTHY 🟢              │
└────────────────────────────────────────────────────────┘
```

---

## Project Health Reports (HTML & JSON)

ImportShield exports comprehensive reports in both human-readable HTML and machine-parsable JSON formats:

- **Executive Summary**: Overview of total assets, healthy references, repaired items, and quarantine status.
- **Dependency Breakdown**: In-depth statistics by asset category (Scenes, Prefabs, Materials, ScriptableObjects, Audio, Textures).
- **Recent Transaction Journal**: Audit trail of every repair plan, import undo, and component stripping operation performed.
- **Exporting a Report**:
  - In the Editor: Navigate to **Reports** tab &rarr; click **Generate Health Report**.
  - Via CLI: Pass `-pdReport html` or `-pdReport json` during automated build steps.

---

## Diagnostic Bundle & Support Exporter

If you ever encounter an anomalous project state or require technical assistance from our engineering team, ImportShield provides a built-in **Diagnostic Bundle Exporter**:

- Open **Tools ▸ ImportShield ▸ Export Diagnostic Bundle...**.
- Generates a sanitized `.zip` archive containing:
  - System environment metadata (Unity version, OS, active render pipeline).
  - Anonymized snapshot manifests (paths and GUID relationships).
  - Recent transaction journals (`journal.json`).
- **Privacy Guarantee**: Zero source code, zero texture images, zero proprietary 3D models, and zero confidential business data are ever included in the bundle. You can inspect every file in the `.zip` before sharing.

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
