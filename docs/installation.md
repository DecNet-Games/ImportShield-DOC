---
layout: default
title: Installation
nav_order: 3
description: "How to purchase, install, and configure ImportShield via the Unity Asset Store."
permalink: /docs/installation/
---

# Installation & Setup

ImportShield is distributed commercially as an Editor package for the Unity engine.

---

## Purchasing & Distribution Notice

{: .important }
> **Commercial Distribution Notice:**
> ImportShield is purchased and licensed exclusively through the **Unity Asset Store**.
> The GitHub repository (`https://github.com/DecNet-Games/ImportShield-DOC`) hosts the public documentation, issue tracking, and automated CI workflows — it is **not** an alternative distribution source for the commercial package.

---

## Installing via Unity Package Manager

Once purchased on the Unity Asset Store:

1. Open your project in the Unity Editor.
2. From the top menu, navigate to:
   ```
   Window ▸ Package Manager
   ```
3. In the Package Manager toolbar drop-down, select **Packages: My Assets**.
4. Locate **ImportShield** in your list of purchased assets.
5. Click **Download** (if not already downloaded locally), then click **Import**.
6. When the Unity package import dialog appears, leave all components selected and click **Import**.

Upon import completion, ImportShield immediately initializes its background indexing hooks. You will see a welcoming notification and the **🟢 ImportShield** indicator pill will appear in your Scene View overlay.

---

## Prerequisites & Compatibility

ImportShield was engineered from day one to operate with zero friction across modern Unity environments:

- **Unity Editor Version**: Unity **2022.3 LTS** or newer (both **Unity 2022.3** and **Unity 6** are officially supported and tested).
- **Dependencies**: **Zero third-party dependencies**. ImportShield does not require external NuGet packages, third-party DLLs, Python runtimes, or external databases.
- **Render Pipelines**: 100% pipeline-agnostic. Fully compatible with:
  - Universal Render Pipeline (URP)
  - High Definition Render Pipeline (HDRP)
  - Built-in Render Pipeline
  - Custom Scriptable Render Pipelines (SRP)
- **Editor-Only**: All code is strictly segregated under `ProjectDoctor.Editor` and `ProjectDoctor.Cli` assemblies, with `ProjectDoctor.Core` compiled with `"noEngineReferences": true`. Zero bytes of ImportShield code are ever included in your runtime standalone player builds.

---

## Recommended Editor Configuration

While ImportShield functions across all standard Unity project configurations, following these best practices guarantees optimal speed and precision:

### 1. Asset Serialization: Force Text
ImportShield performs byte-accurate line repairs directly on serialized YAML files.
- Navigate to: **Edit ▸ Project Settings ▸ Editor ▸ Asset Serialization ▸ Mode**.
- Ensure the mode is set to **Force Text** (the default setting in modern Unity).
- *Note:* If your project uses **Force Binary**, ImportShield falls back to asset-level dependency graph analysis and warns you prior to attempting text diffs.

### 2. Version Control Mode: Visible Meta Files
- Navigate to: **Edit ▸ Project Settings ▸ Editor ▸ Version Control ▸ Mode**.
- Ensure mode is set to **Visible Meta Files**. This allows ImportShield to inspect, backup, and restore `.meta` files alongside asset files.

---

## Testing & Verification Suite

If you are evaluating ImportShield or incorporating it into automated team workflows, you can execute the bundled test suites:

### Unity EditMode Integration Tests
To execute the EditMode test suite from within your Unity Editor:
1. Ensure `com.projectdoctor` is registered under `testables` in your `Packages/manifest.json`:
   ```json
   {
     "testables": ["com.projectdoctor"]
   }
   ```
2. Open **Window ▸ General ▸ Test Runner**.
3. Select the **EditMode** tab and click **Run all**.

### Pure .NET Core Unit Test Suite
The underlying core engine (`ProjectDoctor.Core`) is completely decoupled from the Unity engine runtime and can be tested headlessly on any machine with the .NET SDK:

```bash
dotnet test Packages/com.projectdoctor/Tests~/Core/ProjectDoctor.Tests.csproj
```

---

## Safety Commitment

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*

Every installation includes built-in transactional journals and crash recovery monitors. Proceed to [**Quick Start**]({{ site.baseurl }}/docs/quick-start/) to run your first protected operation.
