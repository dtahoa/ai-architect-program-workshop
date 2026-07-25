# Kịch bản chi tiết phần Q&A

## Variant B — AI Retrofit

> **Mục tiêu:** Sẵn sàng cho 10–15 phút hỏi đáp sau bài trình bày 7 phút.
> **Nguyên tắc:** Trả lời ngắn gọn, vững vàng, không phỏng đoán, không hứa suông. Nếu không biết, nói “cần kiểm chứng bằng replay” thay vì bịa số liệu.

## I. Chiến thuật tổng thể cho phần Q&A

### 1. Thái độ

- Điềm tĩnh, không phòng thủ quá mức. Đây là kiến trúc bảo thủ; chúng ta đang bảo vệ MERLIN, không xin phép phá vỡ nó.
- Trả lời trực tiếp câu hỏi trước, giải thích sau. Tránh lan man 30 giây mới vào câu trả lời chính.
- Dùng số liệu đã có; không dùng số liệu tưởng tượng. Nếu câu hỏi vượt ra ngoài design target, nói rõ đó là “giả định cần replay”.

### 2. Cấu trúc mỗi câu trả lời (3 bước)

- Khẳng định ngắn: “Câu trả lời là…” (1 câu).
- Bằng chứng / cơ chế: Dùng artifact tương ứng (A1, A2, A5…) để giải thích tại sao.
- Hành động / Fallback: “Nếu không đạt thì…” để chứng minh fail-open.

### 3. Những từ ngữ cấm trong Q&A

| Không nói | Nên nói |
|---|---|
| “AI quyết định số lượng đặt hàng” | “AI đề xuất; MERLIN quyết định” |
| “Real-time inventory” | “Nightly snapshot với calibrated interval” |
| “Hệ thống đã sẵn sàng production” | “Kiến trúc được chấp nhận cho shadow; production influence vẫn bị giới hạn” |
| “162 triệu forecast đã xử lý được” | “Production gate yêu cầu vượt qua ba lần test liên tiếp ở mức 162M” |
| “Kill switch xoá toàn bộ override table” | “Kill switch chỉ xoá exact unconsumed AI-owned rows” |
| “LLM dự báo nhu cầu” | “Statistical ML forecast; deterministic rules giới hạn quantity” |
| “AI có thể chạy lại lúc 03:30” | “03:30 chỉ dùng cho diagnosis, không rerun” |
| “MERLIN phụ thuộc AI” | “MERLIN không bao giờ phải chờ AI” |

## II. Kịch bản theo nhóm câu hỏi

### Nhóm 1: Ràng Buộc Thời Gian & Deadline

#### Câu 1.1 — Tại sao chọn 02:00 làm AI write close?

**Người hỏi dự kiến:** Operations Director, Engineering Lead
**Mức độ khó:** ★★☆

**Kịch bản trả lời**

> “02:00 hiện là proposed gate, chưa phải con số cố định. Nó chỉ hợp lệ nếu MERLIN P99 đo thực tế không quá 90 phút, để còn tối thiểu 30 phút reserve trước 04:00. Nếu MERLIN P99 lớn hơn, write close phải được chuyển sớm hơn và toàn bộ flow phải retest.”

**Điểm nhấn**
- Nhấn chữ “proposed” — thể hiện tính kỹ thuật, không phải ý kiến chủ quan.
- Đặt điều kiện tiên quyết: MERLIN P99 phải được đo trước.

**Câu chuyển**

> “Vì vậy 02:00 là kết quả của phép tính, không phải quyết định cảm tính.”

#### Câu 1.2 — Nếu batch AI kéo dài quá 01:30, chuyện gì xảy ra?

**Người hỏi dự kiến:** Operations Manager
**Mức độ khó:** ★★★

**Kịch bản trả lời**

> “01:30 là retry stop, không phải deadline của toàn bộ hệ thống. Sau 01:30, hệ thống không được bắt đầu retry mới. Nếu tới 01:45 dữ liệu chưa đầy đủ, chúng tôi publish nothing; AI rút khỏi run hiện tại, MERLIN tiếp tục bằng quy trình min/max mặc định. MERLIN không bao giờ phải chờ AI.”

**Điểm nhấn**
- Phân biệt rõ “retry stop” và “system deadline”.
- Fail-open là hành động, không phải lời hứa.

#### Câu 1.3 — Nếu MERLIN gặp sự cố lúc 03:00, AI có thể thay thế để kịp 04:00 không?

