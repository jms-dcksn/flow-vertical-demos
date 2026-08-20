# Commercial banking payment exception demo specification

## Use case and narrative

- **Domain and solution:** `commercial-banking/payment-exception/payment-exception-solution/`, with globally unique solution and package name `commercial-banking-payment-exception`.
- **Enterprise use case:** A payment-operations analyst resolves a synthetic ISO 20022 payment investigation. The Flow correlates the investigation with the original payment, network status, customer-risk context, screening result, and approved operating policy. The consequential decision is whether to request information, approve a routine repair, return or reject the payment, or escalate it to compliance. The hero moment is a reviewer correcting the proposed repair in the `Payment Exception Console` process app and the Flow resuming from that record change to create mock network and legacy-system receipts.
- **Why this use case:** It is the top-ranked candidate in the [commercial-banking opportunity research](agentic-workflow-opportunities.md), scoring 14/15. It combines a time-sensitive operational exception, structured payment data, bounded agent reasoning, API and UI-system work, human authorization, external communication, and an auditable outcome in one legible Flow.
- **Audience journey:** The demonstrator submits a synthetic investigation, follows four named canvas segments, opens the case in the console, edits and approves a safe repair, and then shows parallel network response, legacy write-back, communication drafting, and audit completion. Flow is the right surface because the value lies in coordinating deterministic logic, agents, RPA, API work, and human authority while making normal and exception routes visible.
- **Success outcome:** The case reaches the correct safe status with cited evidence, an explicit human outcome where required, and reconciled mock receipts. No agent releases, recalls, repairs, returns, rejects, or clears a live payment.
- **Out of scope:** Live Swift, Fedwire, CHIPS, payment-hub, sanctions, customer-master, or case-system connectivity; real customer or watchlist data; autonomous movement of funds; production policy interpretation; cancellation/recall processing; non-synthetic records in Data Fabric; production case-management governance for the case record; and quantified ROI without a design-partner baseline.

## Personas

| Persona | Role in the demo | Authority boundary |
| --- | --- | --- |
| Payment operations reviewer | Reviews evidence and selects the operational outcome. | May request information, approve an allowed repair, or return/reject only through the console review screen; cannot clear a compliance hold. |
| Sanctions/AML reviewer | Resolves possible screening matches or incomplete high-risk party data. | May release the case back to operations, maintain the hold, or request further investigation. |
| Payment operations queue manager | Owns overdue and technical-exception cases in the console work list. | Reassigns work and coordinates recovery; does not override required maker-checker or compliance decisions. |
| Demonstrator | Submits fixtures and narrates evidence, routing, review, and completion. | Uses synthetic data and mock write-backs only. |

## Trigger and case contract

This demo is one of the three Data Fabric process-app variants, so record creation starts the Flow. `caseId` plus `uetr` is the idempotency key, held in the `CorrelationKey` field; a repeated submission reads the existing record and appends a replay audit event.

| Item | Specification |
| --- | --- |
| Trigger | Node `uipath.connector.trigger.uipath-uipath-dataservice.record-created` on entity `CommercialBankingPaymentExceptionCase`. Pass the entity in `--detail.objectName`. Bind Data Fabric connection `b2a02899-3708-4bb6-810a-02321afb77f6` in folder `demos` explicitly, because it is not the default connection. The demonstrator creates the record with the console submit-fixture action; any seeding path must use a single-record insert, because a bulk insert does not fire the event. The API workflow then validates the raw `camt.110`-like investigation and completes the canonical record. |
| Canonical record | Data Fabric entity `CommercialBankingPaymentExceptionCase`, correlated by record `Id` and the unique `CorrelationKey` value `<caseId>:<uetr>`. The record holds the synthetic case, current state, agent output, reviewer decision, receipts, and final status. There is no Orchestrator queue for this case. The entity is tenant-scoped and is an `Entity` resource in the solution. |
| Required inputs | `caseId`, `uetr`, `investigationMessage`, `originalPayment`, `networkStatus`, `customerRiskContext`, `screeningResult`, `policyVersion`, and `receivedAt`. All values are synthetic; account and party identifiers are masked in tasks and logs. |
| Outputs | Classification, deterministic findings, cited evidence summary, advisory recommendation, reviewer outcome, `camt.111`-like response, mock write-back receipts, final status, and append-only audit events. |

### Input schema

| Field | Type | Required | Contract |
| --- | --- | --- | --- |
| `caseId` | string | yes | Synthetic case identifier. |
| `uetr` | UUID string | yes | Correlates payment and investigation events. |
| `investigationMessage` | object | yes | Raw and parsed `camt.110`-like reason, requester, requested fields, and timestamps. |
| `originalPayment` | object | yes | Synthetic `pacs.008`-like amount, currency, parties, agents, remittance data, and value date. |
| `networkStatus` | object | yes | Mock status, timestamps, and prior-message references. |
| `customerRiskContext` | object | yes | Synthetic customer ID, expected-activity summary, risk tier, and jurisdiction flags. |
| `screeningResult` | object | yes | Mock status, possible-match reason, list version, and timestamp. |
| `policyVersion` | string | yes | Pins the approved operating-policy corpus used by retrieval. |
| `receivedAt` | ISO 8601 string | yes | Starts case aging and timeout measurement. |

### Output schema

