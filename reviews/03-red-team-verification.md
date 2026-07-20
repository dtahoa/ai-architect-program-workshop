# NovaMart Group 4 - Red-Team Correction Verification

> **Historical verification record — superseded by [`final-review.md`](final-review.md).** The current artifacts additionally resolve the former RT-06 and RT-07 minor findings.

Verification date: 2026-07-17

Reviewer role: Strict workshop trainer / red-team verifier

Scope:

- Corrected `artifacts/02-architecture-decision-summary.md` through `artifacts/14-final-artifact-pack.md`
- `reviews/01-red-team-review.md`
- `reviews/02-resolution-log.md`
- Original Workshop 1 PDF and attached orchestration requirements

Purpose: verify closure of RT-01 through RT-05 only, recheck related cross-file contradictions and arithmetic, and identify any remaining Critical or Major issue that is real and reproducible.

## Verification outcome

All five Critical/Major findings are resolved at architecture-artifact level. No remaining Critical or Major finding was reproduced.

This does not convert unmeasured production assumptions into evidence. Replenishment influence correctly remains blocked until MERLIN timing, extension contracts, 6x runs, transaction rate/lock behavior, kill-switch drills, review capacity, Finance treatment, and named operations ownership pass. Definitive safety answers correctly remain blocked until authoritative content, deterministic multilingual benchmarks, signing/revocation, audit, residency, and escalation gates pass.

## RT-01 through RT-05 closure

### RT-01 - Critical - Generative allergen and food-safety output

**Status: Closed.**

Verified evidence:

- `artifacts/04-c4-container.mmd` separates a non-safety language service from a deterministic safety renderer.
- `artifacts/05-request-and-batch-flows.md` routes safety domains to an exact approved structured field/passage plus fixed pre-approved locale wording and citation. The LLM may process input intent/entities or non-safety questions but cannot compose, paraphrase, translate, or summarize displayed safety output.
- `artifacts/07-nfr-table.md`, NFR-15 and NFR-19, set generated safety output to zero, require deterministic output/template hashes, and treat any LLM-produced safety display as critical.
- `artifacts/09-safety-and-governance.md` requires exact deterministic display or refusal, with 100% policy, eligible-display/citation, and required-refusal benchmark results.
- `artifacts/10-technology-selection-matrix.md`, `artifacts/11-adr.md`, `artifacts/12-risk-register.md`, and `artifacts/14-final-artifact-pack.md` carry the same prohibition.

Stale-claim search found no remaining artifact text that permits an LLM to summarize, paraphrase, translate, or compose displayed allergen/food-safety/recall/sellability/storage/shelf-life output.

### RT-02 - Major - Unsupported pre-publication MERLIN baseline

**Status: Closed.**

Verified evidence:

- The corrected flow explicitly says the sidecar does not read or recreate a current MERLIN result before publication.
- Pre-publication policy uses non-negative whole cases, an initial maximum of two case packs/store-SKU, and the lower of shelf-life sell-through, inventory-interval feasibility, DC allocation, and category absolute caps.
- `artifacts/04-c4-container.mmd` and `artifacts/05-request-and-batch-flows.md` show MERLIN generated order ID/quantity/correlation flowing from the supported post-generation review-queue contract to the review adapter only after engine generation.
- A relative delta and current-run comparison are calculated only after that certified post-generation join. If the fields/correlation are not supported, no fourth interface or engine replica is introduced and automated influence remains disabled.
- Audit records no longer assume a pre-publication MERLIN quantity.

No unsupported pre-generation current-MERLIN-baseline dependency remains in artifacts 02-14.

### RT-03 - Major - Kill-switch correctness and executability

**Status: Closed at design level; measured proof remains an explicit release gate.**

Verified evidence:

- A durable quorum-backed on-premises latch holds signed `{epoch,state,run_id,scope_hash,manifest_hash,not_after}` with monotonic epochs and deny-default behavior for absent, unreachable, unsigned, expired, stale, or invalid state.
- The adapter has no standing writer credential. A broker issues a run/scope/manifest-bound permit valid for no more than 60 seconds.
- The supported activation/direct-write transaction must recheck the current epoch immediately before commit in the same transaction. Stale epoch or permit aborts and rolls back; inability to implement this without changing MERLIN blocks influence.
- A `DISABLING` epoch requires component acknowledgements within 30 seconds and rejection of new writes within 60 seconds.
- Independent SAP break-glass authority can revoke permit issuance and DB/network access and terminate active sessions without relying on the adapter or primary control service.
- A separate cleanup identity/runbook reconciles exact pre-activation ledger keys and removes only unconsumed AI-owned rows. Manual rows are excluded.
- `VERIFIED_SAFE` within five minutes requires no valid permits, sessions, transactions, or unexplained active rows plus complete acknowledgement, revoke, cleanup, and checksum evidence.
- Drills cover latch loss, stale epoch, check/write race, adapter failure/compromise, missing acknowledgement, independent revoke/session termination, and cleanup failure. Any failed drill leaves the system shadow-only.

