# Project Pulse dashboard implementation plan

## Summary

Build Mona's lightweight static Project Pulse dashboard so contributors can quickly see project names, owners, current status, recent activity, priority or risk, and a concise summary. The dashboard should be a polished, responsive, accessible page backed by a local JSON file and open directly to its UI from VS Code.

## Repository findings and constraints

- The brief is `.github/project-pulse-brief.md`. It requires a top-level `projects` array with `name`, `owner`, `status`, `recentActivity`, and `priority`; it also asks for contributor-friendly summaries, so include a non-empty `summary` for every project.
- The requested app files already exist: `app/index.html`, `app/styles.css`, and `app/project-data.json`. The existing page uses client-side `fetch("project-data.json")`, renders cards with `.project-card`, and shows loading, empty, and failure states. Preserve these behaviors while implementing or refining the app.
- Existing CSS provides `.dashboard` and `.project-card` hooks, project status/priority treatments, a responsive layout, reduced-motion handling, and forced-colors handling. Designer should retain or improve these affordances rather than replace them with an unstyled page.
- `.vscode/launch.json` already defines **Run Project Pulse Dashboard** with `python3 -m http.server 5500`, a working directory of `${workspaceFolder}/app`, and a `serverReadyAction` URL ending in `/index.html`. Keep it strict JSON and preserve this behavior.
- There is no app package manifest or dedicated browser-test suite. `scripts/validate-exercise.sh` checks exercise scaffolding and conventions; the Step 3 workflow checks required files, phrases, and JSON parsing, but neither proves the dashboard renders in a browser.
- `.vscode/tasks.json` and the dev-container scripts support the separate Copilot CLI exercise workflow; they are outside the implementation scope and must remain unchanged.

## Agent responsibilities and file ownership

| Agent | Responsibilities | Assigned files |
|---|---|---|
| **Planner** | Research the brief, repository, existing implementation, and validation limits; produce this plan with explicit ownership, dependencies, risks, and acceptance checks. Do not implement app files. | `docs/project-pulse-plan.md` |
| **Orchestrator** | Ask for and use the Planner's plan; confirm the shared UI/data contract; delegate non-overlapping work; sequence dependent work; track integration issues; run or coordinate final checks; report results and remaining risks. Coordinate only, do not implement the app. | No app implementation files |
| **Designer** | Shape and implement the visual system and responsive/accessibility behavior: information hierarchy, typography, card layout, status badges, priority/risk treatment, spacing, contrast, reduced-motion, and narrow-screen behavior. Share CSS hooks and design decisions before page integration. | `app/styles.css` |
| **Coder** | Implement the semantic page and data-driven behavior, representative deterministic project data, and runnable VS Code preview configuration. Connect the page to the agreed CSS hooks/data contract; handle loading, empty, malformed-data, and fetch-failure cases; validate assigned files. | `app/index.html`, `app/project-data.json`, `.vscode/launch.json` |

The Orchestrator must not broaden these scopes without explicitly reassigning ownership. In particular, Designer owns the stylesheet; Coder owns the HTML, data, and launch configuration. The plan is the Planner's only deliverable.

## Implementation phases and dependencies

### Phase 1 — Confirm the contract and assignments

- **Owner:** Orchestrator, informed by Planner and Designer.
- **Files:** No implementation files.
- Confirm that each project record contains non-empty `name`, `owner`, `status`, `recentActivity`, `priority`, and `summary` strings. Keep status and priority meaningful as visible text, not color alone.
- Agree on the shared markup/CSS hooks (`.dashboard`, `.project-grid`, `.project-card`) and the page's loading, success, empty, and error states. Keep project examples deterministic and contributor-friendly.
- Confirm the launch contract: serve `app/` on port `5500` and open `/index.html`, not the server root.
- **Dependency:** None. This contract gates Phase 2.

### Phase 2 — Build independent foundations (parallel)

After Phase 1, these file-scoped assignments can proceed in parallel:

1. **Designer — `app/styles.css`:** Create or refine the polished responsive visual design. Include `.dashboard` and `.project-card`; style cards, readable metadata, status and priority badges; provide clear contrast and preserve reduced-motion/forced-colors support where applicable.
2. **Coder — `app/project-data.json` and `.vscode/launch.json`:** Provide multiple representative records using the agreed six-field contract, and keep the launch configuration strict JSON with the exact name **Run Project Pulse Dashboard**, `cwd` `${workspaceFolder}/app`, command `python3 -m http.server 5500`, and a readiness action that opens `http://localhost:%s/index.html`.

- **Dependencies:** Both assignments depend on Phase 1. They use disjoint files and may run concurrently. Coder should share the data field names and launch assumptions with Designer/Orchestrator; Designer should communicate its hooks before HTML integration.

### Phase 3 — Integrate page and data (sequential)

