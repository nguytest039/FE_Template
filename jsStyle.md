# JavaScript Frontend Style Guide (Project-Derived, Library-Agnostic)

## 1. Purpose
This guide documents the coding style and architectural patterns extracted from this project and generalizes them for reuse in other JavaScript frontend codebases.

Goals:
- Keep modules easy to read and modify.
- Keep UI logic, domain logic, and infrastructure concerns separated.
- Reduce regressions by using predictable naming, structure, and behavior.
- Stay independent of specific libraries for HTTP, loading indicators, charting, or UI frameworks.

## 2. Scope
This guide applies to:
- Plain JavaScript modules (`import`/`export`).
- DOM-driven pages (forms, tables, charts, interactive views).
- Feature modules with asynchronous data loading and user actions.

This guide does not require a specific framework.

## 3. Core Principles
- Prefer explicit over clever.
- Keep each function small and single-purpose.
- Separate "fetch/map data" from "render UI."
- Use consistent naming to make behavior obvious.
- Minimize hidden dependencies and global side effects.
- Handle unhappy paths intentionally (validation errors, empty data, failed requests).

## 4. Recommended Folder and Layering Model
- `modules/*`: feature logic (UI orchestration, events, view state).
- `api/*` or `services/*`: transport-facing calls (endpoints, request configuration).
- `shared/*` or `helpers/*`: pure utilities reused across features.
- `common/*`: cross-page UI behavior (navbar, sidebar, auth UI, etc).

Rules:
- Feature modules can import services and helpers.
- Services should not import feature modules.
- Helpers should be mostly pure and framework-independent.

## 5. Standard Feature Module Structure
Use this section order for consistency:
1. Imports
2. Constants and immutable configuration
3. Local runtime state (`GLOBAL` / module-scoped state object)
4. Utility helpers local to the module
5. Render functions (table/chart/DOM sections)
6. Data orchestration functions (load/map/transform)
7. User action handlers (submit/edit/delete/filter)
8. Event wiring (`loadEvent`, delegation setup)
9. Bootstrapping (`ready(...)` or equivalent init entry)
10. Optional `window.*` exports (only for unavoidable legacy inline handlers)

## 6. Naming Conventions
### 6.1 Constants and State
- Constants: `UPPER_SNAKE_CASE` (`DEFAULT_VIEW_MODE`, `MAX_UPLOAD_SIZE`).
- State objects: `GLOBAL`, `DOM`, or explicit names like `viewState`.
- Avoid cryptic names (`a`, `tmp1`, `obj2`) outside very small scopes.

### 6.2 Function Prefixes by Responsibility
- `get*` / `load*`: retrieve or initialize data.
- `render*` / `build*`: prepare or write UI output.
- `handle*`: event handlers, often synchronous routing logic.
- `submit*`: form submit workflows.
- `post*` / `create*` / `update*` / `delete*`: command actions.
- `parse*` / `format*` / `safe*`: conversion and utility helpers.

### 6.3 Boolean and Condition Names
- Use meaningful booleans: `isValid`, `hasData`, `isEditable`, `shouldRefresh`.
- Avoid ambiguous booleans: `flag`, `status1`.

## 7. Function Design Guidelines
- Keep function length practical; split when a function mixes too many concerns.
- Prefer pure helpers for transformation logic.
- Pass dependencies as parameters when practical instead of reading globals everywhere.
- Return predictable shapes from helpers (for example `{ thead, tbody }` instead of mixed types).

Good pattern:
```js
const buildRows = (items) => {
  return items.map((item) => ({
    id: Number(item.id),
    label: item.name || '',
    value: Number(item.value || 0),
  }));
};
```

## 8. State Management (Without Framework)
Use a small module-scoped object for runtime UI state:
```js
const GLOBAL = {
  viewMode: 'week',
  activeDate: null,
  selectedId: null,
};
```

Guidelines:
- Keep state minimal and explicit.
- Store only what is needed to render/act.
- Derive computed values when needed; do not persist everything.
- Reset ephemeral state when modal/page context closes.

## 9. Data Transformation Rules
- Normalize API/domain data before rendering.
- Handle null/undefined/empty string consistently.
- Convert types early (numbers, dates, booleans).
- Keep fallback defaults explicit.

Preferred utility patterns:
- `safeGet(obj, 'a.b.c', fallback)`
- `parseNumber(value, fallback)`
- `normalizeDate(dateLike)`

## 10. DOM and Rendering Guidelines
### 10.1 Rendering Strategy
- Build HTML strings or fragments first, then update DOM once.
- Avoid repeated `innerHTML +=` in loops on large data sets.
- Keep render functions idempotent where possible.

### 10.2 Empty and Error States
- Always render a clear empty state (`No data`, `No records found`).
- Render recoverable error state for critical sections.

### 10.3 Selectors and Access
- Prefer `querySelector`/`querySelectorAll`.
- Cache frequently reused DOM references where useful.
- Avoid deeply chained selectors if a stable ID/class is available.

