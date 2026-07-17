# NovaMart Variant B - Capacity and Cost Check

Owner: TRẦN THANH PHỤNG (218924, PH2)

Decision: **The architecture is arithmetically feasible but unproven. It passes for replay and shadow mode and fails the production-influence gate until measured tests replace the planning assumptions below.**

## 1. Workload reconciliation

The source contains two different populations. They must not be merged:

| Population | Arithmetic | Use in the design |
|---|---:|---|
| Chain-SKU/store forecast matrix | `42,000 x 640 = 26,880,000`, validated as 27,000,000/night | Conservative forecasting, feature, storage and cost baseline |
| Stocked store-SKU order candidates | `11,000 x 640 = 7,040,000`/night | Replenishment optimization and MERLIN order population |
| 6x forecast-equivalent stress | `27,000,000 x 6 = 162,000,000` records/night | Synthetic capacity and cost test while the source's 6x denominator is unresolved |
| 6x stocked optimization-equivalent stress | `7,040,000 x 6 = 42,240,000` records/night | Conservative optimizer/reconciliation test |

Lunar New Year demand can rise 6x without creating six times as many store-SKU keys. Treating it as six times the record cardinality is intentionally conservative. Production influence must still pass the test because the source does not define a narrower denominator.

## 2. Deadline and throughput proof

### 2.1 Fixed walls

- Source wall: 22:00-04:00 = 6 hours = 21,600 seconds.
- Sidecar publication wall: 22:00-02:00 = 4 hours = 14,400 seconds.
- Incumbent reserve: 02:00-04:00 = 120 minutes.
- Required MERLIN proof: `MERLIN P99 + 30-minute reserve <=120 minutes`, so measured MERLIN P99 must be <=90 minutes for the proposed 02:00 gate.
- If MERLIN P99 is `M` minutes, the safe close is `04:00 - M - 30 minutes`. The 04:00 wall never moves.

### 2.2 Stage calculations at the conservative 6x case

| Stage | Peak population | Maximum budget | Required floor | Acceptance throughput | Calculated time at target | Headroom/result |
|---|---:|---:|---:|---:|---:|---|
| Snapshot ingest and validation | 162.0M feature records | 1,200 s | `162M / 1,200 = 135,000/s` | >=180,000/s | 900 s =15.0 min | 25% time headroom |
| Feature plus forecast ML | 162.0M forecast equivalents | 7,200 s | `162M / 7,200 = 22,500/s` | >=30,000/s | 5,400 s =90.0 min | 33% throughput headroom |
| Deterministic optimization | 42.24M stocked equivalents | 3,600 s | `42.24M / 3,600 = 11,733/s` | >=15,000/s | 2,816 s =46.9 min | 27.8% throughput headroom |
| Reconciliation and complete-set scan | 42.24M stocked equivalents | 900 s | `42.24M / 900 = 46,933/s` | >=60,000/s | 704 s =11.7 min | 27.8% throughput headroom |
| Retry reserve | <=39 of 3,840 partitions | 600 s | See Section 3 | Complete inside reserve | <=600 s =10.0 min | Bypass above cap |
| Risk/audit manifest and checksum | One complete run | 600 s | N/A | <=600 s | <=10.0 min | Missing trace blocks publish |
| Adapter publication | Policy ceiling <=70,400 rows; actual measured cap may be lower | 900 s total; certified live lock `L_cert <=60 s` | Raw direct-write floor `70,400/60=1,173.4/s` before commit/checksum overhead; pilot `5,000/60=83.4/s` | Prefer existing supported non-live bulk staging then epoch-guarded <=5 s activation. Without it, cap=`floor(R_cert x (L_cert - T_overhead_p99) x0.80)`. Example with 10 s overhead: expansion needs 1,760/s and pilot 125/s | Preparation/checksum outside live lock; activation <=5 s if supported, or direct transaction <=measured cap/60 s; total <=15 min | Unsupported staging/epoch guard or insufficient direct P99 rate blocks/lower cap; contract proof missing |

