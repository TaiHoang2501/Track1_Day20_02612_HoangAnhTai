# Track1_Day20_02612_HoangAnhTai

- **Họ tên:** Hoàng Anh Tài
- **MHV:** 2A202602612

---

## 00 — Phạm vi

1. **Dự án:** BookingBot AI Agent · Trợ lý Đặt lịch Xem nhà & Giữ căn Vinhomes
2. **Persona:** Người mua nhà (Homebuyer) có nhu cầu tìm mua căn hộ ở thực hoặc đầu tư tại đại đô thị Vinhomes; bận rộn, coi trọng thời gian, sợ tin ảo và ngại bị môi giới telesale làm phiền.
3. **Core job:** Tìm kiếm đúng căn hộ Vinhomes phù hợp tiêu chí và chốt được lịch hẹn đi xem nhà thực tế một cách nhanh chóng, minh bạch.

### Bảng 4 Khái Niệm Cốt Lõi (Core Framework)

| Khái niệm | Câu hỏi định hướng | Áp dụng cho BookingBot AI Agent |
| :--- | :--- | :--- |
| **Core job** | User đang cố hoàn thành việc gì? | **Tìm đúng căn hộ Vinhomes ưng ý và đặt được lịch hẹn đi xem nhà thực tế với chủ nhà/CĐT.** |
| **Core action** | User làm gì trong sản phẩm để tiến tới giá trị? | **Chat tìm căn theo tiêu chí $\rightarrow$ Chọn căn phù hợp $\rightarrow$ Chọn khung giờ rảnh và bấm gửi yêu cầu đặt lịch hẹn (Booking Request).** |
| **Core value** | User nhận được lợi ích gì? | **Không tốn thời gian trao đổi qua lại; chắc chắn có lịch hẹn xem căn thật và được tạm giữ căn (Viewing Hold chống trùng lịch) trong khung giờ xem.** |
| **Core value event** | Sự kiện nào chứng minh value đã xảy ra? | **`booking_confirmed`** *(Chủ nhà duyệt lịch & Viewing Hold được kích hoạt thành công trên hệ thống)* |

---

## 01 — Core Action

### 1. Core Action Card

| Thành phần | Câu hỏi định hướng | Câu trả lời của bạn (BookingBot AI Agent) |
| :--- | :--- | :--- |
| **Target user** | Ai thực hiện hành vi? | **Người mua nhà (Homebuyer / Buyer)** có nhu cầu tìm mua căn hộ Vinhomes để ở thực hoặc đầu tư; muốn quy trình nhanh gọn, không qua trung gian phiền hà. |
| **Core job** | Họ đang cố hoàn thành việc gì? | **Tìm đúng căn hộ Vinhomes ưng ý và chốt được lịch hẹn đi xem nhà thực tế** với chủ nhà/CĐT một cách nhanh chóng, minh bạch. |
| **Core action** | Hành vi được chọn là gì? | **Gửi yêu cầu đặt lịch hẹn xem nhà (Submit Booking Request)** cho một căn hộ cụ thể vào một khung giờ xem khả dụng. |
| **Object** | Hành vi tác động lên đối tượng nào? | Căn hộ mục tiêu (`Property`) và Khung giờ xem nhà (`AvailabilitySlot`), tạo thành bản ghi Lịch hẹn (`Booking`) gắn với Hồ sơ thương lượng (`Deal`). |
| **Preconditions** | Điều gì phải có trước khi hành vi xảy ra? | 1. Căn hộ đã được xác thực thông tin/sổ đỏ và đang ở trạng thái sẵn sàng giao dịch (`AVAILABLE`).<br>2. Khung giờ xem (`AvailabilitySlot`) còn trống, khớp giữa thời gian rảnh của Người mua và Chủ nhà.<br>3. Người mua đã xác thực SĐT qua OTP (đối với khách vãng lai/guest) để tạo phiên đặt lịch hợp lệ. |
| **Completion rule** | Khi nào action được xem là hoàn tất? | Khi hệ thống tạo thành công bản ghi `Booking` với trạng thái `REQUESTED`, sinh mã Deal (`VH-YYYY-XXXXX`), tạm khóa giữ slot và gửi thông báo yêu cầu phê duyệt tới Chủ nhà (`Owner`). |
| **Core value** | Người dùng nhận được lợi ích gì? | Tiết kiệm tối đa thời gian trao đổi; chắc chắn có lịch hẹn xem căn thật; được bảo vệ giữ chỗ (Viewing Hold chống trùng lịch) ngay khi chủ nhà xác nhận. |
| **Evidence of value** | Dấu hiệu nào chứng minh value đã xảy ra? | Lịch hẹn chuyển sang trạng thái **`CONFIRMED`**, cơ chế khóa căn `Viewing Hold` kích hoạt thành công (ngăn chặn tuyệt đối double-booking), và Người mua nhận thông báo xác nhận thành công kèm hướng dẫn đón tiếp. |
| **Candidate event** | Event nào có thể dùng để tracking? | • **Action Event (Hành vi):** `booking_requested` (Payload: `deal_id`, `property_id`, `slot_id`, `buyer_id`).<br>• **Value Event (Giá trị):** `booking_confirmed` (Payload: `booking_id`, `deal_id`, `hold_id`, `confirmed_at`). |

