You are a “vibe-coding” builder agent. Your job is to DESIGN + IMPLEMENT (no need to over-explain) a local, single-page power-user web app that generates portfolio infographic images using Google’s Nano Banana image generation models. Make sensible product/UX choices where details are missing, but stay within the constraints below.

PRODUCT
- Name: Infogen (simple utility branding; minimal UI chrome)
- Purpose: Generate brand-consistent infographic images for my portfolio with ≤5 iterations per infographic.
- Target user: me (power user). Complexity is OK. Navigation is NOT OK.

HARD CONSTRAINTS (DO NOT VIOLATE)
- Single-page app (one route). No multi-page flows.
- 1–2 step workflow: edit inputs → generate → iterate. No “project setup wizard”.
- Progressive disclosure: basic controls upfront; advanced controls collapsible.
- Local-first output: generated images + JSON metadata are saved into a folder inside the same project directory.
- No “Download” button (files are already saved to disk).
- No security/auth work: I will paste my API key manually in the UI; keep it simple.

CRITICAL UX INTENT (MUST ENFORCE)
- This is NOT a “few textareas + generate” app.
- On first load, I must see a control-rich form: chips/toggles/dropdowns/sliders that meaningfully change generation.
- Frameworks must drive the UI: each framework becomes a structured control set, not just instructions.
- Default experience should be ≥70% structured controls and ≤30% free text.
- Free-text is allowed only as “Notes / Overrides” under each framework.

TECH TARGET
- Next.js + React + TypeScript
- Tailwind + shadcn/ui (or similar primitives)
- Local saving using Node filesystem (assume local runtime)
- Clean, extensible architecture

THE ONE PAGE (LAYOUT + UX)
- Two-pane layout:
  - LEFT: Controls (accordion sections)
  - RIGHT: Output (main preview + variants grid + small history)
- Left panel sections (accordion order):
  1) Setup (always visible)
  2) Branding (structured controls + generated brand block)
  3) Prompt Builder (framework-driven structured UI)
  4) Generation Params (Nano Banana variables)
  5) Advanced (collapsed)
- Speed behaviors (power user):
  - Keyboard-friendly, clear tab order
  - “Generate” always visible
  - Clear system feedback: loading, errors, saving status, selected variant

SECTION 1: SETUP (ALWAYS VISIBLE)
Inputs:
- API Key (text input)
- Model dropdown (Nano Banana model identifiers; pick correct current ones per Google docs/SDK)
- Project root path (string input; sensible default)
- Output folder name (default: ./infogen_outputs/)
Output rule:
- Save to <projectRoot>/<outputFolder>/<sessionOrProjectSlug>/
- Keep structure predictable and sortable

SECTION 2: BRANDING (STRUCTURED, PROMPT-BASED)
Branding must ultimately be a prompt snippet injected into every final prompt, but editing it should be fast via controls.

UI requirements:
- Brand Profile Controls (structured):
  - Palette chips (primary/secondary/accent/background) with hex inputs
  - Typography tone dropdown (e.g., modern/serif/tech/minimal/editorial) + weight/contrast chips
  - Icon/illustration style chips (flat/outline/duotone/3D; stroke weight)
  - Layout vibe chips (airy/dense/grid/card-based) + corner radius chips
  - Do/Don’t toggles (e.g., “no gradients”, “limited to 2 colors”, “no shadows”, “high whitespace”)
- Auto-compose a “Brand Prompt Block” textarea from these controls:
  - Show it as editable text (source of truth can remain the textarea), but changes in controls update it.
  - Allow manual edits; if manual edits diverge, keep both (show “customized” state) without breaking.

SECTION 3: PROMPT BUILDER (FRAMEWORK-DRIVEN, STRUCTURED CONTROLS)
Core rule:
- Each framework must define a schema of typed controls (chips/toggles/dropdowns/sliders/repeatable blocks).
- Only a small “Notes / Overrides” textarea is allowed per framework.

Framework selector UI:
- Use tabs, chips, or dropdown (choose best), but switching frameworks must visibly swap the control set.

Include these frameworks (make them real UIs, not text guidance):
1) Narrative Descriptive Prompt
   - Controls: subject chips, setting dropdown, style chips, mood chips, detail level slider, composition chips
2) Composition / Photography Controls
   - Controls: shot type dropdown, lens chips, lighting chips, color grading chips, framing chips, depth-of-field toggle
3) Text-in-Image / Poster Template
   - Controls: exact headline field, subtext fields, text placement dropdown, hierarchy chips, “text clarity” slider, margin/spacing chips
