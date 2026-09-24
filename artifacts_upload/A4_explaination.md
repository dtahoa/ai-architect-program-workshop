# 1. Artifact 4 là gì?

Artifact 4 là một **ADR — Architecture Decision Record**.

ADR không dùng để so sánh nhiều phương án bằng điểm số như Artifact 3. ADR dùng để ghi lại một quyết định kiến trúc đã được chọn, bao gồm:

* Bối cảnh tại thời điểm quyết định.
* Constraint nào buộc nhóm phải chọn như vậy.
* Những phương án đã xem xét.
* Phương án cuối cùng.
* Hệ quả tích cực và tiêu cực.
* Cách rollback.
* Khi nào quyết định cần được xem xét lại.

Nói đơn giản:

> Artifact 3 trả lời “phương án nào phù hợp nhất?”.
> Artifact 4 trả lời “chúng ta chính thức quyết định làm gì, chấp nhận điều gì và vận hành quyết định đó như thế nào?”.

---

# 2. Tên ADR có ý nghĩa gì?

## `ADR-001: Fail-Open AI Sidecar`

Tên này chứa hai quyết định quan trọng.

### AI Sidecar

AI được triển khai thành một hệ thống nằm bên ngoài MERLIN.

Nó không:

* Chạy bên trong MERLIN.
* Thay thế MERLIN.
* Gửi supplier order.
* Thay đổi POS.
* Thay đổi EDI.
* Trở thành system of record.

Nó chỉ tạo forecast và recommendation, sau đó đưa kết quả vào một boundary được kiểm soát.

### Fail-open

Fail-open trong bài toán này có nghĩa:

> Khi AI không hoạt động, MERLIN vẫn tiếp tục hoạt động bằng logic hiện tại.

Ví dụ sidecar:

* Không chạy.
* Chạy chậm.
* Sinh output thiếu.
* Model bị lỗi.
* Adapter bị khóa.
* Bị kill-switch.
* Không hoàn tất trước deadline.

Thì MERLIN vẫn phải tiếp tục quy trình native của nó.

Fail-open ở đây không có nghĩa là “bỏ qua tất cả kiểm tra an toàn”. Nó có nghĩa:

> Hệ thống kinh doanh cốt lõi không phụ thuộc vào sự sống còn của AI.

---

# 3. Vì sao status là “Accepted for replay/shadow”?

ADR ghi:

> Accepted for replay/shadow; production influence remains gated.

Điều này có nghĩa quyết định kiến trúc đã được chấp nhận để:

* Chạy replay với dữ liệu lịch sử.
* Chạy shadow với dữ liệu thật.
* Đo latency.
* Đo accuracy.
* Đo cost.
* Kiểm tra adapter.
* Kiểm tra audit.
* Kiểm tra fail-open.
* Kiểm tra kill-switch.

Nhưng chưa được chấp nhận để AI trực tiếp ảnh hưởng supplier order.

## Vì sao phải giới hạn như vậy?

Bởi vì matrix hoặc ADR không thể tự chứng minh rằng:

* 27 triệu forecasts hoàn tất đúng giờ.
* Tải 6× vẫn đáp ứng deadline.
* Adapter không khóa MERLIN table.
* Inventory accuracy 78% vẫn cho recommendation an toàn.
* Forecast đủ chính xác.
* Kill-switch thật sự có tác dụng trong 60 giây.
* Rollback hoàn tất trong 5 phút.
* Cost đúng như ước tính.
* Food-safety constraints đầy đủ.

Vì vậy ADR đang nói:

> Chúng tôi chấp nhận kiến trúc này để tạo evidence, chưa chấp nhận nó để thay đổi production orders.

Đây là một thông điệp quản trị rủi ro rất quan trọng.

---

# 4. Giải thích phần Context

ADR viết:

> NovaMart needs forecasting without replacing MERLIN.

Đây là yêu cầu nền tảng:

* NovaMart muốn thêm AI forecast.
* Nhưng không muốn thay thế replenishment engine hiện tại.
* MERLIN vẫn là hệ thống chính thức.

## Vì sao MERLIN phải giữ authority?

Vì MERLIN hiện đang kiểm soát:

* Logic tạo order.
* Order lifecycle.
* Review.
* Supplier EDI.
* Các quy tắc nghiệp vụ hiện có.
* Quy trình vận hành đã được tổ chức sử dụng.

Nếu sidecar trở thành order authority thì đây không còn là AI retrofit nữa. Nó sẽ trở thành:

* Core-system replacement.
* ERP/replenishment replatform.
* Một chương trình chuyển đổi lớn hơn rất nhiều.

ADR đóng boundary rõ:

> AI đưa ra recommendation; MERLIN vẫn quyết định và tạo order chính thức.

---

## “MERLIN remains order authority” nghĩa là gì?

Điều này có nghĩa chỉ MERLIN mới được:

1. Tạo official order.
2. Gán order ID.
3. Áp dụng các native controls.
4. Cho phép người dùng review.
5. Gửi supplier EDI.
6. Ghi nhận order lifecycle chính thức.

AI sidecar có thể đề xuất:

* SKU nào cần đặt thêm.
* Suggested quantity.
* Confidence hoặc forecast interval.
* Reason code.
* Model version.
* Correlation ID.

Nhưng sidecar không tự tạo supplier order.

---

## Vì sao engine, POS và EDI không được thay đổi?

### MERLIN engine

Nếu sửa core engine để gọi AI hoặc chạy AI logic:

