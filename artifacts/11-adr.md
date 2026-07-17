# Artifact 4 - ADR-001: Fail-Open AI Sidecar Through MERLIN's Supported Override and Review Extensions

Owner: PHẠM THỊ THANH HUYỀN (164955, PH2)

- Status: Proposed and accepted for historical replay/shadow only
- Date: 2026-07-17
- Production decision: Rejected until every release condition below is evidenced
- Scope: Demand forecasting and replenishment recommendations for NovaMart Variant B

## Context

NovaMart needs forecast-backed replenishment without replacing or modifying MERLIN. MERLIN is the authoritative supplier-order system, its 2006 min/max engine cannot change, POS is untouchable, supplier EDI and the 04:00 cut-off are fixed, inventory is only 78% accurate, and AI failure must never prevent morning orders.

MERLIN exposes exactly three supported extension points: a nightly batch, an override table read before order generation, and a category-manager review queue after generation. MERLIN has no automated regression suite and releases only twice yearly through a certified partner. The source also requires shadow operation before AI recommendations affect orders.

Capacity planning uses 27M forecasts/night, a conservative 162M forecast-equivalent 6x test, 7.04M stocked order candidates, and a 42.24M optimization-equivalent 6x test. The proposed sidecar closes at 02:00 to reserve two hours for MERLIN, but actual MERLIN P99 and adapter behavior are not yet measured.

## Decision drivers

1. MERLIN must remain authoritative and independently able to order when every AI component is absent.
2. Only the three supported extension points may cross the MERLIN boundary.
3. The sidecar must never touch POS, transmit supplier EDI, move 04:00, or create a second order authority.
4. Late, partial, low-confidence, stale, residency-invalid, or untraceable AI output must safely become no override.
5. Forecasting must use statistical ML; replenishment must use deterministic reproducible constraints; no LLM may produce numeric forecasts/order quantities or compose displayed allergen/food-safety content.
6. The architecture must scale to the conservative 6x test without placing 6x capacity or new code inside MERLIN.
7. Rollout must begin in shadow and support a locally executable kill switch and clean rollback.
8. Human review must be risk-ranked and capacity-capped rather than an unbounded prerequisite.

## Options considered

| Option | Disposition | Reason |
|---|---|---|
| External fail-open sidecar | **Selected** | Best fit for supported nightly interfaces, independent scale/release, failure isolation, shadow operation, and exact rollback controls; weighted score 4.80/5.00. |
| Event-driven integration across MERLIN boundary | Rejected | MERLIN and the source estate are nightly, not event-streaming; a new event contract would invent a fourth integration point. Internal sidecar events remain acceptable implementation mechanics. |
| API gateway / synchronous AI call | Rejected | No documented real-time replenishment API exists, and a synchronous dependency would put AI availability on the ordering path. |
| Shadow-assist as the architecture | Rejected as incomplete; retained as rollout mode | Shadow describes whether recommendations influence orders, not where the system runs or how it integrates. |
| In-process MERLIN plugin | Rejected | Conflicts with the unmodifiable engine, absent tests, certified release cadence, and required failure isolation. |
| Replace, fork, or gradually strangle MERLIN | Prohibited | The workshop scenario explicitly rules out core replacement and dual supplier-order authority. |

## Decision

Build a separately deployed, asynchronous AI sidecar with an on-premises integration/control zone adjacent to MERLIN and an approved elastic batch data plane.

The sidecar reads immutable snapshots through the supported nightly-batch extension, builds uncertainty-aware features, creates statistical demand forecasts, applies deterministic replenishment constraints, and stages a complete current-run recommendation set. It may influence MERLIN only through:

1. a certified override-table adapter that atomically activates absolutely bounded current-run rows under a same-transaction monotonic epoch guard before the hard close; and
2. a certified review-queue adapter that receives documented post-generation generated order ID/quantity/correlation keys, joins them to the exact publication ledger, and then attaches capacity-capped provenance.

MERLIN remains the only order generator, system of record, review authority, and supplier-EDI sender. Its schedule never waits for the sidecar. The sidecar does not receive/recreate a current MERLIN result before publication; it uses non-negative whole-case and absolute shelf-life/inventory/DC/category caps (initial pilot <=2 case packs/store-SKU). No override is the normal fail-open state. Relative comparison occurs only after a certified supported queue/reporting output supplies the generated quantity; absent that output, automated influence is blocked.

