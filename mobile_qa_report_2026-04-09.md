# 📱 Mobile QA Report — DMS
**Date:** 2026-04-09 (Updated after Dev Fixes)  
**Device Viewport:** iPhone 14 Pro (390x844px)  
**Environment:** Production (`https://dms.giatbh.io.vn`)

---

## 🛑 Bức Tranh Tổng Quan (Cập Nhật)
Sau bản vá nóng (hotfix) từ Dev Team, trải nghiệm trên Mobile đã lột xác hoàn toàn. Tình trạng State Amnesia (mất trí nhớ điều hướng) bị khai tử. 

---

## 🐞 Mobile Bug Triage

### 🚨 BUG-012 (CRITICAL): Mất nút đăng nhập Local trên Mobile — [⏳ ĐANG CHỜ XỬ LÝ RIÊNG]
- **Hiện tượng:** Tại màn hình `/login`, nút "Alternative Admin Login" bốc hơi ở màn hình có chiều rộng `< 1024px`.
- **Note từ Dev Team:** *Bỏ qua theo yêu cầu, sẽ xử lý ở một ticket/nhánh UI riêng.*

---

### ✅ BUG-013: State Amnesia tái phát trên Mobile Codebase — [ĐÃ FIX TRIỆT ĐỂ]

Dev đã khắc phục thành công việc đồng nhất Single Source of Truth giữa URL và cả 2 Component (Top Toggle, Bottom Nav) trên Mobile:

- **BUG-013a (Trạng thái Search):** Gõ "ME35" -> ấn vào xem Detail -> ấn mũi tên Back. Mọi thứ được giữ nguyên. URL hiển thị chuẩn `?q=ME35`.
- **BUG-013b (Trạng thái Tab):** 
  - Đang chủ động chọn tab "ALL" (dưới thanh Bottom Navigation).
  - Khi ấn Back quay lại từ trang chi tiết, **Cả thanh dưới (Bottom Nav) và thanh trên (Top Toggle) đều đồng loạt highlight xanh ở chữ "ALL"**. 
  - Không còn tình trạng "trên cắm PENDING, dưới cắm ALL" như trước đó nữa. 
  - URL chuẩn xác: `?filter=all`.

---

### ✨ Những Điểm Cộng Sáng Giá (PASS)
1. **Responsive Design:** Nội dung Text trong danh sách co dãn mềm mại trên màn hình hẹp 390px. Không bị cuộn ngang.
2. **Mobile Pagination (Infinite Scroll):** Chức năng kéo tự động tải thêm dữ liệu rất mượt. Bây giờ lỗi State Amnesia đã hết, việc cuộn xem chi tiết quay lại không bị văng về đầu trang nữa, trải nghiệm cực kỳ liền mạch.
3. **Bottom Navigation:** Hit Box cực lớn, dễ bấm ngón cái, chuẩn Native App.

*(Note nhỏ cho team: Nút Hamburger [3 sọc] trên cùng bên trái hiện tại nhấp vào chả có menu nào mở ra cả. Nếu thừa thì bỏ luôn chức năng đó cho layout trơn tru hơn).*
