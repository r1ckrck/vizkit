# VizKit — Skill Development Workspace

This project is the **development workspace** for VizKit, a set of Claude Code skills for AI media generation. **This file is for working ON the skills (developing, refactoring, extending) — not for using them.** End-user routing happens via each skill's own `SKILL.md` frontmatter.

The skills are project-scoped (`<project>/.claude/skills/<name>/`) and copied (folder-and-all) into other projects when used to generate actual images / videos / GIFs.

---

## Skills built

| Skill | Files | What it does | Status |
|---|---:|---|---|
| **fal-image** | 19 | Text-to-image · edit · upscale · batch via fal.ai | Built · prompt-construction smoke-tested · awaiting live fal validation |
| **fal-video** | 16 | Text-to-video · image-to-video · batch via fal.ai | Built · prompt-construction smoke-tested · awaiting live fal validation |
| **gif-maker** | 5 | Orchestrator: spawns fal-video brief-constructor + runs ffmpeg palette/paletteuse locally | Built · 14 acceptance checks passed · awaiting live validation |
| **media-processing** | 13 | All ffmpeg + ImageMagick capability as on-demand bash recipes | Built · lean reference-only · script-free |

---

## Folder architecture

```
vizkit/
├── .claude/
│   ├── settings.json                   ← user manages fal MCP registration here
│   └── skills/
│       ├── fal-image/
│       │   ├── SKILL.md                ← orchestrator (138 lines)
│       │   ├── agents/brief-constructor.md
│       │   ├── presets/                ← 9 per-domain
│       │   │   ├── cinema · product · portrait · editorial
│       │   │   ├── ui · logo · landscape · abstract · infographic
│       │   ├── preset-template.md
│       │   └── references/             ← 7 files
│       │       ├── prompt-engineering.md (5-component formula)
│       │       ├── domains.md (9 domains + modifier libraries)
│       │       ├── clarification-mcq.md · editing.md
│       │       ├── upscaling.md · batch-variations.md · outputs.md
│       │
│       ├── fal-video/
│       │   ├── SKILL.md                ← orchestrator (142 lines)
│       │   ├── agents/brief-constructor.md
│       │   ├── presets/                ← 6 per-domain (no UI/Logo/Infographic)
│       │   │   └── cinema · product · portrait · editorial · landscape · abstract
│       │   ├── preset-template.md
│       │   └── references/             ← 7 files
│       │       ├── prompt-engineering.md (6-component formula, adds Motion)
│       │       ├── motion-vocabulary.md (camera moves · pacing · intensity)
│       │       ├── domains.md (6 domains + modifier libraries)
│       │       ├── clarification-mcq.md · image-to-video.md
│       │       ├── batch-variations.md · outputs.md
│       │
│       ├── gif-maker/
│       │   ├── SKILL.md                ← orchestrator (127 lines)
│       │   ├── default.preset.md       ← single preset (not per-domain)
│       │   └── references/             ← 3 files
│       │       ├── strategy.md (5 intent buckets + fal-video domain map)
│       │       ├── source-clip-spec.md (override-notes pattern)
│       │       └── ffmpeg-commands.md (two-pass palettegen + paletteuse)
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
├── CLAUDE.md                           ← you are here
└── test-brief-design-engineer-stack.md ← user's test brief for live validation
```

---

## Architectural pattern (used by fal-image and fal-video)

Two-layer thin-orchestrator pattern, borrowed from banana-claude:

| Layer | Lives in | Job |
|---|---|---|
| **Orchestrator** | `SKILL.md` | Detect domain · run MCQs · spawn subagent · call fal MCP · save outputs · report cost |
| **Brief constructor** | `agents/brief-constructor.md` (Sonnet via Task tool) | Read formula + domain + preset → return one optimized prompt string |
| **Per-domain presets** | `presets/<domain>.preset.md` | Locked styling defaults (one per domain) |
| **References** | `references/*.md` | Loaded on demand by the subagent |

**gif-maker** is a pure orchestrator — no own brief-constructor. It spawns fal-video's brief-constructor with GIF-aware constraints injected as `custom override notes`.

**media-processing** is a reference catalog (no orchestrator pattern, no subagent). Claude composes ffmpeg/ImageMagick bash from the references on demand.

---

## Conventions (apply to every skill)

| Convention | Rule |
|---|---|
| **Self-containment** | Each skill folder is portable. `grep -rn "/Users/r1ckrck" .claude/skills/<skill>/` returns 0 hits. Doc placeholders (`/Users/<user>`, `/Users/foo`) are acceptable. |
| **Skill-root resolution** | Use `$PWD/.claude/skills/<skill>/`. No `~`. No env vars in paths. Pass absolute path to subagents at runtime. |
| **Frontmatter triggers** | Locked per skill in `SKILL.md` frontmatter. **Do not modify without re-validating natural-language routing** (the harness reads these for skill selection). |
| **Brief-constructor output contract** | fal-image and fal-video brief-constructors must include: forbidden-preamble list · `BRIEF_FAILED:` failure mode · first-30-chars self-check. **Never weaken this** — preamble contamination was caught and fixed in fal-image's first build. |
| **Output paths** | CWD-only: `<cwd>/generated/{images,videos,gifs}/<YYYY-MM-DD>/<slug>-vNN.<ext>` + `.json` sidecar. Slug rules + JSON schema live in each skill's `references/outputs.md`. |
| **No emojis** | Anywhere in skill files (chat or files). |
| **No meta-comments** | No `// per the plan we…`, no `// WIP`, no `// TODO`. |
| **SKILL.md size** | Orchestrators (fal-image, fal-video, gif-maker) cap at ≤150 lines. media-processing is a reference catalog — no cap applies. |
| **Banned-word handling** | Advisory in v1 (Gemini-tuned originally). Sidecar records `params.banned_word_mode: "advisory"`. Empirical re-validation per fal model is a follow-up. |

---

## Working agreements (when extending a skill)

- **Plan before building.** For non-trivial changes use plan mode → write plan file → ExitPlanMode for approval. Then brick-by-brick.
- **Brick-by-brick execution.** TaskCreate before starting; mark `in_progress` before each step; `completed` only when acceptance gate passes.
- **Acceptance criteria are strict.** Every plan ships with numbered checks (file counts, grep patterns, line counts, smoke tests). All must pass before declaring done.
- **Course corrections are normal input.** When a locked decision changes mid-build, rewrite the plan file fresh. Don't paper over with edits.
- **Name trade-offs explicitly.** No defensive justification when the user pushes back — adjust and move on.
- **Restraint in writing.** Tables for comparison. Bullets for sequence. Bold for scan-keys. No paragraphs longer than 3 sentences in skill files.
- **Self-contain before declaring done.** Final acceptance check is always `grep -rn "/Users/r1ckrck\|<other-deleted-paths>" .claude/skills/<skill>/`.

---

## Open work

1. **Live fal validation** — register fal MCP in `.claude/settings.json`, run real generations against fal-image, fal-video, gif-maker. Verify sidecar reproducibility, cost reporting, and output quality.
2. **Empirical banned-word re-validation per fal model** — after ≥50 real generations across multiple models, audit which banned-list terms actually correlate with degraded output. Promote advisory list to strict per-model where warranted.
3. **gif-maker live test** — exercise all 5 intents (loader, micro-animation, hero, decorative, photographic) end-to-end after fal-video is live-validated.

---

## Reference

The skills are the source of truth. This file captures the conventions used to build them. To extend a skill, read its `SKILL.md` first, then the relevant `references/`, then make changes that respect the conventions above.
