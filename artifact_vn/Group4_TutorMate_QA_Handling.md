# TutorMate — Panel Q&A Handling Guide

**Group 4 · Scenario B — phân công Q&A**

| Mã | Thành viên | Vai trò | Chủ trì câu hỏi |
|---|---|---|---|
| M1 | Trầm Quốc Thuận | Requirements + traceability + integration | Scope, requirement, trade-off (4.5) |
| M2 | Phạm Thị Thanh Huyền | Logical architecture + data/state flow | 4.2 Architecture, C4, ADR, data |
| M3 | Nguyễn Hòa | AI/ML Lead | 4.1 AI/ML, model, RAG, deterministic tools |
| M4 | Trần Thanh Phụng | Responsible AI Lead | 4.4 RAI, safety, privacy trẻ em, leakage |
| M5 | Lê Nguyễn Sỹ Bình | Cloud & Scale Lead | 4.3 Cloud, latency, ops/rollout |
| M6 | Trần Trọng Phú | Evaluation & Cost Lead | Eval, ALR, cost, risk |
| M7 | Đinh Xuân Dũng | Slide + Q&A + rehearsal Lead | Moderator, điều phối Q&A |


Mục đích: chuẩn bị trả lời câu hỏi panel theo **5 tiêu chí rubric**, đúng "altitude" (kiến trúc sư: quyết định → lý do → trade-off → bằng chứng), không sa vào chi tiết code. Mỗi câu có: **Trả lời 30 giây** (nói trước), **Đào sâu** (khi panel hỏi tiếp), **Bằng chứng trong ATAD**, **Bẫy cần tránh**.

## 0a. Quy tắc trả lời chung (Rubric 4.5 – 10%)
1. **Cấu trúc 4 bước:** *Quyết định → Vì sao (NFR/ràng buộc nào thúc đẩy) → Trade-off đã chấp nhận → Số/bằng chứng.* Tối đa 45 giây, rồi dừng, hỏi "panel muốn tôi đi sâu phần nào?"
2. **Phân công:** Dũng (M7) nhận câu hỏi, nói 1 câu định vị ("đây là câu về cost, Phú trả lời") rồi chuyển. Không 2 người cùng nói. Người bổ sung chỉ nói khi được Dũng mời ("Bình bổ sung 1 điểm về eval").
3. **Không biết thì nói không biết + cách sẽ tìm ra:** "Chúng tôi chưa đo được X; kế hoạch là đo bằng Y trong beta 2.000 học sinh, ngưỡng quyết định là Z." Panel chấm cách tư duy, không chấm sự toàn tri.
4. **Không tranh cãi số giá model:** mọi giá là *giả định ±30%*; kiến trúc được thiết kế để giá đổi thì flip alias, không đổi kiến trúc. Chuyển câu hỏi giá thành câu hỏi *levers*.
5. **Luôn quay về Removal Test & ALR:** hai thứ panel Scenario B chấm nặng nhất là AI-First thật và "đo được không lộ đáp án".
6. **Câu hỏi bẫy "vì sao không X":** thừa nhận X có ưu điểm → nêu tiêu chí X thua (kèm trọng số) → nêu điều kiện sẽ đổi sang X (reversibility). Đừng nói X "sai".

---

## 0b. Mười câu hỏi "pass/fail" của workshop — câu trả lời 30 giây

