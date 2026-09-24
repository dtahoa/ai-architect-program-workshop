You are a Senior Full-Stack Engineer responsible for analysing the supplied Figma screenshots and implementing the **Header Threat Score Rules – List Page** for Global Admin and Zone Admin users.

The implementation must follow the existing project architecture, UI patterns, API conventions, authorization model, and testing standards.

# Feature objective

Implement the Header Threat Score Rules list page, including:

* Global Admin rule management.
* Zone Admin visibility of Global default rules.
* Zone-level activation and deactivation of Global default rules.
* Zone-created rules.
* Rule status and score presentation.
* Filters.
* Tooltips.
* Rule-details navigation.
* Role-based actions.
* Loading, empty, and error states.
* Concurrent edit and deleted-rule conflict behaviour.

# Figma inputs

Analyse the actual contents of these screenshots:

1. `Global Zone.png`
2. `Global Zone-Tooltips.png`
3. `score-chip.png`
4. `Zone Admin-threat-score-rules-list-Default rule from Global.png`
5. `Zone Admin-threat-score-rules-list-Default Rule from Global-Filter Dropdown.png`

Do not rely only on the filenames. Inspect the screenshots carefully and derive:

* Page structure.
* Table columns.
* Labels.
* Buttons.
* Chips.
* Tooltips.
* Filters.
* Rule-source indicators.
* Status controls.
* Available actions.
* Disabled states.
* Role-specific differences.
* Navigation behaviour.

Treat the Figma screenshots as the primary UI reference, but inspect the repository before deciding the final technical approach.

# Business rules

## Global Admin

Global Admin creates and manages the default Header Threat Score Rules.

These Global default rules apply to zones that enable the Threat Score Rules feature.

Global Admin must be able to:

* View Global default rules.
* Create a new Global default rule.
* View rule details.
* Edit a Global default rule.
* Activate or deactivate a Global default rule.
* Delete a Global default rule, when permitted by the existing product behaviour.
* Filter the Global rules list.
* See the configured threat score and classification.
* See relevant tooltips defined in the Figma design.

Global Admin must not see zone-specific actions that are only relevant to Zone Admin users.

## Zone Admin

When a Zone Admin enables Threat Score Rules for their zone, Global default rules must appear in the Zone Admin list page.

Global default rules will run for that zone unless the Zone Admin disables them.

For a Global default rule, Zone Admin:

* Can view the rule in the list.
* Can view rule details.
* Can activate the rule for their zone.
* Can deactivate the rule for their zone.
* Cannot edit the Global rule definition.
* Cannot delete the Global rule.
* Cannot modify its score, classification, conditions, name, or description.

The Zone Admin activation state is a zone-level override and must not change the Global rule itself.

For example:

```text
Global rule status: Active
Zone A override: Active
Zone B override: Inactive
Zone C override: No override, use inherited state
```

Inspect the existing configuration and inheritance model to determine whether the implementation stores:

* An explicit zone-level active/inactive override.
* Only disabled Global rule IDs.
* A copied zone-level rule.
* Another existing representation.

Do not duplicate Global rules into zone-owned rules unless that is already the established architecture.

## Zone-created rules

If the current application supports Zone Admin-created rules, distinguish them from Global default rules.

Zone Admin may be able to:

* Create zone-specific rules.
* Edit zone-specific rules.
* Delete zone-specific rules.
* Activate or deactivate zone-specific rules.

Confirm this behaviour from the existing codebase and Figma design before implementing it.

## Default Rule actions

For a Global default rule displayed in the Zone Admin list:

* Do not show an Edit action.
* Do not show a Delete action.
* Do not show the normal action menu if it only contains edit or delete actions.
* Allow navigation to a read-only rule-details page.
* Allow Active/Inactive changes through the status control shown in the Figma design.

Interpret “No action for Default Rule” as no edit/delete action menu. The zone-level Active/Inactive control and View Details navigation are still allowed.

# Rule-source terminology

Determine the exact wording used by the design.

Possible concepts include:

* Default Rule.
* Global Rule.
* Inherited from Global.
* Zone Rule.
* Custom Rule.

Use the wording shown in the Figma design and existing product terminology.

Do not introduce multiple labels for the same concept.

# Phase 1: Analyse the design and codebase

Before implementing code, inspect:

* Existing Header Threat Score Rules pages.
* Existing Header Analysis configuration.
* Existing Global and Zone inheritance logic.
* Existing feature-enable settings.
* Existing list or data-table components.
* Existing filter-dropdown components.
* Existing tooltip components.
* Existing chips and badges.
* Existing active/inactive toggles.
* Existing overflow action menus.
* Existing pagination and sorting behaviour.
* Existing API clients.
* Existing backend rule APIs.
* Existing authorization middleware.
* Existing zone-scoping logic.
* Existing toast and error-handling utilities.
* Existing frontend and backend tests.

Then provide the following analysis.

## 1. Screen-by-screen analysis

For each supplied screenshot, document:

* User role represented.
* Page purpose.
* Page title and description.
* Primary actions.
* Table structure.
* Table columns.
* Row content.
* Status controls.
* Rule-source indicators.
* Score chips.
* Threat-level or classification display.
* Tooltips.
* Filter controls.
* Dropdown options.
* Row actions.
* Empty space and responsive behaviour.
* Differences from the other screenshots.

## 2. Global Admin workflow

Reconstruct the workflow, including:

1. Global Admin opens the rules list.
2. The application loads Global default rules.
3. The admin views rule name, conditions, score, classification, status, or other fields shown by Figma.
4. The admin applies filters.
5. The admin opens rule details.
6. The admin creates, edits, activates, deactivates, or deletes a rule where supported.
7. The list refreshes or updates after a successful operation.
8. The UI handles API failures without displaying an incorrect state.

Include loading, empty, error, permission-denied, and no-filter-results states.

## 3. Zone Admin workflow

Reconstruct the workflow, including:

1. Zone Admin enables Threat Score Rules for the zone.
2. Global default rules appear in the Zone Admin list.
3. Global default rules are visually distinguishable from zone-created rules.
4. Zone Admin can open a Global default rule in read-only mode.
5. Zone Admin can activate or deactivate a Global default rule for their zone.
6. Zone Admin cannot edit or delete a Global default rule.
7. Zone Admin can filter rules, including filtering by rule source if shown in Figma.
8. Zone-level changes must not update the Global rule definition or other zones.

Also analyse the expected list state when Threat Score Rules is disabled for the zone.

## 4. Table model

Document every table column from the screenshots.

For each column, identify:

| Column         | Data source             | Display format | Sortable            | Filterable          | Role difference      |
| -------------- | ----------------------- | -------------- | ------------------- | ------------------- | -------------------- |
| Rule name      | API                     | Text/link      | Confirm from design | Confirm from design | Global/Zone          |
| Source         | API/derived             | Chip or label  | Confirm             | Confirm             | Primarily Zone Admin |
| Score          | API                     | Score chip     | Confirm             | Confirm             | Same or different    |
| Classification | API                     | Chip/text      | Confirm             | Confirm             | Same or different    |
| Status         | Global or zone override | Toggle/chip    | Confirm             | Confirm             | Role-specific        |
| Actions        | Permission-derived      | Menu/buttons   | No                  | No                  | Role-specific        |

Replace this example with the actual columns shown in Figma.

## 5. Score-chip analysis

Use `score-chip.png` to determine:

* Chip text.
* Score format.
* Threat classification format.
* Whether score and classification appear in one chip or separately.
* Tooltip behaviour.
* Colour or design-token mapping.
* Accessibility text.
* Handling of missing or invalid score values.
* Handling of long values.
* Consistency with existing classification components.

Do not hard-code raw colour values if the project uses design tokens.

The score shown in the list is the rule’s configured score, not necessarily the final email score.

Where relevant, the final threat score calculation is:

```text
max(base_score, highest_matching_rule_score)
```

Do not add scores together.

## 6. Tooltip analysis

Use `Global Zone-Tooltips.png` and other screenshots to document:

* Which elements have tooltips.
* Exact or intended tooltip content.
* Tooltip trigger.
* Tooltip placement.
* Whether the tooltip appears on icon hover, chip hover, or label hover.
* Keyboard accessibility.
* Mobile or touch behaviour.
* Whether tooltip content differs by role.

Reuse the existing tooltip component.

## 7. Filter analysis

Use:

* `Global Zone.png`
* `Zone Admin-threat-score-rules-list-Default Rule from Global-Filter Dropdown.png`

Document:

* Filter button or dropdown label.
* Available filter categories.
* Available options.
* Default selection.
* Single-select or multi-select behaviour.
* Apply, reset, or clear behaviour.
* Active-filter indicators.
* Whether filters are client-side or server-side.
* How filters interact with pagination.
* How no-result states are displayed.

Potential filters may include:

* Rule status.
* Rule source.
* Threat classification.
* Score.
* Active/Inactive.
* Default/Custom.

Only implement filters supported by the Figma design or existing requirements.

