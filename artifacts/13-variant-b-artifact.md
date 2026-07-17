# Artifact 6 - Variant B Legacy Assessment and Retrofit Plan

Owner: TRẦN TRỌNG PHÚ (197320, PH2)

Status: **Approved for historical replay and shadow mode only. Production replenishment influence and definitive safety/allergen answers remain rejected until the conditional release gates pass.**

## 1. Legacy assessment

MERLIN is a customized on-premises SAP ERP in operation since 2006. It owns merchandising, inventory records, replenishment execution, supplier orders, and fixed-schema EDI. Its min/max engine has 340,000 hand-maintained parameters, no automated test suite, and a twice-yearly SAP-certified release cycle. A previous $22M replacement failed. MERLIN is therefore a permanent authority in this design, not a migration source.

The safe change boundary is outside the core. MERLIN has exactly three supported extension points: (1) an override table read before order generation, (2) a category-manager review queue after generation, and (3) the existing nightly batch. POS is an offline-first 2011 estate whose modification requires nine-month recertification; it is untouchable. Supplier EDI schema and the 04:00 cut-off are fixed. Inventory is 78% accurate and nightly warehouse data may be 24 hours stale.

## 2. What can change

| Changeable surface | Allowed change |
|---|---|
| External sidecar | Add immutable ingestion/validation, uncertainty-aware features, statistical forecast ML, deterministic optimization, recommendation staging, model/rules registry, monitoring, and append-only audit. |
| Nightly-batch extension | Add a supported read-only dated snapshot and start signal without delaying or changing the incumbent batch. |
| Override-table extension | Through a certified adapter, activate exact AI-owned current-run rows under a same-transaction epoch; use supported staging/constant activation or a measured direct cap. |
| Review-queue extension | Receive documented post-generation generated order ID/quantity/correlation, join to exact publication keys, and attach measured capacity-capped context. |
| Sidecar operations | Add durable deny-default latch, monotonic epochs, short-lived permits, independent credential/session revocation, separate cleanup, scope caps, alerts/runbooks/replay. |
| Separate associate tool | Pilot deterministic exact safety display/citation and optional non-safety language assistance with signed offline packs, refusal, and escalation on non-POS devices. |
| Approved non-personal data plane | Process aggregated store-SKU snapshots and approved external features in an approved region. Member-level loyalty data is excluded from Release 1. |

Offline copilot acceptance is not availability-by-refusal: signed-pack coverage must be 100% and p95 <=500 ms; >=90% of eligible non-safety benchmark questions must return correct cited answers with >=95% answer/citation accuracy; ineligible non-safety refusal precision/recall must each be >=95%; safety policy, eligible exact display/citation, and required-refusal precision/recall must each be 100%. Universal refusal fails.

## 3. What cannot change

| Fixed surface | Non-negotiable boundary |
|---|---|
| MERLIN authority | MERLIN remains the only supplier-order system of record, order generator, native review authority, and EDI sender. |
| MERLIN engine | No code, algorithm, embedded model, hook, call sequence, or wait/dependency change. |
| Extension inventory | No fourth MERLIN integration point. A component must use nightly batch, override table, review queue, or remain entirely external. |
| POS | No AI code, agent, plugin, schema, route, credential, device dependency, timing change, or recertification impact. |
| Supplier EDI | No schema, endpoint, sender authority, timing, or 04:00 change. |
| Incumbent continuity | MERLIN must order and stores must receive morning trucks when the entire AI platform is unavailable. |
| Core replacement | No gradual replacement, strangler, parallel order master, or migration of supplier-order authority. |
| Legal/safety constraints | No cross-border member loyalty data, uncited food-safety answer, arbitrary pricing action, or reduced POS retention. |

## 4. What is unsafe to touch

| Unsafe area | Why | Safe treatment |
|---|---|---|
| Internal min/max engine or broad parameter rewrite | No regression suite and high operational blast radius | Compare in shadow; apply only bounded external overrides after all gates pass |
| Undocumented MERLIN tables/manual override rows | Locking, corruption, and ownership risk | Certified adapter plus exact-key/hash ledger; never broad-delete or select manual rows |
| MERLIN scheduler/critical path | Any wait can miss 04:00 | Independent sidecar; no synchronous callback; 02:00 hard close or earlier |
| POS runtime/database/network/sync | Revenue-path and certification risk | Use existing nightly aggregate outputs only; no POS route |
| Supplier EDI connector | Fixed 2,300-supplier contract | MERLIN alone sends the unchanged order schema |
| Point inventory as shelf truth | 22% measured inaccuracy | Calibrated intervals, data age, bounds, suppression, and outcome slices |
| Unbounded review queue | Human bottleneck and normalization of unsafe recommendations | Risk-rank, cap, expire safely, and use MERLIN when capacity is exhausted |
| Regional member-level loyalty pipeline | One market prohibits cross-border movement | Exclude from Release 1; require a future country-local lawful design |
| Generative safety answer | Fatal-harm and criminal-liability risk | Exact approved structured field/passage + fixed approved locale template/citation; otherwise refusal/escalation; no LLM-composed safety display |