| # | Câu hỏi | Trả lời 30 giây | Bằng chứng (slide / ATAD) | Người |
|---|---|---|---|---|
| 1 | TutorMate khác ChatGPT ở đâu? | ChatGPT tối ưu "trả lời đúng"; TutorMate tối ưu "câu hỏi kế tiếp để em tự làm được". Khác biệt nằm ngoài LLM: Dialogue Policy quyết định mức hé lộ, LLM runtime không bao giờ nhận đáp án, CAS xác minh đúng/sai, BKT nhớ mastery thay vì chat log, guard 2 chiều domain-locked cho trẻ em. ChatGPT không cho phép chứng minh bất kỳ điều nào trong số đó. | Slide 2 · §1.1, §1.7, §3.1 | Hòa |
| 2 | Socratic enforce thế nào? | 5 bước, LLM chỉ ở bước 4: (1) CAS Grader chấm bước; (2) Policy state machine chọn action — advance / explain / raise level / reveal_allowed, rule level+1 chỉ sau ≥ 2 lỗi khác nhau, reveal sau ≥ 90s & 12 phiên; (3) prompt chỉ chứa ladder ≤ level; (4) LLM T1 diễn đạt; (5) Output Guard cắt nếu lộ. Policy unit-test được như code; ALR đo được. | Slide 4, 3 · ADR-001, App. A/B | Huyền / Hòa |
| 3 | Personalization lưu gì / không lưu gì? | Lưu: `learner_id` (HMAC pseudonym), grade_band, vector P(mastery)/skill (~200 số, BKT), 3 flag cờ (misconception, ngôn ngữ, pace). Không lưu: tên/tuổi/lớp trong tutor plane, hội thoại thô quá 30 ngày, ảnh/giọng, hồ sơ hành vi để quảng cáo. PII chỉ ở in-country zone; xoá = cắt mapping. | Slide 4 · ADR-002, §5.7 | Huyền / Phụng |
| 4 | Curriculum grounding / RAG? | Không RAG trên lời giải. Corpus = Hint Ladder (bậc 0..k) + misconception + concept card, sinh offline bằng T3, CAS check, giáo viên duyệt 100%, tag chapter/skill khớp curriculum EduNova, versioned (`ladder_version`). Runtime retrieve theo exercise_id (exact) + pgvector HNSW cho concept card; đáp án cuối nằm trong Answer Vault riêng. | Slide 5 · §5.3, §5.4 | Hòa |
| 5 | Ngăn harmful & answer leakage? | Defense-in-depth 3 lớp mỗi chiều: deterministic (CAS-leak vs Vault, regex PII, allowlist topic, < 5ms) → classifier int8 tự train (jailbreak / answer-fishing / off-topic / toxicity, < 60ms) → managed safety API backstop. Streaming guard theo câu; vi phạm → cắt & hint tĩnh. Self-harm → protocol SLA 15 phút. Đo bằng G-Leak 2k (12 họ tấn công) & G-Safety 1.5k, gate 0 harmful, ALR ≤ 1.0% golden set / ≤ 0.5% production. | Slide 6 · ADR-004, §6.5, §11.5 | Phụng / Hòa |
| 6 | p95 < 3s bằng cách nào? | Latency budget 10 bước cộng lại 2.6s p95 (staging 8×): ladder cache Redis 60ms, guard song song 120ms, CAS 20ms, TTFT T1 500ms, stream + guard theo câu 1.2s; BKT/judge async. 12% lượt templated < 300ms không gọi LLM. Nếu p95 > 2.5s: alert; > 3.5s 10 phút → auto-rollback/T2 cap. | Slide 8 · §2.4, §10.6 | Bình |
| 7 | Scale 5–8× peak? | Stateless orchestrator HPA theo RPS; guard KEDA theo queue; managed inference co giãn + quota 3× + đa nhà cung cấp; Redis cache ladder 95% hit; Postgres read replica; Kafka buffer. Peak ~89 turns/s (8×) đã load-test staging; break point 10× ~111 turns/s. Degraded: Static Ladder Mode (không LLM) vẫn giữ SLO 99.5%. Break point 10× ở provider TPM quota & Postgres write — có kế hoạch. | Slide 8 · §2.3, §10.7 | Bình |
| 8 | Metrics đánh giá? | Offline: ALR (CAS + clf), Pedagogy Score (judge T3 5 tiêu chí, κ ≥ 0.7 vs giáo viên), factual error, harmful pass-through, grader accuracy ≥ 97%, latency, cost/turn — trên 5 golden set, theo slice khối/môn/hành vi/mạng yếu. Online: next-day same-skill accuracy, abandonment trong tutor, ticket "chỉ cho đáp án", judge 3% sampling, human 200/tháng. | Slide 7 · §6, §7 | Phú |
| 9 | Cost model & drivers? | 29M lượt/tháng × blended $0.00069 ≈ $18.4k AI vs budget 8.000×P (10% mix × 20% doanh thu) → viable khi P ≥ ~$2.30/tháng. Drivers theo thứ tự: (1) tỉ lệ lượt T2 (10% → 5% tiết kiệm $3.6k); (2) prompt cache hit; (3) số lượt/HS free (quota 20 → 15); (4) input tokens (history 6 lượt); (5) judge sampling. Levers có sẵn qua Gateway config, không đổi code. | Slide 9 · §10 | Phú / Dũng |
| 10 | Requirement / Assumption / Proposal phân biệt rõ? | Có: [R] Requirement = đề bài đưa (p95 < 3s, 5–8× peak, không lộ đáp án, consent); [A] Working assumption = nhóm giả định, xác nhận ở beta (30% DAU × 8 lượt, thị trường VN, mix trả phí 10%); [P] Architect proposal = nhóm đề xuất, có ADR, panel có thể thách thức (ALR < 1%, golden set ≥ 1,000, hybrid/AWS/BKT); [D] Data/training prerequisite = điều kiện bật (≥ 50k lượt nhãn trước SFT 8B). Mọi ADR có Status Accepted/Proposed; không phát biểu nào được nói như 'đã đo trong production'. | Appendix D · ATAD Appendix E | Thuận |

