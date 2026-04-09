# Fix Report — Issue 4: Approve Button Wrong Action
**Date:** 2026-04-09 (Batch 3)
**Engineer:** Dev Team
**Source:** `qa_test_results_fpt.md` — commit `bf3cfeb` (Issue 4 added by QA)
**Scope:** Issue 4 — Quick Approve button in list card
**Files modified:**
- `fe/app/(protected)/dashboard/page.tsx`
**Status:** ✅ APPLIED & REBUILT

---

## Bug

### ❌ Issue 4 — Request Tag "Approve" Button Incorrect Navigation (High Severity)

**Reported behavior:** Nút **APPROVE** hiển thị inline trên card request trong list view (Dashboard) khi click **không execute approve action** mà thay vào đó **navigate sang trang detail** — hành vi giống hệt nút DETAIL.

**Root cause tìm trong code:**

Trong `ExpandableItemCard` (line 186-188), cả 2 nút APPROVE và DETAIL đều gọi cùng handler:
```tsx
// TRƯỚC — cả 2 đều navigate sang detail
<button onClick={() => item.Code && goToDetail(item.Code)}>  {/* ← WRONG */}
  <span>check_circle</span>APPROVE
</button>
<button onClick={() => item.Code && goToDetail(item.Code)}>
  DETAIL
</button>
```

Nút APPROVE không bao giờ được wire với approve API — chỉ là placeholder UI dùng tạm `goToDetail`.

---

## Fix — 4 thay đổi

### 1. Thêm `onApprove` prop vào `ExpandableItemCardProps`
```ts
interface ExpandableItemCardProps {
  // ... existing props
  onApprove?: (code: string, module: string) => Promise<void>;
}
```

### 2. Thêm `approving` local state + wire APPROVE button
```tsx
const [approving, setApproving] = useState(false);

// APPROVE button — đúng
<button
  disabled={approving || !onApprove}
  onClick={async (e) => {
    e.stopPropagation(); // prevent card expand
    if (!item.Code || !onApprove || approving) return;
    setApproving(true);
    try { await onApprove(item.Code, item._module || item.TransactionType || ""); }
    finally { setApproving(false); }
  }}
>
  {approving ? <spinner /> : <span>check_circle</span>}
  {approving ? "..." : "APPROVE"}
</button>

// DETAIL button — giữ nguyên
<button onClick={() => item.Code && goToDetail(item.Code)}>
  DETAIL
</button>
```

### 3. Thêm `handleCardApprove` callback trong parent `DashboardPage`
```ts
const { approve: doQuickApprove } = useApproveAction();

const handleCardApprove = useCallback(async (code: string, module: string) => {
  const ok = await doQuickApprove(code, module);
  if (ok) refreshDmsData(); // invalidate context → list updates
}, [doQuickApprove, refreshDmsData]);
```

### 4. Pass `onApprove={handleCardApprove}` vào tất cả 5 card usages
- Desktop all-requests list (1 usage)
- Desktop home overview panel (1 usage)
- Desktop overdue section (1 usage)
- Mobile all-list (2 usages)

---

## Behavior sau fix

| Action | Trước | Sau |
|--------|-------|-----|
| Click APPROVE trên card | Navigate sang detail page | Gọi QuickApprove API, spinner hiện, item biến khỏi list khi done |
| Click DETAIL trên card | Navigate sang detail page | Navigate sang detail page (không đổi) |
| Approve loading state | Không có | Button disable + spinner `...` |
| Post-approve | User phải navigate thủ công | `refreshDmsData()` — list tự cập nhật ngay |

---

*Applied và rebuilt Docker container `bonbon-frontend` tại 2026-04-09 batch 3.*
