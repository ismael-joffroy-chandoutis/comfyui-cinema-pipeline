# Style Transfer in ComfyUI — Complete Handbook

> Source: ComfyUI (@ComfyUI) — March 2026
> TLDR: Recraft for style reference generation. NanoBanana Pro for restyling existing images. Grok image edit and Seedream 5.0-lite when using a style reference. For niche styles, train a LoRA.

---

## Decision Tree — What to Use

### You have a style reference image and want to generate NEW content in that style
→ **Recraft** (style reference generation, accepts 1-5 reference images)
→ **Grok Image Edit** (strong with photographic styles)
→ **Seedream 5.0-lite** (fast, good style adherence)

### You have an EXISTING image and want to RESTYLE it
→ **NanoBanana Pro** (best for image-to-image style transfer)
→ **Grok Image Edit** + prompt describing target style
→ **Image-edit LoRA on Qwen Image Edit** (if you trained one for your specific style)

### You need a NICHE or PROPRIETARY style (not in base models)
→ **Image-gen LoRA on Flux** (for generating new content in that style)
→ **Image-gen LoRA on Z-Image** (alternative to Flux for style LoRAs)
→ **Image-edit LoRA on Qwen Image Edit** (for transforming images into that style)

---

## Models — Capabilities

### Recraft
- Built specifically for style transfer as a primary feature
- Accepts 1-5 style reference images
- Strong with abstract and artistic styles
- Best for: brand consistency, visual identity at scale
- Try: Recraft Style Transfer on ComfyCloud

### NanoBanana Pro
- Best model for restyling existing images
- Image-to-image focused
- Try: 1 Click Multi Styles on ComfyCloud

### Grok Image Edit
- Strong with photographic and realistic styles
- Works well with a style reference image + prompt
- Try: Compare Image Edit Models on ComfyCloud

### Seedream 5.0-lite
- Fast generation
- Good style reference adherence
- Lighter than Seedream full model

### Qwen Image Edit (for LoRA training)
- Base model for training image-edit LoRAs
- Trained LoRA = transformation workflow: input image + "Change the image into [style]"
- No complex prompting needed once LoRA is trained

### Flux / Z-Image (for LoRA training)
- Base models for training image-gen LoRAs
- Trained LoRA = text-to-image in a specific style via trigger word
- Flux: more established ecosystem
- Z-Image: alternative option

---

## Why Not Just Use Prompts?

- Generic styles (watercolor, cel shading, film noir) exist in base models — prompts work
- Specific/niche/proprietary styles don't exist in training data — prompts fail
- Style reference images give precision prompting can't achieve
- LoRAs give repeatability across many generations

---

## LoRA Types — Summary

| LoRA Type | Base Model | Use Case | Workflow |
|-----------|-----------|---------|---------|
| Image-gen LoRA | Flux or Z-Image | Generate new content in a style | Text prompt + trigger word |
| Image-edit LoRA | Qwen Image Edit | Transform existing images | Input image + instruction prompt |

---

## Use Cases for Cinema

- **Visual identity**: consistent look across all AI-generated assets (posters, stills, concept art)
- **Archival footage stylization**: apply a specific film grain or era aesthetic to modern footage stills
- **Concept art**: apply reference director's visual style to new scene compositions
- **Color grading preview**: mock up looks before DaVinci session
- **Character consistency**: maintain visual style across generated character images

---

## Image-Edit vs. Image-Gen — La distinction fondamentale

Cette distinction conditionne toutes les décisions en aval.

**Image-edit models** acceptent une image en entrée et travaillent avec.
- Transformer l'image dans un nouveau style
- Utiliser l'image comme référence de style pour une nouvelle génération

**Image-gen models** génèrent depuis une toile blanche, avec style contrôlé via prompts et LoRA.
- Si tu as des images existantes à restyliser → image-edit
- Si tu dois créer du nouveau contenu nativement dans un style → image-gen + LoRA

---

## Image-Edit Models — 2 approches