Target-path total at 6x, using the full retry and publication budgets, is `15 + 90 + 46.9 + 11.7 + 10 + 10 + 15 =198.7 minutes`. Starting at 22:00 gives a calculated finish around 01:19 and 41.3 minutes of sidecar-wall headroom. The NFR remains stricter: measured P99 must be <=225 minutes and the hard close is 240 minutes.

This is only a capacity specification. It is not a benchmark result. Three consecutive end-to-end 6x runs on production-equivalent infrastructure must produce the evidence.

### 2.3 Overall-rate checks

- Normal full-wall minimum: `27M / 21,600 =1,250 forecasts/s`.
- Normal sidecar-wall minimum: `27M / 14,400 =1,875 forecasts/s`.
- Peak full-wall minimum: `162M / 21,600 =7,500 forecast-equivalents/s`.
- Peak sidecar-wall minimum: `162M / 14,400 =11,250 forecast-equivalents/s`.
- The 30,000/s forecast-stage acceptance target is 2.67x the overall peak sidecar minimum because the stage receives only 120 minutes, not four hours.

### 2.4 Data-volume check

Planning sizes, to be replaced by observed serialization sizes:

- Feature input at 2 KB/record: normal `27M x 2 KB =54 GB/night`; peak `162M x 2 KB =324 GB/night`.
- Forecast/recommendation/audit record at 1.2 KB: normal `27M x 1.2 KB =32.4 GB/night`; peak `162M x 1.2 KB =194.4 GB/night`.
- Five-year uncompressed decision volume including the 14-day 6x case: `11.745B records/year x 1.2 KB x 5 =70.47 TB`.
- Worst 30-day hot set with 14 peak and 16 normal days: `(162M x 14 + 27M x 16) x 1.2 KB =3.24 TB` before replication and indexes.

Compression, replicas, indexes, and legal holds must be measured; capacity cannot be approved from raw-byte arithmetic alone.

## 3. Partition, retry and deadline protection

The planning partition count is 3,840, or six logical shards per store:

- Normal average: `27M / 3,840 =7,031` forecast outputs/partition.
- 6x average: `162M / 3,840 =42,188` forecast-equivalent records/partition.
- At 30,000/s aggregate, 39 parallel 6x retry partitions contain `39 x 42,188 =1.645M` records and need about 55 aggregate compute-seconds, excluding scheduling and backoff.
- Two retry delays are 30 s and 120 s. Two repeat computations plus delay remain below 5 minutes at the target; the reserved 10 minutes covers scheduling variance.

Controls:

1. Each task is idempotent by run, partition and version.
2. Maximum two retries; a retry inherits the original stage deadline.
3. At most 1%, or 39, partitions may enter retry during the acceptance case. More means systemic failure and immediate bypass.
4. No new stage retry starts after 01:30; no new adapter retry starts after 01:55; all writes close at 02:00.
5. Partial success is not publication success. Missing manifest entries invalidate the publish scope.
6. Recovery at 03:30 remains closed. It cannot reclaim the day or send a replacement EDI message.

## 4. Override-table and human-capacity limits

The compute pipeline may evaluate every stocked pair, but the legacy adapter is intentionally bounded:

| Mode | Maximum published AI rows/night | Maximum AI review exceptions/night | Release condition |
|---|---:|---:|---|
| Shadow | 0 | 0 in MERLIN; comparison only after documented supported post-generation output | Default until all gates pass |
| Initial pilot | `min(5,000, measured transaction cap)` | 100 | 20 stores, two low-risk non-fresh categories, same-transaction epoch guard, certified post-generation queue fields/correlation and measured capacity |
| Controlled expansion | `min(70,400, floor(R_cert x (L_cert - T_overhead_p99) x0.80))` | Smaller of measured human capacity and risk-ranked demand | Recertify rate/overhead/lock/cleanup and queue join at each raised cap |

