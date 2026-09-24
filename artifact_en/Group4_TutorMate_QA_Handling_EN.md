# TutorMate — Panel Q&A Handling Guide

**Group 4 · Scenario B — Q&A allocation**

| Code | Member | Role | Owns questions on |
|---|---|---|---|
| M1 | Trầm Quốc Thuận | Requirements + traceability + integration | Scope, requirements, trade-offs (4.5) |
| M2 | Phạm Thị Thanh Huyền | Logical architecture + data/state flow | 4.2 Architecture, C4, ADR, data |
| M3 | Nguyễn Hòa | AI/ML Lead | 4.1 AI/ML, models, RAG, deterministic tools |
| M4 | Trần Thanh Phụng | Responsible AI Lead | 4.4 RAI, safety, child privacy, leakage |
| M5 | Lê Nguyễn Sỹ Bình | Cloud & Scale Lead | 4.3 Cloud, latency, ops/rollout |
| M6 | Trần Trọng Phú | Evaluation & Cost Lead | Eval, ALR, cost, risk |
| M7 | Đinh Xuân Dũng | Slide + Q&A + rehearsal Lead | Moderator, Q&A coordination |


Purpose: prepare answers to panel questions along the **5 rubric criteria**, at the right "altitude" (architect: decision → reasoning → trade-off → evidence), without dropping into code detail. Each question has: **30-second answer** (say first), **Deep dive** (if the panel follows up), **Evidence in the ATAD**, **Traps to avoid**.

## 0a. General answering rules (Rubric 4.5 – 10%)
1. **4-step structure:** *Decision → Why (which NFR/constraint drives it) → Trade-off accepted → Numbers/evidence.* Max 45 seconds, then stop and ask "which part would the panel like me to go deeper on?"
2. **Routing:** Dũng (M7) receives the question, says one positioning sentence ("this is a cost question, Phú will answer") and hands over. Never two people speaking at once. A second person adds only when invited by Dũng ("Bình, add one point on eval").
3. **If you don't know, say so + how you'll find out:** "We have not measured X yet; the plan is to measure it with Y in a 2,000-student beta, and the decision threshold is Z." The panel grades reasoning, not omniscience.
4. **Don't argue model prices:** every price is an *assumption ±30%*; the architecture is designed so that a price change flips an alias, not the architecture. Turn pricing questions into *lever* questions.
5. **Always return to the Removal Test & ALR:** the two things a Scenario B panel weighs most are genuine AI-First and "measurably no answer leakage".
6. **Trap question "why not X":** acknowledge X's strengths → name the criteria where X loses (with weights) → state the condition under which you would switch to X (reversibility). Never say X is "wrong".

---

## 0b. The workshop's ten "pass/fail" questions — 30-second answers

