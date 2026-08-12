# Life-insurance underwriting evidence-exception demo specification

## Use case and narrative

- **Domain and solution:** `life-insurance/underwriting-evidence-exception/underwriting-evidence-exception-solution/`, with globally unique solution and package name `life-insurance-underwriting-evidence-exception`.
- **Enterprise use case:** A senior life underwriter decides whether a synthetic individual term-life application can proceed to an offer, needs more evidence, should be postponed, should be declined, or needs escalation. Flow reconciles the application, authorization, extracted documents, synthetic external-evidence responses, and versioned underwriting guidance. The hero moment is the underwriter comparing discrepancies and cited evidence, editing the proposed risk class or evidence request, and returning a named decision that controls downstream work.
- **Why this use case:** It is the top-ranked candidate in the [life-insurance opportunity research](agentic-workflow-opportunities.md), scoring 4.75/5. It combines document intelligence, consent-aware evidence acquisition, deterministic rules, bounded agent reasoning, API and legacy-UI work, consequential human authority, and an auditable outcome. The death-claim candidate is also consequential, but its investigation and beneficiary-payment complexity makes a short, credible synthetic demo harder to bound.
- **Audience journey:** The demonstrator submits a synthetic application packet, follows four named canvas segments, inspects evidence and rule citations, completes an evidence-rich underwriting task, and shows the returned decision drive independent case-record, communication, and audit work before reconciliation. Flow is the right surface because the business value is in coordinating heterogeneous actors while keeping consent stops, review routes, timeouts, and recovery visible.
- **Success outcome:** The case ends with an offer work item, a tracked evidence request, or a human-authorized adverse outcome work item. Every route preserves evidence lineage, proposed-versus-final values, reviewer authority, downstream receipts, and a queryable audit record; no agent binds coverage or issues an adverse notice.
- **Measurable value:** Pilot measures are application-to-decision elapsed time, time waiting for evidence, evidence re-request rate, straight-through eligibility rate, review-referral rate, human override rate, dependency-error rate, and audit-record completeness. Carrier baselines and targets remain human decisions; the demo makes no ROI or improvement claim.
- **Out of scope:** Actuarial pricing or model development; sales advice; premium collection; policy delivery; production medical or consumer-report access; autonomous postpone or decline; adverse-notice legal sufficiency; real applicant communication; production policy administration; and selecting this domain for the later Data Fabric/process-app variant.

## Personas

| Persona | Role in the demo | Authority boundary |
| --- | --- | --- |
| New-business case manager | Monitors intake, authorization, evidence retrieval, and technical exceptions. | May correct intake metadata and request missing evidence; cannot choose a final risk class or adverse outcome. |
| Senior underwriter | Reviews reconciled evidence and makes the primary underwriting decision. | May approve or edit an offer, request evidence, postpone, decline, or escalate; postpone and decline require a separate adverse-outcome control before communication work. |
| Underwriting supervisor | Receives timed-out tasks and performs the second-person adverse-outcome control. | May approve, return, or escalate a proposed postpone/decline; cannot erase the first review or source evidence. |
| Compliance/model-risk reviewer | Inspects citations, versions, overrides, access, and evaluation evidence. | Defines governance requirements and investigates exceptions; does not decide an individual application in this demo. |
| Demonstrator | Submits fixtures and narrates the normal, review, consent-stop, and recovery paths. | Uses synthetic application, health, consumer-report, and rule data only. |

## Trigger and case contract

The initial build uses a manual trigger for repeatability. A new-business event may replace it after the carrier system and event ownership are chosen. `applicationId` is the correlation and idempotency key. An exact replay reads the existing queue item and receipts; a changed payload for the same ID creates a typed `INPUT_VERSION_CONFLICT` instead of overwriting evidence or decisions.

| Item | Specification |
| --- | --- |
| Trigger | Manual Flow trigger accepting `UnderwritingApplicationInput`. The `underwriting-evidence-gateway` API workflow validates and normalizes the payload before any evidence acquisition. |
| Canonical record | Orchestrator queue `LifeUnderwritingCases`, unique reference `applicationId`. Specific content holds the normalized input, authorization, evidence state, recommendation, review state, and audit events; output data holds the final result and receipts. |
| Required inputs | Application event and applicant facts, authorization, document references, carrier-rule version, and `showExternalAgentShowcase`. Only synthetic data is allowed. |
| Sensitive data | Applicant and medical facts are restricted synthetic data. Agents receive derived facts or allowlisted excerpts when raw identifiers are unnecessary. Tasks and traces mask applicant, producer, and source identifiers. |
| Outputs | Final route and risk class when applicable, required evidence, cited findings, reviewer result, communication preview, system receipts, typed exceptions, audit artifact, and final status. |

