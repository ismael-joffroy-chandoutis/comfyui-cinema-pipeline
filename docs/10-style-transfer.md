# Style Transfer for Cinema Production

Complete guide to style transfer techniques in ComfyUI, organized by use case.

Source: [The Complete Style Transfer Handbook](https://blog.comfy.org/p/the-complete-style-transfer-handbook) (March 2026) + production research.

---

## Decision Matrix

| Need | Model Type | Tool | When to use |
|------|-----------|------|-------------|
| New images from style reference | Image-gen | **Recraft** | Concept art, moodboards, consistent visual language |
| Restyle existing footage frames | Image-edit | **NanoBanana Pro** | Archive footage transformation, VHS look, artistic restyle |
| Restyle with style reference image | Image-edit | **Grok Image Edit**, **Seedream 5.0-lite** | Match a specific visual language on existing images |
| Generate in niche/proprietary style | Image-gen + LoRA | **Flux Klein** or **Z-Image** | Film-specific visual identity (Goldberg VHS look, etc.) |
| Transform images in niche style | Image-edit + LoRA | **Qwen Image Edit** | Before/after style transformation at scale |

---

## Out-of-the-Box Models

### Recraft (style reference generation)

The standout for cinema production. Accepts 1-5 style reference images and generates new content that faithfully follows the visual language.

**Strengths**: Abstract/artistic styles, consistent outputs, wide range.
**Best for**: Concept art, storyboards, moodboards with consistent visual identity.

- ComfyCloud workflow: https://links.comfy.org/3Pknxy7
- Multi-style comparison: https://links.comfy.org/4smLbbW

### NanoBanana Pro (image restyling)

Best for maintaining subject identity while transforming style. Based on Gemini architecture.

**Strengths**: Identity preservation, prompt-based style control.
**Best for**: Restyling existing footage frames, documentary archive transformation.
**Note**: Already available via `nanobanana` skill in Claude Code.

### Grok Image Edit

Good with detailed prompting. Accepts style reference image + text description.

**Strengths**: Detailed prompt following, style reference matching.
**Best for**: Precise style matching with reference image.

### Seedream 5.0-lite

Strong with multiple generations and detailed prompts. Style reference support.

**Strengths**: Consistency across multiple generations.
**Best for**: Batch processing with consistent style.

### Qwen Image Edit

Solid image-edit model. Foundation for community LoRA training.

**Strengths**: Clean transformations, good LoRA base.
**Best for**: Base model for training custom style LoRAs.

- Edit models comparison workflow: https://links.comfy.org/4bPPS89

---

## Custom LoRA Training

When the style you need is too specific for any base model.

### Image-Gen LoRAs (simpler to create)

Dataset: Just a collection of images in your target style. No pairs needed.

| Base Model | Parameters | Training Time | Best For |
|-----------|-----------|---------------|----------|
| **Flux Klein** | 9B | Fast | Sweet spot speed/quality. Default choice. |
| **Z-Image** | - | Medium | Good from-scratch generation |
| **Flux** | Full | Slower | Highest quality |

**Training tools**:
- **Fal.ai** (fastest, API-based): endpoints for Z-Image, Flux Klein, Qwen Image Edit. `FAL_KEY` already configured.
- **AI Toolkit** (more control): run on RunPod or locally on RTX 5090.

### Image-Edit LoRAs (paired data required)

Dataset: Before/after pairs showing original → styled transformation.

**Process**:
1. Collect 20-50 images in target style
2. Create/source "unstyled" counterparts
3. Train on Qwen Image Edit or Flux Klein
4. Apply with trigger word in instruction prompt

**Community examples**: `infl8`, `make wojak`, `3d animation`, `realcomic`

---

## Cinema Production Workflows

### Workflow 1: Consistent Visual Language

For maintaining a coherent look across an entire film/project.

```
1. Select 3-5 key frames that define your visual identity
2. Use Recraft with these as style references
3. Generate all concept art, storyboards, moodboards in this style
4. For production frames: train a LoRA on Flux Klein with 30+ examples
5. All generated content inherits the same visual DNA
```

### Workflow 2: Archive Footage Transformation

For documentary: transform archival material into a unified aesthetic.

```
1. Extract key frames from archive footage (ffmpeg)
2. Use NanoBanana Pro to restyle each frame
3. Prompt: "Transform to [style] while preserving subject identity"
4. For temporal consistency: process sequential frames, use EbSynth for interpolation
5. Batch process via ComfyUI API (POST /api/prompt per frame)
```

### Workflow 3: VHS/Analog Look Training

Specific to projects working with analog source material.

```
1. Collect 30-50 frames with the exact VHS/analog look you want
2. Train image-gen LoRA on Flux Klein via fal.ai
3. Use trigger word to generate new content in that analog aesthetic
4. OR train image-edit LoRA on Qwen Image Edit with clean/VHS pairs
5. Apply LoRA to restyle clean digital footage into analog look
```

### Workflow 4: Batch Style Transfer via API

For processing thousands of frames programmatically.

```python
import requests
import json

COMFY_URL = "http://127.0.0.1:8188"  # or cloud.comfy.org with API key

def style_transfer_frame(input_path, style_prompt, workflow_json):
    """Submit a single frame for style transfer."""
    workflow = json.loads(workflow_json)
    # Set input image and style prompt in workflow
    workflow["6"]["inputs"]["text"] = style_prompt
    workflow["3"]["inputs"]["image"] = input_path

    response = requests.post(f"{COMFY_URL}/prompt", json={"prompt": workflow})
    return response.json()["prompt_id"]

# Process all frames in a directory
import glob
frames = sorted(glob.glob("frames/*.png"))
for frame in frames:
    pid = style_transfer_frame(frame, "cinematic film noir style", workflow_api)
    print(f"Submitted {frame} → {pid}")
```

---

## Integration with Claude Code

### Via artokun/comfyui-mcp (recommended)

```
/comfy:gen "style reference: [uploaded image], generate a dark cinematic landscape"
/comfy:batch frames/ style_transfer_workflow.json
```

### Via direct API (Bash tool)

```bash
curl -X POST http://127.0.0.1:8188/prompt \
  -H "Content-Type: application/json" \
  -d @workflow_api.json
```

### Via fal.ai (LoRA training)

```bash
# Train style LoRA
curl -X POST https://queue.fal.run/fal-ai/flux-lora-fast-training \
  -H "Authorization: Key $FAL_KEY" \
  -H "Content-Type: application/json" \
  -d '{"images_data_url": "https://...", "trigger_word": "goldbergvhs"}'
```

---

## Temporal Consistency

The biggest challenge for cinema: maintaining style consistency across frames.

| Technique | Tool | Quality | Speed |
|-----------|------|---------|-------|
| EbSynth propagation | StyleTransferPlus + EbSynth | High | Slow |
| Sequential frame processing | Any image-edit model | Medium | Medium |
| Video-native models | LTX-2, Wan 2.2, Seedance 2.0 | Highest | Varies |
| ControlNet depth consistency | Depth Anything V2 + ControlNet | High | Medium |

**Best practice**: Process key frames with style transfer, then use EbSynth to propagate style to intermediate frames. This gives the best quality/speed tradeoff for cinema.
