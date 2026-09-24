# TutorMate — Kịch bản thuyết trình (Speaker Script)

**Group 4 · Scenario B** · 10 slide core · mục tiêu **13:00–13:30 nói + ~1:30 buffer = ≤15:00**
Quy ước: *(chữ nghiêng trong ngoặc)* = chỉ dẫn sân khấu, không đọc to. **[IN ĐẬM]** = câu chốt phải nói rõ, chậm. Nhịp đọc chuẩn ~140 từ/phút tiếng Việt.

---

## Slide 1 — Mở đầu: Problem & AI-First thesis
**PIC: Thuận · 0:45 · ~105 từ**

*(Bước ra, nhìn panel, không nhìn slide)*

Chào ban giám khảo. Chúng em là Group 4, và đây là TutorMate.

*(chỉ slide)* EduNova đang có 400.000 học sinh cấp hai học Toán và Khoa học. Pain point rất cụ thể: khi một em kẹt bài, app hiện tại đưa luôn lời giải — và lời giải sẵn **không dạy cách suy luận**. Kẹt bài, xem đáp án, bỏ cuộc.

TutorMate làm ngược lại: **dẫn dắt học sinh đi tới đáp án — thay vì đưa đáp án.** *(chỉ vòng lặp bên phải)* Kẹt bài → gợi mở → em tự làm ra → learner model cập nhật, lượt sau cá nhân hoá hơn.

Và đây là AI-First thật: bỏ AI đi, TutorMate chỉ còn lại app trắc nghiệm cũ.

*(chuyển slide — nhường Hòa)*

---

## Slide 2 — TutorMate ≠ ChatGPT + prompt
**PIC: Hòa · 1:00 · ~140 từ**

*(nhận lời tự nhiên)*

Câu hỏi đầu tiên panel thường hỏi: khác ChatGPT ở chỗ nào? Bảng này trả lời.

*(đi từng hàng, nhịp nhanh)* ChatGPT tối ưu để **trả lời** — TutorMate tối ưu để **học**: hé lộ tối thiểu, câu hỏi kế tiếp. ChatGPT dùng kiến thức chung của model — chúng em grounding trên chương trình EduNova đã duyệt, có version. ChatGPT điều khiển bằng prompt — chúng em dùng **state machine tường minh**, test được như code. Và quan trọng nhất: output của ChatGPT đi thẳng tới người dùng — của chúng em **qua Response Validator trước khi tới học sinh**.

*(chỉ callout cuối, nói chậm)* Một chi tiết em muốn panel nhớ: **LLM runtime không bao giờ nhận đáp án.** Đáp án nằm trong Answer Vault, chỉ Grader và Validator đọc. Không có trong context thì không thể "bị gạ" mà lộ.

**Bottom line: LLM sinh ngôn ngữ gia sư; hệ thống kiểm soát pedagogy.**

*(chuyển slide — nhường Huyền)*

---

## Slide 3 — Kiến trúc mục tiêu
**PIC: Huyền · 1:45 · ~245 từ**

Đây là target architecture — cố tình chỉ khoảng mười box, mỗi box có đúng một lý do tồn tại.

*(trace luồng chính bằng tay/con trỏ, trái → phải)* Một lượt học đi như sau: học sinh gửi tin nhắn hoặc công thức → qua API gateway, xác thực, **parental consent gate** và quota → Input Safety chặn moderation, injection, off-topic → vào **Tutor Orchestrator**.

*(dừng ở Orchestrator)* Orchestrator là trái tim: nó sở hữu **tutoring state** và quyết định action — advance, tăng hint level, check, consolidate hay escalate — bằng logic deterministic, unit-test được. Nó kéo từ ba nguồn: Learner Profile chạy BKT, Hint Ladder qua pgvector, và Deterministic Tools như CAS/SymPy cho chấm công thức.

*(sang Model Gateway)* Orchestrator gọi LLM qua **Model Gateway** — tiering, fallback, cache, budget. Lưu ý: LLM chỉ nhận ladder đến level được phép và 6 lượt gần nhất — **không có đáp án**.