* Blast radius tăng.
* Rollback khó hơn.
* Testing scope lớn hơn.
* AI failure có thể ảnh hưởng order generation.

### POS

POS là nguồn dữ liệu và hệ thống bán hàng quan trọng. AI forecast không cần phải can thiệp vào transaction path của POS.

### Supplier EDI

EDI là boundary gửi order ra ngoài doanh nghiệp.

Nếu AI được quyền gửi EDI trực tiếp:

* AI trở thành order authority.
* Sai quantity có thể tạo cam kết thương mại.
* Rollback sau khi supplier đã nhận order rất khó.
* Audit và liability tăng mạnh.

Do đó AI phải dừng trước EDI boundary.

---

## Vì sao 04:00 cutoff không được thay đổi?

04:00 là business deadline.

Forecast không chỉ cần “chạy xong trong ngày”. Nó phải hoàn tất đủ sớm để:

* Validate output.
* Publish recommendation.
* Cho MERLIN xử lý.
* Cho review hoặc downstream processing.
* Không ảnh hưởng supplier ordering window.

Do đó deadline không phải NFR tùy chọn. Nó là hard operational constraint.

---

## Vì sao inventory accuracy 78% được đưa vào Context?

Vì forecast tốt không thể tự động sửa dữ liệu inventory sai.

Ví dụ:

* Hệ thống nghĩ còn 100 đơn vị.
* Thực tế chỉ còn 60.
* Forecast demand là 80.

Nếu optimizer dùng inventory 100, nó có thể đề xuất đặt thiếu.

Ngược lại:

* Hệ thống nghĩ còn 20.
* Thực tế còn 70.

AI có thể đề xuất over-order.

Do đó AI không được đối xử với inventory như sự thật tuyệt đối. Nó cần:

* Inventory confidence.
* Safety caps.
* Exception handling.
* Bounded recommendation.
* Human review với nhóm nhạy cảm.
* Fallback khi snapshot chất lượng thấp.

Việc đưa con số 78% vào Context nhấn mạnh rằng:

> Rủi ro không chỉ nằm ở model; rủi ro còn nằm ở dữ liệu đầu vào.

---

## Vì sao chỉ dùng các extension hiện có?

ADR ghi rằng MERLIN chỉ hỗ trợ:

* Nightly batch.
* Override table.
* Post-generation review queue.

Điều này quyết định boundary kiến trúc.

Nhóm không được tự giả định rằng MERLIN có:

* Realtime forecast API.
* Event bus.
* Plugin framework.
* Transactional event hooks.
* Pre-order baseline comparison API.
* Synchronous decision API.

ADR chọn sidecar vì nó phù hợp với những interface được xác nhận, không phải interface mong muốn.

---

# 5. Giải thích phần Decision drivers

Decision drivers là các yếu tố quan trọng nhất dẫn đến quyết định.

## 5.1 Ordering continuity

Order phải tiếp tục được tạo ngay cả khi AI lỗi.

Đây là driver quan trọng nhất của fail-open.

Nếu AI làm dừng replenishment:

* Store có thể thiếu hàng.
* Supplier không nhận được order.
* Delivery bị gián đoạn.
* Business impact lớn hơn lợi ích AI.

---

## 5.2 Supported interfaces

Giải pháp chỉ được dùng các interface mà MERLIN thực sự hỗ trợ.

Điều này tránh “architecture by assumption”, ví dụ:

* Giả sử có event API.
* Giả sử có plugin point.
* Giả sử override table có transaction semantics nhất định.
* Giả sử MERLIN có baseline trước publication.

ADR chủ động giới hạn thiết kế theo evidence hiện có.

---

## 5.3 Failure and release isolation

AI phải có thể:

* Fail độc lập.
* Deploy độc lập.
* Rollback độc lập.
* Scale độc lập.
* Bị disable độc lập.

Nếu release AI yêu cầu release MERLIN, thì isolation chưa đạt.

---

## 5.4 27M normal and 6× scale

Architecture phải chịu:

* Normal volume khoảng 27 triệu forecasts.
* Conservative peak khoảng 6 lần.

Điều này ảnh hưởng:

* Batch partitioning.
* Compute capacity.
* Retry budget.
* Storage throughput.
* Adapter write rate.
* Deadline controls.

Sidecar cho phép scale AI workload mà không scale trực tiếp MERLIN core.

---

## 5.5 Deterministic quantities

ML nên dự báo demand, nhưng final recommended quantity cần qua deterministic constraints.

Ví dụ:

[
Recommended\ quantity =
f(Forecast,\ Inventory,\ Case\ pack,\ Shelf\ life,\ Capacity,\ Caps)
]

Không nên để model tùy ý tạo order quantity.

Deterministic optimizer có thể áp dụng:

* Minimum order quantity.
* Case-pack rounding.
* Shelf-life constraint.
* Store capacity.
* Distribution-centre capacity.
* Category cap.
* Fresh-food cap.
* Inventory confidence limits.
* Maximum deviation from MERLIN baseline, nếu baseline được hỗ trợ.

Thông điệp là:

> ML dự báo; rule-based optimizer giới hạn hành động.

---

## 5.6 Shadow

Shadow mode cho phép chạy hệ thống thật mà chưa cho nó ảnh hưởng order.

Nó giúp kiểm tra:

* Forecast quality.
* Recommendation stability.
* Batch timing.
* Adapter behavior.
* Data completeness.
* Cost.
* Model drift.
* Difference với MERLIN native outcome.

Shadow là rollout mechanism, không phải kiến trúc độc lập.

---

