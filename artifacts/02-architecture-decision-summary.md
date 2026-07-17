# NovaMart Variant B - Architecture Decision Summary

Owner: LÊ NGUYỄN SỸ BÌNH (122980, ARD)

Status: Proposed for quantitative validation

Source of constraints: `artifacts/01-requirement-baseline.md`

## 1. Architecture thesis

Add a separately deployed, asynchronous AI sidecar that reads only approved nightly snapshots, produces forecast-backed replenishment recommendations, and can publish bounded overrides only through MERLIN's supported override-table extension. MERLIN continues to generate, review, and send every supplier order. Absence, lateness, low confidence, invalid data, or deliberate disabling of the sidecar results in no AI override for that run; MERLIN's incumbent min/max process continues unchanged.

The primary integration pattern is **sidecar**. **Shadow-assist is a rollout mode** used before any recommendation is allowed to affect an order. Internal batch tasks may use queues or events for orchestration, but there is no event-driven dependency across the MERLIN boundary.

## 2. Authority and safety invariants

1. MERLIN remains authoritative for merchandising, inventory records, replenishment execution, supplier orders, and supplier EDI.
2. The MERLIN engine, POS estate, and EDI schema are unchanged. The AI sidecar never calls POS and never appears on the POS revenue path.
3. The only MERLIN crossings are the existing nightly batch, override table, and category-manager review queue.
4. A recommendation is advisory until the supported adapter writes it to the override table. The sidecar cannot create or transmit a supplier order.
5. MERLIN's nightly run is scheduled independently of the sidecar. It never waits for an AI response.
6. Only a complete, current-run, traceable recommendation set can be published. Partial, stale, late, or untraceable sets are discarded.
7. Inventory is an uncertain observation, not shelf truth. A single 78%-accurate stock-on-hand value is never used as a deterministic fact.
8. Forecasting uses statistical ML; order quantity selection uses deterministic constraints and optimization. A language model may classify/translate an input or assist with non-safety questions only. Allergen, food-safety, recall, sellability, storage, and shelf-life displays are deterministic exact approved fields/passages plus fixed wording and citations; an LLM never composes them.

## 3. Scope decision for the 8-month release

The PDF makes all six capabilities candidates and permits explicit deferral. The smallest defensible release is:

| Capability | Release decision | Architectural treatment |
|---|---|---|
| SKU-store-day demand forecasting | Commit | Batch statistical forecasting with prediction intervals, promotion features, versioned models, and WAPE monitoring. Capacity is validated against both 7.04M stocked combinations and the conservative 26.88M/27M requirement. |
| Automated replenishment | Commit, shadow first | Deterministic optimization applies case pack, shelf life, DC capacity, confidence, uncertainty, and bounded-change rules. MERLIN remains the order generator and authority. |
| Promotion planning support | Include as batch output, not a new platform | Promotion lift and cannibalization features share the forecasting pipeline; category managers receive traceable scenarios outside the order path. |
| Associate copilot | Limited read-only pilot, isolated from POS and ordering | Governed retrieval over approved SOP and product content with a signed store-local cache. An optional language service is confined to intent/entity extraction and non-safety language assistance. Safety-domain output is deterministic exact extract/structured-field rendering with fixed wording and a citation, or refusal/escalation. |
| Fresh markdown | Defer production activation | No safe pricing write interface, fairness policy, or acceptance KPI is defined. The sidecar may produce offline analysis only; no price is changed. Revisit after a separately approved, auditable integration exists. |
| Online substitutions | Defer production activation | No online-order interface or ownership boundary is documented. The <500 ms NFR becomes an entry criterion for a later design, not an unproved claim in this release. |

This scope addresses the highest-value ordering problem without inventing integrations. It does not claim that deferred capabilities meet their operational NFRs.

## 4. Workload separation

