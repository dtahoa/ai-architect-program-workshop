# TutorMate — Presentation Outline (10 core + 4 appendix = 14 slides, 13' nói + 2' buffer)

**Group 4 · Scenario B** · Workshop 2 Capstone — AI Technical Architect Training

> **Thesis (1 câu, mọi slide phục vụ câu này):** TutorMate không chỉ là chatbot dùng LLM; nó là một tutoring system có pedagogy được kiểm soát, curriculum-grounded, personalized, safe for minors, measurable, scalable và operable in production.

Nguyên tắc: **Presentation = decision + reasoning + proof points · ATAD = chi tiết kỹ thuật đầy đủ · Appendix = bằng chứng khi bị hỏi sâu.** Không present toàn bộ ATAD.

## Phân công (Person in charge & Task allocation)

| Mã | Thành viên | Vai trò | Sở hữu ATAD | Slide trình bày | Q&A |
|---|---|---|---|---|---|
| M1 | Trầm Quốc Thuận | Requirements + traceability + tổng hợp kiến trúc | §1, tổng hợp, Appendix E | 1, 10, App. D | Scope / Requirement (4.5) |
| M2 | Phạm Thị Thanh Huyền | Logical architecture + data/state flow | §2, §5 | 3, 4 | 4.2 Architecture |
| M3 | Nguyễn Hòa | AI/ML: LLM, RAG, deterministic tools, model strategy | §3, §4, §7 | 2, 5, App. A | 4.1 AI/ML |
| M4 | Trần Thanh Phụng | Responsible AI: safety, privacy, answer leakage | §8, §11, §6 safety | 6, App. C | 4.4 RAI |
| M5 | Lê Nguyễn Sỹ Bình | Cloud, scalability, latency, reliability | §9, §2.3, §10.7 | 8, App. B | 4.3 Cloud/Ops |
| M6 | Trần Trọng Phú | Evaluation + cost/capacity + ATAD consolidation | §6, §10 | 7 | Eval / Cost / Risk |
| M7 | Đinh Xuân Dũng | Slide integration + Q&A + presentation/rehearsal | Deck, `06`, `07` | 9 | Q&A moderator, timekeeper |

## Checklist 10 câu hỏi "chứng minh được" → slide & ATAD

| # | Câu hỏi | Slide | ATAD | Người |
|---|---|---|---|---|
| 1 | TutorMate khác ChatGPT ở đâu? | 2 | §1.1, §1.7, §3.1 | Hòa |
| 2 | Socratic tutoring được enforce thế nào? | 4 (state machine), 3 | ADR-001, Appendix A/B | Huyền / Hòa |
| 3 | Personalization lưu gì / không lưu gì? | 4 (bảng lưu/không lưu) | ADR-002, §5.1, §5.7 | Huyền / Phụng |
| 4 | Curriculum grounding / RAG? | 5 (offline lane) | §5.3, §5.4, ADR-001 | Hòa |
| 5 | Ngăn harmful content & answer leakage? | 6 | ADR-004, §3.8, §6.5, §11.5 | Phụng |
| 6 | Đạt p95 < 3s thế nào? | 8 (latency budget) | §2.4, §10.6 | Bình |
| 7 | Scale 5–8× peak? | 8 (tải + chiến lược scale) | §2.3, §10.7, R-05/R-06 | Bình |
| 8 | Metrics đánh giá model/tutor? | 7 | §6, §7, §9.1 | Phú |
| 9 | Cost model & cost drivers? | 9 | §10.1–10.5 | Phú / Dũng |
| 10 | Requirement / Assumption / Proposal phân biệt rõ? | App. D (+ footnote slide 6, 8, 9) | Quy ước đầu tài liệu, ADR Status, Appendix E | Thuận |

## Timing (rehearsal target 13:00–13:30, KHÔNG rehearsal đúng 15:00)

| Thời gian | Slide | Nội dung | PIC |
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
| 13:45–15:00 | — | Buffer (pause, chuyển slide, interruption) | Dũng giữ giờ |

---

## CORE

## Slide 1 — Problem & AI-First thesis (M1 · Thuận) · 0:45
- 3 ý: 400k MAU Math & Science · học sinh 11–16 · gap: lời giải sẵn không dạy suy luận.
- Hero: **TutorMate dẫn dắt học sinh đi tới đáp án — thay vì đưa đáp án.**
- Visual: vòng lặp Kẹt bài → Gợi mở → Tự tới đáp án → Learner model cập nhật.
- AI-First 1 câu: bỏ AI = còn app trắc nghiệm cũ.
- **Không đưa vào:** assumptions, p95, cost, component.

## Slide 2 — TutorMate ≠ ChatGPT (M3 · Hòa) · 1:00 · Rubric 4.1, 4.2
- Bảng 5 hàng: mục đích chung → domain-locked · tối ưu trả lời → tối ưu học · kiến thức model → curriculum đã duyệt · prompt-driven → state machine · output từ model → output đã validate. Cột 3 = cơ chế trong kiến trúc (ADR).
- Bottom line: **LLM sinh ngôn ngữ gia sư; hệ thống kiểm soát pedagogy, grounding, personalization, safety.**
- Callout: LLM runtime không bao giờ nhận đáp án (Answer Vault).

## Slide 3 — Target Architecture (M2 · Huyền) · 1:45 · Rubric 4.2, 4.3
- ~10 box: Student → API/Identity/Consent → Input Safety → **Tutor Orchestrator** → {Profile (BKT) · Hint Ladder RAG · Deterministic Tools} → Model Gateway → LLM → **Response Validator** → Student. Cột phải: Async (Telemetry · Evaluation · Monitoring · Cost tracking).
- Narrative: "LLM không được tin làm system of record hay policy engine. Orchestrator sở hữu tutoring state, tools làm kiểm tra chính xác, RAG grounding, Validator là cổng cuối trước trẻ em."
- C4 L1–L3 đầy đủ: ATAD §2. Deployment: Appendix B.

## Slide 4 — Pedagogy + Personalization (M2 · Huyền) · 1:45 · Rubric 4.1, 4.4 · trả lời Q2 + Q3
- Trái: state machine Orient → Elicit → Hint → Check → Consolidate → Escalate; rule hint level +1 chỉ sau ≥ 2 lỗi khác nhau (code, không prompt). Ví dụ `2x + 5 = 15`: chatbot "x = 5" vs TutorMate "Em nghĩ bước đầu nên làm gì để bỏ +5?".
- Phải: bảng **LƯU** (P(mastery)/skill, loại lỗi gần nhất, hint history, confidence, profile_version) vs **KHÔNG LƯU mặc định** (transcript dài hạn, danh tính thật trong prompt, hồ sơ tâm lý…).
- Bottom line: **Personalization dựa trên bounded learning signals, không phải bộ nhớ không giới hạn.**

## Slide 5 — Why Hybrid: RAG + LLM + Tools (M3 · Hòa) · 1:30 · Rubric 4.1 (30%)
- Bảng capability → pattern: ngôn ngữ = LLM · grounding = RAG · công thức = CAS tool · mastery = BKT logic · pedagogy = state machine · safety/leak = validator; cột "vì sao không để LLM làm".
- Phải: offline lane curriculum grounding (nội dung → LLM lớn sinh ladder → **giáo viên duyệt & version** → pgvector → runtime retrieve ≤ level).
- Why not fine-tune trước (không có corpus, chương trình đổi) · Why not agent (không cần autonomy; latency/cost/safety).
- Bottom line: **Dùng LLM chỉ ở nơi bài toán thực sự là ngôn ngữ.** Matrix đầy đủ: Appendix A.

## Slide 6 — Safety for minors (M4 · Phụng) · 1:45 · Rubric 4.4 · slide RAI chính
- 4 lớp: Trước model · Trong khi dạy · **Trước khi gửi (Validator)** · Sau khi gửi.
- Callout: **OUTPUT CHƯA VALIDATE KHÔNG BAO GIỜ TỚI HỌC SINH.**
- 2 rủi ro lớn nhất: Harmful content · Answer leakage (state machine + canonical answer check + leak classifier + adversarial eval).
- Footer: NIST AI RMF / ISO 42001 / NĐ13; pháp lý theo thị trường = **assumption** cần legal xác nhận. Threat register: Appendix C.

## Slide 7 — Evaluation (M6 · Phú) · 1:30 · Rubric 4.1, 4.4
- Bảng 8 chiều: Correctness · Grounding · Pedagogy · Leakage · Safety · Adaptation · Performance · Cost — mỗi chiều 1 metric + ngưỡng release.
- Dữ liệu: Golden set ≥ 1,000 · Frozen holdout · Adversarial red-team · Human educator review · Production sampling.
- **Release gate: quality + safety + latency + cost phải đạt cả 4** (shadow → 5% → 25% → 50% → 100%).
- Outcome thật: bỏ dở 35% → ≤ 22%; next-day mastery 48% → ≥ 60%; Free→Paid 5% → ≥ 7%.

## Slide 8 — Performance, Scale & Production (M5 · Bình) · 1:45 · Rubric 4.2, 4.3 · trả lời Q6 + Q7
- Trái: tải — trung bình ≈ 11 turns/s · 5× ≈ 56 · **8× ≈ 89 (design target)** · 10× load test ≈ 111. [A] 30% DAU × 8 lượt.
- Giữa: stateless API/orchestrator → HPA/KEDA · session/profile → managed state · retrieval → scale độc lập · LLM → tiering + 2 provider · evaluation → async.
- Phải: latency budget 4 dòng (policy/profile/retrieval < 0.7s · LLM 1.5–1.8s · validation + network < 0.5s · headroom). **Footnote: budget kiến trúc, chưa phải benchmark đo thật.**
- Bottom line: **bottleneck đầu tiên = provider concurrency/latency, không phải tầng API stateless.** Degrade: Static Ladder Mode. Vendor-neutral stack.

## Slide 9 — Cost & Operability (M7 · Dũng) · 1:15 · Rubric 4.3, 4.2
- Phương trình: Monthly AI cost = turn volume × cost/turn + shared infra (≈ 29M × $0.00069 ≈ $18.4k; budget 20% doanh thu = 8.000×P → viable khi P ≥ ~$2.30).
- 5 cost driver + control: LLM tokens (tiering, context limit, cache) · moderation · retrieval/embeddings · retries/fallback/judge · infra.
- Phải: Model gateway fallback · Release manifest · Canary rollout · Rollback ≤ 5' · Kill switch · Cost/drift alerts.
- Bottom line: **Production-ready = fail an toàn, degrade có kiểm soát, nằm trong budget.**

## Slide 10 — Decision summary & close (M1 · Thuận) · 0:45 · Rubric 4.2, 4.5
1. Hybrid AI, không phải prompt-only · 2. Socratic state machine tường minh · 3. Learner profile tối thiểu · 4. Safety + validation trước khi gửi · 5. Eval-gated, scalable, cost-controlled.
- Thesis: **TutorMate không chỉ là LLM chatbot — là nền tảng gia sư AI được kiểm soát, an toàn, đo được, vận hành được ở quy mô.**
- Footer: team ownership 7 người. Chuyển Q&A: 4.1 → Hòa · 4.2 → Huyền · 4.3 → Bình · 4.4 → Phụng · Eval/Cost → Phú · Scope/Req → Thuận · Moderator → Dũng.

---

## APPENDIX (trong file nộp, không present mặc định)

## Appendix A — Technology Selection Matrix (M3 · Hòa) · Rubric 4.1, 4.3
- RAG vs Fine-tune vs Agent vs Hybrid (5 tiêu chí) · ứng viên & lựa chọn cho LLM T1/T3, vector store, grader, learner model, cloud, gateway. Dùng khi hỏi "why this technology?".

## Appendix B — Detailed Cloud Deployment (M5 · Bình) · Rubric 4.3
- Region/HA · VPC · Edge/API · EKS compute (pod count MVP → 8×) · Aurora + pgvector · Redis · MSK · S3 · LLM provider · Observability · Security — mỗi lớp có thay thế vendor-neutral. GPU strategy: managed MVP, vLLM Phase 2.

## Appendix C — Risk Register top 8 (M4 · Phụng) · Rubric 4.4, 4.2
- R-01 lộ đáp án · R-02 harmful · R-03 hint sai · R-04 dữ liệu trẻ em · R-05 provider outage · R-06 peak · R-07 cost · R-08 cold-start; L/I, kiểm soát, owner. Full: ATAD §8.

## Appendix D — Requirement · Assumption · Proposal (M1 · Thuận) · Rubric 4.5, 4.2 · trả lời Q10
- **[R] Requirement** (đề bài: p95 < 3s, 5–8× peak, không lộ đáp án, consent, freemium) · **[A] Working assumption** (30% DAU × 8 lượt; thị trường VN; mix trả phí 10% sens. 5/15%; small model đủ chất lượng) · **[P] Architect proposal** (ALR < 1%, golden set ≥ 1,000, hybrid/AWS/pgvector/BKT, ≤ $0.0007/lượt) · **[D] Data/training prerequisite** (≥ 50k lượt nhãn trước SFT 8B; ladder duyệt ≥ 80% bài top-traffic).
- Full register: ATAD Appendix E. Mọi ADR có Status Accepted/Proposed.
