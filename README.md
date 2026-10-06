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

---

## 03 — Metric System

### 1. Activation metric (5')

- **Start event:** `first_query_sent`  
  *Thời điểm:* Người mua gửi tin nhắn đầu tiên nêu tiêu chí tìm nhà trên giao diện chat của BookingBot (bắt đầu phiên hội thoại tìm kiếm).
- **Activation event:** `first_booking_requested` *(kèm điều kiện nhận xác nhận `first_booking_confirmed`)*  
  *Thời điểm:* Người mua gửi yêu cầu đặt lịch hẹn xem nhà đầu tiên cho một căn hộ cụ thể và lịch hẹn được đưa vào quy trình tạm giữ chỗ. Đây là sự kiện xác nhận người mua đã đi qua toàn bộ phễu giá trị ban đầu và chạm vào giá trị cốt lõi.
- **Time window:** Trong vòng **72 giờ (3 ngày)** kể từ `Start event`.  
  *Cơ sở:* Nhu cầu tìm mua nhà có tính tập trung cao độ trong 1–3 ngày đầu tiên. Dữ liệu thực tế cho thấy nếu người mua không chuyển hóa thành một yêu cầu xem nhà cụ thể trong vòng 72 giờ, xác suất họ rời bỏ sản phẩm sang các kênh môi giới truyền thống là trên 80%.

> **Tránh lỗi thường gặp:** Không dùng "Hoàn thành tour hướng dẫn" hay "Đăng nhập/Tạo tài khoản" làm Activation, vì người dùng có thể tạo tài khoản nhưng không tìm được căn nào phù hợp và rời đi mà chưa nhận được bất kỳ giá trị thực tế nào.

---

### 2. Engagement metric (3')

Chọn 2 góc đo phù hợp nhất với bản chất sản phẩm:

1. **Góc đo Frequency (Tần suất trong nhịp tự nhiên):**
   - **Chỉ số:** `Weekly Bookings per Active Buyer` — Số lượt gửi yêu cầu xem nhà trung bình mỗi tuần của một người mua đang active trong hành trình tìm nhà.
   - **Kỳ vọng:** **1.5 – 2.5 bookings / active week** (Tập trung khảo sát vào thứ 7 và chủ nhật).
2. **Góc đo Depth (Độ sâu / Chất lượng giá trị của hành động):**
   - **Chỉ số:** `Viewing Confirmation & Completion Rate` — Tỷ lệ các yêu cầu đặt lịch được Chủ nhà phê duyệt (`Confirmed`) và khách hàng thực tế đến tham quan (`Completed`):
     $$\text{Viewing Quality Depth} = \frac{\text{Số lịch hẹn đã đi xem thực tế (Completed)}}{\text{Tổng số yêu cầu đặt lịch (Requested)}} \times 100\%$$
   - **Kỳ vọng:** $\ge 65\%$ (Đảm bảo mỗi lượt booking đều là nhu cầu thật, hạn chế tối đa booking ảo hoặc bị hủy).

---

### 3. North Star Metric, Leading Indicators & Counter-metric (10')

#### A. North Star Metric (NSM)
* **Công thức chuẩn 3 thành phần:**
  $$\text{NSM} = [\text{Unit of value}] + [\text{Quality threshold}] + [\text{Frequency}]$$
* **Tên chỉ số NSM của BookingBot:**
  > **Weekly Confirmed & Held Viewings** *(Số lượt xem nhà được Chủ nhà phê duyệt và kích hoạt Viewing Hold thành công mỗi tuần)*
* **Bóc tách 3 thành phần:**
  - **Unit of value:** Lượt xem nhà thực tế được đảm bảo giữ chỗ độc quyền (`Confirmed & Held Viewing`).
  - **Quality threshold:** Căn hộ có Sổ đỏ xác thực, Chủ nhà duyệt lịch hợp lệ, và ràng buộc cơ sở dữ liệu (Exclusion Constraint) kích hoạt Viewing Hold thành công không xảy ra lỗi trùng lịch (Zero double-booking).
  - **Frequency:** Đo lường theo nhịp tự nhiên hàng tuần (`Weekly`).
