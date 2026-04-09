# DMS QA Test Log
**Date:** 2026-04-09
**Account Tested:** `fpt.dms`
**Target Environment:** `https://dms.giatbh.io.vn`
**Objective:** Systematically test the approval functionality, specifically evaluating 'one approved per contract type', noting down UI/UX problems, functional bugs, and capturing console observations.

## 1. Testing Summary: Approval by Contract Type
The following contract types were tested for their approval workflows:

| Contract Type | Test Result | Request Code | Notes |
| :--- | :--- | :--- | :--- |
| **MM12** | ✅ **Success** | `MM12-CTC-23080040` | Successfully approved. The item immediately disappeared from the pending/overdue triage list as expected. |
| **MM13** | ⚠️ **Failed/Ambiguous** | `MM13-CTD-25070010` | Attempted approval. The "APPROVE" button was clicked, but the item remained in the pending list without updating status. |
| **ME35** | ⚠️ **Failed/Ambiguous** | `ME35-CTC-23100314` | The "APPROVE" button was clicked and subsequently disappeared, however, the request remained in the pending/actionable list without progressing based on history. |
| **ME53N** | ℹ️ *Not tested* | N/A | No pending items matching this type were available for the current user's approval level. |
| **FI47** | ℹ️ *Not tested* | N/A | No pending items matching this type were actionable for the current user's approval level. |

## 2. Functional & UI/UX Issues Identified

### **Issue 1: Detail Panel Synchronization Error**  *(High Severity)*
**Problem:** Clicking a new request from the left-hand navigation list frequently fails to update the right-hand detail panel. 
**Behavior:** The list item highlights as if selected, but the detail panel remains stuck displaying the previously viewed request.
**Workaround:** Requires the user to deliberately re-click the list item multiple times to force the panel to sync. This is highly confusing.

### **Issue 2: Inconsistent Post-Approval UI State** *(High Severity)*
**Problem:** For certain contract types (e.g., ME35), clicking "APPROVE" removes the action buttons but does not finalize the state in the UI.
**Behavior:** The request is not cleared from the current user's active list, nor does the "Audit Trail" / History accurately reflect the real-time processing outcome visually, leaving the user unsure if the action succeeded.

### **Issue 3: Action Buttons Placement Truncation** *(Medium Severity)*
**Problem:** The "APPROVE" and "REJECT" buttons are positioned at the bottom of the right-hand detail panel.
**Behavior:** Depending on screen size and request detail length, these buttons are often pushed off-screen. Users might assume they lack permission if they do not explicitly scroll down past all details.

### **Issue 4: Request Tag "Approve" Button Incorrect Navigation** *(High Severity)*
**Problem:** The inline "Approve" button located directly on a request tag (in the list view) does not trigger the approval action.
**Behavior:** Instead of approving the request, clicking this button unexpectedly navigates the user to the request detail view. The button acts merely as a navigation link rather than executing its intended action.

## 3. Console Logs & Technical Observations
- **Network Requests:** The system correctly pulls data endpoints like `/api/ContractSign/GetApprovedTaskPageOnGoing/...` without structural failures.
- **JavaScript Stability:** No complete browser crashes or fatal JavaScript exceptions were observed breaking the session.
- **Root Cause Indication:** The issues with "State Inconsistency" and "Detail Panel Sync" appear to stem from Frontend state management logic (possibly SPA component reactivity rules failing to re-render or listen to model updates), causing delayed or dropped prop updates upon selection or submission.

---
**Conclusion:** The platform's approval logic is largely functional but suffers from significant UI synchronization issues that may lead to user confusion or accidental double-actions.

## 4. Round 2: Comprehensive Interactivity Testing (All Buttons)
A second testing pass aggressively interacted with every visible button across the Main Dashboard, Triage Lists, and Detail Pains. 

### **New Critical Issues Found**
*   **Logout Bug (High Severity):** Clicking the user profile avatar/name in the bottom-left sidebar instantly logs the user out without any confirmation dialogue. 
*   **Dead Action Buttons (Medium Severity):** The "Generate Report" (Báo cáo) button on the dashboard is completely unresponsive; it triggers no UI change and no terminal/network errors. 
*   **Disabled/Missing Approval Buttons:** In many edge cases directly within the middle-pane list (like expanding MM13 / ME35 cards), both `APPROVE` and `DETAIL` inline buttons fail to respond, or the APPROVE button acts as a duplicate DETAIL button without performing approvals.
*   **Floating Status Tag Ambiguity:** The floating status summary components (bottom right) frequently get stuck for extended periods displaying a loading state ("Tải..."). Clicking them fails to consistently navigate the user to that category.
*   **High Latency Bottlenecks:** API responses when viewing the "All Requests" tab frequently took between **25s to 29s** to resolve, causing the main viewing area to remain blank and feel broken before data suddenly populated.

## 5. Round 3: Detail Panel Sync Stress Test (Amnesia Bug)
A targeted stress-test was performed on the middle-pane request list to aggressively validate the "UI Amnesia" behavior (Detail Panel freezing/failing to update). Roughly **45 distinct requests** were clicked in rapid succession.

### **Test Results & Root Cause Analysis**
*   **100% Success Rate with Delays:** If a user clicks a request and waits sequentially for the right-hand Detail Panel to fully load (approx 2-3 seconds) before clicking the next item, the panel updates correctly every time. 
*   **Amnesia Reproduced via Rapid Clicks (Race Condition):** The synchronization bug was successfully reproduced twice during the stress test. It occurs specifically when clicking rapidly between 5-8 requests without waiting, or immediately interacting after a heavy scroll operation.
*   **Symptom:** The left list correctly highlights the newly clicked item and the browser URL changes, but the right-hand **Detail Panel remains permanently stuck** on a previously fetched request's data. 
*   **Root Cause Hypothesis:** Console logs showed individual detail-fetch API endpoints (`/api/ContractSign/...`, `/api/File/...`) taking anywhere from **1.5s to 20s+ (with one fetching taking 51s)**. The Frontend is failing to cancel outdated asynchronous API requests upon new item clicks, resulting in a race condition where delayed data from an older click permanently overrides or locks the newly intended view state. Additionally, there is a lack of a clear centralized "Loading" skeleton over the entire component to indicate that new data is being fetched.
