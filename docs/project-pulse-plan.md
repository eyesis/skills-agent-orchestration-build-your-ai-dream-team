# Project Pulse Dashboard — Implementation Plan

## 1. Summary

Mona's team needs a small, static Project Pulse dashboard that lets contributors quickly scan active projects, owners, status, recent activity, priority/risk, and a short summary — presented as a polished set of project cards with status badges and readable spacing. The deliverable is exactly four files:

- `app/index.html` — dashboard markup (Designer)
- `app/styles.css` — dashboard visual design (Designer)
- `app/project-data.json` — project data feeding the dashboard (Coder)
- `.vscode/launch.json` — VS Code launch config, "Run Project Pulse Dashboard," serving `app/` and opening `index.html` (Coder)

The repository currently has an **empty `app/` directory** and **no `.vscode/launch.json`** (only `.vscode/tasks.json` for the Copilot CLI terminal). The devcontainer is `mcr.microsoft.com/devcontainers/universal:2`, which typically ships Node.js and Python, so the launch config can rely on `npx`-based static servers or Python's built-in HTTP server — this needs to be confirmed at implementation time (see Open Questions/Risks).

Because `index.html`/`styles.css` (Designer) and `project-data.json`/`launch.json` (Coder) are physically separate files, most of the work can be **parallelized** after a short shared-contract step that fixes the JSON field names, the JS variable/fetch approach for loading data, and the CSS hook names the HTML must use.

## 2. Ordered implementation steps

1. **Step 0 — Shared contract (sequential, brief)**
   Before Designer and Coder touch files independently, fix the exact shape of `project-data.json` (field names/types) and how `index.html` will consume it (e.g., `fetch('project-data.json')` vs. inline `<script>` data, since some static servers may block `fetch` on `file://` — mitigated by always using an HTTP server per the launch config). This can be done as a short written contract in the plan (see below) rather than a separate implementation pass, so Designer and Coder can start immediately without waiting on each other's code.

   **Contract:**
   - `project-data.json` top-level shape: `{ "projects": [ { "name": string, "owner": string, "status": string, "recentActivity": string, "priority": string, "summary"?: string } ] }`. The brief's required fields are `name`, `owner`, `status`, `recentActivity`, `priority`; a `summary` field is recommended (not required) to satisfy "short contributor-friendly summary" but Designer's HTML must not hard-require it if Coder omits it — Designer should code defensively (render only if present, or fall back to blank).
   - `status` and `priority` are free-text strings from a small controlled vocabulary (e.g., status: `Active`, `On Hold`, `Blocked`, `Completed`; priority: `High`, `Medium`, `Low`) — Coder picks the vocabulary, Designer must style generically by value (e.g., data attributes) rather than hardcoding a fixed enum in CSS class names, so future values don't break styling.
   - `index.html` loads data client-side via `fetch('./project-data.json')` and renders cards into a container with class `.dashboard`; each rendered project card carries class `.project-card`.
   - Designer owns the small inline `<script>` (or `<script src>`) in `index.html` that fetches and renders — this is UI rendering logic tied to layout/markup, so it stays with Designer per "owns `app/index.html` structure." If the Orchestrator prefers Coder to own JS logic, that must be called out explicitly as an exception; default assumption here is Designer implements the render script since there is no separate JS file in the required file list and rendering is tightly coupled to markup structure.

2. **Step 1 — Coder: build `app/project-data.json`**
   Create realistic, contributor-friendly sample data (multiple projects) matching the contract in Step 0. No dependency on Designer's markup.

3. **Step 2 — Designer: build `app/index.html`**
   Build semantic, accessible markup: a `.dashboard` container, a template/render script that fetches `project-data.json` and produces `.project-card` elements with name, owner, status badge, recent activity, priority indicator, and summary (if present). Include a `<link rel="stylesheet" href="styles.css">`.