Shadow-assist is mandatory before influence: the full production-like calculation runs, but the override adapter is disabled and zero rows are written. A current-run MERLIN comparison is claimed only from a documented post-generation output. The initial influence scope, only after all gates pass, is 20 stores, two low-risk non-fresh categories, `min(5,000, measured transaction cap)` rows/night, and <=100 review exceptions/night.

The sidecar stops new retries at 01:30, requires a complete signed manifest by 01:45, and closes all writes at 02:00. 70,400 rows in a <=60 s direct lock needs >=1,173.4 rows/s before overhead; the cap is `floor(R_cert x (L_cert - T_overhead_p99) x0.80)` and is allowed only after measurement. Existing supported non-live staging plus <=5 s atomic activation may be used if certified; it is not assumed.

The kill control is a durable deny-default latch `{epoch,state,run,scope_hash,manifest_hash,not_after}` with monotonic epochs, <=60 s single-run permits, no standing writer, and a same-transaction epoch check. `DISABLING` requires component acknowledgements <=30 s, new-write rejection <=60 s, independent SAP credential/DB-network revocation and active-session termination, separate exact-row cleanup, and `VERIFIED_SAFE` <=5 min. Unsupported transaction guard or independent stop means shadow-only.

## Detailed consequences

### Positive consequences

- MERLIN can complete min/max replenishment and fixed-schema EDI with the entire AI platform unavailable, late, or disabled.
- AI code, models, scaling, releases, and incidents are isolated from the untested MERLIN engine and the POS revenue path.
- Shadow mode exercises the real data and computation path without changing an order.
- Statistical forecasting, deterministic optimization, deterministic safety rendering, and optional non-safety language processing remain separable and governable.
- Elastic, partitioned execution can be tested against 162M forecast and 42.24M optimization equivalents without permanent 6x on-prem capacity.
- Complete-run manifests, exact adapter transactions, model/rules/data versions, and MERLIN order references create end-to-end traceability.
- The kill switch and rollback remove future influence without restoring MERLIN code because no MERLIN code is changed.

### Negative consequences and accepted trade-offs

- The sidecar duplicates/stages legacy data, so snapshot drift and reconciliation are new failure modes. The design must operate immutable snapshot IDs, checksums, freshness rules, and a complete-set gate.
- The override adapter is a sensitive integration. Atomic ownership, locks, expiry, cleanup, engine-read timing, and protection of manual rows require SAP-certified discovery and testing.
- A two-system trace is operationally more complex than a single MERLIN job. NovaMart must fund monitoring, WORM-capable audit, on-call coverage, and recurring drills.
- The 02:00 close reduces the usable AI window to four hours. This deliberately favors incumbent deadline reserve over maximizing model runtime.
- Bounded override and review caps leave some forecast value unrealized. Excess, uncertain, or unreviewable cases fall back to MERLIN rather than accumulating.
- The architecture does not repair MERLIN's 340,000 parameters or 78% inventory accuracy. It contains these limitations using intervals, absolute caps, suppression, and only certified post-generation comparison.
- Annual operating cost is planned at $3.8M, which is 3.86% of current net profit. The provider rates, support roster, and Finance treatment are not yet signed.

## Risks created or retained by the decision

- Sidecar or MERLIN timing may consume the 04:00 reserve; production influence is blocked until observed P99 evidence passes.
- Adapter locking, partial transactions, or ownership ambiguity could impair MERLIN or delete manual intent; inability to prove safe behavior forces permanent shadow mode.
- Inaccurate inventory can still lead to bad recommendations; every published row needs a calibrated interval and feasibility across that interval.
- Peak/promotional behavior can degrade throughput or forecast quality; three consecutive 6x tests and slice-level WAPE gates are mandatory.
- Category-manager capacity may be lower than assumed; review admission remains capped and excess uses MERLIN.
- Named Variant B operators, safety/accountability owners, and signed cost/residency rules are absent; influence remains disabled while they are absent.

## Failure behavior

| Trigger | Sidecar result | MERLIN result |
|---|---|---|
| AI unavailable | No publish attempt; alert | Min/max and EDI continue |
| Required output incomplete at 01:45 | Discard current publish scope | Incumbent quantity is used |
| Clock reaches 02:00, including recovery at 03:30 | Adapter remains closed; no retry or rerun | Current authoritative run continues |
| Low confidence or high/uncalibrated inventory uncertainty | Suppress row | MERLIN independently computes its incumbent result; sidecar never reads it pre-publication |
| Stale, incomplete, schema-invalid, or residency-invalid input | Quarantine/bypass affected scope | Incumbent process continues |
| Missing audit/model/rules provenance | Block publication | Incumbent process continues |
| Adapter epoch stale/permit expired/transaction fails or locks | Atomic abort/rollback; staged rows inert; independent credential/session revoke if needed | Continues after certified transaction releases |
| Review capacity exhausted | Do not admit more AI cases | Existing queue/EDI continue; affected quantity uses the safe incumbent behavior |

