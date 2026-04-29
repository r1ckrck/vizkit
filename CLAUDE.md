# VizKit — Skill Development Workspace

This project is the **development workspace** for VizKit, a set of Claude Code skills for AI media generation. **This file is for working ON the skills (developing, refactoring, extending) — not for using them.** End-user routing happens via each skill's own `SKILL.md` frontmatter.

The skills are project-scoped (`<project>/.claude/skills/<name>/`) and copied (folder-and-all) into other projects when used to generate actual images / videos / GIFs.

---

## Skills built

| Skill | Files | What it does | Status |
|---|---:|---|---|
| **fal-image** | 22 | Text-to-image · edit · upscale · batch via fal.ai | Built · model roster baked in (Nano Banana Pro / Recraft V4 Pro / Ideogram V3 / Flux 2 / Flux Kontext) · pre-flight summary · reference-image roles |
| **fal-video** | 19 | Text-to-video · image-to-video · batch via fal.ai | Built · model roster baked in (Veo 3.1 / Kling 2.5 Turbo Pro) · pre-flight summary · audio default-off · reference-image roles |
| **gif-maker** | 8 | Orchestrator: spawns fal-video brief-constructor + runs ffmpeg palette/paletteuse locally | Built · defaults to Kling 2.5 Turbo Pro · 5-intent strategy · pre-flight summary |
| **media-processing** | 13 | All ffmpeg + ImageMagick capability as on-demand bash recipes | Built · lean reference-only · script-free |

---

## Folder architecture

```
vizkit/
├── .claude/
│   ├── settings.json                   ← user manages fal MCP registration here
│   └── skills/
│       ├── fal-image/
│       │   ├── SKILL.md                ← orchestrator (148 lines)
│       │   ├── agents/brief-constructor.md
│       │   ├── scripts/orchestrator.py ← 5 helpers (slug, version, sidecar, strip_preamble, download)
│       │   ├── presets/                ← 9 per-domain (model-agnostic)
│       │   │   ├── cinema · product · portrait · editorial
│       │   │   ├── ui · logo · landscape · abstract · infographic
│       │   ├── preset-template.md
│       │   └── references/             ← 8 files
│       │       ├── prompt-engineering.md (5-component formula + per-model notes)
│       │       ├── domains.md (9 domains + modifier libraries)
│       │       ├── clarification-mcq.md · editing.md
│       │       ├── upscaling.md · batch-variations.md
│       │       ├── outputs.md · fal-mcp-flow.md
│       │
│       ├── fal-video/
│       │   ├── SKILL.md                ← orchestrator (147 lines)
│       │   ├── agents/brief-constructor.md
│       │   ├── scripts/orchestrator.py ← same 5 helpers
│       │   ├── presets/                ← 6 per-domain (model-agnostic; no UI/Logo/Infographic)
│       │   │   └── cinema · product · portrait · editorial · landscape · abstract
│       │   ├── preset-template.md
│       │   └── references/             ← 8 files
│       │       ├── prompt-engineering.md (6-component formula + per-model notes)
│       │       ├── motion-vocabulary.md (camera moves · pacing · intensity)
│       │       ├── domains.md (6 domains + modifier libraries)
│       │       ├── clarification-mcq.md · image-to-video.md (reference-image roles)
│       │       ├── batch-variations.md · outputs.md · fal-mcp-flow.md
│       │
│       ├── gif-maker/
│       │   ├── SKILL.md                ← orchestrator (119 lines)
│       │   ├── default.preset.md       ← single preset (not per-domain)
│       │   ├── scripts/orchestrator.py ← same 5 helpers
│       │   └── references/             ← 4 files
│       │       ├── strategy.md (5 intent buckets + fal-video domain map + source-model picks)
│       │       ├── source-clip-spec.md (override-notes pattern)
│       │       ├── ffmpeg-commands.md (two-pass palettegen + paletteuse)
│       │       └── fal-mcp-flow.md
│       │
│       └── media-processing/
│           ├── SKILL.md                ← reference catalog (549 lines, intentional)
│           └── references/             ← 12 files
│               ├── ffmpeg-encoding · filters · color · subtitles
│               ├── ffmpeg-composition · audio-effects · metadata · analysis
│               ├── imagemagick-editing · batch · modern-formats
│               └── format-compatibility
│
├── .git/
└── CLAUDE.md                           ← you are here
```

---

## Architectural pattern (used by fal-image and fal-video)

Two-layer thin-orchestrator pattern:

| Layer | Lives in | Job |
|---|---|---|
| **Orchestrator** | `SKILL.md` | Detect domain · run content MCQs · pick model from roster · spawn subagent · pre-flight summary · call fal MCP · save outputs · report cost |
| **Brief constructor** | `agents/brief-constructor.md` (Sonnet via Task tool) | Read formula + domain + preset → return one optimized prompt string, opening adapted to `target_model` |
| **Per-domain presets** | `presets/<domain>.preset.md` | Locked styling defaults — model-agnostic, one per domain |
| **References** | `references/*.md` | Loaded on demand by the subagent |
| **Helpers** | `scripts/orchestrator.py` | 5 stdlib helpers — slug, version, sidecar, strip_preamble, download |