The 15-minute stage budget is not the live-table rate. `70,400/60=1,173.4 rows/s` is the absolute direct-write minimum before overhead. Reserving 10 seconds for begin/epoch-check/commit/checksum and 20% headroom gives `70,400/(50 x0.80)=1,760 rows/s`; the analogous pilot rate is `5,000/(50 x0.80)=125 rows/s`. The certified P99 measurement, not 1%, sets capacity. If the existing supported contract provides non-live bulk staging and constant-time pointer/partition activation, stage/checksum outside the read set and require an epoch-guarded activation <=5 seconds. No new table/swap is assumed: without that contract, use the direct formula; if ownership, epoch, rate, lock or cleanup fails, lower the cap or remain shadow.

The sidecar does not receive a current MERLIN min/max result before publication. Pre-publication controls use absolute case/shelf-life/inventory/DC/category limits. The review-queue contract must expose post-generation generated order ID/quantity/correlation keys for comparison/audit; absence keeps automated influence disabled.

## 5. Copilot capacity and deferred online substitution

### 5.1 Copilot pilot

- Daily planning demand: `8,400 associates x 20 queries =168,000 queries/day`.
- Busiest 15 minutes at 10% of daily traffic: `16,800 / 900 =18.7 requests/s`.
- 5x burst: `18.7 x 5 =93.3 requests/s`.
- Acceptance load: 150 requests/s with online p95 <=1.5 s and p99 <=2.0 s.
- By Little's Law, 150/s at 1.5 s implies about 225 in-flight requests; provision and test at >=300 concurrent requests.
- Offline test: 4% of 640 =25.6 stores. Round to 26 and test 2x, or 52 simultaneously disconnected stores. Their local queries create 0 central requests/s.

Offline acceptance cannot be satisfied by refusing everything. On a pre-labelled benchmark, >=90% of eligible non-safety questions must return a correct cited answer with >=95% answer/citation accuracy; ineligible non-safety refusal precision and recall must each be >=95%. Safety policy correctness, exact eligible display/citation correctness, and required-refusal precision/recall must each be 100%. Safety output is exact approved structured data/passage plus fixed wording, never LLM-composed. Signed-pack coverage is 100%, p95 <=500 ms, and a 100% refusal implementation fails. A stale recall manifest correctly refuses but does not count as an eligible answer.

### 5.2 Online substitution

Online substitution is not included in Release 1. Production throughput is 0 requests/s and there is no serving endpoint. Therefore the source's p95 <500 ms target is not claimed as met. It is an entry criterion for a future architecture with its own observed RPS, inventory semantics, tail budget, failure path and owner.

## 6. Annual operating-cost model

### 6.1 Workload quantities

- Nominal forecasts/year: `27M x 365 =9.855B`.
- Conservative peak-adjusted equivalents: `27M x 351 normal days + 162M x 14 peak days =11.745B/year`.
- Copilot queries/year: `168,000 x 365 =61.32M`.

The unit rates below are planning ceilings, not vendor quotes. Procurement and measured pilot bills must replace them before production influence.

| Annual operating item | Driver and arithmetic | Budget |
|---|---|---:|
| Feature, forecast and optimizer variable compute | `11.745B equivalents x $0.00010` | $1,174,500 |
| Batch orchestration, feature services, model training and registry | Fixed planning envelope including monthly retraining and peak reservations | $350,000 |
| Decision/audit storage, query and backup | Up to 3.24 TB hot and 70.47 TB five-year raw/cold before compression; managed envelope | $220,000 |
| Secure on-prem connectivity and transfer | Normal 0.5 TB/night, 3 TB/night peak; private links, egress and transfer envelope | $120,000 |
| Copilot online inference | `61.32M queries x $0.006/query` | $367,920 |
| Copilot evidence index and signed offline distribution | Three languages, 1,400 governed documents, product facts and 640-store pack distribution | $180,000 |
| Monitoring, security, backup/DR and compliance tooling | Metrics, logs, alerts, scanning and recovery exercises | $250,000 |
| Operational support and SAP retainer | 3.0 shared FTE-equivalent planning allowance plus partner support; see Section 9 | $650,000 |
| Expected subtotal | Sum of the eight items | **$3,312,420** |
| Cost/rate/volume contingency | `$3,800,000 - $3,312,420` =14.72% of subtotal | **$487,580** |
| Annual operating target | Expected subtotal plus contingency | **$3,800,000** |
| Board ceiling headroom | `$4,000,000 - $3,800,000` | **$200,000 (5%)** |

