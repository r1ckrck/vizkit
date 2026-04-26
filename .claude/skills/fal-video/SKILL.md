---
name: fal-video
description: Generate text-to-video or image-to-video clips via fal.ai. Triggers on requests like "make a video", "generate a clip of X", "create a short video", "animate this image", "animate X", "bring X to life", "image-to-video". Use for video generation. Do NOT use for converting an existing video to a GIF (that's media-processing) or for creating a GIF from scratch (that's gif-maker).
---

# fal-video

Video generation via fal.ai. Constructs disciplined prompts using a 6-component formula (Subject → Action → Location/Context → Composition → Motion → Style+Lighting) and per-domain styling, calls the fal MCP, saves outputs project-locally with a JSON sidecar.

Two-layer architecture: this orchestrator handles intent detection, MCQs, fal calls, and file writes. A skill-local Sonnet subagent at `agents/brief-constructor.md` owns prompt construction. References under `references/` are loaded on demand by the subagent.

---

## Pre-flight (every invocation)

1. **CWD safety check.** If `pwd` is `$HOME`, `/`, `/etc`, `/usr`, `/var`, or any other system path: warn the user and ask whether to (a) write to fallback `~/Documents/vizkit-outputs/<date>/...` or (b) continue in the current directory. Do not silently pollute system folders.

2. **fal MCP discovery.** Use `ToolSearch` (or your harness equivalent) to find fal MCP tools available in this session. You need tools for: model listing/recommendation, pricing, video generation, file upload (for i2v source), and async job management. **Do not hardcode tool names** — discover them at runtime. If no fal MCP tools are available, instruct the user to register fal MCP in this project's `.claude/settings.json` and stop.

3. **Mode detection.** Parse the user's message to determine mode:
   - **generate** — text-to-video (default)
   - **i2v** — user provided source image + motion verb ("animate this", "bring this to life", "make this move")
   - **batch** — user asked for "N variations", "options", "variants"

---

## Pipeline

### 1. Detect domain

Read `references/domains.md`. Match the user's request against detection cues. Pick the domain with highest confidence. If two+ tie above 60%, ask the disambiguation MCQ first (per `references/clarification-mcq.md`). If none match above 60%, ask the disambiguation MCQ with top-3 candidates.

### 2. Resolve preset

This skill is **project-scoped** (lives at `<project-root>/.claude/skills/fal-video/`). At runtime, compute the skill root by resolving `$PWD` (or equivalent absolute CWD) at session start. The skill root is `$PWD/.claude/skills/fal-video/`.

Default preset: `$PWD/.claude/skills/fal-video/presets/<domain>.preset.md` — pass the absolute expanded path (no `~`, no shell variables) to the Read tool.

Custom: if the user wrote `using @./brand.preset.md` or `using ./path/to/preset.md`, resolve the absolute path and use that instead.

### 3. Evaluate formula gaps

Read `references/clarification-mcq.md` for the gap-counting heuristic. For each of the 6 formula slots, decide if it's filled (by message + preset) or a gap. Count gaps.

### 4. Render content MCQs (if any)

If gaps exist, render up to 2 content MCQs in plain markdown chat (NOT via the AskUserQuestion tool). Use the patterns from `references/clarification-mcq.md`. **Motion is the top-priority MCQ for video** — ask it first if Motion is a gap. Skip if user said "just generate" / fully-specified brief.

### 5. Render model MCQ (always last, unless skipped)

Discover fal video model candidates at runtime via the fal MCP listing/recommendation tool. Pick top 3 (cheapest viable, recommended quality, premium). Show with cost from the pricing tool. Mark recommended. Option (d) is "let me think about it → defaults to recommended."

Skip the model MCQ if user named a model in their request, or said "just generate."

### 6. Spawn brief-constructor subagent

Use the `Task` tool with `subagent_type: "general-purpose"`. Wrapper prompt (substitute the actual absolute path computed from `$PWD`):

> "Read `<absolute-skill-root>/agents/brief-constructor.md` and follow its instructions exactly. Then construct a prompt with these inputs: skill-root=`<absolute-skill-root>`, user request, domain, preset path (absolute), mode, MCQ answers, custom override notes, target duration if specified, target fps if specified, source image URL if mode=i2v, batch info if applicable. Your entire reply must be only the prompt string — nothing before, nothing after, no markdown fence, no explanation."

Where `<absolute-skill-root>` is `$PWD/.claude/skills/fal-video/` resolved to an absolute path. Pass this in the wrapper prompt so the subagent can read its references.