## 1. AI/ML Technical Expertise (30%) — chủ trì: Hòa (M3), hỗ trợ Phú (M6)

### Q1.1 Vì sao chọn hybrid thay vì fine-tune một model cho phong cách Socratic?
- **30s:** Fine-tune là công cụ khoá *phong cách*, không phải công cụ bảo đảm *không lộ đáp án*; và ngày 1 chúng tôi có ~0 dữ liệu hội thoại thật. Fine-tune trên synthetic ngay từ đầu = học lỗi của model lớn. Nên MVP dùng deterministic policy + Hint Ladder + model nhỏ diễn đạt; fine-tune (SFT/DPO) là **ADR-005 Phase 2** khi có ≥ 50k lượt thật gắn nhãn.
- **Đào sâu:** Fine-tune vẫn có thể lộ đáp án nếu đáp án trong context; cái bảo đảm là least-privilege (ADR-001). Khi fine-tune, lợi ích là giảm chi phí (self-host 8B) và ổn định tone; điều kiện go: ALR/Pedagogy self-host ≥ managed trên golden set.
- **ATAD:** §3.1, ADR-001, ADR-005. **Bẫy:** đừng nói "fine-tune không hiệu quả" chung chung; nói *thời điểm* và *mục đích*.

### Q1.2 Tại sao không dùng RAG trên lời giải mẫu — đơn giản và đúng kiến thức?
- **30s:** Vì đưa lời giải vào context là *mời* LLM lộ đáp án; baseline nội bộ prompt-only + lời giải trong context cho ALR 8–15% dưới tấn công nhiều lượt. Chúng tôi vẫn RAG — nhưng trên **Hint Ladder** (câu hỏi gợi mở theo bậc, không chứa đáp án cuối) sinh offline và giáo viên duyệt.
- **Đào sâu:** Ladder cũng là "RAG có cấu trúc": truy vấn exact theo `problem_id`, vector search chỉ cho misconception matching. Đáp án tồn tại duy nhất trong Grader (CAS) và Output Guard.
- **ATAD:** §3.1, §5.3, ADR-001.

### Q1.3 Làm sao ép LLM "kiên trì gợi mở"? Prompt thôi thì ai tin?
- **30s:** Không dựa vào prompt. 4 lớp: (1) **Cấu trúc**: model không có đáp án; (2) **Policy deterministic** quyết định mức hé lộ — LLM không có quyền quyết; (3) **Output Guard** CAS + classifier paraphrase cắt stream khi lộ; (4) **Eval**: G-Leak 2.000 ca tấn công là gate CI, ALR đo 100% lượt bằng CAS trong prod.
- **Đào sâu:** LLM vẫn có thể *tự giải* rồi nói ra → bắt bằi CAS equivalence với Answer Vault (số/biểu thức) và classifier với paraphrase ("gần gấp đôi số bạn vừa tính"). Không xác nhận đáp án qua yes/no → chặn answer-fishing.
- **ATAD:** §2.2 nguyên tắc, §2.4 bước 5–8, ADR-004, §6.5.

### Q1.4 Hint Ladder quá cứng — học sinh giải cách khác thì sao?
- **30s:** Thừa nhận đây là trade-off của ADR-001. Giảm bằng: ladder có nhiều nhánh cho cách giải phổ biến (T3 sinh 2–3 approach khi teacher đánh dấu), misconception bank, và **T2 escalation** khi Grader trả `irrelevant` 2 lần (em đang đi hướng khác). Theo dõi metric "lạc khỏi ladder" — nếu > 10% với một bài thì regen ladder.
- **Đào sâu:** Học sinh đi hướng khác *đúng* → CAS vẫn xác nhận bước đúng vì so *tương đương toán học*, không so chuỗi. **ATAD:** ADR-001 consequences, §6.2 slices, R-08.