| Workload | Technique | Inputs | Output | Prohibited behavior |
|---|---|---|---|---|
| Demand forecast | Statistical or machine-learning batch model | Aggregated nightly sales, calendar, promotion, weather, product and store attributes, uncertainty indicators | Quantiles and expected demand per SKU-store-day plus confidence | No LLM-generated numbers; no personal loyalty data in Release 1 |
| Replenishment recommendation | Deterministic rules and constrained optimization | Forecast distribution, shelf life, case pack, DC capacity, lead time, bounded inventory estimate, business constraints | Recommended order quantity, reason codes, constraint results | No free-form generation; no direct EDI; no assumption that inventory is exact |
| Associate copilot pilot | Governed retrieval; optional LLM for input classification/entity extraction and non-safety language only; deterministic safety renderer | Signed versions of approved SOP/food-safety documents and product master | Non-safety cited answer; safety exact approved field/passage plus fixed template/citation; or refusal/escalation | No LLM-composed safety answer, numeric forecasting, optimization, price decision, uncited claim, or POS dependency |

## 5. Selected retrofit pattern

### Decision

Use an **external sidecar** with an on-premises integration zone adjacent to MERLIN and an isolated batch compute/data plane. It reads approved snapshots at the nightly-batch extension, stages recommendations outside MERLIN, and uses two narrowly scoped adapters:

- an override-table adapter that atomically activates only current-run, policy-approved AI overrides under a transaction-level publication epoch;
- a review-queue adapter that receives the documented post-generation order ID and generated quantity from the supported queue contract, joins them to the exact publication ledger key, and then adds bounded provenance/context.

The sidecar has no pre-generation read of MERLIN's current min/max output. Before publication it uses only absolute policy caps and source facts: non-negative whole case packs, no more than 2 case packs per store-SKU in the initial pilot, and the lower of the approved shelf-life sell-through, inventory-interval, DC-allocation, and category limits. The first production-like operation is shadow-assist: compute and retain a recommendation, but do not write the override table. Comparison with MERLIN occurs only after generation through a certified queue/reporting output; if no documented output exists, no current-run comparison is claimed and influence remains blocked.

### Pattern evaluation

| Pattern | Fit | Decision and reason |
|---|---|---|
| Sidecar | High | **Selected.** It isolates AI failure, permits independent releases, works with nightly batch interfaces, and leaves MERLIN authoritative. Its added data synchronization is accepted and controlled with snapshot IDs and freshness gates. |
| Event-driven integration | Low across the legacy boundary | Rejected as the primary pattern. POS and the warehouse are nightly, MERLIN exposes no reliable real-time event stream, and introducing a new event contract would invent a fourth extension mechanism. Internal sidecar orchestration events remain an implementation detail only. |
| API gateway | Low | Rejected. MERLIN does not expose a documented real-time replenishment API, and a synchronous gateway would put AI availability on the ordering path. It adds no value to the fixed nightly batch. |
| Shadow-assist | High as rollout, incomplete as architecture | Required as the initial **rollout mode**, not selected as the integration architecture. It describes whether recommendations influence decisions, not the deployment or interface boundary. |
| In-process plugin | Unacceptable | Rejected. It conflicts with the unmodifiable engine, twice-yearly SAP-certified release, missing regression suite, and the requirement that AI failure cannot impair MERLIN. |

## 6. MERLIN extension-point mapping

| Supported point | Use | Boundary rule |
|---|---|---|
| Existing nightly batch | Emits a read-only, immutable, dated snapshot and start signal to the ingestion validator | The sidecar does not alter the incumbent batch schedule or wait state. If the snapshot is absent or invalid, the sidecar stops and MERLIN continues. |
| Override table read before order generation | Receives an atomic set of absolutely bounded, approved, current-run recommendations from the adapter | The sidecar does not read a current MERLIN result. The certified contract must provide a transaction-level epoch guard and either supported non-live bulk staging plus constant-time activation, or a measured direct-write capacity. Actual cap is `min(policy ceiling, floor(R_cert x (L_cert - T_overhead_p99) x 0.80))`; unsafe/unsupported behavior means shadow-only. |
| Review queue after order generation | Supplies documented generated order ID/quantity/correlation keys to the adapter and presents only admitted AI-affected orders with provenance | The post-generation join is inside the existing queue extension. If its read fields, correlation, native actions, timeout/default, or capacity cannot be certified, no interface is invented: automated influence stays disabled and comparison remains limited to documented historical/reporting outputs. |

