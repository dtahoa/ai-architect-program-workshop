# NovaMart Variant B - Legacy Assessment and Retrofit Control Design

Owner: TRẦN TRỌNG PHÚ (197320, PH2)

Status: Proposed; production influence is conditional on the discovery and shadow exit gates below

## 1. Legacy position

MERLIN is not a migration source or a temporary dependency. It remains NovaMart's ordering system of record for the life of this design. The retrofit adds an advisory sidecar around stable extension points and preserves the incumbent min/max engine as the always-available fallback.

The 2021 replacement failure, twice-yearly SAP-certified release, 340,000 hand-maintained parameters, and absent automated test suite make core change both unsafe and unnecessary. The safe unit of change is an independently releasable sidecar plus narrowly certified adapters, not MERLIN, POS, or EDI.

## 2. Legacy estate assessment

| Legacy element | Current fact | Architecture consequence |
|---|---|---|
| MERLIN ERP | On-premises SAP-based system customized since 2006 | Treat as a fixed trust boundary and authority, not a platform to refactor |
| MERLIN replenishment engine | 2006 min/max calculator; 340,000 manually tuned parameters; no automated tests | Do not modify, embed AI, or bypass. Preserve it as fail-open fallback and comparison baseline |
| MERLIN releases | Twice yearly through an expensive SAP-certified partner | Keep adapters small, contract-tested, independently deployable where possible, and within supported interfaces |
| Override table | Read by MERLIN before order generation | Only supported influence point for bounded, current-run recommendations; exact schema, locking and ownership semantics must be certified |
| Review queue | Available to category managers after order generation | Use only for capacity-capped AI-affected order context and existing human actions; do not invent new approval semantics |
| Existing nightly batch | One of exactly three extension points | Use as read-only snapshot/start handoff; never make its success depend on AI |
| POS | 2011 offline-first estate; about 7,000 lanes; nightly sync; 9-month recertification to change | Untouchable and excluded from every AI runtime dependency and request path |
| Inventory | Nightly updates; 78% measured accuracy; in-day estimate | Treat as uncertain range/probability with data age and quality bands; never claim real-time shelf truth |
| Supplier EDI | 2,300 suppliers; fixed schema and 04:00 cut-off | MERLIN alone sends orders. No schema, endpoint, timing, or sidecar route changes |
| Data warehouse | Nightly ETL; up to 24 hours stale | Use only dated immutable snapshots with freshness policy and uncertainty; never present as live |
| Store network | 4% of stores lose connectivity for more than an hour weekly | Batch ordering cannot depend on store WAN. Copilot pilot uses a separate signed local cache and safe offline refusal |
| Unified-commerce replacement history | Cancelled after 2 years and $22M | No core replacement, strangler, gradual replacement, dual order authority, or new supplier-order platform |

## 3. Change classification

### What can change

| Changeable surface | Allowed change |
|---|---|
| External AI sidecar | Add independently deployed ingestion, validation, features, statistical forecasting, deterministic optimization, recommendation staging, model/rules registry, monitoring, and immutable audit |
| Nightly-batch extension | Add a supported read-only dated snapshot/start handoff without delaying or altering the incumbent job |
| Override-table extension | Through a certified adapter, activate only AI-owned current-run rows under a same-transaction epoch guard; use supported non-live staging/constant-time activation or a measured direct-write cap |
| Review-queue extension | Through the supported interface, receive generated order ID/quantity/correlation keys after generation, join to the exact publication ledger, and attach capacity-capped provenance |
| Operational controls | Add a durable deny-default publication latch, monotonic epochs, short-lived transaction permits, independent credential/session revocation, hard deadline, scope caps, alerts, and separate cleanup runbook |
| Analytics | Add recommendations, outcome measurement, model monitoring, promotion scenarios, and historical replay outside MERLIN; compare current-run quantities only after a documented supported output exists |
| Separate associate tool | Pilot deterministic exact safety retrieval/display and optional non-safety language assistance on non-POS devices with signed offline content, citations, refusal, and escalation |
| Non-personal data plane | Process approved aggregated store-SKU data and non-authoritative weather/calendar/event features in an approved deployment region |

### What cannot change

