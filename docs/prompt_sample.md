You are a Senior Full-Stack Engineer responsible for analysing the supplied Figma screenshots and implementing the **Header Threat Score Rule – Create/Edit Rule** feature.

## Feature objective

Implement the complete frontend and backend workflow for creating and editing a Header Threat Score Rule.

The rule builder must allow administrators to:

* Create a new rule.
* Edit an existing rule.
* Configure the rule’s basic information.
* Add authentication-check conditions.
* Edit or remove existing conditions.
* Combine conditions using `AND` or `OR`.
* Add nested condition groups where supported by the design.
* Validate the rule before saving.
* Persist the rule through the existing API and data model.
* Load an existing rule into the same form for editing.

## Design inputs

Analyse the following screenshots in workflow order:

### Main workflow

1. `create_a_rule_1.png`
2. `create_a_rule_2.png`
3. `create_a_rule_include_group.png`

### Edit condition

4. `edit-condition.png`

### Add-condition popups

5. `popup_Add_Failed_Check-point-SPF.png`
6. `popup_Add_Failed_Check-point-ARC.png`
7. `popup_Add_Failed_Check-point-DKIM.png`
8. `popup_Add_Failed_Check-point-DMARC.png`

The authentication checks currently supported are:

* SPF
* DKIM
* DMARC
* ARC

Treat the screenshots as the primary UI reference, but also inspect the existing repository for established patterns, existing rule models, API conventions, shared components, validation utilities, design tokens, permissions, and error-handling standards.

Do not invent a separate design system or architecture when an existing project pattern can be reused.

---

# Phase 1: Analyse the design and codebase

Before implementing code, inspect:

* Existing Header Analysis or Header Threat Score functionality.
* Existing create/edit forms.
* Existing condition-builder components.
* Existing modal, drawer, select, input, button, tooltip, and confirmation components.
* Existing API clients and backend routing conventions.
* Existing rule configuration models.
* Existing Global and Zone configuration behaviour.
* Existing authentication and authorization middleware.
* Existing unit, integration, and UI test conventions.

Then produce a concise design analysis containing the following sections.

## 1. Screen-by-screen analysis

For each screenshot, identify:

* Screen purpose.
* Visible fields.
* Available actions.
* Enabled and disabled states.
* Validation messages.
* Navigation behaviour.
* Modal or popup behaviour.
* Condition-group behaviour.
* Differences between create and edit mode.
* Information that must be loaded from the API.
* Information that is only local UI state.

Do not rely only on filenames. Inspect the actual screenshots.

## 2. Reconstructed user workflow

Describe the full workflow, for example:

1. User opens the Create Rule screen.
2. User enters the rule information shown in Step 1.
3. User continues to the condition-building step.
4. User selects the group operator, such as `AND` or `OR`.
5. User selects **Add condition**.
6. User selects ARC, DKIM, DMARC, or SPF.
7. The corresponding popup is displayed.
8. User configures and confirms the condition.
9. The condition appears in the rule builder.
10. User may edit or delete the condition.
11. User may add another condition or a nested group.
12. User reviews and saves the rule.
13. The UI sends a normalized API request.
14. The backend validates and persists the rule.
15. The UI displays success or an actionable error.

Include cancel, back, close-popup, validation-error, API-error, and unsaved-change paths.

## 3. UI state model

Define the frontend state required for:

* Create mode.
* Edit mode.
* Current workflow step.
* Rule metadata.
* Root condition group.
* Nested groups.
* Authentication-check conditions.
* Currently selected condition.
* Add-condition popup.
* Edit-condition popup.
* Form validation.
* Dirty state.
* Loading state.
* Saving state.
* API errors.
* Stale-data or conflicting-update errors, if supported by the existing system.

Clearly separate:

* API data.
* Form state.
* Modal state.
* Derived display state.

## 4. Condition-tree model

Represent rule conditions as a normalized tree rather than hard-coding individual UI rows.

Use an equivalent structure to the following only when it is compatible with the existing codebase:

```ts
type LogicalOperator = 'AND' | 'OR';

type AuthenticationCheck =
  | 'SPF'
  | 'DKIM'
  | 'DMARC'
  | 'ARC';

interface RuleCondition {
  id: string;
  nodeType: 'condition';
  checkType: AuthenticationCheck;
  field: string;
  comparison: string;
  value: string | boolean | string[];
}

interface RuleConditionGroup {
  id: string;
  nodeType: 'group';
  operator: LogicalOperator;
  children: Array<RuleCondition | RuleConditionGroup>;
}
```

The final model must be based on the screenshots and existing backend contracts.

Document:

* Root-group behaviour.
* Nested-group behaviour.
* Maximum supported nesting depth.
* Minimum conditions required per group.
* Whether a group can contain both conditions and child groups.
* Whether empty groups are allowed.
* How IDs are handled before and after persistence.
* How create and edit payloads differ.
* How the UI maps backend data into the condition tree.
* How the tree is serialized back into an API request.

Do not assume unlimited nesting. Derive the supported depth from the design, requirements, configuration, or existing implementation. If it cannot be determined, define a constant and clearly document the assumption.