---

### 2. Tự kiểm 5 tiêu chí (Core Action Evaluation)

| Tiêu chí | Câu hỏi kiểm tra | Đánh giá (Đạt / Trượt) | Giải thích lý do cụ thể cho BookingBot |
| :--- | :--- | :---: | :--- |
| **1. Gần core value** | Hành vi xảy ra là user đã tiến gần rõ rệt tới value chưa? | **ĐẠT** | Ngay khi gửi yêu cầu đặt lịch (`booking_requested`), Người mua đã hoàn tất việc chọn căn và chọn slot giờ; chỉ cần Chủ nhà bấm Duyệt là value kích hoạt ngay (Lịch hẹn chốt & Viewing Hold khóa căn). Đây là bước quyết định đưa người mua từ giai đoạn tìm kiếm sang nhận giá trị thực tế. |
| **2. Có thể lặp lại** | Hành vi có xuất hiện lại khi nhu cầu quay lại không? | **ĐẠT** | Người mua nhà hiếm khi chỉ xem 1 căn duy nhất. Họ sẽ lặp lại hành vi này từ 3–5 lần (đặt lịch xem các căn 2PN, 3PN khác nhau cùng phân khu hoặc so sánh giữa các tòa S1, S2) trong suốt quá trình khảo sát trước khi đưa ra quyết định mua. |
| **3. Có thể quan sát** | Bạn biết chính xác khi nào nó hoàn tất không? | **ĐẠT** | Đo lường chính xác và tường minh ở tầng kỹ thuật/hệ thống: Hoàn tất khi bản ghi `Booking` được insert vào cơ sở dữ liệu với `id` duy nhất, mã `deal_id`, trạng thái `REQUESTED` và hệ thống bắn event analytics `booking_requested`. |
| **4. Có ý nghĩa** | Hành vi tăng có thật sự nghĩa là sản phẩm tốt hơn không? | **ĐẠT** | Số lượng `booking_requested` tăng chứng minh AI Agent bóc tách nhu cầu chuẩn xác, dữ liệu căn hộ uy tín, loại bỏ được lực cản đắn đo của khách (thay vì khách chỉ vào chat vài câu vu vơ rồi rời bỏ - drop-off). |
| **5. Có thể tác động** | Team có thể cải thiện khả năng nó xảy ra không? | **ĐẠT** | Product Team hoàn toàn có thể tối ưu: Nâng cấp thuật toán gợi ý căn hộ (Hybrid SQL + Soft Score), hiển thị thẻ tương tác (Rich Cards) trực quan trong chat, đề xuất slot giờ thông minh, giảm ma sát xác thực OTP và gửi thông báo chăm sóc chủ động (Autonomous Follow-up). |

> **Kết quả đánh giá:** **5/5 tiêu chí ĐẠT** (Vượt ngưỡng yêu cầu $\ge 4/5$).

---

### 3. GATE 1 — Core action đứng vững

- [x] **Có đủ 3 thành tố cốt lõi:**
  - **Actor:** Người mua nhà (`Homebuyer / Buyer`).
  - **Object:** Căn hộ cụ thể (`Property`) + Khung giờ hẹn (`AvailabilitySlot`) $\rightarrow$ Tạo thành bản ghi `Booking` thuộc `Deal`.
  - **Completion Rule:** Bản ghi `Booking` được tạo thành công với trạng thái `REQUESTED`, cấp mã Deal và gửi thông báo duyệt tới Chủ nhà.