## 8. Role and action matrix

Produce a final permission matrix before coding.

Example:

| Action            | Global Admin: Global rule | Zone Admin: Global default rule | Zone Admin: Zone rule |
| ----------------- | ------------------------: | ------------------------------: | --------------------: |
| View list item    |                       Yes |                             Yes |                   Yes |
| View details      |                       Yes |                  Yes, read-only |                   Yes |
| Create            |                       Yes |                  No global rule |               Confirm |
| Edit definition   |                       Yes |                              No |               Confirm |
| Delete            |         Yes, if supported |                              No |               Confirm |
| Activate/Inactive |             Yes, globally |         Yes, zone override only |               Confirm |
| View action menu  |                       Yes |                              No |               Confirm |

Update the matrix based on the actual repository and Figma behaviour.

Frontend hiding or disabling actions is not sufficient. Backend permission checks must remain authoritative.

# Phase 2: Define the data and API behaviour

## List response model

Inspect the existing API before designing a new one.

The list response must provide enough data for the frontend to determine:

* Rule ID.
* Rule name.
* Description or summary if shown.
* Configured score.
* Threat classification.
* Global active state.
* Zone active state.
* Effective active state.
* Rule source.
* Whether the rule is inherited.
* Whether the current user can edit.
* Whether the current user can delete.
* Whether the current user can toggle.
* Created and updated metadata if displayed.
* Pagination metadata if used.

Prefer deriving permissions on the server or from authenticated role and ownership rather than trusting values supplied by the client.

A possible response shape is:

```ts
interface ThreatScoreRuleListItem {
  id: string;
  name: string;
  score: number;
  classification: string;
  source: 'GLOBAL' | 'ZONE';
  inherited: boolean;

  globalActive: boolean;
  zoneOverride?: 'ACTIVE' | 'INACTIVE' | null;
  effectiveActive: boolean;

  permissions: {
    canView: boolean;
    canEdit: boolean;
    canDelete: boolean;
    canToggle: boolean;
  };

  updatedAt?: string;
}
```

Do not use this shape blindly. Adapt it to the existing backend model.

## Effective status

Clearly define how the displayed status is calculated.

For a Global Admin viewing a Global rule:

```text
effective status = Global rule active state
```

For a Zone Admin viewing a Global default rule:

```text
effective status = zone-level override when present
otherwise use the inherited/default behaviour defined by the current product model
```

Confirm how the feature-level enabled setting affects this calculation.

Do not let the UI infer business-critical status from incomplete data when the backend can return the effective state.

## API operations

Inspect and reuse existing endpoints where possible.

Possible operations include:

```http
GET /header-threat-score-rules
GET /header-threat-score-rules/{ruleId}
PATCH /header-threat-score-rules/{ruleId}/status
DELETE /header-threat-score-rules/{ruleId}

GET /zones/{zoneId}/header-threat-score-rules
PATCH /zones/{zoneId}/header-threat-score-rules/{ruleId}/status
```

These are examples only.

Use the actual project naming and routing conventions.

Document:

* Global list API.
* Zone list API.
* List query parameters.
* Filter query parameters.
* Pagination.
* Sorting.
* Status-update API.
* Details API.
* Delete API.
* Permission responses.
* Not-found responses.
* Error responses.

## Zone-level activation API

Zone Admin activation or deactivation of a Global default rule must:

* Update only the current zone’s configuration.
* Not update the Global rule.
* Not affect any other zone.
* Validate that the Global rule still exists.
* Validate the authenticated user’s zone access.
* Return the new effective status.
* Be auditable if the project already supports auditing.

Do not send `zoneId`, user identity, or actor information from the UI when the backend can derive it from the authenticated session.

# Phase 3: Edit and delete conflict behaviour

Follow the existing product behaviour.

## Case 1: Two admins edit the same rule

Scenario:

1. Admin A opens a rule.
2. Admin B opens the same rule.
3. Admin A saves changes.
4. Admin B saves different changes later.

Expected behaviour:

* Last writer wins.
* Do not show a conflict warning.
* Do not introduce optimistic locking or version-checking only for this feature.
* The later successful save replaces the earlier values.
* Follow the behaviour of other existing features.

Do not add version or revision validation unless it is already enforced by the current shared backend framework.

## Case 2: Rule deleted while another admin is editing

Scenario:

1. Admin A opens a rule for editing.
2. Admin B deletes the same rule.
3. Admin A attempts to save.

Expected behaviour:

* The backend returns a not-found or equivalent failure.
* Admin A’s save must not recreate the deleted rule.
* The frontend shows the standard toast:

```text
Something went wrong
```

* Follow the existing application’s generic error-toast behaviour.
* Do not display a successful save notification.
* Preserve form data unless the existing application redirects automatically.
* If existing convention redirects after this error, follow that convention.
* Refresh or invalidate the list/details cache as appropriate.

The list page must also safely handle a delete action when the rule has already been deleted by another admin.

# Phase 4: Frontend implementation

Implement the list page using existing shared components and patterns.

Potential component structure:

```text
HeaderThreatScoreRulesPage
ThreatScoreRuleListHeader
ThreatScoreRuleFilters
ThreatScoreRuleTable
ThreatScoreRuleRow
RuleSourceChip
ThreatScoreChip
ThreatClassificationChip
RuleStatusControl
RuleActionsMenu
RuleListEmptyState
RuleListErrorState
```

Use repository naming and folder conventions rather than forcing these names.

## Page states

Implement:

* Initial loading.
* Loaded list.
* Empty rule list.
* No results after filtering.
* API error.
* Permission denied.
* Status update in progress.
* Delete in progress.
* Partial row-operation error.
* Disabled feature state where applicable.

## Optimistic UI

Inspect existing project behaviour before using optimistic updates.

For Active/Inactive changes:

* Disable duplicate interactions while the request is pending.
* Show loading state according to existing UI conventions.
* Update the UI only after API success unless the existing application has a safe optimistic-update pattern.
* Restore the previous value if an optimistic update fails.
* Show the standard error toast on failure.

Do not allow a Zone Admin’s toggle to temporarily appear as though the Global rule was changed globally.

## Navigation

Implement navigation according to existing routes:

* Global Admin rule details.
* Global Admin create rule.
* Global Admin edit rule.
* Zone Admin read-only Global default rule details.
* Zone Admin zone-rule details where supported.

Ensure route-level authorization matches the list-page action visibility.

A Zone Admin must not be able to access the Global edit route by manually entering the URL.

## Tooltips

Implement all tooltips shown in Figma using the shared tooltip component.

Tooltips must:

* Work with mouse and keyboard.
* Have accessible labels.
* Not block row navigation.
* Not cause layout shifts.
* Use the text from the design or existing product copy.

## Filters

Implement the Figma filter dropdown exactly where possible.

Ensure:

* Filters can be opened and closed.
* Selected options are visible.
* Clear/reset works.
* Dropdown closes according to current application patterns.
* Filters persist during pagination if server-side.
* Filters reset pagination to the first page.
* Filter values are sent using the API’s supported enum values.
* Display labels are separated from backend enum values.

# Phase 5: Backend implementation

Implement or extend backend support for:

* Listing Global rules.
* Listing effective Zone rules.
* Distinguishing Global and Zone sources.
* Returning effective status.
* Filtering.
* Pagination and sorting if used.
* Updating Global active state.
* Updating zone-level active/inactive overrides.
* Deleting Global rules where permitted.
* Authorization and zone scoping.
* Generic conflict/error handling.
* Audit logging if part of existing standards.

## Authorization

Enforce:

* Only authenticated users can access the endpoints.
* Only Global Admin can create, edit, or delete Global rules.
* Zone Admin can only access their permitted zone.
* Zone Admin cannot edit or delete Global rule definitions.
* Zone Admin can only change the zone-level activation state of an inherited Global rule.
* Users cannot manipulate another zone by changing request parameters.
* Permission checks occur on the server even when actions are hidden in the UI.

## Data integrity

Ensure:

* Global rule deletion does not leave invalid zone references.
* Existing zone overrides for a deleted Global rule are removed, ignored, or handled according to the current data model.
* A Zone Admin cannot create an override for a non-existent rule.
* Duplicate override records cannot be created.
* List responses do not return deleted rules.
* Effective status calculation is deterministic.

# Phase 6: Testing

Add or update tests using existing test conventions.

## Frontend tests

Cover:

* Global Admin list rendering.
* Zone Admin list rendering.
* Global default rule indicator.
* Zone-specific rule indicator, if supported.
* Score chip rendering.
* Classification rendering.
* Tooltips.
* Global Admin action menu.
* Zone Admin has no edit/delete action for a Global default rule.
* Zone Admin can view Global rule details.
* Zone Admin can activate a Global default rule.
* Zone Admin can deactivate a Global default rule.
* Status request failure restores or preserves the previous state.
* Filters open and close.
* Each filter option works.
* Clear filters.
* No-filter-results state.
* Loading state.
* Empty state.
* API error state.
* Delete success.
* Delete failure.
* Deleted-by-another-admin failure.
* Generic `Something went wrong` toast.
* Route authorization.
* Pagination and sorting where supported.

