# Life-insurance underwriting evidence-exception demo specification

## Use case and narrative

- **Domain and solution:** `life-insurance/underwriting-evidence-exception/underwriting-evidence-exception-solution/`, with globally unique solution and package name `life-insurance-underwriting-evidence-exception`.
- **Enterprise use case:** A senior life underwriter decides whether a synthetic individual term-life application can proceed to an offer, needs more evidence, should be postponed, should be declined, or needs escalation. Flow reconciles the application, authorization, extracted documents, synthetic external-evidence responses, and versioned underwriting guidance. The hero moment is the underwriter comparing discrepancies and cited evidence, editing the proposed risk class or evidence request, and returning a named decision that controls downstream work.
- **Why this use case:** It is the top-ranked candidate in the [life-insurance opportunity research](agentic-workflow-opportunities.md), scoring 4.75/5. It combines document intelligence, consent-aware evidence acquisition, deterministic rules, bounded agent reasoning, API and legacy-UI work, consequential human authority, and an auditable outcome. The death-claim candidate is also consequential, but its investigation and beneficiary-payment complexity makes a short, credible synthetic demo harder to bound.
- **Audience journey:** The demonstrator submits a synthetic application packet, follows four named canvas segments, inspects evidence and rule citations, completes an evidence-rich underwriting task, and shows the returned decision drive independent case-record, communication, and audit work before reconciliation. Flow is the right surface because the business value is in coordinating heterogeneous actors while keeping consent stops, review routes, timeouts, and recovery visible.
- **Success outcome:** The case ends with an offer work item, a tracked evidence request, or a human-authorized adverse outcome work item. Every route preserves evidence lineage, proposed-versus-final values, reviewer authority, downstream receipts, and a queryable audit record; no agent binds coverage or issues an adverse notice.
- **Measurable value:** Pilot measures are application-to-decision elapsed time, time waiting for evidence, evidence re-request rate, straight-through eligibility rate, review-referral rate, human override rate, dependency-error rate, and audit-record completeness. Carrier baselines and targets remain human decisions; the demo makes no ROI or improvement claim.
- **Out of scope:** Actuarial pricing or model development; sales advice; premium collection; policy delivery; production medical or consumer-report access; autonomous postpone or decline; adverse-notice legal sufficiency; real applicant communication; production policy administration; non-synthetic applicant or medical records in the Data Fabric case entity; and any claim that the Data Fabric record replaces a carrier system of record.

## Personas

| Persona | Role in the demo | Authority boundary |
| --- | --- | --- |
| New-business case manager | Monitors intake, authorization, evidence retrieval, and technical exceptions. | May correct intake metadata and request missing evidence; cannot choose a final risk class or adverse outcome. |
| Senior underwriter | Reviews reconciled evidence and makes the primary underwriting decision. | May approve or edit an offer, request evidence, postpone, decline, or escalate; postpone and decline require a separate adverse-outcome control before communication work. |
| Underwriting supervisor | Receives timed-out tasks and performs the second-person adverse-outcome control. | May approve, return, or escalate a proposed postpone/decline; cannot erase the first review or source evidence. |
| Compliance/model-risk reviewer | Inspects citations, versions, overrides, access, and evaluation evidence. | Defines governance requirements and investigates exceptions; does not decide an individual application in this demo. |
| Demonstrator | Submits fixtures and narrates the normal, review, consent-stop, and recovery paths. | Uses synthetic application, health, consumer-report, and rule data only. |

## Trigger and case contract

This demo is one of the three Data Fabric process-app variants selected on August 20, 2026. A new case record starts the Flow, so the run is still repeatable: the demonstrator submits a fixture from the `Underwriting Evidence Console`, which inserts one record. A carrier new-business event may replace that action after the carrier system and event ownership are chosen. `applicationId` is the correlation and idempotency key. An exact replay reads the existing record and receipts; a changed payload for the same `applicationId` creates a typed `INPUT_VERSION_CONFLICT` instead of overwriting evidence or decisions.

| Item | Specification |
| --- | --- |
| Trigger | `uipath.connector.trigger.uipath-uipath-dataservice.record-created` on entity `LifeUnderwritingCase`. Pass the entity in `--detail.objectName` when configuring the node, and bind Data Fabric connection `b2a02899-3708-4bb6-810a-02321afb77f6` in folder `demos` explicitly, because it is not the default connection. The record must be created by a single-record insert; a bulk insert fires no event. The `underwriting-evidence-gateway` API workflow validates and normalizes the payload before any evidence acquisition. |
| Canonical record | Data Fabric entity `LifeUnderwritingCase`. The record `Id` is the instance correlation value and `applicationId` is the unique business reference. One record holds the normalized input, authorization, evidence state, recommendation, both review stages, receipts, exceptions, and audit references. There is no parallel Orchestrator queue for the same case. |
| Required inputs | Application event and applicant facts, authorization, document references, carrier-rule version, and `showExternalAgentShowcase`. Only synthetic data is allowed. |
| Sensitive data | Applicant and medical facts are restricted synthetic data. Agents receive derived facts or allowlisted excerpts when raw identifiers are unnecessary. Tasks and traces mask applicant, producer, and source identifiers. |
| Outputs | Final route and risk class when applicable, required evidence, cited findings, reviewer result, communication preview, system receipts, typed exceptions, audit artifact, and final status. |

### Input schema

| Field | Type | Required | Contract |
| --- | --- | --- | --- |
| `applicationId` | string | yes | Stable unique correlation and record reference, for example `LI-UW-00042`. |
| `receivedAt` | ISO 8601 string | yes | Starts processing and evidence-wait measurements. |
| `productCode` | string | yes | Must be in the synthetic term-life product allowlist. |
| `faceAmount` | number | yes | Positive synthetic amount in the configured product band. |
| `jurisdiction` | string | yes | Two-letter US state code supported by the fixture rule set. |
| `producerId` | string | yes | Synthetic producer reference; masked outside the canonical record. |
| `channel` | enum | yes | `producer`, `direct`, or `partner`. |
| `applicantFacts` | object | yes | Synthetic `applicantId`, age, residence, tobacco answer, identity status, declared conditions, and medications. |
| `authorization` | object | yes | `status`, allowed `sourceTypes`, `signedAt`, `expiresAt`, and `revokedAt`; acquisition requires `status === "signed"`, current validity, no revocation, and source scope. |
| `documentRefs` | array | yes | One or more synthetic file references with `documentType`, hash, and received timestamp. |
| `rulesVersion` | string | yes | Pinned synthetic underwriting-guide version, initially `UW-GUIDE-2026.3`. |
| `showExternalAgentShowcase` | boolean | no | Defaults to `false`; controls only the non-material Azure AI Foundry branch. |

