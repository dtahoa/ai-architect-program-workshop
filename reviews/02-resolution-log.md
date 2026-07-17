# NovaMart Group 4 - Red-Team Resolution Log

Resolution owner: Developer correction pass

Source review: `reviews/01-red-team-review.md`

Status: **RT-01 through RT-05 accepted and resolved in the artifact set. RT-06 and RT-07 were not applied because they are minor presentation/readability suggestions outside the authorized correction scope.** All production claims remain conditional on measured evidence and named-owner approval.

## Finding dispositions

### RT-01 - Critical - Generative allergen and food-safety output

- Disposition: **Accepted**
- Reason: A cited LLM paraphrase can change a safety qualifier and violates the explicit non-generative safety boundary.
- Artifacts changed: `artifacts/02-architecture-decision-summary.md`, `03-c4-context.mmd`, `04-c4-container.mmd`, `05-request-and-batch-flows.md`, `06-legacy-assessment.md`, `07-nfr-table.md`, `08-capacity-and-cost-check.md`, `09-safety-and-governance.md`, `10-technology-selection-matrix.md`, `11-adr.md`, `12-risk-register.md`, `13-variant-b-artifact.md`, `14-final-artifact-pack.md`.
- Final resolution: All allergen, food-safety, recall, sellability, storage, and shelf-life output is routed to deterministic rendering of an exact approved structured field/passage plus fixed pre-approved locale wording and the exact citation. The LLM may classify/translate input, extract entities, or assist with non-safety wording only; it may never compose, paraphrase, translate, or summarize displayed safety content. Missing, stale, conflicting, inexact, unsigned, or uncited evidence refuses and escalates. Audit includes renderer/template and output/source hashes.

### RT-02 - Major - Unsupported pre-publication MERLIN baseline

- Disposition: **Accepted**
- Reason: The documented extension set provides an override input before generation and a review queue after generation; it does not prove a current MERLIN quantity is available before publication.
- Artifacts changed: `artifacts/02-architecture-decision-summary.md`, `03-c4-context.mmd`, `04-c4-container.mmd`, `05-request-and-batch-flows.md`, `06-legacy-assessment.md`, `07-nfr-table.md`, `08-capacity-and-cost-check.md`, `09-safety-and-governance.md`, `11-adr.md`, `12-risk-register.md`, `13-variant-b-artifact.md`, `14-final-artifact-pack.md`.
- Final resolution: Pre-publication delta/baseline-relative rules were removed. Recommendations use non-negative whole cases with absolute quantity/value and shelf-life, inventory-interval, DC-allocation, and category caps; the initial quantity maximum is two case packs/store-SKU and the lower constraint wins. Current-run comparison/annotation is allowed only when a certified post-generation contract returns generated quantity, order ID, and correlation that join to exact publication keys. If those fields or the join are unavailable, no current-run comparison is claimed and automated influence remains disabled.

### RT-03 - Major - Incomplete kill-switch correctness model

- Disposition: **Accepted**
- Reason: A mutable flag checked before a transaction does not prevent a check/write race, stale process, adapter compromise, or failure of the primary control path.
- Artifacts changed: `artifacts/02-architecture-decision-summary.md`, `03-c4-context.mmd`, `04-c4-container.mmd`, `05-request-and-batch-flows.md`, `06-legacy-assessment.md`, `07-nfr-table.md`, `08-capacity-and-cost-check.md`, `09-safety-and-governance.md`, `11-adr.md`, `12-risk-register.md`, `13-variant-b-artifact.md`, `14-final-artifact-pack.md`.
- Final resolution: Publication now uses a durable deny-default quorum latch containing `{epoch,state,run_id,scope_hash,manifest_hash,not_after}` and monotonic states `DISABLED -> ARMED_SHADOW -> ENABLED_PUBLISH -> DISABLING -> VERIFIED_SAFE -> ARMED_SHADOW`. The adapter has no standing credential; a broker issues a single-run/scope permit valid no more than 60 seconds; the certified transaction checks the current epoch immediately before commit in the same transaction. Control acknowledgement is due within 30 seconds, no new write is accepted after 60 seconds, and verified safe is due within five minutes. Independent SAP break-glass revokes DB/network access and terminates sessions, and a separate identity/runbook cleans exact unconsumed ledger rows only. Re-enable requires a new epoch and current-data shadow run. Drills cover latch loss, stale epoch, check/write race, adapter compromise, missing acknowledgement, out-of-band revoke/session termination, and cleanup failure.

