**Bug ID**: BUG-2026-089
**Title**: 24-Hour Reassignment Lock is Not Enforced via Admin Dashboard API

**Environment**: Staging v2.4.0-rc1
**Severity**: High
**Priority**: High

**Description**:
After successfully reassigning Route RT-105, a manager is able to reassign the same route again immediately. The system fails to enforce the 24-hour lock constraint, allowing back-to-back modifications.

**Steps to Reproduce**:
1. Log into the Admin Dashboard as a Manager.
2. Navigate to Campaign 05 and select Route RT-105.
3. Reassign RT-105 from Agent Alex to Agent Chloe. (Reassignment succeeds).
4. Refresh the page immediately.
5. Attempt to reassign Route RT-105 from Agent Chloe to Agent John.

**Expected Result**:
The system should block the second reassignment, display an error message ("Route locked for 24 hours"), and disable the assignment UI action.

**Actual Result**:
The second reassignment is processed successfully without errors. The route is updated to Agent John immediately.

**Visual Proof / Logs**:
- Attached: [screenshot_of_double_reassignment.png]
- API Response snippet allowing the bypass:
  { "status": "success", "message": "Route reassigned successfully", "lock_until": null }