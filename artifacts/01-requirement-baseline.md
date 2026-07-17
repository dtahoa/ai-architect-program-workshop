# NovaMart Workshop 1 - Requirement Baseline

Source of record: all six pages of `docs/workshop-1-ai-system-design.pdf`, the Group 4 assignment and presentation schedule supplied by the user, and the attached orchestration instructions. This document records requirements and ambiguities only. It deliberately does not select an architecture, integration pattern, technology, model, deployment boundary, or rollout design.

## 1. Business objectives and target KPIs

| ID | Objective or measure | Current baseline | Approved target or constraint | Business significance |
|---|---|---:|---:|---|
| BO-01 | Reduce out-of-stock rate | 7.2% | 3.0% | Current lost-sales estimate is about $180M/year. |
| BO-02 | Reduce fresh waste | 4.8% of fresh revenue | 3.0% of fresh revenue | Fresh/perishables are 31% of revenue; current fresh waste is about $61M/year. |
| BO-03 | Improve demand-forecast accuracy, measured by WAPE | 41% | <= 25% | The workshop expects a quantified forecasting-quality outcome, not a generic accuracy claim. |
| BO-04 | Reduce associate time lost to lookups | About 50 minutes/shift | < 15 minutes/shift | The benefit is redeployment to customer service. |
| BO-05 | Preserve store-associate employment | 8,400 associates | 8,400 associates | This is explicitly not a headcount-reduction program. |
| BO-06 | Protect a thin business margin | 2.4% net margin | Do not erode the 2.4% margin with infrastructure or AI cost | Infrastructure cost directly consumes profit. |
| BO-07 | Realize value against the identified opportunity | About $241M/year identified upside: $61M waste plus $180M lost sales | No separate realized-value target is stated | Any cost or scope case must remain defensible against this upside. |

| ID | Scale fact | Baseline |
|---|---|---:|
| BC-01 | Markets | 3 Asian markets |
| BC-02 | Stores | 640 large-format supermarkets and convenience stores |
| BC-03 | Assortment | 42,000 SKUs chain-wide; about 11,000 per store |
| BC-04 | Transactions | 2.9M/day, with 5 years of POS history and billions of rows |
| BC-05 | Revenue | About $4.1B/year |
| BC-06 | Loyalty | 14M members; 68% of transactions |
| BC-07 | Promotions | About 2,200 events/year; promotion weeks distort demand by 3-10x |
| BC-08 | Suppliers | 2,300 suppliers integrated through fixed EDI |
| BC-09 | POS estate | About 7,000 lanes across 640 stores |

## 2. Functional capabilities

The PDF presents the following six target capabilities but explicitly leaves it to the design to decide how to achieve them or whether some are not worth building. They are therefore capability candidates until scope and acceptance criteria are confirmed.

| ID | Capability | Required behavior or decision scope |
|---|---|---|
| FC-01 | Demand forecasting | Forecast per SKU, per store, per day. The brief states `42,000 x 640 = about 27M` time series. |
| FC-02 | Automated replenishment | Generate order quantities while respecting shelf life, case packs, distribution-center capacity, and the fixed 04:00 cut-off. |
| FC-03 | Fresh markdown or dynamic pricing | Reduce fresh waste without destroying margin; decisions are required three times per day. |
| FC-04 | Store-associate copilot | Answer questions about planograms, SOPs, product knowledge, and food-safety policy, including location, sellability, and allergen questions. |
| FC-05 | Online grocery customer service and substitutions | Recommend what to substitute when an item is out of stock. |
| FC-06 | Promotion-planning support | Forecast promotion lift and cannibalization. |

Available data named by the brief comprises POS, loyalty, inventory snapshots, supplier EDI, planograms, 1,400 SOP and food-safety documents in 3 languages, a 42,000-SKU product master, waste and shrink logs, and third-party weather, local-event, school-calendar, and scraped competitor-pricing data. Availability in the brief is not equivalent to approval for every use.

## 3. Non-functional requirements

