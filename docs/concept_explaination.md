# 1. Trước hết: tách kiến trúc thành ba tầng

Các thuật ngữ trong workshop thuộc ba tầng khác nhau:

```text
    Tầng 1 — Dự báo
    Statistical ML
    → Demand forecast
    → Forecast quantiles / confidence interval

    Tầng 2 — Quyết định đặt hàng
    Deterministic optimizer
    → Case pack
    → Shelf life
    → Inventory uncertainty
    → DC capacity
    → Category caps
    → Recommendation

    Tầng 3 — Tích hợp và kiểm soát
    Certified adapter
    → Override table
    → MERLIN native generation
    → Review queue
    → Supplier EDI
```

Điểm quan trọng:

> **ML dự báo nhu cầu, nhưng ML không trực tiếp quyết định số lượng đơn hàng cuối cùng và không gửi order cho supplier.**

MERLIN vẫn là hệ thống có thẩm quyền tạo official order và gửi EDI.

---

# 2. WAPE

## 2.1 WAPE là gì?

**WAPE — Weighted Absolute Percentage Error** là chỉ số đo tổng sai số dự báo so với tổng nhu cầu thực tế.

Công thức:

[
WAPE =
\frac{\sum |Actual_i - Forecast_i|}
{\sum Actual_i}
\times 100%
]

Trong đó:

* `Actual`: nhu cầu thực tế.
* `Forecast`: nhu cầu dự báo.
* `|Actual - Forecast|`: độ lệch tuyệt đối, không quan tâm dự báo cao hay thấp.
* Tổng sai số được chia cho tổng nhu cầu thực tế.

## 2.2 Ví dụ

| SKU     | Actual | Forecast | Sai số tuyệt đối |
| ------- | -----: | -------: | ---------------: |
| Sữa     |    100 |       90 |               10 |
| Bánh mì |     50 |       70 |               20 |
| Nước    |    150 |      140 |               10 |
| Tổng    |    300 |      300 |               40 |

[
WAPE = \frac{40}{300} \times 100% = 13.33%
]

Ý nghĩa:

> Tổng sai lệch dự báo tương đương khoảng 13,33% tổng nhu cầu thực tế.

## 2.3 WAPE thấp hay cao là tốt?

* WAPE thấp: dự báo tốt hơn.
* WAPE cao: dự báo kém hơn.
* WAPE = 0%: dự báo hoàn toàn chính xác.
* WAPE = 25%: tổng sai số bằng 25% tổng nhu cầu thực tế.

Trong workshop:

* Hiện tại: WAPE khoảng 41%.
* Mục tiêu: WAPE không quá 25%.

Điều này không có nghĩa mọi SKU đều phải có sai số dưới 25%. WAPE là chỉ số tổng hợp có trọng số theo volume. SKU bán nhiều ảnh hưởng đến kết quả nhiều hơn SKU bán ít.

## 2.4 Khi nào sử dụng WAPE?

WAPE thường được dùng khi:

* Đánh giá demand forecasting trên nhiều SKU.
* Các SKU có volume rất khác nhau.
* Muốn có một chỉ số tổng hợp dễ trình bày với business.
* Muốn SKU bán nhiều có trọng số lớn hơn SKU ít bán.

## 2.5 Điểm yếu của WAPE

WAPE không nên là chỉ số duy nhất vì:

* SKU có volume lớn có thể che khuất lỗi ở SKU volume nhỏ.
* Không cho biết model thường dự báo cao hay thấp.
* Không trực tiếp phản ánh out-of-stock hoặc waste.
* Khi tổng actual bằng 0 thì WAPE không tính được.

Vì vậy nên bổ sung:

* Bias.
* WAPE theo category.
* WAPE theo store.
* WAPE theo promotion/non-promotion.
* Quantile coverage.
* Out-of-stock rate.
* Fresh waste rate.

---

# 3. Demand forecast WAPE

**Demand forecast WAPE** không phải một khái niệm khác WAPE. Nó chỉ rõ rằng WAPE đang được dùng để đo chất lượng của **demand forecast**.

Ví dụ:

```text
Forecast demand ngày mai: 120 hộp sữa
Actual demand ngày mai:   100 hộp
Absolute error:            20 hộp
```

Nếu tính trên toàn bộ store–SKU:

```text
Demand forecast WAPE
= Tổng sai số tuyệt đối của các forecast
  / Tổng demand thực tế
```

Trong workshop, đây là KPI đánh giá model dự báo, nhưng không đủ để quyết định model có được production hay không.

Một model có WAPE tốt vẫn có thể bị từ chối nếu:

* Chạy không kịp trước deadline.
* Confidence interval không được hiệu chỉnh tốt.
* Chi phí vượt ngân sách.
* Gây quá nhiều recommendation nguy hiểm.
* Hoạt động kém trong promotion hoặc fresh category.

---

# 4. Statistical ML

## 4.1 Statistical ML là gì?

**Statistical machine learning** là nhóm phương pháp sử dụng dữ liệu lịch sử, thống kê và thuật toán học máy để tìm quy luật và dự báo kết quả.

Trong bài toán NovaMart, input có thể gồm:

* Sales history.
* Day of week.
* Seasonality.
* Promotion.
* Holiday.
* Store.
* SKU.
* Category.
* Price.
* Stock availability.
* Historical out-of-stock.
* Weather nếu được phép và có giá trị.

Output có thể là:

```text
Store 101
SKU MILK-01
Date 2026-07-25

Expected demand: 42 units
P10: 31 units
P50: 42 units
P90: 57 units
Model version: demand-v17
```

## 4.2 Vì sao gọi là statistical ML?

Vì model không chỉ trả lời bằng quy tắc cố định như:

```text
Ngày mai lấy trung bình 7 ngày gần nhất.
```

Model học từ dữ liệu để nhận ra:

* Cuối tuần bán nhiều hơn.
* Promotion làm demand tăng.
* Một số store có pattern khác nhau.
* Lunar New Year tạo peak khác ngày thường.
* Sản phẩm mới cần mượn signal từ category tương tự.

## 4.3 Khi nào sử dụng?

Statistical ML phù hợp khi:

* Có dữ liệu lịch sử đủ lớn.
* Có pattern lặp lại.
* Cần dự báo số lượng.
* Cần xử lý hàng triệu time series.
* Cần uncertainty hoặc forecast quantiles.
* Có thể đánh giá bằng replay/backtesting.

## 4.4 Sử dụng ở đâu trong kiến trúc?

Nó nằm trong container:

```text
Statistical Forecast ML
```

Luồng:

```text
Validated snapshot
→ Feature preparation
→ Statistical ML
→ Forecast point / quantiles
→ Deterministic optimizer
```

Nó không nằm trong:

* MERLIN core.
* POS.
* Supplier EDI.
* Final order submission.

## 4.5 Vì sao không dùng LLM?

Demand forecasting là bài toán số liệu và time series, không phải bài toán ngôn ngữ.

LLM thường không phù hợp vì:

* Không đảm bảo tính ổn định của số lượng.
* Chi phí lớn ở quy mô 27 triệu forecasts.
* Khó tái lập chính xác kết quả.
* Có nguy cơ tạo ra số không dựa trên dữ liệu.
* Khó chứng minh statistical calibration.

