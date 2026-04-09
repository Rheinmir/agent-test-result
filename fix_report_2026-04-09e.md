# Fix Report — Round 3: Detail Panel Race Condition (Batch 5)
**Date:** 2026-04-09 (Batch 5)
**Source:** `qa_test_results_fpt.md` — Round 3 Detail Panel Sync Stress Test
**Scope:** Race condition in attachment fetch (UI Amnesia bug)
**Files modified:**
- `fe/app/(protected)/dashboard/page.tsx`
- `fe/app/(protected)/dashboard/request/[id]/page.tsx`
**Status:** ✅ APPLIED & REBUILT

---

## Bug — Detail Panel "UI Amnesia" Race Condition (High Severity)

**QA stress test:** 45 rapid clicks → reproduced 2 times. API latency 1.5s–51s.

**Symptom:** Left list highlights new item + URL updates, but right-hand Detail Panel stays frozen on stale data from a previous click.

**Root Cause:** `useEffect(() => { fetch(...).then(d => setState(d)) }, [item?.Code])` — khi item thay đổi, React unmount/remount effect nhưng promise cũ vẫn in-flight. Nếu stale promise resolve SAU promise mới, nó gọi `setState` override state đúng → panel bị stuck.

Với DMS API latency lên đến 51s, stale promise có thể resolve rất muộn và "thắng" race.

---

## Fix — 2 Files, cùng pattern

### 1. `dashboard/page.tsx` — `detailItem` attachment fetch

```diff
  useEffect(() => {
      if (!detailItem || !dmsToken) { setAttachments([]); return; }
      setAttachments([]); setAttachmentsLoading(true); setPreview({ type: "idle" });
+     let cancelled = false;
+     const controller = new AbortController();
      fetch("/api/dms-proxy", {
          ...
+         signal: controller.signal,
      })
          .then(async (r) => { ... })
          .catch(() => [])
-         .then((list) => { setAttachments(list); setAttachmentsLoading(false); });
+         .then((list) => { if (!cancelled) { setAttachments(list); setAttachmentsLoading(false); } });
+     return () => { cancelled = true; controller.abort(); };
  }, [detailItem?.Code, dmsToken]);
```

### 2. `request/[id]/page.tsx` — item attachment fetch

```diff
  useEffect(() => {
      if (!item) return;
+     let cancelled = false;
      callDms(buildFileRefUrl(item))
-         .then((d) => setAttachments(d.Result || []))
-         .catch(() => setAttachments([]))
-         .finally(() => setAttachLoading(false));
+         .then((d) => { if (!cancelled) setAttachments(d.Result || []); })
+         .catch(() => { if (!cancelled) setAttachments([]); })
+         .finally(() => { if (!cancelled) setAttachLoading(false); });
+     return () => { cancelled = true; };
  }, [item?.Code]);
```

### Pattern: React useEffect cleanup = automatic abort

```
item click N   → effect runs → fetch N starts
item click N+1 → cleanup runs → cancelled = true, controller.abort()
               → effect runs → fetch N+1 starts
fetch N resolves (slow) → if (cancelled) return → ❌ IGNORED
fetch N+1 resolves      → if (!cancelled) setState → ✅ COMMITTED
```

Panel luôn hiển thị data của item cuối cùng được click, bất kể thứ tự resolve của API.

---

## Note về API Latency 25-51s

Đây là vấn đề backend/DMS upstream, không phải FE. Race condition fix giúp UX không bị stuck, nhưng latency vẫn còn. Cần investigate:
- DMS proxy cache hit rate
- `GetContractNewDetailPageOnGoing` endpoint performance
- Pagination strategy cho All Requests tab