| ID | Category | Binding requirement or validation baseline |
|---|---|---|
| NFR-01 | End-to-end batch deadline | Forecasting and order generation for all 640 stores and about 11,000 SKUs/store must complete in the fixed 22:00-04:00 window. This is 7,040,000 store-SKU combinations if the per-store assortment is used. |
| NFR-02 | Operational cut-off | Every store order must be final by 04:00 so the distribution center can pick it and the 06:00 truck can leave. A missed cut-off means that store receives nothing that day; there are no exceptions and no re-runs. |
| NFR-03 | Forecast scale | Validate capacity and cost against the separately stated 27M forecasts/day (`42,000 x 640 = 26.88M`, rounded). The difference from NFR-01 is unresolved, not silently normalized. |
| NFR-04 | Seasonal peak | Support 6x normal volume for two weeks at Lunar New Year. The calendar date moves each year, and the six-hour window does not expand. |
| NFR-05 | Promotion distortion | Cope with 3-10x demand distortion during promotion weeks. |
| NFR-06 | Copilot latency | Store-associate copilot p95 latency must be < 2 seconds. |
| NFR-07 | Copilot offline operation | The copilot must work when store internet is down. Four percent of stores lose connectivity for more than one hour at least weekly. |
| NFR-08 | Online substitution latency | Online-order substitution p95 latency must be < 500 ms. |
| NFR-09 | Markdown cadence | Fresh markdown decisions must be computed 3 times/day: morning, midday, and evening. |
| NFR-10 | POS availability and isolation | POS availability is 99.99%, POS must run fully offline, and no AI dependency may touch or impair the revenue path. |
| NFR-11 | Ordering continuity | AI unavailability, lateness, low confidence, or bad input must never stop the incumbent replenishment and ordering chain. Stores must still receive morning trucks. |
| NFR-12 | AI run-cost ceiling | Total AI run cost is capped at $4M/year, about $10,959/day. At 27M forecasts/day, the full-budget arithmetic is about $0.0004/forecast; all other AI workloads also consume this same annual total. |
| NFR-13 | Inventory uncertainty | Measured perpetual-inventory accuracy is 78%; more than 1 in 5 records may be wrong about shelf reality. No design may treat inventory as real-time truth. |
| NFR-14 | Data freshness | The data warehouse runs nightly ETL and can be up to 24 hours stale. MERLIN stock-on-hand is updated by nightly POS sync, so in-day inventory is estimated. |
| NFR-15 | Safety | Food-safety and allergen behavior must define when evidence is cited, when the system refuses, and who is accountable. An unsupported answer is unacceptable because liability is criminal and harm can be fatal. |
| NFR-16 | Pricing auditability | Markdown decisions must be auditable; consumer-protection law prohibits arbitrary price discrimination. |
| NFR-17 | Residency and retention | Personal loyalty data for one market must not leave that country. POS data must be retained for 5 years. |
| NFR-18 | Traceability and operations | The orchestration brief requires auditability plus model/version traceability, quantified monitoring thresholds, explicit owners, and operational support appropriate to the available team. |
| NFR-19 | Workload and model fitness | The orchestration brief requires deterministic optimization, forecasting ML, and LLM language use cases to be separated. An LLM may be used only where language understanding is genuinely required, never for numeric forecasting or replenishment optimization. |

The quantitative validation phase must show arithmetic for the six-hour window, 27M/day, 6x peak, retries, deadline protection, failure behavior, data-quality behavior, inventory uncertainty, connectivity loss, applicable online/offline latency, residency, auditability, safety, annual run cost, cost per forecast, and support load. Generic claims such as "scalable" are not acceptable.

## 4. Variant B legacy constraints

