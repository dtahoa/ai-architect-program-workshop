# Final Deliverable Review - PPTX and PDF

Review date: 2026-07-17

Reviewer role: Final workshop deliverable reviewer

## Scope

Reviewed the actual deliverables:

- `output/slides/Group4_VariantB_End_to_End_AI_System_Design.pptx`
- `output/pdf/Group4_VariantB_End_to_End_AI_System_Design.pdf`

Compared them with:

- `docs/workshop-1-ai-system-design.pdf`, including Sections 5 and 6;
- the attached orchestration request and `docs/request.md`;
- `artifacts/01-requirement-baseline.md` through `artifacts/14-final-artifact-pack.md`;
- `reviews/01-red-team-review.md` through `reviews/03-red-team-verification.md`; and
- `presentation/01-eight-slide-outline.md` through `presentation/04-one-page-cheat-sheet.md`.

## Verification performed

- Parsed the PPTX package: 8 slides, 8 speaker-note pages, and slide size `12,192,000 x 6,858,000` EMU, which is exactly 16:9.
- Rendered all 8 PPTX slides with the bundled artifact-tool renderer at 1920 x 1080 and inspected every slide individually at full resolution.
- Ran the presentation overflow check. Result: `Test passed. No overflow detected.`
- Parsed the PDF with Poppler: 8 pages, each 960 x 540 points (16:9).
- Rendered all 8 PDF pages with Poppler at 150 DPI and inspected every page individually at full resolution.
- Compared PPTX and PDF rasters after equalizing dimensions. Mean absolute pixel differences were 2.553-5.395 on a 0-255 scale, consistent with anti-aliasing differences; content and layout otherwise match. The PDF therefore carries the same finding reported below.
- Parsed visible PPTX text and all speaker notes. The notes match the approved eight-slide narrative and contain the corrected red-team positions.
- Rechecked the presentation plan: `45 + 45 + 55 + 60 + 55 + 50 + 45 + 55 = 410 seconds`, within the required 390-420-second range.
- Confirmed 20 trainer Q&A entries; each contains a strong answer, artifact evidence, a weak answer to avoid, and a likely follow-up. The one-page cheat sheet covers the thesis, proof numbers, failures, red-team corrections, and hardest questions.

## Findings

### Critical

None.

### Major

#### FD-01 - Slide 3 reverses the architecture's data and authority flows

Files/pages:

- `output/slides/Group4_VariantB_End_to_End_AI_System_Design.pptx`, slide 3
- `output/pdf/Group4_VariantB_End_to_End_AI_System_Design.pdf`, page 3

Evidence:

- The rendered AI-sidecar arrows point upward: `Certified adapters -> Deterministic optimizer -> Statistical forecast ML -> Validate + features -> Nightly snapshot`. The intended flow in the slide's own bullet, speaker note, `artifacts/04-c4-container.mmd`, and `artifacts/05-request-and-batch-flows.md` is the reverse.
- The rendered MERLIN arrows point upward: `EDI -> Review queue -> MERLIN engine -> Override table`. The authoritative flow is `Override table -> MERLIN engine`, followed by normal generated orders to EDI and a separate exception/review path.
- The solid blue override connector points from the override table back into the adapter, while the dashed post-generation connector points from the adapter into the review queue. Both are opposite to the documented contracts: adapter writes to the override table; the supported review queue returns generated order ID/quantity/correlation to the adapter after generation.

Impact:

The slide is the audience's primary C4/container visual. It visually claims the opposite of the approved architecture, obscures the fail-open authority boundary, and can be read as EDI driving order generation. This is a functional presentation-correctness defect even though the bullets and speaker notes are correct.

Required correction:

1. Reverse the sidecar arrows to `Nightly snapshot -> Validate + features -> Statistical forecast ML -> Deterministic optimizer -> Certified adapter`.
2. Point the certified adapter into the supported override table.
3. Point the override table into the MERLIN engine.
4. Show the authoritative normal path from the MERLIN engine to EDI, with the review queue as the post-generation exception/review path rather than an AI dependency.
5. Point the dashed post-generation return from the review queue to the certified adapter.
6. Preserve the existing prohibitions: no AI route to POS, suppliers, or the EDI connector.
7. Regenerate both PPTX and PDF, rerun the overflow check, and visually inspect slide/page 3 at full resolution.

Developer fix required: **Yes.**

### Minor

None requiring deliverable changes.

The owner placeholders are consistently present as requested. Before submission, the team must replace them with real names because the source guideline requires one named owner per artifact. This is a planned pre-submission action, not a defect against the current placeholder requirement.

## Passed content/readiness checks

- Exactly 8 slides; no scenario re-explanation; the opening states the architecture position immediately.
- All six required artifacts are represented: C4/context and narrated flow, quantified NFRs, technology matrix, ADR/trade-off, exactly five risks, and Variant B legacy/kill-switch content.
- MERLIN remains the authoritative order generator and sole supplier-EDI sender; POS is isolated; no direct AI-to-POS, AI-to-supplier, or AI-to-EDI path is shown.
- Fail-open behavior is explicit: missing, late, uncertain, stale, invalid, disabled, or failed AI output produces no override and MERLIN continues.
- Inventory is treated as uncertain through unsafe-point-inventory language, calibrated intervals, caps, suppression, and thresholds.
- The 04:00 wall is protected by the 01:30 retry stop, 01:45 complete-set gate, 02:00 write close, and the 02:00-04:00 MERLIN reserve.
- Arithmetic is consistent: 27M/7.04M populations, 162M/42.24M conservative peak, 198.7/240 minutes and 41.3 minutes headroom, $3.8M/$4M, and $0.0003856 per nominal forecast.
- Corrected publication arithmetic is visible and correct: 70,400/60 = 1,173.4 rows/s raw; 70,400/(50 x 0.80) = 1,760 rows/s with overhead and headroom.
- Kill-switch controls are technically executable by design: deny-default epoch, short permit, same-transaction guard in notes, independent SAP revoke, exact-row cleanup, and 30-second/60-second/5-minute targets.
- Safety handling is deterministic and evidence-bound; the notes/Q&A prohibit generated or paraphrased displayed safety answers, require exact approved evidence/citation, and make universal refusal fail acceptance.
- The risk slide contains exactly five risks with numeric scores, triggers, and safe responses.
- The matrix and ADR show real trade-offs and correctly distinguish shadow as a rollout state. Unmeasured claims remain release-gated.
- Visible content follows the approved maximum-five-point plan. Titles are 36 pt (54 pt on slide 1), main body text is approximately 16.1-18 pt, and all inspected content is legible with no unintended overlap, clipping, or wrapping.

## Final verdict

# Not ready

Reason: FD-01 is a Major correctness defect in the primary architecture visual and is duplicated in both deliverable formats. No Critical issue was found, and all other requested functional, safety, timing, arithmetic, content, layout, Q&A, and cheat-sheet checks pass. After the slide 3 connectors are corrected and both files are regenerated and rechecked, the deliverables should be eligible for a focused final verification.
