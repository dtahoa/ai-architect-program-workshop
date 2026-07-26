# Transcript thuyết trình 8 phút — Phiên bản tiếng Việt dễ trình bày

> **Cách sử dụng:** Không cần đọc đúng từng chữ. Các phần in đậm là ý cần nhấn giọng. Các thuật ngữ tiếng Anh được giữ lại để thống nhất với nội dung trên slide.

## Slide 1 — Tiêu đề (0:00–0:45)

Xin chào mọi người.

Thông điệp chính trong thiết kế của nhóm chúng tôi rất rõ ràng: **bảo vệ MERLIN, bổ sung năng lực AI, nhưng tuyệt đối không để AI làm gián đoạn việc đặt hàng**.

Với Variant B, chúng tôi đặt một **AI sidecar bất đồng bộ** bên ngoài MERLIN. Có thể hiểu đơn giản: AI chỉ đóng vai trò tư vấn. AI tạo dự báo và đề xuất số lượng bổ sung hàng, nhưng **không trực tiếp tạo đơn hàng**.

MERLIN vẫn là hệ thống duy nhất tạo đơn đặt hàng chính thức và gửi EDI cho nhà cung cấp. Khi AI chạy trễ, thiếu dữ liệu, gặp lỗi hoặc bị tắt, AI sẽ không ghi kết quả. MERLIN vẫn tiếp tục chạy theo cơ chế hiện tại, vì vậy hoạt động cung ứng không bị dừng.

## Slide 2 — Nội dung trình bày (0:45–1:15)

Toàn bộ bài trình bày xoay quanh một quyết định kiến trúc quan trọng: **AI phải nằm ngoài critical path**, tức là nằm ngoài luồng bắt buộc để MERLIN tạo và gửi đơn hàng.

Tôi sẽ trình bày lần lượt năm nội dung: nguyên tắc vận hành; những phần được phép và không được phép thay đổi trong hệ thống cũ; vị trí và các thành phần của AI sidecar; quy trình xử lý ban đêm cùng cơ chế fail-open; cuối cùng là các chỉ số, rủi ro và lộ trình triển khai.

## Slide 3 — Nguyên tắc vận hành (1:15–2:05)

Nguyên tắc của chúng tôi là: **MERLIN luôn giữ quyền kiểm soát, còn AI phải có khả năng thất bại một cách an toàn**.

AI không thay thế MERLIN và cũng không tự quyết định đặt hàng. AI chỉ tạo dự báo nhu cầu và khuyến nghị bổ sung hàng trong một phạm vi đã được giới hạn.

Sau đó, một **certified adapter**, tức là bộ kết nối đã được kiểm soát, chỉ ghi những dữ liệu hợp lệ vào vùng mà MERLIN được phép đọc. MERLIN đọc các khuyến nghị này, chạy engine hiện tại và tự tạo đơn hàng chính thức.

Ví dụ, nếu AI gặp sự cố lúc 03:30, chúng tôi sẽ không cố chạy lại khẩn cấp, vì việc đó có thể làm trễ thời hạn gửi EDI lúc 04:00. Trong trường hợp này, MERLIN bỏ qua AI và tiếp tục sử dụng logic min/max hiện có. Đây chính là **fail-open**: AI thất bại, nhưng hoạt động kinh doanh vẫn tiếp tục.

## Slide 4 — Đánh giá hệ thống legacy (2:05–2:55)

Giải pháp retrofit này bắt đầu từ các ràng buộc của MERLIN, chứ không bắt đầu từ việc chọn công nghệ AI.

Chúng tôi có thể xây dựng AI sidecar bên ngoài, adapter, hệ thống giám sát, audit và cơ chế tắt AI. Tuy nhiên, chúng tôi **không thay đổi** MERLIN engine, chức năng tạo đơn hàng, POS, định dạng EDI hay thời hạn 04:00.

Chúng tôi cũng không truy cập các bảng dữ liệu chưa được tài liệu hóa và không can thiệp vào scheduler nội bộ của MERLIN.

