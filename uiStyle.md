# UI Construction Style Guide (Reusable Across Projects)

## 1. Purpose
This guide defines how to build UI components so they keep a consistent construction style across projects, while allowing different themes and branding.

Focus:
- Layout composition
- Tables
- Charts
- Filters/selects
- Date pickers
- Modals/forms
- Pagination/search

## 2. Core Principles
- Reuse structural blocks, avoid one-off page markup.
- Keep rendering, data mapping, and event handling separated.
- Always provide loading/empty/error states.
- Use predictable HTML anatomy and event delegation.

## 3. Layout Pattern
Recommended page anatomy:
1. Page wrapper
2. Filter/action row
3. Content area (cards/sections)

Recommended section anatomy:
- Section container
  - Section title/header
  - Section body (chart/table/form content)

## 4. Two Composition Modes
1. Analytics mode
- Dense cards
- Multiple charts
- Compact filters
- Sticky table headers

2. Management mode
- Simpler list views
- CRUD actions
- Larger forms
- Table-first workflows

## 5. Table Construction Style
## 5.1 Markup Pattern
```html
<div class="table-responsive h-100">
  <table class="table table-sm table-striped h-100" id="table-1">
    <thead></thead>
    <tbody></tbody>
  </table>
</div>
```

## 5.2 Render Pattern
1. Build `thead` HTML.
2. Build `tbody` HTML.
3. Assign once to DOM.
4. Render explicit empty fallback when no rows.

Example:
```js
const { thead, tbody } = buildTableView(data);
table.querySelector('thead').innerHTML = thead;
table.querySelector('tbody').innerHTML =
  tbody || '<tr><td colspan="20">No data to display</td></tr>';
```

## 5.3 Editable Cell Pattern
- Put edit metadata on cells using `data-*`.
- Delegate click on `tbody`.
- Do not bind per-cell click handlers.

Example:
```js
tbody.addEventListener('click', (event) => {
  const cell = event.target.closest('td[data-action="edit"]');
  if (!cell) return;
  openEditFlow(cell.dataset);
});
```

## 5.4 Seamless Sticky Header Pattern (No `thead` Gap)
Use this pattern to avoid the common "header gap" while scrolling long tables.

Rules:
- Make `thead` sticky as a block (`position: sticky; top: 0`).
- Keep one deterministic border model (`collapse` or `separate`), and keep it consistent with page-level/global table CSS.
- Remove default table bottom spacing (`margin-bottom: 0`).
- Let the wrapper own scroll (`.table-responsive { overflow: auto; }`).
- Use one separator strategy consistently (border or inset-shadow).

Reference CSS:
```css
.table-responsive {
  overflow: auto;
}

.table {
  width: 100%;
  margin-bottom: 0;
  border-collapse: collapse;
  border-spacing: 0;
}

.table thead {
  position: sticky;
  top: 0;
  z-index: 2;
}

.table th,
.table td {
  text-align: center;
  vertical-align: middle;
  border: none;
  box-shadow: inset 0.5px -0.5px 0 var(--table-border-color);
}
```

Production-proven variant (from `me-offline-dashboard` / `stencil.css`):
```css
.table {
  margin-bottom: 0;
  border-right: 1px solid #37567f;
  border-top: 1px solid #37567f;
  border-collapse: separate;
  border-spacing: 0;
}

.table thead {
  position: sticky;
  top: 0;
}

.table th,
.table td {
  border: none;
  box-shadow: inset 0.5px -0.5px 0 #6290b5;
}

.table thead th {
  background-color: #264b8b;
}
```

When to use which:
- Prefer `collapse` variant for isolated pages without strong global overrides.
- Prefer the `stencil` variant when the project already uses global `box-shadow`-based table separators and `thead` sticky behavior.

Troubleshooting if header still looks detached:
- Check parent elements for `overflow: hidden` or transformed contexts.
- Do not mix sticky on both `thead` and each `th` in the same table.
- Ensure `tbody` rows do not add unexpected top margin/padding.
- Audit global overrides first (`style.css`, `style-dashboard.css`, module CSS). Most header-gap bugs are cascade conflicts, not table-render bugs.

## 6. Chart Construction Style
## 6.1 Container Pattern
- One explicit chart container per widget.
- Section title outside the chart.
- Prefer disabled chart title and use section header instead.

## 6.2 Base Options Pattern
Create one shared chart base options function:
- Axis defaults
- Tooltip defaults
- Legend defaults
- Label defaults

Then merge per-chart overrides.

## 6.3 Typical Chart Compositions
- Pie/Donut for distribution
- Column/Stacked column for comparison
- Column + line/spline for trend + rate
- Gauge for KPI thresholds
- Bar for ranking

## 6.4 Granularity Control
Use consistent grain options:
- `day | week | month`

Flow:
- Update filter state
- Refresh only impacted chart(s)

## 7. Filter and Select Construction Style
## 7.1 Filter Bar Rules
- Keep all primary filters in one top bar.
- Use scoped refresh instead of full-page rerender.
- Keep analytics filters compact.

## 7.2 Select Population Pattern
- Render a clear default option first (for example: `Select item`).
- Populate options from data.
- Persist selected values in local page state.

## 7.3 Dropdown Interaction Pattern
On option click:
1. Update local state.
2. Update visible label.
3. Refresh target block.

## 8. Date/Time Picker Pattern
Support two modes:
1. Single datetime for snapshot pages.
2. Range picker for analytics/list pages.

Rule:
- Refresh data only when input state is valid (especially range mode).

## 9. Modal and Form Pattern
## 9.1 Modal Structure
- Fixed header/body/footer anatomy.
- Primary action and close action in footer.
- Forms are self-contained inside modal body.

## 9.2 Validation Lifecycle
1. `preventDefault()`
2. `stopPropagation()`
3. `checkValidity()`
4. Toggle validation class
5. Submit async action
6. On success: reset + close + refresh affected UI

## 9.3 Upload Flow Pattern
- File input + optional sample link
- Validate before submit
- Show feedback
- Refresh only related components

## 10. Tooltip/Inline Editor Pattern
- Use interactive tooltip/popover with embedded mini form.
- Trigger from row/cell interaction.
- Include explicit close control.
- Destroy and recreate tooltips safely when rerendering rows.

## 11. Pagination and Search Pattern
## 11.1 Pagination Behavior
Reusable pager should support:
- Front mode (local array)
- Back mode (remote fetch callback)
- Prev/Next + page numbers + ellipsis
- Optional jump input

## 11.2 Search Behavior
- Debounced input.
- Filter from cached data in front mode.
- Reset to full dataset on empty query.

## 11.3 Row Ordinal Rule
```js
ordinal = index + 1 + pageSize * (currentPage - 1);
```

## 12. Reusable Templates
## 12.1 Section Template
```html
<section class="ui-section">
  <header class="ui-section__title">Title</header>
  <div class="ui-section__body"></div>
</section>
```

## 12.2 Table Section Template
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

## 13. Consistency Rules for Other Projects
If you want consistent style with different branding:
- Keep structural hierarchy unchanged.
- Keep table render/edit flow unchanged.
- Keep chart base + override strategy unchanged.
- Keep filter and modal lifecycles unchanged.
- Keep pagination/search logic unchanged.

You can change:
- Colors
- Fonts
- Radii
- Shadows

Require new pages to compose from these blocks instead of writing page-specific patterns.