4) Step-by-Step Construction
   - Controls: background selector, foreground selector, label style chips, callout toggle, finishing constraints chips
5) Semantic Negatives / Exclusions
   - Controls: “avoid” chips library + custom chip input, plus positive constraint toggles (“empty background”, “no clutter”)
6) Infographic Prompt Canvas (PRIMARY MODE)
   - Structured controls:
     - Infographic type chips: Stat / Comparison / Timeline / Process / Checklist / KPI Card / Flow
     - Title field + optional subtitle
     - Repeatable content blocks:
       - Stat block (label + number + unit + annotation)
       - Comparison block (A vs B fields)
       - Timeline block (date/step + label)
       - Process block (step number + label)
       - Checklist block (bullets)
     - Hierarchy controls:
       - Max items slider (e.g., 3–8)
       - “Primary focus” dropdown (headline / chart / key stat)
       - Emphasis chips (bigger headline, bold numbers, grouped cards)
     - Layout intent controls:
       - Layout chips (grid / 2-column / cards / poster / split)
       - Whitespace slider
       - Alignment chips (left/center)
     - Style intent controls:
       - Visual style chips (vector/flat/minimal/outlined)
       - Detail slider (simple ↔ detailed)
       - Color usage toggle (monochrome/2-color/full palette)
     - “Clarity check” linting panel (auto):
       - Warn if too much text, missing numbers, weak hierarchy, conflicting layout instructions
       - Suggest one-click fixes via chips (e.g., “reduce items to 5”, “convert bullets to cards”)

Prompt composition rules (for all frameworks):
- Always inject Brand Prompt Block into the composed prompt.
- Show “Final Composed Prompt” read-only preview.
- Provide a “Copy final prompt” button.
- Optional: “Raw override” textarea (collapsed) that can append/replace, but does not remove the structured control system.

SECTION 4: GENERATION PARAMS (NANO BANANA VARIABLES + APP CONTROLS)
- Expose key image generation controls as structured UI:
  - Aspect ratio dropdown
  - Resolution/size dropdown
  - Variants count stepper (1–8)
- If API supports other params (seed, guidance/strength, safety, etc.), add them as secondary controls or under Advanced.
Behavior:
- Implement Variants count as repeated calls if the API doesn’t reliably return N images in one response.
- Each variant must be saved with unique names + metadata.

SECTION 5: ADVANCED (COLLAPSED BY DEFAULT)
- Reference images uploader (optional)
- Response modality if supported (image-only vs image+text)
- Quick iteration chips that tweak the prompt without heavy typing:
  - “Increase whitespace”, “Reduce text density”, “Stronger hierarchy”, “Bigger headline”,
    “More minimal”, “Simplify icons”, “Use fewer colors”, “Improve legibility”

OUTPUT PANEL (RIGHT SIDE)
- Primary preview (selected variant)
- Variants grid (click to select)
- Lightweight history list (recent generations)
- Clicking a saved variant repopulates ALL relevant state:
  - framework selection + framework fields
  - brand controls + brand block
  - generation params
  - final composed prompt
- Show status: generating, saved path, errors with actionable hints

FILE SAVING + METADATA (REQUIRED)
- For every generated image, save:
  - Image file (png or best available format)
  - JSON sidecar with:
    - timestamp
    - model
    - framework name
    - full composed prompt
    - brand block snapshot
    - brand control values snapshot (palette, typography, etc.)
    - all generation params
    - variant index
    - output file path
- Naming:
  - slug + timestamp + vXX (clean, sortable)

QUALITY / SUCCESS CRITERIA
- I can create branded infographics by mainly changing:
  - Brand controls (or Brand Prompt Block) + Infographic Canvas blocks
- Typical infographic converges in ≤5 iterations.
- The UI feels like a power tool: one screen, minimal friction, progressive disclosure works.
- The structured controls are the primary way to operate (not typing long prompts).

NON-GOALS (DO NOT ADD)
- Multi-page project management
- Accounts/auth/team features
- Cloud hosting polish
- Overdesigned onboarding
- Export/download workflows

DELIVERABLES
- Working local app meeting the above.
- Short README:
  - how to run
  - where outputs are saved
  - which models/SDK are used
  - how to add a new framework schema/control set

EXECUTION GUIDANCE
- Where details are underspecified, pick sensible defaults and keep it extensible.
- Make it usable first, then refine.
- Avoid over-polishing visuals; prioritize control richness, iteration speed, and reliability.