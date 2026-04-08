# Fix Report — dms.giatbh.io.vn
**Date:** 2026-04-08  
**In response to:** `qa_report_2026-04-08.md`  
**Fixed by:** Dev team (giatbh)  
**Branch:** `giatbh` → `gitlab.com/knight.dragonwar/bonbon-ai-frontend`

---

## Summary

| Bug ID | Severity | Title | Status |
|--------|----------|-------|--------|
| BUG-001 | 🔴 Critical | Login broken — `connectionString null` | ⏳ Infrastructure (BE env config) |
| BUG-002 | 🔴 Critical | Auth bypass — `/dashboard` accessible without login | ⏳ Pending (middleware enforcement) |
| BUG-003 | 🟠 High | Hardcoded `localhost:3000` in metadata/manifest | ✅ Fixed |
| BUG-004 | 🟠 High | Placeholder data (`đ`) in description fields | ℹ️ Data from DMS source — not fixable at FE |
| BUG-005 | 🟡 Medium | Sidebar icons — no text labels | ℹ️ By design (auto-collapses, native `title` tooltip on hover) |
| BUG-006 | 🟡 Medium | Vercel Analytics script 404 | ✅ Fixed |
| BUG-007 | 🟡 Medium | No search/filter in "All Requests" list | ✅ Fixed |
| BUG-008 | 🟡 Medium | No pagination — list too long | ✅ Fixed |

---

## Fixed Bugs (Details)

### ✅ BUG-003 — Hardcoded localhost:3000

**Root cause:** `NEXT_PUBLIC_APP_URL` in `.env` was set to `http://localhost:3000`, causing `site.webmanifest` and OG metadata to reference localhost on production.

**Fix:** Updated `NEXT_PUBLIC_APP_URL=https://dms.giatbh.io.vn` in environment config.

**Verify:** Open DevTools → Network → filter `site.webmanifest`. URL should reference `https://dms.giatbh.io.vn`, not localhost.

---

### ✅ BUG-006 — Vercel Analytics 404

**Root cause:** `@vercel/analytics` script (`/_vercel/insights/script.js`) only works on Vercel-hosted apps. Site is deployed via Cloudflare Tunnel, not Vercel.

**Fix:** Removed `<Analytics />` component from `app/layout.tsx`.

**File changed:** `fe/app/layout.tsx`

**Verify:** Open DevTools → Network → no 404 requests to `/_vercel/insights/script.js`.

---

### ✅ BUG-007 — No Search in "All Requests" list

**Root cause:** No search input existed on the All Requests list.

**Fix:** Added `allSearch` state + search input box to:
- **Desktop** (home view, "All Requests" section): Search bar in list header, filters by Code, Job name, Reason, Partner, Requestor, Contract No., Department.
- **Mobile** (all-list view): Search bar above item list.

Clears automatically when switching filters.

**File changed:** `fe/app/(protected)/dashboard/page.tsx`

**Verify:**
1. Go to dashboard → switch to "ALL" tab
2. Type a contract code (e.g., `ME35`) in the search bar
3. List should filter in real-time

---

### ✅ BUG-008 — No Pagination for All Requests

**Root cause:** All 67 items were rendered at once, making the list excessively long and burying the charts/stats bento grid below.

**Fix:** Added client-side pagination (10 items/page) to the desktop All Requests list:
- Shows `Page X / Y` indicator
- **Prev** / **Next** buttons
- Auto-resets to page 1 when filter or search term changes

**File changed:** `fe/app/(protected)/dashboard/page.tsx`

**Verify:**
1. Go to dashboard → switch to "ALL" (67 items)
2. Only 10 items show at a time
3. "Page 1 / 7" counter and Prev/Next buttons appear below list
4. Charts/bento grid is now visible without excessive scrolling

---

## Not Fixed (Explanation)

### ⏳ BUG-001 — Login broken (`connectionString null`)

This is a **backend infrastructure issue** — the Go backend is missing a `DATABASE_URL` / `CONNECTION_STRING` environment variable in its deployment environment. This cannot be fixed from the frontend codebase.

**Action needed:** Check backend `.env` or deployment secrets and ensure `DB_CONNECTION` / `CONNECTION_STRING` is correctly injected.

---

### ⏳ BUG-002 — Auth bypass on `/dashboard`

Next.js middleware (`middleware.ts`) needs to be enforced to redirect unauthenticated users from `/dashboard` routes to `/login`. This requires backend auth token validation at the edge.

**Action needed:** Implement `middleware.ts` in root of `fe/` to check for auth cookie/token and redirect if missing.

---

### ℹ️ BUG-004 — Placeholder data (`đ` in description)

The description field containing only `đ` comes directly from the DMS API (dms.coteccons.vn). The frontend renders whatever the API returns — no data transformation is applied.

**Action needed:** Data quality fix required on DMS source system side.

---

### ℹ️ BUG-005 — Sidebar icon-only (no labels)

The sidebar is designed to auto-collapse to icon-only after 2 seconds on first visit (UX optimization to maximize content area). When expanded, full text labels are shown. Native browser `title` tooltips show on hover in collapsed state.

If tester prefers labels always visible, this can be changed as a UX preference — not treated as a bug.

---

## Test Checklist for Retesting

- [ ] **BUG-003:** `site.webmanifest` URL = `https://dms.giatbh.io.vn/site.webmanifest` (not localhost)
- [ ] **BUG-006:** No 404 for `/_vercel/insights/script.js` in Network tab
- [ ] **BUG-007 (desktop):** Search box appears in "All Requests" list header, filters in real-time
- [ ] **BUG-007 (mobile):** Search box appears above request list on mobile all-list view
- [ ] **BUG-008:** Desktop All Requests shows max 10 items, Prev/Next pagination controls visible

---

*Report generated: 2026-04-08*