### Q1.5 Tại sao chấm đúng/sai bằng SymPy thay vì LLM?
- **30s:** Tool-fit: đúng/sai của biểu thức là bài toán *đại số*, không phải *ngôn ngữ*. CAS xác định, < 20ms, không hallucinate, gần $0. LLM chỉ dùng để *diễn giải* lỗi (vì sao sai) dựa trên misconception_id mà Grader trả về.
- **Đào sâu:** Tự luận Lý/Hoá không thuần biểu thức → step-alignment: T1 map câu của em vào bước ladder (classification), CAS kiểm phần định lượng; grader accuracy target ≥ 97% trên G-Grade. **ATAD:** §1.1, §3.1, §6.4.

### Q1.6 Learner model: vì sao BKT chứ không Deep Knowledge Tracing hay để LLM "nhớ"?
- **30s:** Ba tiêu chí: explainable cho phụ huynh (P(mastery) là xác suất có nghĩa), data minimisation (~200 số/em, không lưu chat), chi phí O(1). DKT hơn 2–4pt AUC nhưng cần GPU, khó giải thích → candidate Phase 3 nếu A/B chứng minh uplift học tập. LLM "nhớ" qua tóm tắt = lưu nội dung trẻ em + không đo được.
- **ATAD:** ADR-002, §5.7, §11.2.

### Q1.7 Cold-start: synthetic data có đáng tin không? Model lớn dạy sai thì sao?
- **30s:** Synthetic chỉ dùng để *khởi động eval & prompt*, không để fine-tune ngày 1. Quy trình: rubric với giáo viên → mô phỏng student-persona × tutor T3 → **giáo viên curate** 1.500 → judge calibrate κ ≥ 0.7 với giáo viên → beta 2.000 HS → thay 30% golden set bằng dữ liệu thật sau tháng 1 (R-11).
- **Đào sâu:** Kiến thức đúng không đến từ synthetic mà từ ngân hàng nội dung + CAS + teacher review ladder. Synthetic chỉ mô phỏng *hành vi học sinh*. **ATAD:** §6.1, R-11, slide 5.

### Q1.8 Model tier-1 nhỏ có đủ "thông minh" để dạy không?
- **30s:** Nó không cần dạy — ladder + giáo viên đã "dạy" offline. Model nhỏ chỉ diễn đạt bậc gợi ý hiện tại phù hợp câu trả lời của em. Khi cần suy luận (bí ≥ 2 lần, tự luận nhiều bước, guard vừa chặn) → escalate T2 (~10% lượt). Matrix model layer: small managed 4.55 vs frontier 3.30, chủ yếu vì cost (trọng số 20%) và latency.
- **ATAD:** §3.3, §7. **Bẫy:** đừng cam kết tên model cụ thể; nói "class" và alias.

### Q1.9 Embedding model & chunking chọn thế nào?
- **30s:** Corpus < 1M vectors; đơn vị tri thức đã có ngữ nghĩa (ladder step, misconception, concept card 200–400 token) nên không chunk overlap. Multilingual embedding (bge-m3 class) vì tiếng Việt + ký hiệu toán; version ghi kèm mọi vector; re-embed shadow khi đổi model. Truy vấn chính là exact theo `problem_id` — vector chỉ phụ.
- **ATAD:** §5.3, §5.4.

### Q1.10 LLM-as-judge có đáng tin để đo pedagogy?
- **30s:** Chỉ tin sau khi calibrate: ≥ 300 nhãn giáo viên, Cohen κ ≥ 0.7 mới dùng; rubric 5 tiêu chí reference-guided; human eval 200/tháng và bắt buộc khi đổi model/policy. ALR số/biểu thức không dùng judge — dùng CAS.
- **ATAD:** §6.3, §6.6.

### Q1.11 Sinh bài luyện tập mới — làm sao bảo đảm bài sinh ra đúng?
- **30s:** Template-based + LLM T2 tạo biến thể nhắm misconception vừa gặp, **CAS verify** lời giải, template mới qua teacher QA trước khi vào pool; async, không nằm trên đường nóng. **ATAD:** §1.1, §5.2, G-Practice.

### Q1.12 Voice có làm được không?
- **30s:** Ngoài scope MVP theo đề. Nếu làm: STT/TTS cộng ~600–900ms và ×3–5 chi phí/lượt → chỉ khả thi cho paid tier hoặc khi TTS on-device. Đề xuất Phase 3 với gate cost/latency riêng. **ATAD:** slide 10, Appendix D.

---

## 2. Architecture Design & System Thinking (25%) — chủ trì: Huyền (M2), hỗ trợ Thuận (M1)