### Input schema

| Field | Type | Required | Contract |
| --- | --- | --- | --- |
| `applicationId` | string | yes | Stable unique correlation and queue reference, for example `LI-UW-00042`. |
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
| Receive and understand | **1. Receive and validate application** | Manual trigger invokes `underwriting-evidence-gateway`; IXP extracts the synthetic application and medical packet; the queue item is created or read. Output: normalized `UnderwritingCase` and extraction evidence. | Unsupported product/jurisdiction or invalid schema routes to intake exception. Invalid, expired, revoked, or out-of-scope authorization stops before external evidence calls. Any required-field confidence below `0.90` sets `requiresEvidenceReview = true`. |
| Assess and enrich | **2. Gather and reconcile evidence** | API workflow reads synthetic prescription, MVR, prior-application, and EHR contracts; RPA reads a local legacy evidence portal; inline `Underwriting Evidence Reconciler` retrieves approved guide passages; coded `Medical Evidence Chronology` creates a cited timeline. | `evidenceComplete`, `materialInconsistency`, `recommendation.confidence`, and `straightThroughEligible` drive routing. The external showcase branch rejoins with `UnderwritingCase` unchanged. |
| Decide and review | **3. Recommend and underwrite** | Deterministic rules send only clean, authorized, complete offer cases to the synthetic straight-through route. Other material cases open `Underwriting Evidence Review` for a senior underwriter. Postpone/decline opens the same app in `adverse_control` stage for a distinct supervisor. | `straightThroughEligible === true && evidenceComplete === true && materialInconsistency === false && extractionConfidence >= 0.90 && recommendation.route === "OFFER" && recommendation.confidence >= 0.90` permits the synthetic straight-through offer route. Human routes use `humanDecision.outcome`; `POSTPONE` and `DECLINE` require `secondReview.result === "approved"`. Thresholds are demo configuration, not carrier policy claims. |
| Act and communicate | **4. Record decision and notify** | In parallel, RPA writes an approved offer/adverse work item to a local legacy policy-admin UI, API workflow appends the authoritative audit event, and coded agent produces an approved-template status preview. The branches merge and receipts reconcile before completion. Evidence-request routes create a tracked request instead of a policy-admin decision write. | `requiredReceipts.every(receipt => receipt.status === "succeeded")` is required for `completed`. Missing or ambiguous writes set `technical_exception`; no route reports completion before merge and reconciliation. |

## Agentic reasoning and tool use