| ID | Legacy fact or constraint | Baseline implication, without selecting a solution |
|---|---|---|
| LC-01 | MERLIN is an on-premises SAP-based ERP, continuously customized since 2006. | Its current boundaries and supported interfaces are authoritative constraints. |
| LC-02 | MERLIN owns merchandising, inventory, replenishment, and supplier EDI. | These responsibilities cannot be assumed to move to a new core. |
| LC-03 | MERLIN is the system of record for supplier orders. | Later artifacts must preserve and visibly identify that authority. |
| LC-04 | MERLIN's replenishment engine is a 2006 min/max reorder-point calculator. | The incumbent calculation remains part of the operational chain. |
| LC-05 | The engine has 340,000 hand-maintained parameters, tuned by category managers. | Existing manual behavior and parameter risk must be included in assessment and operations. |
| LC-06 | MERLIN has no automated test suite. | Integration and change risk cannot assume regression coverage inside MERLIN. |
| LC-07 | MERLIN ships twice yearly through an expensive SAP-certified partner. | Core release cadence is slow and externally constrained. |
| LC-08 | Exactly three MERLIN extension points exist. | They are: (1) override table read before order generation, (2) category-manager review queue after order generation, and (3) nightly batch. |
| LC-09 | Anything integrated with MERLIN must use one of those three points or live entirely outside MERLIN. | No fourth or implicit integration hook may be invented. |
| LC-10 | The MERLIN replenishment engine cannot be modified. | Internal engine changes are prohibited. |
| LC-11 | The 2011 POS estate is offline-first and syncs nightly. | It is not a real-time inventory source. |
| LC-12 | POS is untouchable; a change requires 9-month payment-processor recertification. | The 8-month program cannot depend on POS changes, and the source forbids them regardless of timing. |
| LC-13 | Measured perpetual-inventory accuracy is 78%. | Inventory must be treated as uncertain. |
| LC-14 | Supplier EDI has a fixed schema and fixed 04:00 cut-off for 2,300 suppliers. | Neither schema nor deadline may change. |
| LC-15 | The data warehouse is refreshed nightly and can be 24 hours stale. | Data timing and quality are legacy constraints. |
| LC-16 | Store connectivity is flaky; 4% of stores lose connectivity for > 1 hour weekly. | Store-facing requirements cannot assume continuous WAN service. |
| LC-17 | A 2021 unified-commerce replacement was cancelled after 2 years and $22M. | Core replacement, including a gradual replacement proposal, is off the table. |
| LC-18 | Replenishment must continue whenever AI is unavailable. | AI must be isolated from the ability to break ordering. |
| LC-19 | A Variant B legacy assessment is mandatory. | It must state what can change, what cannot change, and what is unsafe to touch. |
| LC-20 | One AI-Retrofit pattern must be named and defended. | Candidate patterns mandated for evaluation are sidecar, event-driven, API gateway, shadow-assist, and in-process plugin. No selection is made in this baseline. |
| LC-21 | Variant B must include a technically executable kill-switch design. | The mechanism, activation, authority, time to execute, and ordering behavior must be documented later. |
| LC-22 | Explicit fail-open or bypass behavior is mandatory. | Later artifacts must show how incumbent ordering continues without usable AI output. |
| LC-23 | AI recommendations must operate in shadow mode before they may affect orders. | Later artifacts must distinguish the selected integration pattern from shadow-assist as a rollout mode if those are different. |
| LC-24 | Human review is required wherever risk warrants it. | Later artifacts must state how category managers review exceptions and avoid an unbounded review workload. |
| LC-25 | Failure and recovery cases must be explicit. | Later artifacts must cover AI unavailable, late output, low confidence, stale or incomplete input, rollback, kill switch, review, and which system remains authoritative. |

## 5. Legal, safety, and data-residency constraints

| ID | Area | Constraint |
|---|---|---|
| LS-01 | Personal-data residency | One of the three markets prohibits personal loyalty data from leaving the country. The market and exact regulated fields are not identified. |
| LS-02 | POS retention | POS data must be retained for 5 years. |
| LS-03 | Food-safety authority | The 1,400 SOP and food-safety documents are versioned, legally binding, and exist in 3 languages. Product-master data also includes allergens, storage conditions, and shelf life. The authoritative source hierarchy remains to be confirmed. |
| LS-04 | Allergen evidence | Food-safety and allergen responses must define authoritative sources, citation requirements, confidence policy, refusal policy, human escalation, document-version controls, audit record, and accountability. |
| LS-05 | Prohibited safety behavior | An allergen or food-safety answer must not be uncited, unsupported, or freely generated when evidence is absent or conflicting. |
| LS-06 | Criminal liability | A wrong allergen answer is not merely a customer-service failure; it can cause fatal harm and criminal liability. |
| LS-07 | Pricing fairness | Markdown decisions must be auditable because consumer-protection law prohibits arbitrary price discrimination. |
| LS-08 | Data-use legality | Loyalty, scraped competitor pricing, planograms, and other supplied datasets still require lawful-purpose, access-control, and market-specific use confirmation; listing them in the scenario does not itself grant unrestricted use. |
| LS-09 | Cross-border architecture evidence | Data-residency boundaries must be explicit in the architecture and testable in operations. |