* **Ý nghĩa:** Phản ánh trực tiếp giá trị win-win của cả hai bên: Người mua chắc chắn có lịch xem căn thật không bị tranh chấp, Chủ nhà đón tiếp đúng khách mua tiềm năng, AI Agent hoàn thành xuất sắc vai trò điều phối.

#### B. Leading Indicators (3 chỉ số dẫn dắt dự báo)
1. **Match-to-Card CTR (Tỷ lệ tương tác với thẻ căn hộ AI gợi ý):**
   - *Công thức:* Tỷ lệ người mua bấm xem chi tiết căn hộ từ các thẻ Rich Cards mà AI đề xuất trong đoạn chat.
   - *Dự báo:* Nếu người mua click xem chi tiết $\ge 3$ căn trong phiên chat, điều này chứng minh AI đã hiểu đúng tiêu chí (budget, vị trí, tiện ích) $\rightarrow$ xác suất chuyển đổi sang `Submit Booking Request` tăng hơn 3.5 lần.
2. **Slot Matching Success Rate (Tỷ lệ khớp khung giờ rảnh):**
   - *Công thức:* Tỷ lệ phiên tìm kiếm tìm thấy ít nhất 1 khung giờ rảnh chung giữa Người mua và Chủ nhà trong vòng 48 giờ tới.
   - *Dự báo:* Nếu có sẵn slot giờ khớp ngay lập tức, ma sát chờ đợi được triệt tiêu $\rightarrow$ dự báo số lượt bấm đặt lịch hoàn tất trong phiên tăng thêm 60%.
3. **Owner Median Response Time (Thời gian phản hồi trung vị của Chủ nhà):**
   - *Công thức:* Thời gian trung vị từ khi Người mua gửi yêu cầu đến khi Chủ nhà bấm `Approve` hoặc `Reject`.
   - *Dự báo:* Nếu Chủ nhà phản hồi dưới 30 phút, tâm lý hào hứng của khách mua được duy trì $\rightarrow$ dự báo tỷ lệ khách tiếp tục đặt lịch xem thêm căn thứ 2 (Repeat Booking) trong cùng tuần tăng 45%.

#### C. Counter-metrics (Chỉ số đối trọng chống game metric)
1. **Buyer No-Show Rate (Tỷ lệ khách bỏ hẹn không đến):**
   - *Rủi ro nếu bị game:* Nếu AI thúc ép hoặc tự động đặt lịch vô tội vạ để "thổi phồng" NSM, số lượt booking tăng vọt nhưng khách không đến xem nhà $\rightarrow$ làm phiền Chủ nhà, mất uy tín nền tảng.
   - *Ngưỡng kiểm soát:* **Tỷ lệ No-show phải duy trì $\le 10\%$.**
2. **Double-Booking / Hold Conflict Rate (Tỷ lệ xung đột lịch hoặc lỗi trùng căn):**
   - *Rủi ro nếu bị game:* Lỗi tranh chấp tài nguyên do ép tăng tải giao dịch dẫn đến 2 khách cùng được cấp quyền giữ 1 căn tại 1 thời điểm.
   - *Ngưỡng kiểm soát:* **Tuyệt đối bằng 0 (0% Tolerance).**
3. **AI Spec-Mismatch Complaint Rate (Tỷ lệ khiếu nại thông tin sai lệch từ AI):**
   - *Rủi ro nếu bị game:* AI "ảo giác" (hallucinate) nói sai về phí dịch vụ, tầm view, hoặc hướng nhà để dụ khách đặt lịch.
   - *Ngưỡng kiểm soát:* **Tỷ lệ khiếu nại sai lệch thông tin $\le 2\%$ tổng số lượt xem.**

---

## 04 — Retention Definition

### 1. Bảng 6 thành phần Retention chuẩn mực

