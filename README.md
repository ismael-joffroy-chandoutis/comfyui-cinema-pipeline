# ComfyUI Cinema Pipeline

**AI/generative pipeline architecture for professional cinema production.**

Built by a filmmaker for filmmakers. Not hobbyist image generation.

The hard problem in AI cinema is temporal consistency across a full shot — keeping hair, clothes, micro-details coherent frame by frame. That's what this pipeline solves, by routing Blender 3D geometry data into ComfyUI as ControlNet conditioning.

This repo documents what works, what doesn't, and how to wire it all together for a real production.

**Author:** [Ismaël Joffroy Chandoutis](https://ismaeljoffroychandoutis.com/) — post-documentary filmmaker, César 2022, Cannes (Semaine de la Critique). These workflows are tested on real productions.

**Hardware:** RTX 5090 (32GB local) + Comfy Cloud (96GB RTX 6000 Pro) + Mac for editing.

---

## Films made with this pipeline

| Film | Festival | AI use |
|---|---|---|
| *The Goldberg Variations* (in dev) | Villa Albertine 2026 | Full generative pipeline — Blender scenes → ControlNet → Wan 2.2 |
| *Virus* (in dev) | — | Cybercrime infrastructure visualization, Gaussian Splatting |

*Screenshots and workflow exports from production will be added here as the films progress.*

---

## Why this repo exists

No one has compiled a comprehensive, honest assessment of the ComfyUI ecosystem from a **filmmaker's perspective**. Most resources are for hobbyist image generation. This repo documents:

1. Every relevant ComfyUI workflow for cinema production
2. How to connect Claude Code (LLM) to ComfyUI via MCP
3. How to integrate ComfyUI with professional NLEs (DaVinci Resolve, Premiere, FCP)
4. Simplified frontends for mobile/remote access
5. Hybrid local/cloud GPU orchestration

All links are real. All stability ratings are honest. No hype.

---

## Architecture

```
                    ┌─────────────────┐
                    │   Claude Code   │  ← Brain / orchestrator
                    │   (MCP Server)  │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
     ┌────────▼───────┐ ┌───▼────┐ ┌───────▼───────┐
     │  ComfyUI Local │ │ Comfy  │ │ DaVinci       │
     │  RTX 5090 32GB │ │ Cloud  │ │ Resolve       │
     │  (Windows)     │ │ 96GB   │ │ (MCP + API)   │
     └────────┬───────┘ └───┬────┘ └───────────────┘
              │              │
     ┌────────▼──────────────▼────────┐
     │       Access Interfaces        │
     ├────────────────────────────────┤
     │ SwarmUI/MCWW   (browser)       │
     │ ComfyUI-TG     (Telegram)      │
     │ Comfy Portal   (iOS app)       │
     │ Pocket-Comfy   (PWA mobile)    │
     └────────────────────────────────┘
              │
     ┌────────▼────────┐
     │  Tailscale VPN  │
     │  (mesh network) │
     └─────────────────┘
```

---

## Documentation

| Document | Description |
|---|---|
| [Claude Code + ComfyUI Integration](docs/01-claude-code-integration.md) | MCP servers, API, ComfyScript, LLM-driven workflows |
| [Cinema Workflows Catalog](docs/02-cinema-workflows.md) | 70+ workflows organized by category (video gen, VFX, style transfer, etc.) |
| [Simplified Frontends](docs/03-simplified-frontends.md) | Krea-like interfaces, mobile-first UIs, SwarmUI |
| [Mobile & Remote Access](docs/04-mobile-remote-access.md) | Tailscale setup, iOS apps, Telegram bots, PWA |
| [NLE Integration](docs/05-nle-integration.md) | DaVinci Resolve, Premiere Pro, FCP, Blender, After Effects |
| [Hybrid Local/Cloud](docs/06-hybrid-local-cloud.md) | RTX 5090 + Comfy Cloud + RunComfy + ComfyUI-Distributed |
| [Open Creative Studio (OCS)](docs/07-ocs-perilli.md) | Alessandro Perilli's all-in-one ComfyUI system |
| [Hardware Guide](docs/08-hardware.md) | VRAM requirements, GPU benchmarks, optimization |
| [**Blender VSE + AI Workflow**](docs/09-blender-vse-workflow.md) | **Pallaidium, tin2tin ecosystem, ControlNet stills→video, FCP roundtrip** |
| [**Style Transfer Guide**](docs/10-style-transfer.md) | **Recraft, NanoBanana Pro, Seedream 5.0-lite, LoRA training, batch workflows** |
| [**Cloud API Reference**](docs/11-cloud-api-reference.md) | **ComfyUI Cloud + Local API, MCP servers, hybrid switching, batch scripts** |

---

## Key findings (TL;DR)

### What works now (updated Feb 2026)
- ComfyUI + MCP servers = Claude Code can generate images/video programmatically
- **LTX-2**: 4K, 50fps, 20s, audio+video in one pass — production-grade
- **Wan 2.2**: 1080p native, MoE architecture, camera control
- **Flux.2 Klein**: 4MP output, 10-image multi-reference, ControlNet support
- **Seedance 2.0** (ByteDance): 2K cinema-grade, audio sync, Feb 2026
- SAM 3 rotoscoping replaces hours of manual work
- StyleTransferPlus + EbSynth = temporal-consistent artistic looks
- **Blender VSE + Pallaidium**: stills → video via ControlNet depth workflow
- **FCP → Blender VSE roundtrip** via tin2tin/fcpxml_import (stable)
- **Blender VSE → DaVinci via OTIO** (tin2tin/VSE_OTIO_Export, stable)
- Mobile access via Tailscale + Telegram bot or PWA

### What doesn't work yet
- Blender VSE → FCP roundtrip (OTIO→FCP unreliable; use DaVinci as intermediary)
- ComfyUI-BlenderAI-node on macOS (Windows only, unstable on Mac)
- Comfy Cloud and local ComfyUI don't have transparent switching
- True headless NLE for AI agent control (research phase)

### What we're building
- MCP server configured for cinema workflows
- DaVinci Resolve <-> ComfyUI bridge (Python API)
- Mobile-first frontend for on-set/remote use
- This repo as a living reference

---

## Cinema Workflow Templates

Ready-to-use workflow templates in `workflows/`, auto-discovered by the MCP server:

| Template | Use Case | VRAM | Notes |
|---|---|---|---|
| `sdxl_cinema_image` | Concept art, stills, moodboards | 8GB | Any SDXL checkpoint |
| `wan_t2v` | Text-to-video (Wan 2.1 14B) | 24GB | 480p-720p, 16fps, ~5s clips |
| `wan_i2v` | Animate still into video (Wan 2.1 I2V) | 28GB | First frame from image |
| `img2img_cinema` | Style transfer, variations, film looks | 8GB | Denoise 0.3-0.7 for control |
| `upscale_esrgan` | Upscale to 4K cinema resolution | 2GB | 4x with Real-ESRGAN |
| `frame_interpolation` | Slow-motion, 16fps to 24fps | 4GB | RIFE optical flow |
| `depth_map` | VFX compositing, parallax | 2GB | Depth Anything V2 |

Each template has a `.meta.json` sidecar with defaults, constraints, recommended models, and VRAM estimates.

### Quick Start (MCP Server)

```bash
# 1. Clone this repo
git clone https://github.com/12georgiadis/comfyui-cinema-pipeline.git
cd comfyui-cinema-pipeline

# 2. Run setup (clones MCP server, installs deps, creates .env)
./scripts/setup.sh

# 3. Edit .env with your actual values
# COMFYUI_URL, COMFY_CLOUD_API_KEY, TELEGRAM_*

# 4. Start the MCP server
./scripts/start-mcp-server.sh

# 5. Detect installed nodes
python scripts/detect-nodes.py http://YOUR_COMFYUI_IP:8188
```

---

## Stability ratings

| Tool | Score | Notes |
|---|---|---|
| ComfyUI core | 9/10 | Rock solid |
| **LTX-2** | **9/10** | **4K, 50fps, 20s, audio+video — production-ready** |
| Wan 2.2 workflows | 8/10 | MoE architecture, 1080p native, camera control |
| **Flux.2 Klein** | **8/10** | **4MP, 10-image multi-ref, ControlNet stable** |
| **Seedance 2.0** | **7/10** | **2K cinema-grade, audio sync — Feb 2026** |
| ComfyUI MCP servers | 6/10 | Multiple options, some rough edges |
| DaVinci Resolve MCP | 5/10 | Functional, needs custom dev |
| SwarmUI | 7/10 | Good compromise simplicity/power |
| MCWW (mobile UI) | 7/10 | Auto-adapts to any workflow |
| Comfy Cloud | 7/10 | Works but expensive for heavy use |
| **Blender Pallaidium (Windows)** | **7/10** | **Production-ready on Windows, experimental macOS** |
| FCP → Blender roundtrip | 5/10 | FCPXML import stable (tin2tin), export unreliable |
| FCP + ComfyUI direct | 0/10 | Does not exist |

---

## Screenshots wanted

This repo needs visual documentation. If you're using these workflows in production, PRs with screenshots are the most valuable contribution possible.

What's needed:
- LTX-2 output examples (before/after)
- Wan 2.2 I2V from Blender depth pass
- Temporal consistency comparison (with/without ControlNet depth)
- DaVinci Resolve MCP in action
- Mobile access (Telegram bot, PWA)

---

## Contributing

This is a living document. If you're a filmmaker working with ComfyUI, PRs are welcome. Focus on:
- Real-world production experience (not just "it generates cool images")
- Honest stability assessments
- New NLE integration approaches
- Mobile/remote workflow improvements

---

## Credits

Research compiled by [Ismael Joffroy Chandoutis](https://ismaeljoffroychandoutis.com/) with Claude Code (Anthropic).

February 2025. Updated March 2026.

---

## License

MIT