## 6. Budget, timeline, and team constraints

| ID | Area | Constraint or known fact |
|---|---|---|
| BT-01 | Delivery timeline | Variant B has an 8-month timeline. |
| BT-02 | Project budget | Variant B has a $3.2M constraint. The exact accounting basis is ambiguous because Variant A defines `$3.2M capex over 18 months`, while Variant B says only `same $3.2M`. |
| BT-03 | Operating cost | Total AI run cost must remain <= $4M/year across forecasting and all other AI capabilities. |
| BT-04 | Margin | Business net margin is 2.4%; cost must be justified in that context. |
| BT-05 | Delivery-team size | No Variant B engineering, data-science, platform, SAP-partner, operations, or support-team composition is stated. The 14 engineers, 3 data scientists, and 1 platform engineer are explicitly part of Variant A and must not be imported into Variant B without confirmation. |
| BT-06 | Workshop team | The group has 5-6 people. The submission contains 6 workshop artifacts, each with one named owner, and must state who owned which artifact. Names are not supplied. |
| BT-07 | Presentation format | Maximum 8 slides; 7 minutes presentation, 4 minutes Q&A, 1 minute changeover, within a 12-minute slot. The scenario must not be re-explained. The orchestration brief further constrains planned speaking time to 6 minutes 30 seconds through 7 minutes. |
| BT-08 | Assignment | Group 4 is assigned Variant B, AI-Retrofit/Legacy. |
| BT-09 | Schedule | Session 1 is July 28, 2:00-4:00 PM, for Groups 1-8. Opening speech is 2:00-2:05 PM; Group 4's slot is 2:05-2:17 PM, followed by trainer summary and feedback as stated by the user. The calendar year is not stated in the assignment text. |
| BT-10 | Artifact 1 | C4 system-context and container views; narrated batch and request flows; trust boundaries; authoritative systems; and human decision points. |
| BT-11 | Artifact 2 | Every NFR with actual numbers, target, proposed mechanism, capacity or latency calculation, failure behavior, monitoring metric, warning threshold, critical threshold, and owner. |
| BT-12 | Artifact 3 | Weighted technology selection across at least 3 consequential decisions. For each: options, criteria, criterion weights, 1-5 option scores, weighted result, selection, and accepted trade-off. Weights must not be manipulated to force a result. |
| BT-13 | Artifact 4 | One ADR for the single most consequential decision, including context, drivers, options, decision, detailed positive and negative consequences, risks, rollback, and revisit conditions. |
| BT-14 | Artifact 5 | Exactly the top 5 architecture risks, selected using quantified likelihood and impact. Each needs category, cause, 1-5 likelihood, 1-5 impact, score, mitigation, contingency, monitoring signal, numeric threshold, and owner. |
| BT-15 | Artifact 6 | Variant B legacy assessment: what can change, cannot change, and is unsafe to touch; selected pattern and rationale; kill switch; fail-open behavior; shadow rollout; human review; rollback; and migration phases that do not replace MERLIN. |
| BT-16 | Presentation content | Each slide requires an objective, headline, no more than 5 visible points, recommended visual, speaker script, and target time. Start with the architecture position rather than the scenario; decisions should be defensible in less than one minute each. |
| BT-17 | Presentation package | Produce an outline, speaker notes, trainer Q&A, and one-page cheat sheet after architecture review. Q&A needs at least 20 difficult questions, each with a strong answer, artifact evidence, weak answer to avoid, and likely follow-up. The final executive summary must cover the 7 requested defense points. |
| BT-18 | Executive summary | State the architecture thesis, selected pattern, most consequential trade-off, how 04:00 is protected, how AI failure is isolated, why the design is affordable, and the 3 hardest trainer questions. |
| BT-19 | Review gate | A red-team review must test the 20 specified challenge questions and issue a `Ready`, `Ready with minor changes`, or `Not ready` verdict. Fix every Critical finding and every Major finding affecting correctness, safety, cost, or compliance; log each accepted or rejected finding and its resolution. |

