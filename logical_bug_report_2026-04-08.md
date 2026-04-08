# Logical Bug Report — DMS State Management
**Date:** 2026-04-08  
**Environment:** Production (`https://dms.giatbh.io.vn`)  
**Type:** UX / Logical (State Management)

---

## 🛑 Tổng Quan
Sau khi Retest và kiểm tra chuyên sâu về luồng điều hướng (Navigation Logic), hệ thống bộc lộ khuyết điểm nghiêm trọng về **State Management (Quản lý trạng thái UI)**. 

Khi người dùng thao tác tương tác trên danh sách (đổi tab, tìm kiếm, phân trang) sau đó nhấn vào xem từng bản ghi (Detail View) và bấm **Back**, ứng dụng bị **"mất trí nhớ" (State Amnesia)** và không thể khôi phục lại trạng thái danh sách trước đó.

---

## 🐞 Danh sách Bug Logic Mới Phát Hiện

### ❌ BUG-009: Tab State Amnesia (Mất trạng thái Tab)
- **Mức độ:** 🟠 High
- **Mô tả:** Đang ở tab "ALL", nhấn xem chi tiết của một Request, sau đó nhấn nút Back (quay lại).
- **Kết quả lỗi:** Tự động nhảy về mặc định là tab "PENDING" và báo "Không có yêu cầu chờ duyệt".
- **Kỳ vọng:** Phải giữ nguyên ngữ cảnh là tab "ALL".

### ❌ BUG-010: Search State Amnesia (Quên từ khóa tìm kiếm)
- **Mức độ:** 🟠 High
- **Mô tả:** Gõ chữ "ME35" vào ô tìm kiếm -> thấy kết quả -> nhấn xem chi tiết 1 kết quả -> bấm nút Back.
- **Kết quả lỗi:** Ô tìm kiếm bị làm trắng (clear) hoàn toàn, danh sách hiển thị lại toàn bộ 67 items chưa được lọc.
- **Kỳ vọng:** Giữ nguyên từ khóa "ME35" trong ô input và danh sách vẫn phải đang được lọc.

### ❌ BUG-011: Pagination State Amnesia (Quên trang hiện tại)
- **Mức độ:** 🟡 Medium
- **Mô tả:** Nhấn "Next" qua trang 2 hoặc trang 3 -> bấm xem chi tiết dòng bất kỳ ở đó -> bấm nút Back.
- **Kết quả lỗi:** Khởi động lại ở Trang 1.
- **Kỳ vọng:** Trả người dùng về đúng Trang 2 hoặc Trang 3 mà họ vừa rời đi.

---

## 🛠 Nguyên nhân (Root Cause) & Đề xuất Fix cho Dev Team

Nguyên nhân gốc của chuỗi lỗi BUG-009, 010, 011 là do State của `Search String`, `Page Number`, và `Tab Selection` đang được lưu bằng `useState` (Local React State) thay vì **chiếu lên URL**.

**Giải pháp đề xuất (URL-based State Management):**
Trong Next.js App Router, hãy dùng `useSearchParams` để bắt buộc mọi tương tác state đổi view phải phản ánh lên thanh địa chỉ.
Ví dụ thay vì URL `/dashboard?view=home&filter=all`, hệ thống nên đẩy đủ tham số:
`/?tab=all&search=ME35&page=2`

Khi người dùng nhấn Back cứng từ trình duyệt, Next.js sẽ nạp lại đầy đủ Query Parameters từ URL cũ và hydrate lại trọn vẹn UI state, triệt tiêu dứt điểm "Hội chứng mất trí nhớ" này.