| Actor | Visible responsibility | Structured input and output | Tools, context, and guardrails | Readiness/fallback |
| --- | --- | --- | --- | --- |
| Inline low-code agent: `Underwriting Evidence Reconciler` | Compare declared facts with normalized evidence and approved guide passages, then recommend a bounded route. | Input: derived applicant facts, evidence statuses/findings, product/jurisdiction, and rules version. Output: proposed route/risk class, `evidenceComplete`, `materialInconsistency`, confidence, finding codes, `sourceRefs`, and `ruleRefs`. | Read-only `search_underwriting_guide(query, productCode, jurisdiction, rulesVersion)` and `get_normalized_evidence(applicationId, allowedSourceTypes)` tools. No raw document retrieval, coverage binding, final pricing, request sending, or adverse authority. Deterministic validation rejects unknown enums, uncited findings, and unauthorized source use. | Not provisioned. Versioned tool-response fixtures implement the same contract. Tool/grounding failure returns `insufficient_grounding` and forces human review. |
| Coded agent: `Medical Evidence Chronology` | Turn authorized source excerpts into a compact, reviewer-facing chronology without diagnosing or deciding risk. | Input: application ID, allowlisted source references, and date range. Output: dated events, contradiction flags, citations, omitted-source list, and self-check result. | LangGraph agent calls only `get_authorized_evidence_excerpt(sourceRef, applicationId)` through a least-privilege UiPath MCP server and self-checks every event against a returned source reference. It cannot query unlisted sources or emit a route/risk class. | Not provisioned. A deterministic chronology fixture is the fallback. Missing citations create `manual_chronology_required` but do not fabricate events. |
| External agent showcase: `Azure AI Foundry connectivity` | Display external-agent connectivity without performing underwriting work. | Connection-selected `agent_id` and constant message `UiPath Flow external-agent connectivity showcase for life insurance`; omit `thread_id`. Discard the response. | Node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread`, connection `0107247a-0197-42c9-b957-05d1b722b111`. Do not pass application, applicant, producer, authorization, document, evidence, rule, recommendation, reviewer, communication, receipt, or audit data. | Reverified August 12, 2026 in `uipathlabs / Playground`: the node is tenant-available; the connection is enabled, default, active, and in folder `demos`. The branch is disabled by default; a short timeout, error, or unexpected response records only transient showcase status and rejoins the unchanged core route. Validate the Flow-specific binding and selected agent during implementation. |

## Data and resources

| Resource or data dependency | Purpose and contract | Owner/folder/readiness | Security and failure handling |
| --- | --- | --- | --- |
| IXP project: `LifeUnderwritingPacketExtraction` | Classify and extract application facts, authorization dates/scope, exam/lab fields, attending-physician-statement fields, source page, and per-field confidence from synthetic PDFs. | New project/model beneath the selected `JD/demos` child folder; not provisioned. | Synthetic restricted fixtures only. A required-field confidence below `0.90`, malformed page, or cross-document conflict forces evidence review; versioned JSON preserves the output contract offline. |
| API workflow: `underwriting-evidence-gateway` | Operations `normalize_application`, `retrieve_evidence`, `create_evidence_request`, and `append_audit_event` return canonical schemas and deterministic mock receipts. | New sibling project; carrier/vendor APIs and connections are unverified. | Authorization is evaluated before each source request. Responses distinguish `notFound`, `notAuthorized`, `timeout`, and `providerError`; no error becomes a clean result. |
| RPA: `legacy-underwriting-console` | Operations `read_evidence_status` and `write_decision_work_item` use a local legacy UI fixture and return screenshot-linked receipts. | New sibling project; local mock application and unattended runtime are not provisioned. | UI automation exists only to demonstrate a verified API gap in the future carrier system. Ambiguous writes are not automatically retried or reported successful; recovery requires target-state reconciliation. |
| Queue: `LifeUnderwritingCases` | Canonical demo case, idempotency, state transitions, outputs, ordered audit events, and retry ownership. | New Orchestrator queue beneath the solution deployment folder; not provisioned. | Least-privilege robot access, masked identifiers, configured retention, immutable evidence/decision history, and version-conflict handling. |
| Synthetic evidence services | Contract-shaped prescription, MVR, prior-application, and EHR responses with source, timestamp, consent basis, status, and version. | Repository fixtures and local mock service to be built; no vendor marks or connectivity claims. | Per-source timeouts/retries, provenance, and no silent substitution. Product-rule fixture decides whether a partial case may continue or must request evidence. |
| Context index: `LifeUnderwritingGuide` | Versioned synthetic product eligibility, evidence requirements, referral thresholds, and guide excerpts for cited retrieval. | New Context Grounding index; model and folder not provisioned. | Read-only approved content. Missing or mismatched `ruleRef` forces review; rule version is persisted on recommendations and decisions. |
| MCP evidence server | Exposes only `get_authorized_evidence_excerpt(sourceRef, applicationId)` for the chronology agent. | New demo MCP server or deterministic mock; not registered. | Enforces application/source allowlists, returns synthetic excerpts plus hashes, logs exact calls, and has no search-all or write tools. |
| Azure AI Foundry connection | Supplies the non-material showcase node with a selected agent and constant message; output is discarded. | Shared connection `0107247a-0197-42c9-b957-05d1b722b111`; active, enabled, and default in Playground folder `demos` on August 12, 2026. | No case or sensitive data, no business-variable mapping, fail-open continuation to the same route, and Flow-specific binding validation during implementation. |
| Coded action app: `Underwriting Evidence Review` | Evidence-rich `primary` and `adverse_control` task stages with source excerpts, citations, proposed/final differences, and named outcomes. | Deployed independently and referenced by Flow; not provisioned. | Role-restricted, masked identifiers, required reason/rationale validation, distinct reviewer enforcement for adverse control, and no system credentials. |

Read-only inspection on August 12, 2026 used `/opt/homebrew/bin/uip` version
1.199.0, authenticated to `uipathlabs / Playground`. Tenant registry search
confirmed the Azure AI Foundry node is available, connection discovery found the
specified default connection in folder `demos`, and connection ping reported it
active. No other Flow, IXP, queue, index, agent, app, MCP, API, RPA, or runtime
resource was verified; those remain mocks or owned provisioning gaps.

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
└── underwriting-evidence-review/
```