**Người hỏi dự kiến:** Executive, CFO
**Mức độ khó:** ★★★★ (Cạm bẫy)

**Kịch bản trả lời**

> “Không. MERLIN là hệ thống duy nhất có quyền tạo đơn hàng và gửi EDI. AI sidecar không có kết nối đến EDI hay nhà cung cấp. Nếu MERLIN gặp sự cố lúc 03:00, đó là incident của MERLIN và được xử lý bởi đội vận hành MERLIN hiện tại. AI không thay thế MERLIN trong bất kỳ tình huống nào.”

**Điểm nhấn**
- Dứt khoát từ đầu: “Không.”
- Lặp lại nguyên tắc cốt lõi: AI không nằm trên critical path.

### Nhóm 2: Chất Lượng Dữ Liệu & Inventory

#### Câu 2.1 — Inventory chỉ chính xác 78%, vậy làm sao tin recommendation của AI?

**Người hỏi dự kiến:** Category Manager, Data Governance
**Mức độ khó:** ★★★

**Kịch bản trả lời**

> “Chúng tôi không xem inventory như shelf truth tuyệt đối. Mỗi candidate phải có data age, calibrated interval, và interval coverage. Optimizer kiểm tra feasibility ở cả lower và upper bound, đồng thời áp dụng case-pack, shelf-life, DC capacity và category cap. Khi uncertainty quá cao — ví dụ interval coverage dưới 85% — row bị suppress và MERLIN dùng kết quả mặc định.”

**Điểm nhấn**
- Thừa nhận vấn đề (78%) thay vì phủ nhận.
- Giải thích cơ chế kiểm soát (interval, feasibility, suppression).
- Nhấn: Khi nghi ngờ, quay về MERLIN.

#### Câu 2.2 — Tại sao không dùng real-time inventory thay vì nightly snapshot?

**Người hỏi dự kiến:** Technical Architect
**Mức độ khó:** ★★☆

**Kịch bản trả lời**

> “Vì POS và các bảng nội bộ MERLIN nằm trong vùng không an toàn để can thiệp. Chúng tôi không tự tạo interface mới. Nightly snapshot là một trong ba điểm mở rộng được legacy hỗ trợ. Hơn nữa, real-time inventory vẫn chỉ đạt 78% accuracy; việc giảm độ trễ từ 24 giờ xuống 1 giờ không giải quyết được vấn đề uncertainty chính. Cách tiếp cận của chúng tôi là tính toán với uncertainty thay vì giả vờ uncertainty không tồn tại.”

**Điểm nhấn**
- Không phải vì không muốn, mà vì không được phép can thiệp vào MERLIN core.
- Đặt lại vấn đề: Vấn đề là độ tin cậy, không phải độ trễ.

### Nhóm 3: Capacity & Performance

#### Câu 3.1 — 162 triệu forecast có thực tế không? Hệ thống đã test chưa?

**Người hỏi dự kiến:** CTO, Performance Engineer
**Mức độ khó:** ★★★☆

**Kịch bản trả lời**

> “162 triệu forecast là six-times release test bảo thủ, chưa phải kết quả đã đạt. Đây là production gate: AI chỉ được phép influence production sau khi vượt qua ba lần test liên tiếp đáp ứng throughput, completeness, reproducibility, cost và deadline gate. Cho đến lúc đó, đó là design target để chúng tôi thiết kế kiến trúc, không phải claim đã thành công.”

**Điểm nhấn**
- Phân biệt rõ “design target” vs “benchmark result”.
- Đưa ra điều kiện cụ thể: 3 lần liên tiếp.

#### Câu 3.2 — Nếu throughput không đạt 30.000 forecast/s, backup plan là gì?

**Người hỏi dự kiến:** Infrastructure Lead
**Mức độ khó:** ★★★

**Kịch bản trả lời**

> “Nếu throughput dưới 22.500 record/s — critical threshold — model hoặc slice sẽ bị disable. Hệ thống quay về MERLIN mặc định. Đồng thời, chúng tôi có thể chuyển sang managed elastic batch với pre-peak reservation hoặc cân nhắc giảm partition phức tạp. Nhưng quy tắc cứng: không retry tuyệt vọng khi gần 04:00.”

**Điểm nhấn**
- Có con số cụ thể (22.500), không phải “sẽ xem xét”.
- Fail-open là default, không phải exception.

### Nhóm 4: Kiến Trúc & Integration Pattern

#### Câu 4.1 — Tại sao không kết nối AI synchronously với MERLIN?

