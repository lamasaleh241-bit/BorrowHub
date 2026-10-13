# BorrowHub — Product Requirements Document

**Document status:** Draft for stakeholder review  
**Prepared:** 2026-10-13  
**Product type:** University web application  
**Primary source:** Requirements discovery interview with the product owner  
**Scope note:** This document separates confirmed interview requirements from analyst recommendations and unresolved decisions. It does not specify an implementation stack because none was discussed.

## 1. Product Overview

BorrowHub is a web application for managing university equipment loans, including cameras, projectors, and laptops. It replaces paper-based borrowing logs with a shared catalog, availability checks, human-reviewed requests, individually identified asset tracking, and recorded handover and return events.

The intended institution is one university. The initial operational footprint within that university—campus-wide or limited to a faculty/department—has not been decided. Students and staff borrow equipment; equipment managers administer inventory and loan operations. A separate system administrator role is recommended for account, role, scope, and calendar configuration.

## 2. Problem Statement

The current process uses paper logs. The product owner identified lost assets, unclear availability, and a lack of transparency as the main problems. Manual paperwork also makes it difficult to establish who has a particular item, whether it is available for a requested period, and whether it was returned on time.

BorrowHub should create a reliable digital record from request through return. It must not imply that digitizing the workflow alone prevents loss; individual asset identifiers, operational handover/return recording, and auditability are needed to support that goal.

## 3. Product Goals

1. Replace the paper-based request and loan record with a digital workflow.
2. Let eligible students and staff see equipment status and availability for requested dates and times.
3. Track each physical asset independently using a unique identifier.
4. Require equipment-manager review and prevent approval when requested stock is no longer available.
5. Record physical handover, return, and return inspection.
6. Give borrowers visibility into current and past requests and their statuses.
7. Support timely returns through in-app reminders and a clearly recorded late fee.

**Success measurement:** No numeric targets or baseline were supplied. Candidate metrics are listed in Section 22 and require product-owner approval before being treated as targets.

## 4. Non-Goals

The following are outside the MVP based on the interview:

- Online payment processing or payment-gateway integration. The MVP records a fine and its payment status manually.
- Email or SMS notifications; in-app notifications are the selected channel.
- QR scanning for handover or return; equipment managers record those events manually.
- Native iOS or Android applications.
- Automatic damage/loss charges. No rate, assessment process, or policy was specified.
- Integrations with finance, student information, or other university systems beyond the university single sign-on (SSO), unless separately approved.
- Choosing a technology stack, hosting model, or university identity provider; none was discussed.

## 5. Target Users

| User | Description | Primary jobs |
|---|---|---|
| Student borrower | A university student authenticated through university SSO | Browse equipment, request a quantity and time period, track requests, receive in-app notifications, return items |
| Staff borrower | A university staff member authenticated through university SSO | Same borrowing tasks as a student; no different borrowing policy was confirmed |
| Equipment manager | Staff member responsible for an assigned equipment inventory or unit | Maintain inventory, review requests, assign physical assets, record handover/return and inspection, record fine payment |
| System administrator (recommended) | Administrative operator separate from day-to-day equipment management | Manage equipment-manager accounts/scopes and configure business hours, operating days, and holidays |

The university must decide whether the first release serves the whole campus or one faculty/department. The distinction between student and staff accounts may be supplied by SSO; the attribute mapping is unknown.

## 6. User Personas

These are role-based working personas, not demographic profiles.

| Persona | Context and need | Success outcome |
|---|---|---|
| Student or staff borrower | Needs to know whether a camera, projector, or laptop is available and what happened to a request | Can submit a valid request, see its status, and understand return obligations |
| Equipment manager | Handles inventory and requests for an assigned stock/unit; needs accurate asset-level records | Can make a valid decision, hand over the correct physical unit, and record its return condition |
| System administrator (recommended) | Maintains access and operating-calendar configuration without performing every loan transaction | Can assign least-privilege scopes and maintain operating rules |

## 7. User Needs

- Borrowers need a current view of equipment and availability by requested date/time.
- Borrowers need a simple request form for equipment/type, quantity, pickup date/time, and return date/time.
- Borrowers need to know whether a request is pending, approved, rejected, cancelled, expired, checked out, returned, or overdue.
- Borrowers need in-app notifications for relevant request and late-return events.
- Equipment managers need to identify the physical units assigned to an approved request and record custody transitions.
- Equipment managers need a reliable way to inspect returned equipment and choose its resulting inventory status.
- Administrators need to constrain access to the manager's assigned inventory/unit and configure working times and holidays.

## 8. User Stories