| Field | Type | Contract |
| --- | --- | --- |
| `classification` | enum | `missing_information`, `data_mismatch`, `status_request`, `possible_duplicate`, `screening_escalation`, or `other`. |
| `validationFindings` | array | Rule ID, affected field, actual value, expected constraint, and pass/fail. |
| `evidenceSummary` | object | Bounded narrative, source IDs, policy section citations, and confidence. |
| `requiresCompliance` | boolean | Forces the compliance route when true. |
| `recommendedAction` | enum | `respond_with_status`, `request_information`, `approve_repair`, `return_or_reject`, or `escalate_compliance`; advisory only. |
| `reviewOutcome` | object | Reviewer ID, named outcome, edited repair fields, rationale, and timestamp. |
| `responseMessage` | object | Synthetic `camt.111`-like response or information request. |
| `writeBackReceipts` | array | Mock system, operation ID, status, and timestamp. |
| `finalStatus` | enum | `awaiting_information`, `repair_approved`, `returned_or_rejected`, `compliance_hold`, `completed`, or `technical_exception`. |
| `auditEvents` | array | Actor, action, source references, input/output hashes, rule/model/prompt version, and timestamp. |

These names are the Flow output contract. Each one maps to a record field in the entity schema below, where the field names use the Data Fabric casing.

## Flow topology

Use four blue sticky notes with the titles below. The happy path runs left to right. Schema, matching, compliance, timeout, and dependency exceptions sit below their originating segment. Independent completion work is visually symmetric and merges before the final case update. A separately labelled `External agent showcase` branch sits below segment 2 so the Azure AI Foundry icon and connection are visible without making the placeholder agent part of the payment investigation.

| Reference segment | Domain-specific canvas title | Actors and business output | Branch or merge evidence |
| --- | --- | --- | --- |
| Receive and understand | **1. Receive and correlate** | The record-created trigger starts the Flow and supplies the record `Id`. `payment-message-gateway` API workflow validates and normalizes the message, the Flow writes the normalized case back to the record, and `legacy-payment-console` RPA retrieves the UI-only status. Output: canonical `PaymentExceptionCase` record. | Invalid schema, duplicate `CorrelationKey`, unmatched UETR, or RPA read failure routes to manual intake or technical exception below the happy path. A duplicate correlation value reads the existing record instead of creating a second case. |
| Assess and enrich | **2. Investigate the payment** | Deterministic script evaluates field and status rules. Inline `Payment Exception Investigator` uses read-only case, policy-search, customer-context, and screening tools to create the cited evidence summary and recommendation. A separate Azure AI Foundry node demonstrates external-agent connectivity with static metadata only. | `showExternalAgentShowcase === true` enters the non-material showcase branch and rejoins with the canonical case unchanged. Core routing then uses `requiresCompliance === true`; otherwise, `classification === "status_request" && evidenceSummary.confidence >= 0.85` takes the safe status-response path and all other cases continue to operations review. The confidence threshold is demo configuration, not a production policy claim. |
| Decide and review | **3. Control the resolution** | Safe status enquiries use a bounded no-repair route. For other cases the Flow sets the review state on the record and waits on `uipath.connector.event.uipath-uipath-dataservice.record-updated`. The reviewer works in the `Payment Exception Console` review screen, which shows facts, findings, citations, recommendation, and editable repair fields. A separate quick-form compliance task in Action Center handles screening cases. | Named outcomes route with the persisted `ReviewOutcome` value: `RequestInformation`, `ApproveRepair`, `ReturnOrReject`, or `EscalateCompliance`. Low confidence always enters operations review. The wait node filters on this case's record ID, so another reviewer's decision cannot resume it. A parallel 30-minute demo timer escalates to the queue manager. Only human outcomes enable write operations. |
| Act and communicate | **4. Respond and close** | In parallel: API workflow produces a mock network receipt; RPA records an approved repair in the mock legacy console when applicable; coded `Payment Update Writer` retrieves an approved template and drafts a safe status message. The branches merge, receipts reconcile, and the Flow writes the final status to the record. | `requiredReceipts.every(receipt => receipt.status === "succeeded")` completes the case. Any missing or failed required receipt sets `technical_exception`, leaves the record open, and routes it to the queue manager. |

## Agentic reasoning and tool use

