---
name: gif-maker
description: Create animated GIFs from scratch by orchestrating fal-video (source clip) + media-processing (GIF conversion). Triggers on requests like "make a gif of X", "create an animated gif of X", "generate a gif of X" — when no existing video file is provided. If the user has an existing video file to convert, use media-processing instead.
---

# gif-maker

Animated GIF generation by orchestrating fal-video (source clip) + ffmpeg (palette + paletteuse). Pure orchestrator: no own brief-constructor (spawns fal-video's), no own model layer (uses ltx-video locked default). Has a single `default.preset.md` for GIF output defaults.

**Sibling-skill dependency:** gif-maker requires the `fal-video` skill installed at `$PWD/.claude/skills/fal-video/` and ffmpeg present locally. Both are checked in pre-flight.

---

## Pre-flight (every invocation)

1. **CWD safety check.** If `pwd` is `$HOME`, `/`, `/etc`, `/usr`, `/var`, or any other system path: warn the user and ask whether to (a) write to fallback `~/Documents/vizkit-outputs/<date>/...` or (b) continue. Do not silently pollute system folders.

2. **Sibling-skill check.** Verify `$PWD/.claude/skills/fal-video/agents/brief-constructor.md` exists. If absent: "gif-maker depends on the fal-video skill being installed at `<project>/.claude/skills/fal-video/`. Install it (or copy from a project that has it) before continuing."

3. **fal MCP discovery.** Use `ToolSearch` to find fal MCP tools (model listing, pricing, video-generation, file upload, async job management). **Do not hardcode tool names** — discover at runtime. If no fal MCP available: instruct user to register fal MCP in this project's `.claude/settings.json` and stop.

4. **ffmpeg presence check.** Run `ffmpeg -version` (or equivalent). If missing: "gif-maker needs ffmpeg locally. Install via `brew install ffmpeg` (macOS) / `apt install ffmpeg` (Debian) / `choco install ffmpeg` (Windows), then retry."

---

## Pipeline (11 steps)

### 1. Detect intent

Read `references/strategy.md`. Classify the request into one of 5 intents: `loader`, `micro-animation`, `hero`, `decorative`, `photographic`. If the cues don't match any single intent cleanly, ask one disambiguation MCQ (per `references/strategy.md` "When intent is unclear" section). If user provides a video file path, suggest routing to `media-processing` instead.

### 2. Resolve preset

Default: `$PWD/.claude/skills/gif-maker/default.preset.md` — pass the absolute expanded path to the Read tool. If user wrote `using @./brand-gif.preset.md`, resolve and use that instead.

### 3. Determine GIF parameters

From intent + preset + user overrides:
- **fps** — default 10, override range 8–15
- **width** — default 480px, override allowed (320 / 640 / 720 typical alternatives)
- **max_colors** — 16–32 for loader/micro-animation, 64 for hero, 32–64 for decorative, 128 for photographic
- **dither** — `bayer` for flat content, `floyd_steinberg` for photographic
- **duration** — default model minimum (~5s); trim with ffmpeg `-t` if user wants shorter
- **aspect ratio** — per user intent (square for buttons / 16:9 for hero / vertical for stories)

### 4. Read source-clip-spec

Read `references/source-clip-spec.md`. Build the override-notes block from the template, substituting per-intent values (color budget phrasing, loop phrasing, duration, aspect ratio).

### 5. Spawn fal-video's brief-constructor as subagent

Use the `Task` tool with `subagent_type: "general-purpose"`. Wrapper prompt (substitute the actual absolute path computed from `$PWD`):

> "Read `$PWD/.claude/skills/fal-video/agents/brief-constructor.md` and follow its instructions exactly. Then construct a prompt with these inputs: skill-root=`$PWD/.claude/skills/fal-video`, user request=<verbatim>, detected domain=<per intent mapping in strategy.md>, preset path=`$PWD/.claude/skills/fal-video/presets/<domain>.preset.md`, mode=generate, MCQ answers=none, custom override notes=<the GIF-aware block built in step 4>, target duration=<chosen source duration>, target fps=accept native — fps not controllable on most fal models. Your entire reply must be only the prompt string — nothing before, nothing after, no markdown fence, no explanation."

Capture the reply. Strip leading/trailing whitespace and any accidental code-fence wrapping. If reply starts with `BRIEF_FAILED:`, surface the reason and stop.

### 6. Call fal MCP for source-clip generation

- Default model: `fal-ai/ltx-video` ($0.02 flat). User override allowed (e.g., kling, runway).
- Resolution: smallest tier the model exposes
- Audio: off
- Duration: model minimum (or user override if model supports it)
- Aspect ratio: per intent
- Prompt: brief-constructor's output

On transient failure (timeout, 5xx): one silent retry after 2 seconds. On second failure: report error, do not retry.

### 7. Save source MP4

Compute slug from user request (per the slug derivation rules in fal-video's `references/outputs.md` — same conventions). Compute version by scanning `<cwd>/generated/gifs/<date>/` for existing `<slug>-vNN.gif`. Take max NN, increment +1. If none, use `v01`.

Save the MP4 returned by fal to `<cwd>/generated/gifs/<date>/_source/<slug>-vNN-source.mp4`. mkdir `_source/` if missing. Underscore prefix sorts to bottom in date folder; the source MP4 is kept so the user can re-encode without re-paying fal.

### 8. Run ffmpeg two-pass palette pipeline

Per `references/ffmpeg-commands.md`. Pass 1 generates palette PNG. Pass 2 applies palette + dither to produce GIF. Use the trim variant if user requested a shorter GIF than the source. Discard the intermediate palette PNG after pass 2.

### 9. Save GIF

Final GIF lands at `<cwd>/generated/gifs/<date>/<slug>-vNN.gif`.

### 10. Write JSON sidecar

At `<slug>-vNN.json` with these fields:
- `request`, `slug`, `version`, `type: "gif"`, `skill: "gif-maker"`
- `prompt` (brief-constructor's output)
- `params.fal_model`, `params.source_duration_seconds`, `params.gif_fps`, `params.gif_width`, `params.gif_max_colors`, `params.gif_dither`, `params.intent`, `params.source_path` (relative to project root, points into `_source/`)
- `cost_usd`, `timestamp` (ISO 8601 with offset), `duration_ms`, `output_path` (relative)

### 11. Verify and report

Verify all three files exist with non-zero size: GIF + sidecar + source MP4. On any partial-write failure: echo full sidecar JSON to chat as fallback log; never delete the source MP4 (user paid for it).

In chat, output:
- **Path** (relative): `generated/gifs/<date>/<slug>-vNN.gif`
- **Source MP4 path** (kept): `generated/gifs/<date>/_source/<slug>-vNN-source.mp4`
- **Cost incurred** in USD
- **1–2 refinement suggestions** ("want it shorter / wider / fewer colors / different dither?") tuned to intent

---

## Cost transparency

| Component | Cost |
|---|---|
| fal-video source clip (default `fal-ai/ltx-video`) | **$0.02 flat** |
| ffmpeg palette + paletteuse (local) | **$0** |
| **Total per GIF (default model)** | **$0.02** |

Higher-quality fal models cost more — kling v2.1 standard is $0.25 for 5s. User opts in explicitly; cost is reported in chat after each generation.

---

## Slug & versioning quick reference

Same rules as fal-image / fal-video — per-slug-per-date, two-digit minimum, expand past v99. See fal-video's `references/outputs.md` for the full spec.

---

## Hygiene

- No emojis in any output (chat or files)
- No meta-commentary in the prompt sent to fal
- Prompt sent to fal is exactly what brief-constructor returned, after whitespace/fence strip — nothing added
- No motion-speed manipulation (no `setpts`, no playback-speed dialing)
- Color budget is intent-driven aesthetic guidance, never a literal "use N colors" prompt instruction
