# Project Pulse dashboard implementation plan

## Summary

Build Mona's lightweight Project Pulse dashboard as a static app that helps contributors see active projects, owners, current status, recent activity, priority or risk, and a short contributor-friendly summary. The repository brief requires `app/index.html`, `app/styles.css`, `app/project-data.json`, and a VS Code launch configuration that serves the `app/` directory and opens the dashboard page.

There is no existing `app/` directory or `launch.json`; the files in `.vscode/` currently include `tasks.json` for starting Copilot CLI, which should remain unchanged. The exercise already provides the project brief, the four custom agent definitions, a step-by-step learner workflow, and `scripts/validate-exercise.sh`. That script validates exercise scaffolding and learner-file conventions, but it does not launch or test the dashboard in a browser.

## Roles and file ownership

| Role | Responsibilities | Assigned files |
|---|---|---|
| **Planner** | Research the brief and repository, define the data/UI contract, sequence work, identify risks, and set acceptance checks. This plan is the Planner deliverable; the Planner does not implement app code. | `docs/project-pulse-plan.md` |
| **Orchestrator** | Use this plan to delegate work with explicit scopes, manage the dependencies and parallel phases below, integrate and verify the result, and report blockers or remaining risks. The Orchestrator coordinates but does not implement. | No implementation files |
| **Designer** | Define and implement a polished, responsive, accessible visual system: clear hierarchy, readable spacing and typography, visible project cards, status badges, and clear priority treatment. Include the deterministic `.dashboard` and `.project-card` hooks requested by the exercise. | `app/styles.css` |
| **Coder** | Implement the semantic dashboard page and data-driven rendering, provide deterministic sample project data, and create the runnable VS Code configuration. Keep the page connected to the Designer's CSS and the agreed data contract; handle loading and data errors clearly. | `app/index.html`, `app/project-data.json`, `.vscode/launch.json` |

Do not assign the same file to Designer and Coder. Designer should share the CSS hooks and any layout assumptions with Coder before integration; Coder owns HTML structure and data-to-UI behavior.

## Implementation steps and dependencies

### 1. Agree on the UI and data contract

- **Owner:** Orchestrator, informed by Planner and Designer.
- **Files:** No implementation files.
- Confirm the contract before parallel implementation: `project-data.json` has a top-level `projects` array; each item has `name`, `owner`, `status`, `recentActivity`, and `priority`. Include a concise contributor-friendly `summary` for each project as called for in the brief. Treat status and priority as readable text as well as visual styles, not color alone.
- Define the CSS hooks and the page's loading, success, empty, and error states. Keep the project collection small and deterministic for this static exercise.
- **Dependency:** None. This is the prerequisite for Steps 2 and 3.

### 2. Create the data and visual foundation (parallel)

- **Coder — `app/project-data.json`:** Add valid JSON with a top-level `projects` array and multiple representative project entries. Each entry must include the five required fields and a useful `summary`.
- **Designer — `app/styles.css`:** Implement the polished card-based, responsive styling. Include `.dashboard` and `.project-card`, visible status badges, readable priority/risk treatment, adequate contrast, and clear focus styling where interactive controls exist. Use the agreed markup hooks and avoid assuming color is the only way to identify status or priority.
- **Dependency:** Step 1 is complete. These two tasks can run in parallel because they own separate files and both follow the agreed contract. Designer and Coder should exchange the CSS hooks and field names before integration.

### 3. Connect the dashboard page and preview configuration (sequential)

