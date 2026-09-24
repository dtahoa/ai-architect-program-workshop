## 1. Artifact 3 dùng để làm gì?

**Technology-Selection Matrix** là tài liệu chứng minh rằng nhóm không chọn kiến trúc theo cảm tính, theo công nghệ quen thuộc hoặc vì “AI thì phải dùng công nghệ mới”.

Artifact này trả lời ba câu hỏi:

1. Nhóm đã cân nhắc những phương án nào?
2. Nhóm dùng tiêu chí gì để so sánh?
3. Vì sao phương án được chọn phù hợp hơn với constraint của NovaMart?

Nó không nhằm chứng minh một công nghệ chắc chắn thành công trong production. Nó chỉ chứng minh:

> Với các yêu cầu và rủi ro hiện tại, đây là hướng kiến trúc hợp lý nhất để tiếp tục benchmark, kiểm thử và triển khai có kiểm soát.

---

# 2. Cách tính điểm trong Selection Matrix

Mỗi quyết định có:

* Một tập các tiêu chí.
* Mỗi tiêu chí có một **weight – trọng số**.
* Mỗi phương án nhận một **score từ 1 đến 5**.
* Tổng trọng số luôn bằng 100.

Công thức:

[
Weighted\ Total =
\frac{\sum (Weight_i \times Score_i)}{100}
]

Ví dụ với phương án **External asynchronous sidecar**:

| Tiêu chí             | Weight | Score | Điểm đóng góp |
| -------------------- | -----: | ----: | ------------: |
| Legacy safety        |     30 |     5 |           150 |
| Fail-open continuity |     25 |     5 |           125 |
| Interface fit        |     20 |     5 |           100 |
| Rollback             |     15 |     5 |            75 |
| Cost/operations      |     10 |     3 |            30 |
| Tổng                 |    100 |       |           480 |

[
480 / 100 = 4.80
]

Do đó weighted total là **4.80/5**.

## Weight và score khác nhau thế nào?

### Weight: tiêu chí quan trọng đến mức nào?

Ví dụ:

* Legacy safety có weight 30.
* Cost có weight 10.

Điều đó có nghĩa là trong quyết định tích hợp với MERLIN, nhóm coi việc bảo vệ hệ thống legacy quan trọng gấp ba lần việc tối ưu chi phí vận hành.

Weight không phải là xác suất và cũng không phải mức độ chắc chắn. Nó là tỷ lệ đóng góp của tiêu chí vào quyết định.

### Score: phương án đáp ứng tiêu chí tốt đến đâu?

Score được đánh giá theo cùng một thang:

| Score | Ý nghĩa thực tế                                                   |
| ----: | ----------------------------------------------------------------- |
|     1 | Vi phạm hard constraint hoặc cần capability không được hỗ trợ     |
|     2 | Có khoảng trống lớn; giải pháp giảm thiểu tạo thêm rủi ro đáng kể |
|     3 | Khả thi, nhưng có trade-off và cần thêm evidence                  |
|     4 | Phù hợp tốt; khoảng trống có thể kiểm soát                        |
|     5 | Đáp ứng trực tiếp tiêu chí trong boundary được hỗ trợ             |

Ví dụ, sidecar có score **3 cho Cost/ops**, không phải 5, vì nó tạo ra:

* Hai hệ thống phải vận hành.
* Snapshot bị sao chép.
* Cần reconciliation.
* Cần monitoring riêng.
* Cần certified adapter.
* Cần xử lý consistency giữa sidecar và MERLIN.

Nó thắng không phải vì hoàn hảo, mà vì phần yếu của nó nằm ở chi phí và vận hành, trong khi phần mạnh nằm ở các tiêu chí quan trọng hơn: safety, continuity và rollback.

---

# 3. Vì sao weight lại có các giá trị như vậy?

Các weight không phải những con số được “tính toán khoa học” từ dữ liệu production. Đây là cách nhóm thể hiện **thứ tự ưu tiên kiến trúc** dựa trên:

* Hard constraints.
* Business impact nếu thất bại.
* Mức độ khó rollback.
* Blast radius.
* NFR.
* Yêu cầu đã nêu trong đề bài.
* Quyết định nào đã được xử lý ở artifact hoặc decision khác.

Nói cách khác:

> Weight phản ánh mức độ nguy hiểm nếu tiêu chí đó không được đáp ứng.

---

# 4. Decision 1 – MERLIN integration pattern

## Câu hỏi của quyết định này

> AI sidecar nên kết nối với MERLIN theo kiểu nào để không gây nguy hiểm cho hệ thống replenishment hiện tại?

Các phương án:

1. External asynchronous sidecar.
2. Event-driven boundary.
3. Synchronous API gateway.
4. In-process MERLIN plugin.

## Vì sao weight được phân bổ như vậy?

| Tiêu chí                | Weight | Lý do                                                                                                   |
| ----------------------- | -----: | ------------------------------------------------------------------------------------------------------- |
| Legacy safety           |     30 | MERLIN là hệ thống legacy và system of record. AI không được làm hỏng quy trình replenishment hiện tại. |
| Fail-open continuity    |     25 | Nếu AI chết lúc 03:30, MERLIN vẫn phải tiếp tục tạo order bằng logic native.                            |
| Supported-interface fit |     20 | Không được giả định MERLIN có API, plugin framework hoặc event capability chưa được xác nhận.           |
| Rollback                |     15 | Phải có khả năng tắt AI nhanh mà không sửa hoặc redeploy MERLIN.                                        |
| Cost/operability        |     10 | Quan trọng nhưng thấp hơn safety và business continuity.                                                |

Hai tiêu chí đầu chiếm:

[
30 + 25 = 55%
]

Điều này gửi một thông điệp rất rõ:

> Trong tích hợp với MERLIN, safety và continuity quan trọng hơn sự đơn giản hoặc chi phí.

## Vì sao External asynchronous sidecar đạt 4.80?

### Legacy safety = 5

Sidecar nằm ngoài MERLIN:

* Không chèn AI logic trực tiếp vào code MERLIN.
* Không thay đổi POS.
* Không thay đổi supplier EDI.
* Không thay MERLIN làm system of record.
* Có thể chỉ publish recommendation rows qua adapter được kiểm soát.

Do đó blast radius được cô lập.

### Fail-open = 5

MERLIN không chờ AI trả lời theo synchronous request.

Nếu sidecar:

* Chậm.
* Không chạy.
* Bị kill-switch.
* Không publish trước cutoff.
* Sinh output không hợp lệ.

MERLIN vẫn chạy logic native.

Đây chính là fail-open:

> AI không hoạt động thì hệ thống quay về hành vi cũ, thay vì dừng hoạt động.

### Interface fit = 5

Sidecar có thể dùng các boundary hẹp:

* Snapshot export.
* Recommendation staging rows.
* Certified adapter.
* Append-only audit.
* Current-run data.

Nó không yêu cầu MERLIN phải hỗ trợ realtime API hoặc plugin capability chưa được chứng minh.

### Rollback = 5

Có thể rollback bằng cách:

* Không publish recommendation.
* Disable adapter.
* Revoke permit.
* Tăng disable epoch.
* Cho MERLIN bỏ qua output AI.

Không cần rollback MERLIN application.

### Cost/ops = 3

Đây là điểm yếu có thật:

* Phải vận hành thêm AI platform.
* Có duplicated data.
* Có reconciliation.
* Có monitoring và alerting riêng.
* Có adapter certification.
* Có hai operational domains.

Vì vậy không nên cho score 5.

## Vì sao các lựa chọn khác thấp?

### Event-driven boundary – 2.95

Event-driven nghe hiện đại nhưng có thể cần:

* MERLIN phát event.
* Event schema ổn định.
* Consumer semantics.
* Retry và duplicate handling.
* Replay.
* Ordering guarantees.
* Dead-letter queue.
* Transactional outbox hoặc capability tương tự.