### Q2.1 Vì sao dialogue policy là state machine tự viết thay vì LangGraph / agent framework?
- **30s:** Policy phải *audit & unit-test như code* (100% branch) vì nó quyết định mức hé lộ — thứ panel & phụ huynh cần tin. < 1.000 dòng, không cần framework. LangGraph là fallback nếu luồng phức tạp hơn (điểm 3.95 vs 4.80). Managed agent platform loại vì lock-in (1/5).
- **ATAD:** §3.4, Appendix A.

### Q2.2 Latency p95 < 3s: cộng lại thế nào và đâu là rủi ro?
- **30s:** Budget: guard 120ms ∥, retrieval 20ms (cache 95%), TTFT 500ms, stream 150 token ~1.2s, guard theo câu chạy song song stream, ghi async → **2.6s p95 đo staging 8×**. Rủi ro: TTFT provider giờ cao điểm → failover; T2 chậm hơn → cap 10%; mạng client (RTT > 400ms) → slice riêng target 3.5s.
- **ATAD:** §2.4, §6.2, §10.6.

### Q2.3 Streaming mà vẫn kiểm output — mâu thuẫn không?
- **30s:** Kiểm theo *câu*: deterministic < 5ms + classifier int8 < 60ms mỗi câu; câu chỉ được đẩy xuống client sau khi pass. Vi phạm → cắt, leak-repair 1 lần, rồi hint tĩnh. UX "đang nghĩ lại…". **ATAD:** ADR-004.

### Q2.4 Điều gì xảy ra khi LLM provider chết lúc 20h mùa thi?
- **30s:** Gateway failover sang provider B (khác nhà, đã cấu hình, quota dự phòng); cả hai chết → **Static Ladder Mode**: học sinh vẫn nhận gợi ý theo bậc, không hội thoại. Availability tính degraded là "up" 99.5%, LLM-mode 99.0%. Playbook #3. **ATAD:** §9.2, R-06.

### Q2.5 Kiến trúc có over-engineered cho MVP không (Kafka, lakehouse, Flink)?
- **30s:** Thừa nhận có thể bắt đầu gọn hơn: MVP Phase 0 có thể dùng Postgres + Redis + managed queue, Kafka/Iceberg vào khi 1M lượt/ngày cần eval-in-prod và lineage. Cái *không* cắt được: Gateway, Guard 2 chiều, Ladder, Policy, eval harness — vì gắn với NFR cứng. **ATAD:** §2.2; nói rõ đâu là "ngày 1" đâu là "khi scale".

### Q2.6 Tích hợp với EduNova Core App thế nào? Ai sở hữu learner model?
- **30s:** Contract Avro `content.problem.v2`, `behavior.attempt.v1` (schema registry + Great Expectations); TutorMate ghi mastery về Core qua Kafka topic; Core sở hữu danh tính, TutorMate sở hữu learner model pseudonymous. **ATAD:** §5.6.

### Q2.7 Removal test — nói thật đi, bỏ AI còn gì?
- **30s:** Còn app trắc nghiệm có lời giải (thứ phụ huynh phàn nàn) + Hint Ladder tĩnh. Ladder tĩnh hữu ích hơn app cũ nhưng không đối thoại, không thích nghi theo lỗi cụ thể của em — không phải gia sư. Chúng tôi cố ý biến "phần còn lại" thành chế độ kill-switch có kiểm soát. **ATAD:** §1.8, §9.3.

### Q2.8 Session state đặt ở đâu? Mất Redis thì sao?
- **30s:** Redis (TTL 24h) cho state; mọi transition cũng ghi Kafka → rebuild được. Mất Redis: orchestrator tái tạo state từ 6 lượt cuối trong client + ladder (client giữ bản copy nhẹ) hoặc bắt đầu lại bậc gần nhất. **ATAD:** §2.4, R-12.

### Q2.9 Diagram C4 L3 của các bạn có gì đặc biệt so với web app thường?
- **30s:** Ba điểm AI-specific: (1) pseudonymisation boundary giữa in-country PII zone và tutor plane; (2) node pool riêng cho guard (KEDA) và GPU scale-to-zero (Phase 2); (3) offline plane (enrichment, eval, fine-tune) tách khỏi runtime với spot. **ATAD:** §2.3.

---

## 3. Cloud & Infrastructure Expertise (20%) — chủ trì: Bình (M5)

### Q3.1 Vì sao AWS mà không GCP — Gemini Flash-Lite rẻ nhất mà?
- **30s:** Hai bên hoà 4.15/5; chọn theo team capability và catalogue managed inference rộng (nhiều nhà model trong 1 endpoint). Gemini Flash-Lite vẫn gọi được qua LiteLLM cross-cloud nếu rẻ hơn — hạ tầng và model là hai quyết định độc lập. Mọi thành phần K8s/Postgres/Kafka/S3-compatible → đổi cloud = IaC. **ATAD:** §3.6, ADR-003.

