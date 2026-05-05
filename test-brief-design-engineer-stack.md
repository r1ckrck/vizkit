# Test Brief — The Design Engineer's Stack

A multi-format generation brief for vizkit testing. One overarching topic rendered across image, video, GIF, and infographic, all visually and conceptually related.

---

## Topic

A working stack of tools that defines how a design engineer makes things. Reads as a flowchart — each component a named node, connections drawn as labeled flow lines, the whole thing showing how these pieces come together to produce shipped work.

---

## The components (consistent across formats)

The eight nodes that appear in every format:

1. **Markdowns** — text-based knowledge: design system, documentation, project notes, voice
2. **Skills** — named modular capabilities. Notable ones: figma-use, markset, docx, pptx, fal-video, frontend-design, code-reviewer, code-simplifier
3. **Claude Code IDE** — the workspace / brain where everything is orchestrated
4. **MCP (Model Context Protocol)** — the connective tissue. The "hands" layer that plugs external tools into the workflow.
5. **Paper** — AI-native design canvas. Real HTML/CSS, not SVG. Bidirectional with code through MCP.
6. **Figma** — design source-of-truth. Connected via MCP.
7. **fal.ai** — generative media (images, video, GIFs). Connected via MCP.
8. **Production** — where shipped work lives. Terminal node.

### Topology

Markdowns and Skills feed *into* Claude Code IDE. From there, MCP branches outward to Paper, Figma, and fal.ai. Work flows back through MCP into Claude Code IDE, then forward into Production. The whole thing reads as: knowledge + capabilities → orchestrator → connected tools → finished output.

---

## Image — flowchart poster

A poster of the eight components rendered as a designed flowchart. Nodes are clean geometric shapes — rounded rectangles or hexagons — sized by relative importance (Claude Code IDE largest, MCP a distinct connector node, Production a terminal). Connections drawn as hairline flow lines with small arrowheads and short labels at each connector ("orchestrates", "exposes", "calls", "ships").

Layout reads left-to-right or top-to-bottom with a clear primary axis. Markdowns and Skills enter from one side; MCP branches outward to Paper, Figma, and fal.ai; everything converges on Production at the terminal end. Each node carries its name in clean type plus a one-line descriptor in lighter weight underneath.

Mood: technical engineering diagram. The kind of flowchart you'd find in a system architecture spec or a 1970s engineering reference poster — drafted, considered, not a Lucidchart cartoon. Hairline strokes, restrained palette, generous negative space around the connections so the flow is legible at a glance.

Hierarchy reads in a glance: the shape of the system. Closer reading rewards with the connector labels, the node descriptors, and the named skill badges on the Skills node.

---

## Video — 5 seconds, animated flowchart assembly

The same flowchart drawing itself in. Nodes appear in dependency order — Markdowns and Skills first, then Claude Code IDE, then MCP, then the three external tools branching out from MCP, then Production at the end. Each node fades or settles in with a small grounded entrance.

As MCP arrives, the connection lines to Paper, Figma, and fal.ai draw themselves outward — hairline strokes extending and arrowheads landing. The connector labels appear with each line as it completes. Final beat: the flow line into Production lands and the whole diagram holds.

Pacing: roughly half a second per node, with the connection lines drawing in between. Brief hold at the end on the completed flowchart. The viewer feels the system being assembled, never rushing.

Mood: weighted, considered motion. Hairlines that draw with a deliberate hand, nodes that settle without bounce. No spins, no zoom-ins, no transitional flourish.

Audio: a quiet ambient bed underneath — soft room tone, the faint sound of pen on paper as each hairline draws in, a subtle low tone or chord landing on the final beat as the diagram completes. Restrained, drafted feel, never musical or score-like. No voiceover.

Final frame matches the poster image, held for the closing beat.

---

## GIF — same concept, fresh generation, looping

The same flowchart assembly, looping cleanly. The completed flowchart is the rest frame; the loop returns by the connection lines retracting and nodes fading out before re-drawing — never a hard jump-cut back to the start.

Mood matches the video: quiet, weighted, hairline-stroke aesthetic, no GIF-y flourish.

---

## Infographic — dense, intricate, expanded flowchart

Where the poster is a single clean flowchart, this is the same flowchart with every node expanded into its own sub-diagram, all surrounded by supporting annotations and read as one composition.

Sections to include, each its own visual unit clearly delineated:

- The eight nodes from the poster, each one expanded:
  - **Skills** broken into its named examples — figma-use, markset, docx, pptx, fal-video, frontend-design, code-reviewer, code-simplifier — each with a one-line role descriptor, drawn as a sub-flow inside the Skills node
  - **Markdowns** broken into types of docs (system rules, project notes, design tokens, voice & design language, captures), drawn as branching leaves
  - **MCP** broken into the active connections — Paper, Figma, fal.ai, and others — with a "what flows through this connection" annotation per arrow
  - **Paper** with a short note on its bidirectional canvas and HTML/CSS native foundation
- A workflow timeline running across the bottom showing how a single piece of work moves through the flowchart from Markdowns to Production
- A side panel labeling **input → process → output** at each node
- A small detail diagram showing how Claude Code orchestrates across the connected tools, with MCP threads running between the relevant nodes
- A mini-legend defining the symbols used elsewhere on the piece (node shapes, line weights, arrowhead types)
- Margin annotations — ratios, percentages, frequencies where they make sense
- Small trade-off boxes attached to each node
- A footer strip with influences, references, and a version date

Mood: 1970s technical reference poster, modernized. A fold-out from a system architecture manual, or an engineering schematic. Every inch carries information; every section earns its real estate. White space is composed, not abundant.

Type treatment: multiple weights and sizes, clear hierarchy. Numbered or lettered sub-sections. Hairlines for all connectors. Icons only where they do semantic work.

Density target: a viewer scans the whole thing and grasps the system shape in 3 seconds, then spends 2 minutes finding details. That's the reward loop.
