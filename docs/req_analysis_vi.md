Dựa trên PDF, workshop này **không yêu cầu bạn xây dựng một hệ thống chạy thật hoặc viết code hoàn chỉnh**. Công việc chính là thiết kế một kiến trúc AI end-to-end, chứng minh kiến trúc đó đáp ứng các constraint thực tế, rồi bảo vệ các quyết định trước trainer.

Nhóm của bạn là **Group 4 – Variant B: AI-Retrofit**, tức là phải đưa AI vào hệ thống legacy hiện tại mà **không được thay thế hoặc làm hỏng hệ thống cũ**. 

# 1. Mục tiêu thật sự của workshop

Workshop kiểm tra khả năng:

```text
Requirement
→ Phân tích constraint
→ Chọn kiến trúc
→ Chứng minh bằng số liệu
→ Nêu trade-off
→ Nhận diện rủi ro
→ Bảo vệ quyết định
```

Trainer không chỉ xem sơ đồ có đẹp hay không. Họ sẽ hỏi:

* Vì sao chọn giải pháp này?
* Vì sao không chọn giải pháp khác?
* Có hoàn thành trước 04:00 không?
* AI chết lúc 03:30 thì chuyện gì xảy ra?
* Chi phí có vượt ngân sách không?
* Inventory chỉ chính xác 78% thì xử lý thế nào?
* Có đang dùng LLM cho bài toán không cần ngôn ngữ không?
* Thiết kế có làm ảnh hưởng MERLIN, POS hoặc supplier EDI không?

Thông điệp quan trọng nhất của tài liệu là:

> Workshop đánh giá khả năng bảo vệ một kiến trúc, không chỉ khả năng vẽ kiến trúc.

# 2. Bài toán kinh doanh bạn phải giải quyết

NovaMart là hệ thống bán lẻ lớn với:

* 640 cửa hàng.
* Khoảng 11.000 SKU cho mỗi cửa hàng.
* 2,9 triệu giao dịch mỗi ngày.
* Khoảng 27 triệu chuỗi dự báo mỗi ngày.
* Biên lợi nhuận ròng chỉ 2,4%.
* Fresh waste hiện tại 4,8%.
* Out-of-stock hiện tại 7,2%.
* Hệ thống phải hoàn thành forecast và replenishment trong khoảng 22:00–04:00.

Mục tiêu kinh doanh:

| KPI                               |          Hiện tại |        Mục tiêu |
| --------------------------------- | ----------------: | --------------: |
| Out-of-stock                      |              7,2% |            3,0% |
| Fresh waste                       |              4,8% |            3,0% |
| Forecast WAPE                     |               41% |            ≤25% |
| Thời gian associate tìm thông tin | Khoảng 50 phút/ca | Dưới 15 phút/ca |
| Nhân sự store associate           |             8.400 |  Không cắt giảm |

AI được dùng để cải thiện vận hành, không phải thay thế nhân viên.

# 3. Variant B của bạn có nghĩa là gì?

Variant B là **AI-Retrofit cho hệ thống legacy**.

NovaMart đang sử dụng hệ thống MERLIN, một ERP dựa trên SAP đã được customize từ năm 2006.

MERLIN hiện quản lý:

* Merchandising.
* Inventory.
* Replenishment.
* Supplier EDI.
* Order gửi đến supplier.

Bạn không được thiết kế một platform hoàn toàn mới rồi yêu cầu NovaMart chuyển sang đó.

## Những thứ tuyệt đối không được thay đổi

### Không được thay MERLIN

Core replacement đã thất bại vào năm 2021, mất hai năm và 22 triệu USD. Do đó:

```text
Không replace MERLIN
Không gradually replace MERLIN
Không sửa replenishment engine của MERLIN
```

### Không được sửa POS

POS:

* Có khoảng 7.000 lanes.
* Chạy offline-first.
* Đồng bộ dữ liệu mỗi đêm.
* Thay đổi POS cần chín tháng tái chứng nhận với payment processor.

Requirement ghi rất rõ:

```text
You cannot touch POS. Under any circumstances.
```

### Không được thay supplier EDI

