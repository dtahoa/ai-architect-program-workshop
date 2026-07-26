# Artifact 6 - Variant B Legacy Assessment and Retrofit Plan

Owner: **TRẦN TRỌNG PHÚ**

Status: Replay and shadow only until every applicable production gate passes.

## 1. Legacy estate and authority

MERLIN is the customized SAP system of record for merchandising, replenishment, supplier orders, native review, and fixed-schema EDI. Its untested min/max engine remains authoritative. POS is offline-first and untouchable. Inventory is 78% accurate and nightly data may be 24 hours stale.

## 2. Supported extension points

Only three MERLIN crossings exist: a nightly read-only snapshot/start signal, an override table read before order generation, and a review queue after generation. No fourth interface is invented.

## 3. What can change

External snapshot validation, feature pipelines, statistical forecast ML, deterministic optimization, narrow certified adapters, monitoring/audit, disable controls, and a separate read-only associate-copilot pilot may be added outside MERLIN/POS.

## 4. What cannot change

MERLIN authority/core/schedule, POS runtime or interfaces, supplier EDI schema/sender/cut-off, the 04:00 deadline, and morning-order continuity cannot change. There is no gradual MERLIN replacement or parallel order master.

## 5. What is unsafe to touch

Undocumented MERLIN tables, manual rows, internal scheduler/hooks, POS network/database/sync, the EDI connector, member-level cross-border loyalty data, point inventory as shelf truth, unbounded review, and generative safety display are prohibited.

## 6. Selected integration pattern

Use an external, asynchronous, fail-open sidecar adjacent to MERLIN. Internal queues may coordinate sidecar work, but no synchronous or event-driven dependency crosses the legacy boundary. Shadow-assist is the initial rollout state.

## 7. Rationale and accepted trade-off

The sidecar maps to supported extensions, isolates failure and releases, scales independently, and can be removed without MERLIN rollback. It accepts duplicated snapshots, reconciliation, SAP-certified adapter work, two-system traceability, operating cost, and bounded value.

## 8. Fail-open behavior

Unavailable, late, partial, stale, invalid, uncertain, untraceable, or disabled AI produces no override. Stop retries at 01:30, require completeness at 01:45, close writes at 02:00, and reject 03:30 recovery. MERLIN independently generates, reviews, and sends orders.

## 9. Plain and testable kill switch

| Question | Testable answer |
|---|---|
| Who can activate it? | Either named Replenishment Duty Manager; SAP break-glass operator can independently revoke access |
| Where? | Local deny-default control plus independent SAP DB/network access control; neither depends on the cloud sidecar |
| What happens? | Write a new `DISABLING` epoch, stop short-permit issuance, recheck epoch inside active transactions, abort/rollback stale work, and reject new writes |
| How fast? | Acknowledge <=30 s; no new write <=60 s; `VERIFIED_SAFE` <=5 min |
| What about active access? | Independent SAP identity revokes DB/network access and terminates sessions if normal acknowledgement fails |
| What is cleaned? | Separate identity removes only exact unconsumed AI-owned keys/hashes; manual rows are never selected |
| What is audited? | Actor, reason, epoch, scope, permits, sessions, abort/rollback, exact cleanup, checksums, timestamps, and safe-state evidence |
| What does MERLIN do? | Continues its incumbent min/max, review, and EDI path without waiting |

Drills cover control loss, stale epoch, check/write race, adapter compromise, missing acknowledgement, out-of-band revoke, session termination, and cleanup failure. Any failure keeps influence disabled. The adapter has no standing writer credential.

## 10. Shadow-assist rollout

Discovery certifies timings/contracts. Historical replay tests quality, intervals, 6x scale, and cost. Production shadow runs at least four representative weeks with zero MERLIN writes. Only then may a bounded pilot begin.

## 11. Human-review model

Prohibited/high-risk candidates are suppressed before publication. The pilot admits at most 100 risk-ranked exceptions/night. Post-generation order ID/quantity/correlation may return only through the certified native queue. Missing fields, unsafe defaults, or exhausted capacity mean no additional AI influence; review never blocks EDI.

## 12. Rollback

Disable new influence, revoke adapter access if necessary, abort/roll back active work, clean exact unconsumed AI rows, verify safe, and return to current-data shadow. Consumed orders use native pre-EDI correction only; no compensating sidecar EDI or late rerun exists.

## 13. Migration without replacing MERLIN

Phases are discovery -> replay -> >=4-week shadow -> 20-store/two-category pilot -> bounded expansion. Pilot limits are <=min(5,000, measured adapter cap) rows/night, <=2 case packs/store-SKU plus lower constraints, and <=100 reviews/night. Every phase retains the same three interfaces and MERLIN authority.

## 14. Traceability, gates, and accountability

Every influenced row records snapshot, model, rules, intervals, constraints, decision, epoch/permit, exact keys/hashes, and certified post-generation correlation. Production requires measured MERLIN P99 <=90 minutes for the proposed close (or an earlier retested close), three consecutive 162M/42.24M runs at 30k/s and 15k/s, 198.7/240 target validation, calibrated data/quality, deterministic safety, country-local residency, certified adapter/review/kill behavior, approximately $3.8M observed annualized cost under the $4M ceiling, Finance approval, and funded named operators. Calculated targets are not claimed as measured evidence.
