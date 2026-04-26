---
name: fal-image
description: Generate, edit, or upscale images via fal.ai. Triggers on natural-language requests like "generate an image", "create a logo / banner / hero / poster / icon", "make a visual of X", "design an image", "illustrate X", "edit this image", "modify this photo", "change the background of X", "restyle this picture", "upscale this image". Use for any request to create or modify a still visual asset.
---

# fal-image

Image generation, editing, upscaling, and batch variations via fal.ai. Constructs disciplined prompts using a 5-component formula and per-domain styling, calls the fal MCP, saves outputs project-locally with a JSON sidecar.

Two-layer architecture: this orchestrator handles intent detection, MCQs, fal calls, and file writes. A skill-local Sonnet subagent at `agents/brief-constructor.md` owns prompt construction. References under `references/` are loaded on demand by the subagent.

---

## Pre-flight (every invocation)

1. **CWD safety check.** If `pwd` is `$HOME`, `/`, `/etc`, `/usr`, `/var`, or any other system path: warn the user and ask whether to (a) write to fallback `~/Documents/vizkit-outputs/<date>/...` or (b) continue in the current directory. Do not silently pollute system folders.

2. **fal MCP discovery.** Use `ToolSearch` (or your harness equivalent) to find fal MCP tools available in this session. You need tools for: model listing/recommendation, pricing, image generation, file upload (for edit/upscale), and async job management. **Do not hardcode tool names** — discover them at runtime. If no fal MCP tools are available, instruct the user to register fal MCP in this project's `.claude/settings.json` and stop.

3. **Mode detection.** Parse the user's message to determine mode:
   - **generate** — text-to-image (default)
   - **edit** — user provided source image + modification verb
   - **upscale** — user provided source image + "upscale" / "make bigger" / "enhance"
   - **batch** — user asked for "N variations", "options", "variants"

---

## Pipeline

### 1. Detect domain

Read `references/domains.md`. Match the user's request against detection cues. Pick the domain with highest confidence. If two+ tie above 60%, ask the disambiguation MCQ first (per `references/clarification-mcq.md`). If none match above 60%, ask the disambiguation MCQ with top-3 candidates.

### 2. Resolve preset

This skill is **project-scoped** (lives at `<project-root>/.claude/skills/fal-image/`). At runtime, compute the skill root by resolving `$PWD` (or equivalent absolute CWD) at session start. The skill root is `$PWD/.claude/skills/fal-image/`.

Default preset: `$PWD/.claude/skills/fal-image/presets/<domain>.preset.md` — pass the absolute expanded path (no `~`, no shell variables) to the Read tool.

Custom: if the user wrote `using @./brand.preset.md` or `using ./path/to/preset.md`, resolve the absolute path and use that instead.

### 3. Evaluate formula gaps

Read `references/clarification-mcq.md` for the gap-counting heuristic. For each of the 5 formula slots, decide if it's filled (by message + preset) or a gap. Count gaps.

### 4. Render content MCQs (if any)

If gaps exist, render up to 2 content MCQs in plain markdown chat (NOT via the AskUserQuestion tool). Use the patterns from `references/clarification-mcq.md`. Skip if user said "just generate" / fully-specified brief.

### 5. Render model MCQ (always last, unless skipped)

Discover fal model candidates at runtime via the fal MCP listing/recommendation tool. Pick top 3 (cheapest viable, recommended quality, premium). Show with cost from the pricing tool. Mark recommended. Option (d) is "let me think about it → defaults to recommended."

Skip the model MCQ if user named a model in their request, or said "just generate."

### 6. Spawn brief-constructor subagent

Use the `Task` tool with `subagent_type: "general-purpose"`. Wrapper prompt (substitute the actual absolute path computed from `$PWD`):

> "Read `<absolute-skill-root>/agents/brief-constructor.md` and follow its instructions exactly. Then construct a prompt with these inputs: skill-root=`<absolute-skill-root>`, user request, domain, preset path (absolute), mode, MCQ answers, custom override notes, batch info if applicable. Your entire reply must be only the prompt string — nothing before, nothing after, no markdown fence, no explanation."