| # | Question | 30-second answer | Evidence (slide / ATAD) | Who |
|---|---|---|---|---|
| 1 | How is TutorMate different from ChatGPT? | ChatGPT optimizes for "the correct answer"; TutorMate optimizes for "the next question that lets the student do it themselves". The difference lives outside the LLM: the Dialogue Policy decides how much to reveal, the runtime LLM never receives the answer, CAS verifies correctness, BKT remembers mastery instead of chat logs, and a two-way guard keeps it domain-locked for children. ChatGPT lets you prove none of those. | Slide 2 · §1.1, §1.7, §3.1 | Hòa |
| 2 | How is Socratic tutoring enforced? | 5 steps, the LLM only in step 4: (1) CAS Grader grades the step; (2) Policy state machine picks the action — advance / explain / raise level / reveal_allowed; rule: level +1 only after ≥ 2 distinct errors, reveal only after ≥ 90s & 12 sessions; (3) prompt contains ladder ≤ level only; (4) LLM T1 phrases it; (5) Output Guard cuts if it leaks. The policy is unit-tested as code; ALR is measurable. | Slides 4, 3 · ADR-001, App. A/B | Huyền / Hòa |
| 3 | What does personalization store / not store? | Store: `learner_id` (HMAC pseudonym), grade_band, P(mastery)/skill vector (~200 numbers, BKT), 3 flags (misconception, language, pace). Don't store: name/age/class in the tutor plane, raw conversations beyond 30 days, images/voice, behavioural profiles for advertising. PII only in the in-country zone; deletion = cutting the mapping. | Slide 4 · ADR-002, §5.7 | Huyền / Phụng |
| 4 | Curriculum grounding / RAG? | No RAG over worked solutions. Corpus = Hint Ladder (levels 0..k) + misconceptions + concept cards, generated offline by T3, CAS-checked, 100% teacher-reviewed, tagged chapter/skill matching EduNova's curriculum, versioned (`ladder_version`). Runtime retrieves by exercise_id (exact) + pgvector HNSW for concept cards; the final answer lives in a separate Answer Vault. | Slide 5 · §5.3, §5.4 | Hòa |
| 5 | How do we prevent harmful content & answer leakage? | Defence in depth, 3 layers each direction: deterministic (CAS-leak vs Vault, PII regex, topic allowlist, < 5ms) → self-trained int8 classifier (jailbreak / answer-fishing / off-topic / toxicity, < 60ms) → managed safety API backstop. Streaming guard per sentence; violation → cut & static hint. Self-harm → protocol with 15-minute SLA. Measured with G-Leak 2k (12 attack families) & G-Safety 1.5k; gate 0 harmful, ALR ≤ 1.0% golden set / ≤ 0.5% production. | Slide 6 · ADR-004, §6.5, §11.5 | Phụng / Hòa |
| 6 | How do we reach p95 < 3s? | A 10-step latency budget adding to 2.6s p95 (staging 8×): ladder cache Redis 60ms, parallel guard 120ms, CAS 20ms, TTFT T1 500ms, stream + per-sentence guard 1.2s; BKT/judge async. 12% of turns are templated < 300ms with no LLM call. If p95 > 2.5s: alert; > 3.5s for 10 min → auto-rollback / T2 cap. | Slide 8 · §2.4, §10.6 | Bình |
| 7 | How do we scale to 5–8× peak? | Stateless orchestrator HPA on RPS; guard KEDA on queue; elastic managed inference + 3× quota + multi-provider; Redis ladder cache 95% hit; Postgres read replicas; Kafka buffer. Peak ~89 turns/s (8×) load-tested in staging; 10× break point ~111 turns/s. Degraded: Static Ladder Mode (no LLM) still holds the 99.5% SLO. 10× break point at provider TPM quota & Postgres writes — planned for. | Slide 8 · §2.3, §10.7 | Bình |
| 8 | Which metrics? | Offline: ALR (CAS + clf), Pedagogy Score (T3 judge, 5 criteria, κ ≥ 0.7 vs teachers), factual error, harmful pass-through, grader accuracy ≥ 97%, latency, cost/turn — on 5 golden sets, sliced by grade/subject/behaviour/weak network. Online: next-day same-skill accuracy, in-tutor abandonment, "just give me the answer" tickets, 3% judge sampling, 200 human reviews/month. | Slide 7 · §6, §7 | Phú |
| 9 | Cost model & drivers? | 29M turns/month × blended $0.00069 ≈ $18.4k AI vs budget 8,000×P (10% mix × 20% of revenue) → viable when P ≥ ~$2.30/month. Drivers in order: (1) T2 turn ratio (10% → 5% saves $3.6k); (2) prompt-cache hit; (3) turns per free student (quota 20 → 15); (4) input tokens (6-turn history); (5) judge sampling. Levers available via Gateway config, no code change. | Slide 9 · §10 | Phú / Dũng |
| 10 | Requirement / Assumption / Proposal clearly separated? | Yes: [R] Requirement = given by the brief (p95 < 3s, 5–8× peak, no answer reveal, consent); [A] Working assumption = ours, validated in beta (30% DAU × 8 turns, Vietnam market, 10% paying mix); [P] Architect proposal = ours, backed by an ADR, open to challenge (ALR < 1%, golden set ≥ 1,000, hybrid/AWS/BKT); [D] Data/training prerequisite = enabling condition (≥ 50k labelled turns before 8B SFT). Every ADR has Status Accepted/Proposed; nothing is stated as "measured in production". | Appendix D · ATAD Appendix E | Thuận |