## 11. Event Handling Guidelines
### 11.1 Prefer Delegation for Dynamic Content
```js
tbody.addEventListener('click', (event) => {
  const cell = event.target.closest('[data-action="edit"]');
  if (!cell) return;
  handleEdit(cell.dataset);
});
```

### 11.2 Prevent Duplicate Bindings
- Bind listeners in a single init path.
- Avoid rebinding listeners inside render unless intentionally scoped.

### 11.3 Avoid Inline Handlers
- Prefer `addEventListener`.
- If legacy HTML requires inline calls, expose a minimal bridge via `window`.

## 12. Form and Validation Standards
- For submit flows:
  - call `preventDefault()` and `stopPropagation()`.
  - validate before any command action.
  - show field-level feedback and global feedback clearly.
- Keep validation logic close to form handling, with reusable validators in helpers.

Template:
```js
const submitForm = async (event) => {
  event.preventDefault();
  event.stopPropagation();

  if (!event.target.checkValidity()) {
    event.target.classList.add('was-validated');
    return;
  }

  // command action
};
```

## 13. Async, Errors, and User Feedback (Library-Agnostic)
Use a standard async envelope:
```js
try {
  // optional: show busy indicator
  const result = await serviceCall(params);
  // map and render
} catch (error) {
  console.error(error);
  // optional: show toast/dialog/inline message
} finally {
  // optional: hide busy indicator
}
```

Guidelines:
- Never swallow errors silently.
- Surface actionable messages to users; keep technical details in logs.
- Handle cancellation/aborts where relevant (search/filter rapidly changing UI).
- Ensure UI state is restored in `finally`.

## 14. Library Independence Strategy
To keep portability:
- Define adapters/interfaces at the boundary:
  - `dataClient` (HTTP or RPC)
  - `notify` (toast/dialog/alert)
  - `busy` (loading overlay/spinner)
- Keep feature modules calling abstractions, not concrete library APIs.

Example:
```js
// feature module depends on abstraction
await dataClient.get('/items', params);
notify.error(message);
busy.show();
busy.hide();
```

You can later map these to any stack (Fetch, Axios, RTK Query, custom SDK, etc.).

## 15. Formatting and Readability
- Use consistent indentation and semicolon style across repo.
- Keep line length readable.
- Group related constants and functions.
- Use short, meaningful comments only where intent is non-obvious.
- Remove dead/commented blocks before merge unless intentionally retained with reason.

## 16. Internationalization and Text
- Do not hardcode user-facing strings when i18n is required.
- Centralize labels/messages through translation keys.
- Keep fallback text deterministic if translation data is missing.

## 17. Security and Data Safety Basics
- Treat all remote data as untrusted.
- Sanitize or escape user-generated values before injecting into HTML.
- Avoid exposing sensitive data in logs.
- Validate critical numeric/date inputs before command actions.

## 18. Quality and Maintainability Rules
- Keep cyclomatic complexity controlled.
- Avoid huge "god modules"; split by domain when a file grows too much.
- Prefer composition over deep conditional chains.
- When behavior diverges by type/mode, use a strategy map where possible.

Example:
```js
const handlersByType = {
  create: handleCreate,
  update: handleUpdate,
  remove: handleRemove,
};
```

## 19. Practical Do/Don't List
Do:
- Normalize data before rendering.
- Keep render and data-fetch concerns separated.
- Use delegation for dynamic table/list interactions.
- Keep state transitions explicit.

Don't:
- Mix transport, transformation, and UI writes in one giant function.
- Depend heavily on global mutable state without clear ownership.
- Duplicate the same parsing/formatting logic in many modules.
- Rely on implicit type coercion in critical calculations.

## 20. Reusable Feature Skeleton
```js
import { listItems, saveItem } from '../services/itemService.js';

const GLOBAL = {
  filter: '',
  selectedId: null,
};

const renderTable = (rows) => {
  const html = rows
    .map((row) => `<tr data-id="${row.id}"><td>${row.name}</td><td>${row.value}</td></tr>`)
    .join('');

  document.querySelector('#tbl-items tbody').innerHTML =
    html || '<tr><td colspan="2">No data</td></tr>';
};

const getData = async () => {
  try {
    const result = await listItems({ keyword: GLOBAL.filter });
    const rows = (result?.data || []).map((item) => ({
      id: Number(item.id),
      name: item.name || '',
      value: Number(item.value || 0),
    }));
    renderTable(rows);
  } catch (error) {
    console.error(error);
  }
};

const loadEvent = () => {
  document.querySelector('#btn-search').addEventListener('click', () => {
    GLOBAL.filter = document.querySelector('#txt-keyword').value.trim();
    getData();
  });

  document.querySelector('#tbl-items tbody').addEventListener('click', (event) => {
    const row = event.target.closest('tr[data-id]');
    if (!row) return;
    GLOBAL.selectedId = Number(row.dataset.id);
  });
};

const init = () => {
  loadEvent();
  getData();
};

init();
```

## 21. Optional Governance Additions
For team-scale reuse, add:
- ESLint ruleset and Prettier config.
- Pull request checklist for this guide.
- Shared templates for module scaffolding.
- Architecture Decision Records (ADR) for exceptions.
