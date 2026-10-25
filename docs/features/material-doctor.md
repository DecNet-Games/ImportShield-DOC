---
layout: default
title: Material Doctor
parent: Features
nav_order: 7
description: "Material Doctor: automated pink shader healing, Universal Render Pipeline (URP) migration, and material property translation."
---

# Material Doctor (Pink Shader Healing)

Few sights in Unity development are more demoralizing than importing an asset pack or upgrading an older project to the Universal Render Pipeline (URP) and discovering that dozens of materials have turned shocking, neon magenta.

**Material Doctor** scans, diagnoses, and repairs pink and broken shaders across your entire project with automated property translation.

---

## Why Materials Turn Pink

In Unity, a material renders bright pink/magenta (`#FF00FF`) when:
1. The assigned shader cannot be found (GUID missing or deleted).
2. The material uses a legacy Built-in Render Pipeline shader (such as `Standard`, `Mobile/Diffuse`, or `Unlit/Texture`) inside a project configured for Universal Render Pipeline (URP) or High Definition Render Pipeline (HDRP).
3. The shader contains fatal compilation errors on the current graphics platform.

Standard Unity upgrade tools often fail to preserve texture offsets, metallic values, and smoothness sliders when switching shaders, forcing developers to manually re-configure hundreds of inspector properties.

---

## The Material Doctor Repair Flow

```
[ Pink Material Detected ]
           │
           ▼
┌──────────────────────────────────────┐
│ Analyze Material YAML                │
│ - Inspect current shader GUID        │
│ - Parse serialized property maps     │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│ Shader Resolution                    │
│ - Identify URP Lit / Simple Lit GUID │
│ - Match target render pipeline       │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│ Property Translation Table           │
│ - Map legacy names to URP names      │
│ - Preserve colors, textures, floats  │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│ Fail-Closed YAML Patching            │
│ - Line-level patch applied           │
│ - Backed up to ProjectDoctorData/    │
└──────────────────────────────────────┘
```

---

## Automated Property Translation Table

When migrating legacy shaders to modern URP equivalents, Material Doctor translates serialized property names and converts range scales where necessary:

<div class="table-wrapper" markdown="block">

| Legacy Property (Built-in / Standard) | Modern URP Property (`Universal Render Pipeline/Lit`) | Description |
|:---|:---|:---|
| `_MainTex` | `_BaseMap` | Albedo / Diffuse base texture map |
| `_Color` | `_BaseColor` | Tint / diffuse colorRGBA multiplier |
| `_BumpMap` | `_BumpMap` | Tangent-space normal map |
| `_BumpScale` | `_BumpScale` | Normal map strength multiplier |
| `_MetallicGlossMap` | `_MetallicGlossMap` | Combined metallic / smoothness map |
| `_Glossiness` | `_Smoothness` | Surface smoothness float slider (`0.0` – `1.0`) |
| `_GlossMapScale` | `_SmoothnessTextureChannel` | Smoothness source channel setting |
| `_OcclusionMap` | `_OcclusionMap` | Ambient occlusion texture map |
| `_OcclusionStrength` | `_OcclusionStrength` | Ambient occlusion multiplier |
| `_EmissionColor` | `_EmissionColor` | HDR emission color |
| `_EmissionMap` | `_EmissionMap` | Emission texture mask |

</div>

---

## Operating Material Doctor

1. Open the **Materials** tab in the ImportShield Window:
   ```
   Tools ▸ ImportShield ▸ ImportShield Window ▸ Materials
   ```
2. Click **Scan Materials**. Material Doctor inspects all `.mat` files in your project.
3. Materials with missing or pipeline-incompatible shaders are displayed in the interactive grid.
4. Select individual materials or click **Select All Broken**.
5. Choose your target shader:
   - **Universal Render Pipeline/Lit** (Standard PBR lighting)
   - **Universal Render Pipeline/Simple Lit** (Optimized for mobile / stylized)
   - **Universal Render Pipeline/Unlit** (UI / Particle / Flat style)
6. Click **Migrate Selected (Preview)**.
7. Inspect the two-pane diff in the Diff Viewer to verify that texture GUIDs and color floats are preserved.
8. Click **Apply Plan** — materials instantly refresh in your Scene View with correct textures and lighting restored.

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