## 7. Protecting 04:00

The sidecar is never a predecessor that MERLIN must wait for. The proposed internal gates below reserve two hours for the incumbent order-generation, review, and EDI path. These are architecture budgets to be verified against measured MERLIN timings in discovery (OQ-15); if the incumbent reserve is insufficient, every AI gate moves earlier and shadow mode continues.

| Clock | Sidecar gate | Deadline behavior |
|---|---|---|
| 22:00-22:20 | Snapshot readiness, schema, row-count, freshness, residency, and completeness validation | Any critical failure stops the AI run. MERLIN is unaffected. |
| 22:20-00:20 | Feature calculation and statistical forecasting | Work is partitioned by market/store. Retries are bounded by the 01:30 retry-stop gate. |
| 00:20-01:20 | Deterministic optimization and policy checks | Invalid or low-confidence rows are suppressed, not guessed. |
| 01:20-01:45 | Reconciliation, trace manifest, risk scoring, and complete-set validation | A missing model, ruleset, input, audit, or checksum blocks publication. |
| 01:45-02:00 | Stage/checksum outside the live read set, then epoch-guarded atomic activation if the supported contract permits; otherwise use a certified direct transaction with a measured lower cap | On any partial write, stale epoch, expired permit, or lock/rate breach, abort/roll back. No AI writes are allowed after 02:00. The 70,400 ceiling needs at least 1,173.4 rows/s inside 60 s before overhead; with a 10 s overhead reserve and 20% headroom it needs 1,760 rows/s. |
| 02:00-04:00 | Incumbent MERLIN generation, native review, and fixed-schema EDI | MERLIN runs whether or not an AI run exists. The sidecar cannot trigger a re-run after 04:00. |

The exact current MERLIN order-generation start, P95/P99 duration, and table-lock behavior are unknown. Production influence is prohibited until measurement proves the 02:00 gate leaves at least the incumbent P99 duration plus a 30-minute operational buffer. Shadow mode does not require that assumption.

## 8. Inventory uncertainty

The 78% perpetual-inventory accuracy is modeled explicitly:

- forecasts estimate demand; they do not claim shelf availability;
- the optimizer consumes an inventory range or probability distribution derived from age, last sync, shrink history, store/category error, and any confirmed cycle count;
- pre-publication recommendations are capped without a current MERLIN baseline: non-negative whole case packs, at most 2 case packs/store-SKU in the initial pilot, and the lower of approved shelf-life sell-through, inventory-interval, DC-allocation, and category absolute limits;
- low-confidence or high-uncertainty rows produce no AI override or enter the capacity-limited review queue;
- the system records the input snapshot age and uncertainty score for every recommendation;
- forecast quality and order outcomes are sliced by store, category, promotion, and inventory-quality band so errors are not hidden in an enterprise average.

No design component reads in-day POS or calls the stock figure "real time."

## 9. Failure, bypass, and rollback behavior

| Condition | Sidecar behavior | MERLIN behavior |
|---|---|---|
| AI platform unavailable | No publish attempt; alert operations | Runs incumbent min/max and EDI normally |
| Output not complete by 01:45 or write not committed by 02:00 | Discard the run; do not publish late rows | Runs without current-run AI overrides |
| Forecast confidence below the approved threshold | Suppress that row; optionally risk-rank it for human analysis | Uses incumbent result for that row |
| Input stale, incomplete, schema-invalid, or residency-invalid | Quarantine affected partition; fail the complete-set gate for an affected pilot scope | Continues with legacy data and rules |
| Optimizer violates a hard constraint | Reject the row and record the failed constraint | Uses incumbent result |
| Adapter transaction partially fails or locks MERLIN | Roll back all AI-owned writes, open circuit, activate bypass, page SAP operations | Continues once the supported transaction is released; no retry after the hard gate |
| Audit or model/ruleset provenance unavailable | Block publication | Continues unchanged |
| Harmful behavior detected after publish but before MERLIN reads | Activate kill switch; stop adapter and delete only ledger-identified, unconsumed AI rows through the supported table procedure | Runs incumbent logic |
| AI effect already generated into orders | Freeze future AI runs; use native review queue to reject/correct affected orders if the existing interface permits before EDI | MERLIN remains authoritative; no direct sidecar reversal or post-cutoff re-run |