---

# 5. Confidence interval và forecast quantile

## 5.1 Point forecast chưa đủ

Một model có thể nói:

```text
Ngày mai bán 100 đơn vị.
```

Đây là **point forecast** — một con số dự báo trung tâm.

Nhưng không cho biết độ không chắc chắn.

Hai trường hợp sau đều có point forecast 100:

```text
Trường hợp A: gần như chắc chắn demand từ 95 đến 105
Trường hợp B: demand có thể từ 40 đến 180
```

Rủi ro của hai trường hợp hoàn toàn khác nhau.

---

## 5.2 Forecast quantile là gì?

Forecast quantile là mức demand mà model dự đoán với một xác suất nhất định.

Ví dụ:

| Quantile | Giá trị | Diễn giải đơn giản                      |
| -------- | ------: | --------------------------------------- |
| P10      |      70 | Khoảng 10% khả năng demand thấp hơn 70  |
| P50      |     100 | Median forecast                         |
| P90      |     145 | Khoảng 90% khả năng demand thấp hơn 145 |

Có thể tạo interval:

```text
P10–P90 = 70–145
```

Đây là khoảng bất định của forecast.

## 5.3 Confidence interval khác forecast interval thế nào?

Trong ngữ cảnh dự báo, tài liệu thường gọi đơn giản là `confidence interval`, nhưng thuật ngữ chính xác hơn thường là:

> **Prediction interval hoặc forecast interval**

* Confidence interval thường mô tả độ không chắc chắn của một tham số, ví dụ mean.
* Prediction interval mô tả khoảng mà observation tương lai có khả năng rơi vào.
* Forecast quantiles tạo ra các ngưỡng như P10, P50, P90.

Trong workshop, điều quan trọng không phải tên gọi mà là:

> Hệ thống phải biết forecast này chắc chắn đến mức nào để quyết định có cho AI influence hay không.

## 5.4 Khi nào sử dụng?

Sử dụng khi:

* Demand biến động mạnh.
* Inventory không chính xác.
* Sản phẩm fresh có shelf life ngắn.
* Promotion có uncertainty cao.
* Cần tính safety stock.
* Cần suppress recommendation có rủi ro cao.

## 5.5 Sử dụng như thế nào?

Ví dụ:

```text
Forecast:
P10 = 60
P50 = 90
P90 = 140

Inventory estimated:
Lower bound = 20
Upper bound = 45
```

Optimizer phải xem xét cả các trường hợp:

* Demand thấp, inventory cao.
* Demand cao, inventory thấp.

Nếu chỉ dùng:

```text
Demand = 90
Inventory = 30
```

hệ thống đang giả vờ rằng cả hai con số đều chính xác.

---

# 6. Deterministic replenishment constraints

## 6.1 Là gì?

Đây là các quy tắc đặt hàng xác định rõ ràng, có thể tái lập và kiểm tra được.

**Deterministic** nghĩa là:

> Với cùng input và cùng ruleset, hệ thống luôn tạo cùng output.

Ví dụ:

```text
Nếu case pack = 12
thì không được recommend 17 units.
Có thể recommend 12, 24, 36...
```

ML dự báo demand. Deterministic optimizer biến forecast thành recommendation hợp lệ.

## 6.2 Các constraint trong workshop

### Case pack

Supplier chỉ giao theo thùng hoặc gói cố định.

```text
Case pack = 12 chai
Recommendation hợp lệ: 12, 24, 36
Recommendation không hợp lệ: 10, 17, 25
```

### Shelf life

Sản phẩm fresh có thời hạn sử dụng ngắn.

Ví dụ:

```text
Forecast 7 ngày = 100
Nhưng sản phẩm chỉ còn hạn sử dụng 2 ngày
```

Không thể đặt 100 chỉ vì forecast 7 ngày nói như vậy.

### DC capacity

Distribution centre có giới hạn:

* Storage.
* Picking.
* Loading.
* Truck capacity.

Một recommendation đúng ở mức SKU có thể không khả thi ở mức toàn DC.

### Inventory interval

Inventory không được xem là một số chắc chắn.

```text
Inventory lower bound = 10
Inventory upper bound = 25
```

Recommendation cần an toàn trong phạm vi uncertainty này.

### Category caps

Đặt giới hạn theo loại hàng.

Ví dụ:

```text
Fresh dairy:
Không tăng quá 1 case pack trong pilot.

Dry goods:
Có thể tăng tối đa 3 case packs.
```

### Absolute caps

Giới hạn tuyệt đối để tránh model tạo recommendation quá lớn.

```text
Không cho AI tăng quá 24 units/store–SKU/run.
```

## 6.3 Khi nào sử dụng?

Luôn sử dụng sau forecast và trước khi ghi recommendation vào MERLIN.

```text
ML forecast
→ Deterministic constraints
→ Safe recommendation
```

## 6.4 Vì sao không để ML tự học toàn bộ?

Vì có những quy tắc:

* Mang tính hợp đồng.
* Mang tính vật lý.
* Liên quan food safety.
* Liên quan supplier packaging.
* Phải giải thích được.
* Không được phép vi phạm dù model cho rằng có lợi.

---

# 7. Certified adapter

## 7.1 Adapter là gì?

Adapter là thành phần chuyển đổi giữa sidecar và MERLIN.

Nó chịu trách nhiệm:

* Đọc recommendation từ sidecar.
* Chuyển sang schema MERLIN hỗ trợ.
* Kiểm tra quyền ghi.
* Ghi vào override table.
* Quản lý transaction.
* Trả publication evidence.

## 7.2 Vì sao phải “certified”?

MERLIN là SAP system cũ, chỉ cho phép tích hợp thông qua supported extension points và SAP-certified partner.

Certified adapter nghĩa là adapter:

* Dùng interface đã được phê duyệt.
* Không đọc/ghi undocumented table.
* Có schema contract.
* Có transaction semantics được kiểm chứng.
* Được security và SAP partner review.
* Có rollback và kill-switch behavior.
* Không làm MERLIN chờ AI.

## 7.3 Adapter không được làm gì?

* Không tạo supplier order.
* Không gửi EDI.
* Không thay đổi POS.
* Không ghi row của run cũ.
* Không ghi recommendation vượt cap.
* Không xóa manual override.
* Không đọc private internal MERLIN tables.
* Không bypass review hoặc authorization.

## 7.4 “Chỉ ghi bounded, current-run rows” nghĩa là gì?

### Bounded

Mỗi recommendation phải nằm trong giới hạn được cho phép.

Ví dụ:

```text
Pilot cap:
Tối đa 2 case packs/store–SKU.
```

Adapter phải reject recommendation 10 case packs dù model đưa ra con số đó.

### Current-run

Recommendation phải thuộc đúng batch hiện tại.

Ví dụ:

```text
Current run ID: 2026-07-24-NIGHT
```

Adapter phải reject row của:

```text
2026-07-23-NIGHT
```

để tránh stale recommendation ảnh hưởng order mới.

---

# 8. Supplier EDI

## 8.1 EDI là gì?

**EDI — Electronic Data Interchange** là cơ chế trao đổi tài liệu kinh doanh điện tử theo schema chuẩn giữa các công ty.

Ví dụ các tài liệu:

* Purchase order.
* Order acknowledgement.
* Shipping notice.
* Invoice.

Trong workshop, MERLIN gửi supplier order cho khoảng 2.300 supplier thông qua EDI.

## 8.2 Supplier EDI nằm ở đâu?

```text
AI sidecar
→ Recommendation
→ MERLIN override table
→ MERLIN creates official order
→ MERLIN sends supplier EDI
→ Supplier
```

Sidecar không có kết nối trực tiếp tới supplier.

## 8.3 Vì sao không thay đổi EDI?

Vì thay đổi schema có thể yêu cầu:

* Phối hợp lại với 2.300 supplier.
* Certification.
* Regression testing.
* Contract changes.
* Operational rollout rất lớn.

Đây là thay đổi quá rủi ro cho chương trình tám tháng.

---

# 9. Fresh waste

## 9.1 Là gì?

**Fresh waste** là lượng hoặc tỷ lệ hàng tươi sống bị bỏ đi do:

* Hết hạn.
* Giảm chất lượng.
* Không bán hết.
* Bị hư hỏng.
* Không còn đáp ứng tiêu chuẩn bán hàng.

Ví dụ:

```text
Cửa hàng nhận 1.000 đơn vị fresh food
Có 48 đơn vị phải bỏ đi

Fresh waste rate = 48 / 1.000 = 4,8%
```

Trong workshop:

* Hiện tại: 4,8%.
* Mục tiêu: 3,0%.

## 9.2 Vì sao liên quan forecasting?

Dự báo quá cao:

```text
Order quá nhiều
→ Hàng không bán hết
→ Hết hạn
→ Fresh waste tăng
```

Dự báo quá thấp:

```text
Order quá ít
→ Out-of-stock
→ Mất doanh thu
```

Kiến trúc phải cân bằng:

```text
Availability ↔ Waste
```

Không thể chỉ tối ưu WAPE mà bỏ qua fresh waste.

---

# 10. Thời gian nhân viên tra cứu: từ 50 phút xuống dưới 15 phút mỗi ca

## 10.1 Đây là chỉ số gì?

Đây là operational KPI đo tổng thời gian store associate dành để tìm thông tin trong một ca làm việc.

Ví dụ:

* Tìm vị trí sản phẩm.
* Tìm thông tin sản phẩm.
* Kiểm tra policy.
* Kiểm tra promotion.
* Tìm hướng dẫn thao tác.
* Tra cứu thông tin hàng hóa.

Hiện tại:

```text
Khoảng 50 phút/nhân viên/ca
```

Mục tiêu:

```text
Dưới 15 phút/nhân viên/ca
```

## 10.2 Tại sao dùng chỉ số này?

Mục tiêu của associate copilot không nhất thiết là giảm số nhân viên.

Nó giúp nhân viên:

* Trả lời khách nhanh hơn.
* Giảm thời gian chuyển qua nhiều hệ thống.
* Giảm việc hỏi supervisor.
* Tăng thời gian dành cho khách hàng và vận hành cửa hàng.

## 10.3 Cần đo như thế nào?

Không nên chỉ đo thời gian chatbot trả lời.

Cần đo end-to-end:

```text
Từ lúc nhân viên phát sinh câu hỏi
→ tìm được câu trả lời đủ tin cậy
→ hoàn thành công việc
```

Nếu copilot trả lời nhanh nhưng sai, KPI không có giá trị.

---

# 11. Fresh markdown và dynamic pricing

## 11.1 Fresh markdown là gì?

Fresh markdown là giảm giá hàng fresh để bán trước khi hết hạn.

Ví dụ:

```text
Sản phẩm: Sandwich
Giá gốc: $5
Còn hạn: 6 giờ
Inventory: 30
Demand dự kiến: 8

Đề xuất markdown: giảm 30%
```

Mục tiêu:

* Giảm fresh waste.
* Thu hồi một phần doanh thu.
* Tránh phải bỏ sản phẩm.

## 11.2 Dynamic pricing là gì?

Dynamic pricing là thay đổi giá dựa trên các yếu tố như:

* Demand.
* Inventory.
* Thời gian còn lại trước expiry.
* Store.
* Time of day.
* Promotion.
* Competitor price, nếu hợp lệ.

Fresh markdown là một trường hợp cụ thể của dynamic pricing.

## 11.3 Khi nào sử dụng?

Có thể sử dụng khi:

* Có dữ liệu expiry đáng tin cậy.
* Có electronic shelf labels hoặc supported price interface.
* Có legal approval.
* Có policy về minimum price.
* Có audit.
* Không tạo discriminatory pricing.
* POS và shelf price đồng bộ được.

## 11.4 Vì sao chưa tự động hóa ngay trong Release 1?

Vì giá hiển thị, giá POS và giá khách thanh toán phải nhất quán.

Nếu AI thay đổi giá nhưng:

* POS chưa cập nhật.
* Shelf label chưa cập nhật.
* Store offline.
* Promotion conflict.

thì có thể phát sinh legal và customer trust issue.

Do đó workshop có thể chỉ cho phép:

```text
Offline analysis hoặc recommendation
```

thay vì tự động ghi giá production.

---

# 12. Store associate copilot

## 12.1 Là gì?

Đây là trợ lý AI hỗ trợ nhân viên cửa hàng tìm thông tin và thực hiện công việc.

Ví dụ câu hỏi:

* “Sản phẩm này nằm ở aisle nào?”
* “Promotion này còn hiệu lực không?”
* “Quy trình xử lý hàng hư là gì?”
* “Chính sách đổi trả áp dụng thế nào?”
* “SKU này có thành phần gì?”

## 12.2 Copilot không phải autonomous agent toàn quyền

Copilot nên:

* Tìm thông tin.
* Trích xuất dữ liệu.
* Tóm tắt tài liệu không liên quan safety.
* Trả lời có nguồn.
* Refuse khi không có bằng chứng.

Copilot không nên:

* Tự thay đổi inventory.
* Tự tạo order.
* Tự thay đổi giá.
* Tự gửi EDI.
* Tạo nội dung allergen từ trí nhớ model.

## 12.3 Nằm ở đâu trong kiến trúc?

Copilot là boundary riêng:

```text
Associate device
→ Copilot service
→ Approved knowledge sources
```

Không nằm trong:

```text
POS critical transaction path
MERLIN order generation
Supplier EDI
```

Việc tách này ngăn lỗi copilot ảnh hưởng bán hàng hoặc ordering.

---

# 13. Post-generation review queue

## 13.1 Là gì?

Đây là queue để category manager xem order sau khi MERLIN đã tạo.

Luồng:

```text
AI recommendation
→ Override table
→ MERLIN native generation
→ Official order
→ Post-generation review queue
→ Category manager review
```

## 13.2 Tại sao review sau generation?

AI recommendation chưa phải order thực tế.

MERLIN còn áp dụng các rule nội bộ như:

* Supplier constraints.
* Existing orders.
* Case rounding.
* Minimum order.
* DC allocation.
* Manual overrides.
* Inventory logic.
* Order consolidation.

Ví dụ:

```text
AI recommendation: 24
MERLIN final order: 12
```

Category manager cần thấy official result của MERLIN, không chỉ AI recommendation.

## 13.3 Khi nào sử dụng?