## 7. Hard constraints versus assumptions

| ID | Classification | Statement | Treatment |
|---|---|---|---|
| HC-01 | Hard constraint | 22:00-04:00 is a non-negotiable six-hour batch wall; 04:00 cannot move. | Validate every relevant design and failure case against it. |
| HC-02 | Hard constraint | MERLIN remains the supplier-order system of record and its replenishment engine cannot change. | Treat as an acceptance criterion. |
| HC-03 | Hard constraint | POS cannot be touched and must remain offline-capable and isolated from AI. | Treat as an acceptance criterion. |
| HC-04 | Hard constraint | Supplier EDI schema cannot change. | Treat as an acceptance criterion. |
| HC-05 | Hard constraint | AI failure must not break or block normal replenishment. | Treat as an acceptance criterion. |
| HC-06 | Hard constraint | Inventory is only 78% accurate and in-day values are estimates. | Do not assume real-time truth. |
| HC-07 | Hard constraint | 6x Lunar New Year peak and 3-10x promotion distortion must be addressed without extending the batch window. | Quantitatively validate. |
| HC-08 | Hard constraint | Personal loyalty data in one market cannot leave that country; allergen and pricing controls are legally material. | Treat as compliance acceptance criteria. |
| HC-09 | Hard constraint | 8 months, $3.2M project constraint, <= $4M/year total AI run cost, and 2.4% margin. | Show complete cost and delivery assumptions. |
| HC-10 | Hard constraint | Maximum 8 slides and a 7-minute presentation; six artifacts, one named owner each. | Treat as submission acceptance criteria. |
| A-01 | Working assumption from the user assignment | The work is for Group 4, Variant B. | Use unless the user changes the assignment. |
| A-02 | Conservative validation assumption | 27M forecasts/day is binding for capacity and cost validation, even though 640 x 11,000 is 7.04M. | Preserve both figures and seek clarification; do not lower the load silently. |
| A-03 | Working assumption | $3.2M is a binding Variant B project ceiling. | Do not assume its capex/opex split or 18-month accounting period. |
| A-04 | Scope assumption not yet approved | All six functional capabilities remain candidates; none has yet been accepted, deferred, or rejected. | Later architecture work must make and defend scope decisions. |
| A-05 | Ownership assumption | Owner names are unavailable. | Use explicit placeholders only until the group supplies names. |
| A-06 | Schedule assumption | July 28 is binding, but no year is inferred. | Confirm the year before calendar-dependent planning. |

## 8. Explicitly prohibited changes

| ID | Prohibited change or design premise |
|---|---|
| PC-01 | Modify, replace, bypass as authority, or "gradually replace" MERLIN's core or replenishment engine. |
| PC-02 | Integrate into MERLIN through any mechanism other than its override table, review queue, or nightly batch; a component may instead live entirely outside MERLIN. |
| PC-03 | Modify POS directly or indirectly, add an AI dependency to the POS revenue path, or require POS recertification. |
| PC-04 | Change the supplier EDI schema. |
| PC-05 | Move or relax the 04:00 supplier cut-off, extend the Lunar New Year window, or depend on a post-cut-off re-run. |
| PC-06 | Make replenishment or truck delivery depend on AI availability, timely AI output, model confidence, or perfect input data. |
| PC-07 | Treat MERLIN inventory or nightly warehouse data as real-time or perfectly accurate shelf truth. |
| PC-08 | Use an LLM for numeric demand forecasting or replenishment optimization; the attached instructions restrict LLM use to genuine language-understanding needs. |
| PC-09 | Provide uncited, unsupported, or generative allergen and food-safety answers. |
| PC-10 | Base the business case on reducing the 8,400-associate headcount. |
| PC-11 | Accept a design that cannot complete before the operational deadline or withstand the required peak. |
| PC-12 | Exceed the budget without a defensible ROI and explicit approval. |
| PC-13 | Hide legal data-residency boundaries, model/version provenance, human accountability, or audit evidence. |
| PC-14 | Add components merely to make the architecture look impressive when a smaller design can meet the same NFRs. |

