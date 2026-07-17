# NovaMart Group 4 - Trainer Q&A

Owner: ĐINH XUÂN DŨNG (123015, 24R-Humana)

Answer discipline: Lead with `yes`, `no`, or `not yet proven`; give one mechanism, one number, one fallback, and the evidence. Never turn a planning target into measured production proof.

## 1. Can all forecasts and orders really complete by 04:00?

- Strong answer: **Not yet proven, but the design is testable and fail-open.** The conservative 6x target path is 198.7 minutes inside a four-hour sidecar wall, leaving 41.3 minutes. AI writes close at 02:00, preserving two hours for MERLIN. Production influence requires measured MERLIN P99 <=90 minutes for that close, a further 30-minute reserve, three consecutive 6x passes, and certified adapter timing. Any miss publishes nothing and MERLIN runs.
- Exact artifact evidence: `artifacts/08-capacity-and-cost-check.md` Sections 2.1-2.2 and 10; `artifacts/07-nfr-table.md` NFR-01/NFR-02 and Section 4 gate 1.
- Weak answer to avoid: "Yes, our estimates show it fits."
- Likely follow-up: What changes if measured MERLIN P99 is 105 minutes?

## 2. What happens at six-times Lunar New Year peak?

- Strong answer: We conservatively test 162M forecast equivalents and 42.24M optimization equivalents in the same fixed wall, even though demand growth may not multiply key cardinality. Targets are >=30,000 forecast/s and >=15,000 optimizer/s with at least 25% headroom and three consecutive passing nights. Missing a stage floor returns the scope to shadow; MERLIN continues.
- Exact artifact evidence: `artifacts/07-nfr-table.md` VA-02, NFR-03/NFR-04; `artifacts/08-capacity-and-cost-check.md` Sections 1 and 2.2.
- Weak answer to avoid: "Autoscaling handles peak."
- Likely follow-up: Why is the test 6x records rather than only 6x demand values?

## 3. What happens if the AI platform becomes available again at 03:30?

- Strong answer: Nothing is published that night. The adapter hard-closes at 02:00, so recovery at 03:30 cannot retry, rerun, write an override, or send EDI. MERLIN's already-running authoritative process continues. The recovered sidecar may retain diagnostics and prepare for the next approved run.
- Exact artifact evidence: `artifacts/05-request-and-batch-flows.md` Section 3, final state-transition row; `artifacts/11-adr.md` Failure behavior, "Clock reaches 02:00" row.
- Weak answer to avoid: "We catch up if there is still time."
- Likely follow-up: Why not allow a small late batch before 04:00?

## 4. Does the design depend on real-time or accurate inventory?

- Strong answer: No. The 78% figure means about 1.5488M of 7.04M stocked records may be wrong. Every candidate carries age and an inventory interval, must remain feasible across the approved interval, and is subject to whole-case, shelf-life, DC, and category absolute caps. Missing or uncalibrated intervals suppress the row. The sidecar never calls POS.
- Exact artifact evidence: `artifacts/07-nfr-table.md` NFR-13/NFR-14; `artifacts/12-risk-register.md` R-02.
- Weak answer to avoid: "The model will clean the inventory data."
- Likely follow-up: How do you validate the interval is calibrated?

## 5. Can an AI error stop stores from receiving deliveries?

- Strong answer: The intended behavior is no, but we keep the claim conditional until adapter drills pass. MERLIN never waits for AI; no override is normal. The remaining hazard is an adapter lock or bad transaction, so influence requires a same-transaction epoch guard, certified lock/rate behavior, atomic rollback, independent credential/session revocation, and proof that MERLIN wait is exactly zero. Failure means permanent shadow.
- Exact artifact evidence: `artifacts/07-nfr-table.md` NFR-11 and Section 4 gates 3-4; `reviews/03-red-team-verification.md` RT-03/RT-04 closure.
- Weak answer to avoid: "No, because the sidecar is separate."
- Likely follow-up: What if the adapter is compromised after it cached `ENABLED`?

## 6. Have you modified MERLIN, POS, or supplier EDI indirectly?

- Strong answer: No core or revenue-path change is part of the design. MERLIN crossings are limited to its nightly batch, pre-generation override table, and post-generation review queue. POS has zero AI calls, credentials, agents, or dependencies. MERLIN alone sends the unchanged EDI schema before 04:00. If a supported contract cannot supply a needed field, we do not invent a fourth hook; influence stays disabled.
- Exact artifact evidence: `artifacts/13-variant-b-artifact.md` Sections 3 and 5; `artifacts/04-c4-container.mmd` MERLIN and POS boundaries.
- Weak answer to avoid: "Only minor integration changes are needed."
- Likely follow-up: Is an epoch guard itself a new MERLIN extension?

## 7. Why is a sidecar genuinely the right pattern?

