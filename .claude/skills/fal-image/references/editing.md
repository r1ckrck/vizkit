# Editing — fal-image

The image-edit flow: take an existing image and modify it. Routes here when the user provides a source image and asks to change something about it.

---

## When to use

- User provides a local image file path or URL AND asks to modify it
- Trigger phrases: "edit this image", "modify this photo", "change the background", "remove the X from this", "restyle this", "swap X for Y", "make this darker / brighter / cleaner", "fix the lighting in this"

If user wants to *generate fresh* something inspired by an existing image (not edit it), that's still text-to-image with the existing image as a reference style note in the prompt — not the edit flow.

---

## Source-image handling

The source image must reach fal as a URL. Two paths:

| Source format | How to handle |
|---|---|
| **Local file path** (e.g., `~/Documents/photo.jpg`, `./hero.png`) | Upload via the fal MCP file-upload tool (runtime-discover the exact tool name). Get back a CDN URL. Pass that URL to the edit model. |
| **Already a URL** (e.g., from clipboard, web page) | Pass directly to the edit model. No upload needed. |

Always discover the upload tool at runtime. Do not hardcode tool names like `mcp__fal__upload_file` — those names may change. Use `ToolSearch` to find the upload tool if unclear.

---

## Edit-prompt construction

The brief-constructor adapts its formula slightly for edit mode:

| Slot | For edit mode |
|---|---|
| **Subject** | The element being modified ("the lamp post in the upper-left", "the background sky", "the sweater the subject is wearing") |
| **Action** | The modification verb ("remove", "replace with", "make darker", "extend", "swap for") |
| **Location/Context** | What to preserve unchanged ("keep all other elements identical", "preserve the figure's pose and lighting") |
| **Composition** | Frame and crop preserved unless the edit explicitly changes it |
| **Style + Lighting** | Match existing image's register — same color grading, same light direction, same film-stock feel. Critical: **do not introduce a new register**. |

### Key edit-prompt rules

- **Edge preservation language**: "preserve sharp edges where the original is sharp", "match grain and noise", "blend seamlessly at the boundary"
- **Color consistency anchors**: "match the existing color temperature and saturation", "preserve the warm amber-magenta highlights from the source"
- **Lighting consistency**: "preserve the original light direction and shadow pattern"
- **Don't reword the whole image** — describe only the change. The edit model will preserve everything you don't mention.
- **Be specific about location**: "the lamp post in the upper-left third" beats "the lamp post"

### Worked example

User: *"edit this photo to remove the lamp post"* + image at `./photo.jpg`

Constructed edit prompt:

> Remove the cast-iron lamp post standing in the upper-left third of the frame. Replace with seamless extension of the surrounding red-brick wall and overcast grey sky behind it. Preserve all other elements identically — the two figures walking, the wet cobblestone street, the soft directional light from camera-left. Match the existing slightly desaturated color grading, late-afternoon overcast light quality, and subtle film grain. Blend the patched area seamlessly with no visible boundary.

---

## Output naming

Edit outputs land in the same `generated/images/<date>/` folder as text-to-image generations.

| Source | Output slug |
|---|---|
| Edit verb is the dominant action | Slug derived from the verb + object: `remove-lamp-post-v01.png`, `change-background-v01.png` |
| User has a clear named subject in the request | `<subject>-edited-v01.png` |
| Source image has a known related slug (e.g., from prior generation) | `<source-slug>-edited-v01.png` |

JSON sidecar `mode: "edit"` and `params.source_image: "<original-path-or-url>"` to record provenance.

---

## Side-by-side comparison

When the user wants to see before/after side-by-side, this is a **`media-processing` task** (not fal-image). The orchestrator should suggest:

> "Want a side-by-side comparison? `media-processing` can produce a montage of the original and the edited version."

`media-processing` is deferred (Phase 3+). Until then, the user composes manually.

---

## Limitations

- fal edit models cannot extend the canvas beyond the original frame (use outpainting models specifically when needed — different routing)
- Edit models work best on photo-realistic content. For illustrated/stylized edits, results vary
- Text changes within an image are unreliable — small text edits may corrupt other text
- Removing a subject from a complex composition can leave artifacts; verify visually before declaring success
