<h1 align="center">Godot AI GridMap</h1>

<p align="center">
  AI <b>MCP tools</b> for Godot's built-in <b>GridMap</b> (3D tile-based maps driven by a
  <code>MeshLibrary</code>) — an extension for
  <a href="https://github.com/IvanMurzak/Godot-MCP">Godot-MCP / AI Game Developer</a>.
</p>

`Godot-AI-GridMap` is a focused MCP tool family for Godot's built-in `GridMap` node — the 3D
grid of cells where each cell holds a mesh from an assigned `MeshLibrary`. The tools are authored in
C# with `[AiToolType]` / `[AiTool]` (the same model as Unity-MCP and the core Godot-MCP addon), and
shipped as a **source-only NuGet package** that compiles inside any consumer's Godot project against
the consumer's own GodotSharp — no bundled Godot, no version lock. Created from
[`Godot-AI-Tools-Template`](https://github.com/IvanMurzak/Godot-AI-Tools-Template).

## Tools

| Tool | Kind | Description |
| --- | --- | --- |
| `gridmap-defaults` | pure-managed | Return the recommended starter config (cell size, default item id) for a new GridMap. |
| `gridmap-create` | editor (`#if TOOLS`) | Create a `GridMap` node in the edited scene; optional parent, name, `cell_size` (x,y,z). |
| `gridmap-set-mesh-library` | editor (`#if TOOLS`) | Assign a `MeshLibrary` resource (by `res://` path) to a GridMap. |
| `gridmap-set-cell` | editor (`#if TOOLS`) | Set a cell at (x,y,z) to a `MeshLibrary` item id, with optional orientation. |
| `gridmap-clear-cell` | editor (`#if TOOLS`) | Clear a single cell at (x,y,z). |
| `gridmap-clear` | editor (`#if TOOLS`) | Clear every cell in a GridMap. |
| `gridmap-get` | editor (`#if TOOLS`) | Read a GridMap's scalar config (read-only). |

Pure-managed tools (no Godot native API) live under `src/Godot-AI-GridMap/Runtime/` and are
CI-unit-tested; editor-driving tools live under `Editor/` behind `#if TOOLS` and marshal every Godot
call onto the editor main thread via `MainThread.Instance.Run(...)`.

## Install (in a consumer Godot project)

Requires the core [`godot_mcp`](https://github.com/IvanMurzak/Godot-MCP) addon. Then either:

- **Extensions dock** — pick it inside the Godot editor (Install → adds the `<PackageReference>` → rebuild).
- **CLI** — `godot-cli install-extension com.IvanMurzak.Godot.MCP.GridMap`.
- **By hand** — add `<PackageReference Include="com.IvanMurzak.Godot.MCP.GridMap" Version="x.y.z" />`
  to the consumer `.csproj` and rebuild.

After a rebuild the `[AiToolType]` tool family is auto-discovered — no registry edit.

## Build & test (no Godot binary needed)

`Godot.NET.Sdk` pulls GodotSharp from NuGet, so the package builds and unit-tests headless:

```bash
dotnet build src/Godot-AI-GridMap/Godot-AI-GridMap.csproj            # compiles tools (Godot API resolves)
dotnet test  tests/Godot-AI-GridMap.Tests/Godot-AI-GridMap.Tests.csproj   # pure-managed unit tests
dotnet pack  src/Godot-AI-GridMap/Godot-AI-GridMap.csproj -p:Version=0.0.0-ci -o local-nuget
dotnet build testbed/GridMap-Testbed.csproj                          # consumer build = source-injection proof
```

The testbed build proves the source-injection recipe: the package's `.cs` are injected as `<Compile>`
items into the consumer and compile against the consumer's own GodotSharp. CI runs this across a
multi-Godot-version matrix; an end-to-end leg additionally boots real headless Godot, installs the core
addon, and calls each tool via `godot-cli run-tool`.

## Docs

- `docs/source-only-nuget-recipe.md` — the packaging recipe (the centerpiece).
- `docs/ci.md` — workflows, the version gate, the multi-Godot matrix, required secrets.
- `CLAUDE.md` — maintainer notes.

## Publish

Source-only, version-gated release (see `docs/ci.md`): bump `<Version>`
(`commands/bump-version.ps1 -NewVersion x.y.z`), merge to `main`; `release.yml` runs the full matrix,
publishes the package to NuGet via trusted publishing (OIDC), and cuts an atomic GitHub Release.

License: **Apache-2.0**.
