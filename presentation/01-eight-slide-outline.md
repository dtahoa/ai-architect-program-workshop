# NovaMart Group 4 - Eight-Slide Presentation Outline

Owner: ĐINH XUÂN DŨNG (123015, 24R-Humana)

Workshop position: Variant B - AI-Retrofit / Legacy

Communication job: By the end, the trainers should accept that the safest useful design is a measured, fail-open sidecar around MERLIN because it improves forecasting without moving order authority or placing AI on the delivery-critical path.

Design basis: No source template was supplied. Use the built-in Codex Grid visual system at 1280 x 720 with a white canvas, black type, pale-gray structure, and blue only for the selected path, deadline gates, or critical evidence. The layout numbers below are exact Codex Grid references for a later PPTX build.

## Slide 1 - Architecture thesis

- Objective: State the position immediately and make the removal test obvious.
- Takeaway headline: **Keep MERLIN in control; let AI fail without stopping a truck**
- Codex Grid: **Layout 01** (`codex-grid-layout-library#slide-01`, sparse stacked-text-flow). Use the main 80 px field for the headline and the lower field for the thesis proof.
- Visible slide copy (5 points maximum):
  1. External, asynchronous, fail-open AI sidecar
  2. Statistical forecast ML + deterministic replenishment rules
  3. MERLIN generates every order and alone sends supplier EDI
  4. No valid AI output = unchanged min/max replenishment
  5. Replay and shadow only until measured release gates pass
- Recommended visual: One dominant sentence, with a thin line ending in a small green MERLIN authority marker and an unconnected blue sidecar marker. No architecture diagram yet.
- Speaker script: "Our position is simple: add intelligence beside MERLIN, never inside it. The sidecar produces forecasts and bounded recommendations, but MERLIN remains the order generator and the only EDI sender. If AI is absent, late, uncertain, or disabled, no override appears and the incumbent min/max run continues. We are defending replay and shadow today; production influence is conditional on measured evidence."
- Target time: **45 seconds**
- Evidence: `artifacts/02-architecture-decision-summary.md` Sections 1-3; `artifacts/11-adr.md` Decision outcome.
- Transition: If MERLIN must remain in control, the next question is where change is actually permitted.

## Slide 2 - Legacy assessment

- Objective: Prove that the design respects the fixed legacy boundary and uses no hidden fourth interface.
- Takeaway headline: **Change only the edges: three supported MERLIN extensions**
- Codex Grid: **Layout 07** (`codex-grid-layout-library#slide-07`, three-column process layout). Retain three equal columns for changeable, fixed, and unsafe surfaces.
- Visible slide copy (4 points maximum):
  1. Can change: external sidecar, narrow adapters, control plane, audit
  2. Cannot change: MERLIN engine/authority, POS, EDI schema, 04:00
  3. Only crossings: nightly batch, pre-generation override table, post-generation review queue
  4. Unsafe: scheduler, undocumented tables, manual rows, point inventory as truth
- Recommended visual: Three columns labelled **Can change**, **Cannot change**, and **Unsafe to touch**. Place the three extension points as a single blue rule under all columns.
- Speaker script: "The safe unit of change is outside the core. MERLIN has exactly three supported extension points: the nightly batch, the override table before generation, and the review queue after generation. We do not modify the engine, POS, EDI, scheduler, or manual rows. Inventory is only 78% accurate, so a point stock value is also unsafe to treat as shelf truth."
- Target time: **45 seconds**
- Evidence: `artifacts/06-legacy-assessment.md` Sections 2-5; `artifacts/13-variant-b-artifact.md` Sections 1-4.
- Transition: Those boundaries determine the container architecture and every permitted data flow.

## Slide 3 - C4 context and container architecture

- Objective: Show the authority boundary, the advisory computation path, and the certified integration points in one readable picture.
- Takeaway headline: **The sidecar is advisory; every authoritative action stays inside MERLIN**
- Codex Grid: **Layout 08** (`codex-grid-layout-library#slide-08`, half text/half visual). Use the right media field for a simplified native-shape container diagram; use the left for four boundary invariants.
- Visible slide copy (4 points maximum):
  1. Nightly snapshots -> validation -> features -> forecast ML -> deterministic optimizer
  2. Only a certified override adapter can influence MERLIN
  3. MERLIN generates orders; the queue returns post-generation context only if certified
  4. Loyalty PII stays country-local; copilot is separate from POS and ordering
- Recommended visual: Two bordered trust zones. Left: AI sidecar pipeline with six boxes. Right: MERLIN with its three extension points, engine, review queue, and EDI. Use one one-way arrow into the override table and one post-generation arrow back from the review queue. Show suppliers only behind MERLIN.
- Speaker script: "The sidecar reads approved dated snapshots, validates them, builds uncertainty-aware features, runs statistical forecast ML, then applies deterministic case-pack, shelf-life, inventory-interval, DC, and category constraints. It has no route to suppliers. A certified adapter may write bounded rows to the supported override table; MERLIN then generates the authoritative order. Only after generation may the native queue return order ID, quantity, and correlation for review and audit."
- Target time: **55 seconds**
- Evidence: `artifacts/03-c4-context.mmd`; `artifacts/04-c4-container.mmd`; `reviews/03-red-team-verification.md` RT-02 closure.
- Transition: The most important operating proof is not the model - it is protection of the deadline.

