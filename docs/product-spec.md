# World Tree — Product Spec

Oct 6, 2026 · @Fernando Hernandez

## 1. Overview

World Tree is a local-first workspace where software, research and hardware projects are born and grow: projects, sections, tasks, a kanban board, a wiki, git branches and worktrees, and read-only views of code, text and diagrams, in one app. It is the place where AI agents (Hermes Agent, Claude Code, later Eter) read and write project state through a World Tree MCP server; the UI is one client of that server.

This spec covers the full product vision so mockups can cover every screen; phase tags (P1, P2, P3) mark what gets built first. Implementation details stay light and are refined in planning.

**Problem.** Project state is scattered across VS Code, terminal tabs, a codegraph tool, diagram apps, notes and cloud trackers. Agents can only reach it through ad hoc file reads or vendor APIs, and the human has no single view of what exists, what is in progress and what changed.

**Goals**

- One place to see every project, its sections, tasks, docs, branches and recent agent activity.
- Start any project in seconds from a template (Software, Research, Hardware, Mixed, Custom) and switch modules on or off per project; create and edit your own templates.
- Project data lives as plain files in git (Markdown + YAML frontmatter), owned by the user, readable by any tool.
- Agents and the UI share one API: the World Tree MCP server (tasks, wiki, projects, worktrees).
- Kanban and task flow that understand git: a task can link to a branch or worktree and show its diff status.
- Read-only views of code, Markdown, diagrams and the codebase graph, plus an integrated terminal, so reading code and running project commands never require another app.
- First-class tools for research projects (sources, findings, questions) and hardware projects (bill of materials, build log, design files).
- One home for the Alrest ecosystem: provisional screens for Eter, Aegis Eye, Resonance and Siren Panel, filled in as their contracts are defined.

**Non-goals (MVP)**

- A code editor or IDE. Editing code is the agent's job; World Tree links out to the user's editor when a human must edit. The integrated terminal is a shell, not an editor.
- Running or orchestrating agents. Dispatch belongs to Eter Project; World Tree only records and shows.
- Its own codegraph engine (reuse codebase-memory-mcp), its own vector RAG, or its own diagram engine (embed Excalidraw).
- Multi-user, cloud sync or SSO.
- A third-party plugin API. Modules are a fixed built-in catalog switched on per project; external plugins can be reconsidered after the third module ships.

**Context and constraints**

- Single user. Desktop first on macOS; iPhone, Android phones and tablets come later as companion clients of the Mac host, so every screen is designed with a compact layout in mind from the start.
- On desktop it is used for hours a day: density and keyboard flow matter more than spectacle. On mobile the job is glance and act.
- First project to build in the Alrest ecosystem; the other projects (Eter, Aegis Eye, Siren Panel, Resonance, Affinity Tree) will be developed inside it, so dogfooding starts early.
- Visual language comes from the shared Alrest UI design system (Xeno style, Claude Design) in its restrained "workspace" density.
- Confidential work code may live in a workspace: no data leaves the Mac unless the user sends it or one of the user's paired devices reads it.

## 2. Users and primary workflows

There is one human user and several agent clients; mockups design for the human, and every human action must have an MCP equivalent so agents can do the same.

| Actor | Uses World Tree to | Through |
| --- | --- | --- |
| Owner (human) | Plan, triage, review agent output, read code and docs, decide | UI |
| Hermes Agent | Research, update wiki pages, report status, create and move tasks | MCP server |
| Claude Code sessions | Read task specs, update task status, link branches and worktrees | MCP server |
| Eter Project (later) | Pull ready tasks, dispatch them, write back results | MCP server + events |
| Siren Panel (later) | Glance at the board, accept or move tasks | Thin client of the same API |

**Jobs to be done (owner)**

1. Create a project: pick a template or Custom, choose modules, and land on a ready project with sections and starter files.
2. Morning check: what moved since yesterday, what needs me, what is blocked.
3. Capture an idea or task in seconds, into the right project and section.
4. Plan a feature: write a spec page, break it into tasks, order them on the board.
5. Start work on a task: create or link a branch and worktree, open a terminal there, hand the task to an agent.
6. Review: read the diff summary, the agent's notes on the task and the linked wiki changes; accept, send back or close.
7. Understand code without an IDE: browse the tree, open a file read-only, jump to the codegraph.
8. Run project commands (build, test, git, start Claude Code) from the integrated terminal or a saved-command chip.
9. Research: collect sources, write notes, record findings and open questions, promote findings to the wiki.
10. Build hardware: keep the bill of materials and supplier status, log build sessions with photos, store design files.
11. Record a decision (ADR) and see where it applies.
12. See the ecosystem: which agents are running and on what, latest benchmark results, whether the cyberdeck is connected.
13. Shape my own templates: save a project as a template or edit one from scratch.
14. Find anything across projects: task, page, file, branch, decision, source, part.

## 3. Core concepts and domain model

Everything hangs off the **Project**: a folder of files in git, with sections that group its tasks and pages. A **Workspace** is the set of projects World Tree knows about; it is a registry, not a container that owns their files.

| Entity | What it is | Key fields |
| --- | --- | --- |
| Workspace | Registry of projects plus app settings | name, project list (paths), templates folder, preferred editor and shell |
| Template | A reusable starting setup | id, version, name, glyph, modules, sections, labels, board columns, starter pages and tasks, module setup defaults |
| Module | A built-in capability switched on per project | id (wiki, decisions, code-repos, terminal, codegraph, diagrams, research, hardware, agents, benchmarks), phase, settings |
| Project | A unit of work with its own folder | id, key (task prefix), name, template@version it started from, enabled modules, status, description, glyph/color, created |
| Repo | A code repository attached to a project | path or remote, default branch, codegraph index status |
| Section | A grouping inside a project | id, name, order, description |
| Task | A unit of work | id, title, status, section, priority, assignee (human or agent name), labels, links (branch, worktree, pages, decisions, sources, parts), body, checklist, created/updated, due |
| Board | A view over tasks | columns mapped to statuses, filters, swimlanes |
| Page | A wiki or spec document | id, title, path, kind (wiki / spec / note / log), tags, backlinks, last editor |
| Decision | An ADR | id, title, status, date, context, decision, consequences, applies-to links |
| Source | A research input | title, URL or file, type (paper, web, video, repo, book), reading status, notes, highlights |
| Finding | A research claim with evidence | statement, confidence, supporting and contradicting sources, promoted page |
| Question | An open research question | text, status (open / answered / dropped), linked findings |
| Part | A bill-of-materials line | name, part number, quantity, supplier, unit price, currency, status (to buy / ordered / received / installed), link, notes |
| Build log entry | A dated hardware work session | date, summary, photos, linked tasks and parts |
| Design file | A hardware source file | path, type (KiCad, STL, STEP, Fusion export, PDF), revision, preview |
| Diagram | An Excalidraw drawing | path, title, linked pages and tasks |
| Branch / Worktree | Git state shown, not owned | name, path, ahead/behind, dirty, last commit, linked task |
| Terminal session | A shell running in the backend | id, project, working directory, started by, status, saved transcript path |
| Saved command | A one-click project command | label, command, working directory, shown in palette |
| Activity | An append-only event | time, actor (human / agent id), verb, target, summary |
| Provenance | Who and what produced an artifact | actor, model, session or tool, timestamp, source task |