## 5.7 Audit

Mỗi recommendation cần có thể truy vết:

* Input snapshot nào.
* Model version nào.
* Ruleset nào.
* Job run nào.
* Forecast nào.
* Constraint nào được áp dụng.
* Recommendation ban đầu.
* Row nào được publish.
* MERLIN có consume hay không.
* Order ID nào liên quan.

Nếu không có audit, nhóm không thể:

* Giải thích sai quantity.
* Điều tra incident.
* Chứng minh compliance.
* So sánh replay.
* Rollback chính xác.

---

## 5.8 Rollback

Rollback không chỉ là redeploy version cũ.

Trong AI sidecar, rollback phải bao gồm:

* Ngăn recommendation mới.
* Thu hồi quyền ghi.
* Vô hiệu hóa permit.
* Xử lý row chưa consume.
* Không sửa row đã được MERLIN consume một cách tùy tiện.
* Xác nhận MERLIN đã quay về native behavior.

ADR biến rollback thành một phần của architecture, không phải một hoạt động sau incident.

---

# 6. Giải thích các Options considered

## 6.1 External sidecar — Selected

Lý do được chọn:

* Tách failure khỏi MERLIN.
* Không cần thay đổi core.
* Có thể chạy batch riêng.
* Scale độc lập.
* Rollback độc lập.
* Phù hợp với override table hoặc adapter boundary.
* Hỗ trợ shadow mode.

Đây là phương án cân bằng tốt nhất giữa giá trị AI và rủi ro legacy.

---

## 6.2 Event-driven boundary — Rejected

Không phải event-driven là kiến trúc xấu.

Nó bị từ chối vì MERLIN chưa có supported event contract.

Để event-driven hợp lệ, cần xác nhận:

* MERLIN phát event nào.
* Event schema.
* Delivery guarantee.
* Ordering guarantee.
* Retry semantics.
* Idempotency key.
* Replay behavior.
* Dead-letter handling.
* Transaction boundary.
* Versioning.

Nếu những thứ đó chưa có, lựa chọn event-driven sẽ tạo ra một unsupported assumption.

---

## 6.3 Synchronous API gateway — Rejected

Trong phương án này MERLIN phải gọi AI API khi xử lý order.

Ví dụ:

[
MERLIN \rightarrow AI\ API \rightarrow Forecast/Recommendation
]

Rủi ro:

* AI timeout làm chậm MERLIN.
* Network lỗi ảnh hưởng order path.
* Retry storm.
* API latency làm mất cutoff.
* AI trở thành runtime dependency.
* Fail-open khó hơn.
* Release AI có thể ảnh hưởng MERLIN.

Vì thế ADR nói synchronous API làm AI thành **order-path dependency**.

---

## 6.4 In-process plugin — Rejected

AI hoặc integration logic được cài trực tiếp vào MERLIN process.

Rủi ro:

* Thay đổi untested core.
* Memory hoặc CPU contention.
* Coupling release.
* Coupling rollback.
* Shared process failure.
* Khó giới hạn blast radius.
* Có thể ảnh hưởng transaction logic.

Phương án này trái với mục tiêu retrofit an toàn.

---

## 6.5 Shadow-only — Retained for rollout; not an architecture

Đây là một phân biệt rất quan trọng.

Shadow mode chỉ mô tả:

> Output AI có ảnh hưởng production hay không.

Nó không mô tả:

* AI nằm ở đâu.
* Kết nối bằng gì.
* Data đi như thế nào.
* Adapter nào được dùng.
* Failure boundary ở đâu.

Vì vậy shadow-only không phải architecture option.

Một sidecar có thể chạy ở:

* Replay mode.
* Shadow mode.
* Assist mode.
* Bounded influence mode.

Architecture vẫn là sidecar; shadow chỉ là trạng thái rollout.

---

# 7. Giải thích phần Decision

ADR quyết định:

> Deploy an asynchronous sidecar.

“Asynchronous” có nghĩa MERLIN không phải gửi request và chờ AI trả lời trong critical path.

Luồng tổng quát:

1. Tạo immutable snapshot.
2. Sidecar đọc snapshot.
3. Chạy forecast.
4. Áp deterministic constraints.
5. Validate completeness.
6. Adapter ghi bounded recommendation rows.
7. MERLIN đọc row qua supported mechanism.
8. MERLIN tạo hoặc review order.
9. MERLIN gửi supplier EDI.

---

## 7.1 Vì sao phải đọc approved immutable snapshots?

### Approved

Snapshot phải vượt qua validation trước khi AI dùng:

* Đúng business date.
* Đúng store set.
* Đúng SKU set.
* Không thiếu critical fields.
* Schema đúng version.
* Manifest hợp lệ.

### Immutable

Sau khi run bắt đầu, snapshot không được thay đổi.

Nếu input thay đổi giữa chừng:

* Các partition có thể dùng dữ liệu khác nhau.
* Replay không tái tạo được.
* Audit mất giá trị.
* Output không còn nhất quán.

Immutable snapshot cho phép xác định chính xác:

> Recommendation này được tạo từ bộ dữ liệu nào.

---

## 7.2 Vì sao dùng statistical forecast ML?

Bài toán là numeric forecasting quy mô lớn.

Statistical ML phù hợp vì:

* Có thể backtest.
* Có thể tái tạo.
* Có thể đo WAPE.
* Có thể output quantiles hoặc intervals.
* Có thể batch inference.
* Dễ versioning hơn generative output.
* Không cần LLM trong core forecast path.

---