| ID | User story | Priority |
|---|---|---|
| US-001 | As a student or staff borrower, I want to sign in with my university account so that I can use my verified university identity. | Must Have |
| US-002 | As a borrower, I want to browse equipment and check availability for a period so that I do not request unavailable stock. | Must Have |
| US-003 | As a borrower, I want to request a type and quantity with pickup and return times so that the equipment manager can review the request. | Must Have |
| US-004 | As a borrower, I want to view my current and past requests and their statuses so that I know what action is needed. | Must Have |
| US-005 | As a borrower, I want to cancel a request before it is approved so that I can withdraw a request I no longer need. | Must Have |
| US-006 | As an equipment manager, I want to approve or reject requests and assign physical units so that inventory and custody remain clear. | Must Have |
| US-007 | As an equipment manager, I want to record handover, return, and inspection results so that each unit's state is traceable. | Must Have |
| US-008 | As an equipment manager, I want to maintain individually identified assets and their statuses so that availability reflects actual stock. | Must Have |
| US-009 | As a borrower, I want in-app reminders and a visible fine status when a loan is overdue so that I can resolve the overdue loan. | Must Have |
| US-010 | As a system administrator, I want to manage manager scopes and operating calendars so that access and valid pickup/return times match university operations. | Must Have (recommended role) |

## 9. MVP Scope

### Included

- Responsive web experience for borrowers, equipment managers, and a recommended separate system-administrator role.
- University SSO authentication; provider and attribute mapping remain dependencies.
- Equipment catalog with the confirmed catalog fields: name, category, quantity/count, and status.
- Individual physical asset records with unique identifiers.
- Requests for item/type, quantity, pickup date/time, and return date/time.
- Ten-calendar-day maximum loan duration.
- Availability checks at request submission and again at manager approval.
- Human approval for every request; approval or rejection does not require a rejection reason.
- At most two approved active loans per user; pending requests are excluded from this limit.
- Borrower cancellation before approval only.
- In-app notifications and borrower request history.
- Manual equipment-manager handover, return, and return-condition recording.
- Automatic one-time USD 50 fine per overdue loan; manual payment-status recording.
- Two in-app reminders within the first 24 hours after the return deadline; exact send times unresolved.
- Configurable working hours, operating days, and holidays; appointments outside configured windows are blocked.
- Audit trail for important inventory and loan actions.
- Analyst-recommended policy choices accepted under the owner's “choose what is best” direction: block new requests when a borrower has an overdue loan or unpaid fine; automatically expire an unreviewed request at its requested pickup start time and notify the borrower; separate system-administrator role; manager access limited to assigned scope.

### Explicitly excluded from MVP

Online payments, email/SMS, QR scanning, native apps, automatic damage/loss fees, and unconfirmed integrations beyond SSO.

## 10. Core Features

### F-001 — University sign-in and scoped access

| Attribute | Definition |
|---|---|
| Purpose | Authenticate university users and ensure each role can access only its authorized functions and inventory scope. |
| User story | US-001, US-010. |
| Functional requirements | FR-001–FR-005. |
| Business rules | University SSO is required. Roles are borrower, equipment manager, and (recommended) system administrator. Managers are limited to assigned scope. |
| Edge cases | SSO identity has no usable student/staff attributes; manager has no assigned scope; user account is disabled. |
| Error states | Authentication failure, unavailable identity provider, or unauthorized route must fail safely and provide a retry/contact path without exposing protected data. |
| Acceptance criteria | **Given** an authenticated borrower, **when** they open an admin route, **then** access is denied. **Given** a manager with a defined scope, **when** they view inventory, **then** only assigned-scope records are visible. |
| Priority | Must Have |

### F-002 — Catalog and individually tracked inventory

| Attribute | Definition |
|---|---|
| Purpose | Show catalog information while preserving the status and identity of each physical unit. |
| User story | US-002, US-008. |
| Functional requirements | FR-006–FR-010. |
| Business rules | Catalog fields are name, category, count/quantity, and status. Each physical unit has a unique identifier. |
| Edge cases | Duplicate unit identifier; catalog count differs from unit records; unit is in maintenance, lost, unavailable, or retired. |
| Error states | Missing required catalog values or duplicate identifier prevents save; stale availability must not be presented as guaranteed. |
| Acceptance criteria | **Given** a unit marked unavailable, **when** a borrower searches for available stock, **then** it is excluded from the requestable quantity. **Given** a duplicate unique identifier, **when** a manager saves, **then** the system rejects it and identifies the conflict. |
| Priority | Must Have |

### F-003 — Borrow request, availability, and approval

| Attribute | Definition |
|---|---|
| Purpose | Collect complete requests and prevent invalid, conflicting, or over-limit loans. |
| User story | US-002, US-003, US-005, US-006. |
| Functional requirements | FR-011–FR-020. |
| Business rules | Maximum duration is 10 consecutive calendar days; maximum two approved active loans per borrower; pending requests do not count; requests require human approval; unavailable stock is blocked at submission; availability is rechecked at approval; cancellation is allowed only before approval. |
| Edge cases | Availability changes after submission; two requests compete for the same units; request exceeds duration or working window; borrower reaches limit; borrower has overdue loan/unpaid fine. |
| Error states | Validation errors identify the failed rule. Approval must be blocked on a stock conflict. The exact disposition of a pending request with a conflict is unresolved; the system must not silently approve it. |
| Acceptance criteria | **Given** a request for unavailable quantity, **when** the borrower submits it, **then** submission is blocked. **Given** the requested quantity was available at submission but is no longer available, **when** the manager attempts approval, **then** approval is blocked and the conflict is shown. **Given** a borrower has two approved active loans, **when** they submit another request, **then** submission is blocked. |
| Priority | Must Have |

