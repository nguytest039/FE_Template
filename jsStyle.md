# JavaScript Frontend Style Guide (Cross-Project, Project-Derived)

## 1. Purpose
This guide captures coding patterns extracted from real production JavaScript and generalizes them so teams can reuse the same style in different projects, domains, and stacks.

Use this guide to keep code:
- readable and predictable
- modular and testable
- framework/library independent
- stable under changing business requirements

## 2. Scope
Applies to:
- Browser JavaScript modules (`import`/`export`) and legacy script modules.
- DOM-driven pages (forms, tables, charts, dashboards, map-like views).
- Feature code with asynchronous data loading and user interactions.

Does not force:
- any specific HTTP library
- any specific notification/loading library
- any specific UI framework

## 3. Core Principles
- Prefer explicit behavior over implicit side effects.
- Keep each function single-purpose.
- Separate data retrieval, data normalization, and rendering.
- Keep state transitions visible and intentional.
- Handle empty/error/loading states by design, not as afterthoughts.
- Encapsulate reusable UI behavior instead of duplicating event code.

## 4. Layering Model
Recommended layering:
- `modules/*`: feature orchestration and page behavior.
- `services/*` or `api/*`: backend transport calls.
- `shared/*` or `utils/*`: pure helpers and formatting logic.
- `ui/*` or `widgets/*`: reusable UI components/controllers.
- `core/*`: app shell behavior (layout lifecycle, global interactions).

Dependency rules:
- Feature modules can depend on services/utils/widgets.
- Services cannot depend on feature modules.
- Utils should avoid DOM and transport dependencies.
- Core layer should not know business rules.

## 5. Standard Feature File Structure
Use consistent section order:
1. Imports
2. Constants/config
3. Local state
4. Selectors/cache
5. Pure helper functions
6. Data normalization functions
7. Render functions
8. Action handlers (submit/create/update/delete)
9. Event wiring
10. Init/bootstrap

## 6. Naming Conventions
### 6.1 Variables and constants
- Constants: `UPPER_SNAKE_CASE`.
- Local state objects: `state`, `viewState`, `filterState`.
- DOM cache objects: `dom`.

### 6.2 Function prefixes
- `load*`, `fetch*`: get remote/local data.
- `normalize*`, `map*`, `to*`: transform data shape.
- `render*`, `build*`: produce UI output.
- `bind*`, `wire*`: attach events.
- `handle*`: route event/action logic.
- `validate*`: form or business validation.

### 6.3 Boolean naming
- `is*`, `has*`, `can*`, `should*`.
- Avoid ambiguous names like `flag`, `status1`.

## 7. Utility Function Standards
Define shared utility behavior once and reuse everywhere.

Recommended utility groups:
- **Existence/empty checks**: `isNil`, `isBlank`, `isEmptyObject`, `isEmptyArray`.
- **Fallback formatters**: `toEmptyString`, `toNA`, `toZero`.
- **Type conversion**: `toNumber`, `toBoolean`, `toDate`.
- **Safe access**: `safeGet`, `pick`, `coalesce`.

Rules:
- Keep helpers pure (no DOM mutation, no global writes).
- Return deterministic values for invalid input.
- Centralize null/undefined handling to avoid repeated inline checks.

## 8. Data Normalization Contract
Normalize incoming data before rendering.

Normalization checklist:
- convert numeric fields to numbers
- normalize null-like values (`null`, `undefined`, `'null'`, `'undefined'`)
- assign explicit defaults
- normalize date/time formats
- map backend enum/status to UI-friendly structure

Example output model:
```js
{
  id: 0,
  name: '',
  quantity: 0,
  status: { key: 'unknown', label: 'Unknown', tone: 'neutral' }
}
```

## 9. State Management Without Framework
Use one module-scoped state object.

Rules:
- Store minimal mutable state only.
- Compute derived values on demand.
- Reset ephemeral state after modal/dialog closes.
- Do not spread mutable globals across files.

Suggested pattern:
```js
const state = {
  page: 1,
  pageSize: 20,
  keyword: '',
  sort: { field: 'createdAt', direction: 'desc' },
};
```

## 10. Rendering Strategy
### 10.1 Render pipeline
Use a predictable pipeline:
1. load/fetch data
2. normalize data
3. compute view model
4. render once per region
5. bind/update interactions

### 10.2 DOM update rules
- Build HTML string/fragment first, then update DOM once.
- Avoid repeated `innerHTML +=` in loops.
- Keep render functions idempotent.
- Always handle empty state explicitly.

### 10.3 Injection safety
- Escape untrusted strings before injecting into HTML.
- Never trust remote values.

## 11. Event Handling and Interaction
- Prefer event delegation for dynamic content.
- Bind events in one init path to avoid duplicate listeners.
- Avoid inline HTML handlers; use a legacy bridge only when required.
- Guard action handlers early (`if (!target) return`).

Delegation pattern:
```js
tableBody.addEventListener('click', (event) => {
  const action = event.target.closest('[data-action]');
  if (!action) return;
  handleTableAction(action.dataset.action, action.dataset.id);
});
```

## 12. Table and Pagination Logic Standards
### 12.1 Pagination model
Use deterministic pagination state:
- `page`
- `pageSize`
- `total`
- `windowSize` (visible page button window)

### 12.2 Windowed page button logic
For large result sets, render a window around current page and include first/last with ellipsis as needed.

### 12.3 Responsibilities split
- `paginate(data, page, pageSize)`: returns sliced data + page count.
- `buildPaginationView(state)`: returns pagination view model.
- `renderPagination(viewModel)`: renders buttons.
- `wirePaginationEvents()`: updates page and refreshes data.

