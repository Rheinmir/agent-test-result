# 📱 Mobile QA Report — DMS
**Date:** 2026-04-09  
**Device Viewport:** iPhone 14 Pro (390x844px)  
**Environment:** Production (`https://dms.giatbh.io.vn`)

---

## 🛑 Bức Tranh Tổng Quan
Trải nghiệm thực tế trên Mobile hiện đang gặp 2 vấn đề lớn:
1. **Lỗi nghiêm trọng (Blocker):** Người dùng nội bộ không thể đăng nhập bằng tài khoản Local (Alternative Admin Login) vì nút căn chỉnh sai và bị ẩn hoàn toàn trên Mobile.
2. **State Amnesia (Mất trí nhớ giao diện):** Tính năng nhớ URL state dường như chỉ hoạt động trơn tru trên giao diện Desktop. Trên Mobile, người dùng rơi vào vòng lặp "trượt lỗi" mỗi khi back ra.

---

## 🐞 Mobile Bug Triage

### 🚨 BUG-012 (CRITICAL): Mất nút đăng nhập Local trên Mobile
- **Hiện tượng:** Tại màn hình `/login`, người dùng chỉ thấy duy nhất nút **Sign in with Microsoft**. Textlink "Alternative Admin Login" bốc hơi hoàn toàn ở màn hình có chiều rộng `< 1024px`.
- **Hậu quả:** Blocker. Bất kỳ ai định đăng nhập bằng tài khoản nội bộ (như đội ngũ Testing hoặc `fpt.dms`) qua điện thoại di động đều bị chặn đứng ngoài cửa.
- **Đề xuất Fix:** Bỏ class Tailwind dạng `hidden md:block` hoặc CSS gây display none ở thiết bị kích cỡ nhỏ trong file layout của form Login.

---

### ❌ BUG-013: State Amnesia tái phát trên Mobile Codebase
Có vẻ màn hình Dashboard Mobile đang chạy một Component điều hướng Back khác hoàn toàn bản Desktop:
- **Trạng thái Search (Khớp BUG-010):** Gõ "ME35" -> ấn vào xem Detail -> ấn mũi tên Back nhỏ trên góc trái màn hình Mobile. Ô tìm kiếm sau đó bị làm làm trắng tinh.
- **Trạng thái Tab mâu thuẫn (Khớp BUG-009):** Đang chủ động chọn tab "ALL" (dưới thanh Bottom Navigation). Tuy nhiên khi ấn Back ra, thanh Toggle Component trên cùng lại **nhảy xanh lá cây về tab PENDING**, kết quả là màn hình báo "Không có yêu cầu chờ duyệt".
- **Đề xuất Fix:** Mặc dù Desktop đã fix xong, nhưng Dev cần tái sử dụng hàm bốc URL Param (Single Source of Truth) vào Header Tab lẫn Bottom Nav trên **cả giao diện mobile** để dập tắt xung đột UI này.

---

### ✨ Những Điểm Cộng Sáng Giá (PASS)
1. **Responsive Design:** Nội dung Text trong danh sách co dãn rất mềm mại trên màn hình hẹp 390px. Không bị hiện tượng cuộn ngang (horizontal scroll) dư thừa làm bể khung. Thẻ Accordion hoạt động rất gọn gàng.
2. **Mobile Pagination (Infinite Scroll):** Chức năng phân trang bên Desktop đã được thế vai một cách xuất sắc bằng thao tác kéo tự động tải thêm dữ liệu (Infinite Scroll). Chạm trúng thói quen tự nhiên của người dùng di động. 
3. **Bottom Navigation:** To, Hit Box lớn dễ bấm ngón cái, thiết kế giống chuẩn App Native hiện đại.

*(Note nhỏ cho team: Nút Hamburger [3 sọc] trên cùng bên trái hiện tại nhấp vào chả có menu nào mở ra cả. Nếu thừa thì bỏ luôn chức năng đó cho layout trơn tru hơn).*
