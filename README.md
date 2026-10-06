# Track1_Day20_02612_HoangAnhTai

- **Họ tên:** Hoàng Anh Tài
- **MHV:** 2A202602612

---

**00 — Phạm vi**

1. **Dự án:** BookingBot AI Agent · Trợ lý Đặt lịch Xem nhà & Giữ căn Vinhomes
2. **Persona:** Người mua nhà (Homebuyer) có nhu cầu tìm mua căn hộ ở thực hoặc đầu tư tại đại đô thị Vinhomes; bận rộn, coi trọng thời gian, sợ tin ảo và ngại bị môi giới telesale làm phiền.
3. **Core job:** Tìm kiếm đúng căn hộ Vinhomes phù hợp tiêu chí và chốt được lịch hẹn đi xem nhà thực tế một cách nhanh chóng, minh bạch.

---

### 1. Bảng 4 Khái Niệm Cốt Lõi (Core Framework)

| Khái niệm | Câu hỏi định hướng | Áp dụng cho BookingBot AI Agent |
| :--- | :--- | :--- |
| **Core job** | User đang cố hoàn thành việc gì? | **Tìm đúng căn hộ Vinhomes ưng ý và đặt được lịch hẹn đi xem nhà thực tế với chủ nhà/CĐT.** |
| **Core action** | User làm gì trong sản phẩm để tiến tới giá trị? | **Chat tìm căn theo tiêu chí $\rightarrow$ Chọn căn phù hợp $\rightarrow$ Chọn khung giờ rảnh và bấm gửi yêu cầu đặt lịch hẹn (Booking Request).** |
| **Core value** | User nhận được lợi ích gì? | **Không tốn thời gian trao đổi qua lại; chắc chắn có lịch hẹn xem căn thật và được tạm giữ căn (Viewing Hold chống trùng lịch) trong khung giờ xem.** |
| **Core value event** | Sự kiện nào chứng minh value đã xảy ra? | **`booking_confirmed`** *(Chủ nhà duyệt lịch & Viewing Hold được kích hoạt thành công trên hệ thống)* |

---

### 2. Điền Core Action Card (10')

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