* Pilot.
* High-risk category.
* Recommendation vượt threshold.
* Fresh item.
* Inventory confidence thấp.
* Promotion.
* Model disagreement.
* Category manager cần approve exception.

## 13.4 Queue không nên trở thành bottleneck

Nếu AI tạo hàng trăm nghìn review mỗi đêm thì queue không còn hữu ích.

Do đó cần:

* Bounded review volume.
* Risk-based sampling.
* Exception-only review.
* Pilot cap, ví dụ tối đa 100 reviews/night.

---

# 14. Không chạy “late recovery” sau khi cửa sổ an toàn đóng

## 14.1 Late recovery là gì?

Giả sử AI đáng lẽ phải hoàn thành trước 02:00 nhưng bị lỗi.

Đến 03:30 AI phục hồi và tạo được recommendation.

Chạy late recovery nghĩa là cố ghi recommendation lúc 03:30 để ảnh hưởng order hiện tại.

Thiết kế cấm việc này.

## 14.2 Vì sao?

Vì lúc 03:30:

* MERLIN có thể đã tạo order.
* Category manager có thể đang review.
* EDI có thể đang được chuẩn bị.
* Một số order có thể đã gửi.
* Chỉ còn 30 phút trước hard deadline 04:00.

Ghi late override có thể tạo:

* Race condition.
* Partial update.
* Duplicate order.
* Inconsistent review.
* Missed deadline.
* Không đủ thời gian rollback.

## 14.3 Hành vi đúng

```text
AI recovered at 03:30
→ Do not write current-run recommendation
→ Store diagnostic result
→ Investigate incident
→ Apply improvement to next run
```

MERLIN tiếp tục bằng incumbent min/max logic.

---

# 15. Các container trong sidecar

## 15.1 Snapshot validation

### Mục đích

Ngăn dữ liệu không hợp lệ đi vào model.

### Freshness

Dữ liệu có đủ mới không?

```text
Expected snapshot: 22:00 hôm nay
Actual snapshot:   22:00 hôm qua
```

Dữ liệu 24 giờ cũ có thể phải bị reject.

### Completeness

Có thiếu store, SKU hoặc partition không?

Ví dụ:

```text
Expected: 1.200 stores
Received: 1.145 stores
```

Không nên coi batch là hoàn tất.

### Schema

Cấu trúc dữ liệu có đúng không?

```text
Expected inventory_qty: integer
Received inventory_qty: text
```

### Checksum

File có bị thay đổi hoặc hỏng trong quá trình truyền không?

Checksum dùng để xác minh nội dung nhận được đúng với nội dung đã xuất.

### Residency

Dữ liệu có được xử lý và lưu ở đúng quốc gia không?

Ví dụ loyalty personal data của country A không được copy sang region B.

### Khi validation fail

Không cố “đoán” dữ liệu còn thiếu.

Phản ứng an toàn:

```text
Reject partition
Suppress affected scope
Let MERLIN use incumbent logic
```

---

## 15.2 Statistical forecast ML

Nhiệm vụ:

* Nhận validated features.
* Tạo forecast.
* Tạo quantiles.
* Gắn model version.
* Gắn confidence/calibration metadata.

Output ví dụ:

```json
{
  "store_id": "S101",
  "sku": "MILK-01",
  "forecast_date": "2026-07-25",
  "p10": 31,
  "p50": 42,
  "p90": 57,
  "model_version": "demand-v17"
}
```

Container này không biết EDI hoặc MERLIN order schema.

---

## 15.3 Deterministic optimizer

Nó chuyển forecast thành recommendation.

Input:

```text
Forecast quantiles
Inventory bounds
Case pack
Shelf life
DC capacity
Category limits
Existing business rules
```

Output:

```text
Recommended bounded override
hoặc
Suppressed
```

Ví dụ:

```text
P50 demand = 42
Inventory lower = 10
Case pack = 12
Raw need = 32

Rounded recommendation = 36
Category cap = 24

Final recommendation = 24
```

---

## 15.4 Certified adapter

Nhiệm vụ:

* Validate run ID.
* Validate bounds.
* Validate permit.
* Kiểm tra disable epoch.
* Chuyển sang MERLIN schema.
* Ghi transaction.
* Lưu publication evidence.
* Reject stale hoặc unauthorized rows.

Chỉ adapter này có quyền ghi override table.

---

## 15.5 Disable control

Đây là kill-switch control plane.

### Epoch

Epoch là version của trạng thái enable/disable.

Ví dụ:

```text
Epoch 105: ENABLED
Epoch 106: DISABLING
Epoch 107: DISABLED
```

Transaction bắt đầu ở epoch 105 nhưng commit sau khi epoch đã chuyển 106 phải bị reject.

### Short-lived permit

Adapter không được giữ quyền ghi vĩnh viễn.

Nó nhận permit ngắn hạn:

```text
Permit valid for 60 seconds
```

Nếu control plane ngừng cấp permit, write tự động dừng.

### Independent revoke

Có cơ chế thu hồi độc lập với chính adapter:

* Revoke DB credential.
* Revoke network access.
* Terminate session.
* Disable service account.

Điều này cần thiết khi adapter bị lỗi và không tự dừng.

---

## 15.6 Append-only audit

**Append-only** nghĩa là chỉ thêm record mới, không sửa hoặc xóa lịch sử cũ.

Audit lưu:

* Input manifest.
* Snapshot checksum.
* Model version.
* Ruleset version.
* Forecast.
* Quantiles.
* Recommendation.
* Suppression reason.
* Adapter publication result.
* Run ID.
* Timestamp.
* Operator action.

Ví dụ:

```text
Input snapshot X
+ Model v17
+ Ruleset v8
→ Recommendation 24
→ Published at 01:51
→ Consumed by MERLIN run Y
```

Mục đích:

* Điều tra incident.
* Giải thích quyết định.
* Reproduce result.
* Compliance.
* So sánh AI recommendation với MERLIN final order.

---

# 16. Vì sao MERLIN final quantity phải trả lại qua review queue?

## 16.1 Sidecar chỉ biết recommendation

Ví dụ:

```text
Sidecar recommendation: 24
```

Sau đó MERLIN có thể áp dụng thêm rule và tạo:

```text
MERLIN official quantity: 12
MERLIN order ID: PO-829173
```

Nếu sidecar muốn đo hiệu quả, nó cần biết:

* Recommendation nào dẫn tới order nào.
* MERLIN có chấp nhận recommendation không.
* Final quantity là bao nhiêu.
* Có manual change không.

## 16.2 Correlation fields là gì?

Correlation field là khóa dùng để liên kết các record giữa sidecar và MERLIN.

Ví dụ:

```text
run_id
recommendation_id
store_id
sku
business_date
MERLIN order_id
MERLIN order_line_id
```

Một contract có thể là:

```text
recommendation_id AI-20260724-001
→ MERLIN order line PO-829173-L12
```

## 16.3 Vì sao phải là certified fields?

Nếu correlation không chính thức, có thể xảy ra:

* Match nhầm order.
* Hai recommendation map vào cùng order.
* Manual order bị coi là AI order.
* Sai số lượng trong audit.
* Không thể chứng minh AI influence.

## 16.4 Vì sao không được đọc undocumented table?