| Thành phần | Câu hỏi định hướng | Định nghĩa cho BookingBot AI Agent |
| :--- | :--- | :--- |
| **Unit** | User, account, team, organization hay object? | **User cá nhân (Người mua nhà / Buyer)** — định danh qua số điện thoại / User ID. |
| **Cohort entry** | Event nào đưa unit vào cohort? | **`first_booking_requested`** — Thời điểm người mua gửi yêu cầu đặt lịch xem nhà đầu tiên hợp lệ. |
| **Return event** | Core action / value event nào phải lặp lại? | **`repeat_booking_requested`** (Gửi yêu cầu xem căn tiếp theo) HOẶC **`viewing_completed`** (Đến xem nhà thực tế thành công). |
| **Window** | Daily, weekly, monthly, project-based hay custom bracket? | **Custom Journey Brackets trong hành trình tìm nhà 30 ngày (Weekly Brackets):**<br>• *Bracket 1 (W1):* Ngày 1 – 7 kể từ cohort entry.<br>• *Bracket 2 (W2):* Ngày 8 – 14.<br>• *Bracket 3 (W3):* Ngày 15 – 21.<br>• *Bracket 4 (W4):* Ngày 22 – 30. |
| **Threshold** | Một lần hay nhiều lần trong window? | **$\ge 1$ lần** Return Event trong mỗi tuần của cửa sổ 30 ngày. |
| **Segment** | Retention đang áp dụng cho ai? | **Active Homebuyers Cohort** — Khách hàng có nhu cầu mua nhà thực tế trong 30 ngày (loại trừ tài khoản test nội bộ, môi giới bị gắn cờ spam, và khách hàng đã chốt cọc mua thành công). |

---

### 2. Đối chiếu chuẩn mực Cadence & Tính chất danh mục (Category Benchmark)

- **Tránh bẫy D7 / Daily Retention:**
  Bất động sản là ngành hàng giá trị cao (High-ticket transactional category). Việc đòi hỏi khách hàng quay lại "mỗi ngày" (Daily retention) hay chỉ nhìn chỉ số "D7 đơn lẻ" là sai lệch hoàn toàn về bản chất hành vi người dùng. Người mua chỉ đi xem nhà tập trung vào các ngày cuối tuần.
- **Khái niệm "Healthy Exit" (Rời cohort tích cực):**
  Khi người mua hoàn tất giao dịch cọc (`deal_completed_deposit`), họ sẽ dừng việc đặt lịch xem nhà. Trong mô hình retention của BookingBot, trường hợp này được ghi nhận là **Hoàn thành mục tiêu thành công (Success State)** chứ không bị tính là Churn.

---

## GATE 3 — Metric tính được, retention đủ nghĩa

- [x] **Activation hoàn chỉnh:** Có đầy đủ `Start event` (`first_query_sent`), `Activation event` (`first_booking_requested` / `first_booking_confirmed`), và `Time window` (72 giờ). Không dùng các thao tác giao diện hời hợt (mở app, xem tour).
- [x] **Retention đủ 6 thành phần:** Đầy đủ `Unit`, `Cohort entry`, `Return event`, `Window`, `Threshold`, `Segment` và hoàn toàn đồng nhất với nhịp Cadence tuần/hành trình 30 ngày đã xác lập ở Phase 2.
- [x] **North Star Metric đúng chuẩn:** Tuân thủ cấu trúc 3 thành phần `[Unit of value: Confirmed & Held Viewing] + [Quality threshold: Sổ đỏ + Không trùng lịch] + [Frequency: Weekly]`.
- [x] **Có đủ Leading Indicators & Counter-metrics:** Gồm 3 chỉ số dẫn dắt thực tế và 3 chỉ số đối trọng nghiêm ngặt (chống No-show, chống Double-booking, chống AI Hallucination).

---

## 05 — Product Loop

### 1. Sơ đồ Product Loop (Tối thiểu 2 chu kỳ)

Mô hình vòng lặp sản phẩm tự nhiên của BookingBot AI Agent trải qua 2 chu kỳ khảo sát và đối chiếu:

```text
[Chu kỳ 1: Khám phá & Đặt lịch căn đầu tiên]
Natural Trigger 1: Có nhu cầu mua nhà Vinhomes, mệt mỏi vì tin ảo/môi giới telesale
  │
  ▼
Core Action 1: Chat với AI nêu tiêu chí → Chọn căn phù hợp → Gửi yêu cầu đặt lịch (submit_booking_request)
  │
  ▼
Immediate Value 1: Nhận Deal ID, Chủ nhà duyệt lịch → Viewing Hold khóa căn độc quyền chống trùng lịch
  │
  ▼
Saved State / Investment 1: Hệ thống lưu tiêu chí (buyer_profiles), lưu lịch sử tương tác & căn đã quan tâm
  │
  ▼
[Chuyển tiếp tự nhiên không cần spam Notification]
Next Natural Trigger: Khách vừa đi xem căn 1 về → Phát sinh nhu cầu đối chứng/so sánh thêm căn thứ 2
  │
  ▼
[Chu kỳ 2: Khảo sát đối chứng & Củng cố niềm tin]
Core Action 2: Mở lại bot (hồ sơ đã lưu sẵn) → Chọn căn thứ 2 để đối chiếu → Gửi yêu cầu đặt lịch căn 2
  │
  ▼
Repeat Value 2: Tiếp tục chốt lịch xem so sánh có bảo đảm giữ căn → Đầy đủ dữ kiện thực tế để tự tin chốt cọc
```

---

### 2. Loại loop & Metric Hypothesis

1. **Loại loop chính đã chọn:** **Project-based Comparison Loop (Vòng lặp khảo sát đối chiếu theo dự án mua nhà).**

2. **Metric Hypothesis (Bắt buộc một câu):**
   > Nếu loop này hoạt động, metric **Viewing Confirmation & Completion Rate** sẽ thay đổi theo hướng **tăng từ 45% lên trên 65%** trong **khung thời gian 30 ngày (Active Buying Cohort)**, vì **việc lưu trữ hồ sơ tiêu chí người mua (Saved State) giúp AI đề xuất các căn đối chứng chính xác hơn, giảm 60% thời gian tìm kiếm ở chu kỳ 2 và thúc đẩy người mua tự tin hoàn thành lịch xem thực tế**.

3. **Phân tích chiều sâu — "Reason to Return" nếu loại bỏ hoàn toàn Notification:**
   - **Bản chất hành vi mua nhà:** Mua bất động sản là quyết định tài chính hệ trọng; người mua không bao giờ mua ngay căn đầu tiên mà luôn có nhu cầu tự nhiên phải xem ít nhất 2–3 căn để so sánh giá, view, tầng và nội thất.
   - **Sức hút từ Saved State (Tài sản dữ liệu đã lưu):** Khi không có bất kỳ thông báo nhắc nhở nào, người mua vẫn chủ động quay lại vì BookingBot đã đóng vai trò là "Bàn làm việc thẩm định BĐS" của riêng họ (đã lưu sẵn ngân sách, hướng nhà, các căn đã khảo sát, Deal ID đang theo dõi). Quay lại BookingBot giúp họ tiếp tục tiến trình ngay lập tức mà không phải tốn công giải thích lại từ đầu cho môi giới mới.

---

## 06 — Tracking nhanh

### 1. Bảng Core Events (4–8 core events dạng `object_action`)