The solution has exactly one `.uipx` manifest and is independently deployable. The coded action app remains independently deployed if required by the platform, with its versioned contract pinned by solution configuration. Before packaging, run `uip solution resources refresh`, restore dependencies, and dry-run pack. Pull requests validate only the changed solution; publish and deploy occur only after merge to `main` through `playground-deploy`, using an immutable package version and a child folder beneath `JD/demos`.

## Human decisions

- **Primary review:** The senior underwriter sees normalized application facts, authorization status, extracted fields and source pages, evidence provenance/status, chronology, discrepancies, missing evidence, guide citations/version, recommendation/confidence, and prior audit events.
- **Editable values:** Proposed risk class, required-evidence items, reason code, and rationale. Application/source facts, authorization, citations, confidence, and prior events remain read-only.
- **Primary outcomes:** `Approve`, `ApproveEdited`, `RequestEvidence`, `Postpone`, `Decline`, and `Escalate`. Every outcome requires a reason code; edited, postpone, decline, and escalate outcomes require free-text rationale.
- **Adverse control:** `Postpone` and `Decline` create a second task in `adverse_control` stage for a distinct underwriting supervisor. Outcomes are `ApproveAdverse`, `ReturnForReview`, and `EscalateAdverse`; no adverse communication work begins without `ApproveAdverse`.
- **Timeout and resumption:** Timeout values remain carrier-owned configuration. Expiry assigns the task to an underwriting supervisor queue without changing coverage status. Flow resumes from the completion handle, validates returned field IDs, role separation, enums, and required rationale, persists proposed-versus-final differences, and routes only from the returned outcome.
- **Downstream authority:** Agents recommend or draft only. Only validated deterministic rules or returned human outcomes enable evidence requests, policy-admin work items, or communication previews. No component sends an applicant message in the demo.

## Controls and safety

| Control | Design decision | Evidence in the Flow or demo |
| --- | --- | --- |
| Authorization before acquisition | Validate signed state, source scope, expiry, and revocation before each external-evidence call; invalid authorization stops acquisition. | Authorization decision before API/RPA evidence nodes and an evaluator asserting zero calls for the expired fixture. |
| Adverse-outcome safety | No autonomous postpone/decline; named primary outcome plus distinct adverse control is required. | Completion handles, reviewer-role check, returned outcomes, and second-review audit event. |
| Grounded recommendation | Findings require resolvable `sourceRef` and `ruleRef`; unknown enums, missing citations, or insufficient evidence force review. | Structured-output validator, Context Grounding tool, and citation evaluator. |
| Access and data | Synthetic restricted data, least-privilege identities, scoped tools, managed connections, masked logs/tasks, and configured retention. | Fixture inventory, connection bindings, tool allowlists, task roles, and redaction assertions. |
| Agent boundaries | Reconciler recommends only; chronology agent neither diagnoses nor routes; communication output is preview only; no agent writes policy state. | Prompt/tool schemas, absent write/send tools, deterministic route validation, and human completion edges. |
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
| Human task timeout or invalid return | Keep case pending with no coverage-state change and assign the supervisor queue. | Supervisor reassigns or completes a valid task; invalid payload is rejected and audited. |
| Ambiguous or failed RPA/API write | Do not retry an unknown write automatically or create a success receipt; set `technical_exception`. | Operations reconciles target state, records the result, and explicitly resumes the failed branch. |
| Communication chronology/template failure | Create a manual-preview work item; preserve decision and system receipts without claiming communication completion. | New-business communications owner supplies approved copy. |
| Audit append failure | Block final completion and create an operations incident. | Platform owner restores persistence and replays the idempotent audit append before merge reconciliation completes. |
| Azure AI Foundry timeout, error, or unexpected response | Record only transient `externalAgentShowcaseStatus`, discard the response, and rejoin the exact same core route. | Demo owner may disable the branch; business users take no recovery action because business state is unchanged. |

## Observability and evaluation

