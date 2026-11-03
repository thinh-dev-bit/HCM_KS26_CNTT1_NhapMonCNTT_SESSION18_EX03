# Phần 1. Phân tích: 
**1. Định vị phần tử (Inspect Element):**
*   Sử dụng tổ hợp phím **Ctrl + Shift + C** (hoặc nhấp chuột phải chọn *Inspect*).
*   Trỏ chuột thẳng vào Khung "Thông tin tài xế" trên màn hình. DevTools sẽ tự động mở tab **Elements** và bôi xanh đúng thẻ HTML đang kiểm soát khung này trong cây DOM Tree.

**2. Phân tích Box Model (Tab Computed):**
Thay vì nhìn vào mã CSS lộn xộn, chuyển sang tab **Computed** (nằm bên phải hoặc bên dưới tab Styles) để xem sơ đồ hình hộp 4 lớp (Box Model) mà Rendering Engine đã tính toán:
*   **Content (Nội dung):** Vùng lõi chứa chữ và ảnh tài xế.
*   **Padding (Vùng đệm):** Khoảng cách từ nội dung chữ đến viền của khung.
*   **Border (Viền):** Đường viền bao quanh khung.
*   **Margin (Lề ngoài):** Khoảng cách từ khung này đến khung "Danh sách trạm" kế bên.
*   *Chẩn đoán lỗi:* Thông qua tab Computed, ta phát hiện lớp **Margin-bottom** (lề dưới) đang bị thiết lập giá trị âm (ví dụ: `-50px`), khiến khung tài xế bị kéo thụt xuống và đè trực tiếp lên khung "Danh sách trạm" ở dưới.

---

# Phần 2. Triển khai: 

Để khắc phục hiện tượng chồng lấn mà không chạm vào mã nguồn gốc, ta thực hiện chỉnh sửa trực tiếp trên bộ nhớ RAM của trình duyệt:

**Quy trình can thiệp CSS:**
1.  Vẫn chọn thẻ HTML của Khung "Thông tin tài xế" bên tab Elements.
2.  Nhìn sang bảng **Styles** (hiển thị các class CSS đang áp dụng).
3.  Tìm thuộc tính `margin-bottom: -50px;` đang gây lỗi. Click thẳng vào con số này và sửa lại thành số dương, ví dụ: **`margin-bottom: 24px;`**.
4.  Ngay lập tức, trình duyệt sẽ vẽ (paint) lại giao diện. Khung tài xế sẽ đẩy khung danh sách trạm xuống dưới tạo ra khoảng cách 24px tách biệt hoàn toàn hai phần tử.
5.  *Kết quả đầu ra:* Chụp ảnh màn hình giao diện đã sửa kèm thông số CSS ở tab Styles để gửi lại cho đội Dev cập nhật vào file mã nguồn gốc.

---

# Phần 3. Kiểm thử: 

**Tình huống giả định:** Đã thay đổi `margin-bottom: 24px;` ở tab Styles nhưng khung "Thông tin tài xế" vẫn đứng im không di chuyển, giao diện vẫn bị đè lên nhau.

**Chẩn đoán nguyên nhân (Xung đột ở tầng Rendering Engine):**
1.  **Lỗi tách luồng (Positioning):** Phần tử đang bị áp dụng thuộc tính `position: absolute;` hoặc `position: fixed;`. Thuộc tính này ra lệnh cho Rendering Engine bứt phần tử đó ra khỏi luồng bố cục thông thường (Normal Flow). Khi ở trạng thái lơ lửng này, các thuộc tính `margin` để đẩy các phần tử khác sẽ bị vô hiệu hóa.
    *   *Khắc phục:* Tìm thuộc tính `position` trong tab Styles, thử tắt nó đi (bỏ dấu tích) hoặc chuyển về `position: relative;`.
2.  **Ghi đè độ ưu tiên (CSS Specificity & `!important`):** Khai báo margin bạn vừa nhập bị một dòng CSS khác ở file khác đè lên (bị gạch ngang chữ trong tab Styles). Dòng đè lên có thể mang mức ưu tiên cao hơn do chứa cờ `!important`.
    *   *Khắc phục:* Gõ thêm `!important` vào sau giá trị vừa sửa (ví dụ: `margin-bottom: 24px !important;`) để ép trình duyệt nhận diện lệnh này ở mức ưu tiên cao nhất.