# Hướng dẫn Workshop 1 — Group 4, Variant B: AI-Retrofit / Legacy

Tài liệu này tóm tắt yêu cầu của đề bài Workshop 1, giải thích sáu artifact bắt buộc và đưa ra sườn trình bày PowerPoint trong tối đa bảy phút. Nguồn yêu cầu chính là [`req_analysis_vi.md`](req_analysis_vi.md); các quyết định và con số cụ thể được đối chiếu với bộ artifact và presentation hiện tại.

## 1. Thông tin buổi workshop

| Nội dung | Thông tin |
|---|---|
| Nhóm | Group 4 |
| Chủ đề | Variant B — AI-Retrofit / Legacy |
| Phiên trình bày | Session 1 — ngày 28/07/2026 |
| Khung giờ của nhóm | 2:05–2:17 PM |
| Thuyết trình | 7 phút |
| Q&A | 4 phút |
| Chuyển nhóm | 1 phút |
| Hạn nộp theo email | 12:00 PM ngày 24/07/2026 |
| Giới hạn bài trình bày | Tối đa 8 slide |
| Bài nộp bắt buộc | Đúng 6 artifact, mỗi artifact có một owner được ghi tên |

Workshop không yêu cầu xây một hệ thống production hoàn chỉnh hoặc viết toàn bộ code. Nhóm phải thiết kế kiến trúc AI end-to-end, chứng minh kiến trúc phù hợp với các ràng buộc thực tế và bảo vệ được các quyết định trước trainer.

## 2. Đề bài thực sự đánh giá điều gì?

Chuỗi tư duy trainer muốn nhìn thấy là:

```text
Requirement
→ Constraint
→ Architecture decision
→ Quantitative proof
→ Failure behavior
→ Trade-off
→ Production gate
```

Một sơ đồ đẹp nhưng không trả lời được các câu hỏi sau vẫn có thể bị từ chối:

- Hệ thống có bảo vệ deadline 04:00 không?
- Nếu AI chết hoặc phục hồi muộn lúc 03:30 thì MERLIN có tiếp tục không?
- Có xử lý được 27 triệu forecast/ngày và peak 6× không?
- Inventory chỉ chính xác 78% thì có thể tự động ghi override đến mức nào?
- AI có vô tình sửa POS, supplier EDI hoặc MERLIN core không?
- Kill switch có thực sự ngăn được write mới và stale write không?
- Chi phí có nằm dưới trần 4 triệu USD/năm không?
- LLM có bị dùng sai cho forecast, quantity, price hoặc safety output không?
- Food-safety/allergen output có evidence, citation, refusal và audit không?

## 3. Bài toán kinh doanh

NovaMart có quy mô và mục tiêu chính như sau:

| Chỉ số | Baseline | Mục tiêu |
|---|---:|---:|
| Cửa hàng | 640 | Phục vụ toàn hệ thống theo lộ trình kiểm soát |
| SKU trung bình/cửa hàng | Khoảng 11.000 | Dùng để tính 7,04 triệu stocked candidates |
| Giao dịch/ngày | Khoảng 2,9 triệu | Không làm ảnh hưởng POS |
| Forecast/ngày | Khoảng 27 triệu | Hoàn thành trong batch window |
| Out-of-stock | 7,2% | 3,0% |
| Fresh waste | 4,8% | 3,0% |
| Forecast WAPE | 41% | ≤25% |
| Thời gian associate tìm thông tin | Khoảng 50 phút/ca | <15 phút/ca |
| Store associates | 8.400 | Không cắt giảm vì AI |
| Biên lợi nhuận ròng | 2,4% | Phải được bảo vệ trong thiết kế chi phí |

Thông điệp kinh doanh: AI được dùng để cải thiện replenishment và hỗ trợ nhân viên, không thay thế nhân viên và không được làm tăng rủi ro mất đơn hàng buổi sáng.

## 4. Variant B có nghĩa là gì?

Variant B là đưa AI vào một hệ thống legacy đang vận hành. Kiến trúc không được giả định rằng NovaMart sẽ xây một core ERP mới hoặc dần thay thế MERLIN.

### 4.1 Các ràng buộc không được vi phạm

