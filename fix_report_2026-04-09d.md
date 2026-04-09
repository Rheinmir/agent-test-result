# Fix Report — Round 2 QA Issues (Batch 4)
**Date:** 2026-04-09 (Batch 4)
**Engineer:** Dev Team
**Source:** `qa_test_results_fpt.md` — Round 2 Comprehensive Interactivity Testing
**Scope:** Logout, Generate Report button, DETAIL event bug
**Files modified:**
- `fe/components/dms/dms-sidebar.tsx`
- `fe/app/(protected)/dashboard/page.tsx`
**Status:** ✅ APPLIED & REBUILT

---

## Fix 1 — Logout Without Confirmation (High Severity)

**Bug:** Click avatar/tên user ở bottom-left sidebar → logout ngay lập tức, không có dialog confirmation.

**Root Cause:** `onClick={logout}` gán thẳng lên div avatar container.

**Fix:** Tách thành `LogoutSection` sub-component với 2-step confirm flow:
1. Click avatar → hiện mini panel "Sign out?" với 2 nút
2. `Cancel` — đóng panel
3. `Sign Out` — confirm thực sự gọi `logout()`
4. Auto-dismiss sau 4s nếu không chọn

```
[Click avatar] → [Sign out? | Cancel | Sign Out] → [4s auto-dismiss nếu ko action]
```

---

## Fix 2 — "Generate Report" Button Dead (Medium Severity)

**Bug:** Button "Generate Report" hoàn toàn không responsive — không trigger UI, không có network call.

**Action:** Comment out button trong JSX. Sẽ implement sau khi có feature report thực sự.

```diff
- <button className="...">Generate Report</button>
+ {/* Generate Report button — hidden until implemented */}
```

---

## Fix 3 — DETAIL Button Event Propagation (Medium Severity)

**Bug:** Clicking DETAIL button trong expanded card còn trigger card toggle (expand/collapse) đồng thời với navigate → user thấy card thu lại + navigate cùng lúc, cảm giác "unresponsive".

**Root Cause:** DETAIL buttons thiếu `e.stopPropagation()`. Card header là `<button>` bao ngoài, click DETAIL propagate lên trigger card toggle.

**Fix:**
```diff
- <button onClick={() => item.Code && goToDetail(item.Code)}>DETAIL</button>
+ <button onClick={(e) => { e.stopPropagation(); item.Code && goToDetail(item.Code); }}>DETAIL</button>
```
Áp dụng cho cả 2 variants: `isUserItem=true` (2-col layout) và `isUserItem=false` (full-width).

---

## Trace — MM13/ME35 "No Approve Buttons" Clarification

**QA report:** Buttons APPROVE/DETAIL không respond trên MM13/ME35 sau khi đã approve.

**Kết quả trace:** Đây là **correct behavior**, không phải bug:
- `checkIsMyTurn(item, user, waitingItems)` trả về `false` khi item không còn trong waiting list (đã approve/reject)
- `isMyTurn=false` → `isUserItem=false` → chỉ render DETAIL (không có APPROVE)
- DETAIL vẫn hoạt động — nhưng bị nhầm lẫn bởi event propagation bug (Fix 3)

Sau Fix 3, DETAIL sẽ navigate clean mà không có card toggle đồng thời.

---

## Items Deferred

| Item | Status |
|------|--------|
| Floating status tag "Tải..." stuck | 🔍 Cần investigate riêng |
| API latency 25-29s (All Requests tab) | 🔍 Cần trace backend/proxy caching |

---
*Applied và rebuilt 2026-04-09 batch 4. Commit: `19ce0ed`*