## 7.3 Vì sao deterministic quantity constraints?

Forecast không đồng nghĩa với order quantity.

Ví dụ forecast demand là 137, nhưng:

* Case pack là 12.
* Shelf capacity chỉ còn 48.
* Hàng có shelf life ngắn.
* Distribution centre bị giới hạn.
* Inventory confidence thấp.

Final quantity phải được tính bằng rules có thể audit.

Ví dụ:

[
Raw\ need = Forecast - Available\ inventory
]

Sau đó:

[
Bounded\ need =
\min(Raw\ need,\ Store\ cap,\ Category\ cap,\ DC\ cap)
]

Tiếp theo làm tròn theo case pack.

Điều này giúp quantity:

* Giải thích được.
* Tái tạo được.
* Có giới hạn.
* Không phụ thuộc hoàn toàn vào model.

---

## 7.4 Vì sao chỉ được ghi qua certified adapter?

Sidecar không nên kết nối trực tiếp và tự do vào MERLIN database.

Certified adapter chịu trách nhiệm:

* Validate schema.
* Validate run ID.
* Validate time window.
* Validate current disable epoch.
* Validate permit.
* Enforce quantity bounds.
* Enforce row count.
* Reject stale rows.
* Đảm bảo idempotency.
* Ghi audit.
* Không sửa dữ liệu ngoài phạm vi.

Adapter là safety boundary giữa AI và MERLIN.

---

## 7.5 “Bounded rows” nghĩa là gì?

Sidecar không được ghi dữ liệu tùy ý.

Mỗi row phải bị giới hạn về:

* Table.
* Column.
* SKU/store scope.
* Quantity range.
* Run ID.
* Business date.
* Validity period.
* Model version.
* Correlation ID.
* Current-run status.

Ví dụ sidecar không được:

* Update official order.
* Update historical run.
* Xóa MERLIN data.
* Ghi trực tiếp EDI status.
* Sửa inventory ledger.

---

## 7.6 Vì sao MERLIN vẫn generate, review và send every order?

Đây là dòng quan trọng nhất trong ADR.

Nó giữ ba quyền trong MERLIN:

### Generate

Official order được tạo bởi MERLIN.

### Review

Các workflow kiểm tra hoặc human review vẫn nằm trong MERLIN.

### Send

Chỉ MERLIN gửi EDI tới supplier.

Như vậy AI chỉ ảnh hưởng recommendation, không sở hữu order lifecycle.

---

# 8. “No pre-publication MERLIN baseline is assumed” nghĩa là gì?

Dòng này chủ động tránh một unsupported assumption.

Một thiết kế dễ mắc lỗi là giả định:

> Trước khi AI publish recommendation, MERLIN đã tạo sẵn native baseline order để sidecar so sánh.

Nhưng interface hiện có có thể chỉ trả về thông tin sau khi MERLIN đã generate order, ví dụ:

* Order ID.
* Final quantity.
* Correlation data.

Do đó ADR nói:

> Chúng tôi không thiết kế safety control dựa trên một MERLIN baseline trước publication nếu chưa chứng minh interface đó tồn tại.

Đây là cách tránh fake control.

Ví dụ nếu một rule nói:

> AI không được thay đổi quá 20% so với MERLIN baseline.

Nhưng baseline chỉ có sau khi AI đã publish, thì control này không thể dùng trước publication.

Trong trường hợp đó cần dùng các boundary khác:

* Absolute quantity cap.
* Category cap.
* Historical-demand cap.
* Inventory-confidence cap.
* Shadow comparison sau generation.
* Review queue.

---

# 9. Vì sao có timeline 01:30, 01:45 và 02:00?

ADR đặt ba mốc:

* 01:30: stop retries.
* 01:45: require completeness.
* 02:00: close writes.
* 04:00: business cutoff.

Đây là một **deadline ladder**.

## 9.1 Stop retries at 01:30

Sau 01:30 không tiếp tục retry forecast jobs.

Lý do:

* Retry muộn có thể tạo output quá sát cutoff.
* Output hoàn thành từng phần có thể không nhất quán.
* Retry storm có thể chiếm compute và adapter capacity.
* Cần chuyển từ “cố hoàn thành” sang “bảo vệ MERLIN”.

Nếu chưa hoàn tất, run có thể bị đánh dấu unusable.

---

## 9.2 Require completeness at 01:45

Đến 01:45 hệ thống phải biết run có đầy đủ hay không.

Completeness có thể bao gồm:

* Đủ expected stores.
* Đủ expected SKU partitions.
* Manifest khớp.
* Không thiếu mandatory categories.
* Không có poison partition chưa xử lý.
* Output count trong khoảng hợp lệ.

Nếu không đầy đủ, không nên publish một phần mà MERLIN có thể hiểu nhầm là toàn bộ.

---

## 9.3 Close writes at 02:00

Sau 02:00, adapter không chấp nhận AI writes.

Mục tiêu:

* Tạo khoảng đệm trước 04:00.
* Tránh late-arriving output.
* Cho MERLIN đủ thời gian xử lý.
* Cho monitoring và operator phản ứng.
* Tránh sidecar ghi khi MERLIN đã bắt đầu order processing.
* Giảm race condition.

Khoảng thời gian:

[
04{:}00 - 02{:}00 = 2\ giờ
]

Hai giờ này là safety and processing buffer.

Tuy nhiên cần lưu ý:

> Các mốc 01:30, 01:45 và 02:00 chỉ hợp lý nếu được chứng minh bằng measured MERLIN timing.