- MERLIN vẫn là system of record cho supplier order.
- Không thay thế hoặc sửa replenishment engine của MERLIN.
- Không chạm vào POS, không thêm code, route, credential hoặc runtime dependency từ AI đến POS.
- Không thay supplier EDI schema, supplier integration hoặc deadline 04:00.
- MERLIN phải tiếp tục chạy khi AI unavailable, late, invalid hoặc disabled.
- Inventory accuracy chỉ 78%; dữ liệu nightly có thể cũ đến 24 giờ và không phải real-time shelf truth.
- Dữ liệu loyalty cá nhân phải ở trong quốc gia tương ứng.
- POS records tiếp tục được giữ ít nhất 5 năm trong incumbent governed data estate; AI không tạo một bản sao retention mới nếu chưa được phê duyệt.

### 4.2 Chỉ có ba MERLIN extension points được hỗ trợ

1. **Nightly batch** — cung cấp snapshot theo đêm cho sidecar.
2. **Override table** — MERLIN đọc override hợp lệ trước khi tạo order.
3. **Review queue** — category manager review sau khi MERLIN tạo recommendation/order.

Bất kỳ scheduler hook, undocumented table, direct supplier call hoặc in-process plugin nào khác đều không được xem là integration hợp lệ.

## 5. Kiến trúc được chọn

### Architecture thesis

> Dùng một AI sidecar bất đồng bộ, fail-open, tích hợp với MERLIN chỉ qua nightly batch, override table và review queue. Statistical ML dự báo nhu cầu; deterministic optimizer tạo quantity bị giới hạn. MERLIN vẫn là hệ thống duy nhất tạo order và gửi supplier EDI. Nếu AI không tạo được output hợp lệ và đúng hạn, MERLIN tiếp tục min/max replenishment hiện tại.

### Phân biệt hai khái niệm dễ nhầm

- **Sidecar** là architecture/integration pattern: AI tồn tại bên cạnh MERLIN.
- **Shadow-assist** là rollout mode: AI chạy và được đánh giá nhưng chưa ghi ảnh hưởng vào MERLIN.

### Scope Release 1

- Cam kết: demand forecasting và bounded replenishment recommendation.
- Có thể pilot riêng: associate copilot read-only, tách khỏi POS và ordering.
- Deferred: production markdown write, online substitution và promotion-planning automation.

## 6. Các con số phải thống nhất trong mọi artifact và slide

| Chủ đề | Giá trị chuẩn | Cách hiểu |
|---|---:|---|
| Batch window | 22:00–04:00 | Constraint cố định |
| Retry stop | 01:30 | Không bắt đầu retry mới sau mốc này |
| Complete-set gate | 01:45 | Phải có đủ manifest, partition và version |
| AI write close | 02:00 | Quyết định thiết kế tạm thời; phải đo MERLIN P99 |
| MERLIN reserve | 02:00–04:00 | Yêu cầu MERLIN P99 ≤90 phút và ≥30 phút reserve |
| Forecast/stocked candidates | 27M/7,04M mỗi đêm | Hai population khác nhau |
| Conservative 6× test | 162M/42,24M | Test population, không phải observed result |
| Throughput target | 30.000 forecast/s; 15.000 optimization/s | Phải pass 3 lần liên tiếp ở 6× |
| Sidecar target path | 198,66 → 198,7 phút | Design calculation, chưa phải benchmark |
| Calculated headroom | 41,34 → 41,3 phút | Chưa phải measured evidence |
| Inventory accuracy | 78% | Phải dùng interval, age, cap và suppression |
| Cost | Khoảng 3,8 triệu USD/năm; ceiling 4 triệu USD | Cần observed bills và Finance approval |
| Kill switch | Ack ≤30 giây; reject write mới ≤60 giây; verified safe ≤5 phút | Phải được drill và đo |
| Pilot | 20 stores; 2 low-risk non-fresh categories; ≤5.000 rows trong measured cap; ≤100 reviews/đêm | Chỉ sau khi pass gate |

Mốc 03:15 trong đề bài chỉ là ví dụ về fallback. Thiết kế hiện tại dùng mốc close 02:00 để dành hai giờ cho MERLIN; nếu measured MERLIN P99 không đủ, mốc close phải được đưa sớm hơn.

## 7. Giải thích sáu artifact bắt buộc

### Artifact 1 — C4 Context, Container và flow