| Fixed surface | Boundary |
|---|---|
| MERLIN authority | MERLIN remains authoritative for supplier orders, merchandising, replenishment execution, and EDI |
| MERLIN engine | No code, algorithm, call sequence, embedded model, hook, or dependency change |
| Extension inventory | No fourth MERLIN hook; integrations use only nightly batch, override table, review queue, or remain entirely external |
| POS | No code, agent, plugin, library, API call, schema, device-path, deployment, timing, or recertification impact |
| Supplier EDI | No schema, sender authority, endpoint contract, or 04:00 cut-off change |
| Incumbent continuity | MERLIN must run and stores must receive morning trucks if every AI component is unavailable |
| Core replacement prohibition | No gradual replacement, parallel order master, strangler of MERLIN, or migration of supplier-order authority |
| Legal constraints | No cross-border personal loyalty export, uncited safety answer, arbitrary price discrimination, or loss of required POS retention |

### What is unsafe to touch

| Unsafe area | Why unsafe | Safe alternative |
|---|---|---|
| Internal min/max code or its 340,000 parameters at scale | No regression suite; high operational blast radius; SAP partner release constraint | Compare against it in shadow and apply only bounded external overrides |
| Direct writes to undocumented MERLIN tables | Locking, ownership, stale-row, and corruption risk | Certified override-table adapter using the documented extension contract only |
| Manual override rows | They encode category-manager intent and may share the table | External publication ledger identifies exact AI-owned keys/hashes; manual rows are never selected |
| MERLIN scheduler or critical path | A wait or callback could miss 04:00 | Independent sidecar deadline; no synchronous dependency; hard close at 02:00 or earlier |
| POS runtime, database, lanes, network or sync contract | Revenue-path availability and 9-month recertification exceed the program | Consume existing nightly aggregate exports only; no sidecar-to-POS route |
| Supplier EDI connector | 2,300 fixed integrations and immovable cut-off | MERLIN continues sending the same fixed-schema orders |
| A point stock-on-hand value as truth | More than 1 in 5 records may differ from the shelf | Use uncertainty bands, age, shrink history, bounds and suppression |
| Unbounded native review queue | Human bottleneck can delay or normalize unsafe approvals | Risk-rank, cap admission, suppress excess, measure reviewer capacity |
| Member-level loyalty data in a shared regional pipeline | One market prohibits personal data leaving country | Exclude from Release 1; require a future country-local legal/data design |
| Generative allergen or food-safety answer | Potential fatal harm and criminal liability | Exact approved structured field/passage in the approved locale plus fixed pre-approved wording and visible citation; otherwise refusal/escalation; an LLM never composes the display |

## 4. Chosen AI-Retrofit pattern

### Primary pattern: external sidecar

The selected architecture is a fail-open AI sidecar with an on-prem integration zone adjacent to MERLIN. Compute may use an approved elastic data plane, but the authority and adapter boundary remain on-prem. MERLIN sees only supported nightly snapshots, bounded override-table rows, and review-queue context.

Why it fits:

1. It provides independent scaling and release cadence without placing AI code inside an untested legacy engine.
2. It permits asynchronous batch work while MERLIN retains an independent schedule and fallback.
3. It maps exactly to the three declared extension points and does not invent a fourth.
4. It allows a shadow period using the production-like data path without any ordering effect.
5. It contains failures, costs, permissions, and model risk in a separately operable boundary.
6. It supports complete model/data/rules provenance while MERLIN remains the source of authoritative order IDs.

Accepted trade-off: the sidecar introduces snapshot synchronization and potential divergence from MERLIN. The design accepts this in exchange for isolation, controlling it with immutable snapshot IDs, checksums, current-run manifests, absolute policy caps, and a post-generation comparison only when a documented queue/reporting output supplies MERLIN's generated quantity. The sidecar never reads or recreates a current MERLIN baseline before publication.

### Competing-pattern disposition

| Candidate | Decision | Defensible boundary |
|---|---|---|
| Sidecar | **Select as primary architecture** | External services plus on-prem certified adapters; asynchronous and fail-open |
| Event-driven integration | Reject across MERLIN boundary | Legacy sources are nightly and MERLIN exposes no event stream. Internal task events, if used, do not create a new MERLIN integration pattern |
| API gateway | Reject | There is no documented real-time MERLIN API and no need for synchronous order-path calls. A gateway would couple availability to AI |
| Shadow-assist | Use as rollout mode | Same sidecar computes recommendations, but the adapter is disabled and MERLIN receives nothing |
| In-process plugin | Reject | Contradicts the unmodifiable engine, certified release process, absent tests, and failure-isolation requirement |

