# Recon map: Tekt (text-to-IFC), web + optional desktop

Scope: the core loop. Prompt in, streamed build visible in a 3D viewer, IFC out, then edit by prompt as versions. Deliverables (drawings, code screen, cost) are tiered as should/could.
For: not yet stated. The recon assumes a general text-to-BIM product and leaves the audience open. **Please confirm scope and audience.**
Date: 2026-10-10

Method note: the target is an open-source hackathon repo, not a hosted product. Following the skill's rules, this recon uses its **public documentation only** (README, `docs/architecture.md`, the one screenshot in `docs/`). No source files were read. The repo's file *names* (for example `Composer.tsx`) were used only to confirm that a screen or component exists.

## Sources

| # | source | URL | notes |
| --- | --- | --- | --- |
| 1 | README (pitch, flow, stack, limits) | https://github.com/nocoastaec/text-to-ifc/blob/main/README.md | primary |
| 2 | Architecture doc (steps, API, UI behaviour, deliverables, decisions) | https://github.com/nocoastaec/text-to-ifc/blob/main/docs/architecture.md | the richest source: sections 4.1-4.16 |
| 3 | Reference screenshot (early UI, "Generative BIM") | https://github.com/nocoastaec/text-to-ifc/blob/main/docs/screenshot.png | saved as `replica/screens/ref-early-ui-generative-bim.png`; the README says the UI has since changed |
| 4 | Newer UI screenshot | linked from the README (github user-attachments) | not fetched; **gap** |
| 5 | Project write-up (Google Doc) | https://docs.google.com/document/d/1tU3kCJchFRInzQXuF_GBEn_fQGtBhatj | not read; **gap**, worth a look |
| 6 | Dependency manifests | `backend/requirements.txt`, `frontend/package.json` in the repo | for stack awareness only |
| 7 | Live app | none; it is run locally | no hosted instance, no user account to drive; **screens are inferred from the docs, not seen** |

## Core loop

Type "two-storey house with a kitchen, three bedrooms and a garage", watch it build up in 3D one room, door and window at a time, download a valid IFC, then keep chatting to change it, each prompt becoming a new version.

## Screens

Confidence on screens is **medium**: assembled from the architecture doc and one older screenshot. Nothing was clicked through.

| ID | screen | route / how to reach | purpose | key components | states seen |
| --- | --- | --- | --- | --- | --- |
| S01 | Home | app start; no active session | start a project; list recorded runs | prompt composer, recorded-runs list, sample entry | empty, with recorded runs |
| S02 | Workspace (main) | active session | the working view: 3D model plus chat and history | 3D viewer, toolbar (Open IFC, Sample, Save IFC, Fit, Hide, Isolate, Show all), version badge | empty, building (live preview), done, load error |
| S03 | Sessions sidebar | left panel | switch, create or delete conversations; "New" | session list, New, Undo | empty, several sessions |
| S04 | Assistant / composer | in workspace | enter prompt or change, attach images, send (Ctrl+Enter) | multiline input, "+" attach menu, thumbnails, send button | idle, sending, attachments |
| S05 | Reasoning trace | in assistant, while and after a run | show the approach, per-step sentence with reason, grouped by phase, draft line, heartbeat, rejections | step list, phase groups, "drafting..." line, "waiting for the model... 12 s" | streaming, finished, step rejected, fix round |
| S06 | Model tree and properties | right panel | hierarchy, classes tab, element properties | tree, class visibility toggles, properties pane | nothing selected ("Click an element to inspect it"), selected |
| S07 | Version history | in sidebar or assistant | one card per version; snapshot becomes live orbit on hover; Inspect loads it into the main viewer; per-card IFC and Bundle download | version cards, prompt text, change summary chips | snapshot, live hover (max 3), current version highlighted |
| S08 | Deliverables | tabs on a version | code review, drawings, cost & carbon; "Show in model" per clause | tab bar, clause list, SVG sheet viewer, estimate table | per-version, loading |
| S09 | Export menu | top bar | IFC by default; caret lists drawings, DXF, XLSX, BCF, review, estimate, bundle | split button with menu | menu open |
| S10 | Viewer overlays | on the 3D view | floating inspector, view palette, section tool | floating panels, drag handles | collapsed, open |
| S11 | Replay mode | pick a recorded run, or `?replay=` | play back a stored run at 10x; then continue editing | same workspace plus playback | playing, finished |
| S12 | Settings / model config | `.env` file only, no UI | choose LLM provider | n/a | not a screen in the original; **gap** |

## Flows

