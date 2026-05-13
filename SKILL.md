---
version: 0.3.0
name: higgsfield
description: |
  Full Higgsfield AI skill suite. Covers all four capabilities:
  1. higgsfield-generate — image/video generation, Marketing Studio ads, Virality Predictor
  2. higgsfield-product-photoshoot — brand product photography via GPT Image 2 + prompt enhancer
  3. higgsfield-marketplace-cards — marketplace listing image sets (main, secondary, A+)
  4. higgsfield-soul-id — Soul Character training for face-faithful identity reuse
allowed-tools: Bash
---

# Higgsfield AI — Complete Skill Reference

---

## Step 0 — Bootstrap (all skills)

Before any command:

1. Install CLI if missing:
   ```bash
   curl -fsSL https://raw.githubusercontent.com/higgsfield-ai/cli/main/install.sh | sh
   ```
2. Check auth:
   ```bash
   higgsfield account status
   ```
   If `Session expired` or `Not authenticated` → ask user to run `higgsfield auth login`.
3. Soul training requires a paid plan (Basic+). Check before submitting.

---

## Skill 1 — `higgsfield-generate`

General image/video generation, Marketing Studio ads, and Virality Predictor scoring.

### When to use

- "generate an image / make a video / animate this photo"
- "image-to-video / edit / stylize / remix this image"
- "create an ad / make a UGC video / product demo / unboxing / brand video"
- "import product from URL / create avatar for ad"
- "analyze video virality / score this ad / evaluate the hook"

**NOT for:** Soul Character training (→ Skill 4), product photoshoots (→ Skill 2), marketplace listing cards (→ Skill 3).

---

### Model Selection

#### Image defaults

| Intent | Model |
|---|---|
| High-fidelity general / graphic design / UI / banners / on-image text | **GPT Image 2** (default) |
| Branded ad image with avatar + product | **Marketing Studio Image** |
| Aesthetic UGC / fashion editorial / lifestyle character | **Soul 2.0** |
| Cinematic still frame | **Soul Cinema** |
| Highly characterful persona (text-only) | **Soul Cast** |
| Locations / environments / no-people scenes | **Soul Location** |
| Vector illustrations or face edit + complex scene swap | **Seedream 4.5** |
| Character or cartoon-style work | **Nano Banana 2** → **Nano Banana Pro** for harder briefs |
| Fast / cheap iteration / drafts | **Z Image** |
| Anime / stylized where defaults feel flat | **Flux Kontext Max** or **Grok Imagine** |
| Auto-route by prompt | **Auto** |
| Brand product visual (Pinterest pin, lifestyle, hero, ad pack, try-on) | → use **Skill 2** |

#### Video defaults

| Intent | Model |
|---|---|
| All advertising / commercial / branded ad video | **Marketing Studio** |
| All-purpose serious video (multi-shot, consistent identity, 4–15s) | **Seedance 2.0** (SOTA, default) |
| Single-plane scene, cheaper than Seedance 2.0 | **Kling 3.0** |
| Budget clean single-take | **Seedance 1.5 Pro** |
| Cinema-grade highest fidelity | **Cinema Studio Video 3.0** |
| Cheap with strong physics, no audio | **Minimax Hailuo** |
| Fast batch / volume | **Veo 3.1 Lite** |
| Stylized / audio-synced | **Wan 2.7** |

#### Video analysis

| Intent | Model |
|---|---|
| Hook strength, virality, attention, retention, distraction risk | **Virality Predictor** (`brain_activity`) |

To find the exact `--model` ID: `higgsfield model list --json | jq`

---

### Media Flags

| Flag | Purpose | Models |
|---|---|---|
| `--image <path-or-id>` | Reference image | Most image models, `seedance_2_0`, `veo3`, `marketing_studio_video` |
| `--start-image <path-or-id>` | First frame for image-to-video | `kling3_0`, `kling2_6`, `veo3_1`, `seedance_2_0`, `marketing_studio_video` |
| `--end-image <path-or-id>` | Last frame for transitions | `kling3_0`, `seedance_2_0`, `marketing_studio_video` |
| `--video <path-or-id>` | Reference or analyzed video | `seedance_2_0`, `brain_activity` |
| `--audio <path-or-id>` | Audio reference (lipsync) | `seedance_2_0` only — do NOT use `--generate-audio` |