*(cổng cuối)* Output qua **Response Validator** — safety, answer-leak, grounding, age-appropriate — rồi mới tới học sinh. Cột bên phải là làn async: telemetry, evaluation sampling, monitoring, cost tracking.

**Nguyên tắc xuyên suốt: LLM không phải policy engine, cũng không phải system of record.** C4 đầy đủ ba cấp trong ATAD mục 2.

*(chuyển slide — Huyền tiếp tục)*

---

## Slide 4 — Pedagogy enforce + Personalization giới hạn
**PIC: Huyền · 1:45 · ~240 từ**

Hai câu hỏi lớn của đề bài — "dạy Socratic thật sự thế nào" và "cá nhân hoá mà không thu thập dữ liệu trẻ em" — đều nằm ở slide này.

*(bên trái — state machine)* Socratic không nằm trong prompt. Dialogue Policy là state machine: Orient → Elicit → Hint → Check → Consolidate → Escalate. Và rule quan trọng: **hint chỉ tăng một level sau ít nhất hai lỗi khác nhau**, được CAS chấm — đây là code, không phải câu nhắc trong prompt.

*(ví dụ, nói như kể chuyện)* Nhìn ví dụ `2x + 5 = 15`. Chatbot trả "x = 5" — học sinh học được zero. TutorMate ở state Elicit hỏi: *"Em nghĩ bước đầu tiên nên làm gì để bỏ cộng 5 ở vế trái?"* Em trả lời sai — "chia cho 2" — thì Check gắn misconception "thứ tự phép toán", đưa Hint level 1, **vẫn chưa lộ bước**. Đúng rồi thì Consolidate: *"Vì sao trừ 5 cả hai vế vẫn giữ đẳng thức?"* — rồi mới cập nhật mastery.

*(bên phải — bảng lưu/không lưu)* Personalization chỉ dùng **bounded learning signals**: P(mastery) theo skill, loại lỗi gần nhất, hint history — pseudonymous. **Không lưu** transcript dài hạn, không danh tính thật trong prompt, không hồ sơ tâm lý.

**Cá nhân hoá dựa trên tín hiệu học tập có giới hạn — không phải bộ nhớ không giới hạn.**

*(chuyển slide — nhường Hòa)*

---

## Slide 5 — Vì sao Hybrid: RAG + LLM + Tools
**PIC: Hòa · 1:30 · ~200 từ**

Đây là quyết định kiến trúc trung tâm — và là phần 30% rubric AI/ML nên em đi kỹ.

*(bảng trái)* Pattern của chúng em là **hybrid**: mỗi capability gán đúng công nghệ. Diễn đạt gợi mở — đó là bài toán ngôn ngữ, dùng LLM. Nhưng grounding curriculum — RAG trên Hint Ladder giáo viên duyệt, vì model không biết chương trình EduNova và ta cần version, audit. Chấm đúng-sai công thức — **CAS/SymPy, không LLM**: LLM tính toán không tin được và tốn token. Cập nhật mastery — **BKT**: giải thích được, ổn định. Kiểm soát sư phạm — state machine. Safety — Validator; không để model tự chấm chính mình.

*(bên phải — offline lane)* Hint Ladder được sinh offline: LLM lớn draft ladder từ nội dung EduNova → **giáo viên duyệt, tối thiểu hai reviewer** → version → index pgvector. Runtime chỉ retrieve tới level được phép.

*(hai callout cuối)* Không fine-tune trước vì corpus gia sư ≈ 0 và chương trình đổi liên tục — FT 8B là Phase 2 khi đủ 50 nghìn lượt có nhãn. Không agent vì flow gia sư hữu hạn — agent chỉ thêm latency, cost và rủi ro.

**Dùng LLM chỉ ở nơi bài toán thực sự là ngôn ngữ.**

*(chuyển slide — nhường Phụng)*

---

## Slide 6 — An toàn cho trẻ em: 4 lớp phòng thủ
**PIC: Phụng · 1:45 · ~240 từ**