## 5. Selected AI-Retrofit integration pattern

The primary pattern is an **external, asynchronous, fail-open sidecar**. An on-premises integration/control zone sits adjacent to MERLIN; an approved elastic data plane may run the partitioned batch workload. The sidecar reads only supported immutable snapshots and writes only through narrow certified adapters.

**Shadow-assist is a rollout state, not the integration architecture.** Internal queues/events may coordinate sidecar tasks, but no event-driven dependency crosses the MERLIN boundary.

## 6. Why this pattern was selected

- It maps to exactly the three supported extension points and needs no MERLIN/POS/EDI modification.
- MERLIN never waits for AI; absence of an override is a normal fail-open condition.
- AI models, compute, cost, and releases scale independently of the fragile core.
- The full path can run in shadow with zero table writes.
- Exact data/model/rules/decision provenance can be recorded outside MERLIN and linked to authoritative order IDs.
- It scored 4.80/5.00 in the weighted pattern matrix, ahead of shadow-as-architecture (4.00), event-driven (2.65), API gateway (2.15), and in-process plugin (1.65).

Accepted trade-off: duplicated snapshots and possible divergence from MERLIN. The sidecar does not read/recreate a current MERLIN result pre-publication; it uses absolute caps. Immutable IDs/checksums/manifests and a certified post-generation queue join control divergence. Without documented generated quantity/order correlation, no current-run comparison is claimed and automation stays disabled.

## 7. Kill-switch design

**State/control:** A durable quorum-backed on-prem latch stores signed `{epoch,state,run_id,scope_hash,manifest_hash,not_after}`. States are `DISABLED -> ARMED_SHADOW -> ENABLED_PUBLISH -> DISABLING -> VERIFIED_SAFE -> ARMED_SHADOW`; every transition increments epoch. Unknown/unreachable/unsigned/expired/non-current state denies.

**Transaction enforcement:** The adapter has no standing write credential. A local broker issues a single-run/scope/manifest permit valid <=60 s only in `ENABLED_PUBLISH`. The certified activation/direct-write transaction checks current epoch immediately before commit in the same transaction; stale/expired state atomically aborts. Unsupported guard means shadow-only.

**Execution:**

1. Either Duty Manager writes a new global `DISABLING` epoch when scope is uncertain.
2. Latch quorum, broker, DB/network gateway, orchestrator and session monitor acknowledge <=30 s. No new write is accepted after <=60 s.
3. Missing acknowledgement invokes a separate SAP break-glass identity to revoke issuance/DB-network access and terminate active sessions, independent of the adapter/control service. Uncommitted work rolls back; staged rows stay inert.
4. A separate cleanup identity/runbook reconciles immutable pre-activation exact keys/hashes with MERLIN audit evidence and removes only unconsumed AI rows; manual rows are never selected.
5. `VERIFIED_SAFE` <=5 min requires no permits, sessions, transactions or unexplained active rows plus acknowledgement/checksum/cleanup audit evidence. Failure leaves `DISABLING` and access revoked while MERLIN runs.
6. Consumed rows use native pre-EDI correction only; no compensating EDI or post-04:00 rerun exists.

Re-enable requires clean evidence, a current-data `ARMED_SHADOW` run, model/data/rules approval and dual approval for a new epoch; direct enable is invalid. Pre-pilot/monthly drills cover latch loss, stale epoch, check/write race, adapter compromise/failure, missing ack, independent revoke/session termination and cleanup failure. Any failure, or unsupported exact cleanup without schema/core change, means permanent shadow-only.

## 8. Fail-open behavior

| Trigger | Sidecar action | Authoritative outcome |
|---|---|---|
| AI platform unavailable | No adapter call; page operations | MERLIN min/max and fixed-schema EDI continue |
| Required run incomplete at 01:45 | Discard the publish scope | MERLIN independently computes its incumbent result; sidecar never reads it pre-publication |
| Result late or service recovers at 03:30 | 02:00 hard close rejects it; no retry/rerun | MERLIN's current run continues |
| Low confidence/high or uncalibrated inventory uncertainty | Suppress row | MERLIN independently computes its result |
| Input >24 h stale, <99.5% complete, schema/checksum/residency invalid | Quarantine and bypass affected scope | MERLIN continues with incumbent inputs/rules |
| Optimizer hard-constraint violation | Reject and audit row | MERLIN independently computes its result |
| Missing model/rules/data/audit provenance | Block publication | MERLIN continues |
| Stale epoch/expired permit/adapter commit or lock failure | Atomic abort/rollback; staged rows inert; independent session revoke if needed | MERLIN continues after the certified transaction releases |
| Review capacity exhausted | Admit no additional AI case | Existing review/EDI continue; affected scope uses incumbent behavior |