## 1. AI/ML Technical Expertise (30%) — lead: Hòa (M3), support Phú (M6)

### Q1.1 Why hybrid instead of fine-tuning a model for the Socratic style?
- **30s:** Fine-tuning is a tool to lock in *style*, not a tool to guarantee *no answer leakage*; and on day 1 we have ~0 real dialogue data. Fine-tuning on synthetic from the start = learning the large model's mistakes. So the MVP uses a deterministic policy + Hint Ladder + a small model for phrasing; fine-tuning (SFT/DPO) is **ADR-005 Phase 2** once ≥ 50k real labelled turns exist.
- **Deep dive:** A fine-tuned model can still leak if the answer is in context; the guarantee is least-privilege (ADR-001). When we do fine-tune, the benefit is lower cost (self-hosted 8B) and stable tone; go condition: self-host ALR/Pedagogy ≥ managed on the golden set.
- **ATAD:** §3.1, ADR-001, ADR-005. **Trap:** don't say "fine-tuning doesn't work" in general; talk about *timing* and *purpose*.

### Q1.2 Why not RAG over the worked solutions — simple and factually correct?
- **30s:** Because putting the solution in context *invites* the LLM to leak it; our internal prompt-only baseline with the solution in context gave ALR 8–15% under multi-turn attack. We still use RAG — but over the **Hint Ladder** (leveled guiding questions that never contain the final answer), generated offline and teacher-reviewed.
- **Deep dive:** The ladder is "structured RAG": exact lookup by `problem_id`, vector search only for misconception matching. The answer exists only in the Grader (CAS) and the Output Guard.
- **ATAD:** §3.1, §5.3, ADR-001.

### Q1.3 How do you force the LLM to "keep guiding"? Who trusts a prompt?
- **30s:** We don't rely on the prompt. 4 layers: (1) **Structure**: the model doesn't have the answer; (2) **Deterministic policy** decides the reveal level — the LLM has no say; (3) **Output Guard** CAS + paraphrase classifier cuts the stream on a leak; (4) **Eval**: G-Leak with 2,000 attack cases is a CI gate, ALR measured on 100% of turns via CAS in prod.
- **Deep dive:** The LLM can still *solve it itself* and say so → caught by CAS equivalence against the Answer Vault (numbers/expressions) and by the classifier for paraphrases ("about double the number you just computed"). No yes/no confirmation of answers → blocks answer-fishing.
- **ATAD:** §2.2 principles, §2.4 steps 5–8, ADR-004, §6.5.

### Q1.4 Isn't the Hint Ladder too rigid — what if the student solves it differently?
- **30s:** Acknowledged as the trade-off of ADR-001. Mitigated by: multi-branch ladders for common approaches (T3 generates 2–3 approaches when a teacher flags it), a misconception bank, and **T2 escalation** when the Grader returns `irrelevant` twice (student is on another path). Track a "left the ladder" metric — if > 10% for an exercise, regenerate the ladder.
- **Deep dive:** A student on a *correct* alternative path is still confirmed by CAS because it compares *mathematical equivalence*, not strings. **ATAD:** ADR-001 consequences, §6.2 slices, R-08.