| Actor | Visible responsibility | Structured input and output | Tools, context, and guardrails | Readiness/fallback |
| --- | --- | --- | --- | --- |
| Inline low-code agent: `Payment Exception Investigator` | Correlate evidence and produce a bounded, cited recommendation. | Input: canonical case, deterministic findings, `policyVersion`. Output: `classification`, `evidenceSummary`, `requiresCompliance`, `recommendedAction`, `confidence`. | Read-only tools: `get_customer_risk_summary`, `get_screening_result`, and `search_payment_policy`. Policy search is pinned to the approved version. Every material claim needs a source ID; unavailable evidence yields `insufficient_evidence`; web search and write tools are prohibited. | Not provisioned. Build with local synthetic tool responses and a repository policy fixture; bind approved tenant resources later. Retrieval failure bypasses the recommendation and still permits manual review. |
| Coded agent: `Payment Update Writer` | Draft one internal or customer-safe case update after the human decision. | Input: case ID, outcome, non-restricted facts, recipient type, and response deadline. Output: subject, body, `templateId`, `templateVersion`, and safety findings. | LangGraph agent calls `get_approved_payment_message_template` through a least-privilege UiPath MCP server, then self-checks for restricted sanctions rationale, unsupported promises, and missing required fields. A trajectory evaluator requires the template call. | Not provisioned. Use a local approved-template fixture and mock MCP tool during build. Tool failure routes to a manual-draft task; no message is sent automatically. |
| External agent showcase: `Azure AI Foundry connectivity` | Display the Azure AI Foundry node and prove that Flow can bind an external-agent connection; it performs no payment work. | Inputs are a connection-selected `agent_id` and the constant `message` `UiPath Flow external-agent connectivity showcase for commercial banking`; omit `thread_id`. The response is discarded and is never mapped into `PaymentExceptionCase`. | Node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` with Azure AI Foundry connection `0107247a-0197-42c9-b957-05d1b722b111`. Do not pass `caseId`, UETR, payment, customer, screening, policy, reviewer, or receipt data. The node has no authority to influence routing or write-backs. | Verified on August 12, 2026: the node is tenant-available and the connection is enabled and default in UiPath Labs Playground folder `demos`. Each Flow must still validate its binding. The branch is disabled by default, and timeout or failure records only a transient showcase status before rejoining the unchanged core route. |

## Data and resources

| Resource or data dependency | Purpose and contract | Owner/folder/readiness | Security and failure handling |
| --- | --- | --- | --- |
| IXP/document intelligence | Not used: the chosen intake is structured ISO 20022-like data, so adding document extraction would be ornamental. Attachments remain out of scope. | Not applicable. | If future investigation attachments become material, define fields and confidence review before adding IXP. |
| API workflow: `payment-message-gateway` | Operation `parse_investigation` validates and normalizes input; operation `send_response` writes a mock `camt.111`-like record and returns a receipt. Flow invokes it in segments 1 and 4. | New sibling project in the solution; connector/HTTP target is a local mock endpoint. Tenant connection and target folder are unverified. | Schema errors return typed validation findings. Calls use the idempotency key, two bounded retries for transient failures, and a technical-exception result without fabricated success. |
| RPA: `legacy-payment-console` | Operation `read_status` retrieves the UI-only payment status; `record_repair` writes only human-approved repair fields and returns a receipt. UI automation is purposeful because the demo console exposes no API. | New sibling project targeting a local mock console; production application and unattended runtime are unverified. | Synthetic data only. Selector or application errors return a typed exception and screenshot reference; no retry occurs after an ambiguous write. |
| Data Fabric entity: `CommercialBankingPaymentExceptionCase` | Canonical demo case, unique `CorrelationKey`, state transitions, agent output, reviewer decision, receipts, and final status. | New tenant-scoped entity, added to the solution as an `Entity` resource; not provisioned. Create it with `uip df entities create`. | Least-privilege record access, masked identifiers in list views and logs, row-level access for the reviewer role, and synthetic-only retention set during provisioning. A duplicate `CorrelationKey` reads the existing record. |
| Choice sets: `CbPaymentExceptionStatus`, `CbPaymentExceptionOutcome` | Enumerate case state and the reviewer outcome so state is a real field, not free text. | New choice sets, added to the solution as `ChoiceSet` resources; not provisioned. | Values read and write as integer `numberId` values. The Flow and the console translate them in both directions, so an unknown value fails validation instead of routing. |
| Data Fabric connection | Supplies the record-created trigger, the record-updated wait, and every record activity. | Connection `b2a02899-3708-4bb6-810a-02321afb77f6`, enabled, in folder `demos`; verified on August 20, 2026. It is not the default connection. | Bind the connection explicitly in each node. Validate the binding during implementation instead of trusting the recorded date. |
| Coded process app: `Payment Exception Console` | Operations cockpit and the review surface for the primary decision. It reads and writes the same record and creates no second source of truth. | Repository folder `payment-exception-console/`; coded-app package `commercial-banking-payment-exception-console`; deployed independently to a child folder of `JD_Demos/demos`; not provisioned. | OAuth PKCE sign-in, role-restricted write access, masked identifiers, required rationale, client and record-level validation of the outcome value, and no payment-system credentials. |
| Context index: `CommercialBankingPaymentPolicy` | Versioned synthetic operating procedure used by `search_payment_policy`. | New context index; corpus owner is the payment-policy owner; not provisioned. | Only approved synthetic policy content. Retrieval failure produces `insufficient_evidence` and forces human review. |
| Static screening and customer tools | Return versioned synthetic risk and screening summaries. | Local mocks initially; compliance and customer-data owners must approve any real connection. | Read-only, minimum fields, no raw watchlist data, and compliance hold on ambiguity or failure. |
| Azure AI Foundry external-agent connection | Supplies `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` on the non-material showcase branch. Its request contains only a selected agent and constant message; its response is discarded. | Shared connection `0107247a-0197-42c9-b957-05d1b722b111`; verified enabled and default in UiPath Labs Playground folder `demos` on August 12, 2026. | No case or sensitive data, no business-variable mapping, and fail-open continuation to the same merge point. Validate the node binding during Flow implementation. |
| MCP template server | Exposes only `get_approved_payment_message_template(templateId, recipientType)`. | New demo MCP server or local mock; messaging owner approves templates; not provisioned. | No send capability. Tool inputs exclude raw account identifiers and screening rationale; failure creates a manual-draft task. |
| Action Center quick form: `Sanctions disposition` | Role-restricted compliance task with a completion handle, kept so the demo shows both human-in-the-loop mechanisms. | Action Center task in the solution deployment folder; not provisioned. | Sanctions role only, masked identifiers, restricted rationale rules, and no release authority for any agent. |

### Solution boundary and layout

```text
commercial-banking/payment-exception/
├── payment-exception-solution/
│   ├── commercial-banking-payment-exception.uipx
│   ├── payment-exception-flow/
│   │   └── payment-exception-flow.flow
│   ├── payment-message-gateway/
│   ├── legacy-payment-console/
│   └── payment-update-writer/
└── payment-exception-console/
```

The solution has exactly one `.uipx` manifest and is an independent deployment boundary. The `Payment Exception Console` is a coded web app. It is not registered in the `.uipx`, so it is packed, published, and deployed on its own, and the solution configuration pins its name, version, folder, and record contract. The entity and its choice sets are solution resources. Before packaging, run `uip solution resources refresh`, restore dependencies, and run a dry-run pack. Pull requests validate only the changed solution. Publishing and deployment occur after merge to `main` through `playground-deploy`, using an immutable package version and a child folder beneath `JD_Demos/demos`.

## Human decisions

- **Reviewer and decision:** A payment operations reviewer decides `RequestInformation`, `ApproveRepair`, `ReturnOrReject`, or `EscalateCompliance`. The console review screen shows original and proposed values, deterministic findings, payment/network status, policy citations, confidence, and agent rationale.
- **Review experience:** The console reads the case record and shows case facts and evidence as read-only. It exposes only policy-allowed repair fields for editing, requires reviewer rationale, and then writes the decision envelope with one single-record update: outcome, edited repair fields, rationale, reviewer ID, and submitted timestamp. High-risk fields are not editable. A second-person approval field is required when the selected policy configuration marks the repair maker-checker controlled.
- **Compliance task:** Possible matches, incomplete high-risk party data, or an explicit escalation create a role-restricted Action Center quick form. Outcomes are `ReleaseToOperations`, `MaintainHold`, or `InvestigateFurther`; the agent cannot select or execute release. This task keeps a completion handle, so the demo shows both resumption mechanisms.
- **Timeout and resumption:** The Flow resumes from the record-updated event, confirms the event belongs to this record, reads the record, validates the outcome value and returned field IDs, and routes only from the persisted values. A parallel 30-minute demo timer assigns the case to the queue manager and leaves the record open.
- **Downstream routes:** `RequestInformation` creates a mock information request and sets `awaiting_information`; `ApproveRepair` enables the RPA repair branch; `ReturnOrReject` creates a mock return/reject response; `EscalateCompliance` or `MaintainHold` sets `compliance_hold`; `ReleaseToOperations` returns to the operations review without performing a write.

## Data Fabric process-app variant

The demo portfolio owner selected this demo as one of the three Data Fabric process-app variants on August 20, 2026. The shared rules, verified node evidence, and platform constraints are in [`reference-solution/claims-process-flow.md`](../reference-solution/claims-process-flow.md); this section states only the domain design.

### Entity schema

Entity `CommercialBankingPaymentExceptionCase`. Field names are used verbatim, because Data Fabric drops unknown keys without an error and checks required fields case-sensitively.

| Field | Type | Written by | Purpose |
| --- | --- | --- | --- |
| `Id` | UUID | Data Fabric | System primary key. It is the correlation handle for the Flow instance and the console URL. |
| `CorrelationKey` | TEXT | Insert | Unique `<caseId>:<uetr>` value. A duplicate value reads the existing record. |
| `CaseId` | TEXT | Insert | Synthetic case identifier shown in the case table. |
| `Uetr` | TEXT | Insert | Payment correlation value shown beside the case identifier. |
| `Status` | CHOICE_SET_SINGLE | Flow and console | Lifecycle and terminal state from `CbPaymentExceptionStatus`, including `review_required`, `review_submitted`, `compliance_hold`, `completed`, and `technical_exception`. |
| `Stage` | TEXT | Flow | Current canvas segment, so the console can show progress without guessing. |
| `ReceivedAt` | DATETIME_WITH_TZ | Insert | Domain receipt time used for aging. It is separate from the `CreateTime` audit column. |
| `PaymentAmount` | NUMBER | Flow | Amount shown as a case-table column. |
| `PaymentFacts` | MULTILINE_MAX | Flow | Normalized payment, party, currency, and network facts for the detail view. |
| `Classification` | TEXT | Flow | Agent classification value that segment 2 routes on. |
| `ValidationFindings` | MULTILINE_MAX | Flow | Deterministic rule results with field, actual value, and expected constraint. |
| `EvidenceSummary` | MULTILINE_MAX | Flow | Cited agent narrative with source IDs and policy citations. |
| `EvidenceConfidence` | NUMBER | Flow | Confidence value that segment 2 routes on. |
| `RequiresCompliance` | BOOL | Flow | Forces the compliance route when true. |
| `RecommendedAction` | TEXT | Flow | Advisory recommendation. It never authorizes a write. |
| `ProposedRepair` | MULTILINE_MAX | Flow | Proposed repair fields with original values, so the console can show before and after. |
| `ReviewOutcome` | CHOICE_SET_SINGLE | Console | Named reviewer outcome from `CbPaymentExceptionOutcome`. |
| `ApprovedRepair` | MULTILINE_MAX | Console | Reviewer-approved or edited repair fields. |
| `ReviewerRationale` | MULTILINE_MAX | Console | Required rationale text. |
| `ReviewerId` | TEXT | Console | Signed-in reviewer reference. |
| `ReviewSubmittedAt` | DATETIME_WITH_TZ | Console | Domain decision timestamp used for review-duration measures. |
| `WriteBackReceipts` | MULTILINE_MAX | Flow | Mock system, operation ID, status, and timestamp per receipt. |
| `AuditEvents` | MULTILINE_MAX | Flow and console | Ordered append-only events with actor, action, source references, and versions. |

### Write-back points

Every Flow write uses `uipath.connector.uipath-uipath-dataservice.update-entity-record`, and the console writes through the SDK single-record update. Both are single-record writes, so both fire record-updated events. The wait node's filter matches this case's record ID and the submitted status, so an intake or assessment write never resumes a case.

| Flow point | Fields written | Trigger effect |
| --- | --- | --- |
| Segment 1, after normalization | `Status`, `Stage`, `PaymentAmount`, `PaymentFacts`, `AuditEvents` | Fires an event that no active wait node accepts, because the status value is not `review_submitted`. |
| Segment 2, after assessment | `Classification`, `ValidationFindings`, `EvidenceSummary`, `EvidenceConfidence`, `RequiresCompliance`, `RecommendedAction`, `ProposedRepair`, `Status`, `Stage` | Same. Agent output becomes record state before any human sees it. |
| Segment 3, before the wait | `Status` set to `review_required` | Publishes the case to the console work list. |
| Console review submit | `ReviewOutcome`, `ApprovedRepair`, `ReviewerRationale`, `ReviewerId`, `ReviewSubmittedAt`, `Status` set to `review_submitted`, `AuditEvents` | The single write that resumes the Flow. |
| Segment 4, after merge | `WriteBackReceipts`, terminal `Status`, `Stage`, `AuditEvents` | Final state. The console shows the closed case and its receipts. |
| Any exception route | `Status` set to `technical_exception` or `compliance_hold`, `AuditEvents` | Keeps the record open with an owned recovery route. |

### Human-in-the-loop record change

The Flow waits on `uipath.connector.event.uipath-uipath-dataservice.record-updated`, configured with the entity in `--detail.objectName` and the same Data Fabric connection.

1. The record-created trigger already gave the instance its own record ID. The wait node filters the event on that ID, so it resumes only for this case. Set `inputs.detail.filterExpression` to ``=js:`Id == '${$vars.<triggerNodeId>.output.Id}' && Status == '<review_submitted value>'` ``, which matches the record and the submitted state in one condition.
2. The node fires once, for one case. There is no scan-and-discard loop and no marker field.
3. The Flow then reads the record with `get-entity-record-by-id` and routes from `ReviewOutcome`.
4. The demo accepts up to 60 seconds between the reviewer submitting and the Flow resuming. The demo script narrates that wait instead of hiding it.
5. The wait node's `error` handle routes to the technical-exception path, so a failed event subscription creates an owned recovery item instead of stalling the case.

The single-record read in step 3 is required, not optional: a list or query read returns a size marker for `MULTILINE_MAX` fields such as `ApprovedRepair` and `ReviewerRationale`.

`uip maestro flow node configure --detail` refuses `filterExpression` and asks for a structured `filter` tree, whose builder takes literal values only. Set `filterExpression` in the node's `inputs.detail` and validate the file; the shared contract in [`reference-solution/claims-process-flow.md`](../reference-solution/claims-process-flow.md) records the verification and the runtime check that implementation must still perform.

### Review timeout

The observed wait node exposes no timeout property, and its input definition was empty. The demo therefore does not claim a native wait timeout. A parallel 30-minute timer branch runs beside the wait node, sets an overdue flag, assigns the case to the queue manager, and leaves the record open for the reviewer. Confirming whether the node supports a native timeout is an implementation task.

### Console screens

`Payment Exception Console` is a coded web app in React and TypeScript that uses the `@uipath/uipath-typescript` SDK.

| Screen or element | Content |
| --- | --- |
| Sign-in | OAuth PKCE sign-in with the reviewer's own identity. No shared account and no stored token. |
| Dashboard landing | KPI cards for cases awaiting review, overdue cases, compliance holds, technical exceptions, and median review time. |
| Frame | Company and app header, sidebar navigation for Cases, Compliance, and Exceptions, and a user button with the signed-in reviewer and sign-out. |
| Case table | Paginated table of records with page size 25 to 50, cursor paging, and a "Showing X to Y of Z" summary. Columns are case identifier, UETR, amount, status, stage, age, and confidence. |
| Case detail | Identifiers, stage progress, payment facts, deterministic findings, cited agent evidence, confidence, recommendation, receipts, and the ordered audit trail. Tabs separate evidence, activity, and receipts; modals hold the full narrative fields and the audit entries. |
| Review hero moment | Original and proposed repair values side by side, edit fields for policy-allowed values only, a required rationale box, and the four named outcome actions. Submit writes the decision envelope in one update. |
| Chat entry | A persistent chat icon on every page opens the conversational agent for questions about the case list and the open record. |
| Theme and states | One opinionated theme built with the Apollo Vertex design library, plus explicit loading, empty, error, and permission states. A reviewer without write access sees the case read-only. |

### Record-write constraints that shape this design

- Single-record writes fire trigger events; bulk writes do not. The console and every seeding path use single-record writes.
- Choice-set fields carry integer `numberId` values. The Flow and the console translate names and integers in both directions, and an unmapped value fails validation instead of routing.
- `Id`, `CreateTime`, `UpdateTime`, `CreatedBy`, and `UpdatedBy` cannot be written, so `ReceivedAt` and `ReviewSubmittedAt` exist as domain fields.
- Unknown keys are dropped without an error, so the console validates its payload against the read schema before every write.
- `MULTILINE_MAX` fields return a size marker on list and query reads and cannot be filtered. The detail view and the Flow read them with the single-record read.
- Every list call returns one page. The case table pages through the cursor and never claims a total it did not count.

### Boundary and readiness

The entity and both choice sets are solution resources, so `uip solution resources refresh` records them in the inventory. The console deploys on its own with `uip codedapp pack`, `publish`, and `deploy`. It is not a solution project and never deploys through `uip solution`; declare it as an `App` resource so the dependency stays visible. The Data Fabric connection is verified; the entity, the choice sets, and the console are not provisioned.

## Controls and safety

| Control | Design decision | Evidence in the Flow or demo |
| --- | --- | --- |
| Routing safety | Deterministic validation precedes agent reasoning. Compliance or confidence rules force review; write paths require a validated human outcome. | Decision expressions use `RequiresCompliance`, `EvidenceConfidence`, and `ReviewOutcome` read from the record; exception paths are below the happy path. |
| Record write authority | Only the console may write `ReviewOutcome`, `ApprovedRepair`, `ReviewerRationale`, `ReviewerId`, and `ReviewSubmittedAt`. Only the Flow may write state transitions, receipts, and terminal status. No agent has any Data Fabric write tool. | Record-level access rules, the Flow's write-back table, agent tool allowlists that contain read activities only, and validation that rejects an outcome value the console did not write. |
| Access and data | Use synthetic data, least-privilege service identities, masked identifiers, read-only enrichment tools, role-restricted tasks, and mock write targets. | Connection bindings, task roles, mock endpoint configuration, and redacted trace fields are visible during implementation review. |
| Agent boundaries | Domain agents recommend and draft only. They cannot clear screening, authorize an outcome, mutate a payment, or send a message. The Azure AI Foundry showcase agent receives only constants, and its output is discarded. Claims need source IDs; missing evidence triggers manual review. | Structured output schemas, allowlisted read-only tools, prohibited-action prompt rules, a required reviewer decision on the record before any write branch, and no output edge from the showcase node into case variables. |
| Resilience | Use idempotency, typed errors, bounded transient retries, no retry after ambiguous writes, receipt reconciliation, and an owned technical-exception route. | Unique `CorrelationKey`, retry configuration, receipt merge, wait-node error handle, and queue-manager route. |

## Error paths and recovery

| Failure | Safe route | Recovery owner and condition |
| --- | --- | --- |
| Invalid message schema or unmatched UETR | Create a manual-intake item; do not invoke agents or write systems. | Payment operations corrects or rejects intake, then resubmits with the same idempotency key. |
| Policy retrieval unavailable or uncited agent output | Set `insufficient_evidence` and bypass the recommendation. | Payment operations reviews raw facts; platform owner restores the index before agent use resumes. |
| Screening ambiguity or dependency failure | Set `requiresCompliance = true` and create the compliance task. | Sanctions/AML reviewer selects a named outcome. |
| Review overdue after 30 demo minutes | Keep the record open, set the overdue flag, and assign it to the queue manager. | Queue manager reassigns or cancels according to the demo runbook. |
| Record-updated event fails or the subscription is lost | Take the wait node's `error` edge, set `technical_exception`, and keep the reviewer decision on the record. | Platform owner restores the event subscription; the Flow resumes from the persisted decision without asking the reviewer again. |
| API or RPA read failure | Retry transient reads at most twice, then set `technical_exception`. | Platform or RPA owner resolves the dependency and retries from the failed activity. |
| Ambiguous or failed mock write | Do not retry automatically and do not create a success receipt. | Queue manager reconciles target state and explicitly resumes or closes the case. |
| Communication template or safety-check failure | Create a manual-draft task; other successful receipts remain recorded. | Messaging owner supplies an approved draft before merge completion. |
| Azure AI Foundry showcase timeout, error, or unexpected response | Record only `externalAgentShowcaseStatus` in transient trace data, discard the response, and rejoin the same core route. | Demo owner may disable the branch; payment operations takes no recovery action because business state is unchanged. |

## Observability and evaluation

| Signal or test | What it proves | Expected result or threshold |
| --- | --- | --- |
| Correlation and audit | Case state, deterministic findings, agent citations, reviewer edits/outcome, tool versions, and receipts can be reconstructed. | Every trace, task, and receipt carries the record `Id` and `CorrelationKey`; the record holds an ordered audit-event list with no unmasked account identifier. |
| Record write-back completeness | The record is the canonical case, not a partial copy of Flow variables. | After each fixture run, every field in the write-back table holds a value for the route taken, and no visible console value is derived outside the record. |
| Wait-node correlation isolation | A reviewer decision on one case cannot resume another case. | Update a second record to `review_submitted` while a case waits: the waiting instance stays waiting and resumes only when its own record is submitted. |
| Flow route evaluator | The principal business claim: each case reaches the correct safe route and final status. | 5/5 initial synthetic cases match the expected review route and final status before promotion. |
| Evidence evaluator | Recommendations are bounded by supplied facts and the pinned policy. | 100% of material claims contain valid source IDs; prohibited unsupported actions occur 0 times. |
| Tool-use/trajectory evaluator | The coded writer uses the approved-template tool and no send tool. | Required template-tool call in 5/5 applicable runs; no unapproved tool calls. |
| External showcase isolation | The placeholder external agent cannot change a business decision or case result. | With the showcase flag off, on, or failing, the same fixture produces identical `PaymentExceptionCase`, route, receipts, and final status; only transient `externalAgentShowcaseStatus` may differ. |
| Receipt reconciliation | Completion never hides failed or ambiguous work. | `completed` occurs only when every required receipt is present and `succeeded`; dependency-failure cases end in `technical_exception`. |

### Synthetic evaluation set

Dataset name: `commercial-banking-payment-exception-v1`.

| Case | Expected route | Expected business output |
| --- | --- | --- |
| Complete status enquiry | Deterministic safe status-response route; bypass operations review. | Cited status summary, mock response receipt, safe message draft, and `completed`. |
| Missing creditor identifier | Operations review with `recommendedAction = request_information`. | Named requested field, due date, mock information request, and `awaiting_information`. |
| Policy-allowed remittance repair | Console review with an editable low-risk field and `ApproveRepair`. | Reviewer edit persisted on the record and consumed downstream, API and RPA receipts reconciled, and `repair_approved`. |
| Possible sanctions match | Compliance route regardless of recommendation confidence. | No repair/write action, compliance task created, and `compliance_hold` until human disposition. |
| Payment-hub timeout | Technical exception below segment 4. | No false receipt, the record remains open, and queue-manager recovery data is present. |

## Demo script

1. Show the four-segment canvas and explain that exception routes sit below the happy path.
2. Submit the policy-allowed remittance-repair fixture from the console. Show the new record, the shared `caseId` and UETR, and the Flow starting from record creation.
3. Open the investigation trace to show deterministic findings, read-only tool calls, policy citations, confidence, and the advisory recommendation, then show the same values written back to the record.
4. Open the case in the `Payment Exception Console`, compare original and proposed remittance values, edit the proposed value, enter rationale, and select `ApproveRepair`. Point out that the Flow is waiting on the record change and resumes within about a minute.
5. Return to the Flow and show the API response, RPA repair, and message-drafting branches running independently and merging.
6. Open the record in the console to show the reviewer edit, template-tool call, mock receipts, final status, and ordered audit events.
7. Point out the disabled `External agent showcase` branch and its Azure AI Foundry icon. Show that its input is static, its output is discarded, and enabling or failing the branch cannot change the payment route.
8. Run or preview the sanctions fixture to show that `requiresCompliance` overrides confidence and prevents payment write-back.

## Success measures

- **Business proof:** The demo makes reduced evidence-assembly and handoff time measurable through median/p90 resolution time, case age, human touch time, correct-triage rate, override/reopen rate, and evidence completeness. These are pilot measures, not promised savings.
- **Flow proof:** A viewer can see deterministic checks, two material agent roles, the non-material Azure AI Foundry showcase, API and RPA responsibilities, business-value routing, human authority, parallel follow-up, merge, and recovery without opening generic plumbing nodes.
- **Demo proof:** In under ten minutes, a viewer can verify the trigger, cited recommendation, reviewer correction, downstream consumption, reconciled receipts, and compliance exception.
- **Build proof:** Each project validates, the five-case evaluation set passes, every resource binding is recorded, `resources refresh` and dry-run pack succeed, and the solution is deployable with an immutable version through the changed-solution CI path.

## Reference mapping

| Reference requirement | Domain-specific implementation | Evidence or gap |
| --- | --- | --- |
| 3-4 segment topology and canvas rules | Four blue sticky notes: Receive and correlate, Investigate the payment, Control the resolution, Respond and close; happy path left-to-right and exceptions below. | Fully specified; canvas implementation remains. |
| IXP/document intelligence, when relevant | Not used because the selected intake is structured ISO 20022-like data and attachments are out of scope. | Rationale recorded; revisit only if attachments become a required input. |
| API workflow and RPA on the intended path | `payment-message-gateway` parses/sends; `legacy-payment-console` reads status and records an approved repair. | Contracts specified; mock endpoint and UI remain to build. |
| Inline agent with a wired tool | `Payment Exception Investigator` uses customer, screening, and pinned-policy tools and returns branch-driving fields. | Contract specified; resources are unverified. |
| Coded agent with visible value-add | `Payment Update Writer` retrieves an approved template, drafts a safe update, and self-checks it. | Contract specified; coded agent and template server remain to build. |
| Shared external-agent showcase | Node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` uses connection `0107247a-0197-42c9-b957-05d1b722b111` on a demo-flag branch with a static message, discarded output, and fail-open rejoin. | Node availability and connection state are verified in Playground; the Flow-specific binding and agent selection remain to validate during implementation. |
| Real business decision and safe exception | Compliance/confidence expression plus named human outcomes; schema, hold, timeout, and technical routes are explicit. | Fully specified; expressions remain to implement and evaluate. |
| Human decision and returned outcome data | The console writes the named outcome, edits, rationale, reviewer, and timestamp to the record; the Flow resumes from the record-updated event, validates the persisted values, and consumes them downstream. The sanctions quick form keeps a task completion handle. | Contract specified; maker-checker policy is an open human decision. |
| Purposeful parallelism and merge | Network response, applicable RPA write-back, and safe message draft run independently, then merge before closure. | Fully specified; receipt requirements vary by outcome and remain to encode. |
| Evaluation set and evaluator | Five synthetic cases plus route, evidence, trajectory, reconciliation, write-back, and wait-node isolation evaluators, with exact initial thresholds. | Fully specified; fixtures and evaluators remain to build. |
| Process-app variant | Selected on August 20, 2026 in issue #56. Entity `CommercialBankingPaymentExceptionCase` is the canonical record, the record-created trigger starts the Flow, the Flow writes state and agent output back, the primary review happens in `Payment Exception Console`, and the Flow resumes from a record-updated event. See [Data Fabric process-app variant](#data-fabric-process-app-variant). | Design fully specified and the connection is verified; the entity, choice sets, and console remain to build. |
| Solution boundary and delivery contract | `commercial-banking/payment-exception/payment-exception-solution/`, one `.uipx`, nested Flow layout, entity and choice sets as solution resources, independently deployed console, resource refresh, immutable version, changed-solution CI. | Fully specified; deployment folder child and resources require provisioning. |