- Strong answer: It scored 4.80/5 because supported-boundary fit and replenishment continuity each carry 25% of the weight. It fits nightly snapshots, scales and releases independently, runs in shadow, and can be removed without affecting MERLIN. The accepted cost is snapshot duplication, reconciliation, a sensitive adapter, and two-system operations. Suitability remains conditional on the certified contract and measured deadline.
- Exact artifact evidence: `artifacts/10-technology-selection-matrix.md` Decision A; `artifacts/11-adr.md` Options considered and Detailed consequences.
- Weak answer to avoid: "Sidecars are standard for legacy modernization."
- Likely follow-up: Why not run all compute on-premises beside MERLIN?

## 8. Is shadow-assist the architecture or only a rollout mode?

- Strong answer: It is a rollout mode of the external sidecar. The architecture defines deployment, trust boundaries, and the three supported interfaces. Shadow uses the same production-like calculation path but disables the override adapter and writes zero MERLIN rows. It therefore proves timing, quality, cost, and controls without claiming influence.
- Exact artifact evidence: `artifacts/02-architecture-decision-summary.md` Sections 1 and 5; `artifacts/13-variant-b-artifact.md` Sections 5 and 9.
- Weak answer to avoid: "We selected a hybrid sidecar-shadow architecture."
- Likely follow-up: What evidence is impossible to collect in pure shadow?

## 9. Is the kill switch technically executable within minutes?

- Strong answer: It is executable by design but still requires a measured drill. A durable quorum latch holds a signed monotonic epoch; the adapter has no standing writer and receives a run/scope permit valid <=60 seconds. The current epoch is rechecked inside the same transaction before commit. `DISABLING` needs acknowledgements <=30 seconds, rejects new writes <=60 seconds, and reaches verified safe <=5 minutes. Independent SAP authority revokes DB/network access and terminates sessions; a separate identity cleans exact unconsumed AI rows.
- Exact artifact evidence: `artifacts/13-variant-b-artifact.md` Section 7; `reviews/03-red-team-verification.md` RT-03 closure.
- Weak answer to avoid: "Operations flips a feature flag."
- Likely follow-up: What happens if the control service itself is down?

## 10. Will category managers be overloaded?

- Strong answer: AI is not allowed to create an unbounded approval dependency. Initial admission is <=100 AI exceptions/night across 20 stores and two low-risk categories. The long-term cap is the smaller of risk-ranked demand and measured human capacity. Excess does not queue or accumulate; it uses MERLIN. Medium-risk influence remains disabled until native timeout/default behavior is proved non-blocking.
- Exact artifact evidence: `artifacts/05-request-and-batch-flows.md` Section 4; `artifacts/08-capacity-and-cost-check.md` Section 4.
- Weak answer to avoid: "Managers review only the important cases."
- Likely follow-up: What metric tells you 100 is too high?

## 11. Are you using an LLM for any non-language decision?

- Strong answer: No. Forecasting uses statistical ML; replenishment uses deterministic constraints and optimization; safety display uses a deterministic renderer. An LLM may classify or translate input, extract entities, or assist with non-safety language only. It has no route to forecast numbers, order quantities, prices, the override adapter, or displayed safety text.
- Exact artifact evidence: `artifacts/07-nfr-table.md` NFR-19; `artifacts/09-safety-and-governance.md` Sections 2 and 9.
- Weak answer to avoid: "The LLM only recommends; humans decide."
- Likely follow-up: Why keep any language model in the copilot pilot?

## 12. Is the cost calculation complete and affordable?

- Strong answer: It is internally reconciled but not procurement-proven. Eight operating items total $3.31242M; $487,580 contingency brings the target to $3.8M, leaving $200,000 under the $4M ceiling. That is $0.0003856 per nominal forecast when all AI costs are blended. Provider quotes, loaded staffing, bills, and Finance's treatment of the separate $3.2M delivery ceiling are still gates. If $3.2M includes first-year run cost, the scope fails.
- Exact artifact evidence: `artifacts/08-capacity-and-cost-check.md` Sections 6-7 and 11; `artifacts/07-nfr-table.md` NFR-12.
- Weak answer to avoid: "It is well below the $241M opportunity."
- Likely follow-up: Which cost would you cut first if the projection exceeds $4M?

## 13. Are data-residency boundaries actually visible and enforceable?

- Strong answer: The context and container views show a country-local loyalty boundary, and Release 1 exports zero member-level loyalty records or features to the regional sidecar. Schema classification, DLP, and egress denial target zero cross-border personal bytes. However, the regulated market, field list, derived-feature treatment, approved region, and support-access policy remain Legal release gates.
- Exact artifact evidence: `artifacts/03-c4-context.mmd` COUNTRY boundary; `artifacts/07-nfr-table.md` NFR-17; `artifacts/09-safety-and-governance.md` Section 8.
- Weak answer to avoid: "We anonymize data before moving it."
- Likely follow-up: Are pseudonymous derived features personal data?

## 14. Can the copilot ever generate or paraphrase an allergen answer?

