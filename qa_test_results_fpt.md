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
