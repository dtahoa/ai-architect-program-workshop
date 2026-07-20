# NovaMart Group 4 - One-Page Cheat Sheet

Presentation integrator: **ĐINH XUÂN DŨNG**

## Team ownership

A1 LÊ NGUYỄN SỸ BÌNH · A2 TRẦN THANH PHỤNG · A3 NGUYỄN HÒA · A4 PHẠM THỊ THANH HUYỀN · A5 TRẦM QUỐC THUẬN · A6 TRẦN TRỌNG PHÚ · Presentation ĐINH XUÂN DŨNG

## Opening / close

**Open:** Keep MERLIN authoritative; let AI fail without stopping a truck.
**Close:** Earn influence through evidence. Remove the sidecar and MERLIN still orders.

## Architecture in 20 seconds

Nightly immutable snapshot -> validate -> statistical forecast ML -> deterministic optimizer -> certified adapter -> override table -> MERLIN engine -> native review/EDI. Post-generation correlation may return through the certified review queue. No AI route reaches POS, suppliers, or EDI.

## Numbers to memorize

| Topic | Canonical answer |
|---|---|
| Clock | 01:30 retry stop; 01:45 complete set; 02:00 AI close; 04:00 EDI |
| Populations | 27M forecasts; 7.04M stocked candidates; 162M/42.24M 6x tests |
| Rates | 30k forecast/s; 15k optimization/s; three consecutive 6x passes |
| Path | 198.7/240 min; 41.3 min calculated, unmeasured headroom |
| Inventory | 78% accurate; intervals, age, caps, suppression; never shelf truth |
| Pilot | 20 stores; 2 low-risk categories; <=5,000 measured-cap rows; <=100 reviews/night |
| Kill | acknowledge <=30 s; reject writes <=60 s; verify safe <=5 min |
| Cost | approximately $3.8M/year target; $4M ceiling; approximately $0.0003856/forecast |

## Failure answers

- Late/incomplete/invalid/disabled AI: publish nothing; MERLIN continues.
- AI returns at 03:30: remain closed; diagnosis only.
- Unsafe adapter: abort/rollback, independently revoke SAP access, clean exact AI rows, remain shadow.
- Low-confidence inventory: suppress row; MERLIN computes incumbent result.
- Missing safety evidence: exact refusal/hold/check instruction and escalation; never generated fallback.
- Universal copilot refusal: fails usefulness acceptance.

## Evidence boundary

Calculated targets are not benchmark results. Production still needs measured MERLIN P99, certified adapter/review contracts, 6x tests, calibrated quality, safety/residency benchmarks, observed tagged cost, Finance approval, kill drills, and funded named operators.

## Answer pattern

**Decision -> number/boundary -> safe failure -> evidence status.** If unsure, return to the authority rule: MERLIN generates and sends every order; AI absence must be harmless.
