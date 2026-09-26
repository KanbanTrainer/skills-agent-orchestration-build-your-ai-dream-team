# Project Pulse Dashboard — Implementation Plan

## Summary

Build a small static dashboard, "Project Pulse", so Mona's contributors can see project name, owner, status, recent activity, and priority at a glance. The deliverable is four files: `app/index.html`, `app/styles.css`, `app/project-data.json`, and `.vscode/launch.json`. A launch configuration named **Run Project Pulse Dashboard** must serve the `app/` directory with `python3 -m http.server 5500` and open `http://localhost:%s/index.html` via `serverReadyAction` so learners see the dashboard UI, not a directory listing.

Work splits along agent boundaries defined in `.github/agents/`:
- **Coder** owns HTML structure/logic, JSON data, and the VS Code launch config (per `coder.agent.md` — runnable app support, strict JSON, deterministic `cwd`, opens `index.html`).
- **Designer** owns visual design in `styles.css` per `designer.agent.md` "Project Pulse design expectations" (polished dashboard, project cards, status badges, priority treatment, responsive layout, deterministic hooks `.dashboard` and `.project-card`, rounded corners, shadows, contrast, typography).

The four target files are not tracked in the template (verified by `scripts/validate-exercise.sh`) — the plan creates them fresh.

## Ordered Implementation Steps

1. **Lock the data contract.** Define the JSON schema (top-level `projects` array; each item has `name`, `owner`, `status`, `recentActivity`, `priority`) plus the finite value sets for `status` and `priority` so Designer and Coder share vocabulary.
2. **Author sample data** in `app/project-data.json` conforming to that schema (3–6 realistic sample projects covering multiple statuses and priorities, including at least one long name).
3. **Scaffold `app/index.html`** with semantic structure, exact `<title>Project Pulse</title>`, an `<h1>` heading, a `.dashboard` container, and card rendering logic (client-side `fetch('./project-data.json')` that produces one `.project-card` per project with visible `status`, `recentActivity`, and `priority`). Link `styles.css`. Include CSS hooks Designer will style: `.dashboard`, `.project-card`, `.status-badge`, `.status-badge--<status>`, `.priority`, `.priority--<priority>`, `.project-owner`, `.project-activity`, `.empty-state`.
4. **Style `app/styles.css`** (Designer) using the hooks from step 3: layout grid for `.dashboard`, card treatment with `border-radius` and `box-shadow`, status badges, priority visual weight, responsive breakpoints (single column on narrow viewports), accessible typography and color contrast, focus styles.
5. **Create `.vscode/launch.json`** (Coder) as strict JSON (no comments) with a configuration named exactly `Run Project Pulse Dashboard`, `cwd` set to `${workspaceFolder}/app`, command `python3 -m http.server 5500`, and a `serverReadyAction` pattern that opens `http://localhost:%s/index.html`.
6. **Integration validation.** Launch via VS Code Run and Debug → **Run Project Pulse Dashboard**, confirm the browser opens the dashboard (not a directory listing), and verify visuals/accessibility/console.

## File Assignments

| File | Owner | Description |
|---|---|---|
| `app/project-data.json` | Coder | Strict JSON with top-level `"projects"` array. Each object: `name` (string), `owner` (string), `status` (enum: e.g. `On Track`, `At Risk`, `Blocked`, `Complete`), `recentActivity` (short human-readable string), `priority` (enum: e.g. `Low`, `Medium`, `High`, `Critical`). 3–6 realistic sample entries; at least one long name and one entry per status/priority variant to exercise styling. No comments, no trailing commas. |
| `app/index.html` | Coder | Semantic HTML5. Exact title `Project Pulse`. Links `styles.css`. `<main class="dashboard">` wrapper. Inline `<script>` (or small module) that `fetch`es `./project-data.json`, iterates `data.projects`, and renders a `<article class="project-card">` per project containing name, owner, `status-badge`, `recentActivity`, and `priority`. Renders an empty-state message if `projects` is empty or missing. Uses safe DOM APIs (`textContent`) to avoid HTML injection from data. Handles fetch errors with a visible message. Exposes deterministic class hooks Designer expects. |
| `app/styles.css` | Designer | Visual design for the polished dashboard. Selectors must include `.dashboard` and `.project-card` (validated). Uses `border-radius` and `box-shadow` (validated). Adds status badge styles (color-coded per status), priority emphasis, readable typography scale, spacing tokens, responsive grid layout (e.g. `grid-template-columns: repeat(auto-fill, minmax(...))`), and accessible focus/contrast. No dependence on data content strings beyond the documented enum values. |
| `.vscode/launch.json` | Coder | Strict JSON, no comments. One configuration named `Run Project Pulse Dashboard`. `type: node-terminal` (or equivalent that lets VS Code run a shell command and match `serverReadyAction`). `cwd: ${workspaceFolder}/app`. `command: python3 -m http.server 5500`. `serverReadyAction` with `pattern` matching Python's "Serving HTTP on ... port %s" and `uriFormat: http://localhost:%s/index.html`, `action: openExternally` (or `debugWithChrome` equivalent). Must open `index.html`, not `/`. |

## Dependencies

