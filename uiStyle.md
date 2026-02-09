# UI Construction Style Guide (Reusable Across Projects)

## 1. Purpose
This guide defines **how to build UI components** so they follow the same structural style as this project, while allowing different color themes and branding.

Focus areas:
- Page layout and composition
- Tables
- Charts
- Selects and filter bars
- Date/time pickers
- Modals and forms
- Pagination, search, and inline editing

This is intentionally **construction-first** (structure + behavior), not visual token-first (color palette).

## 2. Core Construction Principles
- Build pages from reusable layout blocks, not one-off markup.
- Keep component structure stable; allow theme values (colors, spacing, fonts) to vary by project.
- Separate concerns:
  - Data loading/transformation
  - Rendering
  - Interaction handlers
- Use predictable component IDs/classes and event wiring patterns.
- Always support empty states and error states.

## 3. Page Layout Pattern
Recommended shell:
1. Page wrapper
2. Filter/action row at the top
3. Main content area (cards/sections)

Recommended section pattern:
- `Section/Card Container`
  - `Section Title`
  - `Section Body`

Behavior:
- The content area should calculate height based on whether the filter bar exists.
- Each section body should fully own its scroll behavior (chart/table internal scroll when needed).

## 4. Two UI Modes (Recommended)
Use two composition modes, regardless of theme colors:

1. Analytics/Dashboard mode
- Dense cards
- Many charts + compact controls
- Sticky table headers inside panel bodies
- Emphasis on comparison and trend readability

2. Management/List mode
- Simpler white/light surfaces (or equivalent neutral surfaces)
- Larger forms and row actions
- Stronger CRUD affordances (upload, delete, edit, export)

Important:
- Keep these modes separated in class naming and layout utilities.

## 5. Table Construction Style
## 5.1 Markup Pattern
```html
<div class="table-responsive">
  <table class="table table-sm" id="tbl-entity">
    <thead></thead>
    <tbody></tbody>
  </table>
</div>
```

## 5.2 Rendering Pattern
Use a deterministic render flow:
1. Build `thead` HTML string (or DOM fragment)
2. Build `tbody` HTML string
3. Assign once to DOM
4. Render fallback row when empty

Example:
```js
const { thead, tbody } = buildTableView(data);
table.querySelector('thead').innerHTML = thead;
table.querySelector('tbody').innerHTML =
  tbody || '<tr><td colspan="20">No data to display</td></tr>';
```

## 5.3 Interaction Pattern for Editable Cells
- Mark interactive cells with `data-action` and `data-*` payload.
- Use event delegation on `<tbody>`.
- Never bind one click handler per cell.

Example:
```js
tbody.addEventListener('click', (event) => {
  const cell = event.target.closest('td[data-action="edit"]');
  if (!cell) return;
  openEditFlow(cell.dataset);
});
```

## 5.4 Table UX Rules
- Keep headers sticky in scrollable bodies.
- Keep numeric columns aligned consistently.
- Provide one explicit empty state message.
- Keep row action buttons compact and status-aware.

## 6. Chart Construction Style
## 6.1 Container Pattern
- Every chart gets its own explicit container ID.
- Section title is outside the chart; chart title is usually disabled.

## 6.2 Baseline Chart Options
Define one base configuration function and reuse it for all charts:
- Shared axis styling
- Shared tooltip behavior
- Shared label/data label style
- Shared legend and credits defaults

Then apply per-chart overrides.

## 6.3 Chart Types and Composition
Commonly used composition:
- Pie/Donut for distribution summaries
- Column/Stacked column for category comparison
- Column + line/spline for trend + rate overlays
- Gauge for KPI thresholds
- Bar for ranking

## 6.4 Time Grain Controls
Use a standardized control for chart granularity:
- `day | week | month`
- User selection updates local state and redraws only impacted charts

## 7. Filter and Select Construction Style
## 7.1 Filter Bar Rules
- Place filter controls in a dedicated top row.
- Keep controls compact in analytics mode.
- Trigger scoped reloads (only necessary chart/table blocks).

