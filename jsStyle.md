# JavaScript Style Guide (Baseline from ie-system + material-management)

## 1. Purpose
This document records the **actual coding style currently used** across the two source codebases.

It is intended to:
- keep new code consistent with existing modules
- reduce style drift when multiple developers or agents contribute
- provide a practical baseline for other projects with similar constraints

This is an **as-is baseline**, not a pure "ideal architecture" document.

## 2. Current Stack Reality
The observed style is a hybrid of:
- ES module files (`import`/`export`) for feature code
- global utility/runtime objects (`Helper`, `loader`, `ready`, `SYSTEM_PATH`, `i18n`, `USER`)
- vanilla DOM APIs (`querySelector`, `addEventListener`)
- jQuery in legacy/shell behavior and selected views
- direct third-party UI usage (Highcharts, flatpickr, tippy, bootstrap modal, alert/confirm, Swal/alertify)

## 3. Hard Minimal Rules (Agent Must Follow)
- Minimal diff: do not refactor unrelated code.
- Do NOT create new helpers if logic is under 20 lines and used once.
  Exception: only if the same file already has a local-helper section AND the extraction removes duplicated logic (>= 2 identical blocks) OR reduces nesting (>= 2 levels).
- Do NOT add new validation unless the task introduces new user input, or a concrete failure case is reported.
- Validation should live at the nearest boundary (form submit / API call site).
  Do NOT add extra validators in render functions or deep helpers for small tasks.
- Do NOT introduce new abstraction layers (adapter/widget/service split) for small tasks.
- Do NOT reformat or reorder unrelated lines/sections. Keep changes localized to the smallest area that satisfies the task.
- Prefer editing existing functions over moving code across sections.
- Keep behavior and API stable unless explicitly required.

Anti-Hallucination Guardrails:
- Do NOT assume a function/variable/endpoint exists. Verify in the target file or imports first.
- Do NOT claim behavior/test results unless you actually verified them.
- If uncertain, state uncertainty explicitly and avoid presenting guesses as facts.

## 4. Local-Context Rule (Pick the File's Existing Style)
Before coding, classify the target file:
- If it already uses jQuery heavily, keep jQuery for the change.
- If it is ES module + vanilla DOM, keep vanilla DOM.
- If it already uses API wrapper functions, use wrapper functions.
- If it already uses direct `fetch`, use direct `fetch`.

Do not mix styles inside the same small feature unless unavoidable.

## 5. Output Contract
Applies to AI/agent task responses (not mandatory for internal documentation pages).
- `PATCH` (unified diff) first.
- `RATIONALE`: max 6 bullets.
- `TEST`: max 3 bullets.
- No extra sections.

## 6. Architectural Baseline (What Is Actually Used)
### 6.1 API Layer
Two patterns coexist and are both valid in the current baseline:
- Wrapped transport (`apiBase` with `get/post/put/del`, then feature-specific API modules)
- Direct `fetch(...)` inside feature modules

### 6.2 UI/Feature Layer
Feature files usually combine:
- state (`GLOBAL`, sometimes `DOM`)
- helper functions (local `parseNumber`, `safeGet`, formatters)
- render functions (charts/tables)
- async data loaders
- event wiring
- init/bootstrap (`ready(...)`)

### 6.3 Global Bridge Layer
For dynamic HTML handlers (`onclick`, inline `onsubmit`), functions are exposed on `window.*`.

This pattern is still used and considered part of the current style.

## 7. File Structure Pattern (Observed)
A common file order is:
1. Imports
2. Global constants (`scale`, config maps)
3. Local state (`GLOBAL`, optional `DOM`)
4. Local helper functions
5. Render functions
6. Async data functions (`get*`, `load*`, `post*`)
7. Event binding (`loadEvent`, delegated handlers)
8. `window.*` exports (if dynamic inline handlers exist)
9. `ready(...)` bootstrap

## 8. Naming Conventions (Observed)
- State: `GLOBAL`, `DOM`, `viewMode`, `feeTableCache`
- Async loaders: `get*`, `load*`, `fetch*`
- Renderers: `render*`, `draw*`, `build*`
- Actions/submits: `submit*`, `post*`
- Handlers: `handle*`

The codebase allows mixed naming granularity; consistency **within the same file/module** is prioritized.

## 9. State Management Style
Use module-scoped mutable objects for runtime view state.

Typical state content:
- selected filter values
- selected ids
- active date/tab/view mode
- temporary modal/edit context

Rules used in practice:
- keep state close to the module that owns it
- reset transient state on modal close where needed
- do not force global state managers

## 10. Data Access and Response Handling
### 10.1 Wrapped API style
When using wrapper modules:
- API function names represent domain intent
- wrappers call shared `get/post` helpers
- call sites consume `result.data` or `result.result` and normalize locally

### 10.2 Direct fetch style
When using direct `fetch`:
- build params with `URLSearchParams`
- parse `await response.json()`
- check `response.ok`
- throw/display `result.message` on failure

Both are baseline-valid. Do not force migration from one to the other in small tasks.

## 11. Async + Feedback Pattern
Most modules use this envelope:
```js
try {
  loader.load();
  // async work
} catch (error) {
  console.error(error.message || error);
  alert(error.message || 'Unexpected error');
} finally {
  loader.unload();
}
```