- **Owner:** Coder; Orchestrator coordinates.
- **File:** `app/index.html`.
- Build a semantic page with the exact title **Project Pulse**, stylesheet link to `styles.css`, and client-side load of `project-data.json`. Render one `.project-card` per project and visibly show its name, summary, owner, status, recent activity, and priority. Match the Designer's agreed hooks and use safe text insertion for data.
- Preserve understandable loading, empty-project, invalid/missing-field, and fetch/HTTP failure messages. Ensure the page opens as the dashboard rather than relying on a `file://` fetch.
- **Dependencies:** Phase 2 must establish the data contract and CSS hooks before page integration. This phase is sequential with Phase 2's outputs; it may not overwrite Designer-owned CSS.

### Phase 4 — Integrated validation and fixes (sequential)

- **Owner:** Orchestrator coordinates; Coder fixes HTML/data/launch defects in its assigned files and Designer fixes CSS defects in `app/styles.css`.
- Validate all four implementation files together, launch the preview, check normal and failure states, then resolve mismatches within the existing assignments.
- **Dependencies:** Phases 1–3 are complete and implementation files are stable. Runtime validation is last because it must exercise the integrated page, data, styles, and launch configuration.

## Parallel versus sequential work

- **Parallel:** In Phase 2, Designer may work on `app/styles.css` while Coder works on `app/project-data.json` and `.vscode/launch.json`, after the contract and hooks are agreed. The file scopes do not overlap.
- **Sequential:** Agree the data/UI/launch contract first; finish the CSS hooks and data contract before Coder integrates `app/index.html`; validate the launch configuration only after all app files exist; run final runtime and accessibility checks after all implementation changes stop.
- **No overlapping edits:** Designer changes only `app/styles.css`; Coder changes only `app/index.html`, `app/project-data.json`, and `.vscode/launch.json`. Orchestrator coordinates and verifies rather than implementing. If an assignment needs to change, the Orchestrator should state the new ownership explicitly before work continues.

## Validation and acceptance expectations

1. **File assignment and boundaries**
   - Confirm the four expected outputs exist: `app/index.html`, `app/styles.css`, `app/project-data.json`, and `.vscode/launch.json`.
   - Review the diff to ensure this implementation work has not changed `.vscode/tasks.json` or unrelated dev-container/exercise files.

2. **Data contract**
   - Run `python3 -m json.tool app/project-data.json`.
   - Check that the root is an object with a `projects` array containing multiple entries and that each entry has non-empty `name`, `owner`, `status`, `recentActivity`, `priority`, and `summary` values.

3. **Launch configuration**
   - Run `python3 -m json.tool .vscode/launch.json` and confirm there are no comments or trailing-comma syntax errors.
   - Inspect the **Run Project Pulse Dashboard** configuration: command `python3 -m http.server 5500`, `cwd` `${workspaceFolder}/app`, a server-ready pattern matching Python's startup output, and URL `http://localhost:%s/index.html`.

4. **Page and styles**
   - Confirm the document title is **Project Pulse**, `styles.css` is linked, `project-data.json` is fetched, and the rendered card markup uses `.project-card`.
   - Confirm each project's name, summary, owner, status, recent activity, and priority is visible; verify the CSS contains `.dashboard`, `.project-card`, rounded card styling, shadows, and responsive rules.
   - Inspect loading, empty, invalid-data, and failed-fetch messages. Confirm data is inserted as text rather than interpreted as HTML.

5. **Runtime and responsive/accessibility review**
   - In VS Code, start **Run Project Pulse Dashboard**. Confirm the browser opens `http://localhost:5500/index.html`, displays the dashboard rather than a directory listing, and shows one card for each JSON entry. Stop the server after checking.
   - Inspect at desktop and narrow mobile widths (including around 320px); content must reflow without horizontal scrolling. Check heading/section structure, readable contrast, keyboard-visible focus for any interactive controls, and that status/priority meaning does not depend on color. Verify reduced-motion and forced-colors rules remain effective if present.

6. **Exercise checks**
   - Run `scripts/validate-exercise.sh` as a supplementary repository/exercise check.
   - The Step 3 GitHub Actions workflow (`.github/workflows/3-step.yml`) checks the four required files, expected phrases/selectors, JSON parsing, and launch name/target. It does not start the server or inspect rendered browser behavior; treat the runtime and accessibility checks above as separate acceptance requirements.

## Risks and open questions

- The brief does not prescribe sample projects, exact colors, or a fixed status/priority vocabulary. Choose consistent, contributor-friendly labels and communicate their visual mappings.
- The required data list omits `summary`, while the brief also requires a short contributor-friendly summary. Include `summary` as an additional required string field and render it.
- `fetch()` of the JSON is supported through the HTTP launch configuration, not by opening `app/index.html` directly as a `file://` URL.
- Port `5500` is part of the exercise's expected launch setup. If it is occupied, report the conflict rather than silently changing the expected URL.
- If the Planner agent cannot run because its configured model is unavailable, the Orchestrator should prepare this repository-grounded plan as a fallback and continue to honor the same file assignments and dependencies.