### Q1.5 Why grade with SymPy instead of an LLM?
- **30s:** Tool fit: correctness of an expression is an *algebra* problem, not a *language* problem. CAS is deterministic, < 20ms, does not hallucinate, costs ~$0. The LLM is used only to *explain* the error (why it's wrong) based on the misconception_id the Grader returns.
- **Deep dive:** Free-text Physics/Chemistry answers aren't pure expressions → step alignment: T1 maps the student's sentence to a ladder step (classification), CAS checks the quantitative part; grader accuracy target ≥ 97% on G-Grade. **ATAD:** §1.1, §3.1, §6.4.

### Q1.6 Learner model: why BKT rather than Deep Knowledge Tracing or letting the LLM "remember"?
- **30s:** Three criteria: explainable to parents (P(mastery) is a meaningful probability), data minimisation (~200 numbers per student, no chat stored), O(1) cost. DKT gains 2–4 pts AUC but needs GPUs and is hard to explain → Phase 3 candidate if an A/B proves a learning uplift. LLM "memory" via summaries = storing children's content + not measurable.
- **ATAD:** ADR-002, §5.7, §11.2.

### Q1.7 Cold start: can synthetic data be trusted? What if the large model teaches wrongly?
- **30s:** Synthetic is used only to *bootstrap eval & prompts*, not to fine-tune on day 1. Process: rubric with teachers → simulated student-persona × T3 tutor → **teachers curate** 1,500 → judge calibrated to κ ≥ 0.7 with teachers → 2,000-student beta → replace 30% of the golden set with real data after month 1 (R-11).
- **Deep dive:** Correct knowledge doesn't come from synthetic data but from the content bank + CAS + teacher-reviewed ladders. Synthetic only simulates *student behaviour*. **ATAD:** §6.1, R-11, slide 5.

### Q1.8 Is a small tier-1 model "smart" enough to teach?
- **30s:** It doesn't need to teach — the ladder + teachers already "taught" offline. The small model only phrases the current hint level to fit the student's reply. When reasoning is needed (stuck ≥ 2 times, multi-step free text, guard just blocked) → escalate to T2 (~10% of turns). Model layer matrix: small managed 4.55 vs frontier 3.30, mainly on cost (weight 20%) and latency.
- **ATAD:** §3.3, §7. **Trap:** don't commit to a specific model name; talk "class" and alias.

### Q1.9 How did you choose the embedding model & chunking?
- **30s:** Corpus < 1M vectors; the knowledge units are already semantic (ladder step, misconception, concept card of 200–400 tokens) so no overlapping chunks. Multilingual embedding (bge-m3 class) for Vietnamese + math notation; version recorded with every vector; shadow re-embed on model change. The primary query is exact by `problem_id` — vectors are secondary.
- **ATAD:** §5.3, §5.4.

### Q1.10 Is LLM-as-judge trustworthy for measuring pedagogy?
- **30s:** Only after calibration: ≥ 300 teacher labels, Cohen κ ≥ 0.7 before use; 5-criteria reference-guided rubric; 200 human evals/month and mandatory on any model/policy change. Numeric/expression ALR does not use a judge — it uses CAS.
- **ATAD:** §6.3, §6.6.

### Q1.11 Generating new practice items — how do you ensure they're correct?
- **30s:** Template-based + LLM T2 generates variants targeting the misconception just seen, **CAS verifies** the solution, new templates pass teacher QA before entering the pool; async, off the hot path. **ATAD:** §1.1, §5.2, G-Practice.

### Q1.12 Could you do voice?
- **30s:** Out of MVP scope per the brief. If done: STT/TTS adds ~600–900ms and 3–5× cost/turn → only viable for the paid tier or with on-device TTS. Proposed Phase 3 with its own cost/latency gate. **ATAD:** slide 10, Appendix D.

---

## 2. Architecture Design & System Thinking (25%) — lead: Huyền (M2), support Thuận (M1)

### Q2.1 Why a hand-written state machine for the dialogue policy instead of LangGraph / an agent framework?
- **30s:** The policy must be *auditable & unit-testable as code* (100% branch coverage) because it decides the reveal level — the thing the panel and parents need to trust. < 1,000 lines, no framework needed. LangGraph is the fallback if flows get more complex (score 3.95 vs 4.80). Managed agent platforms rejected for lock-in (1/5).
- **ATAD:** §3.4, Appendix A.

### Q2.2 Latency p95 < 3s: how does it add up and where's the risk?
- **30s:** Budget: guard 120ms ∥, retrieval 20ms (95% cache), TTFT 500ms, streaming 150 tokens ~1.2s, per-sentence guard in parallel with the stream, async writes → **2.6s p95 measured in staging at 8×**. Risks: provider TTFT at peak → failover; slower T2 → 10% cap; client network (RTT > 400ms) → separate slice with 3.5s target.
- **ATAD:** §2.4, §6.2, §10.6.

### Q2.3 Streaming while still checking output — isn't that a contradiction?
- **30s:** Check per *sentence*: deterministic < 5ms + int8 classifier < 60ms per sentence; a sentence is pushed to the client only after it passes. Violation → cut, one leak-repair attempt, then static hint. UX shows "thinking again…". **ATAD:** ADR-004.

### Q2.4 What happens when the LLM provider goes down at 8pm in exam season?
- **30s:** Gateway fails over to provider B (different vendor, pre-configured, reserve quota); both down → **Static Ladder Mode**: students still get leveled hints, no dialogue. Availability counts degraded as "up" at 99.5%, LLM-mode 99.0%. Playbook #3. **ATAD:** §9.2, R-06.

### Q2.5 Isn't the architecture over-engineered for an MVP (Kafka, lakehouse, Flink)?
- **30s:** Acknowledged — we could start leaner: Phase 0 MVP can run on Postgres + Redis + a managed queue; Kafka/Iceberg come in at 1M turns/day when eval-in-prod and lineage are needed. What *cannot* be cut: Gateway, two-way Guard, Ladder, Policy, eval harness — they are bound to hard NFRs. **ATAD:** §2.2; be explicit about "day 1" vs "at scale".

### Q2.6 How do you integrate with the EduNova Core App? Who owns the learner model?
- **30s:** Avro contracts `content.problem.v2`, `behavior.attempt.v1` (schema registry + Great Expectations); TutorMate writes mastery back to Core via a Kafka topic; Core owns identity, TutorMate owns the pseudonymous learner model. **ATAD:** §5.6.

### Q2.7 Removal test — honestly, what's left without the AI?
- **30s:** A quiz app with worked solutions (what parents complain about) + a static Hint Ladder. The static ladder is more useful than the old app but has no dialogue and doesn't adapt to the student's specific error — it's not a tutor. We deliberately turned "what's left" into a controlled kill-switch mode. **ATAD:** §1.8, §9.3.

### Q2.8 Where does session state live? What if Redis is lost?
- **30s:** Redis (24h TTL) for state; every transition is also written to Kafka → rebuildable. Losing Redis: the orchestrator reconstructs state from the last 6 turns in the client + ladder (the client keeps a light copy) or restarts at the most recent level. **ATAD:** §2.4, R-12.

### Q2.9 What's special about your C4 L3 diagram compared with a normal web app?
- **30s:** Three AI-specific points: (1) a pseudonymisation boundary between the in-country PII zone and the tutor plane; (2) a dedicated node pool for the guard (KEDA) and GPU scale-to-zero (Phase 2); (3) an offline plane (enrichment, eval, fine-tune) separated from runtime, on spot. **ATAD:** §2.3.

---

## 3. Cloud & Infrastructure Expertise (20%) — lead: Bình (M5)

### Q3.1 Why AWS and not GCP — Gemini Flash-Lite is the cheapest?
- **30s:** The two tie at 4.15/5; we chose on team capability and the breadth of the managed inference catalogue (many model vendors behind one endpoint). Gemini Flash-Lite is still callable via LiteLLM cross-cloud if cheaper — infrastructure and model are two independent decisions. Everything is K8s/Postgres/Kafka/S3-compatible → changing cloud = IaC. **ATAD:** §3.6, ADR-003.

### Q3.2 GPUs: why not self-host from day one to be cheaper and avoid lock-in?
- **30s:** Do the multiplication: 2.6B tokens/day; L4 ~2k tok/s → ~15 GPUs baseline, ~120 at 8× peak; cost ≈ managed small model ($8–9k/month) but plus GPU ops, on-call, and slower autoscale than managed. Not worth it at MVP; ADR-005 Phase 2 once fine-tuning is done and volume is stable — then ~40% saving on T1. **ATAD:** §3.3 (C), ADR-005, slide 8, Appendix B.

### Q3.3 How do you handle 5–8× peak? Does autoscaling keep up?
- **30s:** Stateless orchestrator → HPA on RPS (scales in ~60s), scheduled pre-scale (5pm, exam season); guard pool on KEDA; elastic managed LLM + 3× quota + multi-provider; Redis ladder cache 95% keeps the DB from seeing the peak. The 8× load test is a staging gate. **ATAD:** §2.3, §6.6, §10.7.

### Q3.4 Serverless or containers?
- **30s:** Containers for the hot path (guard, orchestrator) because they need warm instances & p95; serverless for async (practice generation, DSAR, reports). No serverless for the guard because a 2–5s int8 model cold start breaks p95. **ATAD:** slide 8, Appendix B.

### Q3.5 ~$18.4k cost — viable only when P ≥ ~$2.30?
- **30s:** Agreed — that is the nature of freemium, and P is a variable we have not assumed; budget = 8,000×P so it is viable when P ≥ ~$2.30 (AI variable) or ~$3.70 (incl. infra). 6 ordered levers (T2 cap → cache → templated → quota → history → self-host) with ~$10k total headroom; alerts at 80/100/110% daily spend; hard stop at 130% moves the free tier to static mode, paid prioritised. Sensitivity: 5% mix → P ≥ ~$4.60. ±30% price sensitivity in backup B3. **ATAD:** §10.3, §9.4 governance, R-05.

### Q3.6 Cost per query p95 of $0.0028 is 5× typical — why accept it?
- **30s:** p95 is a T2 turn (genuinely stuck, free text) — exactly where pedagogical value is highest; capped at 10% of turns so it contributes only $0.00028 to blended. If T2 ratio > 12% → warn, > 18% → page and auto-reduce the cap. **ATAD:** §10.2, §9.1.

### Q3.7 Data residency: where does children's PII live if the cloud has no in-country region?
- **30s:** Split: identity/consent store in-country (compliant DC/partner); the tutor plane in a nearby region (SEA) receives only the HMAC `learner_id` + grade band; no PII leaves; the LLM receives redacted prompts with zero retention. If regulation tightens → the whole K8s-portable tutor plane moves to an in-country DC. **ATAD:** §2.3, §5.7, R-04.

### Q3.8 What's the first 10× break point?
- **30s:** Provider TPM/RPM quota → 429 signal at the gateway before latency rises → multi-provider / provisioned throughput / self-host. Next the Postgres write path (mastery + logs) → Kafka lag → micro-batch BKT, drop direct writes. Then the guard pool. **ATAD:** §10.7.

### Q3.9 Weak networks — what does the client-side architecture do?
- **30s:** SSE resume token, server-side state (Redis 24h), offline queue for answers, gzip payload < 300KB, Capacitor/PWA for low-end devices, degraded static mode. Separate eval slice for RTT > 400ms. **ATAD:** §2.3, §6.2, R-12.

### Q3.10 DR / RTO / RPO?
- **30s:** Warm standby in a second region: K8s min replicas, Postgres async replica; RTO 30 min, RPO 5 min; ladder & prompt artifacts replicated on object storage; the PII zone is not replicated outside the country (in-country DR). **ATAD:** §2.3.

---

## 4. Responsible AI & Governance (15%) — lead: Phụng (M4)

### Q4.1 EU AI Act: is TutorMate high-risk?
- **30s:** Education is Annex III when the system *evaluates learning outcomes or steers the learning path*. A purely guiding tutor could be limited-risk, but the learner model adjusts difficulty → affects the path. With children we **proactively adopt a high-risk posture**: risk management (§8), data governance (§5), logging (§9), human oversight (§11.4), Art. 50 disclosure. Cheaper than being reclassified later. **ATAD:** §11.1.

### Q4.2 How do you do parental consent at 400k scale?
- **30s:** Consent ledger with scopes (tutor, retain_chat_30d, analytics), parent verified via the Core App (email/payment), checked at the Gateway every session (freshness < 5 min); missing scope → tutor not enabled; self-service withdrawal → TTLs run. **ATAD:** §5.6, §5.7.

### Q4.3 XAI for LLMs — do SHAP/LIME apply?
- **30s:** Not for LLM text generation. We use *process transparency* for students, *learner-model transparency* (BKT is a self-explaining probability) for parents, *attribution* (ladder_version, misconception, judge rationale) for teachers; SHAP applies to the **guard classifier** and feature importance to BKT. **ATAD:** §11.2.

### Q4.4 Bias: you don't collect gender/region, so how do you test fairness?
- **30s:** With *synthetic personas* (paired male/female dialogues, North/Central/South dialects, unaccented typing) and available slices (proficiency, device). Gap threshold ≤ 0.2–0.3 is the gate. We don't collect real attributes because of child data minimisation. **ATAD:** §11.3.

### Q4.5 Prompt injection: a student types "ignore previous instructions" — then what?
- **30s:** User text never enters the system role; ladders are generated offline from approved content; Input Guard injection classifier; runtime is **tool-less** (no tool calls → no side effects); output JSON schema enforced; and even if "persuaded", the model has no answer to leak. **ATAD:** §11.5.

### Q4.6 Self-harm / abuse shows up in chat — what do you do?
- **30s:** Layer 3 managed safety API + classifier catch it; the policy switches to a safety script (helpline, tutoring stops), HIL safety officer with 15-minute SLA, parent notification as required; 100% of cases reviewed. **ATAD:** §11.4, R-02.

### Q4.7 Risk register: are the thresholds tied to real operations?
- **30s:** Every threshold is an alert in §9.1: ALR > 1.5%/1h freeze, > 3% rollback; ≥ 1 confirmed harmful → kill switch; cost > $0.0009/turn → reduce T2 cap. Register reviewed quarterly by the RAI board. **ATAD:** §8 ↔ §9.1.

### Q4.8 Model extraction — someone scrapes the whole Hint Ladder?
- **30s:** Per-learner rate limit (20/200 turns/day), anomaly detection (> 200 exercises/hour), no public API, ladder wording watermarked for tracing. Risk accepted at medium (R-10) because the ladder is not the answer. **ATAD:** R-10, §11.5.

### Q4.9 Is sending children's data to an LLM provider a violation?
- **30s:** No PII is sent: redacted beforehand, the `learner_id` is not in the prompt, content is math/science; zero-retention / no-training contract; private endpoint; egress allowlist; recorded in the DPIA. **ATAD:** §5.7, §11.5.

### Q4.10 Do students know they're talking to an AI?
- **30s:** Yes — clear disclosure in the UI (Art. 50), the persona doesn't impersonate a real person, and each session ends with a summary of what the AI did. **ATAD:** §11.1.

---

## 5. Eval & LLMOps (cross-cutting 4.1/4.2) — leads: Phú (M6, eval/cost) & Bình (M5, ops/rollout)

### Q5.1 How do you know the ALR after changing a prompt/model?
- **30s:** CI runs the full G-Leak + G-Safety and 500 judge samples; pass → 5% canary with 20% judge sampling for 24h; prod: 100% of turns through CAS-ALR, 3% judge; dashboard by `prompt_version × model_id × slice`; auto-freeze/rollback on thresholds. **ATAD:** §6.6, §6.7, §9.

### Q5.2 The provider silently swaps the model under the same name — how do you detect it?
- **30s:** Daily shadow canary of 300 fixed golden samples → Δ Pedagogy/ALR > 0.3 vs the 7-day baseline → alert + pin version (if available) / failover. **ATAD:** R-07, §6.7.

### Q5.3 How long does rollback take and what does it include?
- **30s:** ≤ 5 minutes: flip alias/prompt pointer in the gateway (config, no deploy) → invalidate cache → verify ALR/latency for 10 min → incident. Ladder rollback per `ladder_version`. **ATAD:** §9.3.

### Q5.4 ALR threshold 1% — why not 0%?
- **30s:** 0% is not measurable with a generative system; the gate is ≤ 1.0% on the adversarial golden set and ≤ 0.5% on production-sampled traffic; Phase 2 may tighten to 0.3%. If the panel wants tighter → lever: T2 for turns with fishing signals, more leak-repair. **ATAD:** §1.4, slides 6, 7.

### Q5.5 What triggers fine-tune / retraining?
- **30s:** Ladder: new exercise / ≥ 3 teacher fixes; misconceptions: weekly mining; guard clf: monthly or > 5 prod FNs; BKT: quarterly; T1 fine-tune: ≥ 50k labelled turns or Pedagogy drift. Feedback loop: prod → judge → teacher queue → labels → golden/DPO set. **ATAD:** §9.4.

### Q5.6 Judge grading the judge — a self-congratulating loop?
- **30s:** The judge is anchored to teachers (κ ≥ 0.7, monthly re-calibration), independent human eval 200/month, and real outcomes (next-day accuracy A/B) are the final yardstick. **ATAD:** §6.3.

---

## 6. "Killer" questions & how to parry

| Question | One-sentence parry | Then move to |
|---|---|---|
| "Isn't this just a chatbot with a long prompt?" | No — the LLM has no answer, doesn't decide the reveal level, doesn't decide safety; those three live outside the LLM. | Slides 2, 3, ADR-001 |
| "If I were a student I'd get the answer in 3 turns." | Try it against G-Leak; mechanism: no yes/no confirmation, ≥ 2 distinct errors + 90s before any reveal. | §6.5, Appendix A/C |
| "Your cost is based on today's model prices." | Correct, ±30%; the architecture absorbs it via alias + 6 levers; a 30% increase → levers 1–3 are enough. | §10.3 |
| "Parents sue because the AI taught something wrong." | Knowledge comes from CAS-checked, teacher-reviewed ladders, not the LLM; factual error gate 0.5%; teacher hotfix of the ladder; logged tuple for tracing. | R-03, §5.3 |
| "Why not buy an off-the-shelf solution (Khanmigo-class)?" | The brief is AI-First on EduNova's data & content; build the core (policy, ladder, guard), buy commodity (LLM, safety API, o11y). | §3 |
| "Can your team actually build this?" | Phase 0 MVP = 6 weeks with 5–6 people; the heavy part is content enrichment (teachers), not code. | Slide 10, Appendix D |
| "What keeps you up at night?" | R-11: cold-start eval not reflecting real students → 2,000-student beta and early replacement of 30% of the golden set. | §8 |

---

## 7. Pre-defence checklist
- [ ] Everyone knows the 3 key numbers: **29M turns/month · $0.00069/turn · ~$18.4k vs 8,000×P (P ≥ ~$2.30)**; **ALR ≤ 1.0% golden / ≤ 0.5% prod**; **p95 2.6s**.
- [ ] Dũng (M7) knows the Q&A allocation table and hand-over lines.
- [ ] Appendix A–D (slides 11–14) open and ready; not presented by default.
- [ ] Demo one 4-turn dialogue example (level 0 → 2) and one blocked attack example (Appendix C) — 30 seconds if the panel asks "show us".
- [ ] Never state a specific model name as a commitment; say "class" + alias.
- [ ] End every answer with: "Which part would the panel like me to go deeper on?"