Nếu chưa có số liệu production, đây là proposed control, chưa phải evidence.

---

# 10. Positive consequences

## 10.1 MERLIN continues without AI

Đây là lợi ích cốt lõi.

AI outage không đồng nghĩa business outage.

---

## 10.2 POS và EDI được cô lập

AI không nằm trong:

* POS transaction path.
* Supplier communication path.

Điều này giới hạn blast radius và liability.

---

## 10.3 AI scales independently

Forecast workload có thể mở rộng mà không phải tăng capacity hoặc sửa code MERLIN core.

---

## 10.4 Shadow tests the real path

Shadow không chỉ test model offline.

Nó test:

* Real snapshot.
* Real orchestration.
* Real adapter.
* Real timings.
* Real data quality.
* Real reconciliation.
* Real cost.

Nhưng chưa cho output AI ảnh hưởng order.

---

## 10.5 Removal needs no MERLIN-code rollback

Nếu loại bỏ sidecar:

* Disable adapter.
* Revoke permits.
* Remove AI jobs.
* MERLIN tiếp tục native processing.

Không cần rollback MERLIN code nếu boundary được thiết kế đúng.

---

# 11. Negative consequences

ADR không che giấu chi phí của quyết định.

## 11.1 Duplicated snapshots

Data được xuất khỏi MERLIN hoặc source system sang AI environment.

Rủi ro:

* Tốn storage.
* Data synchronization.
* Data residency.
* Snapshot lifecycle.
* Access control.
* Duplicate sensitive data.

---

## 11.2 Reconciliation

Cần đối chiếu:

* AI recommendation.
* Adapter publication.
* MERLIN-consumed rows.
* Final order.
* EDI outcome.

Nếu không reconciliation, nhóm không biết recommendation nào thực sự được sử dụng.

---

## 11.3 SAP certification

Nếu MERLIN hoặc adapter liên quan SAP boundary, integration phải được kiểm tra và phê duyệt.

Không thể giả định một database write là an toàn chỉ vì technically possible.

---

## 11.4 Two-system monitoring/on-call

Có thêm một hệ thống nghĩa là:

* Thêm dashboard.
* Thêm alerts.
* Thêm runbook.
* Thêm on-call ownership.
* Thêm incident scenarios.
* Thêm cost.

Sidecar giảm coupling nhưng tăng operational surface.

---

## 11.5 Intentionally bounded value

AI không được tự động hóa toàn bộ.

Ví dụ Release 1 có thể chỉ:

* Forecast.
* Recommendation.
* Shadow comparison.
* Bounded assist.

Không trực tiếp:

* Gửi EDI.
* Dynamic price write.
* Thay POS.
* Tự động xử lý toàn bộ exception.

Đây là trade-off có chủ ý:

> Giá trị ban đầu thấp hơn để đổi lấy safety và khả năng chứng minh.

---

# 12. Giải thích phần Risks

## 12.1 Unmeasured MERLIN timing

Nếu chưa biết P95/P99 runtime của MERLIN, nhóm không thể chắc chắn rằng đóng write lúc 02:00 là đủ.

Cần đo:

* MERLIN start time.
* Read time.
* Order-generation duration.
* Review queue latency.
* EDI preparation window.
* P99 under peak.

---

## 12.2 Adapter locking/cleanup

Adapter có thể gây:

* Database locks.
* Table contention.
* Long transactions.
* Deadlocks.
* Cleanup xóa nhầm row.
* Duplicate writes.
* Race với MERLIN consumption.

Vì vậy adapter cần load test và concurrency test.

---

## 12.3 Inaccurate inventory

78% inventory accuracy có thể dẫn đến:

* Over-order.
* Under-order.
* Waste.
* Stockout.
* False model blame.

Cần data-quality gates và bounded quantities.

---

## 12.4 Peak model failure

Model có thể hoạt động tốt ở normal load nhưng thất bại ở 6×:

* Deadline miss.
* Compute quota.
* Partition skew.
* Memory pressure.
* Retry storm.
* Cost spike.

Scale gate phải được chứng minh bằng representative benchmark.

---

## 12.5 Unsafe safety content

Cụm này có thể diễn đạt chưa rõ. Ý hợp lý nhất là:

> Recommendation có thể vi phạm các ràng buộc an toàn nghiệp vụ, đặc biệt với fresh food hoặc regulated categories.

Ví dụ:

* Shelf-life không đủ.
* Overstock hàng dễ hỏng.
* Cold-chain capacity không đủ.
* Dị ứng hoặc compliance category bị xử lý sai.
* Category cap chưa đầy đủ.

Nên đổi wording thành:

> Missing or incorrect food-safety and regulated-category constraints may block production influence.

Câu này rõ hơn “unsafe safety content”.

---

# 13. Giải thích phần Rollback

Rollback được thiết kế nhiều lớp.

## 13.1 Set a new disable epoch

Disable epoch là một version number hoặc generation number của trạng thái cho phép.

Ví dụ:

* Permit hiện tại được cấp cho epoch 17.
* Incident xảy ra.
* Control plane tăng epoch thành 18.
* Permit thuộc epoch 17 lập tức trở thành stale.

Ưu điểm:

* Không cần tìm và thu hồi từng token.
* Các worker cũ không thể tiếp tục ghi.
* Tránh stale process tiếp tục hoạt động.

---

## 13.2 Stop permits

Sidecar hoặc adapter cần short-lived permit để ghi.

Khi kill-switch được bật:

* Không cấp permit mới.
* Permit cũ nhanh chóng hết hạn.
* Adapter reject write thiếu valid permit.

