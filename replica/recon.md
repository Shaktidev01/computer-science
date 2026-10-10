# Recon map: Tekt (text-to-IFC), web + optional desktop

Scope: the core loop. Prompt in, streamed build visible in a 3D viewer, IFC out, then edit by prompt as versions. Deliverables (drawings, code screen, cost) are tiered as should/could.
For: not yet stated. The recon assumes a general text-to-BIM product and leaves the audience open. **Please confirm scope and audience.**
Date: 2026-10-10

Method note (revised): the target is a public GitHub hackathon repo, not a hosted product, and you asked for a complete pass. So this recon now has three layers:
1. **Docs**: README and `docs/architecture.md`.
2. **The running app**: I installed the repo's dependencies in a scratch directory, ran its backend (offline mock LLM) and frontend, and drove it with a headless browser. Screens, states and labels below marked *seen* come from this. Captures are in `replica/screens/live-*.png` (reference only, never shipped).
3. **UI source, labels only**: I read `Home.tsx`, `Sidebar.tsx`, `TopBar.tsx` and scanned the other components for button labels and titles, to name controls precisely. I did not read the backend or copy any code.

Clean-room note: now that source has been looked at, keep the person who writes the clone separate from these notes where possible, or at least write everything fresh from the behaviour described here. Do not copy code, prompts, the brick JSON or skills markdown.

## Sources