**Người hỏi dự kiến:** Enterprise Architect
**Mức độ khó:** ★★☆

**Kịch bản trả lời**

> “Vì synchronous dependency khiến AI latency hoặc failure có thể làm chậm ordering. External asynchronous sidecar chỉ sử dụng ba interface đã được legacy hỗ trợ: nightly snapshot, override table và review queue. Sidecar có thể bị remove hoàn toàn mà không cần rollback MERLIN code. Điểm số weighted matrix của sidecar là 4.80, cao hơn hẳn synchronous API gateway (1.80) và in-process plugin (1.30).”

**Điểm nhấn**
- Nhấn “4.80” để chứng minh quyết định dựa trên dữ liệu, không phải sở thích.
- Sidecar có thể tháo rời — acceptance test.

#### Câu 4.2 — Tại sao không viết AI trực tiếp vào MERLIN engine luôn cho nhanh?

**Người hỏi dự kiến:** Engineering Manager (có thể thích giải pháp đơn giản)
**Mức độ khó:** ★★★ (Cạm bẫy)

**Kịch bản trả lời**

> “Vì MERLIN engine là thành phần không được phép thay đổi. Nó chứa logic order generation, scheduler nội bộ và các bảng chưa được tài liệu hóa. Can thiệp vào đó là unsafe intervention. Thay vào đó, chúng tôi dùng override table — một interface được hỗ trợ — để MERLIN tự đọc và tự quyết định. AI không bao giờ ghi vào engine trực tiếp.”

**Điểm nhấn**
- Đặt boundary cứng: “không được phép thay đổi”.
- Override table là cách duy nhất hợp lệ.

### Nhóm 5: Model & AI Technology

#### Câu 5.1 — Tại sao chọn hybrid hierarchical model thay vì global model hoặc deep learning?

**Người hỏi dự kiến:** Data Science Lead, CTO
**Mức độ khó:** ★★★

**Kịch bản trả lời**

> “Hybrid hierarchical statistical ML đạt 4.55 trong matrix, nhờ khả năng xử lý sparse series, promotion behaviour, hierarchical information sharing và quantile/uncertainty output. Tuy nhiên, khi tăng trọng số cost, hybrid và global model cùng đạt 4.45. Vì vậy, chúng tôi không tuyên bố hybrid chắc chắn tốt nhất. Quyết định cuối cùng sẽ dựa trên replay evidence đo WAPE, calibration, throughput và cost.”

**Điểm nhấn**
- Thừa nhận uncertainty trong chính quyết định model → tăng credibility.
- Nhấn “replay evidence” — quyết định dựa trên dữ liệu thực.

#### Câu 5.2 — Tại sao không dùng LLM cho forecasting hoặc replenishment?

**Người hỏi dự kiến:** AI Enthusiast, non-technical stakeholder
**Mức độ khó:** ★★☆

**Kịch bản trả lời**

> “Vì numeric forecasting và replenishment optimization yêu cầu reproducibility, throughput lớn, version control, calibrated uncertainty và kết quả có thể audit. LLM không đáp ứng được các yêu cầu này ở quy mô 27 triệu record mỗi đêm với chi phí hợp lý. Statistical ML dự báo nhu cầu; deterministic rules giới hạn số lượng. LLM chỉ xuất hiện ở Associate Copilot, trong phạm vi read-only và non-safety.”

**Điểm nhấn**
- Dùng ngôn ngữ business: “có thể audit”, “chi phí hợp lý”.
- Giải thích rõ ranh giới LLM vs Statistical ML.

### Nhóm 6: Fail-Open, Kill Switch & Operations

#### Câu 6.1 — Điều gì xảy ra khi AI phục hồi lúc 03:30?

**Người hỏi dự kiến:** Operations Director
**Mức độ khó:** ★★☆

**Kịch bản trả lời**

> “Không có thay đổi nào đối với run hiện tại. Kết quả chỉ được lưu để diagnosis và learning. Không có late publication, không rerun, không sidecar EDI. Đây chính là fail-open: AI lỗi hay phục hồi muộn đều không ảnh hưởng đến MERLIN.”

**Điểm nhấn**
- Ba chữ “không” liên tiếp: không late publish, không rerun, không EDI.
- Nhấn: 03:30 chỉ cho học, không cho chạy.

#### Câu 6.2 — Kill switch có thực tế không nếu cloud control plane bị lỗi?

**Người hỏi dự kiến:** Security Lead, CISO
**Mức độ khó:** ★★★★

**Kịch bản trả lời**

