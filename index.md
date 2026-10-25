---
layout: default
title: Home
nav_order: 1
description: "ImportShield for Unity — Import Safety Net for Unity — Preflight Verdicts, Blast-Radius Map & Undoable Repairs."
permalink: /
---

# ImportShield

### Import Safety Net for Unity — Preflight Verdicts, Blast-Radius Map & Undoable Repairs

<p align="center">
  <img src="{{ site.baseurl }}/assets/images/cover.png" alt="ImportShield Cover" style="max-width: 100%; border-radius: 8px; box-shadow: 0 4px 20px rgba(0,0,0,0.5);">
</p>

## Stop Fearing the Import Button.

Every Unity developer knows the dread: you download an exciting new package from the Asset Store, click **Import**, and watch your console erupt in red compilation errors and missing script warnings. Materials turn neon pink, prefab links dissolve, and scenes lose their references.

**ImportShield** is the commercial-grade import safety net and reference repair suite for the Unity Editor. It acts as an automated flight recorder and surgical repair engine:

- **Pre-Import Freeze**: Automatically takes an atomic snapshot of your project's reference graph and byte-backs up assets before an import touches disk.
- **Package Preflight**: Drops `.unitypackage` files into a sandboxed preflight scanner for instant **GREEN / AMBER / RED** safety verdicts before modifying your project.
- **Blast-Radius Visualizer**: Immediately visualizes what the import touched, distinguishing brand-new breakages from pre-existing issues.
- **Evidence-Scored Repairs**: Proposes surgical YAML-level repairs with ranked confidence bands, detailed cause attribution, and a two-pane diff preview.
- **Fail-Closed Rollback**: 1-click transactional rollback restores byte-identical originals, even if Unity crashed or was restarted.

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*

---

## Key Value Propositions

<div class="table-wrapper" markdown="block">

| Capability | What Traditional Workflows Do | What ImportShield Delivers |
|:---|:---|:---|
| **Import Safety** | Blindly extract `.unitypackage` files; hope Git catches the mess or spend hours reverting. | **Package Preflight** scans the archive for overwrites, GUID collisions, and class name duplicates before extraction. |
| **Reference Repair** | Manually re-drag missing scripts and assets onto broken inspectors across dozens of prefabs. | **Evidence-Ranked Recovery** matches moved assets, renamed classes, and content hashes with line-level YAML patching. |
| **Disaster Recovery** | Lose hours manually unstaging Git changes or re-downloading deleted assets. | **Fail-Closed Transactions** store atomic byte backups in `ProjectDoctorData/` for 1-click rollback outside Git. |
| **Pipeline Upgrades** | Manually re-assign shaders on dozens of pink materials across the project. | **Material Doctor** heals legacy Built-in shaders to URP Lit / Simple Lit with automatic property translation. |
| **Dead Component Cleanup** | Click "Remove Component" one by one on hundreds of `Missing (MonoBehaviour)` warnings. | **Missing Component Stripper** safely removes dead script slots from YAML arrays without corrupting siblings. |
| **Asset Cleanup** | Run dangerous sweepers that delete dynamic assets (`Resources.Load`) and break runtime builds. | **Reversible Quarantine** moves unreferenced files to safety while tagging dynamic matches and preserving GUIDs. |

</div>

---

## Quick Navigation

Ready to safeguard your project? Jump directly into the guides:

- [**Getting Started**]({{ site.baseurl }}/docs/getting-started/): Understand the mental model, 3-step onboarding, and baseline protection status.
- [**Installation**]({{ site.baseurl }}/docs/installation/): Install via the Unity Asset Store and configure package prerequisites.
- [**Quick Start (60 Seconds)**]({{ site.baseurl }}/docs/quick-start/): Step-by-step walkthrough of your first protected import and repair.
- [**Features Overview**]({{ site.baseurl }}/docs/features/): Explore the complete matrix of 12 protection and repair engines.
- [**System Architecture**]({{ site.baseurl }}/docs/architecture/): Deep-dive into our Observation &rarr; Diagnosis &rarr; Repair pipeline.
- [**Public API Reference**]({{ site.baseurl }}/docs/api/): Drive ImportShield headlessly from CI/CD pipelines or custom editor extensions.
- [**Interactive Sample Walkthrough**]({{ site.baseurl }}/docs/samples/broken-import-walkthrough/): Experience a real-world broken import scenario using `FantasyProps_v2.unitypackage`.
- [**FAQ**]({{ site.baseurl }}/docs/faq/): Answers to common questions regarding Git independence, crash recovery, and privacy.
- [**Honest Limitations**]({{ site.baseurl }}/docs/limitations/): Our transparent scope boundaries and non-negotiable guarantees.
