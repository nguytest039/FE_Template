# Reusable UI Construction Playbook (Cross-Project)

## 1. Purpose
This document is a **portable adoption plan** for teams that want to reuse frontend UI construction patterns across different projects, domains, and constraints.

It focuses on:
- Structure and composition patterns
- Reusable component boundaries
- Consistent interaction behavior
- Gradual adoption strategy

It does **not** assume a specific product domain, design theme, or framework.

## 2. What This Is (and Is Not)
This is:
- A plan to standardize UI construction approach.
- A guide for building reusable frontend primitives.
- A migration strategy for mixed old/new codebases.

This is not:
- A strict visual design system (colors/branding/tokens).
- A fixed component library tied to one stack.
- A one-time rewrite plan.

## 3. Reusable Construction Layers
Use these layers as a default starting point:

1. `layout`:
- Page shell
- Section/card wrapper
- Header + actions + body pattern

2. `table`:
- Table render pipeline
- Empty/error/loading states
- Editable cell conventions
- Row action conventions

3. `chart`:
- Shared baseline chart options
- Chart factory/adapters
- Time-grain controls and update hooks

4. `filter`:
- Filter state controller
- Control bindings (select/date/search)
- Scoped refresh orchestration

5. `modal-form`:
- Modal lifecycle
- Validation workflow
- Async submit hooks

6. `pagination-search`:
- Front/back mode pagination
- Debounced quick search
- Stable row numbering

## 4. Standard Contracts (Portable APIs)
Define contracts before implementation to avoid framework lock-in.

## 4.1 Layout Contract
- `createPageShell(config)`
- `createSection({ title, actions, content })`

Expectation:
- Consistent section anatomy regardless of visual theme.

## 4.2 Table Contract
- `renderTable({ tableEl, columns, rows, states })`
- `bindTableActions({ tableEl, handlers })`

Expectation:
- One render path for all states (normal/empty/error/loading).
- Event delegation for row/cell actions.

## 4.3 Chart Contract
- `createChartBaseOptions(context)`
- `renderChart(container, options)`
- `updateChartGranularity(grain)`

Expectation:
- Shared behavior defaults, local overrides only where needed.

## 4.4 Filter Contract
- `createFilterStore(initialState)`
- `bindFilters({ store, controls, onChange })`
- `getFilterState()`

Expectation:
- Single source of truth for active filters.
- Scoped updates (refresh only affected blocks).

## 4.5 Modal Form Contract
- `createModalForm({ modalEl, formEl, submit })`
- `validate(formEl)`
- `reset(formEl)`

Expectation:
- Uniform validation/submit lifecycle across all forms.

## 4.6 Pagination Contract
- `createPager({ mode, pageSize, source, fetch, renderRow })`
- `loadPage(pageNo)`
- `attachSearch(inputEl, predicate)`

Expectation:
- Same UX in every listing page, regardless of data source type.

## 5. Interaction Rules That Transfer Well
Apply these consistently across projects:
- Always include explicit empty states.
- Keep loading and error states visible and recoverable.
- Use event delegation for dynamic tables/lists.
- Use deterministic row ordinal across pages.
- Keep filter updates predictable (state change -> targeted refresh).
- Keep modal form lifecycle identical across features.

## 6. Adaptation Matrix (Different Project Constraints)
## 6.1 Small Team / Fast Delivery
- Start with `table`, `modal-form`, `pagination-search`.
- Keep API surface minimal.
- Prefer utility modules over large abstractions.

## 6.2 Medium Team / Multiple Features
- Add `layout` and `filter` controllers.
- Introduce versioned contracts.
- Add usage examples and lightweight lint rules.

## 6.3 Large Team / Multi-App Ecosystem
- Full layer model.
- Package as internal library with semantic versioning.
- Add compatibility policy and migration guides.

## 7. Rollout Strategy (No Big-Bang Rewrite)
## Phase 1: Extract Essentials
- Extract shared patterns from 1-2 high-traffic pages.
- Build first contracts for table/modal/pagination.

## Phase 2: Validate and Harden
- Migrate 2-3 additional pages.
- Resolve API gaps discovered during migration.
- Add basic tests for core behaviors.

## Phase 3: Scale and Govern
- Enforce shared patterns in PR review.
- Add changelog + deprecation policy.
- Require new pages to use reusable layers by default.

## 8. Definition of Done (Per Reusable Layer)
A layer is “ready” only when:
- Public API is documented.
- At least one real page uses it.
- Empty/error/loading behaviors are standardized.
- Basic tests exist for core behavior.
- Example usage exists for onboarding.

## 9. Governance and Team Rules
- New UI work should consume existing reusable layers first.
- If a new pattern appears twice, extract it.
- Avoid page-local utility duplication.
- Any breaking API change requires migration notes.

## 10. Practical Starter Deliverables
If you want immediate adoption:
1. `uiConstructionChecklist.md`:
- Review checklist for consistency.
2. `uiPatternsExamples.md`:
- Copy-paste templates for common page types.
3. `uiContracts.md`:
- Stable API contract definitions for each layer.

These three documents are usually enough to align multiple projects quickly.
