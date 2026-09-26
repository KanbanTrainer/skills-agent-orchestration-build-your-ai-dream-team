# Project Pulse Dashboard — Implementation Plan

## Summary

Mona needs a small static "Project Pulse" dashboard so contributors can see, at a glance, which projects are active, who owns them, current status, recent activity, and priority. The deliverable is exactly four files:

- `app/index.html`
- `app/styles.css`
- `app/project-data.json`
- `.vscode/launch.json`

Work is split along the agent boundaries defined in `.github/agents/`:

- **Coder** (per `coder.agent.md`) owns HTML structure/logic, JSON sample data, and the VS Code launch config. Coder must produce strict JSON (no comments) for `.vscode/launch.json`, set `cwd` to `${workspaceFolder}/app`, and open `index.html` (never a directory listing), with deterministic naming.
- **Designer** (per `designer.agent.md` "Project Pulse design expectations") owns `styles.css`: polished dashboard, project cards, status badges, priority treatment, responsive layout, and the deterministic CSS hooks `.dashboard` and `.project-card`, using rounded corners, shadows, contrast, and clear typography.

Neither agent crosses into the other's files — the agent rules explicitly forbid it ("Stay within the files assigned by the Orchestrator", "Do not change design-only files unless explicitly assigned"). The Planner (this document) does not write code. The learner performs all git actions via Copilot CLI.

The brief mandates a top-level `projects` array with fields `name`, `owner`, `status`, `recentActivity`, `priority`, and a VS Code launch configuration named **Run Project Pulse Dashboard**.

## Ordered Implementation Steps

1. **Lock the shared contract** — the data schema and the CSS hook contract. Define the JSON shape (top-level `projects` array; each item has `name`, `owner`, `status`, `recentActivity`, `priority`) and the fixed enums for `status` and `priority`. Publish the class-hook list Designer will style: `.dashboard`, `.project-card`, `.project-card__header`, `.project-name`, `.project-owner`, `.status-badge`, `.status-badge--<status>`, `.priority`, `.priority--<priority>`, `.project-activity`, `.empty-state`, `.error-state`.
2. **Author sample data** in `app/project-data.json` conforming to the schema — 4–6 realistic projects covering every status and priority enum value, including at least one deliberately long project name and one long `recentActivity` string, to exercise wrapping/truncation.
3. **Scaffold `app/index.html`** — semantic HTML5, `<html lang="en">`, `<title>Project Pulse</title>`, single `<h1>`, a `<main class="dashboard">` container, `<link rel="stylesheet" href="./styles.css">`, and a small inline script that `fetch('./project-data.json')`s, iterates `data.projects`, and renders one `<article class="project-card">` per project using the agreed hooks. Uses `textContent` (not `innerHTML`) for injected data. Renders `.empty-state` when the list is empty and `.error-state` on fetch failure.
4. **Style `app/styles.css`** (Designer) using only the published hooks — responsive grid for `.dashboard`, elevated `.project-card` with `border-radius` and `box-shadow`, color-coded `.status-badge` per status, priority emphasis, accessible typography scale, WCAG-AA contrast, focus outlines, single-column reflow on narrow viewports.
5. **Create `.vscode/launch.json`** (Coder) — strict JSON, no comments, one configuration named exactly `Run Project Pulse Dashboard`, `cwd` = `${workspaceFolder}/app`, serves the static app with `python3 -m http.server 5500`, and uses `serverReadyAction` with `uriFormat: http://localhost:%s/index.html` so the browser opens the dashboard, not a directory listing.
6. **Integration validation** — launch via VS Code Run and Debug → **Run Project Pulse Dashboard**, confirm the dashboard renders, then run the checks listed under "Validation Expectations".

## File Assignments