Đề bài chưa chứng minh MERLIN có các capability này.

Vì vậy **interface fit chỉ 2**.

Event-driven không hẳn sai. Nó chỉ đòi hỏi nhiều thay đổi và giả định hơn so với supported boundary hiện tại.

### Synchronous API gateway – 1.80

MERLIN phải gọi AI API trong runtime path.

Điều này tạo coupling:

[
MERLIN \rightarrow API Gateway \rightarrow AI Service
]

Nếu AI timeout hoặc API không khả dụng:

* MERLIN có thể bị chậm.
* Deadline bị ảnh hưởng.
* Retry storm có thể xảy ra.
* Fail-open khó chứng minh.
* AI trở thành dependency trong critical path.

Do đó fail-open và interface fit đều chỉ đạt 1.

### In-process MERLIN plugin – 1.30

Đây là phương án nguy hiểm nhất vì AI logic nằm ngay trong MERLIN:

* Tăng blast radius.
* Khó rollback độc lập.
* Có thể cần sửa code legacy.
* Có thể ảnh hưởng scheduling, memory hoặc transaction.
* Vi phạm nguyên tắc external sidecar.
* Không còn failure isolation.

Nó có score cost/ops = 4 vì về bề ngoài chỉ có một hệ thống để vận hành. Tuy nhiên lợi ích đó không đủ bù cho safety risk.

Đây là điểm quan trọng của weighted matrix:

> Phương án rẻ hơn hoặc ít component hơn không nhất thiết tốt hơn nếu nó làm tăng rủi ro production.

## Vì sao gọi là “shadow-assist is its initial rollout state”?

**External sidecar** là integration architecture.

**Shadow-assist** là rollout mode ban đầu.

Trong shadow mode:

* Sidecar chạy forecast và recommendation.
* Kết quả được lưu và so sánh.
* MERLIN chưa bắt buộc phải dùng recommendation.
* Không trực tiếp ảnh hưởng supplier order.
* Có thể đánh giá accuracy, latency, cost và safety.

Sau khi có evidence, hệ thống mới có thể chuyển sang assist mode có giới hạn.

## Sensitivity analysis nói gì?

Artifact thử thay đổi weight:

* Giảm legacy safety 10 điểm.
* Tăng cost/operability 10 điểm.

Kết quả:

* Sidecar: 4.60.
* Event-driven: 2.95.

Sidecar vẫn đứng đầu.

Ý nghĩa:

> Sidecar không thắng chỉ vì nhóm cố tình đặt legacy safety weight cao.

Decision 1 tương đối mạnh và ổn định.

---

# 5. Decision 2 – Forecast execution platform

## Câu hỏi của quyết định này

> Nền tảng nào có thể chạy khoảng 27 triệu forecasts mỗi đêm, chịu tải 6× và hoàn thành trước 04:00?

Các phương án:

* Managed elastic batch.
* Kubernetes batch workers.
* Managed Spark.
* Fixed on-premises compute.

## Vì sao weight như vậy?

| Tiêu chí           | Weight | Lý do                                                                                       |
| ------------------ | -----: | ------------------------------------------------------------------------------------------- |
| Deadline control   |     25 | Forecast phải hoàn tất trước cutoff 04:00. Output muộn có thể không còn an toàn để publish. |
| 6× scale           |     25 | Kiến trúc phải chịu peak volume, không chỉ average load.                                    |
| Operability        |     20 | Phải có retry, monitoring, partition tracking, idempotency và recovery.                     |
| Cost               |     15 | 27 triệu forecasts/ngày có thể tạo chi phí đáng kể.                                         |
| Nightly-legacy fit |     15 | Platform phải phù hợp với batch window hiện có của MERLIN.                                  |

Deadline và scale chiếm 50%.

Điều này phản ánh hai NFR quan trọng nhất:

* Có chạy xong đúng giờ không?
* Có scale đủ trong peak không?

Một platform rẻ nhưng không bảo đảm cutoff không phải là lựa chọn hợp lệ.

## Vì sao Managed elastic batch đạt 4.65?

### Deadline = 5

Managed batch hỗ trợ cách tổ chức phù hợp:

* Chia workload thành partitions.
* Chạy song song.
* Theo dõi progress.
* Ưu tiên partition gần deadline.
* Retry partition thay vì chạy lại toàn bộ.
* Có thể dừng expansion khi gần safety cutoff.

### 6× scale = 5

Compute có thể được mở rộng theo số partition và workload, thay vì bị giới hạn bởi fixed capacity.

### Operability = 4

Không cho 5 vì vẫn phải xử lý:

* Job orchestration.
* Quota.
* Retry policy.
* Poison partitions.
* Cost guardrails.
* Capacity availability.
* Data locality.
* Region failure.
* Monitoring.

Managed không có nghĩa là không cần vận hành.

### Cost = 4

Có lợi thế pay-per-use, nhưng chi phí có thể biến động khi:

* Scale lên 6×.
* Retry nhiều.
* Truyền dữ liệu giữa các vùng hoặc hệ thống.
* Job chạy lâu hơn benchmark.
* Partition thiết kế không tốt.

### Legacy fit = 5

MERLIN đang hoạt động theo nightly batch. Managed batch phù hợp với mô hình này hơn việc đưa realtime API vào critical path.

## Vì sao Kubernetes chỉ đạt 4.15?

Kubernetes có thể đáp ứng deadline và scale, nên hai tiêu chí này đều 5.

Tuy nhiên operability chỉ 3 vì nhóm phải tự quản lý nhiều thứ hơn:

* Cluster capacity.
* Autoscaling.
* Node pools.
* Scheduling.
* Pod failures.
* Image lifecycle.
* Security patching.
* Network policy.
* Observability.
* Cost allocation.
* Quota và resource contention.

Kubernetes không bị loại vì yếu về compute. Nó đứng sau vì operational burden lớn hơn.

## Vì sao Managed Spark chỉ 3.90?

Spark rất mạnh với distributed data processing, nhưng không mặc nhiên là lựa chọn tốt nhất cho mọi forecast workload.

Một số vấn đề:

* Startup overhead.
* Shuffle cost.
* Partition skew.
* Memory pressure.
* Tuning complexity.
* Có thể quá nặng nếu mỗi forecast là một tác vụ độc lập nhỏ.
* Operability và cost vẫn cần chứng minh bằng benchmark.

Do đó scale = 5 nhưng deadline = 4 và operability = 3.

## Vì sao fixed on-premises compute chỉ 3.10?

Nó có:

* Operability tương đối quen thuộc: 4.
* Nightly legacy fit tốt: 5.

Nhưng:

* Scale = 2.
* Deadline = 3.
* Cost = 2.

Để chịu 6× peak, phải mua capacity đủ lớn cho peak nhưng phần lớn thời gian có thể không dùng hết.

Nếu peak vượt capacity, hệ thống không thể tự mở rộng đủ nhanh.

## Sensitivity analysis nói gì?

Artifact chuyển 15 weight từ scale sang operability.

Kết quả:

* Managed batch: 4.50.
* Kubernetes: 3.85.

Managed batch vẫn đứng đầu.

Do đó lựa chọn platform không phụ thuộc hoàn toàn vào việc scale được đánh weight cao.

Tuy nhiên artifact vẫn nói:

> Provider choice remains conditional on benchmark and residency evidence.

Nghĩa là matrix chỉ chọn **loại platform**, chưa chọn vô điều kiện AWS, Azure, GCP hoặc một sản phẩm cụ thể.

Provider chỉ được chọn sau khi chứng minh:

* Throughput.
* Quota.
* Data residency.
* Cost.
* Deadline.
* Security.
* Operational readiness.

---

# 6. Decision 3 – Forecast model strategy

