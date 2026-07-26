# Artifact 3 - Technology-Selection Matrix

Owner: **NGUYỄN HÒA**

## Reproducible scoring method

For every decision, criteria weights total 100. Each option receives an integer score and the weighted result is `sum(weight x score) / 100`.

| Score | Anchor |
|---:|---|
| 1 | Violates a hard constraint or requires an unsupported capability |
| 2 | Major gap; mitigation adds material delivery or operational risk |
| 3 | Feasible with known trade-offs and evidence still required |
| 4 | Strong fit; bounded gaps have credible controls |
| 5 | Directly satisfies the criterion using supported boundaries |

## Decision 1 - MERLIN integration pattern

Criteria: legacy safety **30**, fail-open continuity **25**, supported-interface fit **20**, rollback **15**, cost/operability **10** = **100**.

| Option | Legacy safety | Fail-open | Interface fit | Rollback | Cost/ops | Weighted total |
|---|---:|---:|---:|---:|---:|---:|
| External asynchronous sidecar | 5 | 5 | 5 | 5 | 3 | **4.80** |
| Event-driven boundary | 3 | 3 | 2 | 4 | 3 | 2.95 |
| Synchronous API gateway | 2 | 1 | 1 | 3 | 3 | 1.80 |
| In-process MERLIN plugin | 1 | 1 | 1 | 1 | 4 | 1.30 |

**Selected:** external fail-open sidecar; shadow-assist is its initial rollout state. **Accepted trade-off:** duplicated snapshots, reconciliation, narrow SAP-certified adapter work, and two-system operations in exchange for independent failure and rollback.

**Sensitivity:** moving 10 points from legacy safety to cost/operability leaves sidecar first at 4.60 versus event-driven 2.95. The decision is not created by one favorable weight.

## Decision 2 - Forecast execution platform

Criteria: deadline control **25**, 6x scale **25**, operability **20**, cost **15**, nightly-legacy fit **15** = **100**.

| Option | Deadline | 6x scale | Operability | Cost | Legacy fit | Weighted total |
|---|---:|---:|---:|---:|---:|---:|
| Managed elastic batch | 5 | 5 | 4 | 4 | 5 | **4.65** |
| Kubernetes batch workers | 5 | 5 | 3 | 3 | 4 | 4.15 |
| Managed Spark | 4 | 5 | 3 | 3 | 4 | 3.90 |
| Fixed on-premises compute | 3 | 2 | 4 | 2 | 5 | 3.10 |

**Selected:** managed elastic batch with deadline-aware orchestration and idempotent partitions. **Accepted trade-off:** provider dependence, data-transfer controls, and cost variability; residency-approved deployment, quotas, tags, and an expansion freeze bound them.

**Sensitivity:** shifting 15 points from 6x scale to operability keeps managed batch first at 4.50 versus Kubernetes 3.85. Provider choice remains conditional on benchmark and residency evidence.

## Decision 3 - Forecast model strategy

Criteria: WAPE potential **25**, promotion behavior **20**, uncertainty output **20**, scale **15**, operability **10**, cost **10** = **100**.

| Option | WAPE | Promotion | Uncertainty | Scale | Operability | Cost | Weighted total |
|---|---:|---:|---:|---:|---:|---:|---:|
| Hybrid hierarchical statistical ML | 5 | 5 | 5 | 4 | 3 | 4 | **4.55** |
| One global statistical ML model | 4 | 4 | 4 | 5 | 5 | 5 | 4.35 |
| Per-store-SKU models | 5 | 4 | 4 | 1 | 1 | 1 | 3.20 |
| Classical statistical baseline | 3 | 2 | 3 | 5 | 5 | 5 | 3.50 |

**Selected:** hybrid hierarchical statistical ML, subject to representative backtests. **Accepted trade-off:** more feature/model governance than a global model, but better sparse-series sharing, promotion handling, and calibrated intervals. No LLM is considered because numeric forecasts require reproducibility, scale, and approximately $0.0004 blended cost per forecast.

**Sensitivity:** moving 10 points from WAPE to cost produces a 4.45 tie between hybrid and global strategies. Therefore the final model choice must be decided by WAPE, promotion, interval-calibration, throughput, and cost evidence in replay; the architecture supports either statistical option without changing MERLIN.

## Decision boundary

Scores select a direction; they do not prove production readiness. Production influence still requires the NFR, adapter, safety, cost, operational, and rollout gates in Artifacts 2 and 6.