- [x] **Vượt qua kiểm định tiêu chí:** Đạt **5/5 tiêu chí tự kiểm** (không trượt tiêu chí nào).
- [x] **Phân định rõ ràng — Vì sao Core Action này KHÔNG phải là "mở app", "đăng nhập" hay "hỏi AI":**
  - **Không phải "Mở app / Đăng nhập":** Mở app hay đăng nhập chỉ là thao tác hạ tầng/phiên làm việc (session/login), không mang lại giá trị nghiệp vụ và không chứng minh được nhu cầu thực tế của người mua. Thậm chí, BookingBot cho phép **Guest Chat (trải nghiệm không cần đăng nhập trước)**, chứng minh việc đăng nhập thuần túy không tạo ra giá trị cốt lõi.
  - **Không phải "Hỏi AI" (Chat chung chung):** Đặt câu hỏi cho AI (*"Tìm giúp tôi căn 2PN Ocean Park"* hoặc *"Giá căn hộ ở đây thế nào?"*) chỉ là một vi thao tác giao diện (UI interaction) ở bước khám phá. Nếu người dùng hỏi 20 câu nhưng không bao giờ bấm đặt lịch xem căn nào thì họ nhận được **zero core value** (vẫn chưa có lịch xem thực tế, vẫn chưa thẩm định được căn nhà).
  - **Bản chất của `Submit Booking Request`:** Đây là hành vi mang **cam kết giao dịch thực tế (transactional intent)**. Nó kết nối toàn bộ chuỗi giá trị: từ nhu cầu hội thoại $\rightarrow$ lựa chọn căn thật $\rightarrow$ khóa khung giờ hẹn thực tế của chủ nhà. Đây chính là cột mốc phân định giữa "người dùng dạo chơi" và "khách hàng thực sự tiến tới nhận giá trị cốt lõi".

---

## 02 — Nature & cadence

### 1. Điền Action Nature Card (10')

| Thành phần | Câu hỏi định hướng | Câu trả lời của bạn (BookingBot AI Agent) |
| :--- | :--- | :--- |
| **Actor** | User, account, team hay object nào thực hiện? | **User cá nhân (Người mua nhà / Buyer)** — thực hiện dưới phiên tài khoản định danh hoặc khách vãng lai (Guest) đã xác thực SĐT/OTP. |
| **Intent** | Hành vi bắt đầu từ nhu cầu gì? | Nhu cầu **khảo sát thực tế căn hộ Vinhomes** (thẩm định chất lượng xây dựng, hướng nắng gió, tầm view, không gian sống và tiện ích nội khu) trước khi ra quyết định xuống tiền mua nhà hoặc đầu tư. |
| **Trigger** | Do user chủ động, sự kiện bên ngoài, người khác hay hệ thống kích hoạt? | • **Chủ đạo:** **User chủ động** kích hoạt khi tìm thấy căn ưng ý qua đối thoại với AI Agent và chọn slot giờ rảnh.<br>• **Bổ trợ:** **Hệ thống kích hoạt** (Autonomous follow-up: AI chủ động gợi ý khung giờ trống phù hợp hoặc thông báo căn hộ mới khớp tiêu chí đã lưu). |
| **Effort** | Mất bao nhiêu thời gian, suy nghĩ, dữ liệu? | **Mức độ nỗ lực Trung bình – Thấp (1–3 phút):**<br>• *Thời gian:* 1–2 phút trao đổi và chọn lịch trên giao diện thẻ tương tác (Rich Cards).<br>• *Nhận thức:* Đối chiếu lịch rảnh của bản thân/gia đình (thường là cuối tuần hoặc sau giờ làm việc).<br>• *Dữ liệu:* Cung cấp tiêu chí tìm kiếm, chọn slot giờ và nhập SĐT xác thực OTP nếu là khách mới. |
| **Value timing** | Value xuất hiện ngay, trễ, tích lũy, hay phụ thuộc người khác? | **Trễ có điều kiện & Phụ thuộc người khác (Delayed & Dependent):**<br>• Không nhận value tức thì tại giây bấm nút mà phải đợi Chủ nhà duyệt (`Approve`).<br>• *Tầng 1 (An tâm & Tạm giữ căn):* Xuất hiện sau vài phút đến vài giờ khi Chủ nhà duyệt $\rightarrow$ viewing hold được khóa độc quyền.<br>• *Tầng 2 (Trải nghiệm thực tế):* Xuất hiện tại thời điểm người mua trực tiếp đến xem nhà thành công. |
| **State** | Sau action, dữ liệu/trạng thái nào được giữ lại? | • Tạo bản ghi `Booking` với trạng thái `REQUESTED`.<br>• Tạo/Cập nhật bản ghi `Deal` với trạng thái `VIEWING_REQUESTED`.<br>• Tạm khóa slot thời gian tương ứng trong `AvailabilitySlot` để ngăn xung đột lịch.<br>• Lưu tiêu chí tìm kiếm vào `buyer_profiles` trong database để AI ghi nhớ cho các phiên sau. |
| **Dependency** | Có phụ thuộc nguồn cung, thành viên khác, approval, thời điểm? | • **Phụ thuộc Nguồn cung (Supply):** Căn hộ phải đang `AVAILABLE` và Chủ nhà phải mở sẵn các slot lịch trống.<br>• **Phụ thuộc Approval:** Bắt buộc phải có sự phê duyệt từ phía Chủ nhà (`Owner`).<br>• **Phụ thuộc Thời điểm:** Thời gian xem phải diễn ra trong tương lai và tuân thủ quy định ra vào của Ban quản lý tòa nhà Vinhomes. |
| **Repeat condition** | Điều kiện nào khiến action có lý do xuất hiện lại? | • Người mua muốn xem thêm 2–4 căn hộ khác để so sánh (khác tầng, view, phân khu) trước khi chốt mua.<br>• Căn đã xem chưa ưng ý $\rightarrow$ Tiếp tục tìm căn khác.<br>• Chủ nhà từ chối hoặc yêu cầu đổi giờ (`RESCHEDULE_REQUESTED`) $\rightarrow$ Kích hoạt người mua chọn lại khung giờ mới. |