### Q3.2 GPU: tại sao không self-host từ đầu để rẻ và không lock-in?
- **30s:** Làm phép nhân: 2.6B token/ngày; L4 ~2k tok/s → ~15 GPU baseline, ~120 lúc đỉnh 8×; chi phí ≈ managed small model ($8–9k/tháng) nhưng thêm GPU ops, on-call, autoscale chậm hơn managed. Chưa đáng ở MVP; ADR-005 Phase 2 khi fine-tune xong và volume ổn định — khi đó lợi ~40% T1. **ATAD:** §3.3 (C), ADR-005, slide 8, Appendix B.

### Q3.3 Đỉnh tải 5–8× xử lý thế nào? Có autoscale kịp không?
- **30s:** Orchestrator stateless → HPA theo RPS (scale trong ~60s), pre-scale theo lịch (17h, mùa thi); guard pool KEDA; LLM managed đàn hồi + quota 3× + multi-provider; Redis cache ladder 95% giúp DB không thấy peak. Load test 8× là gate staging. **ATAD:** §2.3, §6.6, §10.7.

### Q3.4 Serverless hay container?
- **30s:** Container cho đường nóng (guard, orchestrator) vì cần warm & p95; serverless cho async (practice generation, DSAR, report). Không serverless cho guard vì cold-start int8 model 2–5s phá p95. **ATAD:** slide 8, Appendix B.

### Q3.5 Chi phí ~$18.4k — viable chỉ khi P ≥ ~$2.30?
- **30s:** Đúng — đó là bản chất freemium và P là biến chưa giả định; budget = 8.000×P nên viable khi P ≥ ~$2.30 (AI variable) hoặc ~$3.70 (gồm infra). Có 6 lever theo thứ tự (T2 cap → cache → templated → quota → history → self-host) với tổng dư địa ~$10k; alert 80/100/110% daily spend; hard stop 130% đưa free tier về static mode. Sensitivity: mix 5% → P ≥ ~$4.60. Sensitivity giá ±30% ở backup B3. **ATAD:** §10.3, §9.4 governance, R-05.

### Q3.6 Cost per query p95 $0.0028 gấp 5× typical — sao chấp nhận?
- **30s:** p95 là lượt T2 (bí thật, tự luận) — đúng lúc giá trị sư phạm cao nhất; cap 10% lượt nên đóng góp blended chỉ $0.00028. Nếu T2 ratio > 12% → warn, > 18% → page và tự giảm cap. **ATAD:** §10.2, §9.1.

### Q3.7 Data residency: PII trẻ em đặt đâu nếu cloud không có region trong nước?
- **30s:** Tách: identity/consent store trong nước (DC/partner tuân luật), tutor plane region gần (SEA) chỉ nhận `learner_id` HMAC + grade band; không PII ra ngoài; LLM nhận prompt đã redact với zero-retention. Nếu luật siết hơn → toàn bộ tutor plane K8s-portable sang DC trong nước. **ATAD:** §2.3, §5.7, R-04.

### Q3.8 Break point 10× đầu tiên là gì?
- **30s:** Quota TPM/RPM provider → signal 429 ở gateway trước khi latency tăng → multi-provider/provisioned throughput/self-host. Kế đến Postgres write path (mastery + log) → Kafka lag → micro-batch BKT, bỏ ghi trực tiếp. Rồi guard pool. **ATAD:** §10.7.

### Q3.9 Mạng yếu — kiến trúc phía client làm gì?
- **30s:** SSE resume token, state server-side (Redis 24h), offline queue câu trả lời, payload gzip < 300KB, Capacitor/PWA cho máy yếu, degraded static mode. Slice eval riêng cho RTT > 400ms. **ATAD:** §2.3, §6.2, R-12.

### Q3.10 DR/RTO/RPO?
- **30s:** Warm standby region thứ hai: K8s min replicas, Postgres async replica; RTO 30 phút, RPO 5 phút; ladder & prompt artifacts trên object store replicate; PII zone không replicate ra ngoài nước (DR trong nước). **ATAD:** §2.3.

---

## 4. Responsible AI & Governance (15%) — chủ trì: Phụng (M4)