### Approche 1 : Transformation directe
- Donne l'image à restyliser + prompt décrivant le style cible
- Le modèle applique la transformation
- Fonctionne bien pour les styles génériques (anime, watercolor, etc.)
- Limite : chaque modèle interprète "anime" différemment

### Approche 2 : Style reference
- Donne une image de référence + prompt décrivant ce que tu veux générer
- Le modèle crée du nouveau contenu dans ce style
- Plus variable, dépend fortement de l'image de référence
- **NanoBanana Pro, Grok Image Edit et Seedream 5.0-lite** performent bien ici
- Meilleurs résultats : prompts détaillés + plusieurs générations

---

## LoRAs pour Style Transfer

Quand un style est trop niche ou spécifique pour être connu du modèle de base, les LoRAs custom sont la réponse.

### Image-edit LoRAs
- Entraînés sur des paires avant/après (image originale + même image dans le style cible)
- Base : **Qwen Image Edit**
- Exemples de LoRAs open-source : `infl8`, `make wojak`, `3d animation`, `realcomic`
- Utilisation : input image + prompt instruction "Change the image into realcomic style"
- Contrôle précis et output prédictible
- Contrainte : créer les paires avant/après n'est pas intuitif (besoin de sourcer les versions "non-stylisées")

### Image-gen LoRAs
- Entraînés sur des images dans le style cible uniquement (pas besoin de paires)
- Base : **Flux** ou **Z-Image**
- Utilisation : trigger word dans le prompt → le modèle génère nativement dans ce style
- Liberté totale — pas lié à une image source
- Dataset simple : juste une collection d'images dans le style, sans version "originale"

---

## Entraîner son propre LoRA

### Outils disponibles

| Outil | Avantage | Modèles supportés |
|-------|----------|-------------------|
| **fal.ai** (endpoints training) | Chemin le plus rapide, sans config | Z-Image, Flux Klein, Qwen Image Edit |
| **AI Toolkit** (local ou RunPod) | Contrôle total sur le process | Flux, SDXL, Wan |

### Recommandation de modèle de base pour commencer
**Flux Klein (9B paramètres)** — sweet spot pour l'entraînement LoRA :
- Rapide à entraîner
- Apprend bien sur des datasets de taille raisonnable
- Bon point de départ si tu n'as pas encore de préférence

### Réalités du terrain
- Prévoir plusieurs runs d'entraînement avant d'obtenir un résultat satisfaisant
- La qualité du dataset est le facteur n°1 — des images propres et cohérentes > un grand volume
- Tester chaque LoRA sur un ensemble varié de prompts avant de l'utiliser en production

---

## Recraft vs Midjourney

Recraft se positionne directement face à Midjourney sur la plage stylistique et la qualité esthétique, en ajoutant :
- Contrôle explicite de style via images de référence (1-5 refs)
- Intégration directe dans les workflows existants via API
- Cohérence sur des séries d'assets (Midjourney = moins fiable sur la répétabilité)

---

## Pourquoi pas juste du prompting ?

- Styles génériques (watercolor, cel shading, film noir) → dans les modèles de base → prompts OK
- Styles spécifiques/niche/propriétaires → absents des données d'entraînement → prompts insuffisants
- Style reference images → précision impossible avec du texte seul
- LoRAs → répétabilité sur de nombreuses générations

---

## Résumé opérationnel

| Situation | Solution |
|-----------|----------|
| Style générique | Prompt sur modèle de base (SDXL, Flux) |
| Style de référence → nouvelle image | Recraft, Grok Image Edit, Seedream 5.0-lite |
| Image existante → nouveau style | NanoBanana Pro, Grok Image Edit |
| Style niche, générer du contenu | LoRA image-gen sur Flux ou Z-Image |
| Style niche, transformer des images | LoRA image-edit sur Qwen Image Edit |

---

## Links

- Recraft Style Transfer on ComfyCloud: https://comfy.org (search "Recraft Style Transfer")
- 1 Click Multi Styles on ComfyCloud: https://comfy.org (search "Multi Styles")
- Compare Image Edit Models: https://comfy.org (search "Compare Image Edit")