## Open human decisions

These decisions refine implementation but do not block building the synthetic, mock-backed demo.

| Decision | Owner | Resolution path |
| --- | --- | --- |
| Choose the representative payment rail and final message subset. | Commercial payments product owner | Approve Swift cross-border, Fedwire, CHIPS, or internal-hub framing before replacing the generic fixtures. |
| Confirm repairable fields and maker-checker rules. | Payment operations control owner | Review the editable-field allowlist and identify fields requiring second-person approval. |
| Confirm sanctions disposition roles and customer-safe wording. | Sanctions/AML and legal owners | Approve task outcomes, restricted rationale rules, and message templates. |
| Confirm response deadlines and escalation SLA. | Payment operations owner | Replace the 30-minute demo timeout with the approved test policy for the selected exception types. |
| Choose representative systems and prove whether UI automation is necessary. | Enterprise architect and application owners | Review API availability; retain RPA only for a demonstrably UI-only responsibility. |
| Approve policy corpus, retention, residency, and redaction. | Policy, privacy, and records owners | Sign off the synthetic corpus and environment controls before non-synthetic testing. |
| Select the exact `JD_Demos/demos` child folder and provision remaining resources. | UiPath tenant administrator | Reuse the verified Azure AI Foundry and Data Fabric connections, discover or create the other approved resources, and record every binding in the solution. |
| Approve the entity schema, record retention, and record-level access. | Data owner and UiPath tenant administrator | Review every field, confirm synthetic-only retention, and set row-level access before the console is shared. |
| Confirm console access roles and reviewer identity. | Payment operations control owner and identity owner | Map the reviewer, queue manager, and read-only viewer roles to groups, and confirm that the signed-in identity is the recorded reviewer. |
| Confirm the acceptable resumption latency and event delivery mode. | Demo owner and platform owner | Measure the record-updated delay in `demos`, confirm the delivery mode the entity resolves to, and accept or lower the 60-second demo target. |