### Output schema

| Field | Type | Contract |
| --- | --- | --- |
| `status` | enum | `offer_work_created`, `evidence_requested`, `adverse_control_pending`, `adverse_work_created`, `escalated`, `technical_exception`, or `completed`. |
| `route` | enum | `OFFER`, `OFFER_WITH_RATING`, `REQUEST_EVIDENCE`, `POSTPONE`, `DECLINE`, or `ESCALATE`. |
| `proposedRiskClass` | string or null | Synthetic agent/rule recommendation; never the final binding decision. |
| `finalRiskClass` | string or null | Returned human value or deterministic straight-through fixture value; retained beside the proposal. |
| `requiredEvidence` | array | Source type, reason code, due-state, and evidence request reference. |
| `evidenceFindings` | array | `findingCode`, severity, `sourceRef`, `ruleRef`, confidence, and contradiction status. |
| `recommendation` | object | Proposed route/risk class, rationale summary, confidence, evidence completeness, material-inconsistency flag, and rule/model versions. |
| `humanDecision` | object or null | Stage, named outcome, edited values, reason code, rationale, reviewer role, reviewer reference, and timestamp. |
| `secondReview` | object or null | `approved`, `returned`, or `escalated` result for postpone/decline, with distinct reviewer and rationale. |
| `communications` | array | Audience, approved template/version, preview reference, citation status, and `drafted` or `manual_required`; never a send receipt. |
| `actionReceipts` | array | Actor/system, operation, status, correlation ID, timestamp, and error reference. |
| `exceptions` | array | Typed code, owning role, recovery condition, attempt count, and state. |
| `audit` | object | Rules/model/prompt/tool versions, trace ID, timestamps, input/output hashes, ordered event references, and audit artifact URI. |

## Flow topology

Use four blue sticky notes with the titles below. The happy path runs left to right. Intake, authorization, extraction, evidence, review, timeout, and write-back exceptions sit below their originating segment. Independent completion branches are visually symmetric and merge before reconciliation. A separately labelled `External agent showcase` branch sits below segment 2 and never receives application, applicant, evidence, rule, reviewer, or receipt data.

| Reference segment | Domain-specific canvas title | Actors and business output | Branch or merge evidence |
| --- | --- | --- | --- |
| Receive and understand | **1. Receive and validate application** | The record-created trigger starts the Flow and supplies the record `Id`; `underwriting-evidence-gateway` validates and normalizes the record; IXP extracts the synthetic application and medical packet; the Flow writes the normalized state back to the same record. Output: normalized `UnderwritingCase` and extraction evidence. | Unsupported product/jurisdiction or invalid schema routes to intake exception. Invalid, expired, revoked, or out-of-scope authorization stops before external evidence calls. Any required-field confidence below `0.90` sets `requiresEvidenceReview = true`. |
| Assess and enrich | **2. Gather and reconcile evidence** | API workflow reads synthetic prescription, MVR, prior-application, and EHR contracts; RPA reads a local legacy evidence portal; inline `Underwriting Evidence Reconciler` retrieves approved guide passages; coded `Medical Evidence Chronology` creates a cited timeline. | `evidenceComplete`, `materialInconsistency`, `recommendation.confidence`, and `straightThroughEligible` drive routing. The external showcase branch rejoins with `UnderwritingCase` unchanged. |
| Decide and review | **3. Recommend and underwrite** | Deterministic rules send only clean, authorized, complete offer cases to the synthetic straight-through route. Other material cases set the record to `awaiting_primary_review`, and the senior underwriter decides in the `Underwriting Evidence Console`. The Flow waits at the record-updated event node and resumes from the persisted decision. Postpone or decline then creates the `Underwriting Adverse Control Review` Action Center task for a distinct supervisor. | `straightThroughEligible === true && evidenceComplete === true && materialInconsistency === false && extractionConfidence >= 0.90 && recommendation.route === "OFFER" && recommendation.confidence >= 0.90` permits the synthetic straight-through offer route. The wait node filters the event on this case's record ID and the submitted status, so another underwriter's decision cannot resume it. Human routes then use the persisted `humanDecision.outcome`; `POSTPONE` and `DECLINE` require `secondReview.result === "approved"`. Thresholds are demo configuration, not carrier policy claims. |
| Act and communicate | **4. Record decision and notify** | In parallel, RPA writes an approved offer/adverse work item to a local legacy policy-admin UI, API workflow appends the authoritative audit event, and coded agent produces an approved-template status preview. The branches merge, receipts reconcile, and the Flow writes the final status and receipts to the case record. Evidence-request routes create a tracked request instead of a policy-admin decision write. | `requiredReceipts.every(receipt => receipt.status === "succeeded")` is required for `completed`. Missing or ambiguous writes set `technical_exception`; no route reports completion before merge, reconciliation, and the final record write. |

## Agentic reasoning and tool use

