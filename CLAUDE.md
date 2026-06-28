# CLAUDE.md — Godot-AI-GridMap

A **Godot-MCP extension**: an MCP tool family for Godot's built-in `GridMap` node (a 3D grid of cells,
each holding a mesh from an assigned `MeshLibrary`), shipped as a **source-only NuGet package**
(`com.IvanMurzak.Godot.MCP.GridMap`) that compiles inside a consumer's Godot project against the
consumer's own GodotSharp. Created from
[`Godot-AI-Tools-Template`](https://github.com/IvanMurzak/Godot-AI-Tools-Template). The packaging recipe
is the load-bearing detail — read `docs/source-only-nuget-recipe.md`.

## Layout

- `src/Godot-AI-GridMap/` — the source-only package (`Godot.NET.Sdk`).
  - `Runtime/Tools/Tool_GridMap.cs` — the `[AiToolType]` family (one partial class).
  - `Runtime/Tools/Tool_GridMap.Ids.cs` — all tool-id consts (pure-managed; pinned by tests).
  - `Runtime/Tools/Tool_GridMap.Defaults.cs` — `gridmap-defaults` (pure-managed tool).
  - `Runtime/GridMap/` — pure-managed support types: `GridMapDefaults` (starter config +
    value-clamping rules, all unit-tested) and any cell/coordinate helpers.
  - `Editor/Tools/Tool_GridMap.{Editor,Create,SetMeshLibrary,SetCell,ClearCell,Clear,Get}.cs` — editor
    tools behind `#if TOOLS` (touch `EditorInterface`/live nodes; main-thread-marshalled; E2E-verified).
  - `build/com.IvanMurzak.Godot.MCP.GridMap.props` — the source-injection props (auto-imported by NuGet
    in the consumer; MUST stay named `<PackageId>.props`).
- `tests/Godot-AI-GridMap.Tests/` — xUnit specs for the pure-managed sources only (no Godot binary).
- `testbed/GridMap-Testbed.csproj` — a consumer `Godot.NET.Sdk` project that restores the local-packed
  package; `dotnet build` of it is the source-injection proof.

## Tools

| Tool | Kind | File |
| --- | --- | --- |
| `gridmap-defaults` | pure-managed | `Runtime/Tools/Tool_GridMap.Defaults.cs` |
| `gridmap-create` | editor | `Editor/Tools/Tool_GridMap.Create.cs` |
| `gridmap-set-mesh-library` | editor | `Editor/Tools/Tool_GridMap.SetMeshLibrary.cs` |
| `gridmap-set-cell` | editor | `Editor/Tools/Tool_GridMap.SetCell.cs` |
| `gridmap-clear-cell` | editor | `Editor/Tools/Tool_GridMap.ClearCell.cs` |
| `gridmap-clear` | editor | `Editor/Tools/Tool_GridMap.Clear.cs` |
| `gridmap-get` | editor | `Editor/Tools/Tool_GridMap.Get.cs` |

## Build / test (no Godot binary)

```bash
dotnet build src/Godot-AI-GridMap/Godot-AI-GridMap.csproj   # source-only package compiles tools
dotnet test  tests/Godot-AI-GridMap.Tests/Godot-AI-GridMap.Tests.csproj
dotnet pack  src/Godot-AI-GridMap/Godot-AI-GridMap.csproj -p:Version=0.0.0-ci -o local-nuget
dotnet build testbed/GridMap-Testbed.csproj                  # consumes the local package (injection proof)
```

`Godot.NET.Sdk` supplies GodotSharp from NuGet, so no Godot install is needed to build/test/pack or to
prove the source-injection recipe (the testbed build is a faithful proxy for `godot --build-solutions`).
The recipe is verified to compile into the consumer across the **Godot 4.3 / 4.4 / 4.5** CI matrix. When
proving locally, note `dotnet pack` re-uses the **global NuGet cache** for an already-cached version: if
you re-pack the same `Version`, clear `~/.nuget/packages/com.ivanmurzak.godot.mcp.gridmap/<ver>` (or pack
a unique version) before re-restoring the testbed, or you'll silently build the stale cached source.

## Conventions

- Root namespace `com.IvanMurzak.Godot.MCP.GridMap`. Every `.cs` starts with the Apache-2.0 header.
- **Namespace-shadow gotcha:** the root namespace `com.IvanMurzak.Godot.MCP.GridMap` SHADOWS the engine
  type `Godot.GridMap`, so an unqualified `GridMap` binds to the namespace (`CS0118: 'GridMap' is a
  namespace but is used like a type`). In every editor file that names the engine type, add a per-file
  alias `using GdGridMap = Godot.GridMap;` (or fully-qualify `Godot.GridMap`).
- Pure-managed tools (no Godot native API) → `Runtime/` (outside `#if TOOLS`, unit-testable); editor-driving
  tools → `Editor/` (behind `#if TOOLS`, every Godot call via `MainThread.Instance.Run(...)`, E2E-verified).
- The package declares ONLY the `com.IvanMurzak.McpPlugin` / `com.IvanMurzak.ReflectorNet` min-version
  deps; **GodotSharp must never become a package dependency** (CI asserts the nuspec). Keep the MCP pins in
  lockstep with the core Godot-MCP addon; bump with `commands/update-core.ps1`.
- One `[AiToolType] partial class Tool_GridMap`; one `[AiTool]` method per partial-class file. New
  pure-managed sources must be added to the test csproj `<Compile Include>` list to be unit-tested.

## Find detail in

- `docs/source-only-nuget-recipe.md` — the packaging recipe (the centerpiece) + the consumer story.
- `docs/ci.md` — workflows, the version gate, multi-Godot matrix, required secrets.