- **Coder — `app/index.html`:** Build a semantic page titled **Project Pulse**, link `styles.css`, load `project-data.json`, and render a `.project-card` for each project. Show name, owner, status, recent activity, priority, and summary. Use the `.dashboard` and `.project-card` hooks and handle loading, empty data, and fetch/parse failures with understandable messages. Loading JSON over HTTP via the launch server avoids relying on `fetch()` from a `file://` page.
- **Coder — `.vscode/launch.json`:** Add strict JSON with a configuration named **Run Project Pulse Dashboard**. Serve the app from `${workspaceFolder}/app` using `python3 -m http.server 5500`, and configure the ready action to open `http://localhost:%s/index.html`, not the server root/directory listing.
- **Dependencies:** Step 2 must finish first: the HTML consumes both the agreed data shape and Designer's CSS hooks. The launch configuration depends on the app directory and the chosen `index.html` entry point. The two Coder files may be prepared during the same phase, but verify the actual launch only after all app files exist.

### 4. Integrate and validate (sequential)

- **Owner:** Orchestrator coordinates; Coder fixes implementation defects within its assigned files, and Designer fixes styling defects in `app/styles.css`.
- Review all four required outputs together, run the checks below, launch the preview, and resolve mismatches before handing off.
- **Dependency:** Steps 1–3 are complete. Do not begin final runtime validation while the app files are still changing.

## Parallel and sequential work

- **Can run in parallel:** After the Step 1 contract is agreed, Designer can implement `app/styles.css` while Coder creates `app/project-data.json`. Their file scopes do not overlap.
- **Must be sequential:** Agree on the data and CSS-hook contract before either parallel task; finish both before Coder integrates them in `app/index.html`; complete all implementation before integrated runtime validation.
- **Must not overlap:** Designer owns only `app/styles.css`; Coder owns `app/index.html`, `app/project-data.json`, and `.vscode/launch.json`. The Orchestrator should keep any corrections within those assignments unless it explicitly reassigns ownership.

## Validation expectations

1. **Files and contract:** Confirm all four assigned implementation files exist. In `app/project-data.json`, verify JSON parses, the root is an object with a `projects` array, and every project has non-empty `name`, `owner`, `status`, `recentActivity`, `priority`, and `summary`.
2. **Launch JSON:** Run `python3 -m json.tool .vscode/launch.json` and confirm it is strict JSON with no comments. Inspect that it has the exact **Run Project Pulse Dashboard** name, serves with `python3 -m http.server 5500`, sets `cwd` to `${workspaceFolder}/app`, and opens `/index.html` at the server-ready URL.
3. **Markup and styling:** Confirm `app/index.html` has the exact title **Project Pulse**, links `styles.css`, loads `project-data.json`, and renders project cards with the `.project-card` class. Confirm `.dashboard` and `.project-card` selectors exist and styling includes responsive behavior, readable spacing, status/priority treatments, rounded corners, and shadows.
4. **Runtime:** In VS Code, start **Run Project Pulse Dashboard**. Confirm the browser opens `http://localhost:5500/index.html` and displays the UI rather than a directory listing; check that every project field is visible and the number/content of cards matches the JSON. Stop the server after checking.
5. **Resilience and accessibility:** Inspect the page at a narrow viewport and a desktop width; verify content reflows without horizontal scrolling, text and status/priority remain understandable without color, and headings/regions and any controls are keyboard-accessible with visible focus. Check the loading, empty-project, and failed-data-load messages (using an empty or temporarily unavailable data response as appropriate), then restore the intended sample data.
6. **Repository validation:** `scripts/validate-exercise.sh` may be run as a supplementary exercise check; it is not a substitute for the JSON and browser checks above. It also checks broader repository workflow/dev-container conventions, not runtime dashboard behavior.

## Risks and open questions

- The brief does not prescribe exact sample projects, colors, or status/priority vocabularies. Choose realistic, contributor-friendly examples and consistent labels; keep them easy to edit in JSON.
- Browser data loading requires the HTTP launch configuration. Opening the HTML directly as a local file is not the supported preview path.
- The planned server port is `5500`, as specified in the exercise instructions. If it is already occupied, stop the conflicting process or report the issue rather than silently changing the expected launch URL.
- The brief calls for a contributor-friendly summary but does not include `summary` in its required field list. Include it as an additional per-project field so the dashboard meets both requirements.