### 6.2 Unit and daily checks

- Forecast compute unit target: `$1,174,500 / 11.745B = $0.0001000/forecast-equivalent`.
- Blended total AI target divided by nominal forecasts: `$3.8M / 9.855B = $0.0003856/nominal forecast`.
- Board ceiling divided by nominal forecasts: `$4M / 9.855B = $0.0004059/nominal forecast`, consistent with the source's rounded $0.0004.
- Blended total AI target divided by peak-adjusted equivalents: `$3.8M / 11.745B = $0.0003235/equivalent`; this is reported only for comparison because copilot/support costs are not forecast costs.
- Target annual average: `$3.8M / 365 = $10,411/day`, below the board's `$4M / 365 = $10,959/day`.
- Expected normal day before contingency: `$2,700 forecast compute + $1,008 copilot inference + $4,849 fixed allocation = about $8,557/day`.
- Expected peak day before contingency: `$16,200 forecast-equivalent compute + $1,008 copilot inference + $4,849 fixed allocation = about $22,057/day`.

The peak day exceeds the board's average daily figure; that is not a breach if the annual peak-adjusted total remains within $4M. Monitoring therefore uses rolling annual projection, not an incorrect flat daily hard cap.

### 6.3 Cost gates

- Warning: rolling annualized cost >$3.80M, batch compute >$0.00011/equivalent, 7-day average >$10,411/day, or untagged cost >0.5%.
- Critical: rolling annualized cost >$4.00M, blended nominal cost >$0.0004059, or untagged cost >1%.
- Response order: freeze expansion; stop offline markdown analysis; rate-limit copilot non-safety queries; reduce retraining frequency if model risk accepts; remain shadow-only. MERLIN capacity and ordering are never reduced.

## 7. Reconciling $3.2M delivery with $4M/year run cost

The source says Variant B has the "same $3.2M" as Variant A but does not repeat Variant A's capex/18-month basis. It separately approves $4M/year for total AI run cost. This document treats them as different controls:

| One-time 8-month delivery envelope | Budget |
|---|---:|
| SAP-certified extension discovery, adapter build and contract tests | $600,000 |
| Data, forecasting ML and deterministic optimizer implementation | $750,000 |
| Platform, security, audit and observability implementation | $550,000 |
| Read-only copilot pilot, safety content governance and localization | $300,000 |
| QA, 6x load/failure tests, historical replay and shadow evaluation | $450,000 |
| Program, legal, training and change management | $200,000 |
| Delivery contingency | $350,000 |
| **Delivery ceiling** | **$3,200,000** |

This is a top-down envelope, not a supplier quote. It deliberately does not hide recurring run cost inside capex.

- First-year cash if the controls are separate: `$3.2M delivery + $3.8M run = $7.0M`.
- If the $3.2M must include first-year run cost, the design exceeds it by up to `$7.0M - $3.2M = $3.8M`; production scope must be reduced or the design rejected.
- Finance must confirm whether internal labor, SAP partner fees, first-year cloud use and support are counted in delivery, run cost, or both. Double counting and excluded labor are both prohibited.

## 8. Margin and ROI challenge

- NovaMart annual net profit at the stated margin is `$4.1B x 2.4% = $98.4M`.
- The $3.8M run target consumes `3.8 / 98.4 =3.86%` of current net profit and `3.8 / 4,100 =0.0927%` of revenue. Cost control is material despite the large upside.
- Annual break-even capture against the identified $241M opportunity is `$3.8M / $241M =1.58%`.
- If delivery is amortized over three years, annualized cost is `$3.8M + $3.2M/3 = $4.867M`; break-even capture is `4.867 / 241 =2.02%`.
- First-year cash break-even is `$7.0M / $241M =2.90%` of the identified upside.

These ratios are not an ROI forecast. The $241M is an opportunity estimate, not realized benefit. Influence expansion requires measured incremental out-of-stock/waste improvement, control groups, and Finance-approved attribution. Headcount reduction is excluded.