Vì vậy, sidecar chỉ sử dụng ba điểm tích hợp đã được hỗ trợ: một là snapshot ban đêm ở chế độ chỉ đọc; hai là override table chứa các dòng dữ liệu bị giới hạn trước khi MERLIN tạo đơn; và ba là review queue sau khi tạo đơn để đối chiếu và audit. **Nhóm không tự tạo thêm một interface thứ tư chưa được hệ thống hỗ trợ.**

## Slide 5 — Kiến trúc hệ thống C4 (2:55–3:45)

Ở góc nhìn tổng thể, AI sidecar nằm bên ngoài luồng đặt hàng chính.

Sidecar nhận snapshot chỉ đọc, kiểm tra dữ liệu, tạo dự báo và đưa ra khuyến nghị. Sau đó, nó chỉ được phép ghi những dòng dữ liệu nằm trong phạm vi đã quy định. MERLIN vẫn hoạt động độc lập và vẫn chịu trách nhiệm tạo đơn hàng cũng như gửi EDI.

Kiến trúc này có các ranh giới rất rõ: **AI không gọi trực tiếp POS, không gửi EDI và không liên hệ nhà cung cấp**.

Associate Copilot cũng không được tham gia vào việc tạo đơn. Copilot chỉ đọc thông tin và chỉ trả lời dựa trên nguồn đã được phê duyệt. Điều này đặc biệt quan trọng với nội dung liên quan đến an toàn thực phẩm, vì một câu trả lời sai có thể tạo ra trách nhiệm pháp lý và rủi ro vận hành.

## Slide 6 — Các thành phần bên trong sidecar (3:45–4:35)

Slide này cho thấy cách chúng tôi cô lập rủi ro bên trong AI sidecar.

Đầu tiên, **snapshot validation** kiểm tra cấu trúc dữ liệu, mức độ đầy đủ và yêu cầu lưu trữ dữ liệu đúng khu vực.

Tiếp theo, mô hình dự báo tạo nhu cầu kỳ vọng, các mức dự báo cao và thấp, khoảng tin cậy và ảnh hưởng của chương trình khuyến mãi. Tuy nhiên, mô hình **chỉ dự báo, không tạo đơn hàng**.

Sau đó, **deterministic optimizer** áp dụng các quy tắc kinh doanh bắt buộc như quy cách đóng thùng, hạn sử dụng, năng lực trung tâm phân phối, mức tồn kho có thể dao động và giới hạn theo ngành hàng.

Cuối cùng, certified adapter là đường ghi duy nhất. Adapter chỉ chấp nhận dữ liệu đúng schema, thuộc lần chạy hiện tại, nằm trong giới hạn cho phép, có đầy đủ dấu vết audit và chưa bị kill switch vô hiệu hóa.

## Slide 7 — Batch ban đêm và fail-open (4:35–5:35)

Quy trình ban đêm được thiết kế ngược từ thời hạn gửi EDI lúc 04:00.

Batch bắt đầu lúc 22:00. Các bước kiểm tra dữ liệu, dự báo và tối ưu phải hoàn thành trước những mốc an toàn cuối cùng.

Đến 01:30, hệ thống không bắt đầu thêm retry mới, nhằm tránh tình trạng nhiều tác vụ lỗi cùng chạy lại và làm quá tải hệ thống.

Đến 01:45, **complete-set gate** kiểm tra xem tất cả các phần dữ liệu bắt buộc đã hoàn tất hay chưa. Chỉ cần thiếu một phần, sidecar sẽ không ghi bất kỳ khuyến nghị nào của lần chạy đó. Cách xử lý này tránh việc MERLIN đọc một bộ dữ liệu chỉ hoàn thành một phần.

Đến 02:00, đường ghi của AI được đóng hoàn toàn: không ghi trễ và không chạy lại khẩn cấp.

Tuy nhiên, mốc 02:00 chỉ phù hợp nếu số liệu thực tế cho thấy MERLIN cần tối đa khoảng 90 phút ở mức P99 để hoàn thành đơn hàng. Nếu MERLIN cần lâu hơn, chúng tôi phải đóng đường ghi AI sớm hơn để vẫn bảo vệ deadline 04:00.

