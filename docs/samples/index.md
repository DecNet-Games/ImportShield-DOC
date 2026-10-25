---
layout: default
title: Samples Overview
nav_order: 8
has_children: true
permalink: /docs/samples/
description: "Overview of bundled samples, including the Broken Import Demo (Samples~/BrokenImportDemo)."
---

# Bundled Samples Overview

ImportShield includes hands-on demonstration assets and interactive sample projects so you can experience and evaluate its protection, diagnosis, and repair capabilities in a safe, controlled sandbox.

---

## The Broken Import Demo (`Samples~/BrokenImportDemo`)

The primary bundled sample is the **Broken Import Demo**. It ships directly with the package and contains:

- **Arena Scene (`Arena.unity`)**: A modular combat arena containing character spawners, doors, weapons, and pickups.
- **Prefabs**: `Sword.prefab`, `Shield.prefab`, `Door.prefab`, `Enemy_A.prefab`, `Enemy_B.prefab`, `Pickup_Health.prefab`.
- **ScriptableObjects**: `WeaponData_Sword.asset`, `WeaponData_Shield.asset`.
- **Scripts**: `PlayerController.cs`, `Weapon.cs`, `WeaponData.cs`, `DoorController.cs`, `EnemyAI.cs`, `Pickup.cs`.
- **Materials**: 6 distinct materials configured for arena props.
- **`FantasyProps_v2.unitypackage`**: A specially crafted test package that simulates real-world asset store package collisions (overwriting `Weapon.cs` with a new GUID, replacing `Door.prefab`, and introducing pipeline shader discrepancies).

---

## Installing the Sample

You can import the sample into your project using the standard Unity Package Manager interface:

1. Open **Window ▸ Package Manager**.
2. Select **ImportShield / ImportShield** from the list of installed packages.
3. In the package details pane, expand the **Samples** section.
4. Locate **Broken Import Demo** and click **Import**.
5. Unity installs the sample files into `Assets/Samples/ImportShield/<version>/BrokenImportDemo/`.

---

## Interactive Walkthrough

Follow our step-by-step tutorial to experience the entire observation, diagnosis, and repair cycle:

- [**Broken Import Walkthrough**]({{ site.baseurl }}/docs/samples/broken-import-walkthrough/): A 3-stage interactive guide through baseline protection, package preflight, blast-radius investigation, and transactional repair.

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