- Strong answer: No. Safety domains render an exact approved structured field or passage for the exact SKU, market, locale, and version, plus fixed pre-approved wording and a visible citation. The LLM may process the question, but it cannot compose, paraphrase, translate, summarize, or reorder the displayed safety output. Missing, stale, conflicting, inexact, unsigned, or uncited evidence refuses and escalates.
- Exact artifact evidence: `artifacts/09-safety-and-governance.md` Sections 2, 4, and 5; `reviews/03-red-team-verification.md` RT-01 closure.
- Weak answer to avoid: "The model summarizes only retrieved approved documents."
- Likely follow-up: How do you support three languages without model translation of the answer?

## 15. Does every top risk have a measurable signal and threshold?

- Strong answer: Yes. Each of the exactly five rows has a signal, warning threshold, critical threshold, owner placeholder, mitigation, and contingency. Examples are reserve <30 minutes, interval coverage <85%, forecast throughput <22,500/s, any stale-epoch commit, and any generated safety output. The remaining minor is selection traceability: cost, residency, and review fatigue were not separately scored as omitted contenders.
- Exact artifact evidence: `artifacts/12-risk-register.md` R-01 to R-05; `reviews/03-red-team-verification.md` Remaining minor items.
- Weak answer to avoid: "We monitor all risks on dashboards."
- Likely follow-up: Why is residency not in the top five?

## 16. Does the technology matrix show real trade-offs rather than forced weights?

- Strong answer: The weights were set from hard constraints before scoring: supported boundaries, continuity, deadline, safety, and cost dominate. The matrix names negative consequences for each selection. We also acknowledge the remaining minor: criterion-specific 1/3/5 anchors and sensitivity evidence are not yet shown, so scores guide direction rather than prove an objective winner.
- Exact artifact evidence: `artifacts/10-technology-selection-matrix.md` Scoring method and Decisions A-D; `reviews/03-red-team-verification.md` Remaining minor item 1.
- Weak answer to avoid: "The highest score proves the sidecar is best."
- Likely follow-up: Which one-point score change could reverse a close decision?

## 17. Does the ADR document negative consequences honestly?

- Strong answer: Yes. It accepts snapshot drift, adapter locking and ownership risk, two-system audit and on-call complexity, a shorter four-hour AI window, unrealized value from bounded caps, unchanged 78% inventory weakness, and an unproven $3.8M annual plan. Rollback is correspondingly conservative: disable publication, exact-clean unconsumed AI rows, preserve manual rows, and return through shadow.
- Exact artifact evidence: `artifacts/11-adr.md` Detailed consequences and Rollback strategy.
- Weak answer to avoid: "The only downside is more integration work."
- Likely follow-up: Which negative consequence is most likely to force permanent shadow?

## 18. Is any component present only to make the design look sophisticated?

- Strong answer: The ordering core maps directly to required controls: validation, forecast ML, deterministic optimizer, registry, audit, monitoring, transaction control, and narrow adapters. The optional language service is deliberately constrained and may be removed if non-safety lookup value does not justify it. Markdown and substitutions are deferred because supported interfaces and acceptance criteria are missing.
- Exact artifact evidence: `artifacts/02-architecture-decision-summary.md` Sections 3-4; `artifacts/10-technology-selection-matrix.md` Decision D trade-off.
- Weak answer to avoid: "Every component is industry best practice."
- Likely follow-up: What is the minimum deployable architecture?

## 19. Could a smaller design satisfy the same NFRs?

- Strong answer: Yes. The minimum safe ordering release is the batch sidecar, on-premises adapter/control, monitoring, and immutable audit, with the copilot limited to deterministic signed-pack retrieval or deferred. Fresh markdown and online substitution remain analysis-only or absent. Simplicity is preserved by avoiding streaming, a real-time API, a new order master, member-level loyalty, and any POS or EDI integration.
- Exact artifact evidence: `artifacts/02-architecture-decision-summary.md` Section 3; `artifacts/07-nfr-table.md` NFR-08/NFR-09; `artifacts/13-variant-b-artifact.md` Section 12.
- Weak answer to avoid: "No, all six requested capabilities are mandatory."
- Likely follow-up: Why not defer the copilot entirely?

## 20. Can the group defend each consequential decision in under one minute?

- Strong answer: Yes, using the same pattern: decision, three proof numbers, accepted trade-off, and fallback. For the sidecar: score 4.80, 02:00 close, 30-second/60-second/5-minute kill targets; trade-off is snapshot divergence and two-system operations; fallback is unchanged MERLIN. For safety: zero generated output, 100% safety correctness, 4-hour revocation; fallback is refusal and escalation.
- Exact artifact evidence: `presentation/02-speaker-notes.md` Slides 4, 6, and 8; `artifacts/14-final-artifact-pack.md` Final consistency statement.
- Weak answer to avoid: "The detailed artifacts contain the answer."
- Likely follow-up: Give the 30-second version of the ADR now.