## Slide 8 — Chỉ số kỹ thuật và mục tiêu kinh doanh (5:35–6:35)

Các con số trên slide là **mục tiêu thiết kế và điều kiện để được phát hành**, chưa phải kết quả benchmark thực tế.

Mỗi đêm, hệ thống phải tạo khoảng 27 triệu dự báo. Trong số đó, khoảng 7,04 triệu cặp mặt hàng và cửa hàng thực sự cần đi qua bước tối ưu.

Ở kịch bản tải cao gấp sáu lần, hệ thống phải chịu được 162 triệu lượt dự báo và hơn 42 triệu lượt tối ưu. Tổng thời gian xử lý mục tiêu là 198,7 phút, nằm trong cửa sổ 240 phút và còn khoảng 41 phút dự phòng.

Về kinh doanh, mục tiêu là giảm tỷ lệ hết hàng từ 7,2% xuống 3%; giảm lãng phí hàng tươi sống từ 4,8% xuống 3%; và giảm sai số WAPE từ 41% xuống tối đa 25%.

Một điểm rất quan trọng là độ chính xác tồn kho hiện chỉ khoảng 78%. Vì vậy, hệ thống không coi con số tồn kho là hoàn toàn chính xác. Nó phải tính theo một khoảng ước lượng, và chỉ đưa ra khuyến nghị khi kết quả vẫn an toàn ở cả giới hạn thấp lẫn giới hạn cao. Nếu mức độ không chắc chắn quá lớn, khuyến nghị sẽ bị loại bỏ.

## Slide 9 — Quyết định kiến trúc và quản lý rủi ro (6:35–7:25)

Mỗi thành phần trong kiến trúc đều phải giải quyết một rủi ro cụ thể.

Sidecar bất đồng bộ bảo vệ tính sẵn sàng của MERLIN. Deterministic optimizer giúp khuyến nghị có thể giải thích và kiểm tra lại. Certified adapter giới hạn chặt dữ liệu được ghi vào MERLIN và bảo đảm mọi thay đổi đều có version, đúng schema và có audit.

Năm rủi ro lớn nhất là: xử lý không kịp deadline; dữ liệu tồn kho không chính xác; không đủ năng lực ở tải đỉnh; trách nhiệm liên quan đến an toàn thực phẩm; và kill switch không hoạt động đúng khi có sự cố.

Các biện pháp chính gồm đóng gate sớm, loại bỏ khuyến nghị có độ bất định cao, benchmark ở tải gấp sáu lần, giữ Copilot ở chế độ chỉ đọc và sử dụng kill switch có thể thu hồi quyền ghi độc lập. Quyền ghi của sidecar cũng chỉ có hiệu lực trong thời gian ngắn, nên khi bị vô hiệu hóa, nó không thể tiếp tục dùng một quyền cũ để ghi dữ liệu.

## Slide 10 — Lộ trình và kết luận (7:25–8:00)

Giải pháp được triển khai theo năm bước: **discovery, replay, shadow, pilot và expansion**.

Đầu tiên là hiểu dữ liệu và quy trình hiện tại. Sau đó, chúng tôi chạy lại dữ liệu lịch sử để kiểm chứng. Ở giai đoạn shadow, AI chạy song song nhưng chưa ảnh hưởng đến MERLIN. Khi kết quả đủ an toàn, hệ thống mới được thử nghiệm với phạm vi nhỏ, rồi từng bước mở rộng.

Tiêu chí thành công không chỉ là mô hình dự báo chính xác. Điều kiện quan trọng nhất là **MERLIN vẫn phải hoạt động bình thường khi AI không tồn tại hoặc bị tắt hoàn toàn**.

Vì vậy, đây không phải là kiến trúc lấy AI làm trung tâm. Đây là một cách hiện đại hóa MERLIN có kiểm soát: AI mang lại giá trị, nhưng MERLIN vẫn giữ quyền quyết định, các ranh giới được giới hạn rõ ràng và quy trình đặt hàng không bao giờ phụ thuộc vào AI.