Also observed:
- confirmation via `window.confirm(...)`
- user feedback via `alert(...)` (and in some legacy modules: `alertify`/`Swal`)

## 12. Rendering Style
### 12.1 HTML construction
- Build table rows/cards via template literals.
- Inject via `innerHTML` into scoped containers.
- Build `<thead>` and `<tbody>` separately for table-heavy screens.

### 12.2 Empty fallback
Always render explicit fallback when no data:
- `No data`
- `No data to display`
- placeholder row with `colspan`

### 12.3 Chart rendering
- Centralize shared Highcharts defaults with a module-level init.
- Use dedicated render functions per chart type/section.
- Map raw API payload into chart-ready arrays first.

## 13. Event Handling Style
Observed combination:
- direct listeners for stable controls (`addEventListener`)
- delegated listeners for dynamic tables/lists
- inline handlers in generated HTML (`onclick`, inline `onsubmit`) when dynamic markup is heavily used

When inline handlers are used, corresponding functions are exported to `window`.

## 14. Form, Modal, and Upload Style
### 14.1 Form validation
Pattern used:
```js
event.preventDefault();
event.stopPropagation();

if (!event.target.checkValidity()) {
  event.target.classList.add('was-validated');
  return;
}
```

### 14.2 Modal flow
- open modal
- bind/update form values
- submit async
- close modal on success
- refresh section data

### 14.3 Upload flow
- read file input
- construct `FormData`
- call upload API
- clear input and reload data on success

## 15. Utility Pattern (Shared + Local)
The codebase uses both:
- shared helper container (`Helper.*`) for reusable utilities
- local helper functions in each module (`parseNumber`, `safeGet`, formatters)

Common utility concerns:
- null/undefined/empty normalization
- numeric parsing and formatting
- deep/safe property access
- simple search/filter helpers

Do not remove local helpers unless refactor scope is explicit.

## 16. Pagination Pattern
Two established styles exist:
- utility-style pagination functions + page button rendering logic
- reusable `Pagination` class with client-side/server-side mode

Both support:
- page slicing
- windowed visible page buttons + ellipsis
- jump-to-page input (in class-based variant)

## 17. jQuery and ES6 Coexistence Rules
This baseline supports both.

Use the rule below:
- If file/module is already jQuery-centric, keep jQuery style for small/medium edits.
- If file/module is vanilla ES module style, keep vanilla style.
- Avoid mixing two paradigms inside the same small function unless necessary.

Goal is consistency with local context, not forced uniformity.

## 18. Third-Party API Usage Policy (Adjusted to Reality)
Direct third-party usage is currently normal in feature modules.

So the practical rule is:
- Do **not** force new abstraction layers for tiny changes.
- Introduce abstraction only when change scope is medium/large, or when multiple modules repeat the same vendor-specific logic.

This avoids over-engineering in small tasks.

## 19. Global Runtime Assumptions
Common global dependencies in current style:
- `loader`
- `Helper`
- `SYSTEM_PATH`
- `i18n`
- `USER`
- `ready(...)`

When writing reusable code for another project, provide equivalent runtime hooks or adapters.

## 20. Practical Do/Don't (Based on Current Sources)
Do:
- keep async/UI/update flow explicit
- keep loader lifecycle paired (`load`/`unload`)
- keep table/chart rendering in dedicated functions
- keep fallback states visible
- keep module-internal style consistent with existing file style

Don't:
- introduce large architecture layers for one small endpoint/UI tweak
- force-migrate legacy jQuery files during unrelated fixes
- mix three different feedback styles in one small feature
- hide errors silently

## 21. Baseline Module Skeleton (Closest to Current Practice)
```js
import { fetchData, postSave } from '../../api/featureApi.js';

const GLOBAL = {
  filter: '',
  selectedId: null,
};

const renderTable = (rows) => {
  const html = rows.length
    ? rows.map((item) => `<tr onclick="openEdit(${item.id})"><td>${item.name || ''}</td></tr>`).join('')
    : '<tr><td colspan="10">No data</td></tr>';

  document.querySelector('#tbl tbody').innerHTML = html;
};

const getData = async () => {
  try {
    loader.load();
    const result = await fetchData({ keyword: GLOBAL.filter });
    const rows = result?.data || [];
    renderTable(rows);
  } catch (error) {
    console.error(error.message || error);
    alert(error.message || 'Load failed');
  } finally {
    loader.unload();
  }
};

const loadEvent = () => {
  document.querySelector('#ip-search').addEventListener('input', (e) => {
    GLOBAL.filter = e.target.value.trim();
    getData();
  });
};

window.openEdit = (id) => {
  GLOBAL.selectedId = id;
};

ready(function () {
  loadEvent();
  getData();
});
```

## 22. Cross-Project Reuse Guidance
To reuse this style in another project:
- keep the same control flow shape (state -> load -> map -> render -> events)
- keep hybrid tolerance (ES6 and jQuery where context requires)
- keep feedback and loader lifecycle explicit
- keep inline/global bridge only when dynamic HTML architecture requires it

If the new project is greenfield, you can modernize gradually, but this baseline should be the compatibility reference.

## 23. Style Intent Summary
This guide prioritizes:
- consistency with real production code
- readability over architectural purity
- incremental improvement over forced rewrites

That is the intended standard for current and near-term projects derived from these two sources.
