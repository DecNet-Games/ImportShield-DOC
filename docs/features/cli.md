---
layout: default
title: Command Line Interface (CLI)
parent: Features
nav_order: 11
description: "CommandLineRunner for CI/CD automation: headless batchmode execution, assertions, report exports, and exit codes."
---

# Command Line Interface (CLI) & CI/CD Gates

ImportShield includes a dedicated headless execution runner (`ProjectDoctor.Cli.CommandLineRunner`) designed to operate seamlessly in automated continuous integration (CI) and build verification pipelines.

---

## Why Run in CI/CD?

Prevent broken references and missing scripts from ever reaching your team's `main` development branch:
- **Block Bad Merges**: Fail pull requests automatically if an asset import or script rename introduces dangling pointers.
- **Enforce Reference Hygiene**: Set strict thresholds on newly broken references (`new-broken-refs:0`).
- **Automated Health Reports**: Generate HTML and JSON audit artifacts for every build job.

---

## Invocation Syntax

Invoke Unity in headless batchmode with the `CommandLineRunner.Run` execution method:

```powershell
Unity.exe -batchmode -nographics -projectPath "C:/Path/To/Project" `
    -executeMethod ProjectDoctor.Cli.CommandLineRunner.Run `
    -pdFailOn new-broken-refs:0 `
    -pdBaseline latest `
    -pdReport html `
    -quit
```

On Linux or macOS CI agents:

```bash
/opt/unity/Editor/Unity -batchmode -nographics -projectPath "$(pwd)" \
    -executeMethod ProjectDoctor.Cli.CommandLineRunner.Run \
    -pdFailOn new-broken-refs:0 \
    -pdBaseline latest \
    -pdReport html \
    -quit
```

---

## Supported CLI Flags

<div class="table-wrapper" markdown="block">

| Flag | Argument Syntax | Default | Description |
|:---|:---|:---|:---|
| `-pdFailOn` | `<metric>:<maxAllowed>` | *(None)* | Sets an assertion failure gate. If `<metric>` exceeds `<maxAllowed>`, the process exits with code `1`. |
| `-pdBaseline` | `latest` \| `initial` \| `<SnapshotId>` | `latest` | Specifies which baseline snapshot to compare current state against when calculating new breaks. |
| `-pdReport` | `html` \| `json` \| `both` | *(None)* | Generates structured diagnostic test reports in `ProjectDoctorData/Reports/`. |
| `-pdLogVerbose` | *(Boolean flag)* | `false` | Outputs detailed line-level parsing telemetry to the standard Unity editor log. |

</div>

### Available Metrics for `-pdFailOn`:
- `new-broken-refs`: The number of references broken *since the specified baseline snapshot*. (Recommended: `0`).
- `all-broken-refs`: Total count of unresolved references across the entire project.
- `missing-scripts`: Total count of `Missing (MonoBehaviour)` components.

---

## Exit Codes

ImportShield sets standard exit codes to integrate directly with automated build runners:

- **`0` (Success)**: All assertions passed; zero threshold violations detected.
- **`1` (Assertion Failure)**: One or more assertions failed (e.g. newly broken references exceeded max allowed).
- **`2` (Internal Exception)**: Execution failed due to an unhandled runtime error or missing project files.

---

## GitHub Actions Integration Example

Integrate ImportShield into your GitHub Actions workflow:

```yaml
name: Verify Reference Integrity

on:
  pull_request:
    branches: [ main ]

jobs:
  reference-gate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Project
        uses: actions/checkout@v4

      - name: Run ImportShield CI Gate
        run: |
          /opt/unity/Editor/Unity -batchmode -nographics -projectPath . \
            -executeMethod ProjectDoctor.Cli.CommandLineRunner.Run \
            -pdFailOn new-broken-refs:0 \
            -pdReport html \
            -quit

      - name: Upload Health Report Artifact
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: project-doctor-health-report
          path: ProjectDoctorData/Reports/
```

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