## Câu hỏi của quyết định này

> Loại mô hình forecast nào cân bằng tốt nhất giữa accuracy, promotion handling, uncertainty, scale và cost?

Các phương án:

1. Hybrid hierarchical statistical ML.
2. One global statistical ML model.
3. Per-store-SKU models.
4. Classical statistical baseline.

## Vì sao weight như vậy?

| Tiêu chí            | Weight | Lý do                                                                    |
| ------------------- | -----: | ------------------------------------------------------------------------ |
| WAPE potential      |     25 | Forecast phải đủ chính xác để tạo giá trị cho replenishment.             |
| Promotion behaviour |     20 | Promotion có thể làm demand thay đổi mạnh; model phải xử lý được.        |
| Uncertainty output  |     20 | Optimizer cần interval hoặc quantile, không chỉ một point forecast.      |
| Scale               |     15 | Phải xử lý hàng triệu series mỗi ngày.                                   |
| Operability         |     10 | Model phải retrain, monitor, version và rollback được.                   |
| Cost                |     10 | Chi phí quan trọng nhưng không được đánh đổi accuracy và safety quá mức. |

Ba tiêu chí về chất lượng forecast chiếm:

[
25 + 20 + 20 = 65%
]

Đây là chủ ý.

Quyết định 2 đã chọn execution platform để giải bài toán compute scale. Vì vậy ở Decision 3, trọng tâm chuyển sang:

* Forecast có đúng không?
* Có hiểu promotion không?
* Có tạo uncertainty interval không?

Scale vẫn quan trọng, nhưng không còn chiếm 25 như ở platform decision.

---

# 7. Vì sao Hybrid hierarchical statistical ML đứng đầu?

## Hybrid hierarchical statistical ML – 4.55

### WAPE = 5

Hybrid hierarchical strategy có thể kết hợp thông tin ở nhiều cấp:

* SKU.
* Store.
* Region.
* Category.
* National demand.
* Store cluster.
* Product family.

Các series ít dữ liệu có thể “borrow strength” từ cấp cao hơn hoặc từ nhóm tương tự.

Điều này hữu ích khi nhiều store-SKU có demand sparse hoặc intermittent.

### Promotion = 5

Có thể đưa promotion, price, holiday và calendar features vào model hoặc residual component.

Hybrid design cũng cho phép:

* Baseline statistical component.
* Promotion uplift component.
* Hierarchical reconciliation.
* Segment-specific behaviour.

### Uncertainty = 5

Có thể output:

* Prediction intervals.
* Quantiles.
* Variance estimate.
* Confidence bands.

Optimizer sau đó có thể dùng uncertainty thay vì xem forecast là một con số tuyệt đối.

### Scale = 4

Scale tốt nhưng phức tạp hơn một global model vì có thể có:

* Nhiều segment.
* Nhiều model family.
* Hierarchical reconciliation.
* Additional feature pipelines.
* Calibration jobs.

### Operability = 3

Đây là điểm yếu chính:

* Model governance phức tạp hơn.
* Nhiều component phải version.
* Monitoring theo segment.
* Drift detection khó hơn.
* Interval calibration phải kiểm tra.
* Có thể có fallback giữa các model families.

### Cost = 4

Chi phí vẫn hợp lý nhưng cao hơn global model vì pipeline phức tạp hơn.

---

# 8. Vì sao global model gần bằng hybrid?

Global model đạt **4.35**, chỉ thấp hơn hybrid 0.20.

Ưu điểm:

* Scale = 5.
* Operability = 5.
* Cost = 5.
* Chỉ cần quản lý ít model hơn.
* Dễ batch inference.
* Dễ deploy và rollback.
* Có thể học pattern dùng chung giữa nhiều store-SKU.

Điểm thấp hơn hybrid nằm ở:

* WAPE = 4.
* Promotion = 4.
* Uncertainty = 4.

Điều đó không có nghĩa global model không thể đạt 5. Artifact đang nói:

> Chưa có evidence đủ mạnh để khẳng định một global model sẽ xử lý sparse series, promotion và calibrated uncertainty tốt bằng hybrid strategy.

Đây cũng là lý do sensitivity test tạo ra tie.

Khi chuyển 10 weight từ WAPE sang cost:

| Phương án | Weighted total mới |
| --------- | -----------------: |
| Hybrid    |               4.45 |
| Global    |               4.45 |

Thông điệp:

> Quyết định model chưa được khóa cứng bằng architecture diagram. Nó phải được quyết định bằng backtest và replay evidence.

Đây là một điểm tốt của artifact. Nhóm không giả vờ rằng matrix đã chứng minh hybrid chắc chắn tốt hơn.

---

# 9. Vì sao per-store-SKU model có WAPE 5 nhưng tổng chỉ 3.20?

Phương án này có thể tạo model riêng cho từng store-SKU.

Nó có khả năng fit rất sát từng series, nên WAPE được cho 5.

Nhưng có khoảng 27 triệu time series hoặc forecast outputs cần xử lý. Nếu hiểu theo hướng một model riêng cho mỗi store-SKU thì sẽ tạo ra vấn đề rất lớn:

* Số model quá nhiều.
* Retraining khó quản lý.
* Versioning rất phức tạp.
* Nhiều series không đủ dữ liệu.
* Cold start.
* Monitoring không khả thi.
* Deployment artifact khổng lồ.
* Chi phí compute và storage cao.
* Rollback hàng triệu model khó thực hiện.

Do đó:

* Scale = 1.
* Operability = 1.
* Cost = 1.

Đây là ví dụ rõ nhất về giá trị của weighted matrix:

> Không được chọn model chỉ vì accuracy tiềm năng cao nhất.

Một mô hình có thể tốt trong notebook nhưng không thể vận hành ở production scale.

---

# 10. Vì sao classical baseline đạt 3.50, cao hơn per-store-SKU?

Classical baseline có:

* Scale = 5.
* Operability = 5.
* Cost = 5.

Ví dụ có thể là các phương pháp thống kê truyền thống, đơn giản và dễ tái tạo.

Nhưng:

* WAPE potential = 3.
* Promotion behaviour = 2.
* Uncertainty = 3.

Nó dễ vận hành nhưng có thể không đủ khả năng xử lý promotion và pattern phức tạp.

Dù vậy, classical baseline vẫn rất quan trọng vì nó cung cấp benchmark:

> Model mới chỉ có giá trị nếu thực sự tốt hơn baseline đơn giản sau khi tính cả accuracy, latency, cost và operability.

Baseline không nhất thiết là phương án production cuối cùng, nhưng phải tồn tại để tránh việc nhóm tuyên bố AI thành công mà không có đối chứng.

---

# 11. Tại sao không dùng LLM?

Artifact viết:

> No LLM is considered because numeric forecasts require reproducibility, scale, and approximately $0.0004 blended cost per forecast.

Thông điệp không chỉ là “LLM đắt”.

LLM không phù hợp với core forecast vì:

* Numeric forecasting cần output ổn định.
* Cần backtest có thể tái tạo.
* Cần chạy với volume rất lớn.
* Cần uncertainty được định nghĩa rõ.
* Cần kiểm soát model version.
* Không nên có hallucination.
* Không nên dùng natural-language model cho một bài toán numeric prediction nếu statistical ML phù hợp hơn.

LLM có thể được dùng cho associate copilot ở một capability riêng, nhưng không nên được đưa vào core replenishment forecast path.

---

# 12. Ý nghĩa của Accepted trade-off

Mỗi lựa chọn đều có đoạn:

> Accepted trade-off

Đây là phần rất quan trọng.

Một architecture decision tốt không nói:

> Phương án được chọn tốt nhất và không có nhược điểm.

Nó phải nói:

> Chúng tôi biết rõ mình đang chấp nhận nhược điểm gì để đổi lấy lợi ích gì.