Flags accept local file paths (auto-uploaded) or UUIDs. Prompt-only models (`z_image`, `soul_cast`, `soul_location`) reject all media flags.

---

### Generation Workflow

```bash
# Image
higgsfield generate create gpt_image_2 \
  --prompt "neon city at dusk" \
  --aspect_ratio 16:9 --resolution 2k --wait

# Character/cartoon
higgsfield generate create nano_banana_2 \
  --prompt "anime character, expressive pose" \
  --image ./ref.png --wait

# Image-to-video
higgsfield generate create seedance_2_0 \
  --prompt "camera dollies in slowly" \
  --start-image ./first.png --duration 12 --wait

# Soul image (after training)
higgsfield generate create text2image_soul_v2 \
  --prompt "portrait at golden hour" \
  --soul-id <soul_ref_id> --quality 2k --wait

# Virality Predictor
higgsfield generate create brain_activity --video ./ad.mp4 --wait
```

Always pass `--wait` to block until done. For long jobs: `--wait-timeout 30m`.

---

### Marketing Studio

Branded video/image ads with avatars + products + optional hooks/settings.

**Models:** `marketing_studio_video`, `marketing_studio_image`

#### Modes (`marketing_studio_video`)

| Mode | Hook/Setting | Best for |
|---|---|---|
| `ugc` | ✅ | Default. Casual organic presenter content |
| `ugc_how_to` | ✅ | Tutorial / explainer |
| `ugc_unboxing` | ✅ | Unboxing reveal |
| `ugc_virtual_try_on` | ✅ | Clothing/accessories try-on, UGC vibe |
| `product_review` | ✅ | Presenter opinion |
| `product_showcase` | ❌ | Clean polished product highlight |
| `tv_spot` | ❌ | Broadcast-style commercial |
| `virtual_try_on` | ❌ | Polished model-driven try-on |
| `wild_card` | ❌ | Experimental, model picks the vibe |

`--hook_id` / `--setting_id` are valid only for modes marked ✅, and only for `marketing_studio_video` (not `marketing_studio_image`).

#### Discovery commands

```bash
higgsfield marketing-studio avatars list --json
higgsfield marketing-studio products list --json
higgsfield marketing-studio hooks list --json
higgsfield marketing-studio settings list --json
higgsfield marketing-studio ad-references list --json
higgsfield marketing-studio brand-kits list --json
higgsfield marketing-studio ad-formats list --json
```

#### Quick ad video workflow

```bash
# 1. Get/import product
higgsfield marketing-studio products fetch --url <product-url> --wait

# 2. Generate (write product_ids and avatars as JSON files)
PRODUCT_IDS_JSON=$(mktemp); AVATARS_JSON=$(mktemp)
printf '["<product_id>"]' > "$PRODUCT_IDS_JSON"
printf '[{"id":"<avatar_id>","type":"preset"}]' > "$AVATARS_JSON"

higgsfield generate create marketing_studio_video \
  --prompt "..." \
  --avatars @"$AVATARS_JSON" \
  --product_ids @"$PRODUCT_IDS_JSON" \
  --mode ugc \
  --duration 15 --resolution 720p --aspect_ratio 9:16 \
  --wait
```

Add `--hook_id` / `--setting_id` when setup items are selected. Add `--generate-audio true` for audio.

#### Click-to-Ad shortcut (URL-driven)

```bash
higgsfield marketing-studio products fetch --url https://shop.example.com/product --wait
higgsfield generate create marketing_studio_video \
  --url https://shop.example.com/product \
  --mode ugc --duration 15 --aspect_ratio 9:16 --wait
```

#### Marketing image

```bash
higgsfield generate create marketing_studio_image \
  --prompt "..." --aspect_ratio 1:1 --resolution 2k --wait
```

---

### Virality Predictor