Người dùng của chúng em là trẻ 11 đến 16 tuổi — nên safety không phải một filter gắn thêm, mà là **bốn lớp phòng thủ**.

*(đếm bằng ngón tay / chỉ từng lớp)* **Lớp một, trước model:** identity, parental consent gate, input moderation, injection detection, topic lock. **Lớp hai, trong khi dạy:** tools có giới hạn, retrieval chỉ trong corpus đã duyệt, token và turn budget — và nhắc lại, không có đáp án trong context. **Lớp ba, trước khi gửi:** Validator kiểm tra safety, grounding, age-appropriate, answer-leak — fail thì trả hint tĩnh. **Lớp bốn, sau khi gửi:** monitoring, red-team liên tục, canary, kill-switch về Static Ladder Mode.

*(dừng, nhìn panel — đây là câu quan trọng nhất bài nói)* **Output chưa validate không bao giờ tới học sinh.**

*(hai rủi ro)* Hai rủi ro lớn nhất: harmful content — mục tiêu zero pass-through trên red-team, self-harm thì dừng dạy, đưa hotline, báo phụ huynh. Và answer leakage — state machine cộng CAS check cộng leak classifier, ALR không quá 1% trên adversarial golden set, không quá 0.5% trong production sampling.

*(footer)* Về governance: map NIST AI RMF, ISO 42001, Nghị định 13 — và chúng em nói thẳng: **yêu cầu pháp lý theo từng thị trường vẫn là assumption cần legal xác nhận**, không claim compliance.

*(chuyển slide — nhường Phú)*

---

## Slide 7 — Evaluation: làm sao biết "dạy tốt"?
**PIC: Phú · 1:30 · ~205 từ**

Một gia sư AI mà không đo được chất lượng dạy thì không triển khai được. Chúng em đo **tám chiều**.

*(lướt bảng, nhấn 3–4 dòng)* Correctness — CAS cộng golden set, grader accuracy ≥ 98%. Pedagogy — LLM-judge được calibrate với giáo viên, kappa ≥ 0.7, điểm ≥ 4.0 trên 5. Leakage — ALR không quá 1% trên adversarial golden set, không quá 0.5% production. Safety — zero critical escape trên 300+ case red-team, false refusal dưới 3%. Còn grounding, adaptation, performance, cost đều có ngưỡng release riêng.

*(dữ liệu)* Dữ liệu đánh giá: **golden set tối thiểu 1.000 case** do giáo viên gán nhãn, một **frozen holdout** không ai đụng vào để chỉnh prompt, adversarial red-team, và 200 phiên human review mỗi tuần.

*(release gate — nói rõ)* **Release gate bốn điều kiện — quality, safety, latency, cost — phải đạt cả bốn mới promote:** shadow, rồi 5, 25, 50, 100%.

*(outcome)* Và cuối cùng, metric proxy chỉ là proxy: outcome thật đo bằng A/B — bỏ dở từ 35 xuống 22%, retention kiến thức 48 lên 60%, chuyển đổi 5 lên 7%.

*(chuyển slide — nhường Bình)*

---

## Slide 8 — Performance & Scale: p95 < 3s, chịu 5–8× peak
**PIC: Bình · 1:45 · ~240 từ**

Hai ràng buộc cứng của đề bài: p95 dưới 3 giây và chịu peak 5 đến 8 lần.

*(bảng tải trái)* Workload: 960 nghìn lượt mỗi ngày — trung bình khoảng 11 turns một giây, peak 5 lần khoảng 56, **design target là 89 turns/s ở 8×**, và chúng em load-test ở 10× tức khoảng 111. Nhấn mạnh: giả định 30% DAU × 8 lượt sẽ được **đo lại ở beta hai nghìn học sinh** — đây là assumption có kế hoạch validate.

*(giữa — scale)* Chiến lược: API và Orchestrator **stateless**, autoscale ngang bằng HPA/KEDA theo Kafka lag; session và profile ở managed state — Redis, Aurora read replica; retrieval scale độc lập; LLM tiering hai provider; evaluation hoàn toàn async.