### F-004 — Physical assignment, handover, return, and inspection

| Attribute | Definition |
|---|---|
| Purpose | Record which physical units were issued, when custody changed, and what condition units were in when returned. |
| User story | US-006, US-007, US-008. |
| Functional requirements | FR-021–FR-025. |
| Business rules | Equipment managers record handover and return manually. At return, the manager records “good,” “damaged,” or “incomplete/missing”; the manager then determines the final inventory status after inspection. |
| Edge cases | Partial handover; assigned unit cannot be found; return condition is damaged or incomplete; duplicate return entry. |
| Error states | Do not mark a unit available if the return/inspection transition did not complete; prevent duplicate or invalid transitions and preserve an audit event. |
| Acceptance criteria | **Given** an approved request with assigned units, **when** the manager records handover, **then** the units become checked out and are linked to the borrower/request. **Given** a returned unit, **when** the manager records its inspection, **then** the inspection outcome and manager are recorded and the manager can set the resulting inventory status. |
| Priority | Must Have |

### F-005 — Overdue detection, reminders, and fines

| Attribute | Definition |
|---|---|
| Purpose | Identify late loans, notify borrowers, and record the agreed fine without processing payment online. |
| User story | US-009. |
| Functional requirements | FR-026–FR-030. |
| Business rules | One automatic USD 50 fine is created per overdue loan. Two in-app reminders are sent during the first 24 hours after the due time. Fine payment is recorded manually. New borrowing requests are blocked while an overdue loan or unpaid fine exists (analyst recommendation made under delegated discretion). |
| Edge cases | Scheduler/event repeats; return is recorded near the due-time boundary; multiple loans become overdue; manual payment status is changed incorrectly. |
| Error states | Fine creation must be idempotent per loan. If notification delivery fails, the overdue/fine record remains intact. |
| Acceptance criteria | **Given** a loan remains unreturned after its due timestamp, **when** overdue processing runs, **then** exactly one USD 50 fine is recorded for that loan. **Given** an unpaid fine, **when** the borrower submits a new request, **then** the request is blocked. **Given** a manager records payment, **when** the change succeeds, **then** the fine status is paid and the actor/time are audited. |
| Priority | Must Have |

### F-006 — In-app notifications and borrower history

| Attribute | Definition |
|---|---|
| Purpose | Keep borrowers and managers informed of request and loan events. |
| User story | US-004, US-009. |
| Functional requirements | FR-031–FR-033. |
| Business rules | In-app notifications are the only confirmed channel. Borrowers can view current and past requests and their statuses. Managers receive an in-app notification for a new request. |
| Edge cases | User has not opened the app; notification event is duplicated; a request expires before being reviewed. |
| Error states | Notification failure must not roll back the underlying request or inventory transaction; unread notifications must remain available in the app. |
| Acceptance criteria | **Given** a request is submitted, **when** it is recorded, **then** the borrower and responsible manager receive in-app notification records. **Given** a borrower opens request history, **when** they view it, **then** their current and past requests and status are shown. |
| Priority | Must Have |

### F-007 — Operating calendar and pending-request expiry

| Attribute | Definition |
|---|---|
| Purpose | Restrict pickup/return selections to configured operating periods and avoid stale unreviewed requests. |
| User story | US-003, US-010. |
| Functional requirements | FR-034–FR-036. |
| Business rules | A system administrator configures hours, operating days, and holidays. The system blocks pickup/return selections outside configured windows. An unreviewed request expires at its requested pickup start and the borrower is notified (analyst recommendation made under delegated discretion). |
| Edge cases | Holiday/calendar changes after request submission; requested pickup start arrives while review is in progress; time-zone boundary. |
| Error states | If calendar configuration is missing or invalid, do not accept a date/time that cannot be validated; show a configuration error to administrators. |
| Acceptance criteria | **Given** a date/time outside configured hours or on a configured holiday, **when** the borrower submits, **then** the time is rejected. **Given** a request remains unreviewed at pickup start, **when** expiry processing runs, **then** the request expires and the borrower receives an in-app notification. |
| Priority | Must Have |

### F-008 — Audit trail

| Attribute | Definition |
|---|---|
| Purpose | Make operational changes and decisions traceable. |
| User story | Supports US-006–US-010. |
| Functional requirements | FR-037–FR-038. |
| Business rules | Record who performed important actions, what changed, and when. The product owner explicitly confirmed the need for an audit log. |
| Edge cases | Repeated event, corrected entry, actor account later disabled. |
| Error states | An audit write failure must not silently produce an untraceable high-impact inventory/loan change; transaction handling and recovery must be defined. |
| Acceptance criteria | **Given** a manager changes an asset status, **when** the change is saved, **then** an audit record identifies the actor, action, target, and timestamp. |
| Priority | Must Have |