## Implementation tasks

1. Scaffold `commercial-banking-payment-exception` with the nested Flow project and one `.uipx` manifest.
2. Create the `CbPaymentExceptionStatus` and `CbPaymentExceptionOutcome` choice sets and the `CommercialBankingPaymentExceptionCase` entity, then add all three to the solution as `ChoiceSet` and `Entity` resources.
3. Build the synthetic fixture set, the record contract and its single-record seeding path, the mock payment endpoint, and the mock legacy console.
4. Implement and validate the API workflow and RPA operations with typed errors and receipts.
5. Build the inline investigator, versioned policy context, and structured-output evaluation.
6. Build the coded update writer, least-privilege template MCP tool, and trajectory/safety evaluators.
7. Build and deploy the `Payment Exception Console` coded web app, including the case table, detail view, review screen, chat entry, and state handling, then wire its decision write to the record.
8. Author the four-segment Flow: bind Data Fabric connection `b2a02899-3708-4bb6-810a-02321afb77f6`, configure the record-created trigger and the record-updated wait with `--detail.objectName CommercialBankingPaymentExceptionCase`, set the record-ID `filterExpression` on the wait node, add the wait `error` edge, the parallel overdue timer, the write-back points, exception routes, the non-material Azure AI Foundry showcase branch, and the parallel completion branches with their merge.
9. Confirm whether the wait node exposes a native timeout. If it does, replace the parallel timer and record the evidence; if it does not, keep the timer and record that finding.
10. Bind and validate Azure AI Foundry connection `0107247a-0197-42c9-b957-05d1b722b111`; prove the showcase branch cannot change case data, routing, receipts, or final status.
11. Prove wait-node correlation isolation: update a second record to `review_submitted` and show the waiting instance does not resume.
12. Run project validation and the five-case evaluation set; resolve all warnings and failed thresholds.
13. Refresh solution resources, restore, dry-run pack, and register immutable-version deployment configuration.

