Act as a panel of senior AI architecture trainers and technical editors.

Review and update:

artifacts/
presentation/

Primary requirement source:

docs/req_analysis_vi.md

Assigned topic:

Variant B — AI-Retrofit.

Objectives:

1. Verify that every mandatory requirement in docs/req_analysis_vi.md is
covered.
2. Find rejection-level architecture issues.
3. Update artifacts to be shorter and easier to defend.
4. Preserve all quantitative targets, calculations, failure behavior,
monitoring thresholds, owners, and production gates.
5. Ensure the final presentation has no more than eight slides and fits
within seven minutes.

Do not praise the work.

First create:

reviews/requirement-coverage-matrix.md

Map each requirement to:

- Artifact file and section
- Presentation slide
- Coverage status
- Required correction

Then review the design for:

- 04:00 deadline and MERLIN reserve
- 27M forecasts/day and 6x peak
- 78% inventory accuracy
- MERLIN fail-open
- POS and EDI immutability
- Supported MERLIN extension points
- Kill-switch realism
- Shadow-assist rollout
- Cost arithmetic
- Data residency
- Food-safety liability
- Auditability
- LLM misuse
- Unsupported assumptions
- Fake trade-offs
- Unnecessary components
- Operational support
- Seven-minute presentation feasibility

Update artifacts in place.

Simplification rules:

- Remove duplicated explanations.
- Group related NFRs into approximately 8–12 concise rows.
- Keep all measurable targets and production gates.
- Separate requirement, assumption, design target, measured evidence, and
production gate.
- Remove false precision.
- Replace unnecessary technical jargon with plain architecture language.
- Remove components that do not address a stated requirement or failure mode.
- Keep exactly five highest-priority risks.
- Keep at least three consequential technology decisions.
- Keep one concise ADR.
- Ensure the Variant B artifact covers legacy assessment, selected pattern,
shadow mode, kill switch, fail-open, human review, and rollback.

Presentation constraints:

- Maximum eight slides
- Maximum five visible points per slide
- Total speaking time between 6:30 and 7:00
- Do not re-explain the scenario
- Open directly with the architecture thesis

After editing, return:

1. Rejection-level issues found
2. Files updated
3. Major simplifications
4. Missing requirements added
5. Remaining assumptions
6. Likely trainer questions
7. Estimated presentation duration
8. Final verdict: Ready / Ready with conditions / Not ready

Create:

reviews/final-review.md
reviews/change-log.md