## 7.2 Select Population Pattern
- Render a clear default option first (e.g., “Select item”).
- Populate options from data sources.
- Store current selection in local view state.

## 7.3 Dropdown Interaction Pattern
- Use a single visual pattern for dropdown triggers in all sections.
- For each option click:
  - update state
  - update trigger label
  - refresh target component

## 8. Date/Time Picker Construction Style
Support two standard picker modes:

1. Single datetime
- Used for snapshot-like pages
- Change immediately refreshes dependent widgets

2. Range picker
- Used for analytics/list pages
- Reload only when both start and end dates are valid

UX rule:
- Keep date formatting consistent across query params, labels, and table cells.

## 9. Modal and Form Construction Style
## 9.1 Modal Pattern
- Header, body, footer structure is fixed.
- Footer hosts primary submit and secondary cancel/close actions.
- Forms inside modal should be self-contained.

## 9.2 Validation Pattern
Always use the same submit flow:
1. `preventDefault()`
2. `stopPropagation()`
3. `checkValidity()`
4. toggle validation class
5. submit async action
6. on success: reset validation + close modal + refresh affected UI

## 9.3 Upload Pattern
- File input + sample template link (optional)
- Validate required file before submit
- Show success/failure feedback
- Refresh only affected data block(s)

## 10. Tooltip/Inline Editor Pattern
When inline editing is needed:
- Use interactive tooltip/popover with embedded mini-form.
- Keep row click as the trigger.
- Include explicit close action inside tooltip content.
- Destroy/recreate tooltip safely when rows rerender.

## 11. Pagination and Quick Search Construction Style
## 11.1 Pagination Behavior
A reusable pagination unit should support:
- Front mode (local data)
- Back mode (remote callback)
- Prev/Next
- Number buttons
- Ellipsis for hidden ranges
- Optional jump-to-page input

## 11.2 Quick Search Behavior
- Bind search input with debounce.
- Filter against cached source data (front mode).
- Reset to full source on empty search text.

## 11.3 Row Numbering Rule
- Keep absolute row ordinal stable across pages:
  - `ordinal = index + 1 + pageSize * (currentPage - 1)`

## 12. Reusable Section Templates
## 12.1 Analytics Section Template
```html
<section class="ui-section">
  <header class="ui-section__title">Section Title</header>
  <div class="ui-section__body">
    <div id="chart-or-table-container"></div>
  </div>
</section>
```

## 12.2 Data Table Section Template
```html
<section class="ui-section">
  <header class="ui-section__title">List</header>
  <div class="ui-section__actions"><!-- search/export/upload --></div>
  <div class="ui-section__body">
    <div class="table-responsive">
      <table class="table" id="tbl-list">
        <thead></thead>
        <tbody></tbody>
      </table>
    </div>
  </div>
</section>
```

## 12.3 Filter Bar Template
```html
<div class="ui-filter-bar">
  <select id="filter-a"></select>
  <select id="filter-b"></select>
  <input type="text" id="time-range" />
  <button id="btn-search">Search</button>
</div>
```

## 13. Rules to Keep Other Projects Visually Consistent in Structure
If your target is “same UI style, different brand colors,” enforce these:
- Keep the same component hierarchy (wrapper -> title -> body).
- Keep the same table build/render/edit patterns.
- Keep the same chart baseline + per-chart override strategy.
- Keep the same filter bar behavior and update flow.
- Keep the same modal validation lifecycle.
- Keep the same pagination/search logic.
- Keep status-aware action buttons in list tables.

You can safely swap:
- Color tokens
- Typography set
- Border radii
- Shadows

As long as construction patterns above remain unchanged.

## 14. Recommended Next Step for Team Adoption
Create a small UI kit package with:
1. Layout primitives
2. Table renderer utilities
3. Chart base options factory
4. Filter bar controller
5. Modal form utilities
6. Pagination controller

Then require all new pages to compose from these blocks instead of writing custom page-specific patterns.
