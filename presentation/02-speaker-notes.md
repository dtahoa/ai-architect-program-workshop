# NovaMart Group 4 - Speaker Notes

Presentation integrator: **ĐINH XUÂN DŨNG**

Speak to the decision and evidence; do not recap the shared scenario. Transitions are silent cues.

## Slide 1 - 45 seconds

**Keep MERLIN authoritative; let AI fail without stopping a truck.** We add intelligence beside MERLIN, never inside it. The sidecar runs statistical forecasts and deterministic, bounded replenishment rules, but MERLIN still creates every supplier order and alone sends fixed-schema EDI. If AI is unavailable, late, incomplete, uncertain, invalid, or deliberately disabled, no override appears and MERLIN's incumbent min/max path continues. That is our removal test: take away the sidecar and stores still receive morning deliveries. Today we authorize discovery, replay, and shadow only. Production influence waits for measured MERLIN timing, certified adapters, peak tests, observed cost, and funded operations.

That continuity is the architecture's primary acceptance test.

Transition: Those authority rules define where change is allowed.

## Slide 2 - 45 seconds

**Change only the edges: nightly batch, override table, and review queue.** We may build an external sidecar, monitoring, audit, and narrow certified adapters. We may not modify MERLIN's engine or authority, touch POS, alter supplier EDI, or move 04:00. The nightly extension supplies a dated read-only snapshot. The override table is read before order generation. The review queue operates after generation. Undocumented tables, scheduler hooks, manual rows, and direct supplier paths are unsafe. Inventory is only 78 percent accurate, so it is an uncertainty input with age, intervals, caps, and suppression, never real-time shelf truth.

These are constraints, not preferences we plan to renegotiate.

Transition: The container view follows those boundaries exactly.

## Slide 3 - 55 seconds

**The sidecar is advisory; every authoritative action stays inside MERLIN.** A dated snapshot enters validation for schema, freshness, completeness, checksum, and residency. Statistical ML produces demand quantiles and confidence. A deterministic optimizer applies whole cases, shelf life, inventory intervals, DC capacity, and category caps. Only the certified adapter can write bounded current-run rows to the supported override table. MERLIN reads them, runs its unmodified engine, and owns native review and EDI. The sidecar has no route to POS, suppliers, or the EDI connector. Current MERLIN quantity may return only after generation through certified queue correlation. Without that contract, comparison and automated influence remain disabled.

The dashed return is evidence, not a control dependency.

Transition: The next proof is the clock, not the model.

## Slide 4 - 55 seconds

**AI closes at 02:00 so MERLIN owns the final two hours.** Validation finishes by 22:20, forecasting by 00:20, and deterministic optimization by 01:20. No new retry starts after 01:30. At 01:45 every required partition, model, ruleset, constraint, checksum, and audit record must form one complete manifest. A short permit and same-transaction epoch check guard any certified write, and all AI writes close at 02:00. MERLIN then generates, reviews, and sends EDI before 04:00. Missing partitions, stale inputs, low confidence, expired permits, or failed commits become no override. Recovery at 03:30 stays closed. The 02:00 gate itself needs measured MERLIN P99 of at most 90 minutes or it moves earlier.

Late output is retained only for diagnosis and later learning.

Transition: Now test the design envelope honestly.

## Slide 5 - 55 seconds

**The value and capacity envelopes fit on paper; production waits for measured proof.** Success means out-of-stock falls from 7.2 to 3.0 percent, fresh waste from 4.8 to 3.0 percent, WAPE from 41 to at most 25 percent, and lookup time from about 50 to under 15 minutes per shift, without reducing 8,400 associates or sacrificing a 2.4 percent margin. Capacity uses 27 million forecasts and 7.04 million optimizations, with conservative 162 million and 42.24 million tests at 30,000 and 15,000 per second. The precise planning sum is 198.66 minutes, reported as 198.7, leaving 41.34, reported as 41.3. Cost is approximately 3.8 million dollars under 4 million. Three peak passes, bills, adapter proof, and Finance approval remain gates.

All cost allocations remain approximate until tagged bills exist.

Transition: The matrix explains why these choices fit the constraints.

## Slide 6 - 50 seconds

**Sidecar wins on supported isolation, not convenience.** With criteria totaling 100, the external sidecar scores 4.80 because it best protects legacy safety, fail-open continuity, supported interfaces, and rollback. Managed elastic batch scores 4.65 for deadline control and six-times scale. Hybrid hierarchical statistical ML scores 4.55 for quality, promotions, and uncertainty. We accept duplicated snapshots, reconciliation, SAP-certified adapter work, provider controls, and two-system operations. Sensitivity matters: shifting weight from WAPE to cost ties hybrid and global models at 4.45, so replay evidence decides the final statistical model. Scores select architecture direction; they never waive production gates.

No LLM option enters numeric forecasting or replenishment optimization.

Transition: The accepted trade-offs become measurable risks.

## Slide 7 - 50 seconds

**Every top risk has a tripwire that leaves MERLIN running.** Deadline and inventory each score 20, peak and model behavior scores 16, and legacy integrity plus safety score 15. Critical examples are any write at or after 02:00, inventory coverage below 85 percent, forecast throughput below 22,500 per second at peak, a stale-epoch commit or manual-row change, and any generated or unsupported safety display. The response is bypass, suppression, slice disablement, independent access revocation, or refusal and escalation; it is never a desperate retry toward 04:00. Residency, cost, review fatigue, and provider outage scored 10 or less and remain controlled NFRs.

Omitted contenders are promoted if later evidence changes their scores.

Transition: We therefore earn influence rather than assume it.

## Slide 8 - 55 seconds

**Earn influence in stages; disable it in minutes.** Discovery certifies timing, queue fields, transaction behavior, ownership, and cleanup. Replay tests quality, intervals, six-times load, cost, and failure behavior. Shadow runs for at least four representative weeks with zero MERLIN writes. Only after every gate passes may a pilot cover 20 stores, two low-risk non-fresh categories, no more than 5,000 rows within the measured adapter cap, and at most 100 reviews nightly. To disable, set a new epoch, stop permits, reject writes within 60 seconds, independently revoke SAP access if needed, clean exact AI-owned rows, and verify safe within five minutes. MERLIN continues throughout.

Re-enable requires clean shadow evidence, cause closure, and renewed approval.

## Rehearsal totals

- Exactly 8 scripts.
- Target speaking time: **410 seconds / 6 minutes 50 seconds**.
- Intended script length: **850-900 whitespace-delimited words**, verified by the structural checks.
