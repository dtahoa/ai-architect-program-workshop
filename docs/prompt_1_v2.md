Act as a panel of senior AI architecture trainers, a technical editor,
and a workshop presentation coach.

Review and improve all files under:

artifacts/
presentation/

Use the following file as the primary requirement baseline:

docs/req_analysis_vi.md

Assigned topic:

Variant B — AI-Retrofit / Legacy

Your task is not only to review the current output.

You must also update the artifacts and presentation so that they are:

- Shorter
- Easier to understand
- Internally consistent
- Defensible during trainer Q&A
- Fully traceable to docs/req_analysis_vi.md
- Suitable for a maximum eight-slide presentation
- Presentable within seven minutes

Do not praise the work.

Find rejection risks, correct them, and simplify the output without losing
mandatory requirements, important calculations, failure behavior,
trade-offs, monitoring thresholds, or production gates.

==================================================
SOURCE-OF-TRUTH RULE
==================================================

Treat docs/req_analysis_vi.md as the primary source of truth.

Before editing any artifact:

1. Read the entire file.
2. Extract all mandatory requirements.
3. Classify each requirement as:
   - Business objective
   - Functional capability
   - Non-functional requirement
   - Variant B legacy constraint
   - Safety requirement
   - Compliance requirement
   - Cost constraint
   - Operational constraint
   - Presentation or submission requirement
4. Identify hard constraints that cannot be weakened or removed.
5. Identify assumptions that are not explicitly supported by the document.

Do not invent new workshop requirements.

Do not remove a requirement merely because it makes an artifact shorter.

==================================================
REQUIREMENT COVERAGE BASELINE
==================================================

Create or update:

reviews/requirement-coverage-matrix.md

For every material requirement in docs/req_analysis_vi.md, include:

- Requirement ID
- Requirement summary
- Mandatory or optional
- Source section
- Artifact file covering it
- Artifact section covering it
- Presentation slide covering it
- Coverage status:
  - Covered
  - Partially covered
  - Missing
  - Intentionally deferred
- Required correction

Every mandatory requirement must be covered by at least one artifact.

Important requirements that affect architectural acceptance must also appear
in the presentation.

==================================================
REVIEW FOCUS
==================================================

Review the architecture against at least the following areas:

1. 04:00 deadline feasibility
2. AI processing close time before MERLIN processing
3. 27 million forecasts per day
4. Six-times Lunar New Year peak volume
5. Promotion demand distortion
6. Inventory accuracy of only 78%
7. MERLIN fail-open behavior
8. MERLIN remaining the ordering system of record
9. POS immutability
10. Supplier EDI schema and cut-off immutability
11. Use of only supported MERLIN extension points
12. Kill-switch realism
13. Shadow-assist rollout
14. Human review capacity
15. Cost arithmetic
16. Cost per forecast
17. Data residency
18. Food-safety and allergen liability
19. Auditability
20. Model and data version traceability
21. LLM misuse
22. Unsupported assumptions
23. Fake or weak trade-offs
24. Unnecessary architecture components
25. Whether the proposed system can be operated by the available team
26. Whether all six required artifacts are present
27. Whether each artifact has a named owner
28. Whether the presentation fits within eight slides
29. Whether the presentation fits within seven minutes
30. Whether all artifact files tell the same architecture story

==================================================
SIMPLIFICATION RULES
==================================================

The current artifacts may contain excessive technical detail.

Simplify them using the following rules.

1. Remove duplication

When the same constraint, number, fallback, or explanation appears in
multiple places, keep the full explanation in the most appropriate artifact
and use a short reference elsewhere.

2. Merge related NFRs

Group related NFRs into concise categories where possible:

- Batch deadline and capacity
- Forecast and model quality
- Data and inventory quality
- Legacy isolation and fail-open
- Copilot latency and offline operation
- Food safety
- Cost and FinOps
- Data residency and audit
- Operations and ownership

Do not merge requirements when doing so would hide a different target,
failure behavior, owner, or production gate.

3. Preserve measurable evidence

Do not remove:

- Quantitative targets
- Capacity calculations
- Cost calculations
- Warning thresholds
- Critical thresholds
- Failure behavior
- Fallback behavior
- Production acceptance gates

Reduce unnecessary prose around these items.

4. Remove false precision

Where a value is presented with excessive precision but is based on an
assumption, either:

- Round it to a defensible level, or
- State the assumption and calculation explicitly.

For example, avoid values such as:

$3.31242M

unless the source inputs genuinely justify that precision.

Prefer:

Approximately $3.31M based on the stated compute, storage, monitoring,
and support assumptions.

5. Separate facts, decisions, assumptions, and targets

Every artifact must clearly distinguish:

- Workshop facts
- Architecture decisions
- Design targets
- Assumptions requiring validation
- Measured evidence
- Production acceptance gates

Do not present an untested design target as an observed result.

6. Remove unnecessary implementation complexity

Challenge mechanisms such as:

- Quorum latch
- Multiple hashes
- Complex epoch controls
- Excessive service decomposition
- Unnecessary event buses
- Multiple databases
- Excessive cloud services
- Custom platforms

Keep them only when they address a specific failure mode required by
docs/req_analysis_vi.md.

Explain every retained complex mechanism in plain language.

7. Use plain architecture language

Replace unnecessarily difficult wording with concise explanations.

For example:

Instead of:

"Durable quorum latch with same-transaction epoch guard."

Prefer:

"A centrally controlled publication permit prevents stale or disabled AI
runs from writing MERLIN overrides. The permit is checked atomically when
each override batch is activated."

The precise technical term may remain in supporting notes if necessary.

==================================================
ARTIFACT UPDATE REQUIREMENTS
==================================================

Update the artifact files in place.

Do not create multiple competing final versions.

The final artifact set must contain the six workshop artifacts.

--------------------------------------------------
ARTIFACT 1 — C4 CONTEXT AND CONTAINER DIAGRAM
--------------------------------------------------

Ensure it contains:

- System context
- Container view
- Authoritative systems
- Human actors
- MERLIN extension points
- Trust and data-residency boundaries
- Batch flow
- Copilot request flow if included
- Failure and fallback flow
- Kill-switch path

Simplification target:

- One context diagram
- One container diagram
- One concise narrated batch flow
- One concise failure-flow explanation

Remove infrastructure components that do not affect an architecture
decision.

--------------------------------------------------
ARTIFACT 2 — QUANTIFIED NFR TABLE
--------------------------------------------------

Shorten the NFR table while preserving complete requirement coverage.

Use a concise structure:

| Area | Target | Design mechanism | Failure behavior | Key metric and gate | Owner |

Prefer approximately 8–12 grouped NFR rows instead of many fragmented rows,
provided that no mandatory requirement is lost.

Every row must contain:

- Numeric or objectively testable target
- Concrete mechanism
- Failure or fallback behavior
- Warning or production gate
- Operational owner

Move secondary calculations to a supporting calculation section when needed.

Clearly label values as:

- Requirement
- Design target
- Assumption
- To be measured
- Production gate

--------------------------------------------------
ARTIFACT 3 — TECHNOLOGY-SELECTION MATRIX
--------------------------------------------------

Keep at least three consequential architecture decisions.

For each decision include:

- Options considered
- Weighted criteria
- Scores
- Selected option
- Rejected alternatives
- Trade-off accepted

Remove low-impact technology comparisons.

Prioritize decisions such as:

1. AI-Retrofit integration pattern
2. Forecast execution/model strategy
3. Deployment and batch compute strategy
4. Copilot safety architecture, when included

Do not compare technologies merely because they are popular.

--------------------------------------------------
ARTIFACT 4 — ADR
--------------------------------------------------

Keep one consequential ADR only.

Recommended topic:

Use a fail-open AI sidecar integrated through MERLIN's override table and
review queue, initially operating in shadow-assist mode.

The ADR must remain concise and include:

- Context
- Decision drivers
- Options considered
- Decision
- Positive consequences
- Negative consequences
- Risks
- Rollback
- Conditions for revisiting the decision

The ADR should be readable in approximately two minutes.

--------------------------------------------------
ARTIFACT 5 — RISK REGISTER
--------------------------------------------------

Keep exactly the five highest-priority architecture risks.

Each risk must include:

- Risk
- Category
- Likelihood
- Impact
- Risk score
- Mitigation
- Contingency
- Monitoring signal
- Numeric threshold
- Owner

Do not include low-priority risks merely to show breadth.

Likely rejection-level risks include:

- Missing the operational deadline
- Inventory uncertainty causing unsafe recommendations
- AI or MERLIN integration failure
- Model failure during promotions or Lunar New Year
- Unsafe food-safety output
- Data-residency violation
- Review-queue overload
- Cost overrun

Select the final five using quantified likelihood and impact.

--------------------------------------------------
ARTIFACT 6 — VARIANT B SPECIFIC
--------------------------------------------------

Ensure this artifact clearly contains:

1. Legacy assessment
2. What can change
3. What cannot change
4. What is unsafe to touch
5. MERLIN's three supported extension points
6. Selected AI-Retrofit integration pattern
7. Why it was selected
8. Why alternatives were rejected
9. Shadow-assist rollout
10. Kill-switch design
11. Fail-open behavior
12. Human review model
13. Rollback plan
14. Phased adoption without replacing MERLIN

Present the legacy assessment in a compact table where possible.