Where `<absolute-skill-root>` is `$PWD/.claude/skills/fal-image/` resolved to an absolute path. Pass this in the wrapper prompt so the subagent can read its references.

Capture the reply. Strip leading/trailing whitespace and any accidental code-fence wrapping. If reply starts with `BRIEF_FAILED:`, surface the reason to the user and stop.

### 7. Call fal MCP

Pass the constructed prompt to the appropriate fal MCP generation tool with:
- Model: chosen in step 5 (or recommended default)
- Aspect ratio: from preset's `default aspect ratio` (override if user specified)
- Resolution: from preset or user override; if neither, use a sensible default for the model
- Other params: as needed per model

On transient failure (timeout, 5xx): one silent retry after 2 seconds. On second failure: report error, do not retry.

### 8. Post-call file writes

Read `references/outputs.md` for the full spec (folder layout, slug rules, versioning, JSON sidecar schema, write order, failure handling). The TL;DR:

1. **Compute slug** from user request per the slug derivation rules in `references/outputs.md` (drop articles/filler, lowercase, dash-separated, max 6 words, ASCII, with empty-slug fallback to Subject nouns or `untitled`).
2. **Compute version**: scan `<cwd>/generated/images/<date>/` for existing `<slug>-vNN.*`. Take the max NN, increment +1. If none, use `v01`. Two-digit minimum, expand past `v99` to `v100`+.
3. **mkdir -p** `<cwd>/generated/images/<date>/`.
4. **Save image bytes** to `<slug>-vNN.<ext>` from the fal response.
5. **Write JSON sidecar** at `<slug>-vNN.json` per the schema in `references/outputs.md`.
6. **Verify** both files exist and have non-zero size. **On any partial-write failure** (image saved but sidecar failed, or vice versa): echo the full sidecar JSON to chat as a fallback log so the user has the reproduction info. Do not delete an image that was saved successfully — the user paid for it.

### 9. Report to user

In chat, output:

- **Path** (relative): `generated/images/<date>/<slug>-vNN.<ext>`
- **Prompt used** (the brief-constructor's output, in a code block for clarity)
- **Cost incurred** in USD
- **1–2 refinement suggestions** ("want a moodier version?", "want a different angle?")

---

## Slug & versioning quick reference

Full rules in `references/outputs.md`. Quick reference:

- Empty / under-3-char slug → first 3 noun-like tokens from formula's Subject, falling back to `untitled`
- Strip explicit version tokens (`v01`, `version 1`, `v.2`) from input before slug derivation
- Past `v99`, format as `v100`, `v101` — no truncation, no rollover

---

## Banned-word handling

Banned words (per `references/prompt-engineering.md`) are **advisory in v1**. The brief-constructor prefers prestigious anchors over banned words but does not strictly forbid. Record `params.banned_word_mode: "advisory"` in every sidecar.

This is empirically-tuned advisory because the banned list was originally Gemini-tuned. After enough fal generations, we'll re-validate per fal model and may promote to strict.

---

## Mode-specific notes

### Edit mode

Routes here when user provides a source image + modification verb. Skip the model MCQ if there's only one viable edit model on fal at runtime; otherwise show options. Upload source via fal MCP file-upload tool first. Pass source URL + edit prompt to the edit model. Save output with slug derived from edit verb.

### Upscale mode

Routes here when user provides source + "upscale" intent. Bypass the brief-constructor (no prompt needed). Show model MCQ with scale-factor (2x default, 4x available with cost note). Save output with `<source-slug>-upscaled-vNN` if source has a known slug, else `upscale-<6char-hash>-vNN`.

### Batch mode

Routes here when user asks for N variations. Ask the rotation MCQ if not specified. Lock 4 of 5 formula slots, rotate the 5th. Call brief-constructor N times with each variation's value. Save N outputs as `<slug>-v01` through `<slug>-vNN`. Report total cost = N × per-image.

Cap N at 6 by default; confirm with user before exceeding.

---

## Hygiene

- No emojis in any output (chat or files)
- No meta-commentary in the prompt sent to fal
- The prompt sent to fal is exactly what brief-constructor returned, after whitespace/fence strip — nothing added
