# GridMap Tools

AI MCP tools for Godot GridMap.

A **source-only** MCP tool extension for [Godot-MCP / AI Game Developer](https://github.com/IvanMurzak/Godot-MCP).
The package ships C# source (no compiled DLL, no bundled Godot) that compiles inside your Godot project
against your own GodotSharp, so it never locks you to a Godot version.

## Install

Requires the core [`godot_mcp`](https://github.com/IvanMurzak/Godot-MCP) addon in your Godot C# project.

```bash
# via the godot-cli (resolves from the shared catalog, edits your .csproj, rebuilds)
godot-cli install-extension com.IvanMurzak.Godot.MCP.GridMap

# …or add the reference manually and rebuild:
#   <PackageReference Include="com.IvanMurzak.Godot.MCP.GridMap" Version="0.1.0" />
```

…or pick it from the **Extensions** dock inside the Godot editor.

After a rebuild, the extension's `[AiToolType]` tool families are auto-discovered — no registry edit.

## Tools

| Tool | Kind | Description |
| --- | --- | --- |
| `gridmap-defaults` | pure-managed | Return the recommended starter config (cell size, scale, octant, centering) for a new GridMap. |
| `gridmap-create` | editor | Create a `GridMap` node in the edited scene; optionally set name, parent, and cell size X/Y/Z. |
| `gridmap-set-mesh-library` | editor | Assign a `MeshLibrary` resource (by `res://` path) to a GridMap. |
| `gridmap-set-cell` | editor | Set a single cell to a MeshLibrary item id, with an optional orthogonal orientation (0..23). |
| `gridmap-clear-cell` | editor | Clear a single cell (set it empty). |
| `gridmap-clear` | editor | Clear all cells, leaving the node and its MeshLibrary intact. |
| `gridmap-get` | editor | Read a GridMap's scalar config (cell size/scale/octant/centering, MeshLibrary path, used-cell count). |

All editor tools marshal every Godot call onto the editor main thread; cell sizes, item ids, and
orientations are clamped to valid ranges before they touch a node.

License: Apache-2.0.
