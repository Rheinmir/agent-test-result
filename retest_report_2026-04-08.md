# 🔁 Retest Report — dms.giatbh.io.vn
**Ngày retest:** 2026-04-08  
**Tester:** Antigravity AI QA Agent  
**Lý do retest:** Dev team đã deploy bản fix mới lên production  
**Dựa trên:** `fix_report_2026-04-08.md`

---

## 📊 Tổng hợp kết quả

| Bug ID | Mô tả | Kết quả |
|--------|-------|---------|
| BUG-003 | localhost:3000 hardcoded in manifest | ✅ **PASS** |
| BUG-006 | Vercel Analytics script 404 | ✅ **PASS** |
| BUG-007 | Không có search box trong All Requests | ✅ **PASS** |
| BUG-008 | Không có pagination trong All Requests | ⚠️ **PARTIAL** |

---

## Chi tiết từng bug

### ✅ BUG-003 — localhost:3000 hardcoded FIXED
**Kết quả:** PASS  
**Bằng chứng:** `site.webmanifest` kiểm tra qua `https://dms.giatbh.io.vn/site.webmanifest` — tất cả `src` và `start_url` đã dùng relative paths (`./favicons/...`) thay vì `localhost:3000`.  
**Verify:** Không còn lỗi `net::ERR_CONNECTION_REFUSED` từ localhost trong Network tab.

---

### ✅ BUG-006 — Vercel Analytics 404 FIXED
**Kết quả:** PASS  
**Bằng chứng:** Network log không còn request nào đến `/_vercel/insights/script.js`. Script đã được thay bằng `static.cloudflareinsights.com/beacon.min.js` (Cloudflare Analytics).  
**Verify:** Không còn 404 error trong Network tab.

---

### ✅ BUG-007 — Search box đã được thêm FIXED
**Kết quả:** PASS  
**Bằng chứng (từ screenshot):** Có ô search với placeholder **"Search code, name, partner..."** ở góc trên phải của danh sách All Requests.  
**Test thực tế:** Gõ "NetApp" → danh sách filter còn đúng 2 items liên quan đến NetApp. Chức năng hoạt động đúng.

---

### ⚠️ BUG-008 — Pagination PARTIAL
**Kết quả:** PARTIAL — Cần điều tra thêm  
**Bằng chứng từ screenshot:** Danh sách hiển thị "67 items" và vẫn thấy nhiều items trong một trang — KHÔNG thấy rõ pagination "Page 1/7" hay Prev/Next button trong screenshot hiện tại.  
**Ghi chú của agent:** Agent báo cáo pagination hoạt động (Page 1/7, 10 items/page) nhưng cần scroll xuống hoặc test thêm để xác nhận pagination control hiển thị đúng vị trí.  
**Khuyến nghị:** Cần retest thủ công — scroll xuống cuối danh sách để tìm pagination controls.

---

## 🆕 Phát hiện mới trong lần retest này

### Thay đổi data đáng chú ý:
- **PENDING tăng từ 0 lên 5:** Dashboard lúc trước hiện "0 pending requests", lần này hiện **"5 pending requests"** — data được sync mới.
- **Có thêm các badge mới:** `OVERDUE 4D`, `OVERDUE 3D`, `CHỜ DUYỆT`, `CHỜ ĐỒNG`, `RETURN` — hệ thống đang tracking nhiều trạng thái hơn so với lần test đầu.
- **Giá trị thay đổi:** ME35-CTD-26031051 (Workday HCM) từ 15.530.489.640₫ → 14.380.083.000₫

---

## ✅ Danh sách bug đã đóng

| Bug | Trạng thái cuối |
|-----|----------------|
| BUG-003 localhost:3000 | ✅ CLOSED |
| BUG-006 Vercel 404 | ✅ CLOSED |
| BUG-007 Search box | ✅ CLOSED |

## ⏳ Bug cần theo dõi tiếp

| Bug | Trạng thái | Hành động tiếp theo |
|-----|------------|-------------------|
| BUG-008 Pagination | ⚠️ NEEDS VERIFY | Retest thủ công — scroll xuống cuối list xác nhận Prev/Next |
| BUG-001 Login broken | ⏳ OPEN | Backend team fix env var `CONNECTION_STRING` |
| BUG-002 Auth bypass | ⏳ OPEN | Implement `middleware.ts` trong Next.js |

---

*Report generated: 2026-04-08 by Antigravity AI QA Agent*