```
F01 Create a building from a prompt
    S01 -> S04 (type, send) -> S05 (stream) + S02 (growing model) -> S02 (final)
    happy path clicks: 2 (type, send); the first version appears with no further input
    edge: model silent, invalid step rejected and repaired, requirement unmet -> fix round,
          unsupported wish reported not dropped, provider error, SSE drops mid-run

F02 Edit by prompt
    S02 -> (optionally select an element) -> S04 (type, send) -> S05 -> S07 (new card)
    happy path clicks: 2; 3 with a selected element
    edge: stale base version (409), the edit rejected, a selected element no longer exists

F03 Undo or revert
    S03 Undo or S07 card -> new version appended (copy of the old one)
    happy path clicks: 1
    edge: reverting the first version, concurrent run in flight

F04 Review and export
    S02 -> S08 (Code review / Drawings / Cost) -> S09 (Export) -> download
    happy path clicks: 2 for the IFC, 3 for another format
    edge: a failing clause -> "Show in model" highlights elements; older version exported from its card

F05 Replay a recorded run
    S01 recorded runs -> S11 playback -> S04 continue editing
    happy path clicks: 1 to start

F06 Open an IFC from disk
    S02 toolbar (Open IFC / Sample) -> S02 viewer
    edge: broken IFC file; the file is viewed only, not added to the project

F07 Attach an image to a prompt
    S04 "+" (or drag, or paste) -> thumbnail -> send
    edge: wrong type, too large; bytes never stored client-side
```

## Components

| component | variants | states | used on |
| --- | --- | --- | --- |
| Button | primary (Apply change), secondary (toolbar), split with caret (Export) | default, hover, disabled, loading | S02, S03, S09 |
| Prompt composer | multiline, with attach menu | idle, sending, drag-over | S01, S04 |
| Step list item | ok, rejected, draft, heartbeat | streaming, done | S05 |
| Phase group | collapsible | open, closed | S05 |
| Version card | snapshot, live 3D | hover, current, loading | S07 |
| 3D viewer | main, card | loading, error | S02, S07 |
| Tree view with visibility toggles | hierarchy, classes | expanded, hidden, isolated | S06 |
| Properties pane | element | empty, populated | S06 |
| Floating panel | inspector, view palette | resizable, collapsed | S10 |
| Resizer handle | vertical, horizontal | drag, keyboard, double-click reset | all panels |
| Tabs | deliverables, hierarchy/classes | active | S06, S08 |
| Toast / status line | info, error | n/a | S02 |

## Inferred data model

```
Project     id, name
            evidence: API POST /projects, sessions in sidebar        confidence: high
Version     project_id, number, parent, mode (design|edit|ops|revert|import), prompt, llm,
            spec, design, checks, images, approach, guids, ops, notes, summary, ifc_path
            evidence: docs 4.12                                      confidence: high
Design      name, levels[], rooms[], doors[], windows[], stairs[], fixtures[], balconies[],
            porch, roof, wall_material, columns[], overrides[], notes[]
            evidence: docs 4.1                                       confidence: high
Level       id (L1.. / B1..), name, height
Room        id, name, level, kind, rect or poly, roofed, enclosed
Door/Window room, to, side or near, at (0..1), kind, width, height, sill
Step        step type, fields, optional why (flat vocabulary)         confidence: high
Requirement kind, text, count/side/room, supported flag, status
            (met | unmet | unsupported | not checked)                 confidence: high
BuildingSpec geometric IR: walls, slabs, roofs, doors, windows, columns, beams, spaces, stairs,
            fixtures, railings                                         confidence: medium
GuidMap     element id <-> IFC GlobalId per project                   confidence: high
Brick       parametric component card (geometry as expression nodes)  confidence: medium
Skill       markdown how-to notes the model reads on demand           confidence: high
Attachment  image bytes (content-addressed) per version              confidence: medium
RunRecord   recorded event stream per version (for replay)           confidence: medium
```

Relationships: Project 1-n Version (linear history); Version 1-1 Design; Design 1-n Level, Room, Door, Window ...; Version 1-n Requirement result; Project 1-n GuidMap entries; Version 0-n Attachment.

## Feature matrix

See `features.csv`. Must: 11, should: 13, could: 14 (38 rows), of which 4 are marked skip.

## Out of scope (cannot or should not be cloned)

- The **Tekt** name, logos and brand assets, and the bundled recorded runs and sample IFCs. These are their content.
- The README, docs text and prompts: write your own words.
- **No LICENSE file** was found in the repo. Until the authors say otherwise, treat their code as all-rights-reserved. Reuse ideas and architecture only; do not copy code, the brick library JSON, the skills markdown or the prompts.
- Third-party libraries (IfcOpenShell, That Open, web-ifc, Tauri) have their own licences. They are fine to depend on, but check each (IfcOpenShell is LGPL).
- Optional extras that are in the original's backend without a UI (slicer, G-code, construction job): skipped.
- Arbitrary-IFC editing: not even in the original.

## Size

Screens 12 (11 real), flows 7, entities 12. Hard parts:
1. The **IFC compiler and derivation**: walls from room edges, openings, roofs, stairs, stable GlobalIds. This is where the real work is.
2. **Streaming LLM pipeline**: partial-JSON parsing, per-step validation with repair, concurrent preview compiles, SSE, requirements check.
3. **In-browser IFC viewer** with version-pinning pitfalls (web-ifc), plus the multi-viewer history cards.
Also non-trivial: the visual self-check renderer and the code review, which you can defer.
Size: **L** for the full scope; **M** for the core loop alone (steps, derive, compile, viewer, versions).

## Open questions for you
1. Confirm the slice (core loop first, deliverables later?) and the audience.
2. The Google Doc write-up and the newer README screenshot were not read. Say the word and I'll try them.
3. Whether you want a clean-room clone, a fork or a learning exercise. The missing LICENSE changes that answer.