## 9. Open questions

| ID | Question | Why it matters |
|---|---|---|
| OQ-01 | Which of the six target capabilities are mandatory for the 8-month release, and what is the priority or deferral rule? | The PDF explicitly permits deciding that some are not worth building. |
| OQ-02 | Is the binding daily forecast population 7.04M stocked store-SKU combinations, 26.88M chain-SKU/store combinations, or another filtered count? | Capacity, run time, storage, and cost differ by almost 4x before seasonal peak. |
| OQ-03 | To which inputs or workloads does the 6x Lunar New Year peak apply: transactions, batch records, forecasts, orders, online requests, or all of them? | Quantitative peak proof needs a defined denominator. |
| OQ-04 | How is WAPE calculated and governed: by SKU, store, category, horizon, market, and/or weighted enterprise aggregate? | The <= 25% target is not testable without an agreed formula and slice policy. |
| OQ-05 | Does Variant B's $3.2M mean capex over 18 months, total delivery spend over 8 months, or another budget basis? How does it relate to the separate $4M/year AI run-cost ceiling? | Cost acceptance and ROI depend on scope and accounting period. |
| OQ-06 | What delivery and operational team is actually available for Variant B, including SAP-certified-partner capacity and on-call coverage? | The source gives a team only for Variant A. |
| OQ-07 | Which market has the residency prohibition, which fields count as personal data, and are derived or anonymized features restricted? | Deployment and data-flow boundaries cannot be validated without this. |
| OQ-08 | Which system and document set is authoritative for allergens, sellability, shelf life, recalls, and storage conditions when sources conflict? | Food-safety citations and refusal rules need a source hierarchy. |
| OQ-09 | Who is legally and operationally accountable for food-safety content approval, escalations, and incident response? | The PDF requires named accountability but supplies no role. |
| OQ-10 | What multilingual behavior is required across the 3 document languages, and which languages must work offline? | Safety, latency, storage, and validation scope depend on it. |
| OQ-11 | What does "copilot must work when internet is down" require: all functions, a safety-only subset, cached content, and what maximum staleness? | Offline acceptance criteria are absent. |
| OQ-12 | What are the schemas, throughput limits, locking behavior, batch timing, and support contracts for the 3 MERLIN extension points? | Feasibility and batch-deadline proof need real interface constraints. |
| OQ-13 | What current load and staffing capacity does the category-manager review queue have, and what response time is acceptable? | A required human decision point could become an operational bottleneck. |
| OQ-14 | How is the 78% inventory-accuracy figure measured and segmented by store, category, fresh goods, time of day, shrink, and cycle count? | Uncertainty and quality thresholds require more than an enterprise average. |
| OQ-15 | What data-readiness time, supplier-EDI processing duration, DC processing reserve, and internal stop/abort milestones exist inside 22:00-04:00? | The usable compute window may be materially shorter than six hours. |
| OQ-16 | What acceptance KPIs apply to markdown, substitutions, and promotion planning beyond the board-level waste and out-of-stock targets? | These capabilities lack explicit quality and business thresholds. |
| OQ-17 | What legal rights and freshness expectations apply to scraped competitor-pricing data? | Use may introduce market-specific legal and operational risk. |
| OQ-18 | Which workshop member owns each of the six required artifacts? | The submission requires one named owner per artifact. |
| OQ-19 | What calendar year applies to the July 28 presentation schedule, and is Group 4 definitely the first presentation after the opening speech? | Final rehearsal and submission planning require an unambiguous time. |
| OQ-20 | What baseline and attribution method will determine realized savings and acceptable ROI? | The $241M identified upside is not the same as committed realized benefit. |
| OQ-21 | The PDF labels NFRs as Section 4 but later references "everything in Section 5 (NFRs)" and "every NFR from Section 5." Should those cross-references be read as all NFRs listed in Section 4? | This baseline assumes yes, but records the source-document numbering inconsistency. |