---

## 13.3 Reject writes within 60 seconds

Đây là kill-switch service-level objective.

Sau khi disable:

[
T_{\text{reject write}} \leq 60\ giây
]

Điều này cần được test thực tế:

* Worker đang chạy.
* Worker retry.
* Network partition.
* Cached permit.
* Clock skew.
* Multiple regions hoặc instances.

Nếu chỉ có UI toggle mà worker vẫn ghi trong 10 phút, kill-switch không thực tế.

---

## 13.4 Independently revoke SAP access

Kill-switch trong AI application chưa đủ.

Nếu adapter hoặc credentials vẫn có quyền ghi, cần một đường revoke độc lập:

* IAM deny.
* Secret revoke.
* Database permission revoke.
* Network deny.
* SAP technical-user disable.

“Independently” có nghĩa đường revoke không phụ thuộc vào chính sidecar đang bị lỗi.

---

## 13.5 Clean only exact unconsumed AI rows

Cleanup chỉ được xóa những row đáp ứng đầy đủ điều kiện, ví dụ:

* Đúng run ID.
* Đúng AI source.
* Đúng business date.
* Chưa được MERLIN consume.
* Chưa tạo official order.
* Có correlation ID chính xác.

Không được dùng kiểu cleanup rộng:

```text
DELETE all recommendations created today
```

vì có thể xóa dữ liệu native hoặc dữ liệu đã được sử dụng.

---

## 13.6 Consumed rows use native pre-EDI handling

Nếu MERLIN đã consume recommendation và tạo official order, không nên xóa database row để giả vờ như chưa xảy ra.

Lúc đó phải dùng quy trình native trước EDI, ví dụ:

* Review.
* Hold.
* Cancel.
* Adjust quantity.
* Suppress EDI.
* Operator approval.

Sau khi order đã vào MERLIN lifecycle, MERLIN phải xử lý nó.

---

## 13.7 Verify safe within five minutes

Không chỉ thực hiện kill-switch; phải xác nhận trạng thái an toàn.

Trong vòng năm phút cần kiểm tra:

* Không còn write mới.
* Adapter bị chặn.
* Permit bị vô hiệu.
* Không còn unconsumed unsafe rows.
* MERLIN vẫn chạy.
* EDI không bị gửi sai.
* Alert được ghi nhận.
* Audit có disable event.

Tuy nhiên giống mốc 60 giây, năm phút phải là testable SLO, không phải con số tuyên bố.

---

## 13.8 Re-entry is shadow-only

Sau incident, không được bật lại production influence ngay.

Phải:

1. Xác định root cause.
2. Sửa lỗi.
3. Chứng minh corrective action.
4. Chạy replay.
5. Chạy shadow.
6. Quan sát ổn định.
7. Xin approval để influence trở lại.

Điều này tránh lặp lại incident ngay sau recovery.

---

# 14. Giải thích phần “When to revisit”

ADR không phải quyết định vĩnh viễn.

Nó phải được xem xét lại khi assumptions thay đổi.

## 14.1 Interfaces change

Nếu MERLIN sau này có:

* Supported event contract.
* Stable decision API.
* Certified plugin framework.
* Native forecast extension.

thì sidecar pattern có thể cần được đánh giá lại.

---

## 14.2 P99 cannot preserve 30 minutes

Ý nghĩa là runtime đo được không để lại tối thiểu 30 phút safety margin.

Ví dụ:

[
04{:}00 - P99\ completion < 30\ phút
]

thì architecture hoặc deadline ladder không đủ an toàn.

Nhóm có thể phải:

* Bắt đầu sớm hơn.
* Giảm scope.
* Tăng compute.
* Thay partitioning.
* Bỏ late partitions.
* Không cho production influence.

Cần lưu ý ADR hiện đóng writes lúc 02:00, tức buffer danh nghĩa là hai giờ. Mốc “preserve 30 minutes” nhiều khả năng là minimum safety boundary ở downstream flow. ADR nên mô tả rõ 30 phút được tính từ điểm nào đến điểm nào.

---

## 14.3 Adapter safety cannot be certified

Nếu adapter không thể chứng minh:

* Không lock MERLIN.
* Không ghi ngoài scope.
* Idempotent.
* Kill-switch hiệu quả.
* Cleanup an toàn.

thì sidecar không được production influence.

---

## 14.4 Repeated 6×, quality hoặc cost gates fail

Nếu nhiều lần benchmark vẫn không đạt:

* Scale.
* Forecast quality.
* Deadline.
* Cost.

thì cần xem lại model hoặc platform, thậm chí dừng approach.

ADR không biến sidecar thành quyết định bất chấp evidence.

---

## 14.5 Incident disproves fail-open

Ví dụ một incident cho thấy:

* AI failure làm MERLIN chậm.
* Adapter lock làm order generation fail.
* Disable không ngăn write.
* MERLIN thực tế phụ thuộc AI row.
* Cleanup phá native flow.

Khi đó giả định “fail-open” bị bác bỏ và ADR phải được xem xét lại.

---

## 14.6 MERLIN authority remains fixed

Dù các công nghệ khác thay đổi, một constraint vẫn không đổi:

> MERLIN tiếp tục là order authority.

Điều này ngăn ADR bị mở rộng dần thành việc AI tự tạo hoặc gửi order mà không có quyết định kiến trúc mới.

---

# 15. Sự khác nhau giữa Artifact 3 và Artifact 4