## 10. Kill switch

The kill switch is a deny-by-default transaction control, not an application flag.

1. A durable, quorum-backed on-prem latch stores a signed record `{epoch, state, run_id, scope_hash, manifest_hash, not_after}`. Allowed transitions are `DISABLED -> ARMED_SHADOW -> ENABLED_PUBLISH -> DISABLING -> VERIFIED_SAFE -> ARMED_SHADOW`; every transition increments a monotonic epoch. Unreachable, unsigned, expired, non-current, or invalid state is `DISABLED`.
2. The adapter has no standing write credential. A local credential broker issues a single-run, scope/manifest-bound permit valid for <=60 seconds only for `ENABLED_PUBLISH`. The certified override path must evaluate the same current epoch inside the atomic activation/direct-write transaction immediately before commit; a stale/expired permit aborts the transaction. If this guard cannot be implemented through the supported contract without a new MERLIN hook/schema change, influence is prohibited.
3. Either authorized Duty Manager changes the latch to `DISABLING`, creating a new epoch. Within 30 seconds the latch quorum, broker, DB/network gateway, orchestrator, and session monitor must acknowledge; missing acknowledgement invokes the independent SAP stop. No new write may be accepted after 60 seconds.
4. The independent out-of-band SAP stop uses separate break-glass authority to revoke permit issuance and the adapter's narrowly scoped DB/network access and terminate active adapter sessions. It does not depend on the adapter or sidecar control service. Epoch change/session termination atomically aborts and rolls back uncommitted work; supported staged rows remain inert.
5. A separate certified cleanup identity/runbook, not the adapter identity, reconciles the immutable pre-activation exact-key/hash ledger with MERLIN audit evidence and removes only unconsumed AI-owned rows. Manual rows are never selected. Already consumed rows use native pre-EDI correction only; there is no compensating sidecar EDI or post-04:00 rerun.
6. `VERIFIED_SAFE` requires: no valid permits, no active write session/transaction, zero unexplained activated rows, exact cleanup/checksum result, and acknowledgements/audit IDs. Target is <=5 minutes; failure leaves the state `DISABLING`, access revoked, and MERLIN running independently.
7. Re-enable cannot jump to publish. It requires incident closure, clean evidence, a successful current-data `ARMED_SHADOW` run, approved model/data/rules, and dual Replenishment Operations/Model Risk approval for a new epoch.

Pre-pilot and monthly drills inject latch/control loss, stale cached epoch, a check/write race, adapter failure/compromise, missing acknowledgement, DB/network revocation, and cleanup failure. Any failed drill blocks influence.

## 11. Human review without queue overload

- Shadow mode stores recommendations outside MERLIN. A comparison is shown only after a documented supported queue/reporting output supplies the generated order quantity; otherwise no current-run comparison is claimed.
- In assisted mode, the supported review-queue output must supply generated order ID/quantity and correlation keys. The adapter joins those fields to the exact publication ledger and annotates only admitted AI-affected orders over an absolute value/risk threshold.
- Queue admission is `min(risk-ranked exceptions, measured available reviewer capacity)`. Lower-ranked items fall back to MERLIN; they do not accumulate indefinitely.
- The initial pilot is limited to 20 stores and 2 low-risk, non-fresh categories, with an interim cap of 100 AI exceptions per night until OQ-13 establishes real capacity.
- Category managers see the post-generation MERLIN quantity only when supplied by the certified queue contract, plus AI quantity, post-generation delta, forecast interval, inventory-uncertainty band, constraints, reason code, model/ruleset version, and expiry time.
- Fresh, allergen-related, regulated-price, extreme-delta, low-confidence, and out-of-policy cases are never auto-approved.
- Every accept, reject, expiry, and manual edit is audited. Expired or unreviewed cases use the incumbent MERLIN result.

