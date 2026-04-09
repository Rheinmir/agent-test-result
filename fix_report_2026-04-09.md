# Fix Report — State Amnesia Bugs
**Date:** 2026-04-09  
**Engineer:** Dev Team  
**Scope:** BUG-011, BUG-013a, BUG-013b  
**File modified:** `fe/app/(protected)/dashboard/page.tsx`  
**Status:** ✅ APPLIED & REBUILT

---

## Tóm Tắt

Báo cáo fix tương ứng với `logical_bug_report_2026-04-08.md` và `mobile_qa_report_2026-04-09.md`.  
Tất cả 3 bug thuộc nhóm **State Amnesia** — trạng thái UI được lưu trong React local state nhưng **không được đồng bộ lên URL**, khiến navigation back/forward mất trạng thái.

> **BUG-012 (Login mobile):** Bỏ qua theo yêu cầu, sẽ xử lý riêng.

---

## Chi Tiết Fix

### ✅ BUG-011 — Pagination State Amnesia (Desktop)
**Mức độ:** Medium  
**Root Cause:** `desktopPage` là pure React state, không ghi vào URL. Khi `goToDetail` build `backUrl` từ `searchParams`, param `page` không tồn tại trong URL nên bị mất.

**Cách fix (3 thay đổi):**

1. **Init từ URL** — `desktopPage` đọc `?page=` khi component mount:
   ```ts
   // Trước
   const [desktopPage, setDesktopPage] = useState(0);
   // Sau
   const [desktopPage, setDesktopPage] = useState(() => {
     const p = parseInt(searchParams.get("page") || "0", 10);
     return isNaN(p) || p < 0 ? 0 : p;
   });
   ```

2. **Helper `changeDesktopPage`** — mỗi lần chuyển trang ghi luôn vào URL:
   ```ts
   const changeDesktopPage = (p: number) => {
     setDesktopPage(p);
     const params = new URLSearchParams(searchParams.toString());
     p === 0 ? params.delete("page") : params.set("page", String(p));
     router.replace(`/dashboard?${params.toString()}`, { scroll: false });
   };
   ```

3. **Sync trong `useEffect`** — khi URL thay đổi (back/forward), đọc lại `?page=`:
   ```ts
   const pg = parseInt(searchParams.get("page") || "0", 10);
   setDesktopPage(isNaN(pg) || pg < 0 ? 0 : pg);
   ```

**Kết quả:** URL tại trang 2 là `?view=home&filter=all&page=2`. Khi click vào detail, `goToDetail` build `back=...%26page%3D2`. Bấm Back → trả về trang 2 chính xác.

---

### ✅ BUG-013a — Search State Amnesia (Mobile & Desktop)
**Mức độ:** Medium  
**Root Cause:** `allSearch` init cứng `""`, không đọc `?q=`. Không có cơ chế push search term lên URL khi user gõ.

**Cách fix (3 thay đổi):**

1. **Init từ URL:**
   ```ts
   // Trước
   const [allSearch, setAllSearch] = useState("");
   // Sau
   const [allSearch, setAllSearch] = useState(searchParams.get("q") || "");
   ```

2. **Helper `changeSearch`** — gõ search → ghi `?q=` vào URL, xóa `?page=`:
   ```ts
   const changeSearch = (val: string) => {
     setAllSearch(val);
     setDesktopPage(0);
     const params = new URLSearchParams(searchParams.toString());
     val ? params.set("q", val) : params.delete("q");
     params.delete("page");
     router.replace(`/dashboard?${params.toString()}`, { scroll: false });
   };
   ```

3. **Thay `setAllSearch` → `changeSearch`** ở cả 4 điểm: desktop input `onChange`, desktop clear button, mobile input `onChange`, mobile clear button.

4. **Sync trong `useEffect`:** `setAllSearch(searchParams.get("q") || "")`.

**Kết quả:** Search "ME35" → URL `?q=ME35`. Vào detail → back → ô search tự fill lại "ME35" và danh sách filter đúng. Hoạt động đồng nhất trên cả Desktop và Mobile.

---

### ✅ BUG-013b — Mobile Tab Filter State Amnesia
**Mức độ:** Medium  
**Root Cause:** `allViewMode` (mobile pill toggle) init cứng `"pending"`, không đọc URL. Desktop đã có `desktopFilter` + `changeDesktopFilter` pattern nhưng mobile không áp dụng tương tự.

**Cách fix (3 thay đổi):**

1. **Init từ URL** — đọc `?filter=` khi mount:
   ```ts
   // Trước
   const [allViewMode, setAllViewMode] = useState<"all" | "pending">("pending");
   // Sau
   const [allViewMode, setAllViewMode] = useState<"all" | "pending">(
     (searchParams.get("filter") as "all" | "pending") === "all" ? "all" : "pending"
   );
   ```

2. **Helper `changeMobileFilter`** — tap pill → ghi `?filter=` vào URL:
   ```ts
   const changeMobileFilter = (f: "all" | "pending") => {
     setAllViewMode(f);
     const params = new URLSearchParams(searchParams.toString());
     params.set("filter", f);
     router.replace(`/dashboard?${params.toString()}`, { scroll: false });
   };
   ```

3. **Thay `setAllViewMode` → `changeMobileFilter`** ở 2 pill buttons.

4. **Sync trong `useEffect`:** `if (f === "all" || f === "pending") setAllViewMode(f)`.

**Kết quả:** Tap "ALL" → URL `?filter=all`. Vào detail → back → pill toggle giữ đúng tab "ALL", danh sách hiển thị đúng mode. Không còn nhảy về "PENDING" mặc định.

---

## Kiến Trúc Pattern — Single Source of Truth

Tất cả 3 fix đều áp dụng cùng một pattern đã có sẵn trên Desktop (`desktopFilter` / `changeDesktopFilter`):

```
URL ?param=value  ←──────────────────────────────────┐
      │                                               │
      ▼ init / useEffect sync                         │ router.replace (no history push)
React State                                           │
      │                                               │
      ▼ user action (gõ / tap / click)                │
Helper fn (changeSearch / changeMobileFilter / changeDesktopPage) ──►
```

Mọi trạng thái UI quan trọng đều tồn tại trong URL → `goToDetail` đính kèm đủ params → back navigation khôi phục chính xác.

---

## Scope Không Thay Đổi

| Item | Trạng thái |
|------|-----------|
| BUG-012 (Login mobile) | ⏭️ Bỏ qua theo yêu cầu |
| Hamburger menu không có action (mobile) | ⏭️ Ghi nhận, chưa xử lý |
| `allStatusFilter` dropdown | ✅ Không bị bug — chỉ filter local, không cần URL |

---

*Fix committed và rebuilt Docker container `bonbon-frontend` tại `2026-04-09`.*