*(phải — latency budget)* Budget 3 giây chia thế này: policy, profile, retrieval dưới 0.7 giây; LLM 1.5 đến 1.8 giây; validation cộng network dưới 0.5 giây — còn headroom. *(chân thành)* Và em ghi chú trên slide: **đây là budget kiến trúc, chưa phải benchmark đo thật.**

*(bottom line — nói chậm)* Điểm gãy đầu tiên khi scale **không phải tầng API của chúng em — mà là concurrency và latency của provider**. Khi đó degrade về Static Ladder Mode: học sinh vẫn được dạy, chỉ kém thích nghi.

*(chuyển slide — nhường Dũng)*

---

## Slide 9 — Cost model & vận hành production
**PIC: Dũng · 1:15 · ~175 từ**

*(phương trình)* Chi phí gọn trong một phương trình: **monthly AI cost = volume × cost/turn + shared infra**. Khoảng 29 triệu lượt nhân 0.00069 đô — **xấp xỉ 18.4 nghìn đô một tháng** cho phần AI, cộng infra khoảng 11 nghìn. Budget là 20% doanh thu với kịch bản 10% trả phí, tức 8.000 lần P — **viable khi P từ khoảng 2.3 đô một tháng trở lên**, và P là biến chúng em không giả định.

*(năm driver)* Năm cost driver đều có cơ chế kiểm soát: token — tiering, context limit, cache, và 12% lượt chặn sớm không gọi LLM; moderation — rule trước, API chỉ backstop; embedding offline một lần; retries và judge có quota; infra spot và scale-to-zero ngoài giờ. *(nhanh)* Cost alert ở 80% và kill ở 120% budget ngày.

*(cột phải — ops)* Vận hành: Model Gateway fallback T1 → T2 → provider 2 → Static Ladder; release manifest ký version model, prompt, ladder, policy; canary shadow đến 100%; **rollback dưới 5 phút** chỉ bằng đảo alias; kill switch tắt LLM-mode toàn cục hoặc theo cohort.

**Production-ready nghĩa là: fail an toàn, degrade có kiểm soát, và luôn nằm trong budget.**

*(chuyển slide — nhường Thuận)*

---

## Slide 10 — 5 quyết định & kết
**PIC: Thuận · 0:45 · ~110 từ**

*(đứng giữa, không chỉ slide — tóm lại bằng giọng chắc)*

Kết lại, năm quyết định panel cần nhớ: **một**, hybrid AI — LLM chỉ sinh ngôn ngữ; **hai**, Socratic state machine tường minh, test được; **ba**, learner profile tối thiểu, không PII trong prompt; **bốn**, safety và validation trước khi gửi; **năm**, eval-gated, scalable, cost-controlled.

*(dừng một nhịp — câu thesis)* **TutorMate không chỉ là một LLM chatbot. Đó là một nền tảng gia sư AI được kiểm soát — an toàn cho trẻ em, đo được, và vận hành được ở quy mô.**

*(nhìn panel)* Chúng em sẵn sàng cho phần Q&A. *(Dũng điều phối: 4.1 → Hòa · 4.2 → Huyền · 4.3 → Bình · 4.4 → Phụng · Eval/Cost → Phú · Scope → Thuận)*

---

## Ghi chú rehearsal (không đọc)

- Tổng script ≈ 1.900 từ ≈ **13:00–13:45** ở nhịp 140–150 từ/phút — đúng mục tiêu 12–15 phút. Nếu rehearsal vượt 14:00, cắt ví dụ chi tiết ở slide 4 và phần driver 2–5 ở slide 9 trước.
- Buffer 13:45–15:00 dành cho pause/chuyển slide/interruption — Dũng giữ giờ, hiệu lệnh ở mốc 8:30 (hết slide 6) và 11:45 (hết slide 8).
- Mỗi PIC tập **câu mở đầu và câu chốt in đậm** thuộc lòng; phần giữa được phép nói theo ý, không đọc script.
- Appendix A–D không present mặc định — chỉ bật khi panel hỏi sâu (theo `Group4_TutorMate_QA_Handling.md`).