Rules:

- Statuses are a fixed small set in MVP: `backlog`, `ready`, `in-progress`, `review`, `done`, `blocked`. Columns can rename them, not add new semantics.
- Every write records the actor in Activity and, for agent writes, Provenance. The UI always shows who changed something: human or which agent.
- Links are typed and bidirectional in the index (task ↔ branch, task ↔ page, page ↔ decision), even though they live in one file's frontmatter.
- The task schema is a **shared contract with Eter** (dispatch). It is versioned like any API.

## 4. Workspace structure

Files in git are the source of truth; the SQLite index under `.worldtree/` is a cache that can be deleted and rebuilt at any time. If World Tree disappears, every task, page and decision is still readable Markdown.

```
~/WorldTree/                     # workspace (registry + templates)
├── workspace.yaml               # project paths, editor, shell, theme
└── templates/
    ├── software/template.yaml   # built-in (read-only copy)
    └── my-firmware/             # user template + starter files

siren-panel/                     # one project folder = its own git repo (always)
├── project.md                   # frontmatter: id, key, template, modules, status, repos; body: overview
├── sections.yaml
├── commands.yaml                # saved terminal commands
├── tasks/
│   └── SP-014-choose-display.md
├── wiki/
├── decisions/
├── diagrams/
├── research/                    # Research module
│   ├── sources/                 # one file per source with notes
│   ├── findings.md
│   └── questions.md
├── hardware/                    # Hardware module
│   ├── bom.yaml                 # parts, suppliers, status
│   ├── build-log/2026-10-09.md
│   ├── design/                  # KiCad, STL, STEP
│   └── assets/                  # photos, video (Git LFS or excluded)
├── log/
└── .worldtree/                  # index.sqlite, UI state (gitignored)
```

**Task file**

```markdown
---
id: WT-002
title: Define task schema v1
status: in-progress
section: contracts
priority: high
assignee: claude-code
labels: [contract, eter]
links:
  branch: feat/task-schema
  worktree: ../alrest-wt/task-schema
  pages: [wiki/telemetry-contract.md]
  decisions: [0001]
created: 2026-10-06
---

## Goal
A versioned JSON Schema for tasks shared by World Tree and Eter.

## Checklist
- [x] Draft fields
- [ ] Validate against existing task files
```

Formats above are illustrative; exact schemas are defined in planning and validated with JSON Schema. Agent-maintained wiki pages follow the LLM Wiki convention (index page plus linked topic pages) so Hermes's built-in wiki skill can work on them unchanged.

## 5. Projects, templates and modules

Every project is a core plus a set of modules switched on per project; a template is just a saved starting set of modules, sections, labels and starter files. Creating a project is the first thing a user does, so the New Project wizard is a top-priority screen.

**Core (always on, cannot be disabled)**

- Project overview, tasks and board, activity, search, inspector with provenance.
- Project versioning: the project folder is always a git repository (see below).

**Module catalog**

Modules are a fixed, built-in catalog with on/off flags per project. This is not a third-party plugin API: no external code is loaded, so decision 8 (no plugin framework in MVP) still holds.

| Module | What it adds | Phase |
| --- | --- | --- |
| Wiki | Markdown pages, backlinks, agent-maintained LLM Wiki | P1 |
| Decisions | ADR list and template | P1 |
| Code repos | Attach code repositories; branches, worktrees, commits, read-only files, diffs; powered by Affinity Tree | P1 |
| Terminal | Integrated terminal scoped to the project and its worktrees (section 8) | P1 basic, P2 full |
| Codegraph | Embedded codebase-memory-mcp graph of attached repos | P2 |
| Diagrams | Excalidraw canvases linked to tasks and pages | P2 |
| Research | Sources library, notes, findings, open questions, promote to wiki | P2 |
| Hardware | Bill of materials, suppliers, build log, media gallery, design files | P2 |
| Agents | Project-scoped view of Eter dispatches and Aegis Eye agent state (provisional, section 9) | P3 |
| Benchmarks | Links to Resonance experiments run on this project | P3 |

**Built-in templates**

| Template | Modules on by default | Default sections | Example |
| --- | --- | --- | --- |
| Software | Wiki, Decisions, Code repos, Terminal, Codegraph, Diagrams | Backlog, Architecture, Features, Bugs | Eter Project |
| Research | Wiki, Research, Decisions, Diagrams | Questions, Sources, Experiments, Findings | Local LLM serving |
| Hardware | Hardware, Wiki, Diagrams, Decisions | Design, Parts, Build, Testing | Siren Panel enclosure |
| Mixed | Union of the kinds picked in the wizard (e.g. Software + Hardware) | Merged, deduplicated | Siren Panel (firmware + device) |
| Custom | Nothing beyond core; the user picks every module | One "General" section | Anything else |

A template only seeds a project. After creation the project owns its module list; changing a template later never changes existing projects.

**Git: always on for versioning, optional for code**

Two different things share the word "git" and should be separated in the UI:

1. **Project versioning (core, always on).** The project folder is a git repo, local-only by default, no remote required. World Tree commits automatically in the background (batched, with the actor in the commit message). This is what gives history, diffs of wiki pages, provenance and undo; files-as-source-of-truth depends on it. The user never has to think about it. World Tree never runs git itself: every git operation goes through a GitPort interface implemented by Affinity Tree, the ecosystem's Git client.
2. **Code repos (module, optional).** Attaching one or more code repositories to see branches, worktrees and diffs. A research project usually has none; a mixed project may have several.

Recommendation: do not offer "no git" for project versioning. The only real cost is large binaries in hardware projects (CAD, photos, video); handle them with an `assets/` folder tracked by Git LFS or excluded with a pointer file, chosen in the Hardware module setup.

**New Project wizard (5 steps)**

1. **Start**: choose a template card (Software, Research, Hardware, Mixed, Custom, plus user templates), or "Import existing folder" (detects `.git`, `package.json`, `*.csproj`, `pom.xml`, KiCad or STL files and suggests a template).
2. **Identity**: name, short key (e.g. `WT`, used for task ids), glyph and color, description, location (new folder, existing folder, or inside an existing code repo).
3. **Modules**: checklist pre-filled from the template; Custom starts empty. Each module shows a one-line description and its phase; unavailable ones show "coming later".
4. **Setup**: only for enabled modules: attach code repos, optional git remote for project versioning, index codegraph now or later, terminal default shell and working directory, hardware asset strategy (LFS or excluded), research citation style.
5. **Review**: summary plus a preview of the folder tree and files that will be created; "Create project" opens the new project overview.

**Project settings**

- Turn modules on or off at any time. Turning one off hides it; its files stay on disk untouched.
- Edit identity, sections, labels, board columns, git remote, linked repos.
- "Save as template" creates a user template from this project's current setup.

**Template manager**

- List built-in and user templates; built-ins are read-only but can be duplicated.
- Template editor: name, glyph, description, modules, default sections, board column names, labels, starter pages (README, wiki index), starter tasks, folder layout, and setup defaults per module.
- Live preview of the folder tree a project created from it would get.
- Templates are files in the workspace (`templates/<id>/template.yaml` plus starter files), versioned in git and shareable.

```yaml
# templates/software/template.yaml
id: software
version: 1
name: Software
glyph: code
modules: [wiki, decisions, code-repos, terminal, codegraph, diagrams]
sections: [Backlog, Architecture, Features, Bugs]
labels: [bug, feature, chore, spike]
starter:
  pages: [README.md, wiki/index.md]
  tasks: ["Define MVP scope", "Set up repo"]
setup:
  terminal: { shell: zsh, cwd: repo-root }
```

## 6. Information architecture and navigation

The app is one persistent shell with six regions; navigation changes the center, never the frame. This is the single most important layout to get right in the mockups, because every screen lives inside it.

**Shell regions**

| Region | Contents | Behavior |
| --- | --- | --- |
| Activity bar (far left, \~48 px) | Top: Home, Projects, Search, Activity. Middle (per current project, only enabled modules): Board, Wiki, Decisions, Code, Research, Hardware, Diagrams. Bottom group "Ecosystem": Agents, Monitor, Benchmarks, Deck. Last: Settings | Always visible; module icons change with the project |
| Sidebar (\~260 px, resizable, collapsible) | Project switcher on top, then the project tree: sections → tasks / pages / repos / sources / parts / diagrams | Filterable; drag to reorder sections and move tasks |
| Main area | Tabbed views (task, page, file, board, graph, diagram, BOM, ecosystem screens) | Multiple tabs, split into two panes side by side |
| Bottom panel (collapsible, \~240 px) | Integrated terminal sessions with saved-command chips; later problems and output tabs | Ctrl+\` toggles; sessions persist across tabs |
| Inspector (right, \~320 px, collapsible) | Context of the selected item: metadata, links, provenance, activity | Follows the selection; empty state when nothing is selected |
| Status bar (bottom) | Current project, branch, index status, MCP server status, agents connected, Aegis Eye strip | Glance-only; click opens details |

**Navigation hierarchy**

- Workspace → Project → Section → Task / Page
- Project → Repo → Branch / Worktree → File
- Cross-cutting: Search, Activity feed, Decisions, Command palette (Cmd+K)

**Global interactions**

- Command palette (Cmd+K): jump to any item, run any action (new task, move task, open file, switch project).
- Quick capture (Cmd+Shift+N): a small overlay to add a task or note to any project without leaving the current view.
- Every list and board is fully keyboard navigable; shortcuts are shown in tooltips and in the palette.
- Deep links: every item has a stable URL (`/p/alrest/t/WT-002`) so agents, Siren Panel and notes can point at it.

&#91;embedded content: app shell wireframe · 6 regions + overlays\]

The board in pane A and a task in pane B is the default working layout; the inspector follows whatever is selected in either pane.

**Compact layout (phones and tablets, P3)**

- Phones: the activity bar becomes a bottom tab bar (Home, Board, Capture, Agents, More); the sidebar becomes a project-switcher sheet; the inspector becomes a bottom sheet; tabs and split panes collapse into one navigation stack; no bottom panel (no terminal).
- Tablets: keep the sidebar and two panes (board + task); the inspector opens on demand.
- Every desktop component in the design system needs a compact variant; components are built responsive from P1 even though the mobile app ships in P3.

## 7. Functional requirements

Requirements are grouped by module and tagged with the phase they land in (P1 = MVP, P2, P3; see Phased scope). Mockups cover P1 in detail and P2 at sketch level.

**Workspace and projects**

- FR-WS-1 (P1): Register an existing folder as a project (import) or create a new one through the New Project wizard.
- FR-WS-2 (P1): Project overview: description, status, enabled modules, sections, task counts by status, recent activity, linked repos.
- FR-WS-3 (P1): Watch files on disk; external changes (agent writes, git pull) appear without reload, with the actor shown.
- FR-WS-4 (P1): Archive and unarchive projects; project switcher in the sidebar.
- FR-WS-5 (P1): Background auto-commit of project changes with the actor in the commit message.

**Sections and tasks**

- FR-TK-1 (P1): Create, edit, move and close tasks; edit the body as Markdown with live preview.
- FR-TK-2 (P1): Task detail: title, status, section, priority, assignee, labels, checklist, typed links, activity and provenance.
- FR-TK-3 (P1): Link a task to a branch or worktree; show ahead/behind, dirty state and last commit.
- FR-TK-4 (P1): List view with grouping (section, status, assignee), sorting and saved filters.
- FR-TK-5 (P2): Dependencies (blocks / blocked by) with a warning when a task moves ahead of its blocker.

**Board**

- FR-BD-1 (P1): Kanban per project and across projects, columns mapped to statuses, drag and drop and keyboard moves.
- FR-BD-2 (P1): Card shows title, id, section, assignee (human or agent glyph), labels, checklist progress, branch chip, "needs review" signal.
- FR-BD-3 (P1): Swimlanes by section or assignee; WIP limit per column (visual warning only).

**Wiki, specs and decisions**

- FR-WK-1 (P1): Browse and read Markdown pages with backlinks, outline and Mermaid rendering.
- FR-WK-2 (P1): Edit pages in a Markdown editor (text, not code: this is not an IDE).
- FR-WK-3 (P1): Decisions list and ADR template; status chips; "applies to" links.
- FR-WK-4 (P2): Page history from git with a diff view and the actor per change.

**Code and git (read-only)**

- FR-CD-1 (P1): Repo view: branches, worktrees, recent commits; file tree; read-only file viewer with syntax highlighting.
- FR-CD-2 (P1): Diff view for a branch or worktree against its base (summary + per-file).
- FR-CD-3 (P1): "Open in editor" and "Open in terminal" on any file, repo or worktree.
- FR-CD-4 (P2): Create a worktree for a task from the UI (git CLI); remove a worktree after merge.
- FR-CD-5 (P2): Embedded codegraph view (codebase-memory-mcp UI) scoped to the repo, plus "show in graph" from a file.

**Diagrams**

- FR-DG-1 (P2): Open and edit `.excalidraw` files in an embedded Excalidraw canvas; link diagrams to tasks and pages.

**Templates and modules**

- FR-TP-1 (P1): New Project wizard: template, identity, modules, module setup, review with folder preview.
- FR-TP-2 (P1): Built-in templates Software, Research, Hardware, Mixed, Custom; Mixed asks which kinds to combine.
- FR-TP-3 (P1): Project settings: enable or disable modules at any time; disabling hides the module, never deletes files.
- FR-TP-4 (P1): "Save as template" from a project.
- FR-TP-5 (P2): Template manager and editor with live folder preview; duplicate built-ins; import and export templates as folders.
- FR-TP-6 (P2): Import wizard detects stack files and suggests a template.

**Research module**

- FR-RS-1 (P2): Sources library with type, reading status, tags; add by URL, file or DOI.
- FR-RS-2 (P2): Source detail: metadata, notes, highlights, linked findings and tasks.
- FR-RS-3 (P2): Findings with confidence and supporting / contradicting sources; open questions with status.
- FR-RS-4 (P2): Promote a finding or synthesis to a wiki page, keeping source links.
- FR-RS-5 (P3): Citation export (BibTeX, Markdown references).

**Hardware module**

- FR-HW-1 (P2): Bill of materials table: part, part number, quantity, supplier, unit price and currency, status, link; totals per status.
- FR-HW-2 (P2): Supplier view with availability notes (e.g. local vs import) and order tracking per part.
- FR-HW-3 (P2): Build log: dated entries with photos, linked tasks and parts.
- FR-HW-4 (P2): Design files list with type and revision; inline 3D preview for STL, file preview for PDF and images.
- FR-HW-5 (P2): Media gallery for photos and video stored under `assets/`.

**Integrated terminal**

- FR-TM-1 (P1): One terminal session per project, docked in the bottom panel, starting at the project root.
- FR-TM-2 (P2): Multiple sessions; "Open terminal here" from task, repo, worktree or file.
- FR-TM-3 (P2): Saved commands from `commands.yaml` as chips and palette actions.
- FR-TM-4 (P2): Clickable file paths, URLs and task ids in output; save transcript to the project log.

**Ecosystem (provisional)**

- FR-EC-1 (P3): Agents screen (Eter) and project-scoped Agents module.
- FR-EC-2 (P3): Agent monitor (Aegis Eye) full view and status-bar strip.
- FR-EC-3 (P3): Benchmarks screen (Resonance) with link-out.
- FR-EC-4 (P3): Control surface screen (Siren Panel).
- FR-EC-5 (P1, mockups only): every ecosystem screen has a designed "not connected" empty state.

**Search and activity**

- FR-SR-1 (P1): Global search over tasks, pages, decisions, files and branches, with type filters.
- FR-AC-1 (P1): Activity feed per project and global: who (human or agent), what, when, with links; filter by actor.
- FR-AC-2 (P1): "Since last visit" digest on Home.

**MCP server and agents**

- FR-MC-1 (P1): MCP tools: list/get/create/update tasks, move status, link branch, list/get/write pages, add decision, search, get project overview.
- FR-MC-2 (P1): Connected clients panel: which agents are connected, last call, write permissions.
- FR-MC-3 (P1): Per-client write scope (read-only, tasks only, tasks + wiki), set in Settings.
- FR-MC-4 (P3): Event stream for Eter and Siren Panel (task created, moved, review requested).

## 8. Integrated terminal (codename TBD)

A real shell inside World Tree, scoped to the current project or worktree, so project commands (build, test, git, launching a Claude Code session) never require switching apps. It is a reusable component hosted by World Tree; Siren Panel may host a compact version later.

**Behavior**

- Dock at the bottom of the main area (resizable, collapsible, Ctrl+\`), or open as a full tab.
- Multiple sessions per project, each labeled with its working directory: project root, a code repo, or a task's worktree.
- "Open terminal here" from a task, repo, worktree or file opens a session in the right folder with the project environment loaded.
- Saved commands per project (`commands.yaml`): one-click `pnpm test`, `dotnet build`, `claude` in the worktree. Shown as chips above the terminal and in the command palette.
- Sessions survive switching tabs and projects; they end when closed or when the backend restarts (marked "ended").
- Output is not stored by default; "Save transcript" writes it to the project log with the actor and time.
- Links in output (file paths with line numbers, URLs, task ids like `WT-002`) are clickable and open in World Tree.

**Boundaries**

- The terminal runs what the user types or clicks. Starting agents automatically, queueing them or routing work stays in Eter.
- Not an IDE: no debugger, no editor in the terminal panel beyond what the shell itself offers.
- Secrets come from the OS keychain or the user's own shell profile; World Tree never writes them into files or history.

**Tech (proposed)**: xterm.js in the frontend, portable-pty in the Rust core, over an authenticated local WebSocket. A shell reachable from a browser is a remote-code-execution surface, so this module makes real UI authentication a P1 requirement (section 14).

## 9. Ecosystem screens (provisional)

World Tree hosts a screen for each sibling project so the whole ecosystem is reachable from one place, but it never takes over their logic. These screens are designed now with what each project's overview says today; each carries a "Provisional · contract pending" tag and must be revisited when that project's contract is defined.

| Screen | Owner project | Shows (from current overviews) | Data source | When not connected |
| --- | --- | --- | --- | --- |
| Agents & orchestration | Eter Project | Agents and their lifecycle, task queue fed by World Tree `ready` tasks, dispatch history, model chosen per task, routing rules | Eter API/events (TBD) | Empty state: "Eter not connected", explains what appears here, link to the Eter project |
| Agent monitor | Aegis Eye | Agent cards: name, status, model, context-window usage %, current activity, needs-attention flag; compact strip in the status bar | Aegis Eye telemetry contract: Eter and Claude Code sessions (incl. tmux) | Shows only locally detected Claude Code sessions, or an empty state |
| Benchmarks | Resonance | Recent experiments, leaderboard summary per category, current baselines, link to the Affinity Chart; "Open in Resonance" | Resonance read-only results API | Empty state with a link to start Resonance |
| Control surface | Siren Panel | Device connection status, which views are mirrored on the deck, last commands received, pairing | Siren Panel client over the same HTTP/MCP API | "No device paired" with pairing instructions |

Rules:

- They live under an "Ecosystem" group in the activity bar; project-scoped versions appear as the Agents and Benchmarks modules.
- The task id is the correlation key: an agent card links to the task it works on, and a task shows the agent working on it.
- World Tree renders; it never dispatches, scores or controls hardware itself. Actions on these screens (e.g. "accept", "stop run") are sent to the owning project's API.

## 10. Screen inventory

Thirty-one screens define the full UI scope, designed in four rounds: foundation (1–7), knowledge, code and configuration (8–17), modules (18–25), ecosystem, states and mobile (26–31). Every screen sits inside the shell, supports light and dark themes, keyboard navigation, and empty, loading and error states.

| # | Screen | Phase | Fidelity | Must show |
| --- | --- | --- | --- | --- |
| 1 | Shell + Home | P1 | High | The six shell regions; Home: "since last visit" digest, items needing review, blocked tasks, in-progress by project, recent decisions, quick capture, "New project" button |
| 2 | New Project wizard | P1 | High | Five steps with a stepper: template cards (Software, Research, Hardware, Mixed, Custom, user templates, Import folder), identity, module checklist with phase badges, module setup, review with folder-tree preview |
| 3 | Project overview | P1 | High | Header (name, glyph, template badge, enabled-module chips, status, repos), sections with task counts, recent activity, linked decisions, key pages; layout adapts to modules (a hardware project shows BOM status, a research project shows open questions) |
| 4 | Board | P1 | High | Columns by status with counts and WIP warning, cards (id, title, section, actor badge, labels, checklist, branch chip, review flag), swimlanes, filters, cross-project mode |
| 5 | Task detail | P1 | High | Status, metadata, Markdown body, checklist, typed links (branch with ahead/behind/dirty, pages, decisions, sources, parts), provenance and activity in the inspector, "Open terminal in worktree" |
| 6 | Command palette + quick capture | P1 | High | Palette with grouped results and actions (including saved commands); capture overlay with project/section picker |
| 7 | Terminal panel | P1 | High | Bottom panel with session tabs labeled by folder, saved-command chips, running output with clickable paths and task ids, "Save transcript", a session opened from a task's worktree |
| 8 | Wiki page | P1 | High | Rendered Markdown, outline, backlinks, Mermaid, last-editor badge, edit toggle, history link |
| 9 | Decisions list + detail | P1 | Medium | Table with status chips; ADR detail with applies-to links |
| 10 | Repo + file viewer | P1 | High | Branches and worktrees with status, commits, file tree, read-only file with line links, "Open in editor", "Open terminal here" |
| 11 | Diff review | P1 | High | Files-changed summary, per-file diff, linked task, actions (open in editor, mark review done, send back) |
| 12 | Search results | P1 | Medium | Type filters (task, page, decision, file, branch, source, part), grouped results with snippets |
| 13 | Activity feed | P1 | Medium | Timeline, actor filter (human / each agent), project filter |
| 14 | Project settings | P1 | High | Identity, module toggles with descriptions, sections, labels, board columns, git remote and auto-commit status, linked repos, "Save as template" |
| 15 | Template manager | P2 | High | Built-in and user template cards, module chips, projects using each, duplicate / edit / export / import |
| 16 | Template editor | P2 | High | Form for modules, sections, labels, columns, starter pages and tasks, module setup defaults; live folder-tree preview |
| 17 | Workspace settings | P1 | Medium | Projects, editor and shell, MCP server status, connected clients with write scopes, theme and density, shortcuts, security (sessions) |
| 18 | Codegraph view | P2 | Medium | Embedded graph scoped to a repo, "show in graph" from a file, index status |
| 19 | Diagram canvas | P2 | Medium | Embedded Excalidraw with link panel to tasks and pages |
| 20 | Research: sources library | P2 | High | Table or cards by type, reading status, tags, add by URL / file / DOI |
| 21 | Research: source detail | P2 | Medium | Metadata, notes, highlights, linked findings and tasks |
| 22 | Research: findings and questions | P2 | Medium | Findings with confidence and evidence links; questions with status; promote to wiki |
| 23 | Hardware: bill of materials | P2 | High | Parts table with supplier, price, currency, status chips, totals per status, order links |
| 24 | Hardware: build log | P2 | Medium | Dated entries with photo strip, linked tasks and parts |
| 25 | Hardware: design files and media | P2 | Medium | File list with type and revision, STL 3D preview, photo and video gallery |
| 26 | Agents & orchestration (Eter) | P3 | Provisional | Agent list and lifecycle, queue of ready tasks, dispatch history, model per task; "Eter not connected" state |
| 27 | Agent monitor (Aegis Eye) | P3 | Provisional | Agent cards (status, model, context %, activity, needs attention) in full, compact and strip forms |
| 28 | Benchmarks (Resonance) | P3 | Provisional | Recent experiments, leaderboard summary, baselines, "Open in Resonance" |
| 29 | Control surface (Siren Panel) | P3 | Provisional | Device status, mirrored views, last commands, pairing; "No device paired" state |
| 30 | States sheet | P1 | High | First run (empty workspace), empty board, index loading, file-watcher error, MCP server down, agent write conflict, terminal session ended |
| 31 | Mobile companion set | P3 | High | Phone (390×844): pairing via QR, Home digest, board as a status-grouped list with swipe to move, task detail, diff summary with approve / send back, quick capture, agent monitor. Tablet (1180×820): board + task in two panes |

**Component inventory needed beyond the design system**

- Kanban card (full and compact), task row, branch chip (ahead/behind/dirty), actor badge (human vs named agent), provenance block, activity event row, link chip per type, status selector, project glyph and template badge, module chip and module toggle row, template card, wizard stepper, folder-tree preview, terminal session tab and saved-command chip, BOM row with status chip, source card, finding row with confidence, "provisional" tag, "not connected" empty state, diff file row, palette result row.

## 11. Key flows to prototype

Nine flows cover the jobs that matter most; a clickable prototype of flows F, A, B, C and G is enough to validate the wizard, shell, board and terminal before any code.

**A. Morning check**

1. Open World Tree → Home shows the "since last visit" digest.
2. Click an item flagged "needs review" → task detail opens in a tab, inspector shows the agent's provenance.
3. Open the linked branch diff → diff review.
4. Mark review done → task moves to `done`, Home updates.

**B. Capture and plan**

1. Cmd+Shift+N anywhere → capture overlay; pick project and section; save.
2. Open the project board → the new card sits in `backlog`.
3. Open it, write a short spec in the body, add a checklist, drag to `ready`.

**C. Start work on a task**

1. From task detail, link an existing branch or (P2) create a worktree.
2. Branch chip appears on the card and task; "Open worktree in terminal" launches the user's terminal there.
3. An agent session (outside World Tree) updates the task through MCP; the card moves to `in-progress`, actor badge shows the agent, activity feed records it.

**D. Understand code without an IDE**

1. Code view → pick repo → file tree → open a file read-only.
2. Follow a link from a task or wiki page to a specific file and line.
3. (P2) "Show in graph" opens the codegraph view centered on that file.

**E. Record a decision**

1. Cmd+K → "New decision" → ADR template opens.
2. Fill context, decision and consequences; link the affected tasks and pages.
3. Decision appears in the project overview and in each linked item's inspector.

**F. Create a project (prototype first)**

1. Home → "New project" → template cards; pick Custom.
2. Identity: name "Siren Panel", key `SP`, glyph, location "new folder".
3. Modules: check Hardware, Wiki, Diagrams, Code repos, Terminal; leave Research and Codegraph off.
4. Setup: attach the firmware repo, choose Git LFS for assets, default shell zsh.
5. Review the folder preview → Create → project overview with a BOM summary card and an empty board.
6. Project settings → "Save as template" → name it "Firmware + device".

**G. Run project commands**

1. From task SP-014, "Open terminal in worktree" → bottom panel opens a session labeled with the worktree folder.
2. Click the `pnpm test` chip; output streams; a failing file path is clickable and opens the read-only viewer at that line.
3. Type `claude` to start a Claude Code session there; its MCP updates move the task card, with the agent's actor badge.

**H. Log a hardware build session**

1. Hardware → Build log → "New entry" for today.
2. Add photos, a short summary, link parts SP-display and SP-battery; mark the display "installed" in the BOM.
3. The project overview shows the updated BOM status and the entry in recent activity.

**I. Research a question**

1. Research → Questions → "Which SBC fits the deck?"
2. Add three sources by URL; Hermes adds notes to them through MCP (agent badge on each note).
3. Record a finding with two supporting sources and confidence "medium"; promote it to a wiki page; link it to task SP-014.

## 12. Design direction

World Tree uses the shared Xeno design system in its "workspace" density, with decoration turned down: it is a tool used for hours, so data and text win over HUD effects.

**Principles**

1. Content first: beveled frames, glows and scanlines live only on chrome (activity bar, panel headers, status bar), never inside reading areas.
2. Glance, then detail: board and Home are scannable in two seconds; task and page detail carry the depth.
3. Human vs agent is always visible: every change, card and activity row shows who did it, with a distinct badge per agent. This is World Tree's signature, not an afterthought.
4. Never color alone: statuses pair color with label and glyph (WCAG AA minimum).
5. Keyboard is first-class: focus states are visible and styled, not browser defaults.

**Mapping to design-system tokens**

| World Tree concept | Token / component |
| --- | --- |
| Task status backlog / ready / in-progress / review / done / blocked | status-idle, neutral, status-running, status-waiting-attention, status-done, status-error |
| Agent actor | accent-primary (cyan) badge with agent glyph |
| Human actor | neutral badge with initials |
| Needs review | warning (amber) flag |
| Branch dirty / behind | warning / neutral chips |
| BOM part status to buy / ordered / received / installed | neutral, status-running, status-done, success |
| Module chip (enabled / coming later) | tag, default / disabled |
| Provisional screen | warning-outlined tag "Provisional · contract pending" |
| Terminal panel | console/log line component, mono type, cockpit-style header only |
| Panels, inspector | beveled panel/frame (light decoration) |
| Board card | kanban card composite |
| Project tree | project tree sidebar composite |
| Ecosystem agent cards | Aegis Eye agent card (full / compact / minimal) from the design system |

**Mockup deliverables**

- Desktop frames at 1440×900 and 1920×1080; inspector and bottom panel open and closed.
- Phone frames at 390×844 and tablet frames at 1180×820 for screen 31; each composite component shows its compact variant.
- Dark theme first (primary), light theme for the shell, wizard, board and task detail.
- Rounds 1 and 2 (screens 1–17) at the fidelity in the inventory; round 3 (18–25) high for 20 and 23, medium for the rest; round 4 (26–31) provisional ecosystem mockups with "not connected" states, plus the mobile companion set.
- A clickable prototype of flows F, A, B, C and G.
- States sheet (screen 30).
- Narrow layout check at 1280 px: sidebar, inspector and bottom panel collapse to overlays.

## 13. Architecture

World Tree is one host and many clients: a Rust core runs on the Mac and owns files, the index, git (through Affinity Tree), terminals and the MCP server; every surface is a client of its single API. The UI is Angular, shipped in Tauri 2 shells: the desktop app on macOS first, and the same UI in an iOS and Android app for phones and tablets later.

**Platforms**

| Surface | Role | When |
| --- | --- | --- |
| macOS desktop app | Host + full client: runs the core, all 30 screens | P1 |
| Windows / Linux desktop | Same build target; untested until needed | Later |
| iPhone, Android phones | Companion client of the Mac host: glance and act | P3 |
| iPad, Android tablets | Companion client with a two-pane layout; closer to desktop but still no host | P3 |
| Siren Panel, future watch | Thin clients of the same API (Aegis Eye views) | P3+ |

Phones and tablets never hold the projects and never run the core: git worktrees, shells, codebase indexing and agent sessions need the Mac. They connect to the Mac host over the local network or a private overlay network (Tailscale or WireGuard) after pairing with a QR code.

**Language decision**

| Option | For | Against | Verdict |
| --- | --- | --- | --- |
| Rust core + Tauri 2 | Affinity Tree is Rust, so GitPort is an in-process crate with no bridge; one shell for macOS, Windows, Linux, iOS and Android; small single binary; mature crates for PTY, file watching, SQLite and HTTP; official MCP Rust SDK | Steeper learning curve than your Angular/.NET/Java stack; slower builds; agent-written Rust needs careful review | **Chosen** |
| Go | Simple, fast to learn, strong for servers | No first-class mobile shell; a second systems language next to Affinity Tree's Rust, reached through FFI | Rejected |
| TypeScript host (NestJS) | Fastest for you; same as Resonance | Needs Electron or a Node sidecar to ship; napi bridge to Affinity Tree; a Node runtime in every install | Rejected for the host; TypeScript stays for the UI |
| .NET (MAUI / Blazor Hybrid) | Your strongest backend language; targets macOS, iOS, Android | Poor fit for the web embeds (Excalidraw, xterm.js, CodeMirror); FFI to Rust; heavier runtime | Rejected |
| Flutter / Kotlin Multiplatform | Native-feeling mobile | Throws away the web design system and every web embed | Rejected |

**Components**

| Component | Responsibility | Technology |
| --- | --- | --- |
| UI | All screens, desktop (workspace) and compact (phone) layouts | Angular (standalone, signals) + CDK; Alrest UI design system tokens as CSS custom properties; CodeMirror read-only for code; xterm.js; Mermaid; Excalidraw as a React island |
| Desktop shell | Window, menus, deep links, OS keychain, auto-update | Tauri 2 |
| Mobile shell | iOS and Android app, secure storage of the device token, share sheet into quick capture | Tauri 2 mobile, same Angular build |
| Core (host) | Domain services: projects, templates, tasks, pages, research, hardware, activity, provenance | Rust, tokio |
| API | REST, Server-Sent Events and WebSocket for every UI client, desktop included | axum on 127.0.0.1; a separate TLS listener only when a device is paired |
| MCP server | Agent-facing tools over the same services | Official MCP Rust SDK (rmcp); stdio and streamable HTTP |
| File store | Markdown + YAML read/write, JSON Schema validation, file watching | serde, notify |
| Index | Queries, backlinks, full-text search; rebuildable from files | SQLite with FTS5 (rusqlite) |
| GitPort | Project versioning, auto-commit, code repos, worktrees, diffs, LFS | Affinity Tree crate in-process; a git CLI adapter until it ships |
| Terminal | Sessions per project or worktree | portable-pty, streamed over WebSocket |
| Codegraph (P2) | Graph of attached repos | codebase-memory-mcp as a sidecar process, its UI embedded |
| 3D preview (P2) | STL preview in the Hardware module | three.js |
| Ecosystem adapters (P3) | Read Eter, Aegis Eye, Resonance and Siren Panel data | One thin Rust client per sibling contract |

**Key design decisions**

- One API for every UI client, the desktop webview included, so the mobile app uses exactly what the desktop exercises daily. Tauri IPC only for native features (keychain, windows, deep links).
- The MCP server is the product's real API for agents; anything the UI can do, an agent can do with the right scope.
- Mobile scope is "glance and act": Home, board, task detail, diff and review reading, quick capture, agent monitor, approvals. No wizard, template editor, code browsing beyond diffs, or terminal in the first mobile release.
- Embed, don't rebuild: git (Affinity Tree), codegraph, diagrams and code viewing come from existing pieces.
- Shared with Resonance: Angular design-system components and JSON Schema contracts, not backend code.

&#91;embedded content: component architecture · clients, backend, data\]

Every client, desktop, phone or agent, enters through one of two faces of the same Rust core on the Mac; only the domain services touch files, and only GitPort touches git.

## 14. Security and local-first

World Tree never sends project data off the machine; the only external traffic is what the user starts (opening a link, an agent the user runs with a cloud model).

- Bind to `127.0.0.1` only; validate `Host` and `Origin` to block DNS rebinding and cross-site requests.
- UI auth: reuse the Resonance pattern (single local account, JWT in HttpOnly cookies). Real auth is required in P1 because the integrated terminal exposes a full shell to anything that can reach the backend. Terminal WebSocket connections use the same session cookie plus Origin validation; no tokens in URLs.
- MCP clients get a per-client token and a write scope (read-only, tasks, tasks + wiki). Destructive actions (delete, archive, remove worktree) are never exposed to agents in P1.
- Agent writes go through schema validation; a malformed write is rejected with a clear error, never written partially.
- Conflicts: if a file changed on disk since the client read it, the write is rejected (version check), and the UI shows a conflict state.
- Content from files, wiki pages and research sources is data, never instructions; agents reading it through MCP must treat it as untrusted.
- Strict Content Security Policy; Markdown rendering is sanitized.
- Supply chain: same pnpm rules as Resonance (frozen lockfile, blocked lifecycle scripts, minimum release age).

**Remote devices (phones, tablets, Siren Panel)**

- Off by default. Enabling it starts a second, TLS-only listener; the localhost API is unchanged.
- Pairing by QR code shown on the Mac: each device gets its own token, name and scope (read-only, or act: move tasks, approve reviews, capture). Revocable from Settings, with last-seen time.
- Outside the home network, reach the Mac through a private overlay network (Tailscale or WireGuard); never open a port to the internet.
- No terminal and no destructive actions from mobile in the first release.
- The mobile app stores only its device token (iOS Keychain, Android Keystore) and an optional read-only cache; project files stay on the Mac.

## 15. Non-functional requirements

- NFR-1 Data ownership: deleting `.worldtree/` loses nothing but UI state; the index rebuilds from files.
- NFR-2 Responsiveness: a file change on disk appears in the UI within 1 second; board drag and drop feels instant (optimistic update, reconciled on save).
- NFR-3 Scale: comfortable with 20 projects, 5k tasks, 2k pages, 1k BOM parts and repos of 100k files (code view lazy-loads).
- NFR-4 Offline: everything works without internet.
- NFR-5 Platforms: macOS desktop first (P1); iOS and Android companion apps for phones and tablets (P3); Windows and Linux desktop builds come from the same Tauri project, untested until needed.
- NFR-6 Git-friendly files: stable key order and formatting in frontmatter so diffs stay small and readable; auto-commits batched so history stays readable.
- NFR-7 Accessibility: keyboard navigable, visible focus, WCAG AA contrast in both themes.
- NFR-8 Extensibility: new modules, link types and templates added without changing core flows.
- NFR-9 Terminal: keystroke echo under 50 ms locally; a crashed session never affects the backend or other sessions.
- NFR-10 Graceful absence: every ecosystem screen and optional module works, or shows a designed empty state, when its dependency is missing.

## 16. Phased scope

Build the file store, index and MCP server before UI polish: if agents cannot read and write tasks reliably, no screen matters.

&#91;embedded content: roadmap · 4 phases, 4 gates\]

P1 exit criterion in one line: the Alrest ecosystem itself (this spec, Eter, Aegis Eye, Resonance tasks) is managed in World Tree for two weeks without falling back to another tracker.

## 17. Decisions and open questions

None of these are confirmed yet; they come from the September landscape research and this spec, and need an explicit yes before planning.

| # | Topic | Proposed decision |
| --- | --- | --- |
| 1 | Source of truth | Files in git (Markdown + YAML frontmatter); SQLite is a rebuildable index |
| 2 | Core API | World Tree MCP server; the UI is one client of the same services |
| 3 | Scope boundary | Viewer, not IDE: no code editor; "Open in editor" instead |
| 4 | Agents | World Tree does not run agents; dispatch belongs to Eter |
| 5 | Codegraph | Embed codebase-memory-mcp instead of building one |
| 6 | Diagrams | Embed Excalidraw (React island) |
| 7 | Stack | Rust core (tokio, axum, rmcp) + Angular UI in Tauri 2 shells; Cargo for the core, pnpm for the UI |
| 8 | Extensibility | Modules are a fixed built-in catalog toggled per project; no third-party plugin API in P1–P2 |
| 9 | Visual language | Shared Xeno design system, workspace density, low decoration |
| 10 | Templates | Built-in Software, Research, Hardware, Mixed, Custom; templates only seed a project and are files in the workspace |
| 11 | Git | Project folder is always a git repo (local-only by default, background auto-commit); code repos are an optional module; all git operations go through Affinity Tree via GitPort |
| 12 | Terminal | Integrated terminal is a World Tree module (P1 basic); it runs what the user types and never dispatches agents |
| 13 | Ecosystem screens | World Tree hosts provisional screens for Eter, Aegis Eye, Resonance and Siren Panel; logic stays in each project |
| 14 | UI auth | Real local authentication in P1, same pattern as Resonance, implemented in the Rust core, because of the terminal |
| 15 | Platforms | macOS desktop first; iPhone, Android and tablets later as companion clients of the Mac host via QR pairing; the core never runs on mobile |

- [ ] Phase 0: run the real projects for 2–4 weeks in an Obsidian vault (Kanban, Excalidraw, Git plugins) + codebase-memory-mcp + Hermes, and log friction. Mockups proceed in parallel; the friction log decides MVP cuts.
- [ ] Task schema v1 and its versioning — shared contract with Eter.
- [ ] GitPort contract with Affinity Tree as a Rust trait in a shared crate (operations, error model, events); whether World Tree also embeds Affinity Tree's UI.
- [ ] Where project folders live: inside each code repo, in one central repo, or both supported.
- [ ] Mobile reach: local network only at first, or Tailscale from day one; whether the phone keeps an offline read-only cache.
- [ ] How agent identity is declared over MCP (client name, per-client token, model name for provenance).
- [ ] Terminal codename (Xenoblade naming) and whether it becomes a reusable crate shared with Siren Panel.
- [ ] Auto-commit cadence (per change, batched every N minutes, or on idle) and whether to offer an opt-out per project.
- [ ] Hardware assets: Git LFS by default vs excluded folder with pointer files.
- [ ] Mixed template: fixed combinations or free pick of kinds in the wizard.
- [ ] Ecosystem screen contracts: Eter API, Aegis Eye telemetry, Resonance results API, Siren Panel pairing.
- [ ] Rust ownership plan: which core parts you write or rework by hand versus review only, so you can maintain the core without an agent.

## 18. Ecosystem fit (Alrest)

World Tree is the workspace and owns the project data model; it consumes the other components through their contracts and never absorbs their responsibilities.

| Component | Relationship | Shared contract |
| --- | --- | --- |
| Eter Project | Reads ready tasks, dispatches them to agents, writes results back through MCP. World Tree hosts its provisional Agents screen and never orchestrates. | Task schema v1, task events |
| Aegis Eye | Embedded in the status bar strip, the Agent monitor screen and task detail to show live agent state. | Aegis Eye telemetry contract, task id as a correlation key |
| Siren Panel | Thin client: glance at the board, accept a review, move a task; World Tree shows its device status screen. | Same HTTP/MCP API and event stream |
| Resonance | Benchmark projects can be World Tree projects; its Benchmarks screen links to reports. | Read-only results API; shared Angular design-system components |
| Integrated terminal (codename TBD) | New piece hosted by World Tree as a module; may be reused by Siren Panel. | Terminal session API (WebSocket), saved-commands file |
| Hermes Agent | Primary agent client today: research, wiki upkeep, task updates. | MCP tool set, LLM Wiki convention |
| Affinity Tree | Owns the Git domain (Fork-style client, Rust core). World Tree embeds it for project versioning, code repos, branches, worktrees and diffs, and never reimplements git. | GitPort interface (status, log, diff, branch, worktree, commit, LFS); linked in-process as a Rust crate |

Cross-project impact to track: the task id must appear in Aegis Eye events so World Tree can show which agent is on which task without a second schema.

## 19. Spec Kit handoff

This document is the source material for Spec Kit 1.0's SDD flow, not a spec to paste whole. The constitution comes from the principles and decisions, each feature below becomes one `/speckit.specify` run carrying only the what and why, and the technical sections feed `/speckit.plan`. Tasks are executed with superpowers subagents instead of `/speckit.implement`, and `/speckit.converge` closes each feature.

**Pipeline**

| Step | Tool | Input from this spec | Output |
| --- | --- | --- | --- |
| 1. Init (once) | `uv tool install specify-cli`, then `specify init` with the Claude Code integration key | — | `.specify/` and the `/speckit.*` commands |
| 2. Constitution (once) | `/speckit.constitution` | The draft below (from §1, §12, §14, §17) | Project constitution in `.specify/memory/` |
| 3. Specify (per feature) | `/speckit.specify` | Goals, jobs, the feature's FRs, screens and flows; no stack | The feature's `spec.md` |
| 4. Clarify | `/speckit.clarify` | The §17 open questions that touch the feature | Answers written back into `spec.md` |
| 5. Plan | `/speckit.plan` | §4 formats, §13 architecture, §14 security, §17 decisions | `plan.md` and design artifacts |
| 6. Quality gates | `/speckit.checklist`, `/speckit.tasks`, `/speckit.analyze` | — | `tasks.md`, clean analysis report |
| 7. Execute | superpowers, subagent-driven execution | `plan.md` + `tasks.md` as the plan | Code on the feature branch |
| 8. Converge | `/speckit.converge` | — | "Converged", or new tasks appended → back to step 7 |

**Constitution draft**

```text
/speckit.constitution World Tree principles:
1. Local-first and private: no project data leaves the machine unless the user sends it; the core binds to localhost, and other devices connect only after explicit pairing over an encrypted channel.
2. Files are the source of truth: Markdown + YAML in git; any database is a rebuildable cache.
3. One API for humans and agents: every UI action has an MCP equivalent through the same domain services and rules.
4. Provenance always: every write records its actor (human or named agent) and the UI always shows it.
5. Respect boundaries: never orchestrate agents (Eter), never implement git (Affinity Tree via GitPort), never build a codegraph or diagram engine; embed instead.
6. Viewer, not IDE: code is read-only; editing goes to the user's editor or to an agent.
7. Contracts first: shared schemas (task schema, GitPort, telemetry) are versioned and validated with JSON Schema before code depends on them.
8. Test-first: every requirement has acceptance criteria; TDD for domain services; contract tests for MCP tools and GitPort; Playwright end-to-end tests for P1 flows.
9. Secure by default: real local authentication, Origin checks, per-client MCP write scopes; no terminal session without an authenticated user.
10. Accessible and keyboard-first: WCAG AA, visible focus, every action reachable by keyboard.
11. Simplicity: no third-party plugin API, no abstraction before its third use, minimal dependencies with a written justification for each new one.
```

**Feature breakdown (spec order)**

| # | Feature | Phase | Covers | Depends on |
| --- | --- | --- | --- | --- |
| 001 | project-store (files, schemas incl. task schema v1, watcher, index, GitPort + interim adapter, auto-commit) | P1 | §3, §4, FR-WS-3, FR-WS-5 | — |
| 002 | mcp-server (tools, client identity, write scopes) | P1 | FR-MC-1 to 3 | 001 |
| 003 | local-auth (account, sessions, Origin checks) | P1 | §14 | — |
| 004 | app-shell (Tauri desktop app, six regions, responsive components, Home, palette, quick capture, states) | P1 | §6, screens 1, 6, 30 | 003 |
| 005 | project-creation (wizard, built-in templates, module flags, project settings, save as template) | P1 | §5, FR-TP-1 to 4, screens 2, 14 | 001, 004 |
| 006 | tasks-and-board | P1 | FR-TK-1 to 4, FR-BD, screens 3–5 | 001, 004 |
| 007 | wiki-and-decisions | P1 | FR-WK-1 to 3, screens 8–9 | 001, 004 |
| 008 | code-views (repos, files, diffs through GitPort) | P1 | FR-CD-1 to 3, screens 10–11 | 001, 004 |
| 009 | terminal-basic | P1 | §8, FR-TM-1, screen 7 | 003, 004 |
| 010 | search-and-activity | P1 | FR-SR-1, FR-AC, screens 12–13 | 001, 004 |
| 011 | template-manager | P2 | FR-TP-5, FR-TP-6, screens 15–16 | 005 |
| 012 | research-module | P2 | FR-RS, screens 20–22 | 005 |
| 013 | hardware-module | P2 | FR-HW, screens 23–25 | 005 |
| 014 | codegraph-embed | P2 | FR-CD-5, screen 18 | 008 |
| 015 | diagrams-embed | P2 | FR-DG-1, screen 19 | 004 |
| 016 | terminal-full | P2 | FR-TM-2 to 4 | 009 |
| 017 | worktrees-and-history | P2 | FR-CD-4, FR-WK-4, FR-TK-5 | 008 (Affinity Tree ready) |
| 018+ | ecosystem screens, one per sibling | P3 | §9, FR-EC, screens 26–29 | That sibling's contract |
| 019 | mobile-companion (device pairing, compact layout, iOS and Android builds) | P3 | §13 platforms, §14 remote devices, screen 31 | 002, 003, 004, 006 |

**Rules for each `/speckit.specify` input**

- What and why only: the feature's goal, the jobs it serves (§2), its FRs, the screens and flows it covers, and the acceptance criteria. Stack, libraries and file formats stay out unless the user sees them.
- Attach the exported mockups for its screens as reference.
- One feature per run, small enough to review in one sitting (roughly 15 FRs or fewer).

**Execution with superpowers**

- Skip superpowers' brainstorming and plan-writing for these features: Spec Kit already produced spec, plan and tasks, and running both creates two competing plans.
- Run `/speckit.analyze` until clean, then hand `plan.md` + `tasks.md` to superpowers execution with review checkpoints per phase (Setup, Foundational, one per user story, Polish).
- Each subagent gets the constitution, `spec.md` and `plan.md`, not this whole document, for context hygiene. Tasks marked parallel can run in separate worktrees.
- When execution ends, run `/speckit.converge`; appended tasks go back through superpowers until it reports converged.

Sources: [Spec Kit repository](https://github.com/github/spec-kit) · [Agentic SDD command reference](https://github.github.io/spec-kit/reference/agentic-sdd.html)
