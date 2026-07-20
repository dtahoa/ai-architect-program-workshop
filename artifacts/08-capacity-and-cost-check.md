# Supporting Evidence - Capacity and Cost Arithmetic

This is supporting evidence for Artifact 2, not a seventh submission artifact. Canonical gates are in [`07-nfr-table.md`](07-nfr-table.md).

## Population and peak reconciliation

- Forecast outputs: **27M/night** planning value (`42,000 products x 640 stores = 26.88M`, rounded conservatively).
- Stocked optimization candidates: **7.04M/night** (`11,000 stocked SKUs/store x 640 stores`).
- Conservative 6x tests: **162M forecast** and **42.24M optimization equivalents**. These are test populations, not observed peak results.
- Target rates: **30,000 forecasts/s** and **15,000 optimizations/s**.

## Per-run storage assumptions

The source gives no AI-output size or AI-artifact retention requirement. For capacity planning only, assume **1.2 KB per serialized forecast output** using decimal units:

- Normal run: `27M x 1.2 KB = 32.4 GB`.
- Conservative 6x run: `162M x 1.2 KB = 194.4 GB`.

These are uncompressed, per-run forecast-output estimates; they exclude replicas, indexes, optimizer intermediates, audit manifests, and backups. Measure actual serialized bytes/output during replay. Keep only the current run in the performance tier as a provisional design assumption; Data Governance must approve hot/warm tiers, deletion, legal holds, backups, and every AI snapshot/derived-artifact retention period before production. The source requirement remains unchanged: loyalty data stays country-local and POS records remain in the incumbent governed estate for at least five years. The AI sidecar neither shortens that POS retention nor duplicates or exports POS records.

## Sidecar target-path calculation

| Stage | Planning duration |
|---|---:|
| Snapshot ingestion and validation | 15.0 min |
| Forecasting at 162M / 30,000/s | 90.0 min |
| Optimization at 42.24M / 15,000/s | 46.93 min |
| Reconciliation, policy, and audit manifest | 11.73 min |
| Adapter preparation | 10.0 min |
| Guarded activation/commit allowance | 10.0 min |
| Contingency inside the sidecar wall | 15.0 min |
| **Total target path** | **198.66 min -> 198.7 min reported** |
| **240 min wall minus target path** | **41.34 min -> 41.3 min calculated headroom reported** |

The stage-precision sum is `15.0 + 90.0 + 46.93 + 11.73 + 10.0 + 10.0 + 15.0 = 198.66` minutes, reported as **198.7**. The remaining wall is `240 - 198.66 = 41.34` minutes, reported as **41.3**. Both values are unmeasured planning figures. Three consecutive production-equivalent 6x runs must pass before influence.

## Adapter-cap arithmetic

- Pilot raw minimum: `5,000 / 60 = 83.4 rows/s`.
- With 10 seconds transaction overhead and 20% headroom: `5,000 / ((60-10) x0.80) = 125 rows/s`.
- Expansion policy ceiling raw minimum: `70,400 / 60 = 1,173.4 rows/s`.
- With the same overhead and headroom: `70,400 / ((60-10) x0.80) = 1,760 rows/s`.

These rates do not assert that MERLIN supports the transaction. The actual cap is the smaller of policy scope and a SAP-certified measured rate/lock limit. Unsupported staging, activation, ownership, or cleanup means shadow-only.

## Approximate annual run-cost model

| Planning category | Approximate annual amount |
|---|---:|
| Batch compute and orchestration | $1.05M |
| Storage, transfer, and data processing | $0.60M |
| Security, audit, monitoring, and platform | $0.65M |
| Associate-copilot pilot and signed packs | $0.55M |
| Operations and support | $0.65M |
| Contingency | $0.30M |
| **Planning target** | **$3.80M** |

Nominal forecast count is `27M x365 = 9.855B/year`. Therefore `$3.8M / 9.855B ≈ $0.0003856` per nominal forecast; the `$4M` ceiling is approximately `$0.0004059`. These are blended planning ratios, not provider quotes or measured bills. Finance approval and observed tagged cost are production gates.
