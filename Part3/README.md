# Oppizi Manual Testing Assignment: Route Conflict Detection & Reassignment Auditing
# Test Suite: Route Conflict Detection and Reassignment Auditing

This document provides a comprehensive manual test suite for the new Route Reassignment feature in the Oppizi admin dashboard.

## 1. Test Design (Scenarios & Test Cases)

### Functional Scenarios (Happy Path)
* **TC-FUN-01**: Reassign an outdoor route to an available agent with matching location types before the campaign starts.
* **TC-FUN-02**: Reassign an indoor route to an available agent with matching location types before the campaign starts.

### Negative Scenarios
* **TC-NEG-01**: Attempt to reassign a route after the campaign start date has already passed.
* **TC-NEG-02**: Attempt to reassign an outdoor route to an agent who already has an overlapping outdoor route at the same time.
* **TC-NEG-03**: Attempt to reassign an indoor route to an agent who has an overlapping outdoor route at the same time.
* **TC-NEG-04**: Attempt to reassign a route to an agent whose location type profile does not match (e.g., assigning an indoor route to an outdoor-only agent).
* **TC-NEG-05**: Attempt to reassign a route that was already reassigned less than 24 hours ago (Lockout violation).

### Edge Case Scenarios
* **TC-EDGE-01**: Reassign a route to an agent whose existing route ends exactly when the new route starts (0-minute gap).
* **TC-EDGE-02**: Attempt to reassign a route exactly 1 minute after the campaign has officially started.
* **TC-EDGE-03**: Attempt to reassign a route that was locked, exactly 24 hours and 1 minute after its lockout period started.

### Permission Scenarios
* **TC-AUTH-01**: Verify a user with "Admin/Manager" role can successfully execute a valid route reassignment.
* **TC-AUTH-02**: Attempt to execute a route reassignment using a standard "Agent" or "Read-Only" account role.

## 2. Test Data Table 


| Case_ID | ROUTE_ID | CAMPAING | AGENT | ROUTE_TYPE | CAMPAIGN_STARTS | AGENT_PROFILE | AGENT_SCHEDULE | LAST_REASSIGNED | EXECUTION_TIME | Expected Result |
|---------|----------|----------|-------|------------|-----------------|---------------|----------------|-----------------|----------------|-----------------|
| TC-FUN-01 | ROUTE-01 | Campaign 01 | Agent 01 | Outdoor | 02/06/2026 09:00:00 | Outdoor | No routes | 15/05/2026 12:00 | 06/01/2026 08:00 | Reassignment successful. |
| TC-FUN-02 | ROUTE-02 | Campaign 02 | Agent 02 | Indoor | 02/06/2026 09:00:00 | Both | No routes | 15/05/2026 12:00 | 06/01/2026 08:00 | Reassignment successful. |
| TC-NEG-01 | ROUTE-03 | Campaign 03 | Agent 03 | Outdoor | 31/05/2026 08:00 | Outdoor | No routes | 15/05/2026 12:00 | 06/01/2026 08:00 | Error: Campaign already started. |
| TC-NEG-02 | ROUTE-04 | Campaign 04 | Agent 04 | Outdoor | 02/06/2026 08:00 | Outdoor | Outdoor (08:00-12:00) | 15/05/2026 12:00 | 06/01/2026 08:00 | Error: Outdoor overlap. |
| TC-NEG-03 | ROUTE-05 | Campaign 05 | Agent 05 | Indoor | 02/06/2026 08:00 | Both | Outdoor (08:00-12:00) | 15/05/2026 12:00 | 06/01/2026 08:00 | Error: Schedule overlap. |
| TC-NEG-04 | ROUTE-06 | Campaign 06 | Agent 06 | Indoor | 02/06/2026 09:00 | Outdoor | No routes | 15/05/2026 12:00 | 06/01/2026 08:00 | Error: Profile mismatch. |
| TC-NEG-05 | ROUTE-07 | Campaign 07 | Agent 07 | Outdoor | 05/06/2026 09:00 | Outdoor | No routes | 31/05/2026 15:00 | 06/01/2026 08:00 | Error: Lockout active. |
| TC-EDGE-01 | ROUTE-08 | Campaign 08 | Agent 08 | Outdoor | 02/06/2026 12:00 | Outdoor | Outdoor (08:00-12:00) | 15/05/2026 12:00 | 06/01/2026 08:00 | Success: 0 min gap. |
| TC-EDGE-02 | ROUTE-09 | Campaign 09 | Agent 09 | Outdoor | 01/06/2026 08:00 | Outdoor | No routes | 15/05/2026 12:00 | 06/01/2026 08:01 | Error: Started +1 min. |
| TC-EDGE-03 | ROUTE-10 | Campaign 10 | Agent 10 | Outdoor | 05/06/2026 09:00 | Outdoor | No routes | 31/05/2026 07:59 | 06/01/2026 08:00 | Success: Lockout +24h 1m. |




--

## 3. Audit and Email Verification Steps

### Audit Log Verification (via Admin UI / API)
1. Navigate to the **Audit Logs** section in the Admin Dashboard or call:
   ```http
   GET /api/v1/audit-logs?resource=route&resource_id={Route_ID}
   ```
2. Verify a new entry exists for the specific Route ID.
3. Validate that the log payload contains:
   * Timestamp of the change.
   * Actor ID (The manager who made the change).
   * Action type (`REASSIGN_ROUTE`).
   * Previous values (`original_agent_id`).
   * New values (`dest_agent_id`).

### Email Sent Verification (via Mailhog / Test Inbox)
1. Open the email testing tool .
2. Search for messages sent immediately after the reassignment timestamp.
3. **Verify Email 1** sent to the Original Agent:
   * Subject confirms removal from the route.
   * Body contains Route ID, date, and time.
4. **Verify Email 2** sent to the Receiving Agent:
   * Subject confirms new route assignment.
   * Body contains Route ID, location, type, and schedule.

--

## 4. Bug Reporting Sample

* Use the template file located in this directory to log new issues:
  [Open Bug Report Template](./bugReport_example.md)

--

## 5. Assumptions & Risks

### Assumptions
* **Timezones**: All campaigns, routes, and agent calendars operate on a single matching timezone, or the system auto-converts schedules to UTC before validation.
* **Lock Scope**: The 24-hour lock applies strictly to the route identity itself, meaning no attributes (agent, time, location type) can be edited while locked.
* **Dependencies**: None. The scope is fully self-contained within this module, with no external system or third-party dependencies involved


### Risks
* **Race Conditions**: Two managers attempting to reassign the same route or assign the same agent to different routes at the exact same split-second might bypass the conflict checks.
* **Notification Failures**: If the email service provider fails mid-transaction, a route could be reassigned without either agent being informed, leading to missed shifts.

--

## 6. Checklist for Regression Impact
* **Agent Mobile Application**: Verify the updated schedules sync properly and display the new routes without lagging or crashing.
* **Campaign Progress Metrics**: Validate that changing agents does not duplicate or erase target metrics for the active campaign.

--
## **7. Contact**

If you need something else or you have any doubts, please feel free to [contact me:](mailto:mmungarro@gmailcom)