Ví dụ sidecar chấp nhận:

* Duplicated snapshots.
* Reconciliation.
* Certified adapter work.
* Two-system operations.

Đổi lại nhận được:

* Independent failure.
* Independent rollback.
* MERLIN continuity.
* Lower legacy blast radius.

Managed batch chấp nhận:

* Provider dependence.
* Cost variability.
* Data-transfer controls.

Đổi lại nhận được:

* Elastic scale.
* Deadline control.
* Partitioned execution.

Hybrid model chấp nhận:

* Governance phức tạp hơn.
* Nhiều model components hơn.

Đổi lại nhận được:

* Better sparse-series sharing.
* Promotion handling.
* Calibrated uncertainty.

Thông điệp là:

> Nhóm hiểu chi phí của quyết định, không chỉ hiểu lợi ích.

---

# 13. Sensitivity analysis dùng để làm gì?

Weight trong selection matrix có yếu tố judgement. Trainer có thể hỏi:

> Nếu nhóm thay weight thì kết quả có thay đổi không?

Sensitivity analysis trả lời câu hỏi đó.

Nó cố ý thay đổi một số weight quan trọng và tính lại kết quả.

Có ba trường hợp:

### Decision 1: kết quả ổn định

Sidecar vẫn thắng rõ.

Điều này cho thấy lựa chọn integration pattern tương đối chắc chắn.

### Decision 2: kết quả ổn định

Managed batch vẫn thắng.

Nhưng vẫn cần benchmark và residency evidence trước khi chọn provider.

### Decision 3: kết quả nhạy cảm

Hybrid và global model hòa nhau khi cost được tăng trọng số.

Điều này cho thấy model selection chưa thể quyết định chỉ bằng kiến trúc.

Phải có replay evidence về:

* WAPE.
* Promotion periods.
* Interval calibration.
* Throughput.
* Cost.
* Sparse series.
* Cold-start behaviour.

Sensitivity analysis không làm matrix yếu đi. Ngược lại, nó cho thấy nhóm biết quyết định nào đã chắc và quyết định nào còn phụ thuộc evidence.

---

# 14. Thông điệp chính của Artifact 3

Artifact 3 truyền tải năm thông điệp.

## Thông điệp 1: Architecture được chọn từ constraint, không phải từ sở thích công nghệ

Nhóm không nói:

* Dùng Kubernetes vì Kubernetes hiện đại.
* Dùng Spark vì dữ liệu lớn.
* Dùng LLM vì đây là bài AI.
* Cắm plugin vào MERLIN vì ít component.

Nhóm đánh giá phương án dựa trên NovaMart constraints.

## Thông điệp 2: Safety được ưu tiên hơn sự đơn giản

Đặc biệt trong MERLIN integration:

* Legacy safety.
* Fail-open.
* Rollback.
* Supported interfaces.

được ưu tiên hơn cost và số lượng component.

## Thông điệp 3: Scale phải gắn với deadline

Không chỉ cần xử lý 27 triệu forecasts.

Phải xử lý:

* Trong nightly window.
* Dưới 6× peak.
* Trước 04:00.
* Với retry và partial failure.
* Không publish output muộn hoặc không đầy đủ.

## Thông điệp 4: Model chưa được chọn chỉ vì score cao nhất

Hybrid đang dẫn đầu, nhưng global model rất gần và có thể thắng nếu replay evidence chứng minh:

* Accuracy tương đương.
* Promotion handling đủ tốt.
* Interval calibration đạt yêu cầu.
* Chi phí thấp hơn.
* Operability đơn giản hơn.

## Thông điệp 5: Điểm số không phải production approval

Đây là ý nghĩa của phần:

> Scores select a direction; they do not prove production readiness.

Matrix chỉ chọn hướng để tiếp tục.

Production influence vẫn cần vượt qua:

* NFR gates.
* Adapter certification.
* Safety tests.
* Kill-switch test.
* Cost evidence.
* Data residency approval.
* Operational readiness.
* Rollout gates.
* Shadow-mode evidence.
* Reconciliation evidence.

---

# 15. Decision boundary quan trọng thế nào?

Không có phần Decision boundary, người đọc có thể hiểu sai:

> Sidecar được 4.80, vậy sidecar đã được approved để chạy production.

Điều đó không đúng.

Điểm 4.80 chỉ có nghĩa:

> Trong các phương án đang xét, sidecar là direction phù hợp nhất.

Nó không chứng minh rằng:

* Adapter đã hoạt động đúng.
* Forecast hoàn thành trước 04:00.
* 6× benchmark đã pass.
* Kill-switch thực sự hoạt động.
* Chi phí chính xác.
* Residency đã được phê duyệt.
* MERLIN fail-open đã được thử nghiệm.
* Recommendation an toàn với inventory accuracy 78%.

Artifact 3 trả lời câu hỏi **“chọn hướng nào?”**

Artifacts 2 và 6 trả lời câu hỏi **“khi nào hướng đó được phép ảnh hưởng production?”**

---

# 16. Một điểm nên cải thiện trong matrix

Có một điểm phương pháp luận nên được nói rõ hơn:

> Hard constraints không nên chỉ được đưa vào weighted score; chúng nên là knockout gates.

Hiện tại score 1 được định nghĩa là:

> Violates a hard constraint.

Nhưng về mặt toán học, một option có score 1 ở hard constraint vẫn có thể bù bằng các score 5 ở tiêu chí khác.

Ví dụ, nếu một phương án vi phạm POS immutability, nó không nên được chọn dù tổng weighted score có cao.

Nên bổ sung quy tắc:

> Any option scoring 1 against a declared hard constraint is disqualified before weighted ranking.

Quy trình sẽ trở thành:

1. Kiểm tra hard constraints.
2. Loại phương án vi phạm.
3. Chấm weighted score cho các phương án còn lại.
4. Chạy sensitivity analysis.
5. Ghi accepted trade-offs.
6. Xác định evidence còn thiếu.

Như vậy matrix sẽ chặt chẽ hơn và khó bị trainer phản biện rằng “hard constraint đang bị biến thành soft preference”.

---

# 17. Cách trình bày Artifact 3 trong khoảng một phút

Bạn có thể nói:

> Artifact 3 chứng minh rằng chúng tôi không chọn công nghệ theo cảm tính. Với mỗi quyết định, chúng tôi xác định các tiêu chí dựa trên hard constraints và business risk, phân bổ tổng trọng số bằng 100, sau đó chấm từng phương án theo cùng một thang điểm từ 1 đến 5.
>
> Với MERLIN integration, safety và fail-open chiếm 55% trọng số, vì AI không được làm gián đoạn replenishment hiện tại. External asynchronous sidecar đạt 4.80 và vẫn đứng đầu khi chúng tôi thay đổi trọng số, nên đây là một quyết định tương đối ổn định.
>
> Với forecast execution, deadline 04:00 và khả năng chịu tải 6× chiếm 50%, vì vậy managed elastic batch được chọn, nhưng provider cụ thể vẫn cần benchmark và residency approval.
>
> Với model strategy, hybrid đạt 4.55 và global model đạt 4.35. Sensitivity analysis tạo ra kết quả hòa, nên model cuối cùng sẽ được quyết định bằng backtest về WAPE, promotion, uncertainty, throughput và cost.
>
> Quan trọng nhất, matrix chỉ chọn architectural direction; nó không thay thế production gates trong Artifacts 2 và 6.

## Tóm lại

Thông điệp ngắn gọn nhất của Artifact 3 là:

> **Chúng tôi ưu tiên safety, deadline và evidence hơn sự mới mẻ của công nghệ. Các con số giúp quyết định có thể kiểm tra và tranh luận lại, nhưng không được dùng để giả vờ rằng hệ thống đã production-ready.**
