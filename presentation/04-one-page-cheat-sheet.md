# NovaMart Group 4 - One-Page Presentation Cheat Sheet

Owner: ĐINH XUÂN DŨNG (123015, 24R-Humana)

Slot: Group 4, Variant B | 7 minutes presentation | 4 minutes Q&A | Artifact owners: [Names to be assigned before submission]

## Opening and close

Opening: **Keep MERLIN in control; let AI fail without stopping a truck.**

Close: **Remove the sidecar and MERLIN still orders.**

## Architecture in 30 seconds

- External asynchronous sidecar reads approved nightly snapshots only.
- Statistical ML forecasts demand; deterministic rules/optimization create bounded quantities.
- MERLIN remains the only order generator, review authority, and supplier-EDI sender.
- Exactly three MERLIN crossings: nightly batch, pre-generation override table, post-generation review queue.
- No current MERLIN quantity is assumed before publication. Comparison happens only after a certified queue join; otherwise influence stays disabled.
- POS is untouched. Member-level loyalty stays country-local. Inventory is an interval, never shelf truth.

## Numbers worth memorizing

| Proof | Number | Meaning |
|---|---:|---|
| Forecast population | 27M/night | Conservative capacity and cost baseline |
| Stocked candidates | 7.04M/night | Replenishment optimization population |
| 6x test | 162M / 42.24M | Forecast / optimization equivalents |
| Target path | 198.7 min of 240 | 41.3 min sidecar headroom; not benchmark evidence |
| Gates | 01:30 / 01:45 / 02:00 | Retry stop / complete manifest / write close |
| MERLIN proof | P99 <=90 min | Leaves a further 30-minute reserve before 04:00 |
| Throughput | 30,000/s / 15,000/s | 6x forecast / optimizer acceptance targets |
| Cost | $3.8M vs $4M | Annual target / hard ceiling |
| Unit cost | $0.0003856 | Blended total AI cost per nominal forecast |
| Pilot | 20 / 2 / <=5,000 / <=100 | Stores / categories / measured-cap rows / reviews per night |
| Kill path | 30 s / 60 s / 5 min | Acknowledge / reject new writes / verified safe |

## Failure answers

- AI down, late, incomplete, low confidence, stale, or untraceable: publish nothing; MERLIN runs.
- AI recovers at 03:30: remain closed; no retry, rerun, override, or sidecar EDI.
- Inventory interval missing or uncalibrated: suppress the row; never query POS or infer shelf truth.
- Adapter stale epoch, expired permit, lock, or commit failure: atomic abort/rollback, independent SAP revoke/session termination, exact-row cleanup.
- Review capacity exhausted: excess uses MERLIN and does not accumulate.
- Safety evidence missing, stale, conflicting, inexact, unsigned, or uncited: deterministic refusal/hold/check instruction and human escalation.

## Red-team corrections to mention if challenged

- RT-01: Safety output is exact approved field/passage + fixed locale template/citation. The LLM never composes, paraphrases, translates, or summarizes displayed safety text.
- RT-02: No pre-publication MERLIN baseline. Generated quantity/order ID joins only after generation through a certified review-queue contract.
- RT-03: Kill switch is a deny-default monotonic epoch, short transaction permit, same-transaction recheck, independent SAP revoke, and separate cleanup identity.
- RT-04: 70,400 rows inside 60 seconds needs 1,173.4 rows/s raw; with 10 seconds overhead and 20% headroom, 1,760 rows/s. Actual cap is measured, never assumed.
- RT-05: Offline usefulness requires >=90% eligible non-safety answer coverage and >=95% answer/citation accuracy; all safety metrics are 100%. Universal refusal fails.

## Answer pattern

Say: **status -> mechanism -> number -> fallback -> evidence**.

Example: "Not yet proven. The 02:00 close preserves two hours for MERLIN; production requires MERLIN P99 <=90 minutes and three 6x passes. If either fails, the sidecar stays shadow and MERLIN orders normally."

Never say: "The model will handle it", "autoscaling makes it safe", "the LLM only advises", "we can catch up after 03:30", or "the $241M opportunity proves ROI".

## Executive summary

1. **Architecture thesis:** Add forecast intelligence beside MERLIN, never inside it; no valid AI output means unchanged replenishment.
2. **Selected integration pattern:** External asynchronous fail-open sidecar using only nightly batch, override table, and post-generation review queue; shadow-assist is the rollout mode.
3. **Most consequential trade-off:** Accept snapshot duplication, divergence, certified adapter work, and two-system operations in exchange for isolation, independent scale, and rollback.
4. **How 04:00 is protected:** Stop retries at 01:30, require a complete manifest at 01:45, close all AI writes at 02:00, and reserve MERLIN P99 plus 30 minutes before 04:00.
5. **How AI failure is isolated:** MERLIN never waits; a deny-default epoch, <=60-second permit, same-transaction guard, independent SAP revoke, and exact-row cleanup prevent continued publication.
6. **Why the design is affordable:** The annual target is $3.8M under the $4M ceiling, with a blended nominal unit cost of $0.0003856; production still requires quotes, bills, staffing, and Finance approval.
7. **Three hardest trainer questions:** Answer the three below without overstating evidence.

- **Hardest 1 - Is 04:00 actually proven?** No. It is arithmetically feasible at 198.7 minutes, but production waits for three 6x runs, MERLIN P99 <=90 minutes for the 02:00 gate, and certified adapter timing. Failure means shadow-only.
- **Hardest 2 - Can the kill switch stop a stale or compromised adapter?** By design, yes: no standing writer, a signed monotonic epoch checked in the same transaction, <=60-second permit, independent SAP credential/network revoke and session termination, and separate exact-row cleanup. A failed drill blocks influence.
- **Hardest 3 - Can the offline safety copilot be useful without generating answers or refusing everything?** Yes, only if signed packs return correct cited non-safety answers for >=90% of eligible cases at >=95% accuracy, while safety uses exact deterministic text/citations with 100% policy/display/refusal correctness; missing evidence refuses and universal refusal fails.