Undocumented table là bảng nội bộ không có integration contract chính thức.

Rủi ro:

* SAP upgrade thay đổi schema.
* Field có ý nghĩa khác với giả định.
* Đọc dữ liệu chưa commit.
* Gây lock hoặc performance issue.
* Vi phạm support agreement.
* Bypass security boundary.

Vì vậy:

> Nếu review queue không cung cấp order ID, quantity và correlation key được chứng nhận, automated influence phải giữ trạng thái disabled.

Đây là **go-live dependency**, không phải chi tiết có thể xử lý sau.

---

# 17. Normal volume và six-times peak

## 17.1 Khoảng 27 triệu forecasts mỗi đêm

Có nghĩa batch phải tạo khoảng:

```text
27.000.000 forecast records/night
```

Một forecast record có thể đại diện cho:

```text
Store × SKU × forecast date hoặc forecast horizon
```

Không nên hiểu là có 27 triệu model riêng biệt.

Một model global hoặc hybrid có thể tạo hàng triệu output records.

## 17.2 Khoảng 7,04 triệu stocked candidates

Đây là số store–SKU pair có khả năng đi qua replenishment optimization.

Ví dụ:

```text
Store 101 × SKU A
Store 101 × SKU B
Store 102 × SKU A
...
```

Số forecast có thể lớn hơn số stocked candidates vì forecasting có thể bao gồm:

* Nhiều ngày forecast horizon.
* Nhiều target output.
* Series không đủ điều kiện order.
* Candidate phục vụ analytics nhưng không replenishment.

## 17.3 Tại sao six-times?

Lunar New Year được dùng như peak scenario:

[
27M \times 6 = 162M
]

[
7.04M \times 6 = 42.24M
]

Six-times test không nhất thiết có nghĩa cửa hàng bán chính xác gấp sáu. Đây là capacity test để chứng minh kiến trúc chịu được peak volume được yêu cầu.

---

# 18. Target throughput

## 18.1 Forecast: 30.000 records/second

Với 162 triệu records:

[
162.000.000 / 30.000
= 5.400\ giây
]

[
5.400 / 60 = 90\ phút
]

Như vậy forecast peak hoàn thành trong khoảng 90 phút.

## 18.2 Optimizer: 15.000 records/second

[
42.240.000 / 15.000
= 2.816\ giây
]

[
2.816 / 60 \approx 46,93\ phút
]

Optimizer peak hoàn thành trong khoảng 47 phút.

## 18.3 Hai thời gian này có cộng lại không?

Nếu chạy hoàn toàn tuần tự:

[
90 + 46,93 \approx 136,93\ phút
]

Sau đó còn:

* Snapshot validation.
* Retry.
* Reconciliation.
* Manifest completion.
* Publication.
* MERLIN processing.

Nếu hệ thống hỗ trợ streaming hoặc partition overlap, optimizer có thể bắt đầu trên partition đã forecast xong. Khi đó critical path có thể thấp hơn phép cộng đơn giản.

Nhưng nhóm phải benchmark hành vi thực tế, không được chỉ dựa trên lý thuyết.

---

# 19. Technology selection là gì?

Technology selection không đơn giản là chọn AWS, Azure, Kubernetes hoặc một model vì nhóm quen dùng.

Nó là quá trình:

1. Xác định tiêu chí.
2. Gán trọng số.
3. Chấm từng phương án.
4. Kiểm tra constraint loại trừ.
5. Thực hiện sensitivity analysis.
6. Xác nhận bằng benchmark hoặc replay.
7. Ghi nhận trade-off.

## Weighted score

Ví dụ:

| Tiêu chí             | Trọng số |
| -------------------- | -------: |
| Legacy safety        |      30% |
| Deadline feasibility |      25% |
| Rollback             |      15% |
| Cost                 |      15% |
| Operability          |      15% |

Mỗi phương án được chấm 1–5.

[
Weighted\ score =
\sum Score_i \times Weight_i
]

Điểm cao nhất không tự động thắng nếu vi phạm hard constraint.

Ví dụ một giải pháp có tổng điểm tốt nhưng phải sửa POS vẫn bị loại.

---

# 20. Integration pattern

Integration pattern trả lời:

> Sidecar tương tác với MERLIN theo cơ chế nào?

## 20.1 External asynchronous sidecar — 4.80

### External

Chạy ngoài MERLIN.

### Asynchronous

MERLIN không gọi API và chờ AI trả lời trong critical path.

### Sidecar

Hệ thống hỗ trợ đặt bên cạnh incumbent system.

### Ưu điểm

* Không sửa core.
* Failure isolation.
* Scale độc lập.
* Shadow được.
* Rollback độc lập.
* AI lỗi không chặn MERLIN.
* Phù hợp ba supported interfaces.

### Nhược điểm

* Duplicate data.
* Reconciliation phức tạp.
* Cần adapter.
* Có eventual consistency.
* Tăng operational components.

---

## 20.2 Event-driven boundary — 2.95

Mô hình:

```text
MERLIN event
→ Event bus
→ AI service
→ Result event
```

Phù hợp khi hệ thống legacy có:

* Supported event interface.
* Change-data contract.
* Reliable event IDs.
* Idempotency.
* Replay semantics.

Không phù hợp làm boundary chính trong workshop vì MERLIN không cung cấp supported real-time event interface.

Có thể dùng event-driven bên trong sidecar, nhưng không nên tự tạo một event integration không được hỗ trợ từ MERLIN.

---

## 20.3 Synchronous API gateway — 1.80

Mô hình:

```text
MERLIN
→ gọi AI API
→ chờ response
→ tạo order
```

Vấn đề:

* AI timeout có thể chặn MERLIN.
* Network outage ảnh hưởng critical path.
* Peak latency có thể làm miss 04:00.
* Provider outage trở thành ordering outage.
* Khó fail-open đúng nghĩa.

Chỉ phù hợp nếu:

* Dependency có SLA rất cao.
* Có latency budget rõ.
* Business chấp nhận fallback tức thời.
* Core system hỗ trợ API integration.

Không phù hợp ở đây vì yêu cầu AI không được làm hỏng ordering chain.

---

## 20.4 In-process MERLIN plugin — 1.30

ML code chạy bên trong MERLIN process hoặc SAP extension.

Ưu điểm:

* Ít network hop.
* Có thể truy cập dữ liệu trực tiếp.
* Transaction integration chặt.

Nhược điểm:

* Tăng blast radius.
* Khó scale ML.
* Khó rollback riêng.
* Cần SAP release.
* Chỉ hai release mỗi năm.
* Không có automated test suite.
* Có thể làm MERLIN unstable.

Do đó bị chấm thấp nhất.

---

# 21. Execution platform

Execution platform trả lời:

> Forecast và optimizer sẽ chạy trên hạ tầng nào?

## 21.1 Managed elastic batch — 4.65

Là dịch vụ batch có thể tự tăng giảm tài nguyên.

Đặc điểm:

* Tạo worker khi batch bắt đầu.
* Scale theo queue hoặc partition.
* Giảm worker sau khi hoàn tất.
* Hỗ trợ retry, timeout và deadline.
* Không cần giữ peak capacity quanh năm.

Phù hợp vì workload:

* Chạy nightly.
* Volume lớn.
* Có hard deadline.
* Peak theo mùa.
* Không cần service luôn hoạt động ở maximum scale.

Ví dụ công nghệ có thể thuộc nhóm:

* Managed batch compute.
* Serverless batch.
* Managed container jobs.
* Cloud-native workflow orchestration.

Workshop không nên khóa vào provider trước khi kiểm tra residency, benchmark và cost.

---

## 21.2 Kubernetes batch workers — 4.15

Chạy forecast workers dưới dạng Kubernetes Jobs.

Ưu điểm:

* Kiểm soát runtime tốt.
* Portable.
* Hỗ trợ custom container.
* Scale linh hoạt.
* Có thể chạy multi-cloud hoặc private environment.

Nhược điểm:

* Phải vận hành cluster.
* Capacity planning phức tạp.
* Scheduler tuning.
* Monitoring và upgrade.
* Có thể phải giữ baseline nodes.
* Operational burden cao hơn managed batch.

Phù hợp khi tổ chức đã có Kubernetes platform mạnh và team vận hành đủ năng lực.

---

## 21.3 Managed Spark — 3.90

Spark phù hợp với:

* Distributed data transformation.
* Large-scale feature engineering.
* Batch analytics.
* Parallel processing.

Nhưng không phải mọi forecasting workload đều cần Spark.

Nhược điểm có thể gồm:

* Startup overhead.
* Complexity.
* Khó tối ưu một số model inference workload.
* Chi phí nếu cluster sizing không tốt.
* Có thể là thành phần thừa nếu pipeline đơn giản.

Spark chỉ nên được chọn nếu benchmark chứng minh nó cần thiết.

---

## 21.4 Fixed on-premises compute — 3.10

Dùng cụm máy cố định trong data centre.

Ưu điểm:

* Kiểm soát dữ liệu.
* Dễ đáp ứng một số residency constraint.
* Không phụ thuộc cloud provider.
* Có thể tận dụng hạ tầng hiện có.

Nhược điểm:

* Phải mua đủ capacity cho peak 6×.
* Phần lớn thời gian tài nguyên nhàn rỗi.
* Scale chậm.
* Procurement lâu.
* Khó hoàn thành trong tám tháng.
* Operational maintenance cao.

---

# 22. Forecast model strategy

## 22.1 Hybrid hierarchical statistical ML — 4.55

### Hybrid

Kết hợp nhiều phương pháp hoặc nhiều cấp model.

Ví dụ:

* Global model cho phần lớn SKU.
* Specialized model cho fresh category.
* Classical fallback cho sparse series.
* Promotion adjustment model.
* New-product fallback.

### Hierarchical

Dữ liệu có cấu trúc phân cấp:

```text
Country
→ Region
→ Store
→ Category
→ SKU
```

Forecast ở các cấp phải hợp lý và có thể được reconcile.

Ví dụ:

```text
Tổng forecast của các store
nên phù hợp với regional forecast.
```

### Ưu điểm

* Chia sẻ signal giữa SKU/store.
* Hỗ trợ sparse series.
* Xử lý promotion tốt hơn.
* Không cần 27 triệu model.
* Có thể có category-specific behavior.

### Nhược điểm

* Phức tạp hơn.
* Nhiều model và fallback path.
* Khó vận hành hơn global model.
* Cần monitoring theo segment.

---

## 22.2 One global statistical model — 4.35

Một model được train trên nhiều store và SKU.

Input chứa:

```text
Store ID
SKU ID
Category
Historical demand
Promotion
Calendar features
```

Ưu điểm:

* Dễ chia sẻ signal.
* Ít model artifact.
* Scale inference tốt.
* Dễ cập nhật hơn per-series model.

Nhược điểm:

* Có thể không tối ưu cho category đặc biệt.
* Model lớn.
* Khó xử lý một số behavior ngoại lệ.
* Có nguy cơ performance tốt trung bình nhưng kém ở critical slices.

---

## 22.3 Classical baseline — 3.50

Các phương pháp như:

* Moving average.
* Exponential smoothing.
* Seasonal naïve.
* ARIMA/ETS.
* Same-day-last-week.

Ưu điểm:

* Dễ hiểu.
* Dễ benchmark.
* Chi phí thấp.
* Reproducible.
* Tốt làm fallback.

Nhược điểm:

* Có thể xử lý promotion hoặc complex interactions kém.
* Khó chia sẻ signal cho sparse products.
* Có thể không đạt WAPE target.

Baseline vẫn bắt buộc phải có để chứng minh model phức tạp thực sự tạo thêm giá trị.

---

## 22.4 Per-store-SKU model — 3.20

Tạo model riêng cho từng store–SKU.

Nếu có hàng triệu series thì có thể dẫn đến hàng triệu model.

Vấn đề:

* Train và deploy rất phức tạp.
* Nhiều series không đủ dữ liệu.
* Monitoring gần như không khả thi.
* Model lifecycle rất lớn.
* Chi phí cao.
* Promotion/new product khó xử lý.

Chỉ phù hợp khi số series nhỏ hoặc mỗi series có đủ dữ liệu và giá trị kinh doanh rất lớn.

---

# 23. Sensitivity analysis và replay evidence

## 23.1 Sensitivity analysis

Sensitivity analysis kiểm tra xem kết quả lựa chọn có thay đổi khi đổi trọng số không.

Ví dụ ban đầu:

```text
Accuracy weight: 40%
Cost weight:     20%

Hybrid thắng.
```

Sau đó đổi:

```text
Accuracy weight: 25%
Cost weight:     35%

Hybrid và global cùng đạt 4.45.
```

Điều này cho thấy lựa chọn hybrid không hoàn toàn áp đảo.

## 23.2 Replay evidence

Replay là chạy model trên dữ liệu lịch sử như thể đang ở thời điểm quá khứ.

Ví dụ:

```text
Dùng dữ liệu đến ngày 1/6
→ forecast ngày 2/6
→ so với actual ngày 2/6
```

Replay phải đánh giá:

* WAPE.
* Bias.
* Promotion.
* Fresh category.
* Inventory uncertainty.
* Out-of-stock impact.
* Waste impact.
* Runtime.
* Cost.
* Quantile calibration.

Vì vậy scoring matrix chỉ giúp shortlist. Model cuối cùng phải được quyết định bằng measured evidence.

---

# 24. Food-safety/legal

## 24.1 Food-safety risk là gì?

Bao gồm các tình huống AI:

* Trả lời sai allergen.
* Nhầm SKU.
* Dùng thông tin hết hạn.
* Tự dịch sai cảnh báo.
* Tóm tắt mất nội dung quan trọng.
* Đề xuất markdown cho sản phẩm không còn an toàn.
* Hiển thị safety information không được phê duyệt.

Hậu quả có thể là:

* Tổn hại sức khỏe.
* Legal liability.
* Product recall.
* Regulatory violation.
* Mất uy tín.

## 24.2 Cách xử lý

Đối với allergen và safety:

```text
Approved source
→ Exact retrieval
→ Validate SKU/country/version
→ Fixed rendering
→ Citation
```

Không dùng LLM để:

* Tự tạo câu trả lời.
* Paraphrase.
* Translate tự do.
* Suy luận thành phần.
* Điền thông tin thiếu.