**Mục đích:** cho trainer thấy AI nằm ở đâu, ai có authority, dữ liệu đi qua boundary nào và failure có ảnh hưởng MERLIN hay không.

**Phải thể hiện:**

- Context: MERLIN, POS, supplier EDI, data warehouse, external data và human actors.
- Container: validation, feature/forecast pipeline, deterministic optimizer, certified adapter, monitoring/audit và disable control.
- Ba extension points được hỗ trợ.
- Batch flow, failure/fallback flow, kill-switch path và copilot flow nếu có.
- MERLIN là order authority; AI không có route đến POS, supplier hoặc EDI connector.

**File chuẩn:**

- [`../artifacts/03-c4-context.mmd`](../artifacts/03-c4-context.mmd)
- [`../artifacts/04-c4-container.mmd`](../artifacts/04-c4-container.mmd)
- [`../artifacts/05-request-and-batch-flows.md`](../artifacts/05-request-and-batch-flows.md)

**Owner:** LÊ NGUYỄN SỸ BÌNH — 122980 — ARD.

### Artifact 2 — Quantified NFR Table

**Mục đích:** chứng minh kiến trúc bằng target, metric, threshold, failure behavior và production gate thay vì dùng các câu chung chung như “highly scalable”.

**Mỗi dòng cần có:** area, target, design mechanism, failure behavior, metric/gate và operational owner.

**Nhóm NFR chính:**

- Deadline và capacity.
- Forecast/model quality.
- Inventory/data quality.
- Legacy isolation và fail-open.
- Copilot latency/offline operation.
- Food safety.
- Cost/FinOps.
- Residency/auditability.
- Operations, review capacity và ownership.

**File chuẩn:** [`../artifacts/07-nfr-table.md`](../artifacts/07-nfr-table.md).  
**Supporting calculation:** [`../artifacts/08-capacity-and-cost-check.md`](../artifacts/08-capacity-and-cost-check.md).  
**Owner:** TRẦN THANH PHỤNG — 218924 — PH2.

### Artifact 3 — Technology Selection Matrix

**Mục đích:** chứng minh các lựa chọn công nghệ/kiến trúc được quyết định bằng criteria và trade-off, không phải vì công nghệ phổ biến.

**Ba quyết định quan trọng hiện tại:**

1. Integration pattern — external fail-open sidecar.
2. Batch compute strategy — managed elastic batch.
3. Forecast model family — hybrid hierarchical statistical ML.

Mỗi quyết định cần options, weight tổng 100%, score có thể tính lại, selected option, rejected alternatives, accepted trade-off và sensitivity.

**File chuẩn:** [`../artifacts/10-technology-selection-matrix.md`](../artifacts/10-technology-selection-matrix.md).  
**Owner:** NGUYỄN HÒA — 122989 — PH2.

### Artifact 4 — ADR

**Mục đích:** ghi lại một quyết định kiến trúc quan trọng nhất và lý do chấp nhận trade-off.

**ADR được chọn:** dùng fail-open AI sidecar qua supported MERLIN extensions, bắt đầu bằng shadow-assist.

**Phải có:** context, decision drivers, options, decision, positive/negative consequences, risks, rollback và điều kiện revisit.

**File chuẩn:** [`../artifacts/11-adr.md`](../artifacts/11-adr.md).  
**Owner:** PHẠM THỊ THANH HUYỀN — 164955 — PH2.

### Artifact 5 — Top-five Risk Register

**Mục đích:** cho thấy đúng năm rủi ro kiến trúc cao nhất, mỗi rủi ro có signal, numeric threshold và safe response.

**Năm rủi ro hiện tại:**

1. Miss deadline 04:00.
2. Inventory uncertainty tạo quantity sai.
3. Promotion/Lunar New Year làm giảm quality hoặc capacity.
4. MERLIN adapter/override integrity failure.
5. Unsafe food-safety/allergen output.

Mỗi risk phải có likelihood, impact, score, mitigation, contingency, signal, threshold và owner. Cost, residency, review fatigue và provider outage vẫn được theo dõi nhưng có score thấp hơn top five.

**File chuẩn:** [`../artifacts/12-risk-register.md`](../artifacts/12-risk-register.md).  
**Owner:** TRẦM QUỐC THUẬN — 112287 — 24R-Humana.

