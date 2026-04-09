# Logical Bug Report — DMS State Management
**Date:** 2026-04-09 (Updated)
**Environment:** Production (`https://dms.giatbh.io.vn`)  
**Type:** UX / Logical (State Management)

---

## 🛑 Tổng Quan
Sau khi Dev Team cập nhật bản fix cho lỗi "mất trí nhớ" UI (State Amnesia), đa số các lỗi đã được xử lý bằng cách đồng bộ trạng thái lên URL (Singe Source of Truth). Tuy nhiên, riêng phần phân trang (Pagination) vẫn đang để sót tham số khi điều hướng, dẫn đến một lỗi rớt trạng thái (Regression).

---

## 🐞 Danh sách Bug Logic (Cập nhật 09/04)

### ✅ BUG-001/002: Auth Bypass (Lỗi vượt rào) — [ĐÃ FIX]
- Trạng thái: Dashboard `/dashboard` không còn mở toang. User bị chặn và redirect về `/login` chính xác.

### ✅ BUG-009: Tab State Amnesia (Mất trạng thái Tab) — [ĐÃ FIX]
- **Kiểm chứng:** Dev đã đẩy state lên URL thành `?filter=all`. 
- **Đường dẫn thực tế:** Khi xem chi tiết, URL mang theo params chính xác: `?back=%2Fdashboard%3Fview%3Dhome%26filter%3Dall`. Khi bấm Back, hệ thống trả về đúng tab ALL.

### ✅ BUG-010: Search State Amnesia (Quên từ khóa tìm kiếm) — [ĐÃ FIX]
- **Kiểm chứng:** Gõ "ME35", URL lập tức nhảy thành `?q=ME35`.
- **Đường dẫn thực tế:** Đường dẫn quay lại (`back` param) có đính kèm đầy đủ `&q=ME35`. Ấn Back là tự động fill lại từ khóa và danh sách giữ nguyên filter.

### ❌ BUG-011: Pagination State Amnesia (Quên trang hiện tại) — [TRƯỢT / CHƯA FIX TRIỆT ĐỂ]
- **Mức độ:** 🟡 Medium
- **Mô tả:** Đang ở trang 2 (URL hiển thị `?page=2`), người dùng click xem chi tiết 1 request. Khi bấm nút Back để quay lại, danh sách bị reset về Trang 1.
- **Bằng chứng (Đường dẫn thực tế gây lỗi):**
  - Tại trang 2, URL chứa: `https://dms.giatbh.io.vn/dashboard?view=home&filter=all&page=2`
  - Nhưng khi click vào xem chi tiết, hệ thống sinh ra URL: 
    `https://dms.giatbh.io.vn/dashboard/request/MM12-CTD-26030211?back=%2Fdashboard%3Fview%3Dhome%26filter%3Dall`
- **Nguyên nhân cốt lõi:** Dev **quên đính kèm tham số `page=2`** vào chuỗi query `back` (`%26page%3D2`). Do URL quay về mất đi thông số biến `page`, hệ thống tự động trả người dùng về default là Trang 1.

---

## 🛠 Đề xuất Fix cụ thể cho Dev Team (BUG-011)

Chỉ cần cập nhật logic component sinh thẻ Link/Button hướng sang chi tiết, đảm bảo hàm bắt Link gom ĐỦ toàn bộ biến `searchParams`:
```typescript
// Lấy toàn bộ query params hiện tại (bao gồm cả page, q, filter, view)
const searchParams = useSearchParams();
const currentQueryString = searchParams.toString(); 

// Link sang trang Detail phải mang theo TẤT CẢ các biến đó trong param `back`
<Link href={`/dashboard/request/${req.id}?back=${encodeURIComponent(`/dashboard?${currentQueryString}`)}`}>
```
Bằng cách này, khi user ấn Back, Next.js sẽ hứng trọn vẹn cả `?page=2` và khôi phục mượt mà danh sách như chưa hề click đi đâu.