## 11. Functional Requirements

| ID | Requirement | Priority / status |
|---|---|---|
| FR-001 | Authenticate borrowers, managers, and administrators through university SSO. | Must Have; provider TBD |
| FR-002 | Enforce role-based access and manager scope on every protected operation. | Must Have |
| FR-003 | Provide separate borrower, equipment-manager, and recommended system-administrator roles. | Must Have; administrator separation is an analyst recommendation accepted under delegated choice |
| FR-004 | Allow the system administrator to manage manager accounts, role assignments, and inventory scopes. | Must Have; recommended role |
| FR-005 | Show catalog name, category, quantity/count, and status. | Must Have |
| FR-006 | Represent each physical unit with a unique identifier and individual operational status. | Must Have |
| FR-007 | Allow an equipment manager to create and update inventory data, quantity, and unit status within assigned scope. | Must Have |
| FR-008 | Calculate availability for a requested interval from unit statuses and overlapping approved loans/assignments. | Must Have; exact reservation transition TBD |
| FR-009 | Collect equipment/type, quantity, pickup date/time, and return date/time on a request. | Must Have |
| FR-010 | Reject requests longer than 10 consecutive calendar days. | Must Have |
| FR-011 | Reject a request when the requested quantity is unavailable for the requested period. | Must Have |
| FR-012 | Require an equipment-manager approval or rejection for every valid submitted request. | Must Have |
| FR-013 | Recheck availability atomically when approval is attempted and block approval if insufficient stock remains. | Must Have |
| FR-014 | Limit each borrower to two approved active loans at a time; exclude pending requests from the count. | Must Have; definition of active is open |
| FR-015 | Allow a borrower to cancel their own request only before manager approval. | Must Have |
| FR-016 | Do not require a rejection reason. | Must Have |
| FR-017 | Allow the equipment manager to assign individually identified physical units to an approved request. | Must Have; exact point in approval/handover flow is open |
| FR-018 | Allow the equipment manager to record physical handover manually. | Must Have |
| FR-019 | Allow the equipment manager to record return and inspection outcome: good, damaged, or incomplete/missing. | Must Have |
| FR-020 | Let the manager determine the final unit status manually after inspection; do not automatically mark a damaged/incomplete unit as lost or unavailable. | Must Have |
| FR-021 | Show borrowers current and past requests with statuses. | Must Have |
| FR-022 | Send in-app notifications to the borrower for submission, approval/rejection, expiry, and relevant overdue/fine events. | Must Have; exact notification matrix to finalize |
| FR-023 | Notify the relevant equipment manager in-app when a new request arrives. | Must Have |
| FR-024 | Create one automatic USD 50 fine per overdue loan. | Must Have; amount/currency should be institutionally validated |
| FR-025 | Track fine status at minimum as unpaid or paid; allow manual payment-status recording by authorized staff. | Must Have |
| FR-026 | Send two in-app reminders during the first 24 hours after overdue begins. | Must Have; exact send times open |
| FR-027 | Block new requests if the borrower has an overdue loan or an unpaid fine. | Must Have; analyst recommendation accepted under delegated choice |
| FR-028 | Configure working hours, operating days, and holidays; reject pickup/return times outside the configuration. | Must Have |
| FR-029 | Expire an unreviewed request at requested pickup start and notify the borrower. | Must Have; analyst recommendation accepted under delegated choice |
| FR-030 | Keep an audit trail of inventory changes, request decisions, handovers, returns, inspections, and fine status changes. | Must Have |

## 12. Non-Functional Requirements

No service-level targets, throughput estimates, or retention rules were supplied. The following are minimum quality requirements; quantitative targets remain open.

| ID | Requirement | Verification |
|---|---|---|
| NFR-001 | Enforce authorization server-side for all requests and data access; UI hiding alone is insufficient. | Automated authorization tests for each role and scope |
| NFR-002 | Prevent unauthorized cross-unit access to inventory, requests, borrower identity, and fines. | Negative tests across distinct manager scopes |
| NFR-003 | Prevent double assignment of the same physical unit to overlapping approved loan intervals. | Concurrency/integration test with competing approval attempts |
| NFR-004 | Make fine creation idempotent so retries cannot create duplicate fines for one loan. | Retry/replay test |
| NFR-005 | Store timestamps consistently and display them in the university's configured time zone. | Boundary tests; time zone must be selected |
| NFR-006 | Preserve the loan/request transaction when notification delivery fails; notifications are retriable independently. | Fault-injection test |
| NFR-007 | Meet accessibility expectations for keyboard operation, visible focus, labels, and status communication. | Accessibility review and automated checks |
| NFR-008 | Maintain an auditable actor and timestamp for critical operational changes. | Audit completeness tests |
| NFR-009 | Protect authentication and personal data in transit and at rest according to university policy. | Security review; exact controls and policy are dependencies |
| NFR-010 | Define production availability, backup, restore, recovery-time, and recovery-point targets before launch. | Operational readiness review; numeric targets open |
| NFR-011 | Define supported browser versions and performance thresholds before launch. | Compatibility and performance test plan; targets open |