4. **Step 3 — Designer: build `app/styles.css`**
   Style `.dashboard` (responsive grid/flex layout, spacing) and `.project-card` (rounded corners, shadow, contrast, typography), plus status-badge and priority visual treatment (e.g., color-coded via `data-status`/`data-priority` attributes set in the HTML render step). This depends on Step 2's markup/attribute names existing (or being defined concurrently by the same agent), so Steps 2 and 3 are effectively one Designer work unit and can be done back-to-back by Designer without waiting on Coder.

5. **Step 4 — Coder: build `.vscode/launch.json`**
   Add the "Run Project Pulse Dashboard" launch configuration: strict JSON, no comments, `cwd` = `${workspaceFolder}/app`, must serve the `app/` directory as a static site and open `index.html` directly (not a directory listing). No dependency on Designer's file contents, only on the fact that `app/index.html` will exist at that path.

6. **Step 5 — Integration check (sequential, after Steps 2–4 complete)**
   Confirm: `index.html`'s fetch path (`./project-data.json`) resolves correctly when served from `app/` as the web/working root; confirm the launch config's server root and "open index.html" behavior actually renders the dashboard, not a directory listing or 404; confirm CSS class hooks (`.dashboard`, `.project-card`) are present and used by the HTML exactly as Designer intends.

7. **Step 6 — Validation pass (Orchestrator or learner-driven, no new files)**
   Run through the Validation Expectations checklist below.

## 3. File assignments

| File | Owner | Notes |
|---|---|---|
| `app/index.html` | **Designer** | Structure, semantic markup, accessibility, `.dashboard`/`.project-card` hooks, the small render/fetch script that turns `project-data.json` into cards. |
| `app/styles.css` | **Designer** | All visual polish: cards, badges, spacing, responsive layout, color/contrast for status and priority. |
| `app/project-data.json` | **Coder** | Data content only: top-level `projects` array with `name`, `owner`, `status`, `recentActivity`, `priority` (plus optional `summary`). Strict, valid JSON. |
| `.vscode/launch.json` | **Coder** | Strict JSON, no comments. "Run Project Pulse Dashboard" config. `cwd`: `${workspaceFolder}/app`. Must serve `app/` and open `index.html`, not a directory listing. |

**Hand-off note:** Designer's `index.html` script must reference the *exact* field names Coder uses in `project-data.json` (`name`, `owner`, `status`, `recentActivity`, `priority`, optional `summary`). This is the one place both agents' outputs are coupled — it is resolved by the Step 0 contract above, not by either agent editing the other's file. Neither agent should directly edit the other's assigned file; if a field-name mismatch is discovered during integration (Step 5), the Orchestrator should route a small, explicitly-scoped fix back to whichever agent owns the mismatched file.

## 4. Designer responsibilities

- Own `app/index.html`: semantic structure (e.g., `<main class="dashboard">`, one project card element per project), accessible markup (proper heading levels, ARIA labeling for status/priority badges where color alone conveys meaning, alt/text equivalents), and the client-side rendering script that fetches `project-data.json` and builds the cards.
- Own `app/styles.css`: responsive grid/flex layout for `.dashboard`; `.project-card` visual treatment (rounded corners, shadow, spacing, contrast); distinct, readable status badge styling; a clear visual treatment for priority/risk (e.g., color-coded left border, icon, or badge) that also has a non-color signal (text/icon) for accessibility.
- Ensure the first rendered view unmistakably reads as a "Project Pulse dashboard," not a bare unstyled page.
- Use deterministic, stable class names (`.dashboard`, `.project-card`, plus any additional hooks like `.status-badge`, `.priority` used consistently) so validation and any later automation can reliably target them.
- Code defensively against data variability: unknown status/priority values should still render with a sane default style rather than breaking layout.
- Report design decisions, exact class/attribute names introduced, and any accessibility follow-ups.

## 5. Coder responsibilities

