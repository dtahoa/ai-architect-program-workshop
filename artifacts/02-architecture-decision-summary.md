# NovaMart Variant B - Canonical Decision and Number Registry

Status: **Source of truth for cross-artifact decisions and planning values.** Values marked `target` or `gate` are not measured evidence.

## Ownership registry

| Canonical artifact | Files | Owner |
|---|---|---|
| A1 - C4 context, container, and flows | `03-c4-context.mmd`, `04-c4-container.mmd`, `05-request-and-batch-flows.md` | LÊ NGUYỄN SỸ BÌNH |
| A2 - Quantified NFR table | `07-nfr-table.md` | TRẦN THANH PHỤNG |
| A3 - Technology-selection matrix | `10-technology-selection-matrix.md` | NGUYỄN HÒA |
| A4 - ADR | `11-adr.md` | PHẠM THỊ THANH HUYỀN |
| A5 - Top-five risk register | `12-risk-register.md` | TRẦM QUỐC THUẬN |
| A6 - Variant B legacy artifact | `13-variant-b-artifact.md` | TRẦN TRỌNG PHÚ |
| Presentation integration | `presentation/01-eight-slide-outline.md` through `04-one-page-cheat-sheet.md` | ĐINH XUÂN DŨNG |

## Decision registry

| ID | Canonical decision | Evidence status |
|---|---|---|
| D-01 | Use an external, asynchronous, fail-open AI sidecar. Shadow-assist is the initial rollout state, not the architecture. | Approved design decision |
| D-02 | MERLIN remains the sole order system of record, order generator, native review authority, and supplier-EDI sender. | Hard constraint |
| D-03 | Cross the MERLIN boundary only through nightly batch snapshots, the pre-generation override table, and the post-generation review queue. | Hard constraint |
| D-04 | POS is untouched. AI has no POS code, route, credential, runtime dependency, or availability budget. | Hard constraint |
| D-05 | Use statistical/time-series ML for forecasts and deterministic rules/optimization for quantities. Never use an LLM for numeric decisions. | Approved design decision |
| D-06 | Treat 78% inventory accuracy as uncertainty: age, calibrated intervals, absolute caps, and suppression; never claim real-time shelf truth. | Required control |
| D-07 | Display safety/allergen content only as an exact approved field or passage plus fixed approved locale wording and citation. Missing or conflicting evidence causes refusal and escalation; universal refusal fails acceptance. | Required control |
| D-08 | Exclude member-level loyalty data from Release 1 regional processing; personal data stays country-local. | Required control |
| D-09 | Release 1 prioritizes demand forecasting and replenishment recommendations. Associate copilot is a separate read-only pilot. Markdown, online substitution, and promotion-planning automation are deferred. | Scope decision |

## Number registry

| Topic | Canonical value | Status / interpretation |
|---|---:|---|
| Operating window | 22:00-04:00 | Fixed source constraint |
| Retry stop / complete-set gate / AI write close / EDI cut-off | 01:30 / 01:45 / 02:00 / 04:00 | Proposed operating gates; 02:00 requires measured MERLIN P99 <=90 min or moves earlier |
| Forecast outputs / stocked candidates | 27M / 7.04M per night | Planning populations |
| Conservative 6x tests | 162M forecast / 42.24M optimization equivalents | Test populations, not observed peak results |
| Design throughput | >=30,000 forecasts/s / >=15,000 optimizations/s | Targets; require three consecutive 6x passes |
| Sidecar target path | 198.66 -> 198.7 min of 240; 41.34 -> 41.3 min headroom | Stage-precision calculation; unmeasured design budget |
| Inventory accuracy | 78% | Source fact; 22% uncertainty is not shelf truth |
| Pilot scope | 20 stores; 2 low-risk non-fresh categories; <=min(5,000, measured adapter cap) rows/night; <=100 review exceptions/night | Conditional gate |
| Kill timing | acknowledge <=30 s; reject new writes <=60 s; verified safe <=5 min | Test targets |
| Cost | approximately $3.8M/year target; $4.0M/year ceiling; approximately $0.0003856 per nominal forecast | Planning estimate; provider bills and Finance approval required |
| Quality | enterprise rolling-28-day WAPE <=25%; promotion WAPE <=35%; calibrated 90% inventory interval coverage >=90% | Release gates |
| Data | snapshot age <=24 h; required-field completeness >=99.5%; manifest 100% | Release gates |
| Safety | 0 generated safety displays; exact approved evidence/citation or refusal; all safety benchmark metrics 100% | Acceptance-critical gate |

The workshop's `03:15` value is a source example only. It is not an operational gate in this design. The canonical late-result behavior is: stop retries at 01:30, require completeness by 01:45, close AI writes at 02:00, and reject any later recovery.

## Production-influence gate

Production influence remains disabled until measured MERLIN timing, certified adapter and review contracts, three consecutive 6x tests, data/quality/residency/safety checks, kill-switch drills, observed cost, review capacity, and funded operating ownership all pass. Replay and shadow results must be labeled evidence; calculations and thresholds alone are not proof.
