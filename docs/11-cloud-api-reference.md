# ComfyUI Cloud API Reference

Complete API reference for ComfyUI Cloud, local ComfyUI, and MCP integration.

---

## ComfyUI Cloud API

**Base URL**: `https://cloud.comfy.org`
**Auth**: `X-API-Key` header (generate at https://platform.comfy.org/login)
**Docs**: https://docs.comfy.org/development/cloud/overview

### Submit a workflow

```bash
curl -X POST https://cloud.comfy.org/api/prompt \
  -H "Content-Type: application/json" \
  -H "X-API-Key: YOUR_API_KEY" \
  -d @workflow_api.json
```

Response:
```json
{ "prompt_id": "abc123-def456" }
```

### Monitor progress

**Polling**:
```bash
curl https://cloud.comfy.org/api/job/abc123-def456/status \
  -H "X-API-Key: YOUR_API_KEY"
```

Statuses: `pending` → `in_progress` → `completed` | `failed` | `cancelled`

**WebSocket** (real-time):
```
wss://cloud.comfy.org/ws?clientId=YOUR_UUID&token=YOUR_API_KEY
```

### Retrieve outputs

```bash
# List output files
curl https://cloud.comfy.org/api/history_v2/abc123-def456 \
  -H "X-API-Key: YOUR_API_KEY"

# Download file (302 redirect to signed URL)
curl -L https://cloud.comfy.org/api/view?filename=output.png&subfolder=&type=output \
  -H "X-API-Key: YOUR_API_KEY" \
  -o output.png
```

---

## Local ComfyUI API

**Base URL**: `http://127.0.0.1:8188` (default, or Tailscale IP for remote)
**Auth**: None (secure via Tailscale network)

### Same endpoints, same format

```bash
# Submit
curl -X POST http://127.0.0.1:8188/prompt \
  -H "Content-Type: application/json" \
  -d @workflow_api.json

# Monitor (WebSocket)
# ws://127.0.0.1:8188/ws

# History
curl http://127.0.0.1:8188/history/abc123-def456

# View output
curl http://127.0.0.1:8188/view?filename=output.png&subfolder=&type=output -o output.png
```

### Additional local endpoints

```bash
# List installed models
curl http://127.0.0.1:8188/object_info

# System stats (VRAM, queue)
curl http://127.0.0.1:8188/system_stats

# Queue info
curl http://127.0.0.1:8188/queue

# Interrupt current generation
curl -X POST http://127.0.0.1:8188/interrupt

# Clear queue
curl -X POST http://127.0.0.1:8188/free
```

### Get API-format JSON

In ComfyUI UI:
1. Enable Dev Mode in Settings
2. Click "Save (API Format)"
3. This saves the workflow as API-compatible JSON (different from the normal .json)

---

## CLI: comfy-cli

```bash
# Install
pip install comfy-cli

# Run a workflow
comfy run --workflow workflow_api.json

# Launch ComfyUI
comfy launch

# Install custom nodes
comfy node install ComfyUI-VideoHelperSuite
```

---

## MCP Servers for Claude Code

### artokun/comfyui-mcp (RECOMMENDED)

31 tools, 10 slash commands, 3 agents. Most complete.

```json
{
  "comfyui": {
    "command": "npx",
    "args": ["-y", "comfyui-mcp"],
    "env": {
      "CIVITAI_API_TOKEN": "",
      "COMFYUI_URL": "http://127.0.0.1:8188"
    }
  }
}
```

**Key tools**: execute workflow, visualize (Mermaid), batch, model management, VRAM control
**Slash commands**: `/comfy:gen`, `/comfy:viz`, `/comfy:debug`, `/comfy:batch`, `/comfy:gallery`

### shawnrushefsky/comfyui-mcp

70+ workflow templates. Docker-based. Agent memory across sessions.

```bash
docker run -d ghcr.io/shawnrushefsky/comfyui-mcp:latest
```

### Peleke/comfyui-mcp (production/distributed)

Fly.io + RunPod + Supabase + Tailscale. For production deployments.

### claude-code-comfyui-nodes (reverse)

ComfyUI nodes that embed Claude Code agents inside ComfyUI workflows. The opposite direction: ComfyUI calls Claude, not Claude calls ComfyUI.

GitHub: https://github.com/christian-byrne/claude-code-comfyui-nodes

---

## Hybrid Switching (Local ↔ Cloud)

Since APIs are compatible, switch by changing the URL:

```python
import os

COMFY_URL = os.getenv("COMFYUI_URL", "http://127.0.0.1:8188")
COMFY_CLOUD_URL = "https://cloud.comfy.org"
COMFY_API_KEY = os.getenv("COMFY_CLOUD_API_KEY", "")

def submit_workflow(workflow_json, force_cloud=False):
    """Submit to local or cloud based on VRAM needs."""
    url = COMFY_CLOUD_URL if force_cloud else COMFY_URL
    headers = {"Content-Type": "application/json"}
    if force_cloud:
        headers["X-API-Key"] = COMFY_API_KEY

    import requests
    return requests.post(f"{url}/api/prompt", json={"prompt": workflow_json}, headers=headers)
```

**Auto-routing logic**:
- VRAM < 32GB → local RTX 5090
- VRAM > 32GB (VACE 14B, HunyuanVideo) → Comfy Cloud
- Batch > 50 frames → Comfy Cloud (parallel execution on Pro plan)
- On the go (no local access) → Comfy Cloud

---

## Batch Processing Script

```python
#!/usr/bin/env python3
"""Batch process frames through ComfyUI (local or cloud)."""

import requests
import json
import time
import glob
import sys

COMFY_URL = sys.argv[1] if len(sys.argv) > 1 else "http://127.0.0.1:8188"
WORKFLOW = json.load(open(sys.argv[2])) if len(sys.argv) > 2 else {}
FRAMES_DIR = sys.argv[3] if len(sys.argv) > 3 else "frames/"

def submit(frame_path):
    workflow = WORKFLOW.copy()
    # Adapt: set input image path in the workflow
    resp = requests.post(f"{COMFY_URL}/prompt", json={"prompt": workflow})
    return resp.json().get("prompt_id")

def poll(prompt_id):
    while True:
        resp = requests.get(f"{COMFY_URL}/history/{prompt_id}")
        if prompt_id in resp.json():
            return resp.json()[prompt_id]
        time.sleep(2)

frames = sorted(glob.glob(f"{FRAMES_DIR}/*.png"))
print(f"Processing {len(frames)} frames via {COMFY_URL}")

for i, frame in enumerate(frames):
    pid = submit(frame)
    result = poll(pid)
    print(f"[{i+1}/{len(frames)}] {frame} → done")
```

---

## Pricing Quick Reference (March 2026)

| Plan | Monthly | GPU/month | Workflow limit | LoRAs |
|------|---------|-----------|---------------|-------|
| Standard | $20 | ~4.4h | 30 min | No |
| Creator | $35 | ~7.7h | 30 min | CivitAI/HF |
| Pro | $100 | ~22h | 1 hour | CivitAI/HF |

Credits: 211 = $1. GPU rate: 0.266 credits/sec. Top-ups roll over 1 year.