| Artifact 3 — Selection Matrix             | Artifact 4 — ADR                        |
| ----------------------------------------- | --------------------------------------- |
| So sánh các lựa chọn                      | Ghi lại quyết định cuối cùng            |
| Dùng weight và score                      | Dùng context, decision và consequences  |
| Trả lời “tại sao phương án này đứng đầu?” | Trả lời “chính xác chúng ta sẽ làm gì?” |
| Có sensitivity analysis                   | Có rollback và revisit conditions       |
| Chọn direction                            | Đóng decision boundary                  |
| Không mô tả đầy đủ vận hành               | Mô tả cách triển khai và dừng hệ thống  |

Hai artifact hỗ trợ nhau:

* Matrix chứng minh lựa chọn không tùy tiện.
* ADR biến lựa chọn đó thành quyết định có trách nhiệm.

---

# 16. Thông điệp chính của Artifact 4

Artifact 4 truyền tải sáu thông điệp.

## Thông điệp 1: AI không thay thế MERLIN

AI chỉ là sidecar tạo forecast và recommendation.

MERLIN vẫn:

* Tạo order.
* Review order.
* Gửi order.
* Sở hữu order lifecycle.

---

## Thông điệp 2: AI phải fail mà business vẫn tiếp tục

Kiến trúc được thiết kế sao cho:

> AI failure không được trở thành replenishment failure.

Đây là ý nghĩa thực sự của fail-open.

---

## Thông điệp 3: Chỉ dùng interface đã được hỗ trợ

Không thiết kế dựa trên:

* Event contract chưa tồn tại.
* API chưa được xác nhận.
* Plugin point chưa được chứng minh.
* MERLIN baseline chưa có.

---

## Thông điệp 4: ML không được tự do quyết định quantity

ML dự báo demand.

Deterministic optimizer và adapter giới hạn quantity.

MERLIN giữ quyền quyết định cuối cùng.

---

## Thông điệp 5: Rollback là một capability, không phải một lời hứa

ADR chỉ rõ:

* Disable epoch.
* Permit revocation.
* Write rejection.
* Access revocation.
* Exact cleanup.
* Native handling cho consumed rows.
* Shadow-only re-entry.

Điều này cho thấy rollback đã được thiết kế từ đầu.

---

## Thông điệp 6: Accepted không có nghĩa là production approved

ADR chỉ accepted cho:

* Replay.
* Shadow.
* Evidence collection.

Production influence vẫn bị chặn cho đến khi vượt qua các gate.

---

# 17. Điểm mạnh của ADR này

ADR có một số điểm mạnh rõ ràng:

* MERLIN authority được ghi rõ.
* Fail-open được đặt thành design principle.
* Các phương án bị loại có lý do cụ thể.
* Shadow được phân biệt với architecture.
* Positive và negative consequences đều được ghi.
* Rollback đủ chi tiết để kiểm thử.
* Có điều kiện revisit.
* Không giả định pre-publication MERLIN baseline.
* Production influence vẫn gated.

---

# 18. Những điểm nên làm rõ hoặc cải thiện

## 18.1 Các mốc thời gian cần evidence

01:30, 01:45 và 02:00 rất cụ thể, nhưng cần chỉ rõ nguồn:

* Derived từ measured runtime?
* Capacity model?
* Operational agreement?
* Hay chỉ là proposed safety windows?

Nên thêm:

> These times are provisional until validated against measured MERLIN P95/P99 batch timings.

---

## 18.2 “30 minutes” cần định nghĩa rõ

Cần nói rõ:

* 30 phút trước 04:00?
* 30 phút trước MERLIN generation?
* 30 phút trước EDI handoff?
* 30 phút sau AI write closure?

Nếu không, trainer có thể hỏi khoảng đệm này được tính từ boundary nào.

---

## 18.3 “Certified adapter” cần tiêu chí certification

Nên ghi adapter phải pass:

* Concurrency test.
* Lock-contention test.
* Idempotency test.
* Stale-run rejection.
* Permission-boundary test.
* Kill-switch test.
* Cleanup test.
* Shared-load test với MERLIN.
* Recovery test.

---

## 18.4 “Unsafe safety content” nên đổi wording

Nên dùng:

> Missing or incorrect food-safety, shelf-life, and regulated-category constraints.

Cụm này cụ thể và dễ bảo vệ hơn.

---

## 18.5 Fail-open cần có measurable invariant

Có thể bổ sung invariant:

> Absence, lateness, incompleteness, or rejection of AI output must not prevent MERLIN from executing its native order-generation path.

Như vậy fail-open trở thành điều có thể test.

---

# 19. Cách present Artifact 4 trong khoảng một phút

Bạn có thể trình bày:

> Artifact 4 ghi lại quyết định kiến trúc chính thức của nhóm: sử dụng một AI sidecar bất đồng bộ và fail-open. AI đọc immutable snapshots, tạo statistical forecasts, sau đó áp dụng deterministic quantity constraints. Nó chỉ được ghi các recommendation row có giới hạn thông qua certified adapter.
>
> MERLIN vẫn là order authority: MERLIN tạo, review và gửi mọi supplier order. Chúng tôi loại event-driven vì MERLIN chưa có supported event contract, loại synchronous API vì nó đưa AI vào critical order path, và loại in-process plugin vì nó thay đổi untested core. Shadow được giữ lại như rollout mode, không phải một kiến trúc riêng.
>
> ADR cũng định nghĩa deadline ladder, rollback bằng disable epoch, short-lived permits và independent access revocation. Quan trọng nhất, quyết định này chỉ được accepted cho replay và shadow. Production influence vẫn bị chặn cho đến khi timing, scale, quality, adapter safety, cost và fail-open được chứng minh.