- Own `app/project-data.json`: produce a realistic, varied sample set of projects (recommend 4–6) covering multiple statuses and priorities so Designer's styling has real variety to prove out. Ensure valid, parseable JSON (no trailing commas, no comments — JSON doesn't support comments).
- Own `.vscode/launch.json`: strict JSON, no comments; configuration named exactly **"Run Project Pulse Dashboard"**; `cwd` set to `${workspaceFolder}/app`; behavior must serve the `app/` directory as the web root and land the learner on `index.html` (e.g., via a static file server command/task that defaults to `index.html`, or a browser-launch config pointing at `http://localhost:<port>/index.html` after a server task starts) — must not surface a raw directory listing.
- Verify the chosen serving mechanism is actually available in the devcontainer (Node/`npx` or Python) before finalizing the launch config; document the exact command used.
- Validate JSON files parse cleanly (e.g., via `node -e "JSON.parse(...)"` or `python -m json.tool`) before reporting completion.
- Report what was implemented, how it was validated, and any remaining risk (e.g., tool availability assumptions).

## 6. Dependencies between steps

- Step 0 (contract) → must precede Steps 1–4 conceptually, but since it's a short agreed-upon spec (not code), it doesn't block parallel start — both agents can begin as soon as the contract is communicated.
- Step 1 (`project-data.json`) has no file dependency on Steps 2–4, but Step 5 integration needs Step 1's actual field names to match what Step 2's script expects.
- Step 2 (`index.html`) and Step 3 (`styles.css`) are both Designer-owned and naturally sequential/iterative within Designer's own work (markup often needs to exist before styling it), but neither depends on Coder's files to be authored — only the shared field-name contract.
- Step 4 (`launch.json`) depends only on `app/index.html` existing at the expected path by the time it's actually run/tested — it can be authored before `index.html` is finished, but cannot be *validated end-to-end* until `index.html` exists.
- Step 5 (integration check) depends on Steps 1–4 all being complete.
- Step 6 (validation) depends on Step 5.

## 7. Work that can run in parallel

- **Designer track (`app/index.html` + `app/styles.css`)** and **Coder track (`app/project-data.json` + `.vscode/launch.json`)** can run fully in parallel immediately after the Step 0 contract is set, because:
  - The two tracks touch entirely disjoint files — no shared file is edited by both agents.
  - The only coupling point (JSON field names ↔ HTML render script) is resolved by the shared contract up front, not by either agent reading the other's in-progress file.
  - `launch.json`'s correctness doesn't depend on the *content* of `index.html`/`styles.css`, only on the *path* `app/index.html` existing, which is guaranteed by directory convention, not by waiting on Designer's output.
- This is the Orchestrator's natural two-phase-in-parallel split: Phase A = {Designer: Step 2+3} run concurrently with Phase A = {Coder: Step 1+4}.

## 8. Work that must run sequentially

- Step 0 (contract) before both tracks start, to avoid field-name drift.
- Within Designer's track, `index.html` markup/attributes (Step 2) should be settled (at least the class/data-attribute names) before final styling polish (Step 3) locks in selectors — practically these can iterate together since one agent owns both files.
- Step 5 (integration check) must run after both tracks complete — this is the first point Coder's and Designer's outputs are exercised together.
- Step 6 (validation) must run after Step 5.
- Any fix arising from Step 5 mismatches must be routed back to the single owning agent for that file (sequential patch), not resolved by cross-editing.

## 9. Edge cases to handle

- **Empty or malformed `project-data.json`**: Designer's render script should handle a fetch failure or empty `projects` array gracefully (e.g., show a "No projects yet" state) rather than throwing an unhandled error into a blank page.
- **Unknown/future `status` or `priority` values**: styling should degrade gracefully (default badge style) instead of showing unstyled text or broken layout.
- **Missing optional `summary` field**: HTML should not render an empty/undefined string literal ("undefined") — must conditionally render.
- **`file://` fetch restrictions**: opening `index.html` directly by double-click (without a server) will likely block `fetch()` of local JSON in some browsers due to CORS/file-protocol restrictions. This is exactly why `launch.json` must serve via HTTP, not just open the file — Coder must ensure the launch config actually starts an HTTP server rooted at `app/`, not just open the file path.
- **Directory listing instead of `index.html`**: if the chosen static server doesn't default to `index.html`, learners will see a file listing — Coder must explicitly configure/verify default-document behavior or hardcode the URL to `.../index.html`.
- **Port conflicts**: if the launch config hardcodes a port, document it and pick a low-collision port; consider deterministic but non-common ports.
- **JSON strictness**: no trailing commas, no comments, no single quotes — both `project-data.json` and `launch.json` must be strict JSON (VS Code's `launch.json` normally tolerates comments via jsonc, but the brief explicitly requires **no comments, strict JSON** here — Coder must not rely on VS Code's usual comment-friendly leniency).
- **Long text overflow**: `recentActivity`/`summary` values of varying length must not break card layout — Designer should ensure text wrapping/truncation handling.
- **Accessibility of color-only signals**: status/priority must not rely on color alone (colorblind users) — pair color with text/icon.
- **Devcontainer tool availability**: the plan assumes Node/Python is present in `universal:2` for serving static files; if neither is confirmed, the launch config approach needs adjustment (see Open Questions).