## Quality rubric

| Dimension | Score (0-3) | Evidence and remaining gap | Owner and resolution path |
| --- | ---: | --- | --- |
| Enterprise credibility | 2 | Consequential role, decision, data contract, controls, and pilot measures are explicit; no design-partner baseline or verified estate exists. | Product and operations owners validate the workflow and baseline during discovery. |
| Flow differentiation | 3 | Four segments visibly coordinate deterministic logic, agents, API, RPA, human decisions, parallel work, merge, and safe recovery. | Flow implementer preserves the specified layout and validates route expressions. |
| Demo clarity | 3 | The repair hero journey and sanctions exception have named, observable proof points and a timed script. | Demo owner rehearses both fixtures after deployment. |
| Build feasibility | 2 | Inputs, outputs, mocks, fallbacks, solution boundary, evaluation, and delivery are specified. The Data Fabric and Azure AI Foundry connections and the required Data Fabric node types are verified; the entity, the two choice sets, the console, and the remaining domain resources are not provisioned. | UiPath tenant administrator and implementers provision and record resource readiness, starting with the entity and choice sets that the trigger depends on. |
| **Total** | **10/12** | **Ready for implementation planning as a synthetic demo; not ready for production integration or deployment until owned resource gaps are resolved.** | **Start with solution and fixture scaffolding, then close the listed human decisions before replacing mocks.** |