The control is now transactionally guarded, out-of-band stoppable, deny-default, and testable. It is no longer merely a feature flag.

### RT-04 - Major - Publication arithmetic and atomicity

**Status: Closed.**

Recalculated values:

| Case | Calculation | Correct result | Artifact result |
|---|---:|---:|---:|
| 70,400-row raw direct transaction | `70,400 / 60` | 1,173.333... rows/s | Rounded up to 1,173.4 rows/s |
| 70,400 rows with 10 s overhead and 20% headroom | `70,400 / ((60 - 10) x 0.80)` | 1,760 rows/s | 1,760 rows/s |
| 5,000-row raw pilot transaction | `5,000 / 60` | 83.333... rows/s | Rounded up to 83.4 rows/s |
| 5,000 rows with 10 s overhead and 20% headroom | `5,000 / ((60 - 10) x 0.80)` | 125 rows/s | 125 rows/s |

The corrected cap is `min(policy ceiling, floor(R_cert x (L_cert - T_overhead_p99) x 0.80))`, with `L_cert <= 60 s`. The artifacts clearly distinguish the 15-minute publication-stage budget from the live-table transaction rate.

An existing non-live staging path may be used only when the supported contract certifies it and epoch-guarded activation completes within five seconds. The artifacts do not invent a new table, schema, pointer, or swap. Without certified staging, measured direct rate/overhead determines a lower cap or the system remains shadow-only.

### RT-05 - Major - Offline usefulness versus universal refusal

**Status: Closed.**

Verified acceptance contract:

- Signed-pack coverage: 100%.
- Offline p95: <=500 ms.
- Eligible non-safety answer coverage: >=90%.
- Eligible non-safety answer/citation accuracy: >=95%.
- Ineligible non-safety refusal precision and recall: each >=95%.
- Safety policy correctness: 100%.
- Eligible safety exact-display/citation correctness: 100%.
- Required safety-refusal precision and recall: 100%.
- Benchmarks label eligible-answer and required-refusal cases separately.
- Empty-pack and universal-refusal implementations explicitly fail.

The acceptance test can no longer pass through fast refusal of every query.

## Cross-file and arithmetic recheck

- The 27M normal forecast, 7.04M stocked candidate, 162M forecast-equivalent peak, and 42.24M optimization-equivalent peak populations remain consistent.
- The stage-precision values are 46.93 and 11.73 minutes; their full sum is 198.66, reported as 198.7. The resulting 41.34-minute sidecar headroom is reported as 41.3, so the displayed arithmetic is reproducible without a presentation-rounding discrepancy.
- The corrected transaction rates and cap formula are consistent in the architecture summary, flows, NFRs, capacity check, ADR, risk register, Variant B artifact, and consolidated pack.
- Kill-switch timings are consistently acknowledgement <=30 seconds, new-write rejection <=60 seconds, and verified safe <=5 minutes.
- The risk register still contains exactly five full risk rows, each with a measurable signal and numeric threshold.
- Markdown code-fence counts are balanced across artifacts 02-14.
- Stale searches returned no old 78.2 rows/s/313 rows/s capacity claim, no feature-flag-only kill switch, no pre-publication MERLIN comparison claim, no safety summarization permission, and no acceptance language that lets universal refusal pass.

## Former minor items

- **RT-06 is closed:** the current matrix has shared score anchors, formula, reproducible totals, trade-offs, and sensitivity checks.
- **RT-07 is closed:** the current register has a scored omitted-contender note, inclusion threshold, and tie-break rule while retaining exactly five full risks.

## Final verdict

# Superseded — see current final review

The Critical and Major correction gate has passed. The architecture artifacts may proceed to the presentation-coach phase. Keep every production claim conditional, and do not present unmeasured MERLIN/SAP behavior as proven. The two remaining minor findings and the resolution-log labels should be corrected if time permits, but they do not block workshop preparation.
