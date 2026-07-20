# NovaMart Group 4 - Eight-Slide Presentation Outline

Presentation integrator: **ĐINH XUÂN DŨNG**

## Slide 1 - Architecture thesis

- **Objective:** State the decision immediately, without recapping the scenario.
- **Headline:** Keep MERLIN authoritative; let AI fail without stopping a truck.
- **Visible points (5):** external asynchronous sidecar; statistical forecast ML; deterministic quantities; MERLIN alone generates/sends orders; no valid AI output means incumbent min/max.
- **Visual:** One authority line: advisory sidecar beside MERLIN, with MERLIN continuing to EDI.
- **Notes pointer:** `02-speaker-notes.md`, Slide 1.
- **Target time:** 45 seconds.

## Slide 2 - Legacy boundaries

- **Objective:** Show exactly what may change and the three supported crossings.
- **Headline:** Change only the edges: nightly batch, override table, review queue.
- **Visible points (4):** can change; cannot change; unsafe to touch; supported crossings only.
- **Visual:** Three-column boundary assessment with a lower extension-point rail.
- **Notes pointer:** `02-speaker-notes.md`, Slide 2.
- **Target time:** 45 seconds.

## Slide 3 - C4 architecture and authority flow

- **Objective:** Make data and authority direction unambiguous.
- **Headline:** The sidecar is advisory; every authoritative action stays inside MERLIN.
- **Visible points (4):** snapshot -> validation -> forecast -> deterministic optimization; certified adapter -> override table; MERLIN engine -> native review/EDI; post-generation correlation returns to adapter only if certified.
- **Visual:** Two trust zones with arrowheads: sidecar pipeline into the override table, MERLIN engine to EDI and review, dashed review-to-adapter return; POS isolated.
- **Notes pointer:** `02-speaker-notes.md`, Slide 3.
- **Target time:** 55 seconds.

## Slide 4 - Batch clock and failure path

- **Objective:** Prove how proposed gates protect 04:00.
- **Headline:** AI closes at 02:00 so MERLIN owns the final two hours.
- **Visible points (5):** validate 22:00-22:20; forecast 22:20-00:20; optimize 00:20-01:20; retry stop 01:30 and complete-set gate 01:45; close 02:00 then MERLIN to 04:00.
- **Visual:** Timeline with a full-width fail-open rail: any late, incomplete, invalid, or disabled run publishes nothing.
- **Notes pointer:** `02-speaker-notes.md`, Slide 4.
- **Target time:** 55 seconds.

## Slide 5 - Quantified NFR proof

- **Objective:** Tie mandatory business outcomes to the calculated capacity and cost envelope while separating targets from measured proof.
- **Headline:** The value and capacity envelopes fit on paper; production waits for measured proof.
- **Visible points (5):** OOS 7.2% -> 3.0% and fresh waste 4.8% -> 3.0%; WAPE 41% -> <=25% and lookup about 50 -> <15 min/shift; retain 8,400 associates with no AI-driven reduction and protect 2.4% margin; 27M/7.04M normal, 162M/42.24M 6x, 30k/s/15k/s; 198.7/240 min with 41.3 min calculated headroom and approximately $3.8M under $4M.
- **Visual:** Business-outcome strip above three proof blocks plus a banner: **Targets, not benchmark results**.
- **Notes pointer:** `02-speaker-notes.md`, Slide 5.
- **Target time:** 55 seconds.

## Slide 6 - Technology decisions and ADR

- **Objective:** Defend the selected pattern and a consequential sensitivity.
- **Headline:** Sidecar wins on supported isolation, not convenience.
- **Visible points (5):** sidecar 4.80; managed batch 4.65; hybrid model 4.55; accepted two-system trade-off; model sensitivity can tie hybrid/global at 4.45.
- **Visual:** Three compact winner cards with score anchors and a trade-off rail.
- **Notes pointer:** `02-speaker-notes.md`, Slide 6.
- **Target time:** 50 seconds.

## Slide 7 - Exactly five risks

- **Objective:** Show scored risk selection, numeric triggers, and safe responses.
- **Headline:** Every top risk has a tripwire that leaves MERLIN running.
- **Visible points (5):** deadline 20; inventory 20; peak/model 16; legacy integrity 15; safety/legal 15.
- **Visual:** Five-row table: risk/score, critical trigger, response, owner. Footer: omitted contenders score <=10.
- **Notes pointer:** `02-speaker-notes.md`, Slide 7.
- **Target time:** 50 seconds.

## Slide 8 - Rollout and executable kill switch

- **Objective:** End with conditional permission and a tested stop path.
- **Headline:** Earn influence in stages; disable it in minutes.
- **Visible points (5):** discovery/replay; shadow >=4 weeks and zero writes; pilot 20 stores/2 categories/<=5,000 measured-cap rows/<=100 reviews; scale only after repeated gates; disable <=30 s acknowledgement/<=60 s write rejection/<=5 min safe.
- **Visual:** Three-stage rollout plus control strip: new epoch -> stop permits -> independent SAP revoke -> exact-row cleanup.
- **Notes pointer:** `02-speaker-notes.md`, Slide 8.
- **Target time:** 55 seconds.

## Presentation verification

- Exactly **8 slides**.
- Planned time: `45 +45 +55 +55 +55 +50 +50 +55 = 410 seconds` (**6:50**).
- Every slide has <=5 visible points.
- Narrative: thesis -> boundary -> architecture -> clock -> numbers -> choices -> risks -> controlled rollout.