==================================================
PRESENTATION UPDATE REQUIREMENTS
==================================================

Update files under presentation/ so the final presentation contains no more
than eight slides and fits within seven minutes.

Do not explain the NovaMart scenario again.

The opening must state the architecture position immediately.

Recommended structure:

Slide 1 — Architecture thesis
Slide 2 — Legacy assessment and non-negotiable constraints
Slide 3 — C4 context and container architecture
Slide 4 — Batch timeline, 04:00 protection, and fail-open flow
Slide 5 — Quantified NFR, capacity, quality, and cost proof
Slide 6 — Technology decisions and ADR
Slide 7 — Top five risks and monitoring controls
Slide 8 — Shadow rollout, kill switch, and final defense

For each slide provide:

- Slide objective
- One clear headline
- Maximum five visible points
- Recommended visual
- Speaker notes
- Target speaking time

Total speaking time must be between:

6 minutes 30 seconds and 7 minutes

Do not solve overcrowding by using smaller text.

Move detailed evidence to artifact files or speaker notes.

==================================================
CONSISTENCY CHECK
==================================================

Verify that all files agree on:

- Scope of Release 1
- Selected integration pattern
- Whether shadow-assist is a pattern or rollout mode
- MERLIN's authoritative role
- AI close time
- MERLIN reserve time
- Forecast volumes
- Peak factor
- Inventory accuracy
- Cost target and ceiling
- Kill-switch timing
- Fail-open behavior
- Data-residency boundary
- Food-safety response policy
- Deferred capabilities
- Owners
- Production acceptance gates

Where inconsistencies exist, choose the interpretation best supported by
docs/req_analysis_vi.md and update all affected files.

==================================================
TRAINER RED-TEAM REVIEW
==================================================

After updating the artifacts, perform a final red-team review.

Do not praise the work.

Find reasons the architecture could still be rejected.

Explicitly challenge:

1. Is the 02:00 AI close justified by measured MERLIN P99?
2. Can 27 million forecasts complete in the stated window?
3. Can the same design handle six-times peak?
4. What happens if AI is unavailable at 03:30?
5. Can any AI failure delay MERLIN?
6. Does the design assume inventory is real-time or accurate?
7. Can stale recommendations be written after the kill switch?
8. Is the kill switch executable within the claimed timing?
9. Can category managers process the expected review volume?
10. Does the architecture violate POS or EDI constraints indirectly?
11. Is any LLM used for forecast, quantity, price, or safety output?
12. Is food-safety output deterministic and citation-grounded?
13. Are data-residency boundaries enforceable?
14. Is the cost estimate complete and based on stated assumptions?
15. Are any components included only to make the design look impressive?
16. Does every risk have a measurable signal and threshold?
17. Are the stated trade-offs real?
18. Can the team operate the system with available staffing?
19. Does each artifact cover its required workshop purpose?
20. Can every slide be defended within the available Q&A time?

==================================================
EDITING BEHAVIOR
==================================================

You are authorized to update files under:

artifacts/
presentation/
reviews/

Before editing:

- Inspect all current files.
- Identify duplicates and contradictions.
- Preserve useful calculations and evidence.
- Do not overwrite useful detailed evidence without relocating it.

When detailed evidence is too long for a primary artifact, move it to:

artifacts/supporting/

Use supporting files only when necessary.

Primary artifact files must remain concise and reviewable.

==================================================
REQUIRED OUTPUT
==================================================

After completing the review and edits, return:

1. Rejection-level issues found before correction
2. Files updated
3. Major simplifications made
4. Requirements that were previously missing or partially covered
5. Requirements intentionally deferred and the reason
6. Remaining unsupported assumptions
7. Trainer questions most likely to be asked
8. Final requirement-coverage status
9. Estimated presentation duration
10. Final verdict:
    - Ready
    - Ready with conditions
    - Not ready

Also create:

reviews/final-review.md
reviews/change-log.md

The change log must include:

| File | Problem | Change made | Requirement preserved or added |

==================================================
FINAL ACCEPTANCE CONDITIONS
==================================================

Return "Ready" only if:

- Every mandatory requirement in docs/req_analysis_vi.md is covered
- All six workshop artifacts are present
- Each artifact has a named owner
- Artifacts are internally consistent
- MERLIN remains authoritative
- AI failure never blocks replenishment
- POS is untouched
- EDI schema and 04:00 cut-off remain unchanged
- Inventory uncertainty is explicitly handled
- Cost arithmetic is defensible
- LLMs are not used for numeric or safety decisions
- Food-safety output has deterministic evidence, citation, and refusal controls
- Kill-switch behavior is measurable and testable
- The presentation has no more than eight slides
- The presentation fits within seven minutes
- No rejection-level issue remains unresolved