### Q4.1 EU AI Act: TutorMate là high-risk không?
- **30s:** Giáo dục thuộc Annex III khi hệ thống *đánh giá kết quả học tập hoặc điều hướng quá trình học*. Tutor gợi mở thuần túy có thể ở limited-risk, nhưng learner model điều chỉnh độ khó → ảnh hưởng lộ trình. Với trẻ em, chúng tôi **chủ động áp posture high-risk**: risk mgmt (§8), data governance (§5), logging (§9), human oversight (§11.4), Art. 50 disclosure. Rẻ hơn bị phân loại lại sau. **ATAD:** §11.1.

### Q4.2 Parental consent làm thế nào ở quy mô 400k?
- **30s:** Consent ledger với scopes (tutor, retain_chat_30d, analytics), xác thực phụ huynh qua Core App (email/thanh toán), kiểm tra tại Gateway mỗi phiên (freshness < 5 phút); thiếu scope → không mở tutor; rút consent tự phục vụ → TTL chạy. **ATAD:** §5.6, §5.7.

### Q4.3 XAI cho LLM — SHAP/LIME áp dụng được không?
- **30s:** Không cho LLM sinh văn bản. Chúng tôi dùng *process transparency* cho học sinh, *learner-model transparency* (BKT là xác suất tự giải thích) cho phụ huynh, *attribution* (ladder_version, misconception, judge rationale) cho giáo viên; SHAP áp dụng cho **classifier guard** và feature importance cho BKT. **ATAD:** §11.2.

### Q4.4 Bias: các bạn không thu giới/vùng thì test fairness kiểu gì?
- **30s:** Bằng *persona synthetic* (paired dialogues nam/nữ, phương ngữ Bắc/Trung/Nam, viết không dấu) và slice có sẵn (học lực, thiết bị). Threshold gap ≤ 0.2–0.3 là gate. Không thu attribute thật vì data minimisation trẻ em. **ATAD:** §11.3.

### Q4.5 Prompt injection: học sinh gõ "ignore previous instructions" thì sao?
- **30s:** User text không bao giờ vào system role; ladder sinh offline từ nội dung đã duyệt; Input Guard classifier injection; runtime **tool-less** (không tool call → không side-effect); output JSON schema enforced; và dù bị "thuyết phục", model không có đáp án để lộ. **ATAD:** §11.5.

### Q4.6 Self-harm / bạo hành xuất hiện trong chat — làm gì?
- **30s:** Lớp 3 managed safety API + classifier bắt; policy chuyển sang script an toàn (đường dây hỗ trợ, không tiếp tục bài), HIL safety officer SLA 15 phút, thông báo phụ huynh theo quy định; ca được review 100%. **ATAD:** §11.4, R-02.

### Q4.7 Risk register: threshold có gắn với vận hành thật không?
- **30s:** Mỗi threshold là alert trong §9.1: ALR > 1.5%/1h freeze, > 3% rollback; ≥ 1 harmful xác nhận → kill-switch; cost > $0.0009/turn → giảm T2 cap. Register review quý bởi RAI board. **ATAD:** §8 ↔ §9.1.

### Q4.8 Model extraction — ai đó cào toàn bộ Hint Ladder?
- **30s:** Rate limit theo learner (20/200 lượt/ngày), anomaly (> 200 bài/giờ), không API public, watermark wording ladder để truy vết. Rủi ro chấp nhận ở mức trung bình (R-10) vì ladder không phải đáp án. **ATAD:** R-10, §11.5.

### Q4.9 Gửi dữ liệu trẻ em cho LLM provider có vi phạm không?
- **30s:** Không gửi PII: redact trước, chỉ `learner_id` không có trong prompt, nội dung là toán/khoa học; contract zero-retention/no-training; private endpoint; egress allowlist; DPIA ghi nhận. **ATAD:** §5.7, §11.5.

### Q4.10 Học sinh có biết đang nói với AI không?
- **30s:** Có — disclosure rõ trong UI (Art. 50), persona không giả người thật, cuối phiên có summary AI làm gì. **ATAD:** §11.1.

---

## 5. Eval & LLMOps (cắt ngang 4.1/4.2) — chủ trì: Phú (M6, eval/cost) & Bình (M5, ops/rollout)

### Q5.1 Làm sao biết ALR sau khi đổi prompt/model?
- **30s:** CI chạy full G-Leak + G-Safety và judge 500 mẫu; pass → canary 5% với sampling judge 20% 24h; prod: 100% lượt qua CAS-ALR, 3% judge; dashboard theo `prompt_version × model_id × slice`; auto-freeze/rollback theo ngưỡng. **ATAD:** §6.6, §6.7, §9.

### Q5.2 Provider đổi model ngầm dưới cùng tên — phát hiện thế nào?
- **30s:** Daily shadow canary 300 mẫu golden cố định → Δ Pedagogy/ALR > 0.3 so baseline 7 ngày → alert + pin version (nếu có) / failover. **ATAD:** R-07, §6.7.

### Q5.3 Rollback mất bao lâu và gồm gì?
- **30s:** ≤ 5 phút: flip alias/prompt pointer trong gateway (config, không deploy) → invalidate cache → verify ALR/latency 10 phút → incident. Ladder rollback per `ladder_version`. **ATAD:** §9.3.

### Q5.4 Ngưỡng ALR 1% — vì sao không 0%?
- **30s:** 0% không đo được với hệ sinh; gate ≤ 1.0% trên adversarial golden set, production sampled ≤ 0.5%; Phase 2 cân nhắc siết 0.3%. Nếu panel cho rằng cần chặt hơn → lever: T2 cho lượt có dấu hiệu gạ, tăng leak-repair. **ATAD:** §1.4, slide 6, 7.

### Q5.5 Trigger fine-tune / retrain là gì?
- **30s:** Ladder: bài mới/≥ 3 teacher fix; misconception: weekly mining; guard clf: monthly hoặc > 5 FN prod; BKT: quý; T1 fine-tune: ≥ 50k lượt nhãn hoặc Pedagogy drift. Feedback loop: prod → judge → teacher queue → nhãn → golden/DPO set. **ATAD:** §9.4.

### Q5.6 Judge chấm judge — vòng lặp tự sướng?
- **30s:** Judge được neo vào giáo viên (κ ≥ 0.7, re-calibrate tháng), human eval 200/tháng độc lập, và outcome thật (next-day accuracy A/B) là thước đo cuối. **ATAD:** §6.3.

---

## 6. Câu hỏi "sát thủ" & cách đỡ

| Câu hỏi | Đỡ trong 1 câu | Rồi chuyển sang |
|---|---|---|
| "Đây có phải chỉ là chatbot với prompt dài?" | Không — LLM không có đáp án, không quyết mức hé lộ, không quyết an toàn; 3 thứ đó nằm ngoài LLM. | Slide 2, 3, ADR-001 |
| "Nếu tôi là học sinh, tôi gạ được trong 3 lượt." | Mời thử với G-Leak; cơ chế: không xác nhận yes/no, cần ≥ 2 lỗi khác nhau + 90s mới reveal. | §6.5, Appendix A/C |
| "Chi phí các bạn dựa trên giá model hôm nay." | Đúng, ±30%; kiến trúc hấp thụ bằng alias + 6 lever; nếu giá tăng 30% → kéo lever 1–3 đủ. | §10.3 |
| "Phụ huynh kiện vì AI dạy sai." | Kiến thức đến từ ladder có CAS check + teacher review, không từ LLM; factual error gate 0.5%; teacher hotfix ladder; log tuple để truy vết. | R-03, §5.3 |
| "Sao không mua giải pháp sẵn (Khanmigo-class)?" | Đề là AI-First từ đầu với dữ liệu & nội dung EduNova; build core (policy, ladder, guard), buy commodity (LLM, safety API, o11y). | §3 |
| "Team có làm nổi không?" | MVP Phase 0 = 6 tuần với 5–6 người; phần nặng là content enrichment (giáo viên) không phải code. | Slide 10, Appendix D |
| "Điều gì khiến các bạn mất ngủ?" | R-11 cold-start eval không phản ánh học sinh thật → beta 2.000 HS và thay 30% golden set sớm. | §8 |

---

## 7. Checklist trước khi bảo vệ
- [ ] Mỗi người thuộc 3 con số chính: **29M lượt/tháng · $0.00069/lượt · ~$18.4k vs 8.000×P (P ≥ ~$2.30)**; **ALR ≤ 1.0% golden / ≤ 0.5% prod**; **p95 2.6s**.
- [ ] Dũng (M7) thuộc bảng phân công Q&A và câu chuyển tiếp.
- [ ] Appendix A–D (slide 11–14) mở sẵn; không present mặc định.
- [ ] Demo 1 ví dụ hội thoại 4 lượt (bậc 0 → 2) và 1 ví dụ tấn công bị chặn (Appendix C) — đọc 30 giây nếu panel hỏi "cho xem thử".
- [ ] Không nói tên model cụ thể như cam kết; nói "class" + alias.
- [ ] Kết mỗi câu bằng: "Panel muốn tôi đi sâu phần nào?"