## Slide 4 - End-to-end batch flow and deadline

- Objective: Demonstrate how the proposed clock gates protect 04:00 and convert every AI failure into bypass.
- Takeaway headline: **AI closes at 02:00 to protect the immovable 04:00 cut-off**
- Codex Grid: **Layout 24** (`codex-grid-layout-library#slide-24`, five-column Gantt). Map its five time columns to the five macro stages below; use the full-width lower rail for fail-open behavior.
- Visible slide copy (5 points maximum):
  1. 22:00-22:20 - validate immutable snapshot
  2. 22:20-00:20 - statistical forecasts
  3. 00:20-01:20 - deterministic optimization
  4. 01:20-02:00 - reconcile, sign manifest, guarded publication
  5. 02:00-04:00 - MERLIN generation, review, and EDI; failed AI means no override
- Recommended visual: A five-band Gantt with small gate markers at 01:30 retry stop, 01:45 complete-set gate, and 02:00 write close. Render point 5 as the full-width lower rail so it also communicates the fallback without adding a sixth message.
- Speaker script: "The sidecar has four hours, not six. We validate by 22:20, forecast by 00:20, optimize by 01:20, stop new retries at 01:30, require a complete signed manifest at 01:45, and close every write at 02:00. MERLIN owns the final two hours. Any missing partition, stale input, low confidence, expired permit, or failed commit becomes no override. Recovery at 03:30 stays closed."
- Target time: **60 seconds**
- Evidence: `artifacts/05-request-and-batch-flows.md` Section 2; `artifacts/08-capacity-and-cost-check.md` Sections 2-3.
- Transition: With the clock protected, the next challenge is whether the arithmetic and cost envelope are credible.

## Slide 5 - NFR and cost proof

- Objective: Give the trainers the minimum quantitative proof and distinguish targets from measured evidence.
- Takeaway headline: **The design fits the envelope on paper - production waits for measured proof**
- Codex Grid: **Layout 19** (`codex-grid-layout-library#slide-19`, three large-stat layout). Use three dominant figures and two short proof lines above them.
- Visible slide copy (5 points maximum):
  1. **27M** forecasts/night; **7.04M** stocked optimization candidates
  2. **162M / 42.24M** conservative 6x test populations
  3. **198.7 min of 240** target path; **41.3 min** sidecar headroom
  4. **$3.8M/year** target vs **$4M** ceiling; **$0.0003856** blended nominal cost
  5. Production gates: three 6x passes, MERLIN P99 <=90 min, certified adapter, Finance sign-off
- Recommended visual: Three stat blocks for 27M, 198.7/240, and $3.8M/$4M. A slim top line reconciles forecast versus stocked populations; a blue footer states **Targets, not benchmark results**.
- Speaker script: "We keep both source populations visible: 27 million forecast outputs and 7.04 million stocked order candidates. For the ambiguous six-times peak, we deliberately test 162 million forecast and 42.24 million optimization equivalents. The planned path totals 198.7 minutes inside the 240-minute sidecar wall. Annual run cost targets 3.8 million dollars, or 0.0003856 dollars per nominal forecast, but none of those figures are called proven until three peak runs, measured MERLIN P99, provider bills, and Finance approval pass."
- Target time: **55 seconds**
- Evidence: `artifacts/07-nfr-table.md` NFR-01 to NFR-04 and NFR-12; `artifacts/08-capacity-and-cost-check.md` Sections 1, 2.2, and 6.
- Transition: The numbers explain feasibility; the decision matrix explains why this pattern beats the alternatives.

## Slide 6 - Technology decisions and ADR

- Objective: Show that sidecar selection follows the hard constraints and name the accepted trade-off.
- Takeaway headline: **Sidecar won because isolation matters more than convenience**
- Codex Grid: **Layout 20** (`codex-grid-layout-library#slide-20`, chart plus three interpretation rails). Use a single-series horizontal bar chart for the five pattern scores and the right rails for decision, trade-off, and evidence boundary.
- Visible slide copy (5 points maximum):
  1. Sidecar **4.80**; shadow-as-architecture **4.00**
  2. Event-driven **2.65**; API gateway **2.15**; in-process plugin **1.65**
  3. ADR: use only nightly batch, override table, and post-generation review queue
  4. Accepted trade-off: snapshot duplication/divergence and two-system operations
  5. Scores choose direction; measured release gates still decide influence