MERLIN's scheduler never waits for any sidecar state or response. A recovered AI platform at 03:30 is too late by design.

## 9. Shadow-mode rollout

| Phase | Activity | MERLIN effect | Exit gate |
|---|---|---|---|
| 0 - Contract/timing discovery | Measure MERLIN P99, post-generation order fields/correlation, same-transaction epoch, staging/direct rate/overhead, locks/cleanup, review default, and support | None | Certified contract, independent stop and safe timing/cleanup proof |
| 1 - Historical replay | Rebuild past runs; validate WAPE, uncertainty intervals, deterministic constraints, 6x throughput, cost, and trace | None | Quantitative accuracy/capacity/cost/safety acceptance |
| 2 - Production shadow | Run >=4 weeks; zero rows; compare only after documented post-generation output | None | Stable operations; representative slices; all fail-safe drills pass |
| 3 - Human-reviewed pilot | 20 stores, two low-risk categories, `min(5,000, measured transaction cap)` rows and <=100 reviews/night | Bounded supported overrides after every gate | No deadline regression; queue join, quality, rollback/cost pass |
| 4 - Bounded influence | Expand with absolute quantity/value, confidence/uncertainty and measured transaction caps | MERLIN still generates/sends every order | Repeated NFR/business/operational gates |
| 5 - Controlled scale | Scale partitions and support, retaining the same three interfaces and authority | No core migration | Continuous gates and recurring fallback drills |

Production influence requires MERLIN P99 <=90 min (or earlier close/retest), three 6x nights, certified post-generation queue join, no pre-publication baseline, same-transaction epoch, supported staged <=5 s activation or direct cap formula (`70,400/60>=1,173.4/s` raw), <=30 s ack/<=60 s rejection/<=5 min safe-state plus independent revoke, calibrated intervals, WAPE/data/residency/audit, measured review, signed cost, and named owners.

## 10. Human review model

| Risk tier | Treatment |
|---|---|
| Prohibited/high | Fresh, safety-sensitive, regulated-price, excessive absolute quantity/value, low-confidence, high-uncertainty, invalid, or untraceable candidate is suppressed pre-publication. An extreme delta is assessed only after certified post-generation join. |
| Medium | Influence only if the native queue's timeout/default is proven non-blocking and preserves a safe incumbent outcome; otherwise suppress. |
| Low | Eligible for bounded pilot influence after shadow gates; a capacity-capped subset is annotated for review and learning. |

Admission is `min(risk-ranked demand, measured reviewer capacity)` with interim 100/night. Excess never accumulates; MERLIN independently uses its result. Managers see generated quantity/order ID only from the certified post-generation queue contract, plus AI quantity, computed delta/value, interval, uncertainty, age, reasons, versions and expiry. Missing fields/correlation keeps influence disabled.

## 11. Rollback plan

1. Activate the kill switch and block all new publications.
2. Remove only exact unconsumed AI-owned rows through the certified transaction; preserve manual overrides.
3. Remove supported review annotations only when doing so cannot delay the incumbent queue.
4. Keep the sidecar in shadow for diagnosis or stop it if security/cost requires.
5. Continue MERLIN's min/max parameters, schedule, native review, fixed EDI, and supplier contracts unchanged.
6. Reconcile affected runs/orders and retain immutable evidence.
7. Re-enter only through a new shadow run and all re-enable approvals.

Rollback changes no MERLIN or POS code, creates no second order authority, moves no cut-off, and performs no post-04:00 rerun.

## 12. Migration without replacing MERLIN

This is external capability adoption, not core migration. Removing the sidecar leaves MERLIN functional. Expansion changes configuration/certified measured capacity, never authority, min/max, hooks, EDI or POS. Forecast ML, deterministic optimization, deterministic safety renderer and optional non-safety language service can be independently disabled. Production markdown/substitution stay deferred.

## Facts, assumptions, and conditional gates

**Facts:** 640 stores; about 11,000 SKUs/store; 27M forecast requirement; 6x peak; 22:00-04:00 wall; 04:00 EDI cut-off; 78% inventory accuracy; exactly three MERLIN extension points; MERLIN/POS/EDI constraints; $3.2M delivery and $4M/year AI run-cost controls as separately stated.

**Planning assumptions:** 162M/42.24M equivalents; 3,840 partitions; 02:00 close; 70,400 is only a policy ceiling and actual cap=`min(ceiling,floor(R_cert x (L_cert-T_overhead_p99)x0.80))`; 20-store/two-category pilot; 5,000 is only a pilot ceiling; 100 reviews; $3.8M run target. Raw direct rate is >=1,173.4/s for 70,400 in 60 s; these are targets, not observations.

**Unresolved release gates:** MERLIN P99; post-generation fields/correlation; epoch guard/staging or direct rate/lock/cleanup/independent stop; 6x/bills; review capacity/defaults; deterministic safety hierarchy and eligible/refusal benchmark; residency; Finance basis; named owners. Until all pass, remain shadow-only and safety refusal-only.