| # | source | URL | notes |
| --- | --- | --- | --- |
| 1 | README (pitch, flow, stack, limits) | https://github.com/nocoastaec/text-to-ifc/blob/main/README.md | primary |
| 2 | Architecture doc (steps, API, UI behaviour, deliverables, decisions) | https://github.com/nocoastaec/text-to-ifc/blob/main/docs/architecture.md | the richest source: sections 4.1-4.16 |
| 3 | Reference screenshot (early UI, "Generative BIM") | https://github.com/nocoastaec/text-to-ifc/blob/main/docs/screenshot.png | saved as `replica/screens/ref-early-ui-generative-bim.png`; the README says the UI has since changed |
| 4 | Newer UI screenshot | linked from the README (github user-attachments) | **blocked**: the sandbox proxy returned 403 for `github.com/user-attachments`. Replaced by my own captures of the running app (#7) |
| 5 | Project write-up (Google Doc) | https://docs.google.com/document/d/1tU3kCJchFRInzQXuF_GBEn_fQGtBhatj | **blocked**: `docs.google.com` is not reachable from this sandbox's network policy. Still a **gap** |
| 6 | Dependency manifests | `backend/requirements.txt`, `frontend/package.json` in the repo | for stack awareness only |
| 7 | Running app (local) | built from the repo, served on localhost during recon | no hosted instance exists, so I ran it. **Mock LLM only**: no API key, so real-model output quality and the model's reasoning text were not seen live. Replay of a recorded run was started but not observed to completion |

## Core loop

Type what you want to design (a house, but the home screen also suggests a bridge, road, data centre, rail station, treatment plant, substation), watch it build up in 3D one room, door and window at a time, download a valid IFC, then keep chatting to change it, each prompt becoming a new version.

## Screens

Screens S01-S10 were **seen** in the running app (mock LLM). S11 was started but not watched to the end. S12 does not exist as a screen.

| ID | screen | route / how to reach | purpose | key components | states seen |
| --- | --- | --- | --- | --- | --- |
| S01 | Home | start; or breadcrumb "Home" | start a session | headline "What are we designing today?", hero composer (placeholder "Describe an asset - a bridge, a road, a data center, a building..."), model picker ("Offline mock" dropdown), "+" attach menu, send arrow, 7 suggestion chips (Rome sample, Highway bridge, Access road, Data center, Rail station, Treatment plant, Substation) plus "Open an IFC file", "Recorded runs" list (prompt, model, "9 min live") | empty, with recorded runs, backend offline (sidebar pill) |
| S02 | Sidebar | left rail, collapsible | sessions and files | logo, search toggle, collapse, "New Session", "Open IFC...", "Sample model", "Past sessions" list with delete, backend status card ("Connected", "Model: mock · IfcOpenShell") | no sessions ("Nothing yet - describe an asset to start."), several, search with no matches, backend offline |
| S03 | Top bar | top of workspace | navigation and export | "Home / <session title>" breadcrumb, toggle sidebar, toggle assistant, "..." menu (Delete session), split "Export IFC" button with a format menu, light/dark theme toggle | export disabled until a version exists, menu open |
| S04 | Workspace: 3D model tab | centre, after first prompt | main model view | file tab (`preview.ifc` then `v1.ifc`), tabs 3D model / Drawings / Code review / Cost & carbon (locked until built), status badge ("preview 4 · 127 elements" then "v1 · final · 162 elements"), floating view palette (Top, Bottom, Front, Back, Left, Right, Isometric, reset, show-spaces), info card ("Click an element to inspect it and to make the next prompt about it"), section slider ("cut at 2.7 m · inside Ground Floor", "follow build" checkbox), axis gizmo | live preview growing, final, element selected |
| S05 | Assistant panel | right, collapsible | conversation, reasoning, versions | prompt echo, "The parti: ..." summary, "Working · 19.4s" status, step groups with timings ("Reading the brief 2.1 s", "Design thinking 81 ms"), requirement chips ("2 storeys", "1 kitchen"...), italic reasoning text, jump-to-latest, version card, composer docked ("Describe an asset..." / "Describe a change...") | streaming, done, stopped with error ("the model produced no applicable steps"), step rejected |
| S06 | Version card | in assistant, per version | one entry per version | 3D snapshot ("Rendering model..." then "Hover to explore in 3D"; "Drag to orbit · scroll to zoom", Reset, Inspect), actions IFC / PDF / Bundle / "In workspace", text summary ("Version 1: 2 levels and 162 elements, as valid IFC4. Spaces: ..."), counts per IFC class (IfcWall 29, IfcWindow 11...), collapsibles "Interpreted by <llm>" and "Structured BIM instructions", "Requirements: n of m met" chip, View / restore | rendering, ready, current, older (restore), failed |
| S07 | Drawings tab | tab on a version | the generated sheet set | sheet thumbnails (G-001 Cover, G-002 Code analysis, A-101 Ground Floor plan 1:100, A-102 Level 2...), main sheet viewer, toolbar: full-screen, SVG, "PDF set", DXF, Schedules, Bundle; "7 sheets · A3 · US NCS numbering" | loading, loaded |
| S08 | Code review tab | tab on a version | indicative IRC/IBC/ADA screen | header "IRC 2021 · indicative design-stage screen", summary cards (Occupancy R-3, Occupant load, Gross area, Construction type), filter chips ("11 clauses", "10 pass", "1 info"), clause rows (measured vs required, section ref) with "Show in model", BCF issues and Report buttons | pass, warn, fail, info |
| S09 | Cost & carbon tab | tab on a version | cost plan and embodied carbon | construction cost range (AACE Class 5) with UniFormat bars, upfront carbon kgCO2e/m2 with a LETI band scale, "Design moves that cut carbon" list, quantities table, Workbook (.xlsx) and Cost plan (.csv) buttons | loaded |
| S10 | Export menu | top bar caret | 12 formats | IFC, Drawing set (.pdf), CAD plans (.dxf), Schedules (.xlsx), Review issues (BCF 2.1), Full bundle (.zip), Design review (.md), Cost & carbon (.csv), Design summary, Schedule (.csv), BIM spec (.json), Design record (.json) | open |
| S11 | Replay | click a "Recorded run" on Home, or `?replay=` | play back a real model run | same workspace; header shows "waiting for the model..." at start | starting (not observed to the end) |
| S12 | Model settings | `.env` file only | choose provider | n/a, no in-app settings screen (the composer has a model picker only) | n/a |

Also seen: a **dark theme** (full restyle, same layout) and a **light theme** (default).
Mobile: not tested; the layout is a three-column desktop app.

## Flows

```
F01 Create a building from a prompt
    S01 -> type in hero composer -> Enter (or click a suggestion chip: 1 click) -> S05 reasoning stream + S04 growing model -> final version
    happy path clicks: 2 to send (1 with a suggestion chip); takes about 20 s on the mock model, 6-9 min on the recorded real-model runs
    seen: the build is shown as numbered previews ("preview 4 · 127 elements") ending in "v1 · final · 162 elements"
    edge: model silent, invalid step rejected and repaired, requirement unmet -> fix round,
          unsupported wish reported not dropped, provider error, SSE drops mid-run

F02 Edit by prompt
    S04 -> (optionally click an element) -> docked composer ("Describe a change...") -> S05 -> S06 (new card)
    seen: with the mock model, "Add a basement" ended in a red "Stopped after 0.9 s · 10 steps ... the model produced no applicable steps" state, so a failed edit leaves the previous version in place
    happy path clicks: 2; 3 with a selected element
    edge: stale base version (409), the edit rejected, a selected element no longer exists

F03 Undo or revert
    S06 older version card -> restore ("Make this version the head again") -> new version appended (copy of the old one)
    happy path clicks: 1
    edge: reverting the first version, concurrent run in flight

F04 Review and export
    S04 -> S07 / S08 / S09 tabs -> S10 export menu or the per-tab buttons -> download
    happy path clicks: 2 for the IFC, 3 for another format
    edge: a failing clause -> "Show in model" highlights elements; older version exported from its card

F05 Replay a recorded run
    S01 recorded runs -> S11 playback -> S05 continue editing
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

See `features.csv`. Must: 12, should: 19, could: 18 (49 rows), of which 4 are marked skip.

## Out of scope (cannot or should not be cloned)

- The **Tekt** name, logos and brand assets, and the bundled recorded runs and sample IFCs. These are their content.
- The README, docs text and prompts: write your own words.
- **No LICENSE file** was found in the repo. Until the authors say otherwise, treat their code as all-rights-reserved. Reuse ideas and architecture only; do not copy code, the brick library JSON, the skills markdown or the prompts.
- Third-party libraries (IfcOpenShell, That Open, web-ifc, Tauri) have their own licences. They are fine to depend on, but check each (IfcOpenShell is LGPL).
- Optional extras that are in the original's backend without a UI (slicer, G-code, construction job): skipped.
- Arbitrary-IFC editing: not even in the original.

## Size

Screens 12 (11 real, 10 seen live), flows 7, entities 12. Hard parts:
1. The **IFC compiler and derivation**: walls from room edges, openings, roofs, stairs, stable GlobalIds. This is where the real work is.
2. **Streaming LLM pipeline**: partial-JSON parsing, per-step validation with repair, concurrent preview compiles, SSE, requirements check.
3. **In-browser IFC viewer** with version-pinning pitfalls (web-ifc), plus the multi-viewer history cards.
Also non-trivial: the visual self-check renderer and the code review, which you can defer.
Size: **L** for the full scope; **M** for the core loop alone (steps, derive, compile, viewer, versions).

## Open questions for you
1. Confirm the slice (core loop first, deliverables later?) and the audience.
2. The Google Doc write-up is still unread (`docs.google.com` is blocked by the sandbox network policy). Allow that host in the environment's network settings and I'll fetch it. The newer README screenshot was replaced by my own live captures.
3. Whether you want a clean-room clone, a fork or a learning exercise. The missing LICENSE changes that answer.
4. Real-model behaviour (reasoning text quality, build time, cost) was not observed live, only through docs and the two recorded Claude runs. Running with an Anthropic key would close that gap.