| Actor | Visible responsibility | Structured input and output | Tools, context, and guardrails | Readiness/fallback |
| --- | --- | --- | --- | --- |
| Inline low-code agent: `Underwriting Evidence Reconciler` | Compare declared facts with normalized evidence and approved guide passages, then recommend a bounded route. | Input: derived applicant facts, evidence statuses/findings, product/jurisdiction, and rules version. Output: proposed route/risk class, `evidenceComplete`, `materialInconsistency`, confidence, finding codes, `sourceRefs`, and `ruleRefs`. | Read-only `search_underwriting_guide(query, productCode, jurisdiction, rulesVersion)` and `get_normalized_evidence(applicationId, allowedSourceTypes)` tools. No raw document retrieval, coverage binding, final pricing, request sending, or adverse authority. Deterministic validation rejects unknown enums, uncited findings, and unauthorized source use. | Not provisioned. Versioned tool-response fixtures implement the same contract. Tool/grounding failure returns `insufficient_grounding` and forces human review. |
| Coded agent: `Medical Evidence Chronology` | Turn authorized source excerpts into a compact, reviewer-facing chronology without diagnosing or deciding risk. | Input: application ID, allowlisted source references, and date range. Output: dated events, contradiction flags, citations, omitted-source list, and self-check result. | LangGraph agent calls only `get_authorized_evidence_excerpt(sourceRef, applicationId)` through a least-privilege UiPath MCP server and self-checks every event against a returned source reference. It cannot query unlisted sources or emit a route/risk class. | Not provisioned. A deterministic chronology fixture is the fallback. Missing citations create `manual_chronology_required` but do not fabricate events. |
| External agent showcase: `Azure AI Foundry connectivity` | Display external-agent connectivity without performing underwriting work. | Connection-selected `agent_id` and constant message `UiPath Flow external-agent connectivity showcase for life insurance`; omit `thread_id`. Discard the response. | Node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread`, connection `0107247a-0197-42c9-b957-05d1b722b111`. Do not pass application, applicant, producer, authorization, document, evidence, rule, recommendation, reviewer, communication, receipt, or audit data. | Reverified August 12, 2026 in `uipathlabs / Playground`: the node is tenant-available; the connection is enabled, default, active, and in folder `demos`. The branch is disabled by default; a short timeout, error, or unexpected response records only transient showcase status and rejoins the unchanged core route. Validate the Flow-specific binding and selected agent during implementation. |

## Data and resources

| Resource or data dependency | Purpose and contract | Owner/folder/readiness | Security and failure handling |
| --- | --- | --- | --- |
| IXP project: `LifeUnderwritingPacketExtraction` | Classify and extract application facts, authorization dates/scope, exam/lab fields, attending-physician-statement fields, source page, and per-field confidence from synthetic PDFs. | New project/model deployed with the solution under the approved `JD_Demos/demos` parent; not provisioned. | Synthetic restricted fixtures only. A required-field confidence below `0.90`, malformed page, or cross-document conflict forces evidence review; versioned JSON preserves the output contract offline. |
| API workflow: `underwriting-evidence-gateway` | Operations `normalize_application`, `retrieve_evidence`, `create_evidence_request`, and `append_audit_event` return canonical schemas and deterministic mock receipts. | New sibling project; carrier/vendor APIs and connections are unverified. | Authorization is evaluated before each source request. Responses distinguish `notFound`, `notAuthorized`, `timeout`, and `providerError`; no error becomes a clean result. |
| RPA: `legacy-underwriting-console` | Operations `read_evidence_status` and `write_decision_work_item` use a local legacy UI fixture and return screenshot-linked receipts. | New sibling project; local mock application and unattended runtime are not provisioned. | UI automation exists only to demonstrate a verified API gap in the future carrier system. Ambiguous writes are not automatically retried or reported successful; recovery requires target-state reconciliation. |
| Data Fabric entity: `LifeUnderwritingCase` | Canonical demo case, idempotency, state transitions, agent outputs, both review stages, outputs, and audit references. Replaces the Orchestrator queue. | New tenant-scoped entity registered as a solution resource of kind `Entity`; not provisioned. | Least-privilege access, masked applicant and producer identifiers, restricted-field list, configured retention, append-only evidence and decision history, and version-conflict handling. |
| Choice sets: `LiUnderwritingStatus` and `LiUnderwritingOutcome` | Enumerate case status and named decision outcomes so the app, the Flow, and the event filter share one value list. | New choice sets registered as solution resources of kind `ChoiceSet`; not provisioned. | Values read and write as integer `numberId` values. A value list change is a schema change and needs a matching filter and app update. |
| Data Fabric connection | Binds the record-created trigger, the record-updated wait, and every record read or write node. | Connection `b2a02899-3708-4bb6-810a-02321afb77f6`, enabled, in folder `demos` on August 20, 2026. It is not the default connection. | Bind the connection explicitly in each node. Least-privilege scope, no non-synthetic records, and Flow-specific binding validation during implementation. |
| Coded process app: `Underwriting Evidence Console` | Operational console and primary review surface. Reads and updates the same record and creates no competing source of truth. Package name `life-insurance-underwriting-evidence-console`. | Coded web app deployed independently with `uip codedapp pack`, `publish`, and `deploy`; not provisioned and not registered in the `.uipx`. | Role-restricted sign-in, masked identifiers, server-side enum and rationale validation, read-only source facts, and no policy-admin or vendor credentials. |
| Synthetic evidence services | Contract-shaped prescription, MVR, prior-application, and EHR responses with source, timestamp, consent basis, status, and version. | Repository fixtures and local mock service to be built; no vendor marks or connectivity claims. | Per-source timeouts/retries, provenance, and no silent substitution. Product-rule fixture decides whether a partial case may continue or must request evidence. |
| Context index: `LifeUnderwritingGuide` | Versioned synthetic product eligibility, evidence requirements, referral thresholds, and guide excerpts for cited retrieval. | New Context Grounding index; not provisioned. | Read-only approved content. Missing or mismatched `ruleRef` forces review; rule version is persisted on recommendations and decisions. |
| MCP evidence server | Exposes only `get_authorized_evidence_excerpt(sourceRef, applicationId)` for the chronology agent. | New demo MCP server or deterministic mock; not registered. | Enforces application/source allowlists, returns synthetic excerpts plus hashes, logs exact calls, and has no search-all or write tools. |
| Azure AI Foundry connection | Supplies the non-material showcase node with a selected agent and constant message; output is discarded. | Shared connection `0107247a-0197-42c9-b957-05d1b722b111`; active, enabled, and default in Playground folder `demos` on August 12, 2026. | No case or sensitive data, no business-variable mapping, fail-open continuation to the same route, and Flow-specific binding validation during implementation. |
| Coded action app: `Underwriting Adverse Control Review` | Second-person adverse control only. Shows the primary decision, cited evidence, and proposed adverse outcome, and returns `ApproveAdverse`, `ReturnForReview`, or `EscalateAdverse` through its task completion handle. | Deployed independently and referenced by Flow; not provisioned. | Role-restricted, masked identifiers, required rationale, enforced distinct reviewer, and no system credentials. |

Read-only inspection on August 12, 2026 used `/opt/homebrew/bin/uip` version
1.199.0, authenticated to `uipathlabs / Playground`. Tenant registry search
confirmed the Azure AI Foundry node is available, connection discovery found the
specified default connection in folder `demos`, and connection ping reported it
active. A second read-only inspection on August 20, 2026 with the same CLI
version confirmed the Data Fabric connector, the enabled connection in folder
`demos`, the tenant-available record-created trigger and record-updated event
nodes, and the record read and write activities. No Flow, IXP, entity, choice
set, index, agent, app, MCP, API, RPA, or runtime resource was created or
validated; those remain mocks or owned provisioning gaps.

### Solution boundary and layout

```text
life-insurance/underwriting-evidence-exception/
├── underwriting-evidence-exception-solution/
│   ├── life-insurance-underwriting-evidence-exception.uipx
│   ├── underwriting-evidence-exception-flow/
│   │   └── underwriting-evidence-exception-flow.flow
│   ├── underwriting-evidence-gateway/
│   ├── legacy-underwriting-console/
│   └── medical-evidence-chronology/
├── underwriting-adverse-control-review/
└── underwriting-evidence-console/
```

The solution has exactly one `.uipx` manifest and is independently deployable. Two user interfaces sit outside it and deploy independently: the `Underwriting Adverse Control Review` coded action app and the `Underwriting Evidence Console` coded process app. Neither is registered in the `.uipx`, so solution configuration pins each one's name, version, deployment folder, and record or action contract. The `LifeUnderwritingCase` entity and its two choice sets are solution resources. Before packaging, run `uip solution resources refresh`, restore dependencies, and dry-run pack. Pull requests validate only the changed solution; publish and deploy occur only after merge to `main` through `playground-deploy`, using an immutable package version under the approved Playground parent `JD_Demos/demos`.

## Human decisions

- **Primary review:** The senior underwriter works in the `Underwriting Evidence Console` on the case-detail screen. It shows normalized application facts, authorization status, extracted fields and source pages, evidence provenance/status, chronology, discrepancies, missing evidence, guide citations/version, recommendation/confidence, and prior audit events.
- **Editable values:** Proposed risk class, required-evidence items, reason code, and rationale. Application/source facts, authorization, citations, confidence, and prior events remain read-only.
- **Submission and resumption:** The console writes the decision envelope to the case record with one single-record update, which fires the record-updated event. The Flow resumes from that event, validates the written field IDs, enums, reviewer role separation, and required rationale, persists proposed-versus-final differences, and routes only from persisted record values. An invalid envelope is rejected, audited, and returned to the console for correction.
- **Primary outcomes:** `Approve`, `ApproveEdited`, `RequestEvidence`, `Postpone`, `Decline`, and `Escalate`. Every outcome requires a reason code; edited, postpone, decline, and escalate outcomes require free-text rationale.
- **Adverse control:** `Postpone` and `Decline` create the `Underwriting Adverse Control Review` Action Center task for a distinct underwriting supervisor. Outcomes are `ApproveAdverse`, `ReturnForReview`, and `EscalateAdverse`. This stage still resumes from the task completion handle, so the demo shows both human-in-the-loop mechanisms. No adverse communication work begins without `ApproveAdverse`.
- **Timeout:** Timeout values remain carrier-owned configuration. Expiry assigns the case to an underwriting supervisor pool and changes no coverage status. The primary stage uses a parallel timer path beside the wait node; the adverse-control task uses its own task timeout.
- **Downstream authority:** Agents recommend or draft only. Only validated deterministic rules or returned human outcomes enable evidence requests, policy-admin work items, or communication previews. No component sends an applicant message in the demo.

## Data Fabric process-app variant

This demo is one of the three selected Data Fabric process-app variants; the decision closed on August 20, 2026 and the shared contract is in [`reference-solution/claims-process-flow.md`](../reference-solution/claims-process-flow.md).

### Entity schema

Entity `LifeUnderwritingCase`. The schema stays demo-grade: every field below has a visible purpose in the Flow, the console, or the audit story.

| Field | Type | Written by | Purpose |
| --- | --- | --- | --- |
| `Id` | UUID | Data Fabric | Row key and the Flow's instance correlation value. |
| `applicationId` | TEXT | Console insert | Unique business reference and idempotency key. |
| `caseStatus` | CHOICE_SET_SINGLE | Flow and console | Current state from `LiUnderwritingStatus`. The set holds every `status` value in the output schema plus the in-flight values `intake`, `evidence_gathering`, `awaiting_primary_review`, `primary_decision_submitted`, and `supervisor_pool`. The event filter uses the single literal `primary_decision_submitted`. |
| `route` | TEXT | Flow | Final route, validated against the `route` enum in the output schema. It stays TEXT so the two choice sets carry only status and human outcome. |
| `productCode` | TEXT | Console insert | Product allowlist check and console filter. |
| `jurisdiction` | TEXT | Console insert | Rule-set selection and console filter. |
| `faceAmount` | NUMBER | Console insert | Band check and KPI card value. |
| `authorizationState` | TEXT | Flow | `signed`, `expired`, `revoked`, or `out_of_scope`; gates evidence acquisition. |
| `authorizationExpiresAt` | DATETIME_WITH_TZ | Console insert | Validity check before each source request. |
| `extractionConfidence` | NUMBER | Flow | Lowest required-field IXP confidence; drives evidence review. |
| `evidenceComplete` | BOOL | Flow | Branch-driving completeness result. |
| `materialInconsistency` | BOOL | Flow | Branch-driving discrepancy result. |
| `evidenceFindings` | MULTILINE_MAX | Flow | Finding codes with `sourceRef`, `ruleRef`, severity, and confidence. |
| `recommendation` | MULTILINE_MAX | Flow | Agent rationale summary and cited guide passages. |
| `recommendationConfidence` | NUMBER | Flow | Confidence used in the straight-through expression. |
| `proposedRiskClass` | TEXT | Flow | Agent or rule proposal; never the binding decision. |
| `versionStamp` | TEXT | Flow | Rules, model, prompt, and tool versions for the audit record. |
| `primaryOutcome` | CHOICE_SET_SINGLE | Console | Named senior-underwriter outcome from `LiUnderwritingOutcome`: `Approve`, `ApproveEdited`, `RequestEvidence`, `Postpone`, `Decline`, or `Escalate`. |
| `finalRiskClass` | TEXT | Console | Reviewer value, retained beside `proposedRiskClass`. |
| `primaryDecisionDetail` | MULTILINE_MAX | Console | Reason code, rationale, and edited evidence items. |
| `primaryReviewerRef` | TEXT | Console | Masked reviewer reference used for separation of duty. |
| `primarySubmittedAt` | DATETIME_WITH_TZ | Console | Domain decision time. It is a custom field because `UpdateTime` cannot be written. |
| `secondReviewResult` | TEXT | Flow from the adverse-control task | `approved`, `returned`, or `escalated`. |
| `secondReviewerRef` | TEXT | Flow from the adverse-control task | Must differ from `primaryReviewerRef`. |
| `receipts` | MULTILINE_MAX | Flow | System, operation, status, correlation ID, and timestamp per write. |
| `exceptions` | MULTILINE_MAX | Flow | Typed code, owning role, recovery condition, and state. |
| `auditRef` | TEXT | Flow | Trace ID and audit artifact URI. |

Applicant and medical facts stay restricted: the record holds derived facts, masked identifiers, and source references, not raw applicant, producer, or medical values.

### Write-back points

Every Flow write uses `uipath.connector.uipath-uipath-dataservice.update-entity-record`, which is a single-record write and fires a record-updated event. Only the submitted-decision write matches the wait node's filter, so the other writes cannot resume a waiting instance.

| Flow point | Fields written | Trigger effect |
| --- | --- | --- |
| Intake validated | `caseStatus`, `authorizationState`, `versionStamp` | Event fires and no wait node accepts it. |
| Extraction complete | `extractionConfidence`, `evidenceFindings` | Event fires and no wait node accepts it. |
| Evidence reconciled | `evidenceComplete`, `materialInconsistency`, `evidenceFindings`, `recommendation`, `recommendationConfidence`, `proposedRiskClass`, `caseStatus` | Event fires and no wait node accepts it. Status becomes `awaiting_primary_review`. |
| Primary decision submitted | `primaryOutcome`, `finalRiskClass`, `primaryDecisionDetail`, `primaryReviewerRef`, `primarySubmittedAt`, `caseStatus` | Written by the console with one single-record update. This is the event the wait node accepts. |
| Adverse control returned | `secondReviewResult`, `secondReviewerRef`, `caseStatus` | Written by the Flow after the task completes. |
| Completion or exception | `route`, `receipts`, `exceptions`, `caseStatus`, `auditRef` | Final state for the console and the audit record. |

### Human-in-the-loop record change

The primary review resumes from a record change, not a task completion handle.

1. The Flow pauses at `uipath.connector.event.uipath-uipath-dataservice.record-updated`, configured with `--detail.objectName LifeUnderwritingCase` and the `demos` Data Fabric connection.
2. The record-created trigger already gave the instance its own record ID, so the node filters the event on that ID and resumes only for this case. Set `inputs.detail.filterExpression` to ``=js:`Id == '${$vars.<triggerNodeId>.output.Id}' && caseStatus == '<primary_decision_submitted value>'` ``, which matches the record and the submitted state in one condition. `uip maestro flow node configure --detail` refuses `filterExpression` and asks for a structured `filter` tree whose builder takes literal values only, so set `filterExpression` in the node's `inputs.detail` and validate the file. The shared contract records the verification and the runtime check implementation must still perform.
3. The node fires once, for one case, with no scan-and-discard loop and no marker field. The Flow reads the full record with `get-entity-record-by-id`, because `MULTILINE_MAX` fields return only a size marker on list and query reads.
4. The Flow then validates the decision envelope and routes from the persisted values.
5. The demo accepts a resumption delay of up to two minutes and narrates it as event delivery, not Flow latency. Event mode resolves per entity at configure time and must be recorded then.
6. The wait node's `error` handle routes to the owned `EVENT_WAIT_FAILURE` exception, so a delivery failure does not leave the case silently pending.

The adverse-control stage keeps its Action Center task and completion handle. A postpone or decline cannot proceed without `secondReviewResult === "approved"` from a reviewer whose reference differs from `primaryReviewerRef`.

### Primary-review timeout

The record-updated node exposes no timeout property, so the spec claims none. A parallel timer path beside the wait node owns expiry: on the demo timeout it sets `caseStatus` to the supervisor-pool value, assigns the case to the underwriting supervisor pool, and changes no coverage state, risk class, or route. Validating whether the node offers a native timeout is an implementation task.

### Console screens

`Underwriting Evidence Console` is a coded web app in React and TypeScript that uses the `@uipath/uipath-typescript` SDK.

| Screen or element | Content |
| --- | --- |
| Sign-in | OAuth PKCE sign-in with a permission-denied state for a user without the underwriter or supervisor role. |
| Dashboard landing | KPI cards for cases awaiting primary review, cases in adverse control, evidence requests open, and median time to decision. |
| Frame | Company and app header, sidebar navigation, and user button on every page. |
| Case table | Paginated list with page size 25 to 50, next and previous controls, and a "Showing X to Y of Z" summary. Filters are product, jurisdiction, status, and aging. |
| Case detail | Identifiers, progress through the four segments, agent activity with cited findings, and the hero moment: proposed against final risk class with the guide citation beside each edited value. |
| Progressive disclosure | Tabs for evidence, chronology, decision history, and audit. Modals for the decision form and for source excerpts. |
| Chat entry | Persistent conversational-agent icon on every page for questions about the case record. |
| Theme and states | Apollo Vertex theme with consistent light and dark tokens, plus explicit loading, empty, error, and permission states. |

### Record-write constraints

- Single-record writes fire trigger events; bulk writes do not. Fixture seeding and every decision write use single-record writes.
- Choice-set values read and write as integer `numberId` values. The console and the Flow both translate names to integers and back.
- `Id`, `CreateTime`, `UpdateTime`, `CreatedBy`, and `UpdatedBy` cannot be written, so `primarySubmittedAt` and any other domain timestamp needs its own field.
- Unknown keys are dropped without an error and required-field checks are case-sensitive. Read the schema and use field names verbatim.
- `MULTILINE_MAX` fields return a size marker on list and query reads and cannot be filtered. Read them one record at a time.
- Every list call returns one page. The console pages through the cursor and never claims a total from a single call.

### Boundary and readiness

The console deploys independently with `uip codedapp pack`, `publish`, and `deploy`. It is not a solution project and never deploys through `uip solution`; declare it as an `App` resource so the dependency stays visible. The entity and both choice sets are solution resources. The Data Fabric connection is verified; the entity, the choice sets, and the console are not provisioned.

## Controls and safety

| Control | Design decision | Evidence in the Flow or demo |
| --- | --- | --- |
| Authorization before acquisition | Validate signed state, source scope, expiry, and revocation before each external-evidence call; invalid authorization stops acquisition. | Authorization decision before API/RPA evidence nodes and an evaluator asserting zero calls for the expired fixture. |
| Adverse-outcome safety | No autonomous postpone/decline; a named primary outcome plus a distinct adverse control is required. | Persisted primary decision fields, the adverse-control task completion handle, the reviewer-reference check, and the second-review audit event. |
| Record write authority | Only the console writes the primary decision fields. Only the Flow writes state, agent output, receipt, exception, and second-review fields, and it writes the second-review fields only from a completed task. No agent writes any decision, route, or status field. | Field-level write map in the variant section, app role restriction, agent tool allowlists without write tools, and a write-back evaluator. |
| Grounded recommendation | Findings require resolvable `sourceRef` and `ruleRef`; unknown enums, missing citations, or insufficient evidence force review. | Structured-output validator, Context Grounding tool, and citation evaluator. |
| Access and data | Synthetic restricted data, least-privilege identities, scoped tools, managed connections, masked logs/tasks, and configured retention. | Fixture inventory, connection bindings, tool allowlists, task roles, and redaction assertions. |
| Agent boundaries | Reconciler recommends only; chronology agent neither diagnoses nor routes; communication output is preview only; no agent writes policy state or a decision field. | Prompt/tool schemas, absent write/send tools, deterministic route validation, and human-only decision fields on the record. |
| Receipt truth | Required writes return unambiguous receipts; failed or unknown target state cannot become completion. | Merge followed by receipt-reconciliation expression and owned recovery exception. |
| External showcase isolation | Azure AI Foundry receives constants only and cannot affect case state, routing, decisions, writes, communications, receipts, audit, or status. | No business-variable mappings and an off/on/failing isolation evaluation. |

## Error paths and recovery

| Failure | Safe route | Recovery owner and condition |
| --- | --- | --- |
| Invalid schema, unsupported product/jurisdiction, duplicate conflict | Set `INTAKE_EXCEPTION`; do not call IXP, evidence, or agent actors. | New-business case manager corrects the payload or resolves the version conflict, then resubmits with the same application ID. |
| Missing, expired, revoked, or out-of-scope authorization | Set `REQUEST_AUTHORIZATION`; make zero external-evidence calls and create a tracked request. | Case manager resumes only after a new valid authorization version is recorded. |
| IXP low confidence, malformed document, or conflicting extraction | Preserve source pages and force evidence review; do not make an adverse recommendation. | Case manager or underwriter corrects/accepts fields through the governed task. |
| Evidence source timeout or provider error | Retry a transient read at most twice, preserve the exact source status, and follow the product-rule fixture to review or request evidence. | Integration owner restores the mock/source; case manager resumes from the failed source without converting the result to `notFound`. |
| Agent tool unavailable or uncited output | Ignore the recommendation, retain deterministic facts, and force human review. | Platform owner restores the resource; underwriter may proceed only from source-linked evidence. |
| Review timeout or invalid decision envelope | Keep the case pending with no coverage-state change and assign the supervisor pool. | Supervisor completes the review in the console or completes the adverse-control task; an invalid envelope is rejected, audited, and returned for correction. |
| Record-updated event delivery failure | Take the wait node `error` handle to `EVENT_WAIT_FAILURE`; do not treat the case as decided. | Platform owner restores event delivery; the case resumes from the persisted record state without re-running earlier work. |
| Ambiguous or failed RPA/API write | Do not retry an unknown write automatically or create a success receipt; set `technical_exception`. | Operations reconciles target state, records the result, and explicitly resumes the failed branch. |
| Communication chronology/template failure | Create a manual-preview work item; preserve decision and system receipts without claiming communication completion. | New-business communications owner supplies approved copy. |
| Audit append failure | Block final completion and create an operations incident. | Platform owner restores persistence and replays the idempotent audit append before merge reconciliation completes. |
| Azure AI Foundry timeout, error, or unexpected response | Record only transient `externalAgentShowcaseStatus`, discard the response, and rejoin the exact same core route. | Demo owner may disable the branch; business users take no recovery action because business state is unchanged. |

## Observability and evaluation

| Signal or test | What it proves | Expected result or threshold |
| --- | --- | --- |
| Correlation and audit | Application, authorization, evidence, tools, recommendation, human changes, writes, and recovery can be reconstructed. | Every trace, task, tool call, receipt, and audit event contains the record `Id` and `applicationId`; rules/model/prompt/tool versions and input/output hashes are present; no unmasked applicant identifier appears outside approved storage. |
| Record write-back completeness | The record holds every state transition and agent output, so the console never derives a value the Flow did not write. | After each fixture run, all six write-back points are present on the record in order, and the console displays only stored fields. |
| Wait-node correlation isolation | A record change on another case cannot resume this case. | With two open cases, submitting a decision on case B leaves case A waiting; case A resumes only when its own record is updated. Zero cross-case resumptions in 5/5 attempts. |
| Flow route evaluator | Each fixture reaches the correct safe route and final status. | 5/5 initial synthetic cases match expected route, human stage, and status before promotion. |
| Authorization/tool-call evaluator | Restricted sources are never accessed without valid scope. | Zero evidence or chronology calls for the expired-authorization case; every other source call is allowlisted and logged. |
| Grounding evaluator | Material recommendation/chronology claims resolve to evidence and approved rules. | 100% of material findings and chronology events have valid fixture `sourceRef`; every recommendation has a valid `ruleRef`; zero invented citations. |
| MCP trajectory evaluator | The chronology agent uses only its least-privilege evidence tool. | Required tool called for every requested source reference; zero calls outside the provided allowlist and zero write tools. |
| Adverse-control evaluator | A postpone or decline cannot bypass accountable human review. | Every adverse fixture records primary and distinct second reviewer outcomes before an adverse work item; zero autonomous adverse completions. |
| External showcase isolation | The placeholder external agent cannot change business results. | With the branch off, on, or failing, identical `UnderwritingCase`, route, decisions, writes, previews, receipts, audit events, and final status; only transient showcase status may differ. |
| Receipt reconciliation | Completion never hides a missing or ambiguous write. | `completed` only when every route-required receipt is present and `succeeded`. |

### Synthetic evaluation set

Dataset name: `life-insurance-underwriting-exceptions-v1`.

| Case | Expected route | Expected business output |
| --- | --- | --- |
| Clean, authorized, complete application | Synthetic straight-through `OFFER`; no wait at the event node. | Cited recommendation, final fixture risk class, policy-admin offer work item, status preview, audit receipt, and `completed` on the case record. |
| Medication declaration discrepancy | Primary review in the console with `ApproveEdited`; the Flow resumes from the record change. | Prescription fixture and guide rule are cited; proposed and final risk classes differ on the record and the override rationale is persisted. |
| Expired authorization | `REQUEST_AUTHORIZATION` before evidence acquisition. | Zero external-evidence/MCP calls; tracked authorization request and `evidence_requested`. |
| Prescription endpoint timeout | Product-rule fixture routes to `RequestEvidence`. | Status remains `timeout`, never `notFound`; request receipt, retry count, and owner are audited. |
| Material adverse evidence | Primary `Decline` in the console, then distinct `ApproveAdverse` in the Action Center task. | Two reviewer references on the record, reason code and rationales, adverse work item and preview, no autonomous or sent notice, and reconciled audit evidence. |

## Demo script

1. Show the four-segment canvas, consent stop, review route, completion branches, and exceptions below the happy path.
2. Submit the medication-discrepancy fixture from the console and point out the new case record, `applicationId`, authorization scope, packet confidence, and pinned rules version.
3. Open the reconciliation trace to show the read-only evidence and guide tools, cited discrepancy, materiality flag, and recommendation, then show the same values written back to the record.
4. Open the console case detail, compare the application, prescription fixture, chronology, and guide passage; edit the risk class, enter a reason and rationale, and submit `ApproveEdited`. Show the record update and the Flow resuming at the wait node.
5. Return to Flow and show legacy policy-admin work, communication preview, and audit append running independently, merging, and reconciling receipts.
6. Open the case record to show proposed-versus-final values, reviewer change, source/rule references, versions, receipts, and final status.
7. Point out the disabled `External agent showcase` branch. Show its constant input, omitted `thread_id`, discarded response, and isolation from all underwriting variables.
8. Preview the expired-authorization fixture to show zero evidence calls, then show the adverse fixture's distinct second-review gate.

## Success measures

- **Business proof:** The demo exposes decision elapsed time, evidence waiting/re-requests, straight-through eligibility, referrals, overrides, dependency errors, and audit completeness as measurable pilot signals without promising a carrier result.
- **Flow proof:** A viewer can see IXP, authorization control, deterministic rules, two material agent roles, the non-material Azure AI Foundry showcase, API/RPA contrast, a record-created start and a record-change resumption, an Action Center task beside the console review, purposeful parallel work, merge, and safe recovery.
- **Demo proof:** In under ten minutes, a viewer can verify source-to-recommendation citations, a human correction consumed downstream, authorization-enforced tool isolation, receipt reconciliation, and a second-person adverse control.
- **Build proof:** Every project validates, the five-case evaluation set passes, bindings are recorded, `resources refresh` and dry-run pack succeed, and the immutable package can follow changed-solution CI into Playground after merge.

## Reference mapping

| Reference requirement | Domain-specific implementation | Evidence or gap |
| --- | --- | --- |
| 3-4 segment topology and canvas rules | Four blue sticky notes: Receive and validate application; Gather and reconcile evidence; Recommend and underwrite; Record decision and notify. Happy path is left-to-right, exceptions below, and completion branches are symmetric. | Fully specified; canvas implementation remains. |
| IXP/document intelligence | `LifeUnderwritingPacketExtraction` extracts application, authorization, exam/lab, and physician-statement fields with page/confidence evidence. | Contract and fixture fallback specified; project and model require provisioning. |
| API workflow and RPA on the intended path | `underwriting-evidence-gateway` normalizes, retrieves, requests, and audits; `legacy-underwriting-console` performs evidence/status reads and decision-work-item writes in a local UI fixture. | Contracts specified; projects, fixtures, runtime, and future API-gap validation remain. |
| Inline agent with a wired tool | `Underwriting Evidence Reconciler` uses scoped evidence and versioned guide-search tools and returns branch-driving completeness, inconsistency, route, and confidence. | Contract specified; agent, tools, model, and index remain unverified. |
| Coded agent with visible value-add | LangGraph `Medical Evidence Chronology` uses a least-privilege MCP evidence tool and creates a cited reviewer timeline. | Contract specified; coded agent and MCP server remain to build/register. |
| Shared external-agent showcase | Verified Azure AI Foundry node/connection on a disabled-by-default, constant-input, no-`thread_id`, discarded-output, fail-open branch. | Node and connection are tenant-ready and active; Flow binding and selected agent remain to validate. |
| Real business decision and safe exception | Consent, evidence, inconsistency, extraction, recommendation, and confidence expression controls the synthetic route; adverse, missing, timeout, and write exceptions remain safe. | Fully specified; demo thresholds require implementation and evaluation, carrier policy remains a human decision. |
| Human decision and returned outcome data | The console persists the named primary outcome, edited risk class, reason, rationale, reviewer, and time to the case record, and the Flow resumes from the record-updated event and routes from those fields. The adverse-control task returns its outcome through a completion handle. | Contract specified; authority matrix, service levels, entity, console, and action app remain open. |
| Purposeful parallelism and merge | Policy-admin/evidence-request work, communication preview, and audit append run independently and merge before receipt reconciliation. | Fully specified; route-required receipt matrix remains to encode. |
| Evaluation set and evaluator | Five fixtures plus route, authorization, grounding, MCP trajectory, adverse-control, isolation, and reconciliation checks. | Exact initial thresholds specified; fixtures/evaluators remain to build. |
| Process-app variant | Selected on August 20, 2026 in issue #56. Entity `LifeUnderwritingCase` is the canonical record, the record-created trigger starts the Flow, the `Underwriting Evidence Console` owns the primary review, and the record-updated event resumes the Flow. See [Data Fabric process-app variant](#data-fabric-process-app-variant). | Design fully specified; entity, choice sets, and console require provisioning. |
| Solution boundary and delivery contract | One `life-insurance-underwriting-evidence-exception` solution, one `.uipx`, nested Flow layout, entity and choice sets as solution resources, resource refresh, immutable version, changed-solution CI, and approved deployment parent `JD_Demos/demos`. The console and the action app deploy independently. | Fully specified; non-Azure resources require provisioning. |

## Open human decisions

These decisions refine or authorize implementation but do not block a synthetic, mock-backed build.

| Decision | Owner | Resolution path |
| --- | --- | --- |
| Confirm carrier, jurisdictions, product, face-amount band, and individual-versus-group scope. | Demo portfolio owner and carrier underwriting/legal owners | Approve the US individual term-life fixture framing or revise rules, sources, and cases before non-synthetic integration. |
| Approve straight-through, referral, materiality, confidence, evidence, and adverse-control policies. | Chief underwriter, compliance, and model-risk owners | Replace demo thresholds with versioned carrier rules and authority matrix; retain human adverse review. |
| Confirm authorization form, permitted sources/fields, retention, and revocation handling. | Privacy, legal, medical-information, and security owners | Review every input, tool, prompt, task, trace, and persistence field for the intended jurisdictions. |
| Choose application, evidence, audit, and policy-admin systems of record. | Enterprise architect and system owners | Map the canonical contracts, APIs/events, write authority, replay semantics, and record ownership. |
| Validate the legacy UI API gap and RPA responsibility. | Policy-admin/evidence-system owner and enterprise architect | Retain RPA only for a governed UI-only step; otherwise select a credible legacy gap or approve a reference deviation. |
| Confirm vendor/model sandboxes and source-specific timeout/retry rules. | Underwriting operations, vendor management, and model-risk owners | Verify permitted MVR, prescription, prior-application, EHR, and model resources; keep contract-shaped mocks until approved. |
| Set reviewer roles, separation of duty, outcomes, task timeout, and escalation roster. | Underwriting operations and compliance owners | Approve the console review and adverse-control task contracts and supply role groups and service levels. |
| Set pilot baselines, targets, and fairness-monitoring measures. | Underwriting operations, compliance, and model-risk owners | Supply observed timing, referral, override, outcome, error, and audit baselines before any benefit claim. |
| Approve the `LifeUnderwritingCase` schema, restricted-field list, retention, and record-level access control. | Privacy, records, and data-platform owners | Review every field for restricted content, set retention, and decide whether row-level access control is required before non-synthetic data. |
| Set console access roles and reviewer identity for separation of duty. | Underwriting operations and identity owners | Map underwriter and supervisor role groups to app permissions, and confirm that `primaryReviewerRef` and `secondReviewerRef` come from an identity the platform can prove. |
| Approve the acceptable resumption latency and the event delivery mode. | Demo portfolio owner and platform owner | Confirm the two-minute demo allowance, record the event mode resolved for the entity, and decide the rehearsal narration. |

## Implementation tasks

1. Scaffold `life-insurance-underwriting-evidence-exception` with exactly one `.uipx` and the nested Flow project.
2. Create choice sets `LiUnderwritingStatus` and `LiUnderwritingOutcome` and entity `LifeUnderwritingCase`, then add both kinds to the solution resource inventory.
3. Build the five-case fixture set, the record contract, the synthetic rule guide, the evidence services, and the local legacy UI. Seed fixtures with single-record inserts only.
4. Implement and validate the API workflow, IXP extraction contract, and RPA operations with typed status and receipts.
5. Build the inline reconciler, scoped evidence and guide tools, deterministic structured-output validator, and routing evaluation.
6. Build the LangGraph chronology agent, least-privilege MCP evidence tool, self-check, and trajectory/grounding evaluations.
7. Bind the record-created trigger and the record-updated wait node with `--detail.objectName LifeUnderwritingCase` and the `demos` Data Fabric connection. Set the record-ID and status `filterExpression` on the wait node, wire its `error` handle, and prove at run time that the event matcher applies the expression per instance. Record the resolved event mode.
8. Build and deploy the `Underwriting Evidence Console`, wire its decision write to the record, and prove the write fires the event the Flow waits on.
9. Build and deploy the `Underwriting Adverse Control Review` action app, then wire its completion handle, returned IDs, role separation, and outcomes into Flow.
10. Author the four-segment Flow, authorization stop, real business expressions, record write-back points, exception routes, non-material Azure showcase, parallel completion branches, merge, and reconciliation.
11. Validate the Azure binding and prove the showcase cannot change case data, routes, decisions, writes, previews, receipts, audit events, or final status.
12. Prove wait-node correlation isolation with two concurrent cases, and validate whether the record-updated node exposes a native timeout. If it does not, keep the parallel timer path.
13. Run project validation and the five-case evaluation set; resolve every warning and failed threshold without executing against production resources.
14. Refresh solution resources, restore, dry-run pack, and register immutable deployment configuration under approved parent `JD_Demos/demos` after human-owned policies are approved.

## Quality rubric

| Dimension | Score (0-3) | Evidence and remaining gap | Owner and resolution path |
| --- | ---: | --- | --- |
| Enterprise credibility | 2 | Consequential underwriting decision, roles, typed case/evidence contracts, controls, audit story, and measures are explicit; carrier policies, systems, authority, and baselines remain unverified. | Underwriting, legal, privacy, compliance, model-risk, and system owners validate during discovery. |
| Flow differentiation | 3 | Four segments visibly coordinate IXP, consent and deterministic logic, agents/tools, API, RPA, two-stage human authority, parallel completion, merge, and recovery. | Flow implementer preserves the topology and validates registry schemas, bindings, expressions, and receipts. |
| Demo clarity | 3 | The medication-discrepancy hero journey, authorization stop, adverse control, Azure isolation, and observable proof points form a timed script. | Demo owner builds representative fixtures and rehearses after deployment. |
| Build feasibility | 2 | Inputs, outputs, mocks, fallbacks, solution boundary, evaluation, current authenticated target, and the verified Azure and Data Fabric connections and node types are recorded; the entity, the two choice sets, the console, the action app, and the domain resources are not provisioned. | Tenant administrator and implementers provision resources, verify each binding, and close human-owned policies before live integration. |
| **Total** | **10/12** | **Ready for implementation planning as a synthetic demo; not ready for real applicant data, carrier decisioning, production integration, or communication.** | **Start with the solution and fixtures, then close owned human decisions before replacing mocks.** |
