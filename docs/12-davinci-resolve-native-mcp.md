# DaVinci Resolve 21.1 Native MCP Server

DaVinci Resolve Studio 21.1, released on September 8, 2026 at IBC, ships its own MCP server. This page records what it is, what it changes for editing driven by an assistant, and what it leaves unchanged. Every line marked *verified* was read on an installed copy of Resolve Studio 21.1.0 on macOS on September 10, 2026. Everything else carries a dated source at the bottom.

---

## 1. What ships with the application

- The bundle `DaVinciResolve.mcpb` (bundle version 1.0, proprietary license, macOS, Windows, Linux) sits inside the app at `Contents/Resources/DaVinciResolve.mcpb` on macOS. It contains a Node wrapper (`server/index.js`) that proxies stdio JSON-RPC to a binary. *verified*
- The binary `ResolveMCP` lives at `/Applications/DaVinci Resolve/DaVinci Resolve.app/Contents/Applications/ResolveMCP` on macOS, `C:\Program Files\Blackmagic Design\DaVinci Resolve\ResolveMCP.exe` on Windows, `/opt/resolve/bin/ResolveMCP` on Linux. It is a launcher for the Python 3.14 runtime Resolve now embeds (`ResolvePython`). *verified*
- Transport is stdio only, MCP protocol 2024-11-05. There is no HTTP and no network mode. *verified*
- The wrapper runs `ResolveMCP --dump-tools` at startup to read the tool definitions, spawns the persistent binary on the first `tools/call`, kills it after 5 minutes of inactivity, fails queued requests if the binary does not initialize within 30 seconds. *verified*
- The instructions the server sends to the model ask the assistant to call `get_whats_new` first, because Resolve adds features faster than model training data follows. *verified*

You can list the tools without Resolve running:

```bash
BMD_IS_MCPB=1 "/Applications/DaVinci Resolve/DaVinci Resolve.app/Contents/Applications/ResolveMCP" --dump-tools
```

## 2. The 14 tools

| Tool | Parameters | What it does |
|---|---|---|
| `launch_resolve` | none | Launches Resolve and waits up to 60 s. |
| `get_resolve_status` | none | Tells whether Resolve is running and reachable. |
| `get_whats_new` | `since` | Returns changelog entries after a version or date. |
| `get_scripting_api` | `api`, `types`, `outline`, `as_file` | Returns the Python stubs (`.pyi`) of the installed API version. |
| `search_scripting_api` | `pattern`, `api` | Regex search over types, functions, constants and their descriptions. |
| `run_script` | `script`, `timeout` | Runs a Python 3.14 script with `resolve` and `project` pre-injected. Set `result` to return data; `print` output is captured. Default timeout 10 s, maximum 60 s. The sandbox blocks `os`, `sys`, `pathlib`, `shutil`, network and subprocesses. |
| `run_script_unsafe` | `script`, `timeout` | Same, with full filesystem, network and subprocess access. |
| `get_scripting_docs` | `document`, `section`, `as_file` | Serves two documents: the scripting README (32 KiB) and the DCTL reference (40 KiB), whole or by heading. |
| `list_dctls` | none | Lists `.dctl` files under the LUT folder. |
| `list_luts` | none | Lists `.cube`, `.3dl`, `.dat`, `.lut`, `.olut` files under the LUT folder. |
| `update_dctl` | `path`, `content` | Compiles the DCTL first, then writes it into `LUT/MCP`. |
| `delete_dctl` | `path` | Deletes a DCTL from `LUT/MCP`. |
| `delete_lut` | `path` | Deletes a LUT from `LUT/MCP`. |
| `generate_lut` | `path`, `size`, `transform` | Evaluates a Python function body (`r, g, b` in `[0,1]`, must return a tuple) on every lattice point and writes a `.cube` into `LUT/MCP`. Usual sizes 17, 33, 65. |

Two things the tool list does not say by itself:

- The sandbox is not a read only mode. `run_script` is declared `destructiveHint: true`. Deleting clips or starting a render through it is real.
- The 60 s cap applies to one call, not to the work. Start a render in one call, poll its status in the next.

## 3. Setup

`File > Setup AI Assistants` detects the assistants installed on the machine and writes their configuration. According to Blackmagic staff on the official forum (September 8, 2026), the detected clients are Claude Code, Claude Desktop, Antigravity (Google), Codex (OpenAI) and Grok Build (xAI). Any other MCP client can be pointed at the binary by hand.

Manual configuration for Claude Code on macOS:

```bash
claude mcp add --scope user --transport stdio davinci-resolve-native -- \
  "/Applications/DaVinci Resolve/DaVinci Resolve.app/Contents/Applications/ResolveMCP"
```

Generic stdio configuration, reported working with LM Studio and a local Qwen3 coder model (forum, September 9, 2026):

```json
{
  "mcpServers": {
    "davinci-resolve": {
      "type": "stdio",
      "command": "/Applications/DaVinci Resolve/DaVinci Resolve.app/Contents/Applications/ResolveMCP"
    }
  }
}
```

Known setup traps from the first 48 hours:

- Claude Desktop shows "Couldn't load the extension preview" until you click **Install** in the Resolve setup dialog with Claude closed, then add the connector with the **+** button in the chat (forum, September 8).
- Codex Desktop needs the `davinci_resolve` server listed and enabled under Settings > Plugins > MCPs (forum, September 8).
- An agent running on another machine cannot connect: the server is a local stdio process and needs a Studio license on the machine where it runs (forum, September 8 and 9).

## 4. Studio only, and what the free edition lost

The AI assistant integration appears on the Studio 21.1 support page and not on the free 21.1 page (Blackmagic support center, September 9, 2026). The release notes add a line that matters for every community bridge: Python scripting moved to the Studio edition. The free console keeps the scripting API, but workflow integrations, the native `UIManager` and external access now need Studio. Blackmagic's stated reason: the Python API was being used to bring Studio features into the free edition.

Consequence for the community MCP server (see section 6): its bridge for the free edition, measured on 21.0.3, no longer has a Python entry point on free 21.1 (samuelgursky/davinci-resolve-mcp issue #203, September 9, 2026).

## 5. What changed in the scripting API itself

`Developer/Scripting/` now contains `README.md` (dated August 31, 2026), `CHANGELOG.md` (September 1, 2026), the typed stub `DaVinciResolveScript.pyi` (2765 lines) and an `Examples/` folder. The old `README.txt` is gone. *verified*

21.1 additions listed in that changelog:

- Python 2 dropped; Python 3.14 embedded; `import DaVinciResolveScript` works from `ResolvePython` with no environment variable (*verified*); overloaded signatures deprecated (`GetSetting`/`SetSetting` in favour of `GetSettings`/`SetSettings`).
- Keyboard presets, project settings presets, render presets (`UpdateRenderPreset`, quick export), audio render codecs and formats.
- `MediaPoolItem.GetTranscription` with speaker and timing data.
- Multicam: `CreateMulticamClip`, `FlattenMulticam`, `PerformMulticamSmartSwitch`, `Timeline.AutoAlignClips`.
- `TimelineItem`: native audio properties, fades, speed, output blanking, `AddTransition`.
- `Timeline.NormalizeAudioLevel`, audio mappings, `MediaStorage.StartCloneMedia`.
- `resolve.ValidateDCTL` and `EncryptDCTL`.
- Shortcuts `resolve.GetCurrentProject`, `GetCurrentTimeline`, `GetMediaPool`, `GetGallery`.

For a ComfyUI roundtrip, the useful ones are the transcription, the alignment, the transitions and the clone tool. None of them is exclusive to the MCP server: any script reaches them.

## 6. Native server versus the community server

The reference community server is [samuelgursky/davinci-resolve-mcp](https://github.com/samuelgursky/davinci-resolve-mcp), version 2.223.0 on September 9, 2026. Its changelog already calls Blackmagic's server "the official Resolve MCP" and validates its own DCTL wrapper against it.

| Capability | Native 21.1 | Community 2.223.0 |
|---|---|---|
| Installation | ships with Studio, configured by the app | `npx davinci-resolve-mcp setup`, Python venv |
| Tool surface | 14 generic tools | 36 compound tools, or 376 granular, plus 18 "Advanced" tools |
| Arbitrary Python on the API | `run_script`, sandboxed | `script_plugin` `run_inline`, Python or Lua, not sandboxed |
| Installed API docs served to the agent | yes, the stubs and README of the running version | a copy of the 21.1 stub taken on September 9 through the official server, plus its own knowledge base |
| DCTL | compile and write (`update_dctl`) | static validation, plus native `ValidateDCTL` since 2.223.0; no write tool |
| LUT generation from a function | `generate_lut` | not found |
| `.drx` grade files, DRP/DRT work without Resolve running | no | Advanced server |
| Guardrails on destructive operations, operation traces | none beyond MCP annotations | yes |
| Media analysis, batch CLI, headless edit loop | no | yes |
| Free edition | Studio only | in-app bridge for 21.0.x only |
| Network transport | none | local HTTP with token, not a remote service |

A working rule: use the native server for exploration, short scripts, DCTL and LUT work, and for reading the documentation of the version you actually run. Use the community server when you want a typed tool with argument checks, media analysis, batch jobs or `.drx` files. Keep anything repeatable in a versioned script; neither server replaces that.

Brent Schooley, who produces developer videos at OpenAI and had built his own 164-action Resolve bridge with Codex before 21.1, put the trade-off in one sentence on September 9, 2026: "It's built in and maps everything in Resolve and will be updated with each release of Resolve. I'll take that over an external dependency every day of the week."

## 7. What does not change

- No arbitrary Color nodes: the `Graph` class exposes LUTs, cache, labels, node enable state and `ApplyGradeFromDRX`. No method adds, deletes or connects Color nodes; no curve, window or Color tracker setter. Fusion graphs are a different API and can be built by script.
- Fairlight: volume, pan, fades and normalization per clip are reachable in 21.1. Bus routing, plugin parameters, automation and the mixing console are not.
- No headless without a process: `-nogui` exists, and every script still needs a running, reachable Resolve.
- Timeline export by script stops at FCPXML 1.10 (`EXPORT_FCPXML_1_8`, `_1_9`, `_1_10` in the stub). *verified*
- The export, process in ComfyUI, reimport shape of section 1 of [NLE Integration](05-nle-integration.md) is unchanged. The native server makes the import and timeline steps callable by any assistant without installing anything; it does not run models.

## 8. Field reports, September 8 to 10, 2026

- Forum thread "AI Assistant - how to?" (16 replies): setup steps from Blackmagic staff; a Codex desktop session took 1 min 30 to trim the end of a clip by 5 seconds and gave up on an audio transcription after 2 min 30; a marker was placed without accounting for a timeline that starts at a non-zero timecode; "it can only act on what is scriptable"; Antigravity on Windows 10 confirmed working; a local model through LM Studio confirmed working.
- Reddit release thread: one editor cut a 50 minute interview to a 10 minute version in five minutes by linking Codex, with the setup being install Codex, run Setup AI Assistants, relaunch Resolve. Another reader adds multicam creation and audio normalization through the API to a podcast prep tool, saving 20 to 30 clicks per episode.
- A Windows 21.1.0.14 install on an RTX 5080 crashes 5 min 12 s after every launch whatever it does (forum, September 9). Nothing published links it to the MCP server.

## 9. A five-call acceptance test

Run these as `run_script` calls on a dedicated test project before trusting the server on real work.

1. **Connection and target**: assert `resolve.IsStudio()`, `resolve.GetVersion()[:2] == [21, 1]`, `project.GetUniqueId() == resolve.GetCurrentProject().GetUniqueId()`, return the color science and frame rate from `project.GetSettings()`.
2. **Sandbox**: `import os` inside `run_script`. The expected result is a refusal. If the import succeeds, the sandbox contract is not what the tool description says.
3. **Sequence ingest**: `MediaPool.ImportMedia([{"FilePath": ".../plan.%05d.exr", "StartIndex": 1, "EndIndex": 48}])`, then assert `GetClipProperty("Frames") == "48"` and read back every metadata field you wrote with `SetMetadata`. Forty-eight isolated stills instead of one clip is the classic failure.
4. **DCTL validation**: `resolve.ValidateDCTL(identity_source)` must return `None`; `resolve.ValidateDCTL("not a DCTL")` must return a non-empty string. Note that `None` means success here, the opposite of most Resolve API methods.
5. **Alignment**: two clips on V1 and V2 whose source timecodes differ by one second, timeline starts at 0 and 48; `timeline.AutoAlignClips(items, {"SyncUsing": resolve.AUTO_ALIGN_CLIPS_USING_TIMECODE})` must return `True` and leave the starts at 0 and 24 with the same unique ids and durations.

## Sources

- Blackmagic Design press release, September 8, 2026: https://www.blackmagicdesign.com/media/release/20260908-03
- Blackmagic support center, Resolve 21.1 and Resolve Studio 21.1 update pages, read September 10, 2026: https://www.blackmagicdesign.com/support/readme/baf7c071c0524fbf8ccc961925c9f443
- Release notes as reposted on r/davinciresolve, September 8, 2026: https://www.reddit.com/r/davinciresolve/comments/1wafb08/davinci_resolve_211_release_notes/
- Blackmagic forum, "AI Assistant - how to?", September 8 to 9, 2026: https://forum.blackmagicdesign.com/viewtopic.php?f=21&t=239831
- Blackmagic forum, "official MCP in Resolve 21.1 for hermes", September 9, 2026: https://forum.blackmagicdesign.com/viewtopic.php?f=21&t=239908
- Blackmagic forum, "Resolve 21.1.0.0014 (Win) crashes exactly ~5:12 min after launch", September 9, 2026: https://forum.blackmagicdesign.com/viewtopic.php?f=21&t=239924
- samuelgursky/davinci-resolve-mcp, changelog 2.223.0 and issue #203, September 9, 2026: https://github.com/samuelgursky/davinci-resolve-mcp
- Brent Schooley on X, September 9, 2026: https://x.com/chefbrent/status/2097489890204107048 and https://x.com/chefbrent/status/2097566720705565127
- Installed files read on September 10, 2026: `DaVinciResolve.mcpb` (manifest, wrapper), `ResolveMCP --dump-tools`, `Developer/Scripting/README.md`, `CHANGELOG.md`, `DaVinciResolveScript.pyi`.