This is not an undocumented hybrid. The integration pattern is sidecar; the rollout state is shadow, assisted, bounded influence, or bypass. Optional events are internal implementation mechanics only.

## 5. Supported extension-point contract

| Extension point | Sidecar use | Fail-open rule | Required proof before influence |
|---|---|---|---|
| Nightly batch | Receive read-only snapshot/start signal | Missing or invalid snapshot stops AI only | Actual data-ready time, schema, completeness, permissions, and no delay to existing batch |
| Override table | Publish absolutely bounded current-run recommendations under the current epoch before hard close; no current MERLIN result is a pre-publication input | No valid epoch/permit/complete commit means no active AI rows; stale/uncommitted work is inert/rolled back | Existing supported staging/activation if available; otherwise direct P99 rate/overhead with cap `floor(R_cert x (L_cert - T_overhead_p99) x 0.80)`; same-transaction epoch guard; ownership; manual-row protection; locks; cleanup; engine read time |
| Review queue | Supply documented generated order ID/quantity/correlation keys after generation, then receive joined capacity-capped context | Missing fields/correlation/default semantics keeps automation disabled; excess never becomes a queue dependency | Supported read fields, correlation to exact publication keys, native actions/default, throughput/staffing, EDI interaction and audit references |

If any proof fails, the safe outcome is shadow-only. Direct table writes, engine patches, a new API, or schedule delays are not fallback options.

## 6. Explicit fail-open design

MERLIN's scheduler is independent. The sidecar can influence only by completing a narrowly timed publication. The absence of that publication is normal and requires no MERLIN error handler.

| Failure | Sidecar action | Authoritative outcome |
|---|---|---|
| Entire AI platform down | No adapter call; page operations | MERLIN min/max creates orders and existing EDI sends them |
| Late result | Hard close rejects it at 02:00; retain for diagnosis only | MERLIN continues; no same-day retry after cut-off |
| Low forecast confidence | Suppress row | MERLIN independently computes its incumbent result; the sidecar does not read it pre-publication |
| High inventory uncertainty | Apply absolute interval/case/DC/shelf-life bound or suppress; never guess shelf truth | MERLIN independently computes its incumbent result when no row is active |
| Stale/incomplete/schema-invalid input | Quarantine and fail the publish scope | MERLIN continues with its incumbent inputs |
| Model/ruleset not approved or provenance missing | Block publish | MERLIN continues |
| Optimizer violates a hard constraint | Suppress row and record reason | MERLIN independently computes its incumbent result |
| Stale epoch/expired permit/partial or locked adapter transaction | Atomically abort/roll back, revoke/terminate if required, open circuit, stop retry | Staged rows remain inert; no partial active set; MERLIN proceeds after lock release |
| Review capacity exhausted | Do not admit additional AI review cases; use legacy result for affected scope | Existing queue and EDI are not delayed by AI volume |

## 7. Kill-switch design

### State, authority, and fail-safe enforcement

- Durable latch record: signed `{epoch,state,run_id,scope_hash,manifest_hash,not_after}` in a quorum-backed on-prem store. States are `DISABLED -> ARMED_SHADOW -> ENABLED_PUBLISH -> DISABLING -> VERIFIED_SAFE -> ARMED_SHADOW`; each transition increments epoch. Unknown, unreachable, unsigned, expired, or non-current state denies.
- Credential model: the adapter has no standing writer. A local broker issues a single-run/scope/manifest permit valid <=60 seconds only in `ENABLED_PUBLISH`. The certified supported activation/direct transaction must evaluate the current epoch immediately before commit in the same transaction. If that is not possible without a new MERLIN hook/schema change, automation is blocked.
- Activation authority: Replenishment Operations and Model Risk approve enablement. Either Replenishment or SAP Duty Manager may immediately set a new `DISABLING` epoch. Global scope is default when uncertain.

### Execution and independent stop