| Signal or test | What it proves | Expected result or threshold |
| --- | --- | --- |
| Correlation and audit | Application, authorization, evidence, tools, recommendation, human changes, writes, and recovery can be reconstructed. | Every trace, task, tool call, receipt, and audit event contains `applicationId`; rules/model/prompt/tool versions and input/output hashes are present; no unmasked applicant identifier appears outside approved storage. |
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
| Clean, authorized, complete application | Synthetic straight-through `OFFER`. | Cited recommendation, final fixture risk class, policy-admin offer work item, status preview, audit receipt, and `completed`. |
| Medication declaration discrepancy | Primary human review with `ApproveEdited`. | Prescription fixture and guide rule are cited; proposed and final risk classes differ and the override rationale is persisted. |
| Expired authorization | `REQUEST_AUTHORIZATION` before evidence acquisition. | Zero external-evidence/MCP calls; tracked authorization request and `evidence_requested`. |
| Prescription endpoint timeout | Product-rule fixture routes to `RequestEvidence`. | Status remains `timeout`, never `notFound`; request receipt, retry count, and owner are audited. |
| Material adverse evidence | Primary `Decline`, then distinct `ApproveAdverse`. | Two reviewer records, reason code/rationales, adverse work item and preview, no autonomous or sent notice, and reconciled audit evidence. |

## Demo script

1. Show the four-segment canvas, consent stop, review route, completion branches, and exceptions below the happy path.
2. Submit the medication-discrepancy fixture and point out `applicationId`, authorization scope, packet confidence, and pinned rules version.
3. Open the reconciliation trace to show the read-only evidence and guide tools, cited discrepancy, materiality flag, and recommendation.
4. Open `Underwriting Evidence Review`, compare the application, prescription fixture, chronology, and guide passage; edit the risk class, enter a reason and rationale, and select `ApproveEdited`.
5. Return to Flow and show legacy policy-admin work, communication preview, and audit append running independently, merging, and reconciling receipts.
6. Open the queue record to show proposed-versus-final values, reviewer change, source/rule references, versions, receipts, and final status.
7. Point out the disabled `External agent showcase` branch. Show its constant input, omitted `thread_id`, discarded response, and isolation from all underwriting variables.
8. Preview the expired-authorization fixture to show zero evidence calls, then show the adverse fixture's distinct second-review gate.

## Success measures

- **Business proof:** The demo exposes decision elapsed time, evidence waiting/re-requests, straight-through eligibility, referrals, overrides, dependency errors, and audit completeness as measurable pilot signals without promising a carrier result.
- **Flow proof:** A viewer can see IXP, authorization control, deterministic rules, two material agent roles, the non-material Azure AI Foundry showcase, API/RPA contrast, consequential human authority, purposeful parallel work, merge, and safe recovery.
- **Demo proof:** In under ten minutes, a viewer can verify source-to-recommendation citations, a human correction consumed downstream, authorization-enforced tool isolation, receipt reconciliation, and a second-person adverse control.
- **Build proof:** Every project validates, the five-case evaluation set passes, bindings are recorded, `resources refresh` and dry-run pack succeed, and the immutable package can follow changed-solution CI into Playground after merge.

## Reference mapping