## 10. Requirement-to-artifact traceability table

Artifact legend: A1 = C4 context and container diagrams plus narrated flows; A2 = numeric NFR table; A3 = weighted technology-selection matrix; A4 = one ADR; A5 = exactly top 5 risks; A6 = Variant B legacy assessment, chosen pattern, and kill-switch design. The final maximum-8-slide presentation is denoted P.

| Requirement IDs | Primary workshop artifact(s) | Required evidence |
|---|---|---|
| BO-01 to BO-05 | A2, A5, P | Current-to-target KPIs, measurable operating signals, and explicit preservation of 8,400 associates. |
| BO-06 to BO-07, BC-01 to BC-09 | A2, A3, A5, P | Scale and complete cost arithmetic tied to 2.4% margin and $241M identified upside. |
| FC-01 to FC-02 | A1, A2, A3, A4, A5 | Forecasting and replenishment scope, flows, quantitative constraints, consequential decisions, and risks. |
| FC-03 to FC-06 | A1, A2, A3, A5 | Clear inclusion/defer decisions, request flows where applicable, latency/safety/cost targets, and risks. |
| NFR-01 to NFR-05 | A1, A2, A3, A5, P | Stepwise batch flow, arithmetic for 22:00-04:00, both forecast-count figures, 6x peak, 3-10x promotions, retries, and deadline behavior. |
| NFR-06 to NFR-09 | A1, A2, A3, A5 | p95 and offline acceptance numbers, markdown cadence, mechanisms, failure modes, metrics, thresholds, and owners for included capabilities. |
| NFR-10 to NFR-11 | A1, A2, A4, A5, A6, P | POS isolation, uninterrupted incumbent ordering, authority, failure behavior, rollback, and kill-switch evidence. |
| NFR-12 | A2, A3, A5, P | Full annual AI cost, daily cost, cost/forecast, peak cost, assumptions, and trade-offs. |
| NFR-13 to NFR-14 | A1, A2, A3, A5, A6 | Inventory uncertainty and stale-data boundaries, validation, failure behavior, and measurable data-quality thresholds. |
| NFR-15 to NFR-17, LS-01 to LS-09 | A1, A2, A3, A5, P | Residency boundaries; authoritative safety sources; citation, refusal, escalation, version, audit, accountability, retention, and pricing controls. |
| NFR-18 to NFR-19 | A1, A2, A3, A4, A5, A6 | Audit records, model/version traceability, numeric alert thresholds, owners, support model, and visible separation of workload/model types. |
| LC-01 to LC-09 | A1, A3, A4, A6, P | MERLIN boundary and authority, exactly 3 extension points, pattern evaluation, and consequential decision record. |
| LC-10 to LC-18 | A1, A2, A4, A5, A6 | Immutable boundaries, POS/EDI/inventory/network constraints, AI-unavailable behavior, and replacement prohibition. |
| LC-19 to LC-25 | A1, A2, A3, A4, A5, A6, P | Legacy assessment, evaluation of all 5 named patterns, one defended selection later, fail-open behavior, shadow-first rollout, human review, failure cases, rollback, and executable kill switch. |
| BT-01 to BT-05 | A2, A3, A5, P | Delivery, project budget, run-cost, margin, and support assumptions with unresolved items identified. |
| BT-06 to BT-19 | A1 to A6, P | Six complete artifacts, one named owner each, exactly top 5 risks, at least 3 weighted decisions, required artifact fields, review gate, maximum 8 slides, 6:30-7:00 planned speaking time, full coaching package, and executive summary. |
| HC-01 to HC-10, PC-01 to PC-14 | A1, A2, A3, A4, A5, A6, P | Cross-artifact acceptance checklist demonstrating that no prohibited dependency or change has been introduced. |
| A-01 to A-06, OQ-01 to OQ-21 | A1 to A6, P | Assumption labels, decisions or owner-confirmed answers, and no silent resolution of material ambiguities. |