## 13. User Flows

### Borrower request flow

1. Borrower signs in through university SSO.
2. Borrower browses catalog and selects an equipment type, quantity, pickup time, and return time.
3. System validates required fields, maximum duration, configured work calendar, availability, active-loan limit, and borrowing blocks (overdue loan/unpaid fine).
4. If valid, the request is recorded as pending review and in-app notification records are created for the borrower and relevant equipment manager.
5. Borrower can cancel while the request remains unapproved.

### Equipment-manager review flow

1. Manager opens a new request in their assigned scope.
2. Manager checks requested quantity and period and chooses approve or reject.
3. On approval attempt, system rechecks availability. If stock is insufficient, approval is blocked and the conflict is shown.
4. Manager assigns individual units as part of the approval/handover workflow; the exact point must be decided.
5. Borrower receives an in-app status notification.

### Handover and return flow

1. Manager manually records the physical handover and the assigned unit(s) become checked out.
2. Borrower returns the unit(s).
3. Manager records return and inspection outcome (good, damaged, incomplete/missing).
4. Manager determines the final inventory status manually after inspection.

### Overdue flow

1. A loan reaches its due date/time without a recorded return.
2. System marks it overdue and creates one USD 50 fine.
3. System sends two in-app reminders during the first 24 hours; exact timing remains open.
4. New borrower requests are blocked while an overdue loan or unpaid fine exists.
5. Authorized manager records payment manually when confirmed outside the system.

### Unreviewed request expiry flow

1. A request remains pending until its requested pickup start.
2. At pickup start, system expires the request and notifies the borrower.
3. Exact treatment of inventory reservation during a pending request is unresolved; no unit may be double-booked.

## 14. Pages/Screens

| Screen | Primary user | Required content/actions |
|---|---|---|
| Sign-in / SSO callback | All | Sign-in initiation, authentication result, safe error/retry state |
| Equipment catalog | Borrower | Search/browse, name/category/count/status, availability by interval |
| Request form | Borrower | Item/type, quantity, pickup and return date/time, validation messages |
| My requests | Borrower | Current and past requests, status, request details, pre-approval cancellation |
| Request details | Borrower / manager | Dates, quantity, status, assigned units as appropriate, event history |
| Manager dashboard / request queue | Equipment manager | New/pending requests in assigned scope, filters, review action, notification state |
| Request review | Equipment manager | Availability recheck, approve/reject, unit assignment, no required rejection reason |
| Inventory list | Equipment manager | Catalog and individual asset rows, identifiers, statuses, create/update actions |
| Asset details | Equipment manager | Identifier, category/type, status, loan/custody history, status update |
| Handover / return | Equipment manager | Manual checkout/return recording, inspection outcome, final status selection |
| Fines | Borrower / equipment manager | Borrower sees amount/status; manager records paid status manually |
| In-app notifications | Borrower / manager | Request, approval, rejection, expiry, overdue, fine, new-request notifications |
| System settings | System administrator | Manager accounts/scopes, working hours, operating days, holidays |
| Audit log | Authorized administrators/managers | Filterable actor/action/target/time record within authorized scope |

Final navigation and whether screens are combined or separated are design decisions.

## 15. Roles & Permissions

| Action | Borrower | Equipment manager (assigned scope) | System administrator (recommended) |
|---|---:|---:|---:|
| Browse catalog and availability | Yes | Yes | Yes |
| Submit request and see own history | Yes | As borrower if eligible; policy TBD | As borrower if eligible; policy TBD |
| Cancel own pending request | Yes | No | No by default |
| Review/approve/reject request | No | Yes | No by default |
| Assign physical units | No | Yes | No by default |
| Record handover, return, inspection | No | Yes | No by default |
| Manage inventory/status | No | Yes within scope | Scope administration; operational management not assumed |
| Record fine as paid | No | Yes within scope | No by default |
| Manage manager roles/scopes | No | No | Yes |
| Configure work calendar | No | No | Yes |
| View audit history | Own relevant events only, exact presentation TBD | Within scope | Administrative access subject to university policy |

Least-privilege access is recommended. The university must approve the final administrator permissions, manager-scope structure, and whether staff may also use the borrower experience.

## 16. Data Requirements

Conceptual data entities required by the confirmed workflows:

| Entity | Minimum fields/relationships |
|---|---|
| User | University identity/SSO subject, display identity, role, status, optional affiliation attributes from SSO (exact attributes TBD) |
| Scope / unit | University faculty/department or inventory scope; exact organizational model TBD |
| Equipment type/catalog record | Name, category, quantity/count, aggregate status as needed for display |
| Physical asset | Unique identifier, equipment type, current status, assigned scope, status history |
| Loan request | Borrower, requested type, quantity, pickup timestamp, return timestamp, request status, created/updated timestamps |
| Asset assignment | Request, physical asset identifier(s), assignment state and timestamps |
| Handover record | Request, asset(s), manager actor, handover timestamp |
| Return/inspection record | Request, asset(s), manager actor, return timestamp, inspection outcome, final status decision |
| Fine | Loan/request, amount, currency, created timestamp, unpaid/paid status, manual payment actor/time (payment evidence policy TBD) |
| Notification | Recipient, event/type, creation/read/delivery state, related request/fine |
| Operating calendar | Time zone, working-hour windows, operating days, holidays, effective dates |
| Audit event | Actor, action, target entity, before/after values where appropriate, timestamp |

The distinction between a catalog model’s aggregate count and the number of individually represented physical assets should be validated and reconciled. Data-retention periods and audit-log retention are open.

## 17. Integrations

| Integration | Need | Status/dependency |
|---|---|---|
| University SSO / identity provider | Authenticate students and staff using university accounts | Confirmed requirement; provider, protocol, test credentials, and identity attributes unknown |
| In-app notification mechanism | Deliver notifications in BorrowHub | Confirmed channel; no external provider selected |
| Payment gateway | Not in MVP | Explicitly excluded; fines are manually marked paid |
| Email/SMS | Not in MVP | Not selected; no integration planned |
| QR/barcode scanner | Not in MVP | Manual handover/return chosen |
| Other university systems | Not specified | Do not assume integration |

## 18. Edge Cases

1. A physical unit becomes unavailable after submission but before approval: block approval and show the conflict; how the pending request is resolved is open.
2. Multiple managers attempt to approve overlapping requests for the same unit: only one valid assignment may succeed.
3. A borrower submits a third approved-active loan: block according to the two-loan limit.
4. A borrower has an overdue loan or unpaid fine: block new request according to the analyst-selected policy.
5. A request is cancelled before approval: release any pending hold if the system creates one; whether pending requests reserve inventory is open.
6. A request is not reviewed by pickup start: expire and notify per analyst-selected policy.
7. A return is recorded around the due timestamp: overdue determination must use one consistent time-zone-aware boundary; exact grace-period policy is not defined and must not be invented.
8. A unit is returned damaged or incomplete: manager records inspection outcome and manually selects final unit status; no automatic damage fee is defined.
9. Fine processing or reminder job retries: create only one fine and avoid duplicate reminder events where idempotency is required.
10. A holiday or operating-hour configuration changes after a request was submitted: whether existing requests are grandfathered or recalculated is open.
11. SSO does not provide affiliation/role attributes: deny privileged access and define an administrator-controlled role-assignment process.
12. Duplicate, missing, or malformed asset identifier: reject save and identify the data error.
13. Notification delivery fails: preserve business transaction; retry or show notification in the in-app inbox.
14. Equipment manager's scope is changed while requests are pending: ensure access follows current authorization while retaining auditability.

## 19. Error States

| Error | Expected system behavior |
|---|---|
| SSO failure or expired session | Do not create or alter a request; show a clear retry/sign-in message |
| Unauthorized access | Deny access without revealing protected record details; log security-relevant events where appropriate |
| Insufficient availability | Block submission or approval as applicable and show the failed quantity/period check |
| Loan duration exceeds 10 days | Block submission and state the maximum allowed duration |
| Outside configured business hours/holiday | Block selection/submission and request another valid time |
| Borrower loan limit reached | Block new submission and explain the two-active-loan rule |
| Overdue loan/unpaid fine | Block new submission and show relevant outstanding status without exposing another user's data |
| Duplicate unit identifier | Reject inventory save and identify the duplicate |
| Invalid status transition | Keep existing valid state, show the issue, and avoid partial custody updates |
| Audit persistence failure | Do not silently complete a critical state change without its audit record; provide recoverable failure behavior |
| Notification service failure | Keep request/loan/fine state intact; queue/retry notification without duplicate business action |
| Concurrent asset assignment | Reject the conflicting assignment and refresh availability |
| Incomplete calendar configuration | Prevent booking against an unknown schedule and notify authorized administrator |

## 20. Security & Privacy Requirements

- Use university SSO; do not create a separate borrower password system unless later approved.
- Enforce authorization in the backend for every request, inventory, fine, and audit operation.
- Enforce equipment-manager scope at record level, not only by UI filtering.
- Restrict borrower access to their own request history and fine records.
- Minimize personal data stored beyond what is required for identity, request operations, notifications, and audit.
- Protect identity and personal data in transit and at rest in accordance with university security policy; exact technical controls are a dependency.
- Record privileged changes and custody transitions in an audit log with actor and timestamp.
- Define retention, deletion, legal hold, and audit access policies with the university before production.
- Do not store payment-card data; no payment gateway is in the MVP.
- Review whether the fine amount and collection workflow comply with university policy and applicable law before enabling fines in production.

