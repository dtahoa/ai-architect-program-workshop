# Artifact 3 - Technology-Selection Matrix

Owner: NGUYỄN HÒA (122989, PH2)

Status: Decision support approved for replay and shadow mode. Production influence remains conditional on the release gates in `artifacts/07-nfr-table.md` and `artifacts/09-safety-and-governance.md`.

## Scoring method and evidence boundary

- Every criterion is scored from 1 (poor) to 5 (excellent).
- Weights total 100 for each decision and were set from the hard constraints before scoring options. Continuity, safety, the fixed 04:00 deadline, and supported legacy boundaries therefore carry more weight than convenience.
- Weighted result = `sum(weight x score) / 100`, with a maximum of 5.00.
- Scores are architecture judgments based on the documented legacy interfaces and planning arithmetic. They are not vendor benchmarks or quotes.
- A high score does not waive a release gate. The selected options may run in historical replay and shadow mode; MERLIN influence remains blocked until measured timing, certified adapter, 6x, cost, review-capacity, residency, safety-authority, and staffing evidence passes.

## Decision A - AI-Retrofit integration pattern

### Criteria and weights

| Code | Criterion | Weight | Why this weight is honest |
|---|---|---:|---|
| A1 | Uses only supported MERLIN boundaries | 25 | A fourth integration point or core modification is prohibited. |
| A2 | Failure isolation and replenishment continuity | 25 | AI must never stop incumbent ordering. |
| A3 | Asynchronous fit with the 22:00-04:00 batch | 15 | A synchronous dependency endangers the fixed cut-off. |
| A4 | Deliverability in 8 months | 10 | The retrofit has a fixed program window. |
| A5 | Rollback and operational control | 10 | The kill switch and bounded recovery must be executable. |
| A6 | Independent scale and release | 10 | The 6x case must scale without a MERLIN release. |
| A7 | Completeness as an integration architecture | 5 | A rollout mode alone does not define deployment or interfaces. |
|  | **Total** | **100** |  |

| Option | A1 | A2 | A3 | A4 | A5 | A6 | A7 | Weighted result |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| External fail-open sidecar | 5 | 5 | 5 | 4 | 5 | 4 | 5 | **4.80** |
| Event-driven integration across the legacy boundary | 2 | 3 | 2 | 2 | 3 | 4 | 4 | 2.65 |
| API gateway / synchronous service path | 2 | 2 | 1 | 2 | 3 | 3 | 4 | 2.15 |
| Shadow-assist treated as the architecture | 3 | 5 | 5 | 5 | 5 | 2 | 1 | 4.00 |
| In-process MERLIN plugin | 1 | 1 | 4 | 1 | 1 | 1 | 5 | 1.65 |

**Selected:** External fail-open sidecar. It uses the existing nightly batch, override table, and review queue through narrow adapters while MERLIN remains independently scheduled and authoritative.

**Trade-off accepted:** Snapshot synchronization and possible divergence from MERLIN are added. Immutable snapshot IDs, checksums, complete-run manifests, bounded overrides, and reconciliation control that risk. Shadow-assist remains the initial rollout mode, not a competing architecture.

## Decision B - Forecasting platform and execution model

### Criteria and weights

| Code | Criterion | Weight | Why this weight is honest |
|---|---|---:|---|
| B1 | 6x throughput and deadline headroom | 25 | The target is 162M forecast-equivalent records inside a fixed wall. |
| B2 | Cost controllability under $4M/year total AI cost | 20 | Forecast compute must leave room for audit, support, and the copilot pilot. |
| B3 | Fitness for numeric forecasting | 20 | WAPE, intervals, and reproducibility are central requirements. |
| B4 | Isolation from MERLIN and POS | 15 | Neither legacy critical path may depend on model execution. |
| B5 | Traceability and reproducibility | 10 | Published decisions require model, data, feature, and output provenance. |
| B6 | Operational fit | 10 | The Variant B team is unconfirmed, so avoid a needlessly bespoke platform. |
|  | **Total** | **100** |  |

| Option | B1 | B2 | B3 | B4 | B5 | B6 | Weighted result |
|---|---:|---:|---:|---:|---:|---:|---:|
| Statistical ML on elastic partitioned batch workers | 5 | 4 | 5 | 5 | 4 | 3 | **4.50** |
| Statistical ML on a fixed on-premises cluster | 3 | 3 | 5 | 5 | 5 | 3 | 3.90 |
| Managed AutoML batch service | 4 | 3 | 4 | 5 | 3 | 4 | 3.85 |
| LLM-generated numeric forecasts | 1 | 1 | 1 | 2 | 1 | 1 | 1.15 |

**Selected:** Provider-neutral statistical ML on 3,840 independently retryable, elastic batch partitions in an approved data region. Acceptance requires >=30,000 forecast outputs/s in the conservative 6x test and a complete current-run manifest.

**Trade-off accepted:** Elastic execution adds platform and cost-governance complexity. The design accepts that to avoid buying permanent 6x capacity. Autoscaling caps, pre-peak reservation, cost tags, idempotent partitions, and the 01:30 retry stop bound the exposure. The choice of a specific service/provider is deliberately deferred until production-equivalent benchmarks and signed quotes exist.