Nếu thiếu evidence:

```text
Refuse
→ Ask associate to follow approved escalation process
```

---

# 25. Reconciliation

## 25.1 Là gì?

Reconciliation là đối chiếu dữ liệu giữa các bước hoặc các hệ thống để đảm bảo không mất, trùng hoặc sai record.

Ví dụ:

```text
Forecast produced:           27.000.000
Optimizer processed:          7.040.000 candidates
Recommendations generated:     500.000
Recommendations published:     499.950
Rejected by adapter:                50
Consumed by MERLIN:            499.900
Unexplained rows:                   50
```

Reconciliation phải giải thích 50 unexplained rows là gì.

## 25.2 Các loại reconciliation

### Input reconciliation

Expected snapshot có đủ không?

```text
Expected 1.200 stores
Received 1.200
```

### Processing reconciliation

Mọi partition đã xử lý chưa?

```text
Started partitions = completed + failed + suppressed
```

### Publication reconciliation

Mọi recommendation được:

* Published.
* Rejected.
* Suppressed.

Không được biến mất không lý do.

### MERLIN reconciliation

Recommendation nào được MERLIN sử dụng?

* Final quantity.
* Order ID.
* Manual modification.
* Not consumed.
* Rejected.

## 25.3 Khi nào thực hiện?

* Sau snapshot.
* Sau forecast.
* Sau optimization.
* Trước publish close.
* Sau MERLIN generation.
* Trong incident investigation.

---

# 26. “Production chỉ được phép khi benchmark đạt ba lần six-times liên tiếp”

## 26.1 Benchmark là gì?

Benchmark là bài kiểm tra có kiểm soát để đo:

* Throughput.
* Latency.
* Error rate.
* Memory.
* Storage.
* Cost.
* Recovery.
* Adapter behavior.

## 26.2 Vì sao phải ba lần liên tiếp?

Một lần pass có thể do:

* Cache thuận lợi.
* Network tốt bất thường.
* Không có resource contention.
* Data partition dễ.
* May mắn.

Ba lần liên tiếp chứng minh kết quả có tính ổn định hơn.

Điều kiện pass nên gồm:

```text
3/3 runs:
- Không miss deadline
- Không data loss
- Không unexplained row
- Throughput đạt target
- Cost nằm trong envelope
- Kill switch hoạt động
- MERLIN không bị block
```

## 26.3 P99 ≤90 phút nghĩa là gì?

P99 là percentile 99.

Nếu MERLIN P99 generation time là 90 phút:

> Khoảng 99% các run được đo hoàn thành trong không quá 90 phút.

Không có nghĩa mọi run đều dưới 90 phút.

Ví dụ 1.000 run:

* Khoảng 990 run ≤90 phút.
* Khoảng 10 run có thể >90 phút.

Do deadline là 04:00, kiến trúc cần thêm buffer chứ không thể dùng đúng 90 phút làm lịch chạy sát mép.

---

# 27. Inventory chỉ chính xác 78%

## 27.1 Không dùng inventory như point truth

Point truth nghĩa là tin rằng:

```text
Inventory = 20
```

là hoàn toàn chính xác.

Với accuracy 78%, cách này không an toàn.

Thay vào đó:

```text
Inventory lower bound = 12
Inventory expected = 20
Inventory upper bound = 27
```

## 27.2 Bounds

Bounds là giới hạn dưới và trên của inventory hợp lý.

Có thể được tính từ:

* Historical inventory error.
* Store/category behavior.
* Snapshot age.
* Shrinkage.
* Recent sales.
* Delivery status.
* Stocktake confidence.

## 27.3 Calibrated interval

Interval được gọi là calibrated nếu tỷ lệ coverage thực tế phù hợp với xác suất đã công bố.

Ví dụ:

```text
Hệ thống gọi là 90% interval.
```

Qua dữ liệu lịch sử, khoảng này phải chứa actual inventory gần 90% trường hợp.

Nếu chỉ chứa 60%, confidence đang bị đánh giá quá cao.

## 27.4 Kiểm tra feasibility ở lower và upper bound

Ví dụ:

```text
Demand P50 = 40
Demand P90 = 55

Inventory:
Lower = 10
Upper = 30
```

Nếu inventory thực là 10:

```text
Có nguy cơ thiếu hàng.
```

Nếu inventory thực là 30:

```text
Đặt quá nhiều có thể gây waste.
```

Recommendation phải được kiểm tra trong cả hai scenario.

## 27.5 Absolute caps

Dù model đề xuất lớn đến đâu, AI chỉ được influence trong giới hạn.

```text
Maximum AI increase = 2 case packs.
```

Điều này giới hạn blast radius khi inventory hoặc forecast sai.

## 27.6 Suppress recommendation

Suppress nghĩa là không publish recommendation.

Ví dụ suppress khi:

* Inventory interval thiếu.
* Interval quá rộng.
* Snapshot quá cũ.
* Promotion data conflict.
* Forecast quantile không calibrated.
* Recommendation không an toàn ở lower/upper scenario.

Khi suppress:

```text
MERLIN dùng incumbent min/max.
```

Đây không phải hệ thống thất bại. Đây là safe behavior.

---

# 28. Vì sao có con số khoảng $0,0004 mỗi nominal forecast?

## 28.1 Số forecast mỗi năm

Giả định:

```text
27.000.000 forecasts/night
365 nights/year
```

[
27.000.000 \times 365
= 9.855.000.000
]

Tức khoảng:

```text
9,855 tỷ forecast records/năm
```

## 28.2 Với hard ceiling $4 triệu/năm

[
$4.000.000 / 9.855.000.000
= $0,000405885...
]

Làm tròn:

[
\approx $0,0004/forecast
]

## 28.3 Với internal target $3,8 triệu

[
$3.800.000 / 9.855.000.000
= $0,00038559...
]

Tức khoảng:

```text
$0,000386/forecast
```

## 28.4 Nominal forecast nghĩa là gì?

Nominal forecast là một forecast record dự kiến theo khối lượng kế hoạch.

Nó không nhất thiết tương đương với:

* Một API call.
* Một model riêng.
* Một GPU inference.
* Một store–SKU duy nhất.

Một model invocation có thể xử lý một batch gồm hàng nghìn forecast records.

## 28.5 Đây có phải giá inference tối đa không?

Không.

Đây là cách phân bổ **toàn bộ annual architecture cost** cho tổng forecast volume.

Phải bao gồm:

* Compute.
* Feature processing.
* Model training.
* Inference.
* Optimization.
* Storage.
* Data transfer.
* Orchestration.
* Monitoring.
* Logging.
* Audit.
* Security.
* Operational support.

Nếu chỉ tính model inference $0,0003/forecast nhưng storage, network và operation thêm $0,0002 thì tổng là:

[
$0,0005
]

và vượt ceiling.

## 28.6 Hạn chế của phép tính

Đây là planning envelope, không phải hóa đơn thực tế.

Cần điều chỉnh nếu:

* Không chạy đủ 365 đêm.
* Volume thực tế thay đổi.
* Peak xảy ra nhiều ngày.
* Một số forecast bị retry.
* Có thêm training và analytics workload.
* Cost cố định không tỷ lệ thuận với số record.

---

# 29. Intent classification