## 21. Analytics & Tracking

Analytics should answer whether the product is replacing paper processes and improving operational visibility. Proposed events (subject to privacy review):

| Event | Suggested properties (minimized) |
|---|---|
| `request_submitted` | request ID, equipment type, quantity, requested duration, outcome |
| `request_decided` | request ID, decision, elapsed review time, scope ID |
| `request_expired` | request ID, time pending, scope ID |
| `handover_recorded` | request ID, unit count, actor role |
| `return_recorded` | request ID, unit count, inspection outcome |
| `loan_overdue` | request ID, duration past due, fine-created flag |
| `fine_status_changed` | fine ID, new status, elapsed time to payment |
| `availability_conflict` | request ID, phase (submit/approval), requested vs. available quantity |

Do not place names, university IDs, or sensitive free text in general analytics. The audit log is operational evidence, not a substitute for privacy-reviewed product analytics. Analytics platform and retention are undecided.

## 22. Success Metrics / KPIs

No baseline, target, or reporting window was specified. Establish these before treating the metrics as commitments:

| KPI | Definition | Target |
|---|---|---|
| Digital request adoption | Requests recorded in BorrowHub divided by all equipment requests | TBD |
| On-time return rate | Loans returned by due time divided by loans due in period | TBD |
| Asset identity coverage | Active physical units with unique valid identifiers divided by active units | TBD |
| Availability conflict rate | Approval attempts blocked for insufficient stock divided by approval attempts | TBD |
| Review turnaround | Median time from submission to manager decision | TBD |
| Lost-asset rate | Units marked lost per active units in period | TBD |
| Fine payment recording completeness | Fines with a recorded status update divided by fines created | TBD |

These are candidate metrics, not approved targets.

## 23. Acceptance Criteria

The MVP is acceptable when the following testable conditions pass in a representative university test environment:

1. **SSO:** Given a valid university identity, when the user authenticates, then BorrowHub creates/loads the correct authorized role and scope; invalid or unauthorized identities cannot access protected data.
2. **Catalog:** Given individually identified units, when a borrower views a type, then name, category, quantity/count, status, and period availability are presented consistently with unit records.
3. **Request validation:** Given a complete request within the 10-day limit and configured working calendar, when the borrower submits with available stock and without borrowing blocks, then it is recorded as pending review.
4. **Unavailable stock:** Given insufficient stock for the requested period, when the borrower submits, then the system blocks submission and explains the availability conflict.
5. **Duration:** Given a requested period longer than 10 consecutive calendar days, when the borrower submits, then submission is rejected with an actionable validation message.
6. **Work calendar:** Given a pickup or return time outside configured hours, operating days, or holidays, when the borrower submits, then the system blocks the invalid time.
7. **Two-loan limit:** Given two approved active loans for a borrower, when they submit another request, then the system blocks it. A pending request does not count toward the limit.
8. **Manager review:** Given a valid pending request, when a manager within the assigned scope approves or rejects it, then the decision is recorded without requiring a rejection reason and the borrower is notified in-app.
9. **Approval-time conflict:** Given availability has changed since submission, when the manager attempts approval, then approval cannot complete and the conflict is shown.
10. **Cancellation:** Given the borrower owns a pending request, when they cancel it before approval, then its status changes to cancelled; after approval, borrower cancellation is unavailable.
11. **Asset-level assignment:** Given a request is approved, when the manager assigns physical units, then each assigned unit is uniquely linked to that loan and cannot be assigned to an overlapping loan.
12. **Handover/return:** Given an assigned loan, when the manager records handover and later return, then the custody timestamps, actor, units, and inspection outcome are stored.
13. **Inspection:** Given a returned item is damaged or incomplete, when the manager records inspection, then the system does not automatically set a final status or create a damage fine; the manager makes the final status decision.
14. **Overdue/fine:** Given a loan remains unreturned after its due timestamp, when overdue processing runs, then exactly one USD 50 fine is created for that loan.
15. **Manual payment status:** Given an authorized manager records a fine as paid, when the update succeeds, then status, actor, and time are recorded; no online payment is processed.
16. **Reminders:** Given a loan becomes overdue, when reminder processing runs, then two in-app reminders are scheduled/delivered within the first 24 hours according to the approved schedule.
17. **Borrowing block:** Given a borrower has an overdue loan or unpaid fine, when they submit a new request, then the system blocks it and shows the applicable reason.
18. **Request history:** Given a borrower has past/current requests, when they open their history, then only their own requests and statuses are visible.
19. **Pending expiry:** Given a request is still pending at its pickup start, when expiry processing runs, then the request is marked expired and the borrower is notified in-app.
20. **Audit:** Given a manager or administrator performs an important inventory, request, custody, or fine action, when it is committed, then an audit record identifies actor, action, target, and timestamp.
21. **Scope isolation:** Given two managers have different assigned scopes, when either queries or mutates data, then neither can access the other's protected records.
22. **Notification resilience:** Given notification delivery fails, when a request/loan transaction succeeds, then the underlying business record remains correct and the notification can be retried without duplicating the transaction.

