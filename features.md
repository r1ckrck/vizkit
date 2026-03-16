Build a local, single-page power-user web tool that generates portfolio-ready infographic images using Google’s Nano Banana image models. The UI exposes structured controls (chips, toggles, dropdowns, sliders) based on established prompt frameworks, so I can compose “smart” infographic prompts quickly without heavy typing. Branding is handled through editable prompt-based style guidelines (auto-composed from simple brand controls) to keep visuals consistent per project and easy to switch between projects. The tool supports generating multiple variants, previewing results, and iterating fast; every output is auto-saved into the project folder with a JSON metadata sidecar capturing the prompt, framework, model, branding snapshot, and generation parameters.

Feature list:
Single-page, power-user UI (no navigation) with progressive disclosure (basic + advanced)
API key input + Nano Banana model selector
Project root + output folder configuration (save into project directory)
Structured Branding controls that auto-compose a “Brand Prompt Block” (still editable)
Framework-driven Prompt Builder (selector swaps structured control sets)
Prompt frameworks implemented as typed controls (chips/toggles/dropdowns/sliders) with optional Notes/Overrides
Primary “Infographic Prompt Canvas” with repeatable content blocks (stats/comparison/timeline/process/checklist) + hierarchy/layout/style controls + prompt linting
Generation parameter controls (aspect ratio, resolution/size, variants count; plus optional advanced params if available)
Variant generation + variant grid preview
History of recent generations
Click a saved variant to reload all settings (framework fields, branding, params, final prompt)
Auto-save image files + JSON metadata sidecars (prompt, framework, brand snapshot, params, model, timestamp, paths)
Optional advanced: reference image upload, modality options (if supported), quick iteration chips for common refinements