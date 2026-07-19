# NLE Integration with ComfyUI

Status of ComfyUI integration with professional video editing software. Honest assessments.

**TL;DR**: No NLE has native ComfyUI integration. DaVinci Resolve is closest. The workflow today is export -> process in ComfyUI -> reimport.

---

## 1. DaVinci Resolve + ComfyUI

### Stability: 5/10 -- Best positioned

**Direct integrations**:

- [comfyUI_DaVinciResolve](https://github.com/barckley75/comfyUI_DaVinciResolve) -- TTS audio only. Dormant. 2/10 stability.
- [DaVinci Resolve MCP (Gursky)](https://github.com/samuelgursky/davinci-resolve-mcp) -- MCP bridge for AI assistants. Claude/Cursor can control Resolve.
- [DaVinci Resolve MCP (Tooflex)](https://glama.ai/mcp/servers/@Tooflex/davinci-resolve-mcp) -- Can execute Lua scripts in Fusion, control all pages.

**Why Resolve is the best option**:
- Full [Python/Lua scripting API](https://deric.github.io/DaVinciResolve-API-Docs/)
- Fusion has its own [scripting guide](https://documents.blackmagicdesign.com/UserManuals/Fusion8_Scripting_Guide.pdf)
- 2 MCP servers already exist
- A custom pipeline is technically feasible:
  1. Script exports clips from timeline (Python API)
  2. Send to ComfyUI via REST/WebSocket API
  3. Receive processed frames
  4. Re-import into timeline via script

**No one has built a production-ready roundtrip yet.** This is the development opportunity.

**Training resources**:
- [Mixing Light ComfyUI Series](https://mixinglight.com/color-grading-tutorials/comfyui-localhost-overview-digital-post-production-part-1/) (5 parts, Oct 2025 - Feb 2026) by Igor Ridjanovic
- [Blackmagic Forum discussions](https://forum.blackmagicdesign.com/viewtopic.php?t=203462&p=1056812) on depth maps + Resolve relight
- [ActionVFX "ComfyUI for VFX"](https://www.actionvfx.com/blog/comfyui-for-vfx) -- EXR workflow for compositing

---

## 2. Final Cut Pro + ComfyUI

### Stability: 7/10 -- Production-ready via SpliceKit MCP (updated April 2026)

**Update April 2026**: The FCP situation changed significantly with [SpliceKit](https://github.com/elliotttate/SpliceKit), a dylib injected directly into FCP's process that exposes 78,000+ ObjC classes via JSON-RPC on localhost:9876. Claude Code connects as an MCP client with 200+ tools. This makes FCP programmatically controllable at a level no other NLE offers today.

**Working workflow (FCP ↔ ComfyUI via SpliceKit):**
1. `export_xml("/tmp/edit.fcpxml")`: export timeline programmatically (no dialog)
2. Parse FCPXML in Python, extract clip list + timings
3. Run frames through ComfyUI pipeline (Wan 2.2, LTX-2, Flux.2, etc.)
4. `import_fcpxml("/tmp/comp.fcpxml")`: inject result back into FCP

**Available SpliceKit tools relevant to ComfyUI pipeline:**
- `export_xml()` / `import_fcpxml()`: FCPXML roundtrip without dialogs
- `get_timeline_clips()`: full clip metadata (path, timecode, duration, effects)
- `detect_scene_changes()`: auto-blade before sending to ComfyUI
- `blade_at_times([...])`: precise editorial cuts post-generation
- `apply_effect()`, `apply_transition()`: apply ComfyUI results to timeline

**Known limitations (macOS 26.3 + FCP 12.2 build 447037):**
- DualTimeline and Lua VM crash on macOS 26.3, fixed in macOS 26.4+
- See [SpliceKit PR #51](https://github.com/elliotttate/SpliceKit/pull/51) for the guard patch
- All 179 MCP tools that don't touch the crashing swizzles work fine on 26.3

**Earlier assessment (before SpliceKit):**
- [FxPlug](https://developer.apple.com/documentation/fxplug) SDK (Metal only): native plugin path, impractical for ComfyUI
- CommandPost / MotionVFX: caption/editing focused, not generative AI
- FCP 12.2 built-in ML (Magnetic Mask, Smart Conform): Apple models only, no ComfyUI

---

## 3. Adobe Premiere Pro + ComfyUI

### Stability: 0/10 -- Nothing exists, but buildable

- **No plugin exists**.
- Adobe transitioning from CEP (legacy) to [UXP](https://blog.developer.adobe.com/en/publish/2025/12/uxp-arrives-in-premiere-a-new-era-for-plugin-development) (Premiere Pro 25.6, Dec 2025).
- **UXP makes building a bridge feasible**:
  1. JavaScript-based panel calls ComfyUI REST API
  2. Sends selected clip frames to ComfyUI
  3. Receives results, imports to timeline
- Probably the easiest NLE to build a bridge for (after Blender), thanks to JS extensions.
- Adobe's Firefly is cloud-based and limited compared to ComfyUI.

---

## 4. Blender + ComfyUI

### 4a. Pallaidium (tin2tin) -- 5/10

- **URL**: https://github.com/tin2tin/Pallaidium (1.3K stars)
- **What**: Generative AI movie studio in Blender's VSE. Does NOT use ComfyUI -- runs models directly.
- **Capabilities**: T2V, T2I, T2S, T2M, V2V, I2I, I2V, LoRA, cloud (MiniMax)
- **Limitations**:
  - **Windows only** (macOS officially unsupported)
  - Painful installation
  - Rapid churn breaks workflows
  - img2img video disabled ("too flickering")
  - Low community engagement
  - VSE is not a serious NLE

### 4b. ComfyUI-BlenderAI-node (AIGODLIKE) -- 4/10

- **URL**: https://github.com/AIGODLIKE/ComfyUI-BlenderAI-node (1.4K stars)
- **What**: ComfyUI nodes as Blender nodes. ComfyUI as server, controlled from Blender.
- **Capabilities**: AI materials, camera input, animation interpolation, grease pencil masks, VSE export
- **Companion**: [ComfyUI-CUP](https://github.com/AIGODLIKE/ComfyUI-CUP)
- **Problems** (acknowledged by devs):
  - "90% of users fail installation"
  - "Cumbersome user interaction"
  - "Failure to work out of the box"
  - Major update planned Feb 2026
- **Fork**: [a-One-Fan/ComfyUI-BlenderAI-node-editor](https://github.com/a-One-Fan/ComfyUI-BlenderAI-node-editor) -- improved version

---

## 5. After Effects + ComfyUI

### Stability: 1/10

- [AppMana plugin](https://github.com/AppMana/appmana-comfyui-after-effects-plugin) -- Pre-alpha. 7 stars, Rust, requires AE 2023 SDK.
- Practical path: render from ComfyUI to EXR/PNG sequences, import into AE.
- [ActionVFX course](https://www.actionvfx.com/blog/comfyui-for-vfx) teaches this ($197, 15 modules).
- [fxphd course](https://www.fxphd.com/details/713/) -- Generative AI for VFX with ComfyUI & InvokeAI.

---

## 6. ComfyUI as NLE (Timeline Nodes)

### ComfyUI-Montagen -- 3/10
- **URL**: https://github.com/MontagenAI/ComfyUI-Montagen (28 stars, v0.2.4)
- Timeline editor inside ComfyUI. Early stage.

### TimeUI -- 2/10
- **URL**: https://github.com/jimmm-ai/TimeUi-a-ComfyUi-Timeline-Node
- Timeline node system. Very experimental.

Not practical for serious editing.

---

## 7. Real-Time Processing

### ComfyStream (Livepeer)
- **URL**: https://blog.livepeer.org/comfyui-and-real-time-video-ai-processing/
- Applies workflows to live video streams.

### ComfyUI_RealtimeNodes
- **URL**: https://github.com/ryanontheinside/ComfyUI_RealtimeNodes
- Webcam capture, motion detection, real-time multimedia.

Real-time works for simple workflows (style transfer, LTX). Not for heavy processing.

---

## 8. General Approaches

### A. Manual Roundtrip (Works Now)
Export -> Load in ComfyUI -> Process -> Export -> Reimport. Works with any NLE.

### B. API + Custom Script (Semi-automated)
Watch folder -> Queue ComfyUI workflows -> Move results -> Trigger NLE import.

### C. Batch Automation
ComfyUI supports [robust batch processing](https://apatero.com/blog/automate-images-videos-comfyui-workflow-guide-2025):
- Auto Queue (continuous loop)
- CSV-driven batch
- Folder-based batch

---

## 9. Krea.ai Feature Mapping

| Krea Feature | ComfyUI Equivalent |
|---|---|
| Inpainting | comfyui-inpaint-nodes (Acly), Fooocus, LaMa, MAT |
| Outpainting | Pad Image, Wan 2.1 VACE (video outpaint) |
| Style Transfer | IPAdapter + ControlNet |
| Upscaling | ESRGAN, SeedVR2, SUPIR |
| Extend Shots | Wan 2.1, LTX, HunyuanVideo (I2V from last frame) |
| Camera Motion | Wan 2.1 camera control, AnimateDiff MotionLoRA |
| Lip-sync | LatentSync, Wav2Lip nodes |

Krea has an [API (Dec 2025)](https://docs.krea.ai/home) but no NLE plugin.

---

## Summary

| NLE | Integration Score | Path Forward |
|---|---|---|
| **Final Cut Pro** | 7/10 | **SpliceKit MCP** (April 2026), in-process dylib, 200+ tools, export_xml→ComfyUI→import_fcpxml. macOS 26.4+ for full stability. |
| **DaVinci Resolve** | 5/10 | Python API + MCP servers (Gursky, Tooflex). Custom roundtrip feasible, not yet prod-ready. |
| **Blender** | 4/10 | Pallaidium (direct models) + ComfyUI-BlenderAI-node (ComfyUI as server). 90% install fail rate. |
| **Premiere Pro** | 0/10 | UXP plugin buildable (Dec 2025+). JS-based, not yet built. |
| **After Effects** | 1/10 | Pre-alpha plugin only. Use EXR sequence workflow. |
| **Manual batch** | 8/10 | Most practical for multi-NLE. Still recommended for DaVinci/Premiere. |