## 24. Future Features

Only consider after validating the MVP workflow and university policy:

- Online fine payment through an approved payment provider.
- Email or SMS notifications in addition to in-app notifications.
- QR/barcode-assisted asset lookup and handover/return.
- Additional university systems integration beyond SSO.
- Native mobile applications.
- Damage/loss assessment and fees, only after policy and amounts are approved.
- Reporting dashboards and export features, if stakeholders identify specific reporting needs.

These are potential future options, not committed roadmap items.

## 25. Risks & Dependencies

| Risk/dependency | Impact | Mitigation / decision needed |
|---|---|---|
| SSO provider and attributes unknown | Authentication and borrower/manager provisioning may block implementation | Identify provider, protocol, test environment, and role-affiliation attributes |
| University scope undecided | Impacts data model, manager scope, pilot size, and rollout | Choose campus-wide vs. faculty/department pilot |
| Asset inventory quality | Missing or duplicate IDs undermine asset-level tracking | Clean/import inventory and agree on unique identifier policy before launch |
| Fine policy/authority | USD 50 fee and blocking rules may conflict with university policy or local requirements | Obtain institutional approval before enabling fines and borrowing blocks |
| Manual payment confirmation | Incorrect status changes can create disputes | Define evidence and who may mark a fine paid |
| Approval/assignment timing unresolved | Could permit accidental double booking or confusion | Define whether units are reserved at approval or only at handover; enforce atomic assignment |
| Active-loan definition unresolved | Borrowing-limit calculation can differ across flows | Define the start/end events of an active loan |
| Reminder times unresolved | Reminder schedule cannot be implemented deterministically | Choose two offsets within first 24 hours |
| Pending requests and stock reservation unresolved | Availability shown to different users may conflict | Decide whether pending requests hold stock; approval recheck remains required |
| Time zone and grace period undefined | Boundary errors may incorrectly generate fines | Select university time zone and explicitly decide whether any grace period exists |
| Business hours/holiday ownership | Incorrect configuration can block legitimate requests | Assign a named operational owner and change process |
| Service levels and retention undefined | Production readiness and compliance cannot be verified | Set availability, backup/recovery, performance, privacy, and retention targets |

## 26. Open Questions

1. Will the first release serve the entire campus or one faculty/department? Which units are in scope?
2. What SSO provider/protocol will be used, and which attributes identify students, staff, and authorized managers?
3. What is the university's official time zone?
4. What are the exact two reminder offsets within the first 24 hours after the due time?
5. At exactly which point are physical units assigned: when approving a request, or immediately before handover?
6. What precisely counts as an “active” loan for the two-loan limit, and when does it stop counting: recorded return or completed inspection?
7. Do pending requests reserve stock? If yes, for how long? If not, availability can change before approval and approval must recheck it.
8. If approval is blocked because stock changed, does the manager reject the request, leave it pending, or return it to the borrower for changes?
9. Is there any grace period after the return deadline before a loan becomes overdue and a fine is created? None was specified.
10. Is USD 50 the institutionally approved amount and currency for every equipment type and borrower category?
11. Who may mark a fine paid, what evidence is recorded, and is a waiver/correction workflow required?
12. What happens to already-approved requests if working hours or holidays change after submission?
13. What data-retention, audit-retention, privacy, availability, backup, recovery, and performance targets apply?
14. How are initial inventory records and unique physical asset identifiers created or imported?
15. Can equipment managers also borrow as users, or are these roles mutually exclusive?

## 27. Recommended Development Phases

These phases organize delivery only; no technology stack or implementation approach is prescribed.

| Phase | Outcome | Exit checks |
|---|---|---|
| 0. Policy and scope confirmation | Confirm operational scope, SSO, asset identifiers, fine policy, active-loan definition, assignment timing, reminder schedule, and manager scope | Resolve blocking questions in Sections 25–26; obtain university policy approval for fines |
| 1. Inventory and access foundation | SSO, roles/scopes, manager setup, catalog, individually identified assets, operating calendar | Role/scope security tests pass; inventory is reconciled and identifiable |
| 2. Request and approval workflow | Availability search, request validation, two-loan limit, manual review, conflict handling, cancellation, request history | Request/approval acceptance criteria pass, including concurrency/availability conflicts |
| 3. Custody and overdue operations | Manual handover, return, inspection, overdue detection, reminders, fines, manual payment status | Custody and fine idempotency tests pass; manager operational process is approved |
| 4. Pilot and launch readiness | Pilot within selected university scope, user training, operational monitoring, analytics, support process | Pilot KPIs reviewed; accessibility/security/backup readiness completed; go-live decision recorded |

Do not begin implementation on unresolved policy-dependent fine behavior until the university has authorized the amount, currency, and collection process.