| Tên Event | Ý nghĩa (Hành vi / Value đại diện) | Thời điểm ghi nhận (Trigger Point) | Metric sử dụng (Map về Phase 3) |
| :--- | :--- | :--- | :--- |
| **`query_sent`** | Người mua gửi tin nhắn tìm kiếm hoặc lọc căn hộ thành công qua khung chat. | Khi backend tiếp nhận và lưu tin nhắn vào bảng `ai_conversations`. | `Activation: Start Event` (lần đầu là `first_query_sent`). |
| **`card_clicked`** | Người mua bấm xem chi tiết một căn hộ cụ thể từ thẻ tương tác do AI gợi ý. | Khi người dùng click vào thẻ Rich Card căn hộ trong giao diện chat. | `Leading Indicator 1: Match-to-Card CTR`. |
| **`booking_requested`** | Người mua hoàn tất gửi yêu cầu đặt lịch xem nhà cho căn hộ và slot giờ cụ thể. | Khi bản ghi `Booking` được insert vào database với trạng thái `REQUESTED` và sinh mã `deal_id`. | `Activation Event`, `Engagement: Weekly Bookings`, `Retention: Cohort Entry`. |
| **`viewing_confirmed`** | Chủ nhà duyệt lịch hẹn và hệ thống kích hoạt Viewing Hold khóa căn thành công. | Khi `Booking` chuyển sang `CONFIRMED` và bản ghi `PropertyHold` chuyển sang `ACTIVE`. | **`North Star Metric (NSM)`**, `Engagement: Depth`. |
| **`viewing_completed`** | Buổi xem nhà thực tế diễn ra thành công (khách có mặt tại căn hộ). | Khi Chủ nhà hoặc Khách bấm xác nhận "Đã hoàn thành buổi xem", chuyển `Booking` sang `COMPLETED`. | `Retention: Return Event`, `Engagement: Depth`, `Counter-metric 1: Buyer No-Show Rate` (nghịch đảo). |
| **`viewing_cancelled`** | Lịch xem nhà bị hủy bởi Người mua hoặc Chủ nhà, hoặc hết hạn chờ duyệt. | Khi bản ghi `Booking` chuyển sang trạng thái `CANCELLED` hoặc `REJECTED`. | `Counter-metric 1: Buyer No-Show & Drop Rate`. |

---

### 2. Tiêu chí nghiệm thu Tracking (Acceptance Criteria)

1. **Tiêu chí 1 (Chỉ ghi nhận khi hành vi thực sự hoàn tất ở Backend):**
   - Với event `booking_requested`, hệ thống **chỉ được phép bắn event** khi API `POST /api/bookings` trả về mã HTTP `201 Created` và cơ sở dữ liệu đã commit thành công bản ghi `Booking` kèm mã `deal_id`. Tuyệt đối không bắn event tại thời điểm người dùng mới click nút trên giao diện khi chưa có phản hồi từ máy chủ.
2. **Tiêu chí 2 (Idempotency — Chống ghi trùng do Reload / Retry / Network Lag):**
   - Với mỗi cặp định danh `(buyer_id, booking_id)`, sự kiện `booking_requested` và `viewing_confirmed` chỉ được ghi nhận **đúng 1 lần duy nhất trong hệ thống Analytics**. Thao tác tải lại trang (Reload), mạng chập chờn gửi lại request (Network retry), hoặc các thao tác cập nhật ghi chú sau đó tuyệt đối không được tạo thêm event trùng lặp cho cùng một lượt đặt lịch.
3. **Tiêu chí 3 (Chống ghi nhận giá trị ảo cho sự cố trùng lịch):**
   - Sự kiện `viewing_confirmed` chỉ được phát ra khi giao dịch database tạo `PropertyHold` thành công qua ràng buộc loại trừ (`ex_property_holds_no_overlap`). Nếu database báo lỗi xung đột khung giờ (Collision), event `viewing_confirmed` **tuyệt đối không được kích hoạt**.

---

## GATE 4 — Loop nối metric, event nối loop

- [x] **Loop $\ge 2$ chu kỳ:** Thiết kế rõ nét 2 chu kỳ: Chu kỳ 1 (Khám phá & Đặt căn đầu) $\rightarrow$ Saved State (Profile & Deal ID) $\rightarrow$ Next Natural Trigger (Nhu cầu đối chứng) $\rightarrow$ Chu kỳ 2 (Đặt căn so sánh & Chốt cọc).
- [x] **Metric Hypothesis chuẩn xác:** Trỏ trực tiếp về metric `Viewing Confirmation & Completion Rate` ở Phase 3 với dự báo định lượng cụ thể (tăng từ 45% lên 65% trong 30 ngày).
- [x] **Map 100% Events về Metric:** Tất cả 6/6 events trong bảng (`query_sent`, `card_clicked`, `booking_requested`, `viewing_confirmed`, `viewing_completed`, `viewing_cancelled`) đều map trực tiếp về ít nhất một metric cốt lõi đã định nghĩa ở Phase 3.
- [x] **Tiêu chí nghiệm thu chặt chẽ:** Đạt cả 2 bẫy phổ biến: Không bắn khi mới bấm nút và Chống duplicate khi reload/retry.