## 10. Validation expectations

- `app/project-data.json` parses as valid JSON (e.g., `python -m json.tool app/project-data.json` or `node -e "JSON.parse(require('fs').readFileSync('app/project-data.json'))"`) with no errors.
- `app/project-data.json` has a top-level `projects` array, and every entry contains at minimum `name`, `owner`, `status`, `recentActivity`, `priority` as non-empty strings.
- `.vscode/launch.json` parses as valid strict JSON (no comments, no trailing commas) and contains a configuration whose `name` is exactly **"Run Project Pulse Dashboard"**.
- `.vscode/launch.json`'s serving configuration resolves `cwd`/working root to `${workspaceFolder}/app` (or an equivalent path that is unambiguously the `app` directory).
- Running the "Run Project Pulse Dashboard" launch configuration results in the learner viewing the rendered Project Pulse dashboard (cards, badges) at `index.html`'s content — **not** a raw file/directory listing and not a blank/error page.
- `app/styles.css` defines rules targeting `.dashboard` and `.project-card` selectors (grep/inspect confirms presence).
- `app/index.html` contains an element with class `dashboard` and, once rendered, produces one or more elements with class `project-card` per project in the data file.
- Visual/manual check: cards show name, owner, status (as a badge), recent activity, and a priority/risk indicator, with readable spacing and no obvious overflow/clipping at common viewport widths (e.g., 375px, 768px, 1280px).
- No console errors on load (fetch succeeds, JSON parses, DOM renders) when served via the launch configuration.

## 11. Open questions

- **Exact static-serving mechanism for `launch.json`**: not yet verified whether Node/`npx` (e.g., `npx serve`, `npx http-server`, `npx live-server`) or Python's `http.server` is the more reliable choice inside the `universal:2` devcontainer without additional installs. Coder should verify tool availability at implementation time and document the chosen command; this plan does not prescribe one to avoid over-constraining an unverified environment detail.
- **Does "open `index.html`" require an actual browser auto-launch, or is landing on the correct URL (no directory listing) sufficient?** If auto-open is required, the chosen server/tool needs an `--open` style flag or the launch config needs a paired browser-debug configuration (e.g., `pwa-chrome` type with `serverReadyAction`) — this affects `launch.json`'s structure and should be confirmed with the Orchestrator/learner before Coder finalizes it.
- **Who owns the render script inside `index.html`?** This plan assumes Designer owns it (since there's no separate `.js` file in the required file list and it's tightly coupled to markup), but the Orchestrator may prefer Coder to own all "logic" per the general Coder/Designer split described in the agent docs. This should be explicitly confirmed before Phase execution to avoid ambiguity.
- **Should `project-data.json` include the optional `summary` field?** The brief explicitly lists five required fields and separately asks for "a short contributor-friendly summary" as a dashboard capability — it's unclear if that summary should come from the data file or be a static/derived UI element (e.g., composed from existing fields). Recommend adding `summary` to the JSON contract, but this should be confirmed.
- **Number and realism of sample projects**: not specified in the brief; this plan recommends 4–6 for meaningful visual variety, but the exact count/content is Coder's judgment call unless the learner specifies otherwise.