Analyzes a finished video: hook strength, attention, retention, distraction risk.

```bash
higgsfield generate create brain_activity --video ./creative.mp4 --wait
```

Output format:
```
Overall score: 44/100
Peak hook: 49% at 1s
Sustain: 89%
Strongest region: Visual Cortex
Risk: Default Mode is high (mind-wandering risk).

Open report: https://<app-domain>/apps/virality-predictor?resultJobId=<id>
```

Report raw `.glb`/`.bin` artifacts only if the user asks for raw data.

---

### Prompt Engineering

- **Structure:** Subject + setting + style. "a red fox in snowy forest, golden hour, cinematic"
- **Camera:** lens (35mm, 85mm), angle (low, overhead), motion (dolly in, tracking shot)
- **Lighting:** rim light, neon glow, moody backlight
- **Length:** under ~200 tokens — very long prompts cause distortion
- **Image-to-image:** describe what changes, not the whole input. "transform into anime style" not "man with brown hair made into anime"
- **Image-to-video:** describe motion, not the static frame. "camera slowly pushes in", "smoke rises"
- **No negative prompts:** phrase positively — "tack sharp" not "no blur", "uninhabited landscape" not "no people"
- **Safety:** avoid real public figures, sexual content, trademarks/branded characters → `nsfw` or `ip_detected` terminal status

---

## Skill 2 — `higgsfield-product-photoshoot`

Brand-quality product photography via `higgsfield product-photoshoot create`. Uses GPT Image 2 with a backend prompt enhancer — never call `gpt_image_2` directly for product work.

### When to use

- "product photo / studio shot / lifestyle image / Pinterest pin"
- "hero/banner / carousel / ad creative / Meta ads"
- "virtual try-on / model wearing / person holding product"
- "levitating/floating/splash product / CGI/surreal product"
- "restyle / seasonal variation"

**NOT for:** no-product text-to-image (→ Skill 1), branded avatar video (→ Skill 1 Marketing Studio), marketplace listing cards (→ Skill 3), Soul training (→ Skill 4).

---

### Modes

| Mode | When user wants |
|---|---|
| `product_shot` | Product on neutral/studio/catalog background |
| `lifestyle_scene` | Product in real-world environment |
| `closeup_product_with_person` | Tight crop with hands/partial face |
| `moodboard_pin` | Vertical 2:3 Pinterest-native aesthetic |
| `hero_banner` | Wide-format website/email/campaign header |
| `social_carousel` | 3–10 connected slides for IG/LinkedIn/Facebook |
| `ad_creative_pack` | Coordinated static ad variants for Meta/TikTok/Pinterest |
| `virtual_model_tryout` | Product worn/used by AI-rendered model |
| `conceptual_product` | Surreal/CGI/levitating/splash/sculptural product |
| `restyle` | Transform an existing image's aesthetic/mood/season |

**Tie-breakers:**
- Pinterest pin of product on counter → `moodboard_pin` (platform wins)
- Hero banner with product in use → `hero_banner` (format wins)
- Carousel in different scenes → `social_carousel` (multi-slide wins)
- Closeup of person applying serum → `closeup_product_with_person` (specific genre wins)

---

### Pre-generation interview (≤4 questions)

Skip questions whose answer is obvious from context.

**Type A — uploaded product photo, vague request:**
1. How many? `[1 / 3 / 5]`
2. Style/mood? `[Clean studio / Lifestyle / Conceptual / With a model / Other]`
3. Where used? `[Shopify / Instagram / Pinterest / Paid ads / Website hero]`
4. Brand colors? (skip if obvious)

**Type B — uploaded product photo, named use case:**
1. How many? (if multi-output)
2. Offer/mood/hook?
3. Anything to emphasize?

**Type C — text only, no photo:**
1. Can you upload a product photo?
2. Describe: category, packaging, color, distinctive features
3. Style? / Where used?

**Type D — existing image, restyle:**
1. Aesthetic? `[Clean girl / Cottagecore / Quiet luxury / Dark academia / Y2K / Other]`
2. Seasonal context? `[Christmas / Valentine's / Halloween / Black Friday / None]`
3. Preserve vs change? (only if ambiguous)