---

### 2. Kết luận cadence (5')

1. **Dạng hành vi đã chọn:** **Hành vi theo dự án (Project-based Journey) kết hợp Giao dịch (Transactional).**  
   *(Mua nhà không phải là thói quen hàng ngày hay hàng tuần quanh năm; đây là một "chiến dịch mua nhà" tập trung diễn ra trong một khoảng thời gian nhất định từ 2–6 tuần, sau đó kết thúc khi hoàn tất giao dịch).*

2. **Kết luận theo chuẩn template:**
   > Đối với **người mua nhà Vinhomes (Homebuyer)**, core action **gửi yêu cầu đặt lịch hẹn xem nhà (Submit Booking Request)** thường xuất hiện **thành từng cụm 2–4 lần trong suốt hành trình tìm mua nhà kéo dài 2–4 tuần (chủ yếu tập trung vào các ngày cuối tuần)** vì **mua bất động sản là quyết định tài chính lớn đòi hỏi khảo sát và đối chiếu trực tiếp nhiều căn hộ trước khi xuống tiền, nhưng một khi đã chốt mua thành công thì nhu cầu sẽ dừng lại**. Do đó, nhịp đo phù hợp là **nhịp theo tuần (Weekly) trong suốt vòng đời dự án tìm nhà (Active Buying Journey / Cohort 30 ngày)** ở cấp **cá nhân người mua (per active buyer)**.

3. **Cân nhắc chiều sâu (Frequency vs. Value trong sản phẩm AI):**
   - **Tần suất cao hơn KHÔNG đồng nghĩa với giá trị cao hơn:** Trong sản phẩm BĐS hỗ trợ bởi AI, nếu một người mua phải gửi yêu cầu đặt lịch 15–20 lần mà vẫn chưa chốt được căn nào, đó là tín hiệu của sự ma sát hoặc thất vọng (AI gợi ý sai nhu cầu, hình ảnh không khớp thực tế, hoặc giá ảo).
   - **Đặc trưng AI:** Trợ lý AI giỏi là trợ lý giúp người mua **tìm đúng căn nhanh nhất với số lượt xem ít nhất nhưng trúng đích nhất** (tối ưu Time-to-Value và Task Completion thay vì kéo dài thời gian onscreen vô nghĩa).

---

### 3. GATE 2 — Cadence từ nature, không từ dashboard

- [x] **Kết luận đúng template:** Đầy đủ các trường `Đối với... core action... thường xuất hiện... vì... Do đó, nhịp đo phù hợp là... ở cấp...`.
- [x] **Lý do "vì" hoàn toàn đứng vững:** Dựa trên bản chất chu kỳ ra quyết định mua tài sản lớn (High-involvement decision cycle) và hành vi thực tế của khách hàng mua nhà đại đô thị.
- [x] **Nhịp đo không mâu thuẫn với dạng hành vi:** Chọn đo lường **Weekly / Journey-based (Cohort 30 ngày)**, dứt khoát không rơi vào bẫy áp đặt các chỉ số DAU/Daily máy móc từ giao diện dashboard thông thường.