## 12. Data, residency, and audit boundaries

- Release 1 forecasting excludes member-level loyalty data. It uses store-SKU-day aggregates without member identifiers.
- Personal loyalty data remains inside its country-local trust boundary. Any future use needs a market-specific legal basis and a country-local pipeline.
- POS is retained under the existing five-year control. The sidecar stores only approved derived snapshots for the minimum documented retention period.
- Weather, calendar, and event data are non-authoritative features. Scraped competitor pricing is excluded until legal rights and freshness are approved.
- Each recommendation records: run ID; market/store/SKU/date; snapshot IDs/ages; feature and model versions; forecast quantiles/confidence; inventory interval; optimizer/ruleset and absolute-cap results; AI recommendation; publication epoch/permit/transaction and exact keys. Post-generation MERLIN quantity/order reference and human decision are appended only when the certified queue/reporting contract supplies them; they are never assumed as pre-publication inputs.
- Audit records are append-only and access-controlled. Missing provenance blocks publication.

## 13. Rollout and rollback

1. **Discovery and contract proof:** measure MERLIN batch timings, table locking, queue semantics, manual-override ownership, and deletion behavior. No production writes.
2. **Historical replay:** reproduce past nights and 6x peak inputs; compare WAPE, order bounds, timing, and cost.
3. **Shadow-assist:** run for at least 4 consecutive weeks including promotion conditions; write nothing to MERLIN.
4. **Human-reviewed pilot:** 20 stores and 2 low-risk categories; queue-capped; kill-switch exercise required before activation.
5. **Bounded influence:** permit only approved confidence/uncertainty and absolute quantity/value bands; compute a relative delta only after a certified post-generation MERLIN result exists; expand by store/category after business, safety, cost, and operations gates.
6. **Scale:** retain the same sidecar boundary and MERLIN authority. Expansion changes configuration and capacity, not the core architecture.

Rollback at any phase disables publication, clears only unconsumed AI-owned overrides through the supported procedure, leaves MERLIN's engine and manual parameters intact, and returns to shadow observation. It never replaces or forks MERLIN.

## 14. Assumptions and validation gates

| Item | Current treatment |
|---|---|
| 27M versus 7.04M forecast population | Capacity and cost validation use 27M/day; both counts remain visible. |
| 6x peak denominator | Conservatively load-test feature ingestion and batch compute at 6x while retaining the fixed 04:00 wall. Forecast output cardinality is measured separately. |
| MERLIN reserve after 02:00 | Unproven. Production influence is blocked until incumbent P99 plus 30 minutes fits before 04:00. |
| Override-table atomicity, transaction epoch, capacity, and safe cleanup | Unproven. The contract must prove same-transaction epoch guard, out-of-band revocation, exact cleanup, and either supported staged activation or a measured cap using `floor(R_cert x (L_cert - T_overhead_p99) x 0.80)`; otherwise shadow-only. |
| Current MERLIN generated quantity | Not available pre-publication. Production influence is blocked until the supported post-generation queue/reporting contract exposes generated order ID/quantity and correlation keys; no fourth read interface or engine replica is allowed. |
| Review capacity | Unknown. Pilot cap is 100/night until measured capacity and service level are agreed. |
| Variant B team and support | Unknown. Delivery and on-call design must be sized before cost approval. |
| $3.2M basis | Treated as an 8-month delivery ceiling without assuming capex/opex split; finance must confirm. |
| Authoritative safety hierarchy | Unresolved. Copilot stays in read-only pilot and refuses conflicts until accountable owners approve the hierarchy. |
| July schedule year | Not inferred here; it does not affect the architecture. |