**Type E — model wearing product:**
1. Model archetype? (suggest 2–3 based on brand audience)
2. Environment? `[Studio clean / Outdoor natural / Street style / Editorial / Home cozy]`
3. Framing? `[Full body / Three-quarter / Waist up / Closeup on product area]`

---

### Command

```bash
higgsfield product-photoshoot create \
  --mode <mode> \
  --prompt "<short user-intent from interview>" \
  [--image <path-or-upload-id>]... \
  [--count <1-10>] \
  [--aspect_ratio <override>]
```

- `--image` accepts local paths (auto-uploaded) or upload UUIDs. Repeat for multiple references.
- `--count 3` returns 3 distinct variants with varied lighting/angle/palette.
- Resolution is always `2k`.
- Backend picks default aspect ratio per mode; override only if user asks.
- Allowed aspect ratios: `1:1`, `4:5`, `5:4`, `3:4`, `4:3`, `2:3`, `3:2`, `9:16`, `16:9`

**Examples:**

```bash
higgsfield product-photoshoot create \
  --mode lifestyle_scene \
  --prompt "cold-brew bottle on sunlit kitchen counter, IG feed" \
  --image bottle.jpg --count 3

higgsfield product-photoshoot create \
  --mode moodboard_pin \
  --prompt "vertical pin for candle brand, cottagecore mood" \
  --image candle.jpg

higgsfield product-photoshoot create \
  --mode restyle \
  --prompt "Christmas version, quiet-luxury aesthetic" \
  --image existing-shot.jpg
```

**Deliver:** print image URLs as a short bulleted list. No JSON, no IDs, no model names, no enhanced prompt text.

---

## Skill 3 — `higgsfield-marketplace-cards`

Marketplace-compliant product listing visuals via `higgsfield marketplace-cards create`. Backend owns compliance templates and prompt assembly.

### When to use

- "marketplace listing images / product detail cards / secondary product images"
- "product infographics / lifestyle listing shots / A+ style content"
- "marketplace image sets / sales-ready product visuals"

**NOT for:** generic brand product photography without marketplace context (→ Skill 2), video/UGC ads (→ Skill 1), Soul training (→ Skill 4).

---

### Scopes

| Scope | Creates |
|---|---|
| `main` | 1 marketplace main image |
| `product-images` | Main image + 5 secondary images |
| `aplus` | Main image + 7 A+ modules |
| `full-set` | Main + 5 secondary + 7 A+ modules (13 total) |

**Custom asset types** (use `--asset` repeated instead of `--scope`):

`main_image`, `infographic`, `multi_angle`, `detail_shot`, `lifestyle`, `whats_in_box`, `aplus_hero_banner`, `aplus_pain_points`, `aplus_features`, `aplus_ingredients`, `aplus_efficacy`, `aplus_how_to_use`, `aplus_endorsement`

---

### Command

```bash
higgsfield marketplace-cards create \
  --scope <main|product-images|aplus|full-set> \
  --prompt "<short product and listing intent>" \
  [--image <path-or-upload-id>]... \
  [--category "<category>"] \
  [--product_context "<context>"] \
  [--brand_context "<context>"] \
  [--visual_style "<style>"]
```

For existing main image jobs:
```bash
higgsfield marketplace-cards create \
  --main-job <completed_main_job_id> \
  --asset infographic --asset lifestyle
```

**Examples:**

```bash
# Product images
higgsfield marketplace-cards create \
  --scope product-images \
  --prompt "sparkling peach lemonade can for marketplace listing" \
  --image ./can.png --category "beverage"

# Full set
higgsfield marketplace-cards create \
  --scope full-set \
  --prompt "premium skincare serum, clean clinical marketplace visual system" \
  --image ./serum.jpg --brand_context "minimal white and sage palette"

# Custom subset
higgsfield marketplace-cards create \
  --asset main_image --asset infographic --asset lifestyle \
  --prompt "..."
```

**Deliver:**
```
Marketplace cards ready:
- Main image: https://...
- Infographic: https://...
- Lifestyle: https://...
```