### Artifact 6 — Variant B Specific

**Mục đích:** chứng minh nhóm hiểu legacy và có thể rollout AI mà không thay thế MERLIN.

**Phải trả lời đầy đủ:**

- Cái gì có thể thay đổi, không thể thay đổi và không an toàn để chạm vào?
- Ba extension points là gì?
- Vì sao chọn sidecar và loại các alternatives?
- Shadow-assist rollout hoạt động thế nào?
- Human review được giới hạn thế nào?
- Kill switch do ai kích hoạt, chặn write trong bao lâu và xử lý stale rows ra sao?
- AI failure và rollback đưa hệ thống về trạng thái nào?
- Các phase tăng influence mà không gradually replace MERLIN như thế nào?

**File chuẩn:** [`../artifacts/13-variant-b-artifact.md`](../artifacts/13-variant-b-artifact.md).  
**Owner:** TRẦN TRỌNG PHÚ — 197320 — PH2.

### Presentation integrator

**Owner:** ĐINH XUÂN DŨNG — 123015 — 24R-Humana.  
Vai trò: đảm bảo cả sáu artifact và tám slide dùng cùng thesis, scope, con số, owner, gate và failure behavior.

## 8. Sáu artifact liên kết với nhau như thế nào?

```text
A6 Legacy assessment
→ giới hạn solution space
→ A3 Technology matrix chọn sidecar/batch/model
→ A4 ADR ghi quyết định quan trọng nhất
→ A1 C4 + flows mô tả kiến trúc và failure path
→ A2 NFR chứng minh bằng số và production gate
→ A5 Risk register giám sát các failure mode cao nhất
```

Nếu các artifact kể các câu chuyện khác nhau, trainer có thể từ chối. Ví dụ không được để C4 nói sidecar, ADR nói event-driven, NFR giả định inventory real-time hoặc risk register dùng deadline khác.

## 9. Sườn trình bày PPTX tám slide

| Slide | Mục tiêu và nội dung chính | Thời gian | Artifact hỗ trợ |
|---:|---|---:|---|
| 1 | Nêu thesis ngay: MERLIN giữ authority; AI sidecar fail-open; bỏ AI thì truck vẫn chạy | 45 giây | A4, A6 |
| 2 | Legacy boundaries: can change/cannot change/unsafe; ba extension points | 45 giây | A6 |
| 3 | C4 architecture: snapshot → validation → forecast → deterministic optimizer → adapter → override table → MERLIN | 55 giây | A1 |
| 4 | Timeline 22:00–04:00; retry 01:30; complete 01:45; close 02:00; mọi lỗi trở thành no override | 55 giây | A1, A2 |
| 5 | Business outcomes, 27M/7,04M, 6×, storage assumption, 198,7/240 phút và khoảng 3,8M/4M USD | 55 giây | A2 |
| 6 | Ba technology decisions, sidecar ADR, accepted trade-off và sensitivity | 50 giây | A3, A4 |
| 7 | Đúng năm risks: score, critical trigger và safe response | 50 giây | A5 |
| 8 | Discovery/replay → shadow → pilot → scale; kill timing; kết luận MERLIN vẫn orders | 55 giây | A6 |
|  | **Tổng** | **410 giây = 6 phút 50 giây** |  |

File presentation:

- [`../presentation/01-eight-slide-outline.md`](../presentation/01-eight-slide-outline.md)
- [`../presentation/02-speaker-notes.md`](../presentation/02-speaker-notes.md)
- [`../presentation/03-trainer-qa.md`](../presentation/03-trainer-qa.md)
- [`../presentation/04-one-page-cheat-sheet.md`](../presentation/04-one-page-cheat-sheet.md)
- [`../output/slides/Group4_VariantB_End_to_End_AI_System_Design.pptx`](../output/slides/Group4_VariantB_End_to_End_AI_System_Design.pptx)

## 10. Cách trình bày trong bảy phút

### Mở đầu

Không kể lại scenario NovaMart. Bắt đầu trực tiếp bằng quyết định:

> Kiến trúc của nhóm giữ MERLIN là hệ thống authoritative và đưa AI vào dưới dạng một fail-open sidecar qua ba extension points được hỗ trợ. AI cải thiện replenishment, nhưng lỗi AI không bao giờ được ngăn cửa hàng nhận chuyến hàng buổi sáng.