## Backend tests

Cover:

* Global Admin list request.
* Zone Admin list request.
* Global default rules included for an enabled zone.
* Effective status without a zone override.
* Effective status with active override.
* Effective status with inactive override.
* Global rule and zone-rule distinction.
* Global Admin can update a Global rule’s status.
* Zone Admin cannot update the Global rule definition.
* Zone Admin can update only their zone-level status.
* Cross-zone request rejected.
* Unauthenticated request rejected.
* Unsupported filters rejected or safely ignored according to project standards.
* Deleted rules excluded.
* Zone override for a deleted rule handled safely.
* Last-writer-wins update behaviour.
* Save after deletion returns not found.
* Delete after another delete returns the expected error.
* Pagination and sorting where supported.

## End-to-end scenarios

At minimum:

1. Global Admin opens the list and views Global rules.
2. Global Admin filters rules by status or classification.
3. Global Admin opens a rule-details page.
4. Global Admin creates a rule and sees it in the list.
5. Global Admin edits a rule and sees the updated values.
6. Global Admin activates or deactivates a rule.
7. Global Admin deletes a rule.
8. Zone Admin opens the list and sees inherited Global default rules.
9. Zone Admin views a Global default rule in read-only mode.
10. Zone Admin deactivates a Global default rule for their zone.
11. Another zone remains unaffected.
12. Zone Admin cannot edit or delete a Global default rule.
13. Admin A saves after Admin B updates the same rule and the last save wins.
14. Admin A attempts to save after Admin B deletes the rule and sees `Something went wrong`.

# Expected output

Provide your work in the following order.

## A. Design analysis

Include:

* Screen-by-screen findings.
* Global Admin workflow.
* Zone Admin workflow.
* Table columns.
* Score-chip behaviour.
* Tooltip behaviour.
* Filter behaviour.
* Rule-source presentation.
* Action and permission matrix.
* Loading, empty, and error states.
* Assumptions or unclear Figma details.

## B. Current codebase analysis

Include:

* Relevant existing pages.
* Relevant components.
* Existing API endpoints.
* Existing data models.
* Existing role checks.
* Existing Global/Zone inheritance patterns.
* Existing toast and conflict behaviour.
* Components and APIs that can be reused.

## C. Technical solution

Include:

* Frontend component design.
* Frontend state model.
* API contract.
* Effective-status calculation.
* Zone-override persistence.
* Backend authorization.
* Filter implementation.
* Cache invalidation or data-refresh strategy.
* File-by-file implementation plan.

## D. Implementation

Implement the frontend, backend, API integration, schemas, persistence changes, and tests.

For every changed file, briefly state:

* Why it changed.
* What behaviour was added.
* Any compatibility concern.

## E. Verification

Run the actual validation commands defined in the repository, such as:

```bash
npm run lint
npm run typecheck
npm test
npm run test:e2e
```

For a multi-project repository, run the relevant frontend and backend commands.

Report:

* Commands executed.
* Passed tests.
* Failed tests.
* Lint errors.
* Type errors.
* Items that could not be verified.

## F. Final summary

Summarize:

* Implemented list-page behaviour.
* Global Admin capabilities.
* Zone Admin capabilities.
* Rule inheritance behaviour.
* Filtering and tooltip support.
* Score-chip behaviour.
* API and data-model changes.
* Authorization controls.
* Conflict handling.
* Test coverage.
* Remaining assumptions.

# Important constraints

* Inspect the screenshots before coding.
* Inspect the existing repository before defining architecture.
* Reuse current table, filter, chip, tooltip, toggle, modal, menu, toast, and API patterns.
* Do not create a second rules architecture.
* Do not copy Global rules into zone-owned rules unless already required by the data model.
* Do not let Zone Admin change the Global rule definition.
* Zone activation changes must affect only the current zone.
* Do not show Edit or Delete actions for inherited Global default rules.
* View Details and zone-level Active/Inactive controls remain available.
* Apply role checks in both frontend and backend.
* Use last-writer-wins behaviour for concurrent edits.
* Do not add a conflict indicator for concurrent edits.
* A save against a deleted rule must fail and show `Something went wrong`.
* Do not recreate a deleted rule during update.
* Do not implement unrelated refactoring.
* Do not introduce new dependencies without clear necessity.
* Clearly document assumptions where Figma or existing code is unclear.
* Complete the implementation rather than returning only analysis, pseudocode, or sample components.