### 12.4 Table update contract
- Render `<thead>` and `<tbody>` from the same schema definition.
- Keep row rendering separate from pagination rendering.
- Ensure deterministic fallback row for empty data.

## 13. Reusable Widget Pattern
When interaction complexity grows (drag reorder, tree grid, advanced filters), move behavior into widget classes/controllers.

Widget rules:
- Constructor receives root element + optional config.
- Expose lifecycle methods: `init()`, `destroy()`, optional `refresh(data)`.
- Keep event handlers bound to widget instance.
- Never leak listeners on re-init.

Minimal structure:
```js
class ReorderTable {
  constructor(root, options = {}) {
    this.root = root;
    this.options = options;
  }

  init() {
    this.bindEvents();
  }

  destroy() {
    this.unbindEvents();
  }
}
```

## 14. Form and Validation Rules
- Stop native submit flow first.
- Validate locally before command actions.
- Show both field-level and global feedback.
- Keep validation rules deterministic and reusable.
- Split `validateForm(data)` from `submitForm(data)`.

## 15. Async and Error Envelope
Use consistent async structure:
```js
try {
  busy.show();
  const raw = await service.get(params);
  const data = normalizeResult(raw);
  render(data);
} catch (error) {
  logger.error(error);
  notify.error(getUserMessage(error));
} finally {
  busy.hide();
}
```

Rules:
- never swallow exceptions
- always restore UI state in `finally`
- keep user messages non-technical
- keep technical detail in logs

## 16. Adapter Boundary (Library Independence)
Do not call concrete third-party APIs directly inside feature modules.

Define boundaries:
- `dataClient`: remote calls
- `notify`: toast/dialog/alert
- `busy`: loading state
- `storage`: local/session persistence
- `exporter`: csv/xlsx/pdf export

Feature modules call adapters, not vendor APIs.

## 17. App Shell and Layout Lifecycle
For dashboard/admin style pages, keep app-shell logic centralized:
- viewport/container height recalculation
- sidebar/nav collapse and responsive toggles
- global tooltip/popover initialization
- global resize/orientation handling

Rules:
- isolate shell behavior from business modules
- keep layout recalculation in dedicated function
- avoid hardcoding business selectors in shell layer

## 18. Session and Security-Oriented Frontend Flow
For auth-sensitive actions (logout, token cleanup):
- confirm user intent
- clear local/session storage keys intentionally
- clear auth cookies consistently
- redirect through canonical logout route

Also:
- do not expose tokens or sensitive payloads in logs
- avoid embedding secrets in client-side source

## 19. Domain Formatting Pattern
Business statuses/enums should be converted through dedicated mappers.

Mapper output should include:
- `key` (stable identifier)
- `label` (display text)
- `tone` or `className` (UI presentation hint)

Never scatter hardcoded status strings across render functions.

## 20. Legacy jQuery to Modern JS Migration Rules
If codebase is mixed (jQuery + modern JS), use incremental migration:
- preserve behavior first, then refactor structure
- replace one region/module at a time
- wrap legacy plugin calls in adapter functions
- avoid introducing new direct jQuery dependency in new modules

Target state:
- modern modules use `querySelector` + `addEventListener`
- jQuery remains only in compatibility layer until retired

## 21. Commenting and Readability
- Comment intent, not obvious syntax.
- Keep comments short and technical.
- Remove dead/commented-out blocks before merge unless intentionally retained with reason.
- Keep function size practical; split when responsibility becomes mixed.

## 22. Testing and Review Checklist
Before merge, verify:
1. Data normalization covers null/invalid input.
2. Empty/error/loading states are visible and correct.
3. Event bindings are not duplicated after rerender.
4. Pagination, sorting, and filtering stay in sync.
5. Reusable widgets clean up listeners on destroy.
6. No direct third-party API leakage in feature code.
7. No sensitive data leakage in logs/UI.

## 23. Reusable Module Skeleton
```js
import { dataClient } from '../adapters/dataClient.js';
import { notify } from '../adapters/notify.js';

const state = {
  page: 1,
  pageSize: 20,
  keyword: '',
};

const dom = {
  tableBody: document.querySelector('#table-body'),
  pagination: document.querySelector('#pagination'),
};

const normalizeItem = (raw) => ({
  id: Number(raw?.id || 0),
  name: String(raw?.name || ''),
  qty: Number(raw?.qty || 0),
});

const renderTable = (rows) => {
  const html = rows.length
    ? rows.map((r) => `<tr data-id="${r.id}"><td>${r.name}</td><td>${r.qty}</td></tr>`).join('')
    : '<tr><td colspan="2">No data</td></tr>';

  dom.tableBody.innerHTML = html;
};

const loadData = async () => {
  try {
    const res = await dataClient.get('/items', { page: state.page, pageSize: state.pageSize, keyword: state.keyword });
    const rows = (res?.data || []).map(normalizeItem);
    renderTable(rows);
  } catch (error) {
    notify.error('Unable to load data');
  }
};

const wireEvents = () => {
  dom.tableBody.addEventListener('click', (event) => {
    const row = event.target.closest('tr[data-id]');
    if (!row) return;
    // handle row action
  });
};

const init = () => {
  wireEvents();
  loadData();
};

init();
```

## 24. Governance for Cross-Project Reuse
For team adoption across projects:
- publish this guide as baseline engineering standard
- keep a shared lint/format profile
- provide starter templates for new modules
- maintain a small changelog when guide rules evolve
- define exception process (when and why a team may deviate)