* Có 2.300 supplier.
* EDI schema cố định.
* Deadline 04:00 cố định.
* Không được yêu cầu supplier thay đổi integration.

## MERLIN chỉ cung cấp ba extension points

Bạn chỉ được tích hợp qua một trong ba điểm:

1. **Override table**

   MERLIN đọc bảng này trước khi tạo replenishment order.

2. **Review queue**

   Category manager xem và duyệt các order/recommendation sau khi MERLIN tạo order.

3. **Nightly batch**

   Các tác vụ có thể được gắn vào batch chạy ban đêm.

Đây là giới hạn quan trọng nhất của kiến trúc Variant B.

# 4. Những constraint khó nhất bạn phải xử lý

## 4.1 Deadline 04:00

Toàn bộ quá trình sau phải hoàn thành trong sáu giờ:

```text
Data preparation
→ Feature generation
→ Forecasting
→ Replenishment optimization
→ Validation
→ Write MERLIN overrides
→ MERLIN creates orders
→ Supplier EDI
```

Nếu bỏ lỡ deadline, 640 cửa hàng có thể không nhận được hàng trong ngày.

Thiết kế phải có:

* Deadline guard.
* Timeout.
* Partial completion strategy.
* Fallback.
* Kill switch.
* Late-result rejection.

Ví dụ:

```text
Nếu AI chưa hoàn thành trước 03:15
→ Không tiếp tục chờ
→ Không ghi override mới
→ MERLIN chạy thuật toán min/max hiện tại
→ Ordering chain vẫn tiếp tục
```

## 4.2 Inventory chỉ chính xác 78%

Stock-on-hand của MERLIN được cập nhật từ POS nightly sync.

Điều này có nghĩa là:

```text
22% dữ liệu inventory có thể không phản ánh đúng hàng trên kệ
```

Bạn không thể thiết kế dựa trên giả định:

```text
Current inventory = exact real-time inventory
```

Thiết kế phải xử lý inventory như một giá trị có độ không chắc chắn, chẳng hạn:

* Confidence score.
* Safety stock.
* Inventory uncertainty bounds.
* Forecast adjustment.
* Human review cho recommendation bất thường.
* Không tự động override nếu dữ liệu quá cũ hoặc confidence thấp.

## 4.3 AI không được làm đứt ordering chain

Đây là requirement bắt buộc:

```text
AI unavailable
→ MERLIN vẫn phải chạy
→ Replenishment vẫn phải được tạo
→ Store vẫn nhận được truck
```

Vì vậy, AI nên đóng vai trò:

```text
Recommendation / optimization sidecar
```

chứ không được trở thành dependency bắt buộc của MERLIN.

Kiến trúc phù hợp thường là:

```text
AI Sidecar
+ Shadow-assist rollout
+ MERLIN override table
+ Review queue
+ Fail-open fallback
```

Trong đó:

* MERLIN vẫn là system of record.
* AI chỉ cung cấp recommendation hoặc override.
* Nếu AI không hoạt động, MERLIN bỏ qua override và chạy logic cũ.
* Kill switch vô hiệu hóa AI override trong vài phút.

## 4.4 Lunar New Year peak

Trong hai tuần Lunar New Year:

```text
Load = khoảng 6 lần bình thường
Promotion demand = có thể tăng 3–10 lần
```

Bạn phải chứng minh kiến trúc xử lý được peak, không chỉ normal load.

## 4.5 Chi phí

Board cho phép khoảng 4 triệu USD/năm cho tổng AI run cost trong bài toán toàn hệ thống.

Tài liệu đưa ra phép tính:

```text
27 triệu forecast/ngày
Chi phí mục tiêu ≈ $0,0004/forecast
```

Điều này gần như loại bỏ phương án dùng LLM để forecast từng SKU.

Phân chia công nghệ hợp lý:

| Bài toán               | Loại công nghệ                          |
| ---------------------- | --------------------------------------- |
| Demand forecasting     | Time-series ML/statistical model        |
| Replenishment quantity | Optimization/rules engine               |
| Dynamic markdown       | Optimization/ML                         |
| Associate copilot      | RAG + LLM                               |
| Allergen answer        | Controlled retrieval, citation, refusal |
| Promo forecasting      | ML/causal forecasting                   |

LLM chỉ nên dùng cho bài toán ngôn ngữ như associate copilot, không dùng để tính 27 triệu forecast mỗi ngày.

## 4.6 Food safety và allergen

Một câu trả lời sai về allergen không chỉ là lỗi customer service mà có thể gây hậu quả pháp lý nghiêm trọng.

Copilot phải có:

* Nguồn dữ liệu authoritative.
* Trích dẫn tài liệu và version.
* Không trả lời khi thiếu bằng chứng.
* Confidence policy.
* Refusal policy.
* Human escalation.
* Audit log.
* Không để LLM tự suy đoán allergen.

# 5. Sáu artifact bạn bắt buộc phải nộp

PDF yêu cầu đúng **sáu artifact**, mỗi artifact phải có một người phụ trách được ghi tên. 

## Artifact 1 — C4 Context và Container Diagram

Bạn cần tạo hai mức sơ đồ.

### C4 System Context

Thể hiện:

* NovaMart AI Platform.
* MERLIN.
* POS.
* Supplier EDI.
* Data warehouse.
* External data.
* Category manager.
* Store associate.
* Supplier hoặc distribution center.

Mục tiêu là cho thấy hệ thống AI nằm ở đâu trong toàn bộ ecosystem.

### C4 Container Diagram

Thể hiện các thành phần bên trong AI platform, ví dụ:

```text
Batch Orchestrator
Data Validation
Feature Pipeline
Forecasting Service
Replenishment Optimizer
Recommendation Store
MERLIN Adapter
Model Registry
Monitoring
Audit Log
Kill-switch Control
```

Ngoài sơ đồ, cần mô tả request flow hoặc batch flow:

```text
1. Nightly data được nhận.
2. Data validation kiểm tra freshness và completeness.
3. Forecasting service tạo demand forecast.
4. Optimization service tạo suggested order.
5. Policy validator kiểm tra output.
6. Valid recommendation được ghi vào override table.
7. MERLIN đọc override và tạo order.
8. Nếu AI thất bại, MERLIN dùng min/max engine hiện tại.
```

## Artifact 2 — NFR Table

Bạn phải cover tất cả non-functional requirements bằng **số liệu thật**, không viết chung chung.

Không nên viết:

```text
The system will be scalable.
The system will be highly available.
```

Nên viết:

```text
27 million forecasts must complete in six hours.

Required average throughput:
27,000,000 / 21,600 seconds
≈ 1,250 forecasts/second.

Design capacity:
2,500 forecasts/second normal,
7,500 forecasts/second peak.
```

NFR table nên có:

| NFR               | Target                       | Design mechanism                     | Measurement             | Failure behavior           |
| ----------------- | ---------------------------- | ------------------------------------ | ----------------------- | -------------------------- |
| Batch deadline    | Trước 04:00                  | Parallel batch + deadline guard      | Batch completion time   | Fallback to MERLIN         |
| Forecast volume   | 27M/day                      | Distributed processing               | Forecasts/second        | Partial fallback           |
| Peak              | 6×                           | Autoscaling/pre-provisioned capacity | Queue lag               | Reduce optional processing |
| AI failure        | Không ảnh hưởng order        | Fail-open sidecar                    | Availability/error rate | Disable overrides          |
| Inventory quality | 78% accuracy                 | Confidence and safety stock          | Data-quality score      | Human review               |
| Data residency    | Loyalty data ở trong country | Regional deployment                  | Cross-border audit      | Block processing           |
| Allergen safety   | Không hallucinate            | Grounded RAG + refusal               | Citation rate           | Escalate                   |

## Artifact 3 — Technology Selection Matrix

Bạn cần có weighted scoring cho **ít nhất ba quyết định quan trọng**.

Ví dụ ba quyết định:

### Quyết định 1: Integration pattern

Các lựa chọn:

* Sidecar.
* Event-driven.
* API gateway.
* Shadow-assist.
* In-process plugin.

Các tiêu chí:

* Legacy safety.
* Delivery risk.
* Fail-open support.
* Time to market.
* Operational complexity.
* Cost.
* Rollback capability.

### Quyết định 2: Forecast execution platform

Ví dụ:

* Managed cloud batch.
* Kubernetes.
* Spark.
* Serverless batch.
* On-premise compute.

### Quyết định 3: Forecast model strategy

Ví dụ:

* Một global model.
* Model cho từng SKU/store.
* Hybrid hierarchical model.
* Statistical baseline.

Bảng cần có:

| Criteria       | Weight | Option A | Option B | Option C |
| -------------- | -----: | -------: | -------: | -------: |
| Reliability    |    25% |        5 |        3 |        2 |
| Legacy safety  |    25% |        5 |        3 |        1 |
| Delivery speed |    20% |        4 |        3 |        2 |
| Cost           |    15% |        4 |        3 |        2 |
| Operability    |    15% |        4 |        2 |        2 |

Sau đó phải nói rõ:

* Option nào được chọn.
* Option nào bị loại.
* Trade-off đã chấp nhận.

## Artifact 4 — Một ADR

ADR là Architecture Decision Record.

Bạn chỉ cần chọn **một quyết định quan trọng nhất**.

ADR phù hợp nhất cho Variant B:

```text
Use a fail-open AI sidecar integrated through
MERLIN's override table and review queue,
with shadow-assist as the initial rollout mode.
```

ADR phải có:

```text
Context
Decision drivers
Options considered
Decision
Positive consequences
Negative consequences
Risks
Rollback
When to revisit
```

Trade-off phải thành thật.

Ví dụ:

```text
Ưu điểm:
- Không sửa MERLIN core.
- Có thể rollback.
- AI failure không chặn replenishment.

Nhược điểm:
- Chỉ tích hợp được theo batch.
- Không có real-time inventory.
- Có khả năng recommendation chậm một ngày.
- Vận hành song song hai logic.
```

## Artifact 5 — Risk Register

Chọn đúng top năm risk.

Gợi ý:

1. Miss 04:00 deadline.
2. Inventory inaccuracy tạo order sai.
3. Model drift trong promotion hoặc Lunar New Year.
4. MERLIN integration hoặc override-table failure.
5. Unsafe allergen answer.

Mỗi risk cần:

| Field             | Nội dung                            |
| ----------------- | ----------------------------------- |
| Category          | Operational, data, model, safety... |
| Likelihood        | 1–5                                 |
| Impact            | 1–5                                 |
| Risk score        | Likelihood × Impact                 |
| Mitigation        | Ngăn ngừa                           |
| Contingency       | Xử lý khi xảy ra                    |
| Monitoring signal | Chỉ số giám sát                     |
| Threshold         | Ngưỡng cụ thể                       |
| Owner             | Người phụ trách                     |

Ví dụ:

```text
    Risk: Batch does not complete before 04:00
    Likelihood: 3
    Impact: 5
    Score: 15
    Signal: Forecast completion percentage
    Warning threshold: <90% by 02:30
    Critical threshold: <98% by 03:15
    Contingency: Disable new overrides and allow MERLIN baseline
```

## Artifact 6 — Variant B Specific

Đây là phần đặc biệt dành cho nhóm bạn.

Phải bao gồm ba phần chính.

### Legacy assessment

Phân loại:

| Có thể thay đổi        | Không thể thay đổi           | Không an toàn để đụng vào   |
| ---------------------- | ---------------------------- | --------------------------- |
| AI sidecar             | MERLIN core                  | POS                         |
| External data pipeline | EDI schema                   | Payment-certified flows     |
| Override generation    | 04:00 deadline               | MERLIN replenishment engine |
| Monitoring             | MERLIN system of record      | Direct supplier integration |
| Review UI/process      | Nightly sync characteristics | Assumed real-time inventory |

### Chọn integration pattern

Bạn phải gọi tên pattern và bảo vệ nó.

Một hướng hợp lý:

```text
Primary architecture pattern: Sidecar

Initial rollout mode: Shadow-assist
```

Cần phân biệt:

* **Sidecar** là cách hệ thống AI tồn tại bên cạnh MERLIN.
* **Shadow-assist** là cách rollout ban đầu, AI tạo recommendation nhưng chưa tự động ảnh hưởng order.

### Kill-switch design

Phải nói rõ kill switch vận hành như thế nào.

Ví dụ:

```text
Kill switch ON
→ Stop publishing recommendations
→ Mark current recommendation batch inactive
→ MERLIN adapter does not write overrides
→ Previously staged but unapproved overrides are invalidated
→ MERLIN runs existing min/max process
→ Alert Operations and Category Managers
```

Không nên chỉ viết:

```text
We will have a kill switch.
```

Bạn phải thể hiện:

* Ai được quyền kích hoạt?
* Kích hoạt ở đâu?
* Có hiệu lực trong bao lâu?
* Có xóa override cũ không?
* MERLIN fallback thế nào?
* Audit log lưu gì?

# 6. Giới hạn bài nộp và thuyết trình

## Bài nộp

* Tối đa **8 slides**.
* Có đủ sáu artifact.
* Mỗi artifact có một named owner.
* Deadline theo email: **12:00 PM ngày 24/07/2026**.

## Thuyết trình

Tổng cộng 12 phút:

```text
7 phút presentation
4 phút Q&A
1 phút changeover
```

Bạn không nên mở đầu bằng việc kể lại NovaMart là công ty bán lẻ thế nào. Tất cả nhóm đều có cùng scenario và trainer có thể dừng phần trình bày nếu bạn tốn thời gian giải thích lại đề bài.

Nên mở đầu trực tiếp:

> Our architecture keeps MERLIN authoritative and introduces a fail-open AI sidecar through the override table and review queue. AI improves replenishment, but its failure can never prevent stores from receiving their morning deliveries.

# 7. Đề xuất cấu trúc 8 slides

## Slide 1 — Architecture thesis

Trả lời trong một slide:

* Chúng ta xây gì?
* MERLIN giữ vai trò gì?
* Integration pattern nào?
* AI failure được cô lập thế nào?

## Slide 2 — Legacy assessment

Ba cột:

```text
Can change
Cannot change
Unsafe to touch
```

Hiển thị ba MERLIN extension points.

## Slide 3 — C4 architecture

Context và container architecture đã đơn giản hóa.

Không nhồi quá nhiều AWS/Azure service icon. Trainer cần hiểu quyết định, không cần xem service catalogue.

## Slide 4 — Batch flow và deadline 04:00

Dùng timeline:

```text
22:00
Data ingest
→ Validate
→ Forecast
→ Optimize
→ Write override
→ MERLIN
04:00
```

Đánh dấu fallback checkpoint ở khoảng 03:15.

## Slide 5 — NFR và cost proof

Chỉ lấy số quan trọng:

* 27 triệu forecast/ngày.
* Sáu giờ.
* Throughput yêu cầu.
* 6× peak.
* Cost per forecast.
* Fail-open.

## Slide 6 — Technology matrix và ADR

Trình bày:

* Integration pattern comparison.
* Sidecar được chọn.
* Shadow-assist là rollout mode.
* Trade-off quan trọng nhất.

## Slide 7 — Top năm risks

Tập trung vào:

* Risk.
* Signal.
* Threshold.
* Response.

## Slide 8 — Rollout và kill switch

Ví dụ:

```text
Phase 1: Shadow
Phase 2: Category pilot
Phase 3: Approved override
Phase 4: Controlled automation
```

Kết thúc bằng:

```text
AI improves MERLIN.
AI does not replace MERLIN.
AI failure does not stop replenishment.
```

# 8. Công việc cụ thể bạn cần làm

## Bước 1 — Chốt architecture thesis

Cả nhóm phải thống nhất một câu, ví dụ:

> We propose a fail-open AI sidecar that runs in shadow-assist mode and integrates with MERLIN through the override table and review queue, while MERLIN remains the ordering system of record.

Nếu câu này chưa thống nhất, chưa nên làm slide.

## Bước 2 — Chốt phạm vi

Sáu capability được đề bài đưa ra không có nghĩa là bắt buộc phải xây toàn bộ ở cùng mức độ.

Trong tám tháng và ngân sách giới hạn, bạn có thể ưu tiên:

```text
Primary:
- Demand forecasting
- Replenishment recommendation

Secondary:
- Associate copilot

Later phases:
- Dynamic markdown
- Online substitution
- Promotion planning
```

Cần giải thích vì sao một số capability được defer.

## Bước 3 — Làm legacy assessment

Liệt kê:

* System of record.
* Extension points.
* Hard constraints.
* Data weaknesses.
* Operational dependencies.
* Unsafe components.

## Bước 4 — Thiết kế architecture và flows

Hoàn thành:

* C4 Context.
* C4 Container.
* Nightly batch flow.
* AI failure flow.
* Kill-switch flow.
* Copilot request flow nếu có.

## Bước 5 — Làm capacity và cost calculation

Tối thiểu phải tính:

```text
Forecast throughput
Batch duration
Peak throughput
Storage volume assumptions
Compute cost
Cost per forecast
Fallback checkpoint
```

## Bước 6 — Tạo technology matrix

Chọn ít nhất ba quyết định quan trọng và chấm điểm.

## Bước 7 — Viết ADR

Chọn quyết định sidecar/fail-open làm ADR chính.

## Bước 8 — Lập risk register

Chọn đúng năm risk, mỗi risk có signal và threshold.

## Bước 9 — Đóng gói thành tám slides

Loại bỏ mọi nội dung không giúp defend một quyết định.

## Bước 10 — Tập Q&A

Mỗi thành viên phải trả lời được ít nhất:

* AI fail thì sao?
* Tại sao không event-driven hoàn toàn?
* Tại sao không replace MERLIN?
* Tại sao inventory 78% vẫn forecast được?
* Làm sao bảo vệ 04:00?
* Chi phí dựa trên giả định nào?
* Shadow mode kéo dài bao lâu?
* Khi nào cho phép auto-override?
* Ai chịu trách nhiệm cho allergen answer?
* Kill switch có thực sự dùng được không?

# 9. Cách chia việc cho nhóm 5–6 người

| Thành viên | Artifact sở hữu    | Công việc phụ                    |
| ---------- | ------------------ | -------------------------------- |
| Member 1   | C4 diagrams        | Architecture thesis              |
| Member 2   | NFR table          | Capacity và deadline calculation |
| Member 3   | Technology matrix  | Technology research              |
| Member 4   | ADR                | Integration pattern              |
| Member 5   | Risk register      | Monitoring thresholds            |
| Member 6   | Variant B artifact | Legacy assessment và kill switch |

Một người nên đóng vai trò **Architecture Integrator** để kiểm tra tất cả artifact có dùng cùng assumption hay không.

Ví dụ lỗi thường gặp:

```text
C4 nói dùng sidecar
ADR nói event-driven
NFR giả định real-time inventory
Risk register lại nói nightly inventory
```

Các artifact phải kể cùng một câu chuyện.

# 10. Definition of Done

Trước khi submit, kiểm tra:

* Đủ sáu artifacts.
* Không quá tám slides.
* Có named owner cho từng artifact.
* MERLIN vẫn là system of record.
* Không sửa POS.
* Không đổi EDI.
* Có xử lý inventory accuracy 78%.
* Có fail-open.
* Có kill switch cụ thể.
* Có shadow rollout.
* Có chứng minh deadline 04:00 bằng số.
* Có tính peak 6×.
* Có cost calculation.
* Không dùng LLM cho demand forecasting.
* Allergen answer có citation và refusal.
* Technology matrix có trade-off thật.
* Risk có monitoring signal và numeric threshold.
* Presentation không quá bảy phút.
* Các thành viên đã luyện Q&A.

Tóm lại, sản phẩm cuối của bạn không phải chỉ là một kiến trúc AI đẹp. Nó phải chứng minh được:

```text
AI tạo ra giá trị
nhưng không phá legacy,
không chặn replenishment,
không vượt chi phí,
và có thể bị tắt an toàn bất cứ lúc nào.
```
