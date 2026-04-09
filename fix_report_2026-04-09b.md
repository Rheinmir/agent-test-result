# Fix Report — Approval UX Issues
**Date:** 2026-04-09 (Batch 2)
**Engineer:** Dev Team
**Source:** `qa_test_results_fpt.md` (fpt.dms account testing)
**Scope:** Issue 1 (partial), Issue 2, Issue 3
**File modified:** `fe/app/(protected)/dashboard/request/[id]/page.tsx`
**Status:** ✅ APPLIED & REBUILT

---

## Tóm Tắt

Báo cáo fix tương ứng với `qa_test_results_fpt.md` — kết quả test thực tế từ account `fpt.dms`.

---

## Chi Tiết Fix

### ✅ Issue 2 — Inconsistent Post-Approval UI State (High Severity)

**Root Cause:**
Sau khi approve/reject thành công, code cũ chỉ gọi `router.refresh()` — đây là Next.js App Router method, chỉ re-fetch **Server Components** data. Tuy nhiên danh sách items được lưu trong **client-side context** (`useDmsData` / `DmsDataContext`), không bị invalidate bởi `router.refresh()`.

Kết quả: approve xong → button biến mất (UI detect OK) nhưng item vẫn còn trong list → user không biết action đã thành công hay thất bại.

**Fix — 2 thay đổi:**

1. **Destructure `refresh` từ context:**
```diff
- const { items, allItems: contextAllItems, loading } = useDmsData();
+ const { items, allItems: contextAllItems, loading, refresh: refreshDmsData } = useDmsData();
```

2. **Gọi `refreshDmsData()` sau mỗi approve/reject thành công:**
```diff
  const handleApprove = async () => {
    if (!item) return;
    const ok = await doApprove(item.Code, item._module ?? "");
-   if (ok) router.refresh();
+   if (ok) { router.refresh(); refreshDmsData(); }
  };
  const handleReject = async () => {
    if (!item) return;
    const ok = await doReject(item.Code, item._module ?? "");
-   if (ok) router.refresh();
+   if (ok) { router.refresh(); refreshDmsData(); }
  };
```

`refreshDmsData()` gọi `fetchItems(invalidateCache=true)` → xóa cache, fetch lại DMS API → items context cập nhật → item vừa approve tự động biến khỏi Waiting list.

**Kết quả:** Sau approve → toast success hiện → list tự refresh → item biến mất. User có phản hồi rõ ràng, không còn ambiguous state.

---

### ✅ Issue 3 — Action Buttons Hidden Off-Screen (Medium Severity)

**Root Cause:**
Desktop layout: right panel là `flex-col` với 3 layers: [tab bar] → [content `overflow-y-auto`] → [footer action bar]. Về mặt CSS, footer đã `shrink-0` nên không bị cuộn. Tuy nhiên trên màn hình nhỏ hoặc khi `rightW` hẹp, user không biết có action bar phía dưới — không có visual cue.

**Fix:**
Thêm badge "Action required ↓" trong tab bar header khi `isMyTurnToApprove === true`:

```tsx
{isMyTurnToApprove && (
  <div className="ml-auto shrink-0 flex items-center gap-1.5 px-3 py-1
                  bg-[#10CFC9]/15 border border-[#10CFC9]/30 rounded-full">
    <span className="material-symbols-outlined text-[12px] text-[#006a61]"
          style={{ fontVariationSettings: "'FILL' 1" }}>check_circle</span>
    <span className="text-[10px] font-bold text-[#006a61] uppercase
                     tracking-widest whitespace-nowrap">Action required ↓</span>
  </div>
)}
```

Badge này xuất hiện ngay đầu màn hình (trong tab bar, luôn visible, không đòi hỏi scroll). Arrow `↓` chỉ hướng xuống footer chứa Approve/Reject buttons.

**Mobile:** Không cần sửa — buttons đã `fixed bottom-0` (luôn hiển thị overlay).

---

### ⚠️ Issue 1 — Detail Panel Sync (Partial Fix via Issue 2)

**Phân tích:**
Issue gốc: Click item mới trong list → detail panel hiển thị item cũ.

Trong `request/[id]/page.tsx`, khi click item trong left panel (line 656-661):
```tsx
router.push(`/dashboard/request/${encodeURIComponent(row.Code)}...`)
```
→ URL thay đổi → `requestCode` thay đổi → `item = items.find(i => i.Code === requestCode)` re-compute.

Nếu `items` context stale (chưa include item mới clicked vì cache cũ), `item` có thể trả về `null` → fallback `contextAllItems` → nếu `allItems` cũng stale → panel không update.

**Partial fix từ Issue 2:** `refreshDmsData()` sau approve giúp context luôn fresh, giảm thiểu stale state. Nhưng không hoàn toàn fix issue click item khi **chưa approve** gì.

**Root cause thực sự cần investigate thêm:** Có thể là React `memo` hoặc `useCallback` với stale closure, hoặc `router.push` không trigger re-render vì Next.js caches route. Cần reproduce với DevTools → để lại cho QA verify sau rebuild này.

---

## Scope Không Thay Đổi

| Item | Trạng thái |
|------|-----------|
| BUG-012 (Login mobile) | ⏭️ Deferred |
| Issue 1 (Detail panel sync) | 🔍 Partial fix — cần QA verify lại |
| MM13 approve failed | ℹ️ Likely backend/permission issue, không phải FE |
| ME35 approve ambiguous | ℹ️ Fixed bởi Issue 2 (refresh context) |

---

*Applied và rebuilt Docker container `bonbon-frontend` tại 2026-04-09 batch 2.*