---

# 20. Tóm tắt ngắn nhất

Thông điệp cốt lõi của Artifact 4 là:

> **AI được phép hỗ trợ MERLIN, nhưng không được trở thành điều kiện để MERLIN hoạt động. MERLIN giữ toàn bộ quyền tạo và gửi order; AI phải có thể bị vô hiệu hóa nhanh, rollback độc lập và chỉ được ảnh hưởng production sau khi có đủ evidence.**

---
**30 phút trong ADR được tính từ thời điểm MERLIN hoàn tất luồng xử lý P99 đến deadline EDI lúc 04:00.**

Cụ thể:

```text
02:00  AI đóng hoàn toàn quyền ghi
       ↓
       MERLIN bắt đầu/tiếp tục:
       order generation → native review → EDI
       ↓
03:30  MERLIN phải hoàn tất theo P99
       ↓
       30 phút safety reserve
       ↓
04:00  EDI deadline bất biến
```

Công thức:

[
02{:}00 + 90\ phút\ MERLIN\ P99 = 03{:}30
]

[
04{:}00 - 03{:}30 = 30\ phút\ safety\ reserve
]

Tài liệu batch flow ghi rõ rằng production influence yêu cầu **measured MERLIN P99 ≤ 90 phút để còn ít nhất 30 phút trước 04:00**; nếu không đạt, mốc đóng AI phải được đẩy sớm hơn. 

## Boundary chính xác

Boundary bắt đầu của phép tính là:

> **Thời điểm MERLIN bắt đầu sử dụng các override rows cuối cùng sau khi AI write window đóng.**

Trong thiết kế hiện tại, boundary này đang được giả định là **02:00**.

Boundary kết thúc là:

> **Deadline supplier EDI bất biến lúc 04:00.**

Tuy nhiên, 30 phút không phải toàn bộ khoảng từ 02:00 đến 04:00. Khoảng hai giờ này được chia thành:

| Khoảng thời gian | Mục đích                                                                             |
| ---------------- | ------------------------------------------------------------------------------------ |
| 02:00–03:30      | Tối đa 90 phút cho MERLIN order generation, native review và EDI processing theo P99 |
| 03:30–04:00      | 30 phút safety reserve trước EDI deadline                                            |

Presentation cũng xác nhận mốc 02:00 chỉ hợp lệ nếu **MERLIN P99 order generation không quá 90 phút**; nếu lâu hơn, AI write-close phải dịch về sớm hơn. 

## 30 phút dùng để làm gì?

30 phút là **operational safety reserve**, không phải thời gian dành cho AI retry hoặc emergency rerun.

Nó bảo vệ trước các biến động cuối luồng như:

* MERLIN chậm hơn bình thường nhưng vẫn chưa vượt P99.
* Native review hoặc queue processing có độ trễ.
* EDI preparation hoặc transmission có jitter.
* Database hoặc SAP processing có temporary contention.
* Clock skew, scheduling delay hoặc final operational checks.

Nếu AI hồi phục lúc 03:30 thì không được chạy lại. Output chỉ được giữ để chẩn đoán; MERLIN tiếp tục current run. 

## Ví dụ khi P99 thay đổi

### Trường hợp hiện tại

```text
EDI deadline             = 04:00
Safety reserve           = 30 phút
Measured MERLIN P99      = 90 phút
Latest safe AI close     = 04:00 - 30 phút - 90 phút
                         = 02:00
```

### Nếu đo được MERLIN P99 là 110 phút

```text
Latest safe AI close = 04:00 - 30 phút - 110 phút
                     = 01:40
```

Khi đó mốc 02:00 không còn hợp lệ; AI write-close phải chuyển thành **01:40 hoặc sớm hơn**.

### Nếu P99 là 75 phút

```text
Latest calculated close = 04:00 - 30 phút - 75 phút
                        = 02:15
```

Về toán học có thể đóng lúc 02:15, nhưng kiến trúc không nhất thiết phải nới gate. Nhóm có thể giữ 02:00 để có thêm 15 phút bảo vệ.

## Vì sao câu trong ADR gây mơ hồ?

Câu hiện tại là:

> Revisit if interfaces change, measured P99 cannot preserve 30 minutes...

Nó chưa nói rõ:

* P99 của bước nào.
* Tính bắt đầu từ thời điểm nào.
* 30 phút còn lại trước deadline nào.

Dựa trên batch-flow và presentation, cách hiểu đầy đủ phải là:

> **Measured P99 của MERLIN từ khi AI override writes đóng đến khi hoàn tất order generation, native review và EDI phải cho phép giữ lại tối thiểu 30 phút trước deadline 04:00.**

Có thể sửa ADR thành:

> Revisit if measured MERLIN P99 from AI write-close through order generation, native review, and EDI exceeds 90 minutes, or otherwise leaves less than 30 minutes of safety reserve before the fixed 04:00 EDI deadline.

Như vậy trainer sẽ hiểu ngay:

[
AI\ write\ close + MERLIN\ P99 + 30\text{-minute reserve} \leq 04{:}00
]

**Kết luận:** 30 phút là khoảng dự phòng nằm **sau khi MERLIN hoàn thành theo P99 và trước deadline EDI 04:00**. Với mốc AI close 02:00, ADR đang giả định MERLIN P99 tối đa 90 phút, hoàn thành vào 03:30 và để lại 30 phút dự phòng.

