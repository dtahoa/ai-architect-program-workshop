# 8-Minute Presentation Transcript

## Slide 1 — Title (0:00–0:45)
Good morning everyone. Our core design message is simple: **protect MERLIN, add intelligence, and never block orders**. Variant B uses an external asynchronous AI sidecar. AI recommends, but it does not order. MERLIN remains the only system that creates supplier orders and sends EDI. If the AI run is late, incomplete, disabled, or unsafe, it publishes nothing. The existing MERLIN process continues and the supply chain is protected.

## Slide 2 — Presentation Flow (0:45–1:15)
I will keep this presentation focused around one architectural decision: **keep AI outside MERLIN’s critical path**. The story has five parts. First, the operating principle. Second, the legacy constraints. Third, the system and container architecture. Fourth, the nightly batch and fail-open gates. Finally, the measurable release gates, risk controls, and rollout plan.

## Slide 3 — Executive Message (1:15–2:05)
The executive message is: **keep MERLIN in control and let AI fail safely**. AI is not a replacement for MERLIN. It produces forecasts and bounded replenishment recommendations only. The certified adapter writes approved rows into a controlled interface. MERLIN reads those rows, runs the unchanged engine, and creates the official order. A useful test is the 03:30 failure scenario. If AI dies late in the night, we do not run an emergency recovery that could risk the 04:00 EDI deadline. MERLIN proceeds through the existing min/max path.

## Slide 4 — Legacy Assessment (2:05–2:55)
The retrofit starts from constraints, not technology. We can change the external AI sidecar, certified adapters, monitoring, audit, and disable controls. We cannot change the MERLIN engine, MERLIN order creation, POS, fixed EDI format, or the 04:00 deadline. We also avoid unsafe areas such as undocumented MERLIN tables and internal scheduler behavior. Therefore, the sidecar uses exactly three supported extension points: a read-only nightly snapshot, a bounded override table before order generation, and a post-generation review queue for audit. No fourth interface is invented.

## Slide 5 — C4 Context (2:55–3:45)
At the system context level, the AI sidecar sits outside the ordering critical path. It consumes read-only snapshot data, generates forecasts and recommendations, and writes only bounded rows. MERLIN remains autonomous. It still owns order generation and EDI. This slide also defines hard boundaries: AI does not call POS, AI does not send EDI, AI does not contact suppliers, and the associate copilot does not write orders. The copilot is read-only and uses approved evidence only, especially for food-safety-related content.

## Slide 6 — Container View (3:45–4:35)
The container view explains how runtime risk is isolated. Snapshot validation checks schema, completeness, and residency. Forecast ML produces expected demand, quantiles, confidence intervals, and promotion demand signals — never an order. The deterministic optimizer applies hard constraints such as case pack, shelf life, DC capacity, inventory interval, and category caps. The certified adapter is the only write path. It allows only current-run, schema-valid, bounded, auditable rows, and it respects the kill switch.

## Slide 7 — Nightly Batch and Fail-Open Gates (4:35–5:35)
The nightly process protects the 04:00 EDI deadline. The batch starts at 22:00. Validation, forecasting, and optimization must complete before the late-stage gates. At 01:30, new retries stop to avoid recovery storms. At 01:45, the complete-set gate checks whether all required partitions are present. If any partition is missing, the sidecar publishes nothing for that run. At 02:00, AI write closes absolutely: no late publication and no emergency rerun. This 02:00 gate is valid only if measured MERLIN P99 order generation is at most 90 minutes. Otherwise, the AI close gate must move earlier.

## Slide 8 — Release Gates and Business Goals (5:35–6:35)
The numbers are release gates, not claimed benchmark results. The system must handle 27 million forecasts per night and around 7.04 million optimized candidates. At six-times peak, the target becomes 162 million forecasts and 42.24 million optimization candidates. The target runtime is 198.7 minutes within the 240-minute night window. Business goals include reducing out-of-stock from 7.2% to 3%, fresh waste from 4.8% to 3%, and WAPE from 41% to 25% or better. Because inventory accuracy is 78%, inventory is treated as an interval, not absolute truth. High-uncertainty recommendations should be suppressed.

## Slide 9 — ADR and Risk Controls (6:35–7:25)
Each design decision answers a rejection risk. The external async sidecar protects MERLIN availability. The deterministic optimizer ensures recommendations are explainable, bounded, and auditable. The certified adapter makes the write path narrow, versioned, schema-valid, and kill-switch aware. The main risks are deadline miss, inventory error, peak capacity, food-safety liability, and kill-switch realism. The mitigations are concrete: earlier gates, interval-based suppression, six-times benchmark testing, read-only approved evidence, and epoch-based disable with independent revoke and short-lived permits.

## Slide 10 — Rollout Roadmap and Close (7:25–8:00)
The rollout is deliberately conservative: discovery, replay, shadow, pilot, then expansion. The acceptance criterion is not only model accuracy. MERLIN must continue operating normally without AI support. The kill switch must detach sidecar publication immediately during severe incidents. The conclusion is that this is not an AI-first architecture. It is a **MERLIN-safe modernization architecture**: narrow interfaces, measurable release gates, audited recommendations, and no dependency on AI for order completion.