## 5. Popup mapping

For each authentication-check popup, document:

| Check | Popup fields    | Available options | Default value   | Required fields | Output condition     |
| ----- | --------------- | ----------------- | --------------- | --------------- | -------------------- |
| SPF   | From screenshot | From screenshot   | From screenshot | From screenshot | Normalized condition |
| DKIM  | From screenshot | From screenshot   | From screenshot | From screenshot | Normalized condition |
| DMARC | From screenshot | From screenshot   | From screenshot | From screenshot | Normalized condition |
| ARC   | From screenshot | From screenshot   | From screenshot | From screenshot | Normalized condition |

Do not assume that all four popups use identical fields. Capture their actual differences.

## 6. Validation rules

Derive validation from the screenshots, current requirements, and existing code.

At minimum, evaluate:

* Required rule fields.
* Rule-name length and uniqueness.
* Score format and allowed range, if score is part of the form.
* Classification or threat-level requirements, if included.
* At least one condition before save.
* Empty condition groups.
* Duplicate conditions.
* Unsupported check and comparison combinations.
* Invalid nested groups.
* Maximum nesting depth.
* Missing popup values.
* Saving while an unfinished condition is being edited.
* Invalid API data loaded in edit mode.
* Concurrent or stale updates, if relevant.

Validation must be implemented on both frontend and backend. Frontend validation improves usability, while backend validation remains authoritative.

---

# Phase 2: Define the technical solution

After analysing the screenshots and repository, provide a short implementation plan covering:

## Frontend

Identify:

* Pages or routes to add or update.
* Existing components to reuse.
* New components required.
* Form-state approach.
* Condition-tree rendering strategy.
* Popup architecture.
* Create/edit mode handling.
* API integration.
* Error and success notifications.
* Unsaved-change protection.
* Accessibility considerations.

Prefer reusable components such as:

```text
HeaderThreatScoreRuleForm
RuleDetailsStep
RuleConditionBuilder
RuleConditionGroup
RuleConditionRow
AddConditionMenu
AuthenticationCheckModal
SPFConditionForm
DKIMConditionForm
DMARCConditionForm
ARCConditionForm
RuleFormActions
```

Use the repository’s naming and folder conventions instead of forcing these exact names.

## Backend

Identify:

* Existing endpoint to extend or new endpoints required.
* Request and response schemas.
* Create service method.
* Update service method.
* Get-rule detail method.
* Backend validation.
* Authorization.
* Persistence changes.
* Audit fields.
* Error responses.
* Backward compatibility.

Prefer existing API patterns. Do not introduce new endpoints when an existing endpoint is intended to support this feature.

A possible contract may resemble:

```http
POST /header-threat-score-rules
GET /header-threat-score-rules/{ruleId}
PUT /header-threat-score-rules/{ruleId}
```

However, inspect the repository and use the project’s actual routing conventions.

## API contract

Document the final create and update contracts before coding.

Include:

* Request schema.
* Response schema.
* Validation errors.
* Not-found response.
* Permission-denied response.
* Conflict or stale-update response.
* Server error handling.

Example only:

```json
{
  "name": "Authentication failure rule",
  "description": "Detects failed email authentication checks",
  "score": 80,
  "classification": "High",
  "enabled": true,
  "conditionGroup": {
    "operator": "AND",
    "children": [
      {
        "nodeType": "condition",
        "checkType": "ARC",
        "field": "chainValidation",
        "comparison": "EQUALS",
        "value": "FAIL"
      },
      {
        "nodeType": "group",
        "operator": "OR",
        "children": [
          {
            "nodeType": "condition",
            "checkType": "DKIM",
            "field": "authentication",
            "comparison": "EQUALS",
            "value": "FAIL"
          },
          {
            "nodeType": "condition",
            "checkType": "SPF",
            "field": "authentication",
            "comparison": "EQUALS",
            "value": "FAIL"
          }
        ]
      }
    ]
  }
}
```

Do not copy this payload blindly. Replace it with the structure supported by the screenshots and current project model.

---

# Phase 3: Implement the feature

After completing the analysis and plan, implement the feature in the repository.

Do not stop after producing documentation.

## Implementation requirements

### Create mode

* Render the create workflow according to the screenshots.
* Start with clean default values.
* Allow navigation between workflow steps.
* Preserve entered data when navigating backward and forward.
* Prevent invalid progression where required by the design.
* Submit the normalized create request.
* Show success feedback and navigate according to existing application behaviour.

### Edit mode

* Load the existing rule by ID.
* Map API data into the same form model used by create mode.
* Display existing conditions and nested groups.
* Allow editing the rule metadata.
* Allow editing, adding, and removing conditions.
* Allow changing logical operators where permitted.
* Submit only through the supported update contract.
* Handle missing, deleted, invalid, or inaccessible rules.

### Add condition

* Clicking **Add a condition** must open the appropriate selection experience.
* Selecting SPF, DKIM, DMARC, or ARC must open the corresponding popup.
* Use the exact fields and options shown in the relevant screenshot.
* Confirming the popup must add a valid normalized condition.
* Cancelling or closing the popup must not mutate the rule.
* Reopening the popup must not retain stale values unless the design requires it.

