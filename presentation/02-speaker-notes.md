# NovaMart Group 4 - Speaker Notes

Owner: ĐINH XUÂN DŨNG (123015, 24R-Humana)

Delivery rule: Speak to the decision and evidence. Do not re-explain the NovaMart scenario. The notes below are the rehearsed script; the bold opening sentence on each slide should be delivered exactly.

## Slide 1 - 45 seconds

**Keep MERLIN in control; let AI fail without stopping a truck.** Our position is to add intelligence beside MERLIN, never inside it. An external asynchronous sidecar produces statistical forecasts and deterministic, bounded replenishment recommendations. MERLIN still generates every supplier order and remains the only system that sends EDI. If AI is absent, late, uncertain, invalid, or deliberately disabled, no current-run override appears and the incumbent min/max process continues. That is the removal test: take away the entire AI platform and trucks still receive orders. We are defending replay and shadow today. Production influence remains conditional on measured MERLIN, adapter, peak, cost, and operating evidence.

Transition: If authority stays with MERLIN, where are we allowed to change anything?

## Slide 2 - 45 seconds

**Change only the edges: three supported MERLIN extensions.** We can add the external sidecar, narrow adapters, a control plane, monitoring, and immutable audit. We cannot modify MERLIN's replenishment engine or authority, touch POS, change supplier EDI, or move 04:00. Every MERLIN crossing maps to one declared extension: the nightly batch supplies a dated snapshot, the override table is read before generation, and the review queue operates after generation. The scheduler, undocumented tables, and manual rows are unsafe to touch. So is treating inventory as shelf truth: accuracy is 78 percent, which makes uncertainty a core design input rather than a data-cleaning footnote.

Transition: Those boundaries now determine the full container flow.

## Slide 3 - 55 seconds

**The sidecar is advisory; every authoritative action stays inside MERLIN.** The sidecar ingests approved immutable snapshots, validates schema, freshness, completeness, residency, and checksums, then builds features including inventory intervals. Statistical ML returns demand quantiles and confidence. A deterministic optimizer applies whole cases, shelf life, lead time, DC capacity, inventory feasibility, and category caps. The sidecar cannot contact suppliers. Only a certified adapter can write bounded current-run rows to the supported override table. MERLIN then runs its unmodified engine, creates the authoritative order, and owns native review and EDI. A current MERLIN quantity is not available before publication; it may return only after generation through a certified queue join. No certified join means no current-run comparison and no automated influence.

Transition: The decisive operating constraint is the clock, not the model.

## Slide 4 - 60 seconds

**AI closes at 02:00 to protect the immovable 04:00 cut-off.** The sidecar gets a four-hour window. It validates the snapshot by 22:20, completes forecasting by 00:20, and deterministic optimization by 01:20. No new internal retry begins after 01:30. At 01:45, every required partition, model, ruleset, constraint, checksum, and audit record must form one complete signed manifest. Publication uses a short permit and a transaction-level epoch check, and every write closes at 02:00. MERLIN then owns two hours for generation, native review, and fixed-schema EDI. A missing partition, stale input, low confidence, bad checksum, expired permit, or failed transaction means publish nothing. If AI recovers at 03:30, it remains closed; MERLIN continues the current run.

Transition: The clock is protected by design, but do the scale and cost numbers fit?

## Slide 5 - 55 seconds

**The design fits the envelope on paper - production waits for measured proof.** We preserve both source populations: 27 million forecast outputs and 7.04 million stocked optimization candidates. Because the six-times denominator is ambiguous, the capacity test is intentionally conservative: 162 million forecast and 42.24 million optimization equivalents. At the target rates, the end-to-end sidecar path is 198.7 minutes inside 240, leaving 41.3 minutes. Annual operating cost targets 3.8 million dollars under the 4 million ceiling, or 0.0003856 dollars per nominal forecast when all AI costs are blended. These are targets, not benchmark results. Production requires three consecutive six-times runs, measured MERLIN P99 of at most 90 minutes for this close, a certified adapter, observed bills, and Finance approval.

Transition: Given those constraints, why select a sidecar rather than another pattern?

## Slide 6 - 50 seconds

**Sidecar won because isolation matters more than convenience.** Supported boundaries and replenishment continuity carry half the integration-pattern weight, so the external fail-open sidecar scores 4.80. Shadow scores 4.00, but shadow is a rollout state, not an architecture: it says nothing about deployment or interfaces. Event-driven integration would invent a legacy event contract; an API gateway would put availability on a synchronous path; an in-process plugin violates the unmodifiable, untested core. The ADR accepts real negatives: duplicated snapshots, possible divergence, certified adapter work, two-system traceability, and additional on-call cost. We accept those costs because they buy independent failure and rollback. The scores select direction; they do not waive release gates.

Transition: That trade-off only works if its risks have measurable stop conditions.

## Slide 7 - 45 seconds

**Every top risk has a threshold and a safe fallback.** Deadline and inventory each score 20, peak and model behavior scores 16, and adapter integrity plus safety score 15. A deadline reserve below 30 minutes, a published row without a calibrated interval, peak throughput below its floor, a stale-epoch commit, a changed manual row, or any generated safety statement is a critical stop. The response is never to keep retrying toward 04:00. We bypass the AI run, suppress the row, disable the affected slice, activate the kill path, or refuse and escalate the safety question. MERLIN and POS remain independent. The remaining review minors concern score sensitivity and why omitted contenders ranked below the top five, not the operating controls.

Transition: So the final decision is a staged permission model, not immediate automation.

## Slide 8 - 55 seconds

**Earn influence in stages; disable it in minutes.** Discovery first certifies timing, queue fields, transaction behavior, locking, ownership, cleanup, and named operators. Historical replay proves forecast quality, inventory intervals, six-times throughput, cost, and failure behavior. Production shadow then runs for at least four weeks with zero MERLIN writes and representative promotions. Only after every gate passes do we consider a pilot: 20 stores, two low-risk non-fresh categories, no more than the measured transaction cap, and at most 100 review exceptions per night. A new disable epoch must be acknowledged within 30 seconds, reject new writes within 60, and reach verified safe within five minutes using independent SAP access revocation and exact-row cleanup. Our final defense is simple: remove the sidecar and MERLIN still orders.

## Rehearsal totals

- Total target: **410 seconds / 6 minutes 50 seconds**
- Rehearsed script body: **923 words**; headings, metadata, and silent transition cues are excluded.
- Delivery target: approximately **130-135 spoken words per minute**, with a short pause after each bold opening sentence.
- Ownership is confirmed across the six required workshop artifacts using the supplied Group 4 roster.