1. A disable writes the new epoch; latch quorum, broker, DB/network gateway, orchestrator and session monitor acknowledge <=30 seconds. No new write can be accepted after <=60 seconds.
2. Missing acknowledgement invokes a separate SAP break-glass path that revokes permit issuance and adapter DB/network access and terminates active sessions. It does not depend on the adapter or main control service.
3. Epoch invalidation/session termination aborts and rolls back uncommitted direct work; supported staged rows remain inert because activation did not commit.
4. A separate certified cleanup identity/runbook reconciles the immutable pre-activation exact-key/hash ledger with MERLIN audit evidence and removes only unconsumed AI-owned rows. It never uses broad predicates or selects manual rows.
5. `VERIFIED_SAFE` <=5 minutes requires no valid permit, active writer/session/transaction, or unexplained active row, plus exact count/checksum/cleanup and acknowledgement audit IDs. Failure leaves access revoked/state `DISABLING`; MERLIN never waits.
6. Consumed rows use native pre-EDI correction only. There is no sidecar EDI, compensation, or post-04:00 rerun.

### Re-enable and test

Re-enable requires cause closure, clean evidence, a new current-data `ARMED_SHADOW` run, approved model/data/rules and dual approval for a new epoch; direct `VERIFIED_SAFE -> ENABLED_PUBLISH` is invalid. Before pilot and monthly, drills cover latch/control loss, stale epoch, check/write race, adapter compromise/failure, missing acknowledgement, independent revoke/session termination, and cleanup failure. Any failure blocks influence. Safe guard/identification/cleanup must use supported contracts without MERLIN schema/core change; otherwise permanent shadow-only.

## 8. Shadow-assist rollout

Shadow-assist is an operating state of the selected sidecar:

- use production-like snapshots and the full forecasting/optimization/validation path;
- compare AI recommendations with MERLIN outputs only after a documented supported queue/reporting export supplies generated quantity/order ID; without it, retain the recommendation but claim no current-run comparison;
- write evidence to the sidecar audit/monitoring systems only;
- keep the adapter policy disabled and publish zero override-table rows;
- show category managers comparison reports only from that documented post-generation join;
- measure WAPE, promotion/peak slices, absolute quantity/value distributions and, when available post-generation, deltas; also inventory sensitivity, completion, cost, exception volume and reviewer capacity;
- rehearse bypass, kill switch, late-output, stale-data, partial-adapter, and 03:30 recovery scenarios.

Minimum proposed duration is four consecutive weeks with representative promotions. Production influence additionally requires quantitative NFR approval, successful 6x testing, validated MERLIN timing reserve, safe adapter contract, approved queue semantics, and named operational owners.

## 9. Human review model

| Risk tier | Treatment | Reason |
|---|---|---|
| Prohibited/high | Safety-sensitive, fresh, regulated-price, excessive absolute quantity/value, invalid, low-confidence, high-uncertainty, or out-of-policy candidate is not published | Human review must not legitimize an unsafe or untraceable recommendation |
| Medium | Publish only if the native queue has a tested non-blocking and safe fallback; otherwise suppress | Unknown queue semantics cannot be solved by inventing a new approval system |
| Low | Eligible for bounded pilot influence after shadow gates; annotate a capacity-capped subset for review/learning | Keeps review volume finite while preserving measurable human oversight |

Initial scope is 20 stores, 2 low-risk non-fresh categories, and no more than 100 AI review exceptions/night. Those figures are conservative rollout assumptions, not enterprise capacity claims. After OQ-13 is measured, the cap is set to the smaller of available reviewer capacity and risk-ranked value. Items beyond the cap use MERLIN's incumbent result and never accumulate.

Category managers receive MERLIN generated quantity/order ID only through the certified post-generation queue contract, plus proposed quantity, post-generation delta/value, prediction interval, uncertainty, source age, constraints/reasons, versions, and expiry. If the contract cannot provide/correlate these fields, influence remains disabled. Outcomes link to the order and exact sidecar publication key.

## 10. Rollback plan

Rollback does not restore code inside MERLIN because none was changed.

1. Activate the kill switch and block all new publications.
2. Delete only unconsumed AI-owned rows using the publication ledger and certified procedure; preserve manual overrides.
3. Cancel review annotations that the supported queue contract permits, without delaying incumbent queue processing.
4. Keep forecasting and optimization in shadow for diagnosis or stop them if cost/security requires.
5. MERLIN continues with its existing min/max parameters, schedule, review process, EDI connector, and supplier contracts.
6. Reconcile affected runs and orders, assess business impact, and retain immutable evidence.
7. Re-enter only through shadow mode after the re-enable gates pass.