> “Có một independent SAP break-glass path. Nếu normal acknowledgement thất bại, một identity riêng có thể revoke database access, revoke network access, và terminate active sessions. Cleanup chỉ sử dụng exact AI-owned ledger và không chạm manual row. Trong vòng 5 phút, hệ thống phải chứng minh: không còn active permit, không còn active session, không còn active transaction, không còn unexplained AI row. Trạng thái cuối: VERIFIED_SAFE.”

**Điểm nhấn**
- Liệt kê 4 điều kiện “không còn…” để tạo cảm giác chắc chắn.
- Nhấn “independent” — kill switch không phụ thuộc vào cloud provider duy nhất.

#### Câu 6.3 — Nếu AI viết nhầm vào override table, làm sao dọn dẹp?

**Người hỏi dự kiến:** Database Administrator
**Mức độ khó:** ★★★

**Kịch bản trả lời**

> “Mỗi dòng AI ghi vào override table đều có correlation key, epoch và hash ledger. Cleanup identity chỉ xoá exact unconsumed AI-owned keys được xác định bằng exact key và hash ledger. Không được chọn manual row để tránh xoá nhầm dữ liệu của MERLIN hoặc Category Manager.”

**Điểm nhấn**
- “Exact key” và “hash ledger” — chứng minh traceability.
- Nhấn “không được chọn manual row” — bảo vệ dữ liệu legacy.

### Nhóm 7: Business Value & ROI

#### Câu 7.1 — Business target giảm out-of-stock từ 7.2% xuống 3% có thực tế không?

**Người hỏi dự kiến:** Category Director, CFO
**Mức độ khó:** ★★★☆

**Kịch bản trả lời**

> “Đây là design target và release gate, chưa phải kết quả đã đo. Nó sẽ được kiểm chứng qua historical replay so sánh AI recommendation với MERLIN actual outcome, sau đó qua production shadow chạy 4 tuần. Nếu shadow không đạt target, AI không được phép pilot. Chúng tôi không yêu cầu doanh nghiệp tin AI chỉ vì kiến trúc sử dụng công nghệ hiện đại; chúng tôi đề xuất cho phép AI chứng minh giá trị một cách an toàn.”

**Điểm nhấn**
- Thừa nhận đây là target, không phải guarantee.
- Đặt gate rõ ràng: không đạt shadow = không pilot.
- Nhấn câu kết mạnh mẽ về “chứng minh giá trị”.

#### Câu 7.2 — 3.8 triệu đô là chi phí cuối cùng hay ước tính?

**Người hỏi dự kiến:** CFO, Procurement
**Mức độ khó:** ★★☆

**Kịch bản trả lời**

> “3.8 triệu là target cost; 4 triệu là hard ceiling. Nếu cost vượt 4 triệu, AI influence phải dừng lại. Con số này bao gồm compute, storage, monitoring, certified adapter, on-call roster và reconciliation. Tất cả vẫn phải được chứng minh bằng observed production evidence trước khi mở rộng pilot.”

**Điểm nhấn**
- “Hard ceiling” — tạo cảm giác kiểm soát tài chính.
- Nhấn: Vượt ceiling = dừng, không đàm phán.

### Nhóm 8: Risk & Compliance

#### Câu 8.1 — Food-safety liability được kiểm soát thế nào?

**Người hỏi dự kiến:** Legal, Compliance Officer
**Mức độ khó:** ★★★★

**Kịch bản trả lời**

> “Associate Copilot hoàn toàn tách biệt khỏi ordering. Đối với nội dung food-safety, Copilot chỉ hiển thị exact approved evidence, fixed approved wording, có citation và source version. LLM không được tự tạo, tóm tắt, dịch hoặc diễn giải nội dung an toàn. Nếu phát hiện generated safety content, wrong-SKU content hoặc unsupported content, hệ thống refuse, disable affected domain và escalate.”

**Điểm nhấn**
- “Exact approved evidence” — không có sáng tạo từ AI.
- Nhấn 3 hành động: refuse, disable, escalate.

#### Câu 8.2 — Data residency và PII được bảo vệ ra sao?

**Người hỏi dự kiến:** Data Protection Officer, Legal
**Mức độ khó:** ★★★

**Kịch bản trả lời**

> “Snapshot được kiểm tra data residency trong validation stage. Chúng tôi không export member-level loyalty PII. Audit ledger là immutable và retention policy cho AI artifacts vẫn phải được Data Governance phê duyệt. NFR table có owner cụ thể cho từng hạng mục data residency và retention.”