Capture the reply. Strip leading/trailing whitespace and any accidental code-fence wrapping. If reply starts with `BRIEF_FAILED:`, surface the reason to the user and stop.

### 7. Call fal MCP

Pass the constructed prompt to the appropriate fal MCP video-generation tool with:
- Model: chosen in step 5 (or recommended default)
- Aspect ratio: from preset's `default aspect ratio` (override if user specified)
- Resolution: from preset or user override; if neither, sensible default for the model
- Duration: from preset's `default duration`, user override, or model default
- fps: from preset's `default fps`, user override, or model default
- Audio: from preset's `audio rules` (default `off`), user override
- Other params: as needed per model

On transient failure (timeout, 5xx): one silent retry after 2 seconds. On second failure: report error, do not retry.

### 8. Post-call file writes

Read `references/outputs.md` for the full spec (folder layout, slug rules, versioning, JSON sidecar schema, write order, failure handling). The TL;DR:

1. **Compute slug** from user request per the slug derivation rules (drop articles/filler, lowercase, dash-separated, max 6 words, ASCII, with empty-slug fallback to Subject nouns or `untitled`).
2. **Compute version**: scan `<cwd>/generated/videos/<date>/` for existing `<slug>-vNN.*`. Take the max NN, increment +1. If none, use `v01`. Two-digit minimum, expand past `v99` to `v100`+.
3. **mkdir -p** `<cwd>/generated/videos/<date>/`.
4. **Save video bytes** to `<slug>-vNN.<ext>` from the fal response (default `.mp4`; use whatever extension fal returned).
5. **Write JSON sidecar** at `<slug>-vNN.json` per the schema in `references/outputs.md`.
6. **Verify** both files exist and have non-zero size. **On any partial-write failure**: echo the full sidecar JSON to chat as a fallback log so the user has the reproduction info. Do not delete a video that was saved successfully — the user paid for it.

### 9. Report to user

In chat, output:

- **Path** (relative): `generated/videos/<date>/<slug>-vNN.<ext>`
- **Prompt used** (the brief-constructor's output, in a code block for clarity)
- **Cost incurred** in USD
- **1–2 refinement suggestions** ("want a different camera move?", "want a longer/shorter duration?", "want it looping?")

---

## Slug & versioning quick reference

Full rules in `references/outputs.md`. Quick reference:

- Empty / under-3-char slug → first 3 noun-like tokens from formula's Subject, falling back to `untitled`
- Strip explicit version tokens (`v01`, `version 1`, `v.2`) from input before slug derivation
- Past `v99`, format as `v100`, `v101` — no truncation, no rollover

---

## Banned-word handling

Banned words (per `references/prompt-engineering.md`) are **advisory in v1**. The brief-constructor prefers concrete cinematographer / publication anchors over banned words but does not strictly forbid. Record `params.banned_word_mode: "advisory"` in every sidecar.

This is empirically-tuned advisory because the banned list was originally Gemini-tuned plus video-specific additions. After enough fal generations, we'll re-validate per fal model and may promote to strict.

---

## Mode-specific notes

### i2v mode

Routes here when user provides a source image + motion verb. Read `references/image-to-video.md`. Upload source via fal MCP file-upload tool first (runtime-discover the tool). Pass source URL + motion-focused prompt to the i2v model. Source image already locks Subject/Location/Composition/Style — brief-constructor focuses on Motion + Action only. Prompt target length 50–120 words. Save output with slug `<source-slug>-animated-vNN.mp4` if source has known slug, else `animate-<verb>-vNN.mp4`.

### Batch mode

Routes here when user asks for N variations. Read `references/batch-variations.md`. Lock 5 of 6 formula slots, rotate the 1 chosen slot. **Default rotation: Motion** (most useful exploration axis for video). Ask the rotation MCQ if not specified. Call brief-constructor N times with each variation's value. Save N outputs as `<slug>-v01` through `<slug>-vNN`. Report total cost = N × per-clip.

**Cap N at 4 by default** (lower than fal-image's 6 because video is more expensive). Confirm cost with user before exceeding.

---

## Audio default

Audio is **off** by default across all 6 presets in v1. Saves cost on capable models, irrelevant for VizKit's primary use case (gif-maker source clips, short hero clips). User can override per-request: "with audio" / "include audio" / "audio on" → set `audio: "on"` in the fal call.

---

## Hygiene

- No emojis in any output (chat or files)
- No meta-commentary in the prompt sent to fal
- The prompt sent to fal is exactly what brief-constructor returned, after whitespace/fence strip — nothing added