## Decision C - Batch orchestration and compute strategy

### Criteria and weights

| Code | Criterion | Weight | Why this weight is honest |
|---|---|---:|---|
| C1 | Deadline gates and bounded retry | 25 | No retry may consume the 02:00-04:00 incumbent reserve. |
| C2 | Failure isolation | 20 | Partial or failed AI work must become bypass, not a MERLIN error. |
| C3 | 6x horizontal scale | 20 | Peak tests require parallel work and back-pressure. |
| C4 | Idempotency, manifest, and audit support | 15 | Partial success cannot cross the legacy boundary. |
| C5 | Fit with nightly legacy inputs | 10 | Sources are snapshot-oriented, not real-time event streams. |
| C6 | Support and cost simplicity | 10 | Operational load is material and staffing is still a gate. |
|  | **Total** | **100** |  |

| Option | C1 | C2 | C3 | C4 | C5 | C6 | Weighted result |
|---|---:|---:|---:|---:|---:|---:|---:|
| Deadline-aware workflow orchestrator plus partitioned batch workers | 5 | 5 | 5 | 5 | 4 | 3 | **4.70** |
| One monolithic scheduled batch job | 2 | 4 | 2 | 2 | 4 | 4 | 2.80 |
| Always-on streaming/event backbone | 4 | 4 | 5 | 4 | 2 | 2 | 3.80 |
| MERLIN-native/in-process orchestration | 2 | 1 | 1 | 2 | 1 | 2 | 1.50 |

**Selected:** Deadline-aware workflow orchestration with idempotent partitioned workers, maximum two retries, a 01:30 retry-stop gate, a 01:45 complete-set gate, and a 02:00 adapter hard close.

**Trade-off accepted:** More run-state and partition metadata must be operated than with a single job. That complexity buys bounded retry, complete-set publication, 6x scale, and auditable bypass. Internal queues/events are implementation mechanics inside the sidecar; they do not create an event-driven MERLIN integration.

## Decision D - Associate-copilot retrieval architecture

### Criteria and weights

| Code | Criterion | Weight | Why this weight is honest |
|---|---|---:|---|
| D1 | Evidence safety and refusal controls | 30 | An unsupported allergen answer can cause fatal harm and criminal liability. |
| D2 | Offline operation | 20 | At least 4% of stores lose connectivity for more than one hour weekly. |
| D3 | Tail latency | 15 | Online p95 must be <2 seconds; offline retrieval has a <=500 ms target. |
| D4 | Multilingual language usefulness | 10 | The governed corpus spans three languages. |
| D5 | Isolation from POS and ordering | 10 | Copilot must not enter either critical path. |
| D6 | Operating cost and support | 10 | Copilot shares the $4M/year total AI ceiling. |
| D7 | Decision auditability | 5 | Every safety response/refusal needs evidence and version traceability. |
|  | **Total** | **100** |  |

| Option | D1 | D2 | D3 | D4 | D5 | D6 | D7 | Weighted result |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Governed deterministic safety retrieval/display, optional non-safety language assistance, and signed offline packs | 5 | 5 | 4 | 4 | 5 | 3 | 5 | **4.55** |
| Generic cloud chatbot | 1 | 1 | 3 | 5 | 4 | 3 | 2 | 2.25 |
| Keyword search plus signed offline packs, no language model | 4 | 5 | 5 | 2 | 5 | 4 | 4 | 4.25 |
| Governed central retrieval without offline packs | 5 | 1 | 4 | 4 | 5 | 4 | 5 | 3.85 |

**Selected:** Governed evidence retrieval with deterministic exact safety-field/passage rendering plus fixed approved wording/citation, optional language processing only for input classification/entities/translation and non-safety assistance, and signed store-local packs. No LLM may compose, paraphrase, or translate displayed allergen/food-safety/recall/sellability/storage/shelf-life content. Definitive safety answers remain disabled until the governance gates pass; missing, stale, conflicting, unsigned, or inexact evidence causes refusal/escalation.

**Trade-off accepted:** Signed pack publication, exact locale-specific safety templates, revocation, and three-language governance are more complex than keyword search. That cost is accepted for safe offline evidence. Optional language inference cannot be used on the safety display path and must be justified by non-safety lookup value; the deterministic path is mandatory, not a fallback. The language model has no route to forecasting, optimization, POS, or MERLIN publication.

## Cross-decision result and unresolved gates

The four choices form one coherent boundary: an external sidecar runs deadline-aware elastic statistical forecasting and deterministic optimization; MERLIN receives only certified absolutely bounded outputs; the separate copilot uses deterministic exact safety rendering and confines language technology to input handling/non-safety assistance.

These matrices do not resolve the known unknowns. Production influence remains rejected until all eight NFR production gates and all applicable safety/governance gates pass, including observed MERLIN P99 <=90 minutes for the proposed 02:00 close (or an earlier retested close), three consecutive 6x runs, certified atomic override behavior, <=5-minute kill-switch proof, calibrated inventory intervals, measured review capacity, signed Finance baselines, and named funded owners.
