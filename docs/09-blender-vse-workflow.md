# Blender VSE + Generative AI: Workflow Cinéma

**Pallaidium + tin2tin + ComfyUI bridge pour longs métrages.**

February 2026. Tout ce qui suit est testé ou documenté en production.

---

## Pourquoi Blender VSE pour le cinéma génératif

Blender VSE n'est pas le meilleur montage NLE (pas de gestion de projet robuste, pas de multicam sérieux). Mais c'est le seul environnement open source qui combine timeline vidéo + génération IA + bridge ComfyUI + roundtrip FCPXML dans un seul outil.

**Stack cible :**
1. FCP (assemblage brut, montage narratif)
2. Blender VSE (génération IA via Pallaidium ou ComfyUI bridge)
3. DaVinci Resolve (étalonnage, mixage son, export final)

---

## 1. TIN2TIN : Écosystème VSE (167 repos)

Developer : film director, screenwriter, AI researcher. Le plus prolific contributeur VSE de Blender.

### Repos clés (triés par pertinence cinéma)

#### Génération IA
| Repo | Description | Status |
|------|-------------|--------|
| **[Pallaidium](https://github.com/tin2tin/Pallaidium)** | Studio IA complet dans VSE | Actif, Jan 2026 |
| **[Blender_Screenwriter](https://github.com/tin2tin/Blender_Screenwriter)** | Écriture scénarimage → 3D → VSE auto | Actif |
| **[Sequence_Editing](https://github.com/tin2tin/Sequence_Editing)** | Workspace VSE avec add-ons pré-intégrés | Actif |

#### Roundtrip / Interchange
| Repo | Format | Sens | Status |
|------|--------|------|--------|
| **[fcpxml_import](https://github.com/tin2tin/fcpxml_import)** | FCPXML | FCP → Blender | Stable |
| **[VSE_OTIO_Export](https://github.com/tin2tin/VSE_OTIO_Export)** | OpenTimelineIO | Blender → DaVinci | Stable |
| **[Export_EDL](https://github.com/tin2tin/Export_EDL)** | EDL | Blender → tout NLE | Stable |
| **[VSE_File_Menu](https://github.com/tin2tin/VSE_File_Menu)** | UI UX | Import/export amélioré | Actif |

#### Audio & Sous-titres
| Repo | Description |
|------|-------------|
| **[Subtitle_Editor](https://github.com/tin2tin/Subtitle_Editor)** | Transcription auto Whisper, traduction, batch styling |
| **[Scene_Strip_Tools](https://github.com/tin2tin/scene_strip_tools)** | Ponts 3D View ↔ Sequencer (aperçu 3D en temps réel) |

**Référence complète :** [Awesome_VSE_Addons](https://github.com/tin2tin/Awesome_VSE_Addons), liste curatée par tin2tin lui-même.

---

## 2. Pallaidium : Studio IA dans VSE

### Ce que c'est

L'add-on le plus complet pour la production cinéma IA dans Blender. Génère directement dans la timeline VSE, sans sortir de Blender.

Pallaidium n'utilise **pas** ComfyUI : il tourne les modèles directement via Diffusers/HuggingFace.

### Modèles supportés (Feb 2026)

| Catégorie | Modèles |
|-----------|---------|
| **Vidéo** | LTX-2, Wan 2.1, HunyuanVideo, CogVideoX, FLUX.2 |
| **Image** | Flux.1 Schnell/Dev, SDXL + IP-Adapter, SD 3.5, Kolors |
| **Audio/voix** | Parler TTS, F5-TTS (voice cloning), Chatterbox, MMAudio |
| **Contrôles** | OpenPose, Canny, Reference Attention |

### Compatibilité

| Plateforme | Status | Notes |
|------------|--------|-------|
| Windows | Production | CUDA 12.4, NVIDIA 6GB+ |
| Linux | Limité | Communautaire |
| **macOS** | Expérimental | MPS, via issues GitHub |

**Exigences :** Blender 4.5.3+, CUDA 12.4, 20GB+ espace disque (téléchargements modèles)

### Workflow type

```
VSE Sequencer → Sidebar → Generative AI panel
→ Choisir modèle + prompt
→ Configurer contrôles (depth, pose, canny)
→ Générer → output automatiquement sur nouvelle piste VSE
```

---

## 3. ControlNet pour longs métrages : Workflow photos → vidéo

### Objectif

Partir de 5 à 10 photos de référence et générer des séquences vidéo cohérentes pour un long métrage.

### Pipeline recommandé (ComfyUI + ControlNet)

```
Photos référence
    │
    ▼
Extraction depth maps (ZoeDepth/MiDaS via ComfyUI)
    │
    ▼
ComfyUI Multi-ControlNet
    ├── Depth map (contrôle géométrie, position sujet)
    └── Canny map (silhouettes, détails architecturaux)
    │
    ▼
Modèle vidéo (LTX-2 19B recommandé, ou Wan 2.1)
    │
    ▼
Clips 30-60 frames (1-2 secondes)
    │
    ▼
Assemblage Blender VSE
    │
    ▼
Export ProRes → DaVinci (OTIO) → FCP
```

### Types de contrôle ControlNet

| Contrôle | Usage | Input |
|----------|-------|-------|
| **Depth** | Géométrie caméra, distance sujet, perspective | Depth map 16-bit |
| **Canny** | Silhouettes, contours, typographie | Edge map |
| **Pose (DWPreprocessor)** | Mouvements humains, contraintes squelette | Vidéo ou points 2D |
| **Multi-ControlNet** | Depth + Canny combinés | Plusieurs maps |

### Cohérence temporelle

- **30 frames** : 80-90% de consistance avec Depth + Canny
- **Au-delà** : blending manuel de clips courts plus sûr
- **Stratégie production** : générer en segments de 2 secondes, cross-fade 1 frame

### ControlNet disponibles en Feb 2026

- **XLabs-AI ControlNet** (Canny + Depth v3) : pour Flux.1/Flux.2
- **InstantX Flux Union ControlNet** : multi-mode (canny, depth, blur, pose)
- **LTX-2 ControlNet** (WaveSpeedAI) : depth, canny, pose pour LTX-2 19B
- **Wan 2.1 ControlNet** : inference rapide, bonne pour prototypage

---

## 4. Roundtrip FCP ↔ Blender VSE ↔ DaVinci

### État actuel (Feb 2026)

| Liaison | Status | Notes |
|---------|--------|-------|
| FCP → Blender VSE | Stable | via fcpxml_import (tin2tin) |
| Blender VSE → DaVinci | Stable | via OTIO (VSE_OTIO_Export) |
| Blender VSE → FCP | Partiel | OTIO → FCP non fiable ; EDL possible |
| DaVinci → Blender | Manuel | Export clips individuels, réassemblage |

**Verdict :** Un vrai aller-retour Blender ↔ FCP n'est pas production-ready. La chaîne recommandée est **FCP → Blender VSE → DaVinci**.

### Workflow production recommandé

```
1. Assemblage brut en FCP
   → Marquer les zones "AI generation" avec markers FCP

2. Export FCPXML depuis FCP

3. Import dans Blender VSE (tin2tin/fcpxml_import)
   → Les clips correspondants aux markers sont remplacés
   → par des générations IA (Pallaidium ou ComfyUI)

4. Export séquences finales en ProRes 422 HQ

5. Import ProRes dans DaVinci Resolve
   → Étalonnage couleur
   → Mixage son

6. Export DaVinci → OTIO (si retour Blender nécessaire)
```

### FCP → Blender : détail technique

```python
# tin2tin/fcpxml_import lit le .fcpxml
# et crée les strips VSE avec les timecodes exacts
# Les media paths doivent être valides sur la machine locale
# Nested sequences : import partiel possible
```

### Blender → DaVinci via OTIO

1. Installer [VSE_OTIO_Export](https://github.com/tin2tin/VSE_OTIO_Export)
2. File > Export > OTIO (.otio)
3. Dans DaVinci : Edit page → Media Pool (right-click) → Timelines → Import → OTIO
4. Clips conservent timing, pistes audio, certaines transitions

**Known issues :** OTIO → FCP non fiable ; clips doivent pointer vers media valides ; sequences imbriquées perdent certaines métadonnées.

---

## 5. Blender + ComfyUI (bridge externe)

Pour accéder à tous les modèles ComfyUI depuis Blender (pas seulement ceux de Pallaidium) :

### Méthode 1 : ComfyUI-BlenderAI-node (AIGODLIKE)

- [GitHub](https://github.com/AIGODLIKE/ComfyUI-BlenderAI-node)
- Génère auto une UI Blender qui mirror les nodes ComfyUI
- Envoi vers ComfyUI local ou distant (via Tailscale pour RTX 5090 Windows)
- **Support :** Windows 10/11 pris en charge, Linux partiel, macOS non pris en charge
- **Stability :** 4/10 (2026.2 release améliore installation)

### Méthode 2 : StableGen (sakalond)

- [GitHub](https://github.com/sakalond/StableGen)
- Spécialisé texturing 3D : renders passes Blender → ComfyUI → texture projetée
- Idéal pour scènes 3D avec ControlNet Depth/Canny
- **Stability :** 4/10 (Mac/Linux support partiel)

### Recommandation par contexte

| Contexte | Outil |
|----------|-------|
| Windows + accès tous modèles | ComfyUI-BlenderAI-node |
| Windows + simplicité | Pallaidium (tout-en-un) |
| macOS (avec patience) | ComfyUI via API REST direct |
| Scènes 3D | StableGen |

---

## 6. Modèles vidéo février 2026 : mise à jour

### LTX-2 (Lightricks) : game changer

- **4K** (3840×2160), **50 FPS**, jusqu'à **20 secondes**
- **Audio + Vidéo en un seul pass** : élimine une étape de post-production
- ComfyUI : intégré nativement (aucun node externe requis)
- NVFP8 : -30% taille, 2x plus rapide
- NVFP4 : 3x plus rapide, -60% VRAM (NVIDIA-optimized ComfyUI)
- [Lightricks/ComfyUI-LTXVideo](https://github.com/Lightricks/ComfyUI-LTXVideo)
- [Tutorial officiel](https://docs.comfy.org/tutorials/video/ltx/ltx-2)

### Flux.2 Klein (BFL)

- 4B et 9B params : optimisé hardware consumer
- **Sortie 4MP** (vs 1MP pour Flux.1)
- **Multi-référence** : jusqu'à 10 images simultanées (cohérence de style/sujet)
- **JSON prompt structuré** : contrôle précis de la composition
- ControlNet : XLabs-AI v3 (Canny + Depth), InstantX Union
- Mis à jour dans Pallaidium : 2026-01-23

### Seedance 2.0 (ByteDance) : nouveau

- **2K cinema-grade** output
- **Audio + Vidéo natif** synchronisé
- Durées 5-15 secondes
- Multi-modal input
- Released Feb 2026

### Wan 2.2 (Alibaba)

- **1080p natif** (vs 720p pour 2.1)
- Architecture MoE (Mixture-of-Experts) : premier modèle open-source vidéo avec MoE
- VACE 2.0 : contrôle caméra amélioré
- Disponible : T2V-A14B, I2V-A14B
- Templates "Fun Camera" dans ComfyUI core

---

## 7. RTX 5090 : Configuration ComfyUI

Setup requis pour Pallaidium + ComfyUI sur la machine Windows :

```bash
# CUDA 12.8 minimum obligatoire (12.7 = ne fonctionne PAS)
# Driver NVIDIA 572+ (570 a des bugs connus)
# ComfyUI Portable v0.3.33+

# Vérifier CUDA version
nvcc --version

# Weight Streaming (recommandé pour gros modèles)
# Activer dans ComfyUI Settings > Performance
# Utilise la RAM système comme overflow
# → Permet LTX-2 19B + Flux.2 sur 32GB VRAM

# Undervolting (RTX 5090)
# -100W → +15% performance via réduction thermal throttling
# MSI Afterburner ou EVGA Precision X1
```

**Performance attendue :**
- SDXL 4× 1024px batch : ~15 secondes
- LTX-2 30 frames 1080p : ~45-90 secondes selon quantization
- Wan 2.1 14B 5 secondes : ~3-5 minutes

---

## 8. Issues connues (Feb 2026)

| Outil | Issue | Contournement |
|-------|-------|---------------|
| Pallaidium macOS | MPS compatibility fragile | Suivre les issues GitHub |
| Pallaidium img2img video | Désactivé (trop flickery) | ComfyUI temporalnet à la place |
| ComfyUI-BlenderAI-node | Installation complexe | Lancer setup 2026.2 en tant qu'admin |
| VSE_OTIO_Export → FCP | Import non fiable | Passer par DaVinci comme intermédiaire |
| FCPXML import | Media paths doivent être locaux | Copier les médias avant import |
| LTX-2 haute résolution | VRAM intensive | NVFP4 quantization sur RTX 5090 |

---

## Ressources

- [tin2tin GitHub](https://github.com/tin2tin), tous les repos VSE
- [Awesome_VSE_Addons](https://github.com/tin2tin/Awesome_VSE_Addons), liste curatée
- [Pallaidium](https://github.com/tin2tin/Pallaidium), studio IA VSE
- [ComfyUI LTX-2](https://docs.comfy.org/tutorials/video/ltx/ltx-2), doc officielle
- [XLabs ControlNet](https://github.com/XLabs-AI/x-flux), ControlNet pour Flux
- [LTX-2 ControlNet](https://www.runcomfy.com/comfyui-workflows/ltx-2-controlnet-in-comfyui-depth-controlled-video-workflow), workflow depth-guided
