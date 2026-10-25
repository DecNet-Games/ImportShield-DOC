---
layout: default
title: CLI Automation API
parent: Public API
nav_order: 3
description: "CLI Automation API reference: CommandLineRunner interface, arguments, and batchmode execution."
---

# CLI Automation API Reference

The `ProjectDoctor.Cli` assembly exposes the headless execution engine used to integrate ImportShield into CI/CD build agents and command-line automation tools.

---

## The `CommandLineRunner` Class

The primary entry point is `ProjectDoctor.Cli.CommandLineRunner.Run()`.

### Method Signature
```csharp
namespace ProjectDoctor.Cli
{
    public static class CommandLineRunner
    {
        public static void Run();
        public static int ExecuteWithArgs(string[] args);
    }
}
```

When invoked via Unity's `-executeMethod ProjectDoctor.Cli.CommandLineRunner.Run`, the runner:
1. Automatically retrieves command-line arguments passed to the Unity process via `System.Environment.GetCommandLineArgs()`.
2. Parses `-pdFailOn`, `-pdBaseline`, `-pdReport`, and `-pdLogVerbose` flags.
3. Initializes the `ProjectScanner` and performs an incremental index.
4. Compares the current reference graph against the selected baseline.
5. Evaluates configured metric assertions.
6. Writes test artifact reports if requested.
7. Calls `EditorApplication.Exit(exitCode)` with code `0`, `1`, or `2`.

---

## Programmatic Invocation Example

If you are developing custom editor automation or your own build script (`BuildPlayer` pipeline), you can call the execution engine directly:

```csharp
using ProjectDoctor.Cli;

public static class CustomBuildPipeline
{
    public static void PreBuildVerification()
    {
        string[] args = new[]
        {
            "-pdFailOn", "new-broken-refs:0",
            "-pdBaseline", "latest",
            "-pdReport", "json"
        };

        int exitCode = CommandLineRunner.ExecuteWithArgs(args);

        if (exitCode != 0)
        {
            throw new System.Exception($"ImportShield pre-build gate failed with exit code: {exitCode}");
        }

        UnityEngine.Debug.Log("ImportShield verification passed. Proceeding with player build.");
    }
}
```

---

> *"No tool can truthfully guarantee 'zero risk' or 'zero data loss'; ImportShield fails closed and tells you the truth instead of pretending otherwise."*
