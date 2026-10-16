# NỘI DUNG THAM KHẢO ĐỂ TỰ VIẾT TAY (Lab 7)

Mỗi trang ghi ở góc trên: Họ tên - MSSV - Ngày làm bài - Chữ ký tươi (bút xanh).
Hãy diễn đạt lại theo ý hiểu của bạn khi chép tay.

---

## TRANG 1 – Sơ đồ CI Pipeline (vẽ 6 khối nối bằng mũi tên)

[Git Push Event] → [Checkout Code] → [Setup Python 3.11] → [Install Dependencies] → [Execute Unit Tests] → [Quality Gate]

Ghi chú cạnh khối cuối: Linter sạch + Coverage >= 80% => PASS (xanh); ngược lại => FAIL (đỏ), chặn merge.

---

## TRANG 2 – Bảng thiết kế Test Case (kẻ 4 cột, 7 dòng)

| Tên Test Case | Hàm được gọi | Đối tượng & Giá trị Mocking | Kết quả kỳ vọng (Assert) |
|---|---|---|---|
| Online, mã hợp lệ | apply_promo_code | network_status="online", os=15, mã GIAM10, total=100000 | trả về 90000 |
| Offline | apply_promo_code | network_status="offline" | ném ConnectionError |
| Mã đặc biệt trên OS 15 | apply_promo_code | os_version=15, mã NEW_OS_ONLY | trả về 80000 |
| Mã đặc biệt trên OS 14 | apply_promo_code | os_version=14, mã NEW_OS_ONLY | ném ValueError |
| GPS trong vùng | get_nearby_restaurants | @patch get_current_location, return_value=(10.0, 105.0) | danh sách 2 nhà hàng |
| GPS ngoài vùng | get_nearby_restaurants | return_value=(20.0, 110.0) | danh sách rỗng [] |
| GPS mất tín hiệu | get_nearby_restaurants | side_effect=GPSSignalLostError | chuỗi "Khong the xac dinh vi tri." |

---

## TRANG 3 – Câu 2.1, 2.2, 2.3

**Câu 2.1 – Vì sao Linter đặt trước bước chạy test?**
Linter chạy rất nhanh và không cần thực thi code, nên bắt được lỗi cú pháp/tên chưa định nghĩa ngay lập tức. Nếu code sai cú pháp thì chạy test cũng vô nghĩa và tốn thời gian máy chủ. Đây là nguyên lý Fail-Fast: thất bại sớm, tiết kiệm tài nguyên, phản hồi nhanh cho lập trình viên.

**Câu 2.2 – Cơ chế của --fail-under=80 trong Linux?**
Mọi tiến trình Linux kết thúc bằng một exit code: 0 là thành công, khác 0 là thất bại. Khi tổng độ bao phủ nhỏ hơn 80%, `coverage report` thoát với mã khác 0 (mã 2). Shell của runner thấy step trả về mã khác 0 nên GitHub Actions đánh dấu step thất bại, dừng các step sau và báo build đỏ (Break the Build). Nếu >= 80% thì exit code 0 và pipeline tiếp tục.

**Câu 2.3 – Tham số -m?**
-m (missing) thêm cột "Missing" liệt kê số dòng code chưa được test chạy tới. Nhờ đó kỹ sư biết chính xác dòng nào cần viết thêm test thay vì đoán mò, giúp tăng coverage nhanh và có định hướng.

---

## TRANG 4 – Câu 3.1, 3.2, 3.3

**Câu 3.1 – Flaky Test và lỗi sai lệch môi trường phụ thuộc**
- Flaky Test: test lúc đạt lúc không dù code và dữ liệu không đổi, do yếu tố không ổn định như thời gian, thứ tự chạy, mạng, tín hiệu phần cứng. Ví dụ: test gọi GPS thật, đôi khi tín hiệu nhiễu/mất nên kết quả lúc đúng lúc sai. Cách khắc phục: mock GPS để kết quả xác định.
- Lỗi sai lệch môi trường phụ thuộc: test ổn định nhưng luôn sai ở môi trường này và đúng ở môi trường khác, do khác phiên bản thư viện/Python/HĐH (ví dụ máy cá nhân dùng thư viện bản mới, CI dùng bản cũ). Cách khắc phục: ghim phiên bản, dùng môi trường sạch giống nhau.
- Khác biệt chính: Flaky là kết quả ngẫu nhiên trong cùng một môi trường; lỗi môi trường là kết quả nhất quán nhưng khác nhau giữa các môi trường.

**Câu 3.2 – Vì sao phải ghim phiên bản (Version Pinning)?**
Runner CI là máy sạch, mỗi lần chạy cài lại thư viện từ đầu. Nếu không ghim, pip lấy bản mới nhất, có thể thay đổi API hoặc có lỗi làm build đỏ dù code không đổi. Ghim phiên bản (==) đảm bảo mọi lần build và mọi máy dùng đúng cùng một bộ thư viện: build tái lập được, ổn định, dễ truy vết lỗi, và kiểm soát được lúc nào nâng cấp.

**Câu 3.3 – Quy trình 6 bước nâng cấp thư viện an toàn qua Pull Request**
1. Tạo nhánh mới từ main (ví dụ `upgrade/coverage-7.5`), không sửa trực tiếp main.
2. Sửa số phiên bản mới trong requirements.txt và đọc changelog/breaking changes.
3. Cài lại dependencies và chạy toàn bộ test + linter ở máy cá nhân.
4. Commit, push nhánh và mở Pull Request vào main.
5. Chờ CI chạy tự động trên PR; sửa lỗi nếu có (nhánh đỏ thì không merge); nhờ đồng nghiệp review.
6. CI xanh và được duyệt thì merge vào main, theo dõi pipeline sau merge; nếu lỗi thì revert PR.