### Công thức trình bày mỗi slide

```text
Decision
→ Evidence hoặc calculation
→ Trade-off
→ Failure behavior
```

Ví dụ slide deadline:

```text
Decision: đóng AI write lúc 02:00.
Evidence: sidecar target path 198,7/240 phút.
Trade-off: kết quả AI muộn bị bỏ qua dù có thể hữu ích.
Failure: không ghi override; MERLIN tiếp tục min/max và EDI.
```

### Nguyên tắc nói

- Mỗi slide chỉ có một kết luận chính.
- Nói con số cùng cách hiểu: fact, target, assumption hay measured gate.
- Không đọc toàn bộ chữ trên slide.
- Không nói “highly scalable” hoặc “safe” nếu không kèm metric/threshold.
- Không cố bảo vệ một assumption như measured evidence.
- Khi bị hỏi sâu, trả lời theo bốn bước: decision, proof, trade-off, fallback.
- Kết thúc trước 7 phút để không mất thời gian Q&A.

### Câu kết

> AI cải thiện MERLIN nhưng không thay thế MERLIN. Nếu AI lỗi, bị trễ hoặc bị tắt, replenishment và supplier EDI vẫn tiếp tục bằng logic hiện tại.

## 11. Câu hỏi trainer có khả năng hỏi

1. Vì sao 02:00 là AI close hợp lý nếu chưa có measured MERLIN P99?
2. Làm sao chứng minh 27M forecast và peak 162M hoàn thành đúng hạn?
3. AI unavailable lúc 03:30 thì có chạy lại hoặc ghi late override không?
4. Có bất kỳ dependency nào khiến MERLIN phải chờ AI không?
5. Inventory 78% thì quantity được giới hạn như thế nào?
6. Category managers có xử lý được review queue không?
7. Kill switch có chặn được stale write và compromised adapter không?
8. Vì sao không dùng event-driven integration hoặc API gateway?
9. Vì sao LLM không được dùng cho forecast, replenishment quantity, price hoặc safety output?
10. Allergen answer được citation, version, refusal và escalation như thế nào?
11. Data residency được enforce bằng control nào?
12. Chi phí 3,8 triệu USD gồm những gì và đã có provider bill chưa?
13. Team có đủ primary/backup operator để vận hành 21:30–04:30 không?
14. Điều kiện nào cho phép chuyển từ shadow sang pilot và controlled influence?

## 12. Checklist trước khi nộp và trình bày

- [ ] Có đúng sáu canonical artifact.
- [ ] Mỗi artifact có đúng named owner.
- [ ] PPTX có tối đa tám slide và tổng script không quá bảy phút.
- [ ] MERLIN vẫn là order system of record và EDI sender.
- [ ] POS không bị sửa hoặc phụ thuộc vào AI.
- [ ] EDI schema và deadline 04:00 không đổi.
- [ ] Chỉ dùng nightly batch, override table và review queue.
- [ ] AI failure luôn dẫn đến no override; MERLIN không chờ.
- [ ] Inventory 78% được xử lý bằng uncertainty, interval, age, cap và suppression.
- [ ] Có proof cho 27M, 6× peak, throughput, duration và cost.
- [ ] Không dùng LLM cho numeric decision hoặc generated safety output.
- [ ] Safety output có exact evidence/citation hoặc refusal/escalation.
- [ ] Kill switch có owner, timing, stale-write rejection, independent revoke và audit.
- [ ] Mọi target/assumption/gate được phân biệt với measured evidence.
- [ ] Cả sáu artifact và tám slide dùng cùng thesis, con số và failure behavior.

## 13. Nguồn tra cứu nhanh

- Requirement chính: [`req_analysis_vi.md`](req_analysis_vi.md)
- Decision, number và owner registry: [`../artifacts/02-architecture-decision-summary.md`](../artifacts/02-architecture-decision-summary.md)
- Danh sách sáu artifact chuẩn: [`../artifacts/14-final-artifact-pack.md`](../artifacts/14-final-artifact-pack.md)
- Requirement coverage: [`../reviews/requirement-coverage-matrix.md`](../reviews/requirement-coverage-matrix.md)
- Final review: [`../reviews/final-review.md`](../reviews/final-review.md)

