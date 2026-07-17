Có. Với workshop này, nên điều chỉnh mô hình subagent một chút: **không cần “Developer agent” theo nghĩa viết code**, mà thay bằng **Artifact Builder** để tạo C4, NFR table, technology matrix, ADR, risk register và legacy assessment.

Nhóm của bạn là **Group 4 – Variant B: AI-Retrofit**, nên workflow phù hợp là:

```text
Requirement Analyst
→ Legacy Technical Architect
→ NFR & FinOps Validator
→ Artifact Builder
→ Red-Team Reviewer
→ Presentation Coach
```

PDF yêu cầu Variant B phải có legacy assessment, lựa chọn integration pattern, kill-switch design; toàn bộ bài nộp tối đa 8 slides và phần trình bày chỉ có 7 phút. 

## Prompt tổng dùng trong Codex

Đặt PDF vào repository, ví dụ:

```text
docs/workshop-1-ai-system-design.pdf
```

Sau đó paste prompt này vào Codex:

```text
    You are the lead orchestrator for an AI Technical Architecture workshop.

    Your task is to read and analyze:

    docs/workshop-1-ai-system-design.pdf

    We are Group 4 and have been assigned:

    Variant B — AI-Retrofit / Legacy

    The goal is to produce a defensible end-to-end AI architecture for NovaMart, together with all six required workshop artifacts and an eight-slide presentation.

    Do not start by generating slides.

    First, establish the architecture decisions, validate them against every constraint, and only then create presentation content.

    ==================================================
    WORKFLOW
    ==================================================

    Execute the following workflow in order.

    1. Spawn a Requirement Analyst agent.
    2. Spawn a Legacy Technical Architect agent.
    3. Spawn an NFR, Cost and Safety Validator agent.
    4. Spawn an Artifact Builder agent.
    5. Spawn a Red-Team Reviewer agent.
    6. After resolving critical reviewer findings, spawn a Presentation Coach agent.

    Wait for each phase before proceeding to the next phase.

    Do not allow multiple agents to edit the same artifact concurrently.

    ==================================================
    AGENT 1 — REQUIREMENT ANALYST
    ==================================================

    Read the complete PDF and extract a traceable requirement baseline.

    Produce:

    artifacts/01-requirement-baseline.md

    The document must contain:

    1. Business objectives and target KPIs
    2. Functional capabilities
    3. Non-functional requirements
    4. Variant B legacy constraints
    5. Legal, safety and data-residency constraints
    6. Budget, timeline and team constraints
    7. Hard constraints versus assumptions
    8. Explicitly prohibited changes
    9. Open questions
    10. A requirement-to-artifact traceability table

    Pay special attention to:

    - The 22:00–04:00 batch window
    - The fixed 04:00 supplier EDI cut-off
    - 640 stores and approximately 11,000 SKUs per store
    - 27 million daily forecasts
    - Six-times Lunar New Year peak
    - MERLIN being the system of record
    - MERLIN having only three extension points
    - MERLIN's replenishment engine being unmodifiable
    - POS being completely untouchable
    - Inventory accuracy being only 78%
    - The requirement that replenishment must continue when AI is unavailable
    - Flaky store connectivity
    - Data residency restrictions
    - Food-safety and allergen liability
    - The 8-month timeline
    - The $3.2M constraint
    - The 2.4% business margin

    Do not propose a solution yet.

    ==================================================
    AGENT 2 — LEGACY TECHNICAL ARCHITECT
    ==================================================

    Using the requirement baseline, design the simplest architecture that satisfies the requirements.

    Produce:

    artifacts/02-architecture-decision-summary.md
    artifacts/03-c4-context.mmd
    artifacts/04-c4-container.mmd
    artifacts/05-request-and-batch-flows.md
    artifacts/06-legacy-assessment.md

    The architecture must:

    - Keep MERLIN as the ordering system of record
    - Never modify the MERLIN replenishment engine
    - Never modify POS
    - Never change the supplier EDI schema
    - Never move the 04:00 cut-off
    - Attach only through supported MERLIN extension points
    - Continue normal replenishment when AI components fail
    - Treat inventory as uncertain rather than real-time truth
    - Include an explicit fail-open or bypass mechanism
    - Include a kill switch
    - Support shadow mode before AI recommendations affect orders
    - Include human review where risk requires it
    - Separate deterministic optimization, forecasting ML and LLM use cases
    - Avoid using an LLM for numeric forecasting or replenishment optimization
    - Use an LLM only where language understanding is genuinely required
    - Include auditability and model/version traceability

    Evaluate these AI-Retrofit integration patterns:

    1. Sidecar
    2. Event-driven integration
    3. API gateway
    4. Shadow-assist
    5. In-process plugin

    Select one primary pattern.

    A hybrid may be used only when its boundaries are explicit. For example, a sidecar integration may initially operate in shadow-assist mode.

    For the selected pattern, explain:

    - Why it fits MERLIN
    - Why competing patterns were rejected
    - Where recommendations enter the MERLIN workflow
    - What happens if the AI platform is unavailable
    - What happens if output is late
    - What happens if forecast confidence is low
    - What happens if input data is stale or incomplete
    - How category managers review exceptions
    - How rollback works
    - How the kill switch works
    - Which system remains authoritative

    Create valid Mermaid C4-style diagrams.

    The container diagram should include at minimum:

    - MERLIN
    - Existing nightly batch
    - AI sidecar or selected integration layer
    - Data ingestion and validation
    - Feature pipeline
    - Forecasting service
    - Replenishment optimization service
    - Recommendation or override store
    - MERLIN override-table adapter
    - Review queue integration
    - Model registry
    - Monitoring and alerting
    - Audit log
    - Associate copilot components, only if included in scope
    - Authoritative source systems
    - Human users

    Narrate the main request and batch flows step by step.

    ==================================================
    AGENT 3 — NFR, COST AND SAFETY VALIDATOR
    ==================================================

    Challenge the proposed architecture quantitatively.

    Produce:

    artifacts/07-nfr-table.md
    artifacts/08-capacity-and-cost-check.md
    artifacts/09-safety-and-governance.md

    The NFR table must include actual design numbers, not statements such as
    "the system will be scalable."

    For every NFR include:

    - Requirement
    - Target
    - Proposed mechanism
    - Capacity or latency calculation
    - Failure behavior
    - Monitoring metric
    - Warning threshold
    - Critical threshold
    - Owner

    Validate at least:

    1. Completion within the six-hour batch window
    2. Capacity for 27 million forecasts per day
    3. Six-times peak load
    4. Forecast and optimization retry strategy
    5. Deadline protection before 04:00
    6. AI-unavailable behavior
    7. Data-quality failure behavior
    8. Inventory uncertainty handling
    9. Store connectivity failure
    10. Copilot p95 latency, when included
    11. Offline copilot behavior, when included
    12. Online substitution p95 latency, when included
    13. Data residency
    14. Auditability
    15. Allergen and food-safety safety controls
    16. Total annual AI operating cost
    17. Cost per forecast
    18. Operational support for the available team

    Show the arithmetic.

    State assumptions explicitly.

    Reject any design that:

    - Cannot complete before the operational deadline
    - Depends on perfectly accurate inventory
    - Prevents MERLIN from ordering when AI fails
    - Uses an LLM to generate all forecasts
    - Requires modifications to POS
    - Requires replacing MERLIN
    - Exceeds the budget without a defensible ROI
    - Gives uncited or generative allergen answers

    For food-safety questions, define:

    - Authoritative sources
    - Citation requirements
    - Confidence policy
    - Refusal policy
    - Human escalation
    - Document-version controls
    - Audit record
    - Accountability

    ==================================================
    AGENT 4 — ARTIFACT BUILDER
    ==================================================

    Using the validated architecture, create the six required artifacts.

    Produce:

    artifacts/10-technology-selection-matrix.md
    artifacts/11-adr.md
    artifacts/12-risk-register.md
    artifacts/13-variant-b-artifact.md
    artifacts/14-final-artifact-pack.md

    Required artifact contents:

    ARTIFACT 1 — C4 context and container diagrams

    Include:

    - System context
    - Container view
    - Narrated batch and request flows
    - Trust boundaries
    - Authoritative systems
    - Human decision points

    ARTIFACT 2 — NFR table

    Cover every NFR using actual numbers and concrete mechanisms.

    ARTIFACT 3 — Technology-selection matrix

    Evaluate at least three consequential decisions.

    Suggested decisions:

    A. AI-Retrofit integration pattern
    B. Forecasting platform and execution model
    C. Batch orchestration and compute strategy
    D. Online versus offline feature handling
    E. Copilot retrieval architecture
    F. Cloud versus on-premises deployment boundary

    For each decision include:

    - Options
    - Evaluation criteria
    - Weight for each criterion
    - Score from 1 to 5
    - Weighted result
    - Selected option
    - Trade-off accepted

    Do not manipulate weights to force a preferred answer.

    ARTIFACT 4 — One ADR

    Select the single most consequential architecture decision.

    Recommended ADR topic:

    "Use a fail-open AI sidecar in shadow-assist mode, integrated through MERLIN's override table and review queue, while retaining MERLIN as the ordering authority."

    The ADR must include:

    - Context
    - Decision drivers
    - Options considered
    - Decision
    - Detailed consequences
    - Positive consequences
    - Negative consequences
    - Risks
    - Rollback strategy
    - Conditions under which the ADR should be revisited

    ARTIFACT 5 — Risk register

    Provide exactly the top five architecture risks.

    For each risk include:

    - Category
    - Description
    - Cause
    - Likelihood from 1 to 5
    - Impact from 1 to 5
    - Risk score
    - Mitigation
    - Contingency
    - Monitoring signal
    - Numeric threshold
    - Owner

    Likely risks to assess include:

    - Missing the 04:00 deadline
    - Inventory inaccuracy causing bad orders
    - Model degradation during promotions or Lunar New Year
    - Unsafe copilot answers
    - MERLIN integration failure
    - Cost overrun
    - Data-residency violation
    - Category-manager override fatigue

    Select the top five based on quantified likelihood and impact.

    ARTIFACT 6 — Variant B specific artifact

    Include:

    1. Legacy assessment
    2. What can change
    3. What cannot change
    4. What is unsafe to touch
    5. Selected AI-Retrofit integration pattern
    6. Why this pattern was selected
    7. Kill-switch design
    8. Fail-open behavior
    9. Shadow-mode rollout
    10. Human review model
    11. Rollback plan
    12. Migration phases without replacing MERLIN

    Assign one named owner placeholder to every artifact:

    - Owner: [Name to be assigned]

    ==================================================
    AGENT 5 — RED-TEAM REVIEWER
    ==================================================

    Act as a strict workshop trainer.

    Review all artifacts for unsupported assumptions, architectural contradictions and weak trade-offs.

    Produce:

    reviews/01-red-team-review.md

    Classify findings as:

    - Critical
    - Major
    - Minor

    Explicitly challenge the design with these questions:

    1. Can all forecasts and orders really complete by 04:00?
    2. What happens at six-times peak?
    3. What happens when the AI platform is unavailable at 03:30?
    4. Does the design accidentally depend on real-time inventory?
    5. Can an AI error stop stores from receiving deliveries?
    6. Has the design modified MERLIN, POS or EDI indirectly?
    7. Is the selected integration pattern genuinely appropriate?
    8. Is shadow-assist a rollout mode or an architecture pattern in this design?
    9. Is the kill switch technically executable within minutes?
    10. Are category managers overloaded by review queues?
    11. Is an LLM being used for a non-language task?
    12. Is the cost calculation complete?
    13. Are data residency boundaries visible in the architecture?
    14. Can an allergen answer ever be generated without evidence?
    15. Is every risk connected to a measurable signal and threshold?
    16. Does the technology matrix show real trade-offs?
    17. Does the ADR document negative consequences honestly?
    18. Is any component included merely to make the architecture look impressive?
    19. Could a smaller and simpler design satisfy the same NFRs?
    20. Can the group defend each decision in less than one minute?

    Return a final verdict:

    - Ready
    - Ready with minor changes
    - Not ready

    Do not rewrite artifacts directly.

    Provide concrete recommended corrections.

    ==================================================
    CORRECTION PHASE
    ==================================================

    After the review:

    - Fix all Critical findings.
    - Fix Major findings that affect correctness, safety, cost or compliance.
    - Do not blindly apply stylistic feedback.
    - Record corrections in:

    reviews/02-resolution-log.md

    For each reviewer finding include:

    - Finding
    - Accepted or rejected
    - Reason
    - Artifact changed
    - Final resolution

    ==================================================
    AGENT 6 — PRESENTATION COACH
    ==================================================

    After the architecture passes review, create a maximum eight-slide presentation.

    Produce:

    presentation/01-eight-slide-outline.md
    presentation/02-speaker-notes.md
    presentation/03-trainer-qa.md
    presentation/04-one-page-cheat-sheet.md

    The presentation is seven minutes.

    Do not explain the NovaMart scenario at the beginning. Everyone already knows it.

    Start directly with the architecture position and most important decision.

    Recommended slide structure:

    Slide 1 — Architecture thesis

    State the design in one sentence.

    Include:

    - What we are building
    - What remains authoritative
    - How AI failure is isolated
    - The selected integration pattern

    Slide 2 — Legacy assessment

    Show:

    - What can change
    - What cannot change
    - What is unsafe to touch
    - The three MERLIN extension points

    Slide 3 — C4 context and container architecture

    Show the main containers and system boundaries.

    Slide 4 — End-to-end batch flow and 04:00 deadline

    Show:

    - Data readiness
    - Forecasting
    - Optimization
    - Validation
    - Override creation
    - MERLIN execution
    - Deadline protection
    - Fallback path

    Slide 5 — NFR and cost proof

    Use actual numbers for:

    - 27 million forecasts
    - Six-hour processing window
    - Six-times peak
    - AI cost
    - Cost per forecast
    - Availability and fallback

    Slide 6 — Technology decisions and ADR

    Show:

    - Weighted selection result
    - Chosen integration pattern
    - Most consequential ADR
    - Trade-off accepted

    Slide 7 — Top five risks and controls

    Show measurable signals and thresholds.

    Slide 8 — Rollout, kill switch and final defense

    Show:

    - Shadow mode
    - Limited pilot
    - Human approval
    - Controlled override
    - Kill switch
    - Fail-open behavior
    - Scale-out criteria

    For every slide provide:

    - Slide objective
    - Headline
    - Maximum five visible points
    - Recommended visual
    - Speaker script
    - Target speaking time

    Total speaking time must be between 6 minutes 30 seconds and 7 minutes.

    The Q&A document must contain at least 20 difficult trainer questions.

    For each question include:

    - Strong answer
    - Evidence from the artifacts
    - Weak answer to avoid
    - Follow-up question the trainer may ask

    ==================================================
    FINAL QUALITY RULES
    ==================================================

    The final work must:

    - Contain no more than eight slides
    - Be defendable in seven minutes
    - Not re-explain the scenario
    - Use the simplest architecture that meets the requirements
    - Include numerical evidence
    - Keep MERLIN authoritative
    - Preserve the existing replenishment fallback
    - Treat 78% inventory accuracy as a core architecture problem
    - Never depend on modifying POS
    - Never depend on changing supplier EDI
    - Include a technically precise kill switch
    - Include real trade-offs
    - Use LLMs only for language problems
    - Include citation-grounded allergen controls
    - Be explicit about assumptions
    - Clearly distinguish facts from design decisions

    At the end, provide a concise executive summary containing:

    1. Architecture thesis
    2. Selected integration pattern
    3. Most consequential trade-off
    4. How the 04:00 deadline is protected
    5. How AI failure is isolated
    6. Why the design is affordable
    7. The three hardest trainer questions
```