## 29.1 Là gì?

Intent classification là xác định mục đích của câu hỏi người dùng.

Ví dụ nhân viên hỏi:

```text
“Sữa hạt OatPlus nằm ở đâu?”
```

Intent:

```text
PRODUCT_LOCATION
```

Câu hỏi:

```text
“Sản phẩm này có đang promotion không?”
```

Intent:

```text
PROMOTION_LOOKUP
```

Câu hỏi:

```text
“Khách bị dị ứng đậu phộng có dùng sản phẩm này được không?”
```

Intent:

```text
ALLERGEN_SAFETY_QUERY
```

## 29.2 Khi nào sử dụng?

Trong store associate copilot, trước khi chọn workflow.

```text
User question
→ Intent classification
→ Route to approved handler
```

Ví dụ:

| Intent           | Handler                       |
| ---------------- | ----------------------------- |
| Product location | Product catalogue search      |
| Promotion        | Promotion service             |
| Policy           | Knowledge retrieval           |
| Allergen         | Deterministic safety workflow |
| Order status     | Approved operational API      |
| Unknown          | Refuse hoặc ask clarification |

## 29.3 Tại sao có thể dùng LLM?

Người dùng có thể diễn đạt một ý bằng nhiều cách:

```text
“Món này ở đâu?”
“Kiếm giúp tôi sản phẩm này.”
“Nó nằm kệ mấy?”
```

LLM có thể phân loại ngôn ngữ linh hoạt.

Nhưng output phải bị giới hạn vào danh sách intent đã định nghĩa, không được tự tạo hành động mới.

---

# 30. Entity extraction

## 30.1 Là gì?

Entity extraction là lấy ra các đối tượng cụ thể từ câu hỏi.

Ví dụ:

```text
“Kiểm tra promotion của OatPlus 1L tại store 105 hôm nay.”
```

Entities:

```text
product = OatPlus 1L
store_id = 105
date = today
```

Câu:

```text
“Khách hỏi SKU 48129 có chứa peanut không?”
```

Entities:

```text
sku = 48129
allergen = peanut
```

## 30.2 Khi nào sử dụng?

Sau hoặc cùng lúc với intent classification:

```text
Question
→ Intent: PROMOTION_LOOKUP
→ Entities:
   product = OatPlus 1L
   store = 105
   date = 2026-07-24
```

Sau đó application gọi đúng nguồn dữ liệu.

## 30.3 LLM có được tự quyết định dữ liệu không?

Không.

LLM chỉ trích xuất candidate entity. Hệ thống vẫn phải:

* Resolve product name sang canonical SKU.
* Validate store ID.
* Validate date.
* Xử lý ambiguity.
* Không tự đoán SKU nếu có nhiều match.

---

# 31. Non-safety question answering có citation

## 31.1 Là gì?

Copilot trả lời câu hỏi không liên quan trực tiếp đến sức khỏe hoặc an toàn, đồng thời chỉ ra nguồn thông tin.

Ví dụ:

```text
Hỏi:
“Chính sách đổi trả sản phẩm không có hóa đơn là gì?”

Trả lời:
“Khách có thể đổi trong 14 ngày nếu cung cấp bằng chứng thanh toán thay thế…”

Nguồn:
Returns Policy v12, section 4.2
```

Citation ở đây có thể là:

* Tên tài liệu.
* Section.
* Version.
* Link.
* Last updated date.
* Knowledge record ID.

## 31.2 Khi nào sử dụng?

Phù hợp cho:

* Store policy.
* Product location.
* Promotion explanation.
* Operational procedure.
* Staff handbook.
* Non-safety product information.
* Training documentation.

## 31.3 Sử dụng ở đâu?

Trong associate copilot:

```text
Question
→ Retrieve approved documents
→ LLM constructs answer
→ Attach citations
→ User verifies source
```

Đây là một dạng RAG có kiểm soát.

## 31.4 Khi nào không sử dụng?

Không dùng generated Q&A cho:

* Allergen.
* Food safety.
* Emergency procedure nếu wording bắt buộc chính xác.
* Legal disclaimer bắt buộc.
* Medical advice.
* Price hoặc promotion chưa xác minh.
* Inventory quantity nếu không có authoritative source.

Trong các trường hợp đó nên trả exact approved content hoặc refuse.

---

# 32. Hỗ trợ ngôn ngữ cho nội dung không liên quan safety

## 32.1 Bao gồm những gì?

LLM có thể hỗ trợ:

* Dịch câu hỏi của nhân viên.
* Chuẩn hóa lỗi chính tả.
* Viết lại policy dễ hiểu hơn.
* Tóm tắt tài liệu vận hành.
* Chuyển câu trả lời sang ngôn ngữ nhân viên chọn.
* Giải thích thuật ngữ.
* Tạo câu trả lời thân thiện hơn.

Ví dụ:

```text
Nguồn policy bằng tiếng Anh
→ Nhân viên hỏi bằng tiếng Việt
→ Copilot trả lời tiếng Việt
→ Gắn citation tới policy tiếng Anh
```

## 32.2 Khi nào sử dụng?

Sử dụng khi nội dung:

* Không ảnh hưởng food safety.
* Không phải legally mandated wording.
* Không phải giá trị giao dịch chính thức.
* Có nguồn dữ liệu được phê duyệt.
* Việc diễn đạt lại không làm thay đổi quyết định quan trọng.

## 32.3 Khi nào không sử dụng dịch hoặc tóm tắt tự do?

Không dùng cho:

* Allergen warning.
* Product recall.
* Food handling temperature.
* Legal notice.
* Emergency instructions.
* Safety-critical labels.

Với các nội dung đó, phải dùng bản dịch đã được phê duyệt hoặc fixed wording theo country và locale.

---

# 33. Ví dụ end-to-end để kết nối tất cả khái niệm

## 33.1 Luồng replenishment

```text
1. MERLIN xuất nightly snapshot.

2. Snapshot validation kiểm tra:
   - Freshness
   - Completeness
   - Schema
   - Checksum
   - Residency

3. Statistical ML tạo:
   - P10 = 30
   - P50 = 42
   - P90 = 58

4. Inventory không được tin tuyệt đối:
   - Lower = 8
   - Expected = 15
   - Upper = 24

5. Deterministic optimizer áp dụng:
   - Case pack = 12
   - Shelf-life cap = 24
   - Category cap = 24
   - DC capacity available

6. Optimizer recommend 24.

7. Certified adapter kiểm tra:
   - Current run
   - Valid permit
   - Current epoch
   - Within cap
   - Valid schema

8. Adapter ghi 24 vào supported override table.

9. MERLIN tự chạy native order generation.

10. MERLIN tạo final order quantity = 12.

11. Review queue trả:
    - Recommendation ID
    - MERLIN order ID
    - Final quantity 12
    - Correlation key

12. MERLIN gửi official supplier EDI.
```

## 33.2 Nếu AI lỗi

```text
AI không hoàn thành trước 02:00
→ Không publish
→ Không late recovery
→ MERLIN dùng min/max hiện tại
→ MERLIN vẫn gửi EDI trước 04:00
```

Đây chính là ý nghĩa của:

> **AI influence là tùy chọn, nhưng MERLIN continuity là bắt buộc.**