- Step 2 (data) depends on Step 1 (schema).
- Step 3 (HTML) depends on Step 1 for field names and enum values it will render; it does not need Step 2's actual content, only the shape.
- Step 4 (CSS) depends on Step 3's CSS class hooks being finalized and published to Designer.
- Step 5 (launch.json) depends on knowing the app is served as static files from `app/` and that the entry file is `index.html`. It does not depend on HTML/CSS/JSON content, only on the served path.
- Step 6 (integration) depends on all four files existing.

## Parallel Work

Once **Step 1 (schema)** is agreed and **Step 3 (HTML skeleton with CSS hooks)** is committed to a shared spec, the following can run concurrently because file scopes do not overlap:

- Coder: `app/project-data.json` (Step 2).
- Coder: `.vscode/launch.json` (Step 5) — no dependency on HTML/CSS/JSON content.
- Designer: `app/styles.css` (Step 4) — needs the class-hook contract from Step 3, not the rendering code.

If Step 3 has not yet been written, Step 4 and Step 5 can still proceed in parallel with Step 3 as long as the class-hook contract from Step 1 is treated as authoritative.

## Sequential Work

- Step 1 → Step 2 (schema before sample data).
- Step 1 → Step 3 (schema/class-hook contract before HTML).
- Step 3's class-hook contract → Step 4 (Designer needs the hooks to style).
- Steps 2, 3, 4, 5 all → Step 6 (integration validation).
- Coder must not touch `styles.css`, and Designer must not touch `index.html`, `project-data.json`, or `.vscode/launch.json`, per the agent rules ("Do not change design-only files unless explicitly assigned" / "Stay within the files assigned by the Orchestrator").

## Edge Cases

- **Empty `projects` array or missing key** → render a clearly styled `.empty-state` message ("No projects to display yet"), not a blank page.
- **Missing optional fields on a project** (e.g. no `recentActivity`) → render a graceful placeholder like "—" rather than `undefined`.
- **Unknown `status` or `priority` values** → apply a neutral fallback badge class (e.g. `.status-badge--unknown`) rather than an unstyled badge.
- **Long project names / long recent activity strings** → CSS should wrap or truncate gracefully; cards must not overflow their grid cell.
- **Many projects (10+)** → responsive grid should reflow, not force horizontal scroll.
- **Narrow viewports (~320px)** → single-column layout, tap targets remain legible.
- **`project-data.json` fetch fails** (served from wrong cwd, typo) → visible error message in the UI and a `console.error`; do not leave a blank dashboard.
- **HTML injection via data** → use `textContent`, never `innerHTML`, when inserting field values.
- **JSON strictness** → no trailing commas or comments in either `project-data.json` or `.vscode/launch.json`; both must pass `python3 -m json.tool`.
- **Launch config opens directory listing** — must include `serverReadyAction` with `/index.html` in the URI, matching the brief and the Step 3 validation (`http://localhost:%s/index.html`).
- **Port 5500 already in use** — document that the learner should stop any prior preview server before launching (Step 3 already tells them to stop the server before continuing).
- **Accessibility** — heading order (single `<h1>`, `<h2>` for card names), sufficient color contrast for badges, focus outlines preserved, semantic landmarks (`<main>`, `<article>`), `lang="en"` on `<html>`.

## Validation Expectations

- `python3 -m json.tool app/project-data.json` succeeds; top-level object contains `projects` array; every element has `name`, `owner`, `status`, `recentActivity`, `priority`.
- `python3 -m json.tool .vscode/launch.json` succeeds; contains configuration name `Run Project Pulse Dashboard`; `cwd` is `${workspaceFolder}/app`; URI pattern is `http://localhost:%s/index.html`.
- `app/index.html` contains the literal string `Project Pulse`, references `styles.css` and `project-data.json`, and renders at least one element with class `project-card`.
- `app/styles.css` contains selectors `.dashboard` and `.project-card`, plus `border-radius` and `box-shadow` declarations (matches Step 3 workflow keyphrase checks).
- Launching **Run Project Pulse Dashboard** from VS Code Run and Debug opens a browser to `http://localhost:5500/index.html` and shows the dashboard UI (not a directory listing).
- Manual browser check: cards visible, status badges styled, priority visually distinct, layout reflows on narrow viewport, no console errors, no `undefined` values shown.
- Accessibility spot-check: keyboard focus reaches interactive elements with a visible outline; badge/text color contrast meets WCAG AA; heading order is logical.
- The four files are new and not tracked in the template (aligns with `scripts/validate-exercise.sh` learner-file check).

## Open Questions

1. Should status/priority use fixed enums (recommended: `On Track`, `At Risk`, `Blocked`, `Complete` for status; `Low`, `Medium`, `High`, `Critical` for priority), or should the Designer propose the vocabulary? A fixed enum simplifies styling and fallback behavior.
2. Should recent activity include a timestamp field, or remain a single human-readable string as implied by the brief? The brief only specifies `recentActivity` as one field, so plan assumes a string.
3. Are cards interactive (clickable to a detail view) or purely informational in this iteration? Plan assumes informational only; no routing or detail pages.
4. Is a favicon or brand mark expected? Not required by the brief; plan omits unless Designer requests one within `index.html` scope (which would require Coder collaboration since Designer does not own `index.html`).
5. `launch.json` `type`: `node-terminal` is the most reliable way to run `python3 -m http.server` with `serverReadyAction` in VS Code. If the Orchestrator prefers a different type (e.g. a preLaunchTask), confirm before Coder implements.
