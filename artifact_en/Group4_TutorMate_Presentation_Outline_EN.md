# TutorMate — Presentation Outline (10 core + 4 appendix = 14 slides, 13' talk + 2' buffer)

**Group 4 · Scenario B** · Workshop 2 Capstone — AI Technical Architect Training

> **Thesis (one sentence every slide serves):** TutorMate is not just a chatbot that uses an LLM; it is a tutoring system with controlled pedagogy, curriculum grounding, personalization, safety for minors, measurable quality, scalability and production operability.

Principle: **Presentation = decision + reasoning + proof points · ATAD = full technical detail · Appendix = evidence for deep-dive questions.** Do not present the whole ATAD.

## Person in charge & task allocation

| Code | Member | Role | Owns in ATAD | Presents slides | Q&A |
|---|---|---|---|---|---|
| M1 | Trầm Quốc Thuận | Requirements + traceability + architecture integration | §1, consolidation, Appendix E | 1, 10, App. D | Scope / Requirements (4.5) |
| M2 | Phạm Thị Thanh Huyền | Logical architecture + data/state flow | §2, §5 | 3, 4 | 4.2 Architecture |
| M3 | Nguyễn Hòa | AI/ML: LLM, RAG, deterministic tools, model strategy | §3, §4, §7 | 2, 5, App. A | 4.1 AI/ML |
| M4 | Trần Thanh Phụng | Responsible AI: safety, privacy, answer leakage | §8, §11, §6 safety | 6, App. C | 4.4 RAI |
| M5 | Lê Nguyễn Sỹ Bình | Cloud, scalability, latency, reliability | §9, §2.3, §10.7 | 8, App. B | 4.3 Cloud/Ops |
| M6 | Trần Trọng Phú | Evaluation + cost/capacity + ATAD consolidation | §6, §10 | 7 | Eval / Cost / Risk |
| M7 | Đinh Xuân Dũng | Slide integration + Q&A + presentation/rehearsal | Deck, `06`, `07` | 9 | Q&A moderator, timekeeper |

## Checklist: the 10 questions we must be able to prove → slide & ATAD