No JSON, no job IDs, no internal model names, no enhanced prompt text.

---

## Skill 4 — `higgsfield-soul-id`

Train a face-faithful Soul Character (personalized identity model). One-time training returns a `reference_id` reusable in Skill 1 via `--soul-id`.

### When to use

- "create my Soul / train my face / make my digital twin"
- "build me an avatar / learn my appearance"
- "create a character of me / set up identity for video"
- "I want my face in generated images"

**NOT for:** one-shot face swaps (→ Skill 1 with `--image`), named/non-photo avatars (→ Skill 1 with prompt).

**Requires paid plan (Basic+).**

---

### Photo requirements

- **Quantity:** 5–20 photos (8–12 sweet spot)
- **Content:** clear face, eyes visible, single person, no heavy filters, no sunglasses
- **Variety:** multiple angles (front, 3/4 left/right), different lighting (indoor/outdoor), different expressions, different distances
- **Quality:** sharp, ≥1024×1024, JPEG or PNG
- **Avoid:** group photos, costumes/cosplay, hats covering face, repeated poses

---

### Training workflow

1. **Get name** — one word for reference
2. **Get photos** — 5–20 local paths or upload UUIDs
3. **Pick variant:**
   - `--soul-2` — for image generation (default)
   - `--soul-cinematic` — for cinematic/video work
4. **Submit:**
   ```bash
   higgsfield soul-id create \
     --name "<name>" \
     --soul-2 \
     --image ./photo1.png --image ./photo2.png ...
   ```
5. **Wait (silent, default 30m):**
   ```bash
   higgsfield soul-id wait <id>
   ```
6. **Deliver:** "Soul `<name>` ready. Use in generate with `--soul-id <id>`."

---

### Using a trained Soul

```bash
# Still image
higgsfield generate create text2image_soul_v2 \
  --prompt "portrait at golden hour" \
  --soul-id <ref_id> --quality 2k --wait

# Cinematic still
higgsfield generate create soul_cinematic \
  --prompt "film-grade close-up, dramatic light" \
  --soul-id <ref_id> --quality 2k --wait
```

### Listing existing Souls

```bash
higgsfield soul-id list
higgsfield soul-id get <id>
```

---

## Common Errors & Fixes

| Error | Fix |
|---|---|
| `Session expired` / `Not authenticated` | `higgsfield auth login` |
| `Missing required params: prompt` | Ask user for a prompt |
| `Missing required params: medias` on `brain_activity` | Pass exactly one video via `--video <path-or-id>` |
| `Invalid values: aspect_ratio=... (allowed: ...)` | Pick from the allowed enum |
| `Unknown params: <name>` | Run `higgsfield model get <jst>` and check accepted flags |
| `Job ended with status "failed"` | Content policy likely — rephrase prompt |
| `nsfw` / `ip_detected` | Rephrase; avoid real public figures, sexual content, trademarks |
| `Timeout after 10m` | Add `--wait-timeout 30m` or retry |
| `HTTP 429` | Rate limited — back off |
| `Minimum Basic plan required` (Soul) | User needs to upgrade to Basic+ plan |
| `Training failed` (Soul) | Check photos: need 5+ unique faces, well-lit, no heavy occlusion |
| `Model accepts only --image (no roles)` | Drop role-prefixed flags, use plain `--image` |
| `Model does not accept media inputs` | Drop all media flags (prompt-only model) |

---

## UX Rules (all skills)

1. Be concise. No raw IDs, no JSON dumps, no internal model names in chat output.
2. Detect user's language and respond in it. CLI flags and technical args stay English.
3. Don't batch-ask. Ask one thing at a time, only when genuinely missing.
4. Don't pre-estimate cost unless the user asks (`higgsfield generate cost <jst> ...`).
5. Always pass `--wait` to block until done. Polling is silent.
6. Deliver only URLs + a one-line summary. For Virality Predictor, deliver scores + Open report URL.
7. Never write final image prompts yourself — backend prompt enhancers handle that for product-photoshoot and marketplace-cards.
