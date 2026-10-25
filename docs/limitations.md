---
layout: default
title: Honest Limitations & Scope
nav_order: 10
description: "Honest technical limitations and scope boundaries of ImportShield."
permalink: /docs/limitations/
---

# Honest Limitations & Scope

To ensure 100% safety and transparency, ImportShield adheres strictly to these technical boundaries:

1. **Deleted C# Source Code**: ImportShield repairs serialized references to classes and assemblies, but does NOT decompile or invent deleted C# source files.
2. **Dynamic String Lookups**: Dynamic lookups such as `Resources.Load("name")` or string-based Addressable keys cannot be proven statically with 100% certainty. ImportShield scans `.cs` files for string matches and marks potential targets with the **Possibly Dynamic** tag, disabling bulk-deletion for safety.
3. **Asset Serialization**: Accuracy is highest when Unity's Project Settings > Editor > Asset Serialization is set to **Force Text** (default in modern Unity). When set to Force Binary, asset-level dependency analysis is used.
4. **Editor-Only**: ImportShield contains zero runtime code and has zero performance or memory footprint in your built games.

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