| Reference requirement | Domain-specific implementation | Evidence or gap |
| --- | --- | --- |
| 3-4 segment topology and canvas rules | Four blue sticky notes: Receive and validate application; Gather and reconcile evidence; Recommend and underwrite; Record decision and notify. Happy path is left-to-right, exceptions below, and completion branches are symmetric. | Fully specified; canvas implementation remains. |
| IXP/document intelligence | `LifeUnderwritingPacketExtraction` extracts application, authorization, exam/lab, and physician-statement fields with page/confidence evidence. | Contract and fixture fallback specified; project/model/folder remain to provision. |
| API workflow and RPA on the intended path | `underwriting-evidence-gateway` normalizes, retrieves, requests, and audits; `legacy-underwriting-console` performs evidence/status reads and decision-work-item writes in a local UI fixture. | Contracts specified; projects, fixtures, runtime, and future API-gap validation remain. |
| Inline agent with a wired tool | `Underwriting Evidence Reconciler` uses scoped evidence and versioned guide-search tools and returns branch-driving completeness, inconsistency, route, and confidence. | Contract specified; agent, tools, model, and index remain unverified. |
| Coded agent with visible value-add | LangGraph `Medical Evidence Chronology` uses a least-privilege MCP evidence tool and creates a cited reviewer timeline. | Contract specified; coded agent and MCP server remain to build/register. |
| Shared external-agent showcase | Verified Azure AI Foundry node/connection on a disabled-by-default, constant-input, no-`thread_id`, discarded-output, fail-open branch. | Node and connection are tenant-ready and active; Flow binding and selected agent remain to validate. |
| Real business decision and safe exception | Consent, evidence, inconsistency, extraction, recommendation, and confidence expression controls the synthetic route; adverse, missing, timeout, and write exceptions remain safe. | Fully specified; demo thresholds require implementation and evaluation, carrier policy remains a human decision. |
| Human decision and returned outcome data | Coded action app returns named primary/second outcomes, edited risk/evidence values, reason, rationale, reviewer, and time; Flow validates and consumes them. | Contract specified; authority matrix, SLA, and deployed app remain open. |
| Purposeful parallelism and merge | Policy-admin/evidence-request work, communication preview, and audit append run independently and merge before receipt reconciliation. | Fully specified; route-required receipt matrix remains to encode. |
| Evaluation set and evaluator | Five fixtures plus route, authorization, grounding, MCP trajectory, adverse-control, isolation, and reconciliation checks. | Exact initial thresholds specified; fixtures/evaluators remain to build. |
| Process-app variant | Not selected. | Later cross-domain selection may replace the queue with a Data Fabric canonical record and process app. |
| Solution boundary and delivery contract | One `life-insurance-underwriting-evidence-exception` solution, one `.uipx`, nested Flow layout, resource refresh, immutable version, and changed-solution CI. | Fully specified; deployment child folder and non-Azure resources require provisioning. |

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
| Set reviewer roles, separation of duty, outcomes, task timeout, and escalation roster. | Underwriting operations and compliance owners | Approve the primary/adverse-control app contract and supply role groups and service levels. |
| Select the exact `JD/demos` child folder and provision non-Azure resources. | UiPath tenant administrator | Reuse the verified Azure connection; provision queue, IXP, index, agents, app, runtime, MCP registration, and record every binding. |
| Set pilot baselines, targets, and fairness-monitoring measures. | Underwriting operations, compliance, and model-risk owners | Supply observed timing, referral, override, outcome, error, and audit baselines before any benefit claim. |
| Decide whether life insurance is a Data Fabric/process-app variant. | Demo portfolio owner | Resolve in the later cross-domain selection issue; this spec currently marks it not selected. |

## Implementation tasks

1. Scaffold `life-insurance-underwriting-evidence-exception` with exactly one `.uipx` and the nested Flow project.
2. Build the five-case fixture set, queue contract, synthetic rule guide, evidence services, and local legacy UI.
3. Implement and validate the API workflow, IXP extraction contract, and RPA operations with typed status and receipts.
4. Build the inline reconciler, scoped evidence and guide tools, deterministic structured-output validator, and routing evaluation.
5. Build the LangGraph chronology agent, least-privilege MCP evidence tool, self-check, and trajectory/grounding evaluations.
6. Build and deploy the coded action app, then wire primary and adverse-control completion handles, returned IDs, edits, role separation, and outcomes into Flow.
7. Author the four-segment Flow, authorization stop, real business expressions, exception routes, non-material Azure showcase, parallel completion branches, merge, and reconciliation.
8. Validate the Azure binding and prove the showcase cannot change case data, routes, decisions, writes, previews, receipts, audit events, or final status.
9. Run project validation and the five-case evaluation set; resolve every warning and failed threshold without executing against production resources.
10. Refresh solution resources, restore, dry-run pack, and register immutable deployment configuration after human-owned policies and target folder are approved.

## Quality rubric

| Dimension | Score (0-3) | Evidence and remaining gap | Owner and resolution path |
| --- | ---: | --- | --- |
| Enterprise credibility | 2 | Consequential underwriting decision, roles, typed case/evidence contracts, controls, audit story, and measures are explicit; carrier policies, systems, authority, and baselines remain unverified. | Underwriting, legal, privacy, compliance, model-risk, and system owners validate during discovery. |
| Flow differentiation | 3 | Four segments visibly coordinate IXP, consent and deterministic logic, agents/tools, API, RPA, two-stage human authority, parallel completion, merge, and recovery. | Flow implementer preserves the topology and validates registry schemas, bindings, expressions, and receipts. |
| Demo clarity | 3 | The medication-discrepancy hero journey, authorization stop, adverse control, Azure isolation, and observable proof points form a timed script. | Demo owner builds representative fixtures and rehearses after deployment. |
| Build feasibility | 2 | Inputs, outputs, mocks, fallbacks, solution boundary, evaluation, current authenticated target, and the verified Azure node/connection are recorded; remaining resources are not provisioned. | Tenant administrator and implementers provision resources, verify each binding, and close human-owned policies before live integration. |
| **Total** | **10/12** | **Ready for implementation planning as a synthetic demo; not ready for real applicant data, carrier decisioning, production integration, or communication.** | **Start with the solution and fixtures, then close owned human decisions before replacing mocks.** |