| # | Question | Slide | ATAD | Who |
|---|---|---|---|---|
| 1 | How is TutorMate different from ChatGPT? | 2 | §1.1, §1.7, §3.1 | Hòa |
| 2 | How is Socratic tutoring enforced? | 4 (state machine), 3 | ADR-001, Appendix A/B | Huyền / Hòa |
| 3 | What does personalization store / not store? | 4 (store / don't-store table) | ADR-002, §5.1, §5.7 | Huyền / Phụng |
| 4 | Curriculum grounding / RAG? | 5 (offline lane) | §5.3, §5.4, ADR-001 | Hòa |
| 5 | How do we prevent harmful content & answer leakage? | 6 | ADR-004, §3.8, §6.5, §11.5 | Phụng |
| 6 | How do we reach p95 < 3s? | 8 (latency budget) | §2.4, §10.6 | Bình |
| 7 | How do we scale to 5–8× peak? | 8 (load + scaling strategy) | §2.3, §10.7, R-05/R-06 | Bình |
| 8 | Which metrics evaluate the model/tutor? | 7 | §6, §7, §9.1 | Phú |
| 9 | Cost model & cost drivers? | 9 | §10.1–10.5 | Phú / Dũng |
| 10 | Requirement / Assumption / Proposal clearly separated? | App. D (+ footnotes on slides 6, 8, 9) | Notation at top of document, ADR Status, Appendix E | Thuận |

## Timing (rehearsal target 13:00–13:30 — do NOT rehearse to exactly 15:00)

| Time | Slide | Content | PIC |
|---|---|---|---|
| 00:00–00:45 | 1 | Problem + AI-First thesis | Thuận |
| 00:45–01:45 | 2 | TutorMate ≠ ChatGPT | Hòa |
| 01:45–03:30 | 3 | Target architecture | Huyền |
| 03:30–05:15 | 4 | Pedagogy + personalization | Huyền |
| 05:15–06:45 | 5 | AI pattern / RAG / curriculum grounding | Hòa |
| 06:45–08:30 | 6 | Safety for minors | Phụng |
| 08:30–10:00 | 7 | Evaluation | Phú |
| 10:00–11:45 | 8 | Performance + scale + cloud | Bình |
| 11:45–13:00 | 9 | Cost + operations | Dũng |
| 13:00–13:45 | 10 | 5 decisions + close → Q&A | Thuận |
| 13:45–15:00 | — | Buffer (pauses, slide transitions, interruptions) | Dũng keeps time |

---

## CORE

## Slide 1 — Problem & AI-First thesis (M1 · Thuận) · 0:45
- 3 points: 400k MAU Math & Science · students aged 11–16 · gap: worked solutions do not teach reasoning.
- Hero: **TutorMate coaches students toward the answer instead of simply giving it.**
- Visual: loop Stuck → Guided reasoning → Reaches the answer → Learner model improves.
- AI-First in one sentence: remove the AI and only the old quiz app is left.
- **Do not include:** assumptions, p95, cost, components.

## Slide 2 — TutorMate ≠ ChatGPT (M3 · Hòa) · 1:00 · Rubric 4.1, 4.2
- 5-row table: general-purpose → domain-locked · optimized for answering → optimized for learning · model knowledge → approved curriculum · prompt-driven → state machine · raw model output → validated output. Column 3 = architectural mechanism (ADR).
- Bottom line: **The LLM generates tutoring language; the system controls pedagogy, grounding, personalization and safety.**
- Callout: the runtime LLM never receives the answer (Answer Vault).

## Slide 3 — Target Architecture (M2 · Huyền) · 1:45 · Rubric 4.2, 4.3
- ~10 boxes: Student → API/Identity/Consent → Input Safety → **Tutor Orchestrator** → {Profile (BKT) · Hint Ladder RAG · Deterministic Tools} → Model Gateway → LLM → **Response Validator** → Student. Right column: Async (Telemetry · Evaluation · Monitoring · Cost tracking).
- Narrative: "The LLM is not trusted as the system of record or policy engine. The orchestrator owns tutoring state, tools handle precise checks, RAG grounds responses, and the validator is the last gate before the child."
- Full C4 L1–L3: ATAD §2. Deployment: Appendix B.

## Slide 4 — Pedagogy + Personalization (M2 · Huyền) · 1:45 · Rubric 4.1, 4.4 · answers Q2 + Q3
- Left: state machine Orient → Elicit → Hint → Check → Consolidate → Escalate; rule: hint level +1 only after ≥ 2 distinct errors (code, not prompt). Example `2x + 5 = 15`: chatbot "x = 5" vs TutorMate "What could you do first to remove the +5?".
- Right: **STORE** table (P(mastery)/skill, most recent error category, hint history, confidence, profile_version) vs **DO NOT STORE by default** (long-term transcript, real identity in prompt, psychological profile…).
- Bottom line: **Personalization is based on bounded learning signals, not unrestricted memory.**

## Slide 5 — Why Hybrid: RAG + LLM + Tools (M3 · Hòa) · 1:30 · Rubric 4.1 (30%)
- Capability → pattern table: language = LLM · grounding = RAG · formulas = CAS tool · mastery = BKT logic · pedagogy = state machine · safety/leak = validator; column "why not let the LLM do it".
- Right: curriculum-grounding offline lane (content → large LLM drafts ladder → **teachers review & version** → pgvector → runtime retrieves ≤ level).
- Why not fine-tune first (no corpus, curriculum changes) · Why not agents (no autonomy needed; latency/cost/safety).
- Bottom line: **Use an LLM only where the problem is genuinely linguistic.** Full matrix: Appendix A.

## Slide 6 — Safety for minors (M4 · Phụng) · 1:45 · Rubric 4.4 · main RAI slide
- 4 layers: Before the model · During tutoring · **Before delivery (Validator)** · After delivery.
- Callout: **UNVALIDATED OUTPUT NEVER REACHES THE STUDENT.**
- 2 biggest risks: Harmful content · Answer leakage (state machine + canonical answer check + leak classifier + adversarial eval).
- Footer: NIST AI RMF / ISO 42001 / Decree 13; jurisdiction-specific compliance = **assumption** pending legal confirmation. Threat register: Appendix C.

## Slide 7 — Evaluation (M6 · Phú) · 1:30 · Rubric 4.1, 4.4
- 8-dimension table: Correctness · Grounding · Pedagogy · Leakage · Safety · Adaptation · Performance · Cost — each with one metric + release threshold.
- Data: Golden set ≥ 1,000 · Frozen holdout · Adversarial red-team · Human educator review · Production sampling.
- **Release gate: quality + safety + latency + cost must all four pass** (shadow → 5% → 25% → 50% → 100%).
- Real outcomes: abandonment 35% → ≤ 22%; next-day mastery 48% → ≥ 60%; Free→Paid 5% → ≥ 7%.

## Slide 8 — Performance, Scale & Production (M5 · Bình) · 1:45 · Rubric 4.2, 4.3 · answers Q6 + Q7
- Left: load — average ≈ 11 turns/s · 5× ≈ 56 · **8× ≈ 89 (design target)** · 10× load test ≈ 111. [A] 30% of DAU × 8 turns.
- Middle: stateless API/orchestrator → HPA/KEDA · session/profile → managed state · retrieval → independently scalable · LLM → tiering + 2 providers · evaluation → async.
- Right: 4-line latency budget (policy/profile/retrieval < 0.7s · LLM 1.5–1.8s · validation + network < 0.5s · headroom). **Footnote: architecture budget, not a measured benchmark.**
- Bottom line: **first bottleneck = provider concurrency/latency, not the stateless API tier.** Degrade: Static Ladder Mode. Vendor-neutral stack.

## Slide 9 — Cost & Operability (M7 · Dũng) · 1:15 · Rubric 4.3, 4.2
- Equation: Monthly AI cost = turn volume × cost/turn + shared infra (≈ 29M × $0.00069 ≈ $18.4k; budget 20% of revenue = 8,000×P → viable when P ≥ ~$2.30).
- 5 cost drivers + controls: LLM tokens (tiering, context limit, cache) · moderation · retrieval/embeddings · retries/fallback/judge · infra.
- Right: Model gateway fallback · Release manifest · Canary rollout · Rollback ≤ 5' · Kill switch · Cost/drift alerts.
- Bottom line: **Production-ready = fails safely, degrades gracefully, stays within budget.**

## Slide 10 — Decision summary & close (M1 · Thuận) · 0:45 · Rubric 4.2, 4.5
1. Hybrid AI, not prompt-only · 2. Explicit Socratic state machine · 3. Minimal learner profile · 4. Safety + validation before delivery · 5. Eval-gated, scalable, cost-controlled.
- Thesis: **TutorMate is not just an LLM chatbot — it is a controlled AI tutoring platform that is safe, measurable and operable at scale.**
- Footer: team ownership, 7 people. Hand-over to Q&A: 4.1 → Hòa · 4.2 → Huyền · 4.3 → Bình · 4.4 → Phụng · Eval/Cost → Phú · Scope/Req → Thuận · Moderator → Dũng.

---

## APPENDIX (in the submitted file, not presented by default)

## Appendix A — Technology Selection Matrix (M3 · Hòa) · Rubric 4.1, 4.3
- RAG vs Fine-tune vs Agent vs Hybrid (5 criteria) · candidates & choice for LLM T1/T3, vector store, grader, learner model, cloud, gateway. Use when asked "why this technology?".

## Appendix B — Detailed Cloud Deployment (M5 · Bình) · Rubric 4.3
- Region/HA · VPC · Edge/API · EKS compute (pod count MVP → 8×) · Aurora + pgvector · Redis · MSK · S3 · LLM provider · Observability · Security — every layer has a vendor-neutral alternative. GPU strategy: managed for MVP, vLLM in Phase 2.

## Appendix C — Risk Register top 8 (M4 · Phụng) · Rubric 4.4, 4.2
- R-01 answer leakage · R-02 harmful content · R-03 wrong hint · R-04 child data · R-05 provider outage · R-06 peak · R-07 cost · R-08 cold-start; L/I, controls, owner. Full: ATAD §8.

## Appendix D — Requirement · Assumption · Proposal (M1 · Thuận) · Rubric 4.5, 4.2 · answers Q10
- **[R] Requirement** (brief: p95 < 3s, 5–8× peak, no answer reveal, consent, freemium) · **[A] Working assumption** (30% DAU × 8 turns; Vietnam market; 10% paying mix (sens. 5/15%); small model is good enough) · **[P] Architect proposal** (ALR < 1%, golden set ≥ 1,000, hybrid/AWS/pgvector/BKT, ≤ $0.0007/turn) · **[D] Data/training prerequisite** (≥ 50k labelled turns before 8B SFT; reviewed ladders for ≥ 80% of top-traffic exercises).
- Full register: ATAD Appendix E. Every ADR carries Status Accepted/Proposed.