**Điểm nhấn**
- “Không export PII” — dứt khoát.
- “Immutable audit ledger” — chứng minh traceability.

### Nhóm 9: Rollout & Pilot

#### Câu 9.1 — Tại sao pilot chỉ 20 cửa hàng và 2 category?

**Người hỏi dự kiến:** Business Director
**Mức độ khó:** ★★☆

**Kịch bản trả lời**

> “Vì AI phải từng bước giành được quyền ảnh hưởng. 20 cửa hàng, hai category non-fresh rủi ro thấp, tối đa 5.000 dòng và không quá 100 review mỗi đêm là bounded influence. Nếu pilot thành công, chúng tôi mới mở rộng dần theo điều kiện: quality gate tiếp tục đạt, không có critical incident, cost nằm trong ceiling, operational roster đã được tài trợ, và kill-switch drill tiếp tục thành công.”

**Điểm nhấn**
- “AI phải giành được quyền” — AI không có quyền mặc định.
- Liệt kê 5 điều kiện mở rộng.

#### Câu 9.2 — Bao lâu thì AI có thể ảnh hưởng toàn bộ hệ thống?

**Người hỏi dự kiến:** CEO, Executive Sponsor
**Mức độ khó:** ★★★ (Cạm bẫy — ép timeline)

**Kịch bản trả lời**

> “Không có timeline cố định cho full rollout. Điều kiện tiên quyết là vượt qua từng gate: Discovery → Historical Replay → 4 tuần Production Shadow → Bounded Pilot → Bounded Expansion. Mỗi gate có numeric threshold và kill-switch drill. Nếu một gate thất bại, hệ thống quay về shadow hoặc replay. Chúng tôi ưu tiên safety trước speed.”

**Điểm nhấn**
- Không đưa ra ngày cụ thể dù bị ép.
- Nhấn “safety trước speed” — phù hợp văn hóa bảo thủ.

### Nhóm 10: Câu Hỏi "Cạm Bẫy" Và Cách Xử Lý

#### Câu 10.1 — "Vậy AI này thực ra có giá trị gì nếu nó không tạo đơn hàng?"

**Người hỏi dự kiến:** Skeptical Executive
**Mức độ khó:** ★★★★★

**Kịch bản xử lý**

> “AI tạo demand forecast và bounded recommendation với calibrated uncertainty. Điều này giúp MERLIN — vốn dùng quy tắc min/max tĩnh — tiếp cận gần hơn với actual demand, đặc biệt trong promotion và sparse series. Tuy nhiên, giá trị phải được chứng minh qua replay và shadow, không phải qua lý thuyết. Nếu sau 4 tuần shadow không cho thấy cải thiện WAPE hoặc out-of-stock, chúng tôi sẽ không đề xuất pilot. Đó chính là lý do kiến trúc được thiết kế để tháo rời dễ dàng — nếu AI không mang lại giá trị, doanh nghiệp không mất gì ngoài chi phí shadow có kiểm soát.”

**Chiến thuật**
- Đồng cảm với skepticism, không bác bỏ.
- Đưa ra cơ chế “thoát” an toàn: nếu không hiệu quả, tháo rời.
- Nhấn “doanh nghiệp không mất gì” — giảm rủi ro perceived.

#### Câu 10.2 — "Tại sao không thuê thêm người thay vì bỏ tiền làm AI?"

**Người hỏi dự kiến:** CFO, HR Director
**Mức độ khó:** ★★★★

**Kịch bản xử lý**

> “Category Manager vẫn là người review; AI không thay thế con người. Vấn đề là 27 triệu forecast mỗi đêm và 7.04 triệu stocked candidate vượt quá khả năng phân tích thủ công của con người. AI xử lý scale và pattern; con người giữ vai trò review, judgment và escalation. Đây là augmentation, không phải replacement.”

**Chiến thuật**
- Đưa con số cụ thể để chứng minh scale vượt quá manual capacity.
- Đặt AI và con người vào vị trí bổ trợ, không đối đầu.

#### Câu 10.3 — "Nếu đối thủ đã dùng AI real-time, tại sao chúng ta vẫn dùng batch?"

**Người hỏi dự kiến:** Commercial Director, CEO
**Mức độ khó:** ★★★★

**Kịch bản xử lý**