## Prompt ngắn hơn để chạy review lần cuối

Sau khi Codex tạo artifacts, dùng thêm prompt này:

```text
    Act as a panel of senior AI architecture trainers.

    Review all files under:

    artifacts/
    presentation/

    The assigned topic is Variant B — AI-Retrofit.

    Do not praise the work. Find the reasons this architecture could be rejected.

    Focus on:

    - 04:00 deadline feasibility
    - 27 million forecasts per day
    - Six-times peak volume
    - 78% inventory accuracy
    - MERLIN fail-open behavior
    - POS and EDI immutability
    - Kill-switch realism
    - Cost arithmetic
    - Data residency
    - Food-safety liability
    - LLM misuse
    - Unsupported assumptions
    - Fake trade-offs
    - Architecture components that are unnecessary
    - Whether the presentation fits seven minutes

    Return:

    1. Rejection-level issues
    2. Questions trainers are likely to ask
    3. Required corrections
    4. Final verdict: Ready / Not ready
    ```

    Điểm quan trọng nhất là **không để agent tự tạo slide ngay từ đầu**. Hãy bắt nó đi theo trình tự:

    ```text
    Extract constraints
    → Decide architecture
    → Prove NFR and cost
    → Produce artifacts
    → Red-team review
    → Build eight slides
    → Simulate Q&A
```

Workshop không chỉ kiểm tra khả năng vẽ kiến trúc; tài liệu nhấn mạnh rằng nhóm phải **defend architecture**, chứng minh chi phí, nhận ra các constraint khó và trình bày trade-off thật. 