| File | Owner | What it must contain / do |
|---|---|---|
| `app/project-data.json` | **Coder** | Strict JSON, no comments, no trailing commas. Top-level object with a `"projects"` array. Each element: `name` (string), `owner` (string), `status` (enum: `On Track`, `At Risk`, `Blocked`, `Complete`), `recentActivity` (short human-readable string), `priority` (enum: `Low`, `Medium`, `High`, `Critical`). Include 4–6 sample projects covering all four statuses and multiple priorities, plus one long name and one long activity string. Must pass `python3 -m json.tool`. |
| `app/index.html` | **Coder** | Semantic HTML5 with `<html lang="en">`, exact `<title>Project Pulse</title>`, a single `<h1>Project Pulse</h1>`, `<link rel="stylesheet" href="./styles.css">`, and `<main class="dashboard">`. Inline `<script>` that `fetch('./project-data.json')`s, iterates `data.projects`, and appends one `<article class="project-card">` per project containing name, owner, status badge (`.status-badge .status-badge--<status-slug>`), recent activity, and priority (`.priority .priority--<priority-slug>`). Uses `textContent` for all data. Shows `.empty-state` when `projects` is empty/missing and `.error-state` on fetch failure. No frameworks, no external CDN calls. Exposes exactly the class hooks Step 1 published. |
| `app/styles.css` | **Designer** | All visual design. Must include selectors `.dashboard` and `.project-card` (deterministic hooks required by `designer.agent.md`). Must use `border-radius` and `box-shadow`. Responsive grid (e.g. `grid-template-columns: repeat(auto-fill, minmax(280px, 1fr))`) that collapses to a single column on narrow viewports. Color-coded status badges per enum value, distinct priority treatment (e.g. `Critical` visually strongest). Accessible typography scale, sufficient contrast, visible `:focus-visible` outlines. No JS, no dependence on data content beyond documented enum slugs. |
| `.vscode/launch.json` | **Coder** | Strict JSON, no comments. Contains one configuration named exactly `Run Project Pulse Dashboard`. `type: "node-terminal"` (most reliable for `serverReadyAction` with a shell command in VS Code). `cwd: "${workspaceFolder}/app"`. `command: "python3 -m http.server 5500"`. `serverReadyAction` with `pattern` matching Python's `Serving HTTP on .* port (\d+)`, `uriFormat: "http://localhost:%s/index.html"`, `action: "openExternally"`. Must pass `python3 -m json.tool`. |

## Dependencies

- Step 2 (JSON sample data) depends on Step 1 (schema + enums).
- Step 3 (HTML) depends on Step 1 for field names, enum slugs, and the class-hook contract. It does **not** need Step 2's actual data content — only the shape.
- Step 4 (CSS) depends on the class-hook contract from Step 1 (or, equivalently, Step 3's finalized hooks). It does **not** depend on Step 2 or Step 5.
- Step 5 (`launch.json`) depends only on knowing the app is served as static files from `app/` with `index.html` as entry — a fact fixed by the brief. It does not depend on Step 2, 3, or 4 content.
- Step 6 (integration validation) depends on Steps 2, 3, 4, and 5 all being complete.

## Parallel Work

Once **Step 1 (schema + hook contract)** is locked and shared:

- **Coder** can produce `app/project-data.json` (Step 2), `app/index.html` (Step 3), and `.vscode/launch.json` (Step 5) in parallel — they are non-overlapping files with no runtime dependency on each other's content, only on the Step 1 contract.
- **Designer** can produce `app/styles.css` (Step 4) in parallel with all Coder steps, because Designer only needs the hook contract from Step 1, not the HTML or JSON content.

This gives a natural fan-out: one Planner phase (Step 1), one parallel implementation phase (Steps 2–5), one sequential validation phase (Step 6).

## Sequential Work

- Step 1 must complete before Steps 2, 3, 4, or 5 begin (they all consume the shared contract).
- Steps 2, 3, 4, and 5 must all complete before Step 6.
- Agent boundaries are strictly sequential per file: Coder must not edit `styles.css`; Designer must not edit `index.html`, `project-data.json`, or `.vscode/launch.json`. If either agent believes a cross-file change is required, they must report back to the Orchestrator rather than reach across scope.

## Edge Cases

- **Empty `projects` array or missing key** — render a styled `.empty-state` message ("No projects to display yet"), never a blank page.
- **Missing optional fields** on a project (e.g. no `recentActivity`) — render a graceful placeholder (`—`) rather than the literal string `undefined`.
- **Unknown `status` or `priority` values** outside the enum — apply a neutral fallback class (`.status-badge--unknown`, `.priority--unknown`) so styling degrades gracefully instead of appearing unstyled.
- **Long project names / long recent activity strings** — CSS must wrap or ellipsize; cards must not overflow their grid cell or push horizontal scroll.
- **Many projects (10+)** — responsive grid reflows, no horizontal scroll.
- **Narrow viewports (~320px)** — single-column layout, tap targets remain legible.
- **`fetch('./project-data.json')` fails** (wrong `cwd`, typo, 404) — visible `.error-state` in the UI and a `console.error`; never a blank dashboard.
- **HTML injection via data** — always use `textContent`, never `innerHTML`, when inserting field values from JSON.
- **JSON strictness** — no comments, no trailing commas in either `project-data.json` or `.vscode/launch.json`. Both must pass `python3 -m json.tool`.
- **Launch config opens a directory listing** — must be prevented by `serverReadyAction` with `uriFormat` ending in `/index.html`. This is called out explicitly in the brief and in `coder.agent.md`.
- **Port 5500 already in use** — document in the Coder's report that the learner should stop any prior preview server before relaunching.
- **Accessibility** — one `<h1>`, `<h2>` for card names; semantic landmarks (`<main>`, `<article>`); `lang="en"`; badge contrast ≥ WCAG AA; visible `:focus-visible` outlines preserved by CSS.

## Validation Expectations

- `python3 -m json.tool app/project-data.json` succeeds. Top-level object contains a `projects` array; every element has all five required fields.
- `python3 -m json.tool .vscode/launch.json` succeeds. Contains a configuration whose `name` is exactly `Run Project Pulse Dashboard`; `cwd` is `${workspaceFolder}/app`; `uriFormat` contains `/index.html`.
- `app/index.html` contains the literal string `Project Pulse`, references `./styles.css` and `./project-data.json`, and (after JS runs) renders at least one element with class `project-card`.
- `app/styles.css` contains selectors `.dashboard` and `.project-card`, and includes `border-radius` and `box-shadow` declarations (matches the design expectations in `designer.agent.md`).
- Launching **Run Project Pulse Dashboard** from VS Code Run and Debug opens a browser to `http://localhost:5500/index.html` and shows the dashboard UI — not a directory listing.
- Manual browser check: cards render, status badges are visually distinct per status, priority is visually distinct, layout reflows to one column at ~320px, no `undefined` shown, no console errors.
- Accessibility spot-check: Tab reaches interactive elements with a visible outline; badge/text contrast passes WCAG AA; heading order is logical.
- Agent-scope check: Designer touched only `app/styles.css`; Coder touched only `app/index.html`, `app/project-data.json`, `.vscode/launch.json`.

## Open Questions

1. **Status/priority enums** — plan proposes fixed enums (`On Track`/`At Risk`/`Blocked`/`Complete` for status; `Low`/`Medium`/`High`/`Critical` for priority). Fixed enums make CSS badge styling and unknown-value fallback deterministic. Should the Orchestrator confirm these, or delegate vocabulary to Designer?
2. **`recentActivity` shape** — the brief specifies a single field. Plan assumes a human-readable string (e.g. `"Merged PR #42 · 2h ago"`) rather than a structured `{text, timestamp}` object. Confirm?
3. **Card interactivity** — plan assumes cards are informational only (no click-through, no detail view, no routing) for this iteration. Confirm?
4. **`launch.json` `type`** — plan uses `node-terminal` because it reliably supports `serverReadyAction` with an arbitrary shell command in VS Code. If the Orchestrator prefers a different mechanism (e.g. a `preLaunchTask` plus `chrome` debug type), Coder should be told before implementation.
5. **Favicon / brand mark** — not required by the brief. If Designer wants one, it would require editing `index.html`, which is out of Designer's scope; the Orchestrator would need to reassign or add a Coder follow-up.