> “Vì bối cảnh hệ thống khác nhau. Đối thủ có thể xây hệ thống mới từ đầu; chúng ta có MERLIN legacy với 78% inventory accuracy, EDI cố định và deadline 04:00. Thay vì so sánh với đối thủ, chúng tôi so sánh với incumbent path: MERLIN min/max hiện tại. Nếu AI giúp giảm out-of-stock từ 7.2% xuống gần 3% với chi phí dưới 4 triệu và không làm gián đoạn một ngày nào, đó là lợi ích thực. Không gián đoạn là ràng buộc không thể đánh đổi để đuổi theo real-time.”

**Chiến thuật**
- Không bác bỏ đối thủ, đặt lại bối cảnh.
- Nhấn “không gián đoạn một ngày nào” — giá trị cốt lõi.

## III. Phụ lục: Cheat sheet cho người trình bày

### Những con số phải nhớ

| Con số | Ý nghĩa |
|---:|---|
| **02:00** | Proposed AI write close (điều kiện: MERLIN P99 ≤ 90 phút) |
| **01:30** | Retry stop |
| **01:45** | Complete-set gate |
| **04:00** | Absolute deadline (MERLIN) |
| **27M** | Forecast records / đêm (normal) |
| **7.04M** | Stocked candidate optimization |
| **162M** | Six-times test forecast |
| **30.000/s** | Forecast throughput target |
| **15.000/s** | Optimization throughput target |
| **198.7 phút** | Tổng thời gian target trong cửa sổ 240 phút |
| **7.2% → 3%** | Out-of-stock target |
| **4.8% → 3%** | Fresh waste target |
| **41% → ≤25%** | WAPE target |
| **3.8M USD** | Cost target |
| **4.0M USD** | Hard ceiling |
| **4.80** | Sidecar score (integration pattern) |
| **4.65** | Managed elastic batch score |
| **4.55** | Hybrid model score |
| **20** | Max risk score (deadline, inventory) |
| **30 giây** | Kill switch acknowledge |
| **60 giây** | New write reject |
| **5 phút** | Verified safe |
| **78%** | Inventory accuracy |
| **85%** | Inventory interval coverage threshold |

### Các câu hỏi nên chủ động mời

Nếu khán giả im lặng, hãy tự đặt câu hỏi để dẫn dắt:

- “Một câu hỏi thường gặp là: tại sao không dùng LLM? Câu trả lời là…”
- “Nhiều người hỏi về 02:00. Đây là proposed gate dựa trên…”
- “Về kill switch: nếu cloud control plane lỗi, chúng tôi có independent break-glass path…”

### Khi bị hỏi vượt ra ngoài tài liệu

- **Không nói:** “Tôi không biết.”
- **Nên nói:** “Đó là câu hỏi quan trọng cần được kiểm chứng trong giai đoạn Discovery / Replay. Tôi ghi lại và sẽ cập nhật trong ADR tiếp theo.”

---

Kịch bản này được xây dựng dựa trên 6 artifact của Variant B: A1 (C4), A2 (NFR), A3 (Matrix), A4 (ADR), A5 (Risk), A6 (Legacy Assessment).

Kịch bản Q&A chi tiết đã được tạo xong! Tài liệu bao gồm:

Cấu trúc kịch bản:

10 nhóm câu hỏi với tổng cộng 25+ câu hỏi dự kiến, được phân loại theo ngườI hỏI (Operations, CTO, CFO, Legal, Executive…) và mức độ khó từ 1–5 sao.
Mỗi câu hỏi có đầy đủ: kịch bản trả lờI word-by-word, điểm cần nhấn giọng, và chiến thuật xử lý (đặc biệt cho câu cạm bẫy).
Các nhóm chính:

Ràng buộc thờI gian & Deadline (02:00, 04:00, retry stop)
Chất lượng dữ liệu & Inventory (78%, calibrated interval)
Capacity & Performance (162M forecast, throughput)
Kiến trúc & Integration (tại sao không synchronous)
Model & AI Technology (tại sao không LLM, hybrid vs global)
Fail-open, Kill Switch & Operations (03:30 recovery, break-glass)
Business Value & ROI (3.8M target, OOS 7.2%→3%)
Risk & Compliance (food-safety, data residency, PII)
Rollout & Pilot (20 cửa hàng, 2 category)
Câu hỏi "cạm bẫy" từ skeptical executive
Phụ lục hữu ích:

Cheat sheet con số phải nhớ (27M, 7.04M, 4.80, 4.65, v.v.)
Bảng ngôn ngữ cấm trong Q&A
Cách xử lý khi bị hỏI vượt ra ngoài tài liệu

**Phiên bản:** 1.0 | **Ngày:** 25/07/2026