### Edit condition

* Clicking an existing condition must open the edit experience shown in `edit-condition.png`.
* Populate all existing condition values.
* Saving must update the selected condition without changing its position in the tree.
* Cancelling must preserve the original condition.
* Invalid changes must not be applied.

### Condition groups

* Support `AND` and `OR` according to the design.
* Support adding child groups where shown.
* Render nested groups clearly.
* Prevent invalid empty groups.
* Prevent unsupported nesting depth.
* Confirm destructive removal when deleting a non-empty group if this matches existing UX conventions.
* Do not flatten groups during serialization.
* Preserve child order unless the product requirements specify otherwise.

### Error handling

Handle:

* Rule-load failure.
* Save failure.
* Validation failure.
* Permission failure.
* Rule-not-found failure.
* Network timeout.
* Duplicate rule.
* Unsupported condition.
* Stale update or conflict, if the backend supports it.

Do not report a save as successful unless the backend confirms persistence.

### Security

Use the project’s existing authentication and authorization mechanisms.

Ensure:

* Only authorized Global Admin or Zone Admin roles can create or edit rules, according to current project requirements.
* Zone-level users cannot modify rules outside their permitted zone.
* Client-provided ownership, zone, or actor information is not trusted when the server can derive it.
* Backend validation is authoritative.
* Audit actor information is derived from the authenticated identity.
* API error messages do not expose sensitive implementation information.

---

# Phase 4: Testing

Add or update tests following existing project conventions.

## Frontend tests

Cover:

* Initial create state.
* Step navigation.
* Required-field validation.
* Opening each authentication-check popup.
* Adding SPF condition.
* Adding DKIM condition.
* Adding DMARC condition.
* Adding ARC condition.
* Editing a condition.
* Cancelling condition editing.
* Removing a condition.
* Adding an `AND` group.
* Adding an `OR` group.
* Nested-group rendering.
* Maximum-depth validation.
* Empty-group validation.
* Create request payload.
* Edit data mapping.
* Update request payload.
* Loading state.
* API validation errors.
* Save failure.
* Permission failure.
* Successful save.

## Backend tests

Cover:

* Valid create request.
* Valid update request.
* Invalid rule ID.
* Missing required fields.
* Invalid score or classification when applicable.
* Empty condition group.
* Invalid logical operator.
* Unsupported authentication check.
* Invalid check-field combination.
* Invalid comparison.
* Duplicate condition.
* Excessive nesting depth.
* Unauthorized user.
* Cross-zone access.
* Conflict or stale update.
* Correct persistence of nested condition trees.
* Correct API response mapping.

## End-to-end scenarios

At minimum:

1. Create a rule with one SPF condition.
2. Create a rule containing ARC plus a nested DKIM/DMARC group.
3. Edit an existing rule and change a condition.
4. Add a nested group to an existing rule.
5. Remove a condition and save.
6. Attempt to save an empty rule.
7. Attempt to exceed the maximum nesting depth.
8. Handle an API save failure without losing form data.

---

# Expected output

Provide the result in this order:

## A. Design analysis

* Screen-by-screen findings.
* Full create workflow.
* Full edit workflow.
* Popup mappings.
* Condition-tree behaviour.
* Validation rules.
* Assumptions and unresolved design questions.

## B. Technical solution

* Frontend architecture.
* Backend architecture.
* API contract.
* Data-model changes.
* File-by-file implementation plan.
* Security considerations.
* Testing strategy.

## C. Implementation

Implement the required frontend, backend, schemas, migrations, and tests.

For every changed file, explain briefly:

* Why the file was changed.
* What behaviour was added.
* Any compatibility concern.

## D. Verification

Run the relevant commands available in the repository, such as:

```bash
npm run lint
npm run typecheck
npm test
npm run test:e2e
```

Use the actual commands defined by the repository.

Report:

* Commands executed.
* Tests passed.
* Tests failed.
* Lint or type errors.
* Items that could not be verified locally.

## E. Final summary

Provide:

* Implemented workflow.
* Main frontend changes.
* Main backend changes.
* API changes.
* Test coverage.
* Remaining assumptions or follow-up items.

---

# Important implementation constraints

* Analyse the screenshots before coding.
* Inspect the existing repository before deciding the architecture.
* Reuse current components and conventions.
* Do not create duplicate models or API clients.
* Do not hard-code one UI component per rule row.
* Use a recursive or tree-based condition model.
* Keep create and edit flows based on the same reusable form.
* Do not silently discard unsupported backend data.
* Do not flatten nested condition groups.
* Do not trust frontend validation alone.
* Do not make unrelated refactors.
* Do not change existing behaviour outside Header Threat Score Rule unless required.
* Clearly mark any assumption that cannot be verified from the screenshots or repository.
* Prefer a minimal compatible implementation over introducing new frameworks or dependencies.
* Complete the implementation rather than returning only sample code or pseudocode.