## Rollback strategy

1. Either authorized Duty Manager writes a new monotonic `DISABLING` epoch to the durable quorum latch; unknown/unreachable/expired state already denies.
2. Latch, broker, DB/network gateway, orchestrator, and session monitor acknowledge <=30 s. No new write is accepted after <=60 s; missing ack invokes independent SAP break-glass revoke and session termination.
3. The current-epoch check inside the activation/direct transaction causes stale work to abort/roll back; staged rows remain inert.
4. A separate cleanup identity/runbook uses the immutable exact-key/hash ledger and MERLIN audit evidence to remove only unconsumed AI rows; manual rows are preserved.
5. `VERIFIED_SAFE` <=5 min requires no permit/session/transaction/unexplained row plus acknowledgement and checksum evidence. Consumed rows use native pre-EDI correction only; no sidecar compensation/rerun exists.
6. MERLIN continues unchanged. Re-entry requires clean evidence, a successful `ARMED_SHADOW` run, reapproval and a new epoch; direct enable is invalid.

If ownership-safe cleanup cannot be supported without a schema or engine change, the rollback decision is simple: never enter influence mode; remain shadow-only.

## Production-influence conditions

All conditions are mandatory:

1. Measured MERLIN P99 generation/review/EDI is <=90 minutes for the 02:00 close, leaving >=30 minutes, or the close moves earlier and all capacity tests are rerun.
2. Three consecutive production-equivalent 6x nights meet the 30,000 forecast/s and 15,000 optimization/s targets, all stage gates, P99 <=225 minutes, and cost limits.
3. The certified contracts prove: no pre-publication current MERLIN result; post-generation order ID/quantity/correlation; exact ownership/manual protection; same-transaction epoch guard; supported staged <=5 s activation or measured direct cap/rate/overhead (`70,400/60>=1,173.4/s` raw); <=60 s lock; atomic abort/rollback; safe cleanup/read timing.
4. Drills prove latch-loss deny, stale-epoch/check-write abort, acknowledgements <=30 s, rejection <=60 s, independent credential/session revoke, separate cleanup and safe state <=5 min without manual-row deletion or MERLIN delay.
5. Data completeness/residency, inventory interval coverage >=90%, enterprise WAPE <=25%, promotion WAPE <=35%, and 100% audit completeness pass representative slices.
6. Native review generated order ID/quantity/correlation, capacity and timeout/default behavior are certified non-blocking; missing fields keep influence disabled and the 100/night cap remains.
7. Finance separately signs the $3.2M delivery basis and <=$3.8M annual operating target with all support costs included.
8. Named and funded primary/backup operational owners exist for nightly support, SAP integration, models/rules, replenishment, security, data protection, and safety.

## Conditions under which this ADR must be revisited

- MERLIN adds or removes a supported extension point, changes override/review semantics, or upgrades the engine/order schedule.
- Observed MERLIN P99 cannot leave a 30-minute reserve even after moving the sidecar close earlier.
- The adapter cannot meet post-generation correlation, epoch enforcement, ownership, rate/activation, lock, cleanup, or manual-row protection without changing MERLIN.
- Forecast/optimizer 6x tests, cost projection, WAPE, or inventory-interval calibration repeatedly miss their gates.
- Review demand cannot fit safe human capacity without making AI a blocking approval dependency.
- NovaMart proposes member-level loyalty features, a new country boundary, direct pricing/substitution activation, or any POS/EDI integration.
- A material incident shows that fail-open, kill-switch, audit, or rollback behavior is ineffective.
- A future supported MERLIN capability offers equal or better isolation and observability with less operational complexity. Revisiting does not imply replacing MERLIN; authority remains a separate hard constraint.

## Decision outcome

This ADR accepts the operational complexity and bounded value of a sidecar in exchange for isolating AI failure from a fragile, authoritative legacy order path. It authorizes replay and shadow construction only. It does **not** authorize a production override until all eight conditions pass.