**Model selection.** Each skill carries an advisory roster in `SKILL.md`. The orchestrator auto-picks based on intent + reference-image presence and announces the choice in the pre-flight summary. The user can override at any point. Roster is advisory, not enforced — `recommend_model` from fal MCP is the runtime fallback when a named endpoint is unavailable.

**Pre-flight summary.** Before any generation, the orchestrator surfaces detected domain + chosen model + the constructed prompt + cost estimate. User confirms or redirects.

**Reference images.** Optional input. The orchestrator infers role (style / character / composition / edit-target / animate-target) from the user's phrasing and passes the role to the brief-constructor, which phrases the reference into the prompt. The reference URLs go to the model as separate API parameters.

**gif-maker** is a pure orchestrator — no own brief-constructor. It spawns fal-video's brief-constructor with GIF-aware constraints injected as `custom override notes`. Source-clip default is Kling 2.5 Turbo Pro; user can opt into Veo 3.1 for premium.

**media-processing** is a reference catalog (no orchestrator pattern, no subagent). Claude composes ffmpeg/ImageMagick bash from the references on demand.

---

## Conventions (apply to every skill)

| Convention | Rule |
|---|---|
| **Self-containment** | Each non-orchestrator skill folder is portable. `grep -rn "/Users/r1ckrck" .claude/skills/<skill>/` returns 0 hits. fal-image and fal-video make zero cross-skill references. gif-maker is the one allowed exception — it references fal-video by skill name (never by file path). |
| **Skill-root resolution** | Use `$PWD/.claude/skills/<skill>/`. No `~`. No env vars in paths. Pass absolute path to subagents at runtime. |
| **Frontmatter triggers** | Locked per skill in `SKILL.md` frontmatter. **Do not modify without re-validating natural-language routing** (the harness reads these for skill selection). |
| **Brief-constructor output contract** | fal-image and fal-video brief-constructors must include: forbidden-preamble HARD RULE · `BRIEF_FAILED:` failure mode · first-30-chars self-check. **Never weaken this** — preamble contamination was caught and fixed in fal-image's first build, and `strip_preamble()` in `scripts/orchestrator.py` is the belt-and-suspenders backup. |
| **Output paths** | CWD-only: `<cwd>/generated/{images,videos,gifs}/<YYYY-MM-DD>/<slug>-vNN.<ext>` + `.json` sidecar. Slug rules + JSON schema live in each skill's `references/outputs.md`. |
| **Sidecar required fields** | Every sidecar records `params.model`, `params.model_choice_reason`, `params.reference_images`, `params.preflight_confirmed`, `params.request_id`. Video sidecars also record `params.audio_enabled` + `params.audio_brief`. |
| **Native voice** | All skill content reads as if originally designed that way. No meta-comments, no `// per the plan we…`, no `// WIP`, no `// TODO`, no phase markers, no "previously" / "now we use" / "switched to". |
| **No emojis** | Anywhere in skill files (chat or files). |
| **SKILL.md size** | Orchestrators (fal-image, fal-video, gif-maker) cap at ≤160 lines. media-processing is a reference catalog — no cap applies. |
| **Anchor language** | Brief-constructors prefer concrete real-world anchors (camera bodies, lenses, publications, photographers, cinematographers) over generic quality language. |

---

## Working agreements (when extending a skill)

- **Plan before building.** For non-trivial changes use plan mode → write plan file → ExitPlanMode for approval. Then brick-by-brick.
- **Brick-by-brick execution.** TaskCreate before starting; mark `in_progress` before each step; `completed` only when acceptance gate passes.
- **Acceptance criteria are strict.** Every plan ships with numbered checks (file counts, grep patterns, line counts, smoke tests). All must pass before declaring done.
- **Course corrections are normal input.** When a locked decision changes mid-build, rewrite the plan file fresh. Don't paper over with edits.
- **Name trade-offs explicitly.** No defensive justification when the user pushes back — adjust and move on.
- **Restraint in writing.** Tables for comparison. Bullets for sequence. Bold for scan-keys. No paragraphs longer than 3 sentences in skill files.
- **Self-contain before declaring done.** Final acceptance check is always `grep -rn "/Users/r1ckrck" .claude/skills/<skill>/` plus a cross-skill grep for fal-image and fal-video.

---

## Open work

1. **Live fal validation** — run real generations against fal-image, fal-video, gif-maker with the new roster. Verify model picks behave as advertised, pre-flight summary reads cleanly, sidecar reproducibility, cost reporting. First real-use will surface anything that was theoretically right but operationally awkward.
2. **Iterate on findings.** Once live use surfaces real issues, fix them with the same plan-mode + acceptance-checks discipline.

---

## Reference

The skills are the source of truth. This file captures the conventions used to build them. To extend a skill, read its `SKILL.md` first, then the relevant `references/`, then make changes that respect the conventions above.
