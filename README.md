# kicad-mcp-dsh-plugin

[DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) (dsh) plugin that bridges [kicad-mcp-pro](https://github.com/oaslananka/kicad-mcp-pro) into dsh as an MCP tool family (`mcp__kicad__*`): inspect KiCad schematics and PCBs, run ERC/DRC, review BOM/DFM, and export fabrication outputs — without leaving your dsh session.

This is a thin bridge: it contains no code of its own. It registers kicad-mcp-pro as a stdio MCP server via the official `@deepseek-ai/dsh-mcp-client` and forwards configuration through environment variables.

## Install

```bash
dsh plugin --profile web add github:chunhuachaolu-art/kicad-mcp-dsh-plugin
```

(Works with any profile: `web`, `tui`, `headless`, ...)

## Requirements

- [dsh](https://www.npmjs.com/package/@deepseek-ai/dsh) (`npm i -g @deepseek-ai/dsh`)
- Python **>= 3.13** with [kicad-mcp-pro](https://pypi.org/project/kicad-mcp-pro/) installed:
  ```bash
  pip install kicad-mcp-pro
  ```
  The `kicad-mcp-pro` executable must be reachable (on PATH, or point `KICAD_MCP_CMD` at it).
- KiCad 9 or 10. File-based tools (ERC/DRC/BOM/gerber/step exports) run headless through `kicad-cli`; board-interactive tools additionally require the KiCad GUI running with its socket API enabled.

## Configuration

All knobs are environment variables read by the plugin at dsh startup:

| Variable | Default | Meaning |
|---|---|---|
| `KICAD_MCP_CMD` | `kicad-mcp-pro` | Command used to launch the server. Set to a full path (e.g. a venv's `Scripts/kicad-mcp-pro.exe`) if it is not on PATH. |
| `KICAD_MCP_PROFILE` | `default` | Tool profile of kicad-mcp-pro: `default` (24 read-only tools), `review`, `build`, `release`, `expert`, `full`, `read_only_inspection`, ... |
| `KICAD_MCP_KICAD_CLI` | auto-detect | Explicit path to `kicad-cli` if it is not on PATH. |
| `KICAD_MCP_LOG_LEVEL` | `warning` | Server-side log level. |

Example — pointing at a dedicated venv and a non-PATH KiCad:

```bash
export KICAD_MCP_CMD=/opt/kicad-venv/bin/kicad-mcp-pro
export KICAD_MCP_KICAD_CLI="F:/kicad/bin/kicad-cli.exe"
dsh web
```

## Troubleshooting

- **Tools listed by `--dump-config` but missing in the live session** — `dump-config` prints the composed config tree; it does **not** prove the plugin row actually loads. At startup dsh enforces plugin dependency versions and disables incompatible rows, logging e.g. `dsh: disabling profile plugin row "mcp-kicad": Plugin @deepseek-ai/dsh-mcp-client@… is incompatible with dsh <version>`. Since v0.1.1 this plugin pins `@deepseek-ai/dsh-mcp-client@0.2.1-rc.2` (matching dsh 0.2.1-rc.x), so the gate should not trigger; on version mismatches you can still allow the pinned pair explicitly:
  ```bash
  dsh plugin --profile web allow-version "@deepseek-ai/dsh-mcp-client@0.2.1-rc.2" --dsh-version 0.2.1-rc.2 --accept-risk
  ```
- **`kicad-mcp-pro` not found** — run `kicad-mcp-pro --help` in the same shell you launch dsh from; if that fails, `pip install kicad-mcp-pro` (needs Python >= 3.13) or set `KICAD_MCP_CMD` to the full executable path.
- **DRC/ERC report "kicad-cli not found"** — set `KICAD_MCP_KICAD_CLI` to your `kicad-cli` executable.
- **Board-interactive tools error out** — start the KiCad application and enable its socket API (Preferences → Session → socket); file-based tools work headless regardless.

## Credits & license

- All heavy lifting is done by [kicad-mcp-pro](https://github.com/oaslananka/kicad-mcp-pro) (MIT, by oaslananka) and the official `@deepseek-ai/dsh-mcp-client`.
- This plugin: MIT — see [LICENSE](LICENSE).