## 9. Operational support capacity

The $650,000/year support envelope assumes shared internal roles and a bounded SAP partner retainer, not a wholly new team. Minimum operational coverage is:

- four cross-trained platform/data engineers to sustain a primary plus backup rotation no more frequent than 1-in-4;
- two named SAP integration contacts, one primary and one backup;
- one named forecasting/model owner and one deterministic-rules owner (roles may be shared if competency is proved);
- one replenishment operations duty owner per nightly shift;
- one food-safety content owner and market safety-duty escalation;
- a service owner, Security and Data Protection contacts.

Service targets are 21:30-04:30 primary/backup coverage every night, P1 acknowledgement <=10 minutes, kill switch <=5 minutes, <=2 actionable pages/person/week in normal periods, and a staffed peak roster for all 14 Lunar New Year days. If these are all incremental full-time hires, $650,000 is unlikely to be sufficient; the cost model must be revised and the scope may breach $4M. Until named staffing and loaded cost are known, this is a conditional gate.

## 10. Required failure and load evidence

| Test | Injected condition | Required result | Reject condition |
|---|---|---|---|
| Full AI outage | All sidecar services unavailable from 22:00 | No adapter call; MERLIN min/max and EDI finish normally | MERLIN waits, errors, or misses a truck because of AI |
| 6x end-to-end | 162M forecast and 42.24M optimizer equivalents | Three consecutive nights; P99 <=225 min; all stage floors and cost gates pass | Any write after 02:00 or throughput below a floor |
| Bounded retry | 39/3,840 partitions fail once/twice | Complete inside 10-minute reserve with no duplicates | Partial publish, deadline extension or duplicate current-run key |
| Retry storm | >39 partitions fail | Run becomes bypass before reserve is consumed | System continues retrying toward 04:00 |
| Stale/incomplete data | Age >24 h, completeness <99.5%, missing partition, schema/checksum fault | Affected scope bypasses; no guessed data published | Invalid scope publishes |
| Inventory uncertainty | Interval coverage <85% or missing interval | Affected rows suppressed; MERLIN independently computes its result | Point stock value drives an override |
| Adapter rate/lock/epoch | Direct rate below measured cap, lock exceeds certified limit, stale epoch/check-write race, commit fails | Transaction abort/rollback, staged rows inert, circuit open, independent session revoke, MERLIN released, exact ledger retained | Partial/old-epoch activation, manual-row deletion, lock/scheduler delay, or cap not supported by measured formula |
| Audit unavailable | Append-only audit commit fails | AI publication/answer blocked; MERLIN continues | Untraceable action published |
| Store connectivity | 52 stores disconnected for >1 h | p95 <=500 ms, pack 100%; eligible non-safety coverage >=90% and accuracy >=95%; ineligible refusal P/R >=95%; safety policy/display/refusal metrics 100% | 100% refusal, any metric below target, POS impact, unsigned display, or generated/uncited safety text |
| Safety conflict | Product master and current authority disagree | Deterministic renderer refuses; escalation ack <=5 min | Any model or policy selects/paraphrases/invents a displayed safety answer |
| Residency canary | Synthetic member identifier enters regional pipeline | DLP blocks/quarantines before egress; AI scope bypasses | Any cross-border personal byte |
| Cost surge | Unit rates +20% and copilot burst 5x | Projection alert, expansion freeze and noncritical throttling | Projected annual cost remains >$4M without approved remediation |

## 11. Capacity and cost verdict

No arithmetic proves the architecture impossible: the required peak stage rates are credible batch targets, the four-hour sidecar close preserves a two-hour incumbent reserve, and the $3.8M operating target is below the $4M ceiling. The design also avoids an LLM-per-forecast cost catastrophe.

The current evidence is insufficient for production influence. There is no observed MERLIN P99, production-equivalent 6x benchmark, certified post-generation queue join, same-transaction epoch guard, staging/activation or direct-rate/lock result, independent revoke drill, provider bill, Finance ruling, or named support roster. Therefore the design **passes conditionally for historical replay and shadow mode and must be rejected for production influence until those measurements pass**.