### RT-04 - Major - Publication throughput arithmetic and unsupported staging

- Disposition: **Accepted**
- Reason: `70,400/60` is 1,173.4 rows/s, not the earlier stage-average figure, and a new staging/swap mechanism cannot be assumed inside the supported legacy contract.
- Artifacts changed: `artifacts/02-architecture-decision-summary.md`, `04-c4-container.mmd`, `05-request-and-batch-flows.md`, `06-legacy-assessment.md`, `07-nfr-table.md`, `08-capacity-and-cost-check.md`, `11-adr.md`, `12-risk-register.md`, `13-variant-b-artifact.md`, `14-final-artifact-pack.md`.
- Final resolution: The policy ceilings are not capacity claims. Actual direct cap is `min(policy ceiling, floor(R_cert x (L_cert - T_overhead_p99) x 0.80))`, with `L_cert <=60 s`. Raw minimum rates are 83.4 rows/s for 5,000 and 1,173.4 rows/s for 70,400; with 10 seconds of overhead and 20% headroom they are 125 and 1,760 rows/s. An existing supported non-live staging path may be used only if certified and its epoch-guarded activation is <=5 seconds. No new table, schema, or swap is invented. Otherwise the measured direct formula sets a lower cap or the system remains shadow-only.

### RT-05 - Major - Offline usefulness could pass through universal refusal

- Disposition: **Accepted**
- Reason: Latency and safe-refusal checks alone do not show that the offline copilot answers eligible questions usefully.
- Artifacts changed: `artifacts/05-request-and-batch-flows.md`, `07-nfr-table.md`, `08-capacity-and-cost-check.md`, `09-safety-and-governance.md`, `12-risk-register.md`, `13-variant-b-artifact.md`, `14-final-artifact-pack.md`.
- Final resolution: Offline acceptance requires 100% signed-pack coverage and p95 <=500 ms; eligible non-safety answer coverage >=90% with answer/citation accuracy >=95%; ineligible non-safety refusal precision and recall each >=95%; and safety policy correctness, eligible exact display/citation correctness, and required-refusal precision and recall all 100%. Benchmarks label eligible-answer and required-refusal cases separately. An empty valid pack or universal-refusal implementation fails acceptance.

### RT-06 - Minor - Presentation density

- Disposition: **Rejected for this correction pass**
- Reason: This is a presentation/readability recommendation, not a functional, security, safety, arithmetic, or test-coverage defect. The parent task authorized fixes only for real functional/security/test issues after review.
- Artifacts changed: None specifically for RT-06.
- Final resolution: Deferred to presenter rehearsal and slide editing; no architecture claim was changed on this basis.

### RT-07 - Minor - Terminology density

- Disposition: **Rejected for this correction pass**
- Reason: This is a wording/readability recommendation. Applying it broadly would create unnecessary churn after the critical/major consistency correction.
- Artifacts changed: None specifically for RT-07.
- Final resolution: Terms remain defined in their local control/flow context; simplification is deferred to presentation rehearsal.

## Verification record

The correction pass validates the Markdown/table/fence structure, exact five-risk constraint, matrix arithmetic, required section headings, stale-claim searches, and cross-artifact presence of the five corrected control contracts. Command results are recorded in the parent task's final verification summary; any unavailable renderer or external MERLIN/SAP evidence remains an explicit release risk rather than a claimed pass.
