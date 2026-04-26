# Image-to-Video — fal-video

The image-to-video (I2V) flow: take an existing still image and animate it. Routes here when the user provides a source image and asks for motion.

---

## When to use

- User provides a local image file path or URL AND asks to animate it
- Trigger phrases: "animate this image", "animate this photo", "bring this to life", "image-to-video", "I2V this", "make this move", "make this still photo into a video", "add motion to this"

If the user wants to *generate fresh* a video inspired by an existing image (not animate it), that's still text-to-video with the existing image as a reference style note in the prompt — not the I2V flow.

---

## Source-image handling

The source image must reach fal as a URL. Two paths:

| Source format | How to handle |
|---|---|
| **Local file path** (e.g., `~/Documents/photo.jpg`, `./hero.png`) | Upload via the fal MCP file-upload tool (runtime-discover the exact tool name via `ToolSearch`). Get back a CDN URL. Pass that URL to the I2V model. |
| **Already a URL** (e.g., from clipboard, web page, prior fal generation) | Pass directly to the I2V model. No upload needed. |

Always discover the upload tool at runtime. Do not hardcode tool names like `mcp__fal__upload_file` — those names may change.

---

## I2V prompt construction

The brief-constructor adapts its formula significantly for I2V mode. The source image **already locks** four of the six slots:

| Slot | I2V handling |
|---|---|
| **Subject** | Source image — don't redescribe |
| **Action** | The new motion verb the user wants ("animate this", "make it spin", "have her turn her head") |
| **Location/Context** | Source image — don't redescribe |
| **Composition** | Source image — don't redescribe (the camera framing comes from the still) |
| **Motion** | The dominant slot in I2V — camera move + pacing + subject motion intensity |
| **Style + Lighting** | Source image — match its existing register; do not introduce new lighting |

In I2V the brief-constructor truly fills only **Motion + Action**. Subject / Location / Composition / Style come from the source.

### Key I2V-prompt rules

- **Edge / continuity language**: "preserve the source frame's exact composition", "match the existing lighting direction and color temperature", "no new elements introduced", "subject identity remains constant"
- **Motion-first**: lead with the motion. "Slow push-in over four seconds with the subject..." rather than describing the subject.
- **Subject-motion intensity is critical**: I2V models tend to over-animate stills. State explicitly: "subject motion subtle — only [specific small motion]" or "subject still — only camera moves" or "subject moderate — clear gestures."
- **Don't reword the whole image** — describe only what should move and how.
- **Be specific about what moves**: "the steam wisp drifts upward and curls toward camera-left" beats "things move."

### Prompt length

- Target **50–120 words** for I2V (shorter than T2V's 100–200)
- The source image carries most of the information; padding the prompt with re-description hurts more than helps

---

## Worked example

User: *"animate this product photo with a slow rotate"* + image at `./headphones.jpg`

Constructed I2V prompt:

> Slow rotate of the source frame over four seconds, gentle drift pacing, subject motion still — only the camera or subject rotates on a vertical axis, while the lighting, surface, and shadow pattern remain identical to the source. Single specular highlight travels subtly across the brushed-aluminum coil as the rotation reveals the rear of the earcup. Preserve the source's exact composition, color temperature, and lighting direction. Match the existing soft directional key from camera-right with the white-card fill and the sharp drop-shadow falling away to the lower-left. No new elements introduced.

Length: ~95 words. Motion is the dominant clause. Preservation language is explicit.

---

## Output naming

I2V outputs land in the same `generated/videos/<date>/` folder as text-to-video generations.

| Source | Output slug |
|---|---|
| Source image has a known related slug (e.g., from a prior fal-image generation, sidecar exists) | `<source-slug>-animated-vNN.mp4` |
| User has a clear named subject in the request | `animate-<subject>-vNN.mp4` |
| Source image is arbitrary (no known slug) | `animate-<short-hash>-vNN.mp4` where `short-hash` is a 6-char hash of the source path |

JSON sidecar `mode: "i2v"` and `params.source_image: "<original-path-or-url>"` to record provenance.

---

## Side-by-side comparison

When the user wants to see the source still alongside the animated clip, this is a **`media-processing` task** (not fal-video). The orchestrator should suggest:

> "Want a side-by-side of the source still and the animated clip? `media-processing` can produce a montage."

---

## Limitations

- fal I2V models cannot extend the canvas beyond the source frame's aspect ratio. Use a different aspect ratio? Regenerate from text-to-video instead.
- Some I2V models work best on still-life / product / landscape sources; portrait I2V varies widely by model.
- Text within the source image often corrupts during animation — small text edits across frames produce flickering artifacts. For text-heavy sources, lock the camera move to "locked-off" and the subject motion to "still" to minimize the risk.
- I2V models tend to over-animate stills by default. Always state subject motion intensity explicitly.
- Some I2V models have a maximum input resolution; check at runtime before passing a 4K source.