- Recommended visual: Ranked horizontal bars, with sidecar in strong blue and all alternatives gray. Right-side rails: **Decision**, **Trade-off**, **Conditional evidence**.
- Speaker script: "The sidecar scores 4.80 because supported boundaries and failure isolation carry half the weight. Shadow scores well but is only a rollout state; it does not define deployment or interfaces. Event-driven and API patterns invent synchronous or unsupported legacy behavior, while an in-process plugin violates the core boundary. The ADR accepts snapshot duplication, reconciliation, and extra operations in exchange for independent failure and rollback. These scores are architecture judgments, not benchmarks, so release gates still govern production."
- Target time: **50 seconds**
- Evidence: `artifacts/10-technology-selection-matrix.md` Decision A; `artifacts/11-adr.md` Options, Decision, and Detailed consequences; `reviews/03-red-team-verification.md` Remaining minor items.
- Transition: That trade-off creates risks, so each one needs a threshold and a safe response.

## Slide 7 - Top five risks and controls

- Objective: Show that the team knows what can fail, how it is detected, and what happens next.
- Takeaway headline: **Every top risk has a threshold and a safe fallback**
- Codex Grid: **Layout 14** (`codex-grid-layout-library#slide-14`, intro plus data table). Use exactly five body rows and four concise columns: risk/score, control, critical trigger, safe response.
- Visible slide copy (5 points maximum):
  1. Deadline **20** - reserve <30 min or EDI >=04:00 -> bypass AI
  2. Inventory **20** - interval coverage <85% or missing interval -> suppress
  3. Peak/model **16** - WAPE >30% or forecast <22,500/s -> disable slice
  4. Legacy adapter **15** - stale epoch, manual-row change, MERLIN wait >0 -> kill/shadow
  5. Safety/legal **15** - any generated or unsupported safety output -> refuse/disable/escalate
- Recommended visual: Five-row evidence table with the risk score as a large number in column one. Use blue only for the safe-response column; use no heatmap because all five exceed tolerance for unbounded influence.
- Speaker script: "The register contains exactly five risks. Deadline and inventory score 20; peak behavior scores 16; adapter integrity and safety score 15. Each has a numeric tripwire and a safe response. Crucially, the response is never to push harder toward 04:00. We bypass, suppress, disable the slice, or activate the kill path while MERLIN continues. The two remaining review minors are scoring sensitivity and evidence for omitted contenders; neither changes these controls."
- Target time: **45 seconds**
- Evidence: `artifacts/12-risk-register.md`; `reviews/03-red-team-verification.md` Remaining minor items.
- Transition: The close is therefore not a promise of immediate automation; it is a controlled path to earn influence.

## Slide 8 - Rollout, kill switch, and final defense

- Objective: End with the operating decision, the scale-out gates, and a technically executable stop mechanism.
- Takeaway headline: **Earn influence in stages; disable it in minutes**
- Codex Grid: **Layout 18** (`codex-grid-layout-library#slide-18`, three-card milestone timeline). Use three main cards for shadow, pilot, and controlled influence; use the timeline labels and lower rail for kill/fail-open proof.
- Visible slide copy (5 points maximum):
  1. Discovery + replay: certify contracts, timing, 6x load, cost, and failure drills
  2. Shadow >=4 weeks: zero MERLIN writes; promotion evidence required
  3. Pilot: 20 stores, 2 low-risk categories, <=5,000 measured-cap rows, <=100 reviews/night
  4. Scale only after repeated quality, deadline, cost, queue, and owner gates
  5. Kill: acknowledge <=30 s, reject writes <=60 s, verify safe <=5 min; MERLIN continues
- Recommended visual: Three milestone cards connected by a thin timeline. Add a compact blue control strip under the cards: **deny-default epoch -> short permit -> independent SAP revoke -> exact-row cleanup**. End with the footer **Remove the sidecar and MERLIN still orders.**
- Speaker script: "We earn influence in stages. Discovery certifies the contracts and timing; replay proves quality, peak capacity, cost, and failure behavior; shadow runs for at least four weeks with zero writes. Only then do we consider 20 stores and two low-risk categories, bounded by measured transaction and review capacity. A new disable epoch must be acknowledged in 30 seconds, reject writes in 60, and reach verified safe in five minutes through independent SAP revocation and exact-row cleanup. Remove the sidecar, and MERLIN still orders."
- Target time: **55 seconds**
- Evidence: `artifacts/13-variant-b-artifact.md` Sections 7-11; `reviews/03-red-team-verification.md` RT-03 and RT-04 closure.

## Timing and narrative verification

- Slide count: **8 exactly**
- Planned speaking time: **410 seconds (6 minutes 50 seconds)**
- Per-slide times: `45 + 45 + 55 + 60 + 55 + 50 + 45 + 55 = 410 seconds`
- Visible-point count by slide: `5, 4, 4, 5, 5, 5, 5, 5`
- Narrative continuity: thesis -> permissible legacy boundary -> container flow -> deadline proof -> scale/cost proof -> choice/trade-off -> risk controls -> staged influence and fail-safe close.
- Reviewer correction coverage: deterministic non-generative safety output (RT-01); no pre-publication MERLIN baseline and certified post-generation join (RT-02); deny-default transactional kill path with independent revoke (RT-03); corrected measured transaction-cap arithmetic (RT-04); offline usefulness cannot pass by universal refusal (RT-05).