No rollback step modifies POS, changes EDI, moves 04:00, calls for a post-cut-off re-run, or creates a second order authority.

## 11. Migration phases without replacing MERLIN

| Phase | Change | MERLIN effect | Exit gate |
|---|---|---|---|
| 0. Contract and timing discovery | Read-only inspection, test harness, table/queue/schedule measurement, owner assignment | None | Certified post-generation order fields/correlation; same-transaction epoch guard; staged activation or measured direct cap; P99 reserve; independent stop and safe cleanup proof |
| 1. Historical replay | External data copies, feature/forecast/optimizer evaluation, 6x load and failure tests | None | Quantitative accuracy, timing, cost, safety and traceability acceptance |
| 2. Production shadow | Current nightly recommendation/monitoring, zero table writes; compare only through documented post-generation output | None | At least 4 weeks; promotion slices; stable operations; kill-switch exercise |
| 3. Human-reviewed pilot | 20 stores, 2 low-risk categories, <=100 review exceptions/night | Bounded supported overrides only if queue fallback is proven | No deadline regression; approved quality, review load and rollback evidence |
| 4. Bounded influence | Expand by explicit store/category/configuration, retain absolute quantity/value, confidence, uncertainty and measured transaction caps | MERLIN still generates and sends every order | Repeated NFR, cost, business and operational gates |
| 5. Controlled scale | Scale sidecar partitions and support; do not change authority or interfaces | Same three extension points and legacy fallback | Continuous monitoring and periodic fallback exercises |

This is adoption of an external advisory capability, not gradual replacement. MERLIN remains fully functional at every phase with the sidecar removed.

## 12. Model, decision, and operational traceability

Publication is blocked unless the audit manifest binds:

- dated source snapshots, schema/checksums, freshness and residency classification;
- feature version and explicit inventory-uncertainty band;
- approved forecasting model/version, training approval, quantiles and confidence;
- deterministic optimizer/ruleset version and every hard-constraint result;
- AI recommendation, absolute cap inputs/results, risk tier and suppression/publish reason; no pre-publication MERLIN quantity;
- shadow/influence state, hard-gate time, epoch/permit, adapter transaction and exact AI-owned keys;
- latch acknowledgements, credential/session revocation, independent cleanup and verified-safe evidence;
- post-generation MERLIN quantity/order ID, review actor/outcome and EDI reference only when supplied by a certified supported output.

Monitoring covers deadline consumption, partition completion, forecast quality/drift, data freshness/completeness, inventory-quality bands, suppressed-row rate, adapter locks/rollback, queue backlog/expiry, kill-switch state, audit completeness, and AI run cost. Numeric thresholds and owners are finalized by the NFR validator rather than invented here.

## 13. Open feasibility gates and assumptions

| Baseline item | Architectural treatment |
|---|---|
| OQ-01 capability priority | Commit forecasting and replenishment, include promotion batch output and a read-only copilot pilot, defer production markdown and substitution |
| OQ-02 7.04M vs 26.88M | Validate capacity and cost at 27M; optimize only the approved stocked population; report both |
| OQ-03 6x denominator | Load-test data/feature/batch path conservatively at 6x and keep the same wall; validator must make arithmetic explicit |
| OQ-05 budget basis | Treat $3.2M as binding delivery ceiling without assuming accounting split; finance confirmation required |
| OQ-06 team/support | No Variant A staffing is imported. Named Variant B build/on-call owners are a release gate |
| OQ-07 residency | Exclude member-level loyalty data. Any future personal feature stays in a country-local pipeline after legal approval |
| OQ-08/OQ-09 safety authority | Copilot safety responses remain pilot-only and refuse conflicts until source precedence and accountable owner are approved |
| OQ-12 extension contracts | Production influence is blocked until the post-generation queue fields/correlation, same-transaction epoch guard, permit/revocation path, staging/direct-rate arithmetic, locks, timing, cleanup and support are certified |
| OQ-13 review capacity | Start with <=100/night pilot cap, measure actual capacity, and suppress excess |
| OQ-15 batch reserve | 02:00 hard close is provisional. MERLIN P99 plus 30 minutes must fit; otherwise move earlier or remain shadow |
| A-05 owners | Workshop artifact and operational owners are assigned from the supplied Group 4 roster |
