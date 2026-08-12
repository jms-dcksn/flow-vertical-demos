# Healthcare provider abnormal diagnostic result follow-up demo specification

## Use case and narrative

- **Domain and solution:** `healthcare-provider/abnormal-result-follow-up/abnormal-result-follow-up-solution/`, with globally unique solution and package name `healthcare-provider-abnormal-result-follow-up`.
- **Enterprise use case:** An ambulatory clinician closes the loop on a synthetic abnormal radiology result. Flow correlates a versioned `DiagnosticReport`, source-designated safety flags, responsible-clinician coverage, limited patient context, and a synthetic approved follow-up policy. The consequential decision is whether to accept or edit the proposed follow-up, urgently escalate, record no action with rationale, or request missing context. The hero moment is the clinician correcting the evidence-linked plan in a coded action app and watching those returned fields drive mock EHR documentation, scheduling, and patient-communication work.
- **Why this use case:** It is the top-ranked candidate in the [healthcare-provider opportunity research](agentic-workflow-opportunities.md), scoring 15/15. Public evidence establishes result routing, acknowledgment, ownership, and follow-up as patient-safety concerns, while the workflow remains compact enough to demonstrate deterministic safeguards, document intelligence, agents, API and UI-system work, human authority, parallel completion, and audit lineage with synthetic data. Prior authorization ranked second but introduces payer-specific criteria and portal variability; discharge planning has more live dependencies; referral closure is less visually distinctive for a first demo.
- **Audience journey:** The demonstrator submits a synthetic ambulatory radiology result, follows four named canvas segments, shows deterministic critical/ownership routing and cited policy evidence, opens the clinician review task, edits and approves a follow-up plan, and then shows independent mock documentation, scheduling, and communication-preview branches merge into an auditable status. Flow is the right surface because it makes clinical ownership, AI boundaries, actor contrast, waiting, safe exceptions, and completion evidence visible in one orchestration.
- **Success outcome:** Each unique result version has one correlated case; a source-critical or unresolved-owner case escalates without relying on agent confidence; every proposed plan remains advisory until a clinician acts; and closure occurs only when required mock receipts reconcile. No model diagnoses, sets urgency, chooses treatment, acknowledges for a clinician, sends clinical advice, or writes to a live clinical system.
- **Measurable value:** Pilot measures are time to clinician acknowledgment, results overdue for review, time to patient notification, percentage with a documented plan, follow-up-task completion, exception aging, and amendment re-review. These are measurement candidates from the research evidence, not promised improvements, adopted service levels, or ROI.
- **Out of scope:** Live EHR, LIS, RIS, PACS, portal, identity, on-call, or scheduling connectivity; real protected health information; autonomous diagnosis, urgency classification, treatment selection, patient messaging, or clinical-system writes; production clinical-policy interpretation; measuring patient comprehension; and Data Fabric/process-app scope.

## Personas

| Persona | Role in the demo | Authority boundary |
| --- | --- | --- |
| Ordering or covering clinician | Reviews the result, evidence, policy citation, and proposed plan, then records the clinical disposition. | Owns acknowledgment and follow-up until an explicit accepted transfer; only this role can accept/edit the plan or record no action with rationale. |
| Diagnostic follow-up coordinator | Reconciles low-confidence documents, missing ownership, scheduling exceptions, and overdue work. | May correct administrative context and coordinate recovery; cannot diagnose or authorize a clinical plan. |
| Patient-safety or ambulatory-operations lead | Monitors acknowledgment, aging, handoffs, exceptions, and closure evidence. | May reassign operational recovery and review metrics; cannot override the clinician decision. |
| Demonstrator | Submits fixtures and narrates evidence, review, parallel work, and recovery. | Uses synthetic, de-identified inputs and mock write targets only. |

## Trigger and case contract

The initial build uses a manual trigger for repeatability. A FHIR subscription or diagnostic-source event may replace it after a non-production integration is approved. `<organizationKey>:<resultId>:<resultVersion>` is the idempotency key. An exact replay reads the existing queue item and appends a replay event; an amendment creates the new versioned reference, links `amendsResultId`, marks the prior recommendation stale, and requires a new review.

| Item | Specification |
| --- | --- |
| Trigger | Manual Flow trigger accepting `DiagnosticResultInput`. `diagnostic-result-gateway` validates the synthetic FHIR-like event or PDF metadata and emits the canonical case. |
| Canonical record | Orchestrator queue `HealthcareProviderAbnormalResultCases`, unique reference `<organizationKey>:<resultId>:<resultVersion>`. Specific content stores minimized case state and versioned evidence; output data stores the decision, receipts, exceptions, and final status. |
| Required inputs | Result/ownership fields below plus `reportInputMode`, `policyVersion`, and optional `reportFileName`. All identifiers and clinical content are synthetic; no direct patient identifier is needed on the Flow canvas. |
| Sensitive data | Synthetic, de-identified clinical fixtures only. Patient key, report text, and contact restrictions are treated as sensitive in task access, traces, prompts, and retention even though the demo data is fictitious. |
| Outputs | Advisory assessment, clinician decision, mock `Task`/`ServiceRequest` references, scheduling receipt, patient-message preview, acknowledgment/closure timestamps, typed exceptions, and ordered audit events. |

### Input schema

| Field | Type | Required | Contract |
| --- | --- | --- | --- |
| `organizationKey` | string | yes | Synthetic organization namespace used in correlation. |
| `patientKey` | string | yes | Synthetic opaque patient reference; no direct identifier. |
| `encounterId` | string | no | Synthetic encounter reference when applicable. |
| `orderId` | string | yes | Correlates the diagnostic order and follow-up write-back. |
| `resultId` | string | yes | Stable diagnostic-result identifier. |
| `resultVersion` | string | yes | Monotonic source version used for amendment handling. |
| `orderingClinicianId` | string | yes | Synthetic responsible practitioner reference. |
| `coveringClinicianId` | string | no | Accepted covering practitioner when known. |
| `careTeamIds` | string array | yes | Synthetic fallback team references; may be empty. |
| `resultType` | string | yes | Demo scope is ambulatory radiology. |
| `sourceSystem` | string | yes | Synthetic FHIR or legacy-report source. |
| `issuedAt` | ISO 8601 string | yes | Source issue timestamp used for aging. |
| `status` | enum | yes | `final`, `amended`, `corrected`, or `cancelled`. |
| `sourceAbnormalFlag` | boolean | yes | Deterministic flag supplied by the diagnostic source. |
| `sourceCriticalFlag` | boolean | yes | Deterministic critical-result flag; agents cannot change it. |
| `conclusion` | string | yes | Synthetic source conclusion, never generated by an agent. |
| `observationRefs` | string array | yes | Synthetic references used for evidence lineage. |
| `reportInputMode` | enum | yes | `fhir` or `pdf`. |
| `reportFileName` | string | conditional | Required for `pdf`; repository fixture name only. |
| `amendsResultId` | string | no | Prior result identifier when this event amends or corrects it. |
| `policyVersion` | string | yes | Pins the synthetic approved follow-up policy used for retrieval. |
| `contactPreferences` | object | yes | Synthetic channel, language, and accessibility preferences. |
| `communicationRestrictions` | string array | yes | Synthetic hold/suppression rules checked before any preview. |
| `showExternalAgentShowcase` | boolean | yes | Defaults to `false`; controls only the non-material showcase branch. |

### Output schema

| Field | Type | Contract |
| --- | --- | --- |
| `caseStatus` | enum | `review_required`, `awaiting_context`, `urgent_escalation`, `follow_up_in_progress`, `closed`, or `technical_exception`. |
| `assessment` | object | `summary`, `evidenceRefs`, `policyRefs`, `missingContext`, `proposedDisposition`, `proposedFollowUp`, `confidence`, and `requiresClinicianReview=true`. |
| `ownershipStatus` | enum | `ordering_clinician`, `accepted_covering_clinician`, or `unresolved`. |
| `clinicianDecision` | object | Outcome, approved plan, rationale, responsible clinician, due date, reviewer/version references, and timestamp. |
| `acknowledgedAt` | ISO 8601 string | Set only from the clinician task completion event. |
| `followUpTaskIds` | string array | Mock administrative/clinical task references returned by the API workflow. |
| `serviceRequestIds` | string array | Mock ordered-service references returned only for an approved plan. |
| `schedulingReceipt` | object | Mock appointment request/status or typed scheduling exception. |
| `patientCommunication` | object | Preview, approved template/version, restrictions check, and delivery status `not_sent`. |
| `writeBackReceipts` | object array | System, operation, correlation ID, status, and timestamp. |
| `exceptions` | object array | Typed code, owner, safe state, recovery condition, and timestamp. |
| `auditEvents` | object array | Actor, action, source/model/prompt/policy/tool/reviewer versions, evidence references, hashes, and timestamp. |

## Flow topology

Use four blue sticky notes with the titles below. The happy path runs left to right; invalid input, low-confidence extraction, ownership, timeout, and dependency exceptions sit below their originating segment. Independent post-decision work is visually symmetric and merges before reconciliation. A separately labelled `External agent showcase` branch sits below segment 2 and never receives case or clinical data.

| Reference segment | Domain-specific canvas title | Actors and business output | Branch or merge evidence |
| --- | --- | --- | --- |
| Receive and understand | **1. Receive and normalize the result** | Manual trigger invokes `diagnostic-result-gateway`; structured FHIR-like data is normalized directly, while `HealthcareProviderDiagnosticReportExtraction` processes the legacy-PDF fixture. The queue item is created or read. Output: versioned `AbnormalResultCase`. | Invalid schema or cancelled status stops safely. `reportInputMode === "pdf" && extractionConfidence < 0.90` routes to manual reconciliation. Duplicate version reads the existing case; amendment marks prior assessment stale. |
| Assess and enrich | **2. Assemble evidence and ownership** | API workflow returns minimal synthetic patient/ownership context. Inline `Follow-Up Evidence Synthesizer` uses read-only context and pinned-policy tools to produce a cited advisory assessment. The Azure AI Foundry showcase runs on its isolated flag branch. | `sourceCriticalFlag === true \|\| ownershipStatus === "unresolved"` enters urgent escalation before any routine plan. Otherwise `assessment.confidence >= 0.85 && assessment.missingContext.length === 0 && policyStatus === "current"` presents the advisory plan; the safe alternate presents facts without a recommendation. Both require clinician review. Thresholds are demo configuration, not clinical policy. |
| Decide and review | **3. Clinician owns the follow-up** | `Diagnostic Result Follow-Up Review`, a coded action app, shows source evidence, policy passages, ownership, agent rationale, confidence, and editable follow-up fields. Output: named clinician outcome and returned plan data. | `clinicianDecision.outcome` is `AcceptPlan`, `EditPlan`, `UrgentEscalation`, `NoActionWithRationale`, or `RequestContext`. A 15-minute demo-only timeout routes to the covering pool without acknowledging the result. `RequestContext` opens a coordinator task and resumes at a new clinician review. |
| Act and communicate | **4. Complete and prove the loop** | For an approved plan, parallel branches invoke the API workflow for mock EHR task/order receipts, `legacy-follow-up-scheduler` RPA for a UI-only mock appointment, and coded `Patient Follow-Up Writer` for a non-sent grounded preview. The branches merge before receipt reconciliation and queue completion. | `requiredReceipts.every(receipt => receipt.status === "succeeded")` permits `closed`. Missing write/schedule receipts set `technical_exception`; restrictions or write-back failure hold the preview. `NoActionWithRationale` closes only with the signed rationale and no order/scheduling branch. |

## Agentic reasoning and tool use

| Actor | Visible responsibility | Structured input and output | Tools, context, and guardrails | Readiness/fallback |
| --- | --- | --- | --- | --- |
| Inline low-code agent: `Follow-Up Evidence Synthesizer` | Summarize source evidence and prepare a bounded, policy-cited advisory plan for clinician review. | Input: canonical case, deterministic flags, minimal context, `policyVersion`. Output: the `assessment` object; `confidence`, `missingContext`, and citations drive the review presentation but never bypass it. | Read-only `get_synthetic_patient_context(patientKey, resultId)` and `search_synthetic_followup_policy(policyVersion, resultType, query)` tools. Every material statement needs result/policy evidence. No web search, diagnosis, urgency override, write, message, acknowledgment, or treatment authority. | Not provisioned. Build with versioned local fixtures and a repository policy corpus. Missing/expired policy or unavailable tools returns facts plus `insufficient_evidence` and suppresses the proposed plan while preserving clinician review. |
| Coded agent: `Patient Follow-Up Writer` | Draft a plain-language communication preview from the clinician-approved plan. | Input: case ID, approved plan, allowed facts, language, channel, restrictions result, and template ID. Output: subject/body, `templateId`, `templateVersion`, citations, readability/safety findings, and `deliveryStatus = "not_sent"`. | LangGraph agent calls `get_approved_followup_message_template(templateId, language, channel)` through a least-privilege UiPath MCP server, then self-checks for unsupported clinical claims, missing plan details, prohibited data, and language mismatch. It has no send tool. | Not provisioned. Use approved static synthetic templates and a mock MCP tool during build. Tool or safety-check failure creates a manual-draft task; no message is sent. |
| External agent showcase: `Azure AI Foundry connectivity` | Display Azure AI Foundry connectivity without performing clinical or case work. | Connection-selected `agent_id` and constant message `UiPath Flow external-agent connectivity showcase for healthcare provider`; omit `thread_id`. The response is discarded. | Node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` with connection `0107247a-0197-42c9-b957-05d1b722b111`. Do not pass patient, result, policy, ownership, reviewer, plan, communication, receipt, case, or sensitive data. | The shared connection was verified enabled in Playground folder `JD_Demos/demos` on August 12, 2026, and CLI 1.199.0 currently reports the node tenant-available. The branch is disabled by default; timeout/error records only transient showcase status and rejoins the unchanged core route. Flow-specific binding and the selected `agent_id` must still be validated during implementation. |

### Actor inventory

| Actor | Implementation contract and readiness |
| --- | --- |
| Trigger | Manual trigger in the Flow deployed to the approved Playground parent `JD_Demos/demos`; consumes `DiagnosticResultInput`; correlation is `<organizationKey>:<resultId>:<resultVersion>`. Event ownership remains unselected. |
| IXP | `HealthcareProviderDiagnosticReportExtraction` in the solution folder extracts `resultId`, `resultVersion`, `issuedAt`, `resultType`, abnormal/critical flags, conclusion, observation references, and ordering-clinician reference from the synthetic PDF. Confidence below `0.90` or a missing required field routes to manual reconciliation. The model is not provisioned. |
| API workflow | `diagnostic-result-gateway` operations `normalize_result`, `read_minimal_context`, and `record_follow_up`; consumes/returns the schemas in this spec and is invoked in segments 1, 2, and 4. It targets a local mock endpoint until a governed EHR sandbox exists. |
| RPA | `legacy-follow-up-scheduler` operation `request_appointment`; consumes the clinician-approved service, due window, and synthetic patient key; returns a typed receipt. It targets a UI-only local scheduler fixture; production API availability is unverified. |
| Inline agent | `Follow-Up Evidence Synthesizer`, with the structured contract, two read-only tools, policy pinning, and prohibited actions above. Model and tenant resource are not provisioned. |
| Coded agent | LangGraph `Patient Follow-Up Writer`, with one communication-preview responsibility, one template MCP tool, structured output, self-review, and output/trajectory evaluators. Project and server are not provisioned. |
| External agent | Required Azure AI Foundry node/connection, constant message, no `thread_id`, discarded response, disabled-by-default flag, short timeout/error continuation, and invariant business state as specified above. |
| Human task | Independently deployed coded action app `Diagnostic Result Follow-Up Review`; displays source/policy evidence and returns exact named outcome and field IDs. App and role assignment are not provisioned. |
| MCP server/tool | Demo-owned, least-privilege server exposing only `get_approved_followup_message_template`; expected once per applicable draft and forbidden from sending or retrieving patient data. Server is not provisioned. |

## Data and resources

| Resource or data dependency | Purpose and contract | Owner/folder/readiness | Security and failure handling |
| --- | --- | --- | --- |
| IXP project: `HealthcareProviderDiagnosticReportExtraction` | Extract the required source fields from synthetic abnormal, amended, and low-confidence radiology PDFs. | New project/model in the approved Playground parent `JD_Demos/demos`; not provisioned. | Synthetic files only. Confidence below `0.90`, missing required fields, or conflict with structured input forces manual reconciliation; versioned JSON fixtures preserve the contract offline. |
| API workflow: `diagnostic-result-gateway` | Validate/normalize intake, read minimum synthetic context/coverage, and return idempotent mock `Task`/`ServiceRequest` receipts. | New sibling project; local mock FHIR endpoint initially. Vendor, FHIR release/profile, scopes, and writable resources are unverified. | Minimum necessary fields, two bounded retries for transient reads, typed errors, idempotency key on writes, and no fabricated receipt after failure or ambiguity. |
| RPA: `legacy-follow-up-scheduler` | Request the clinician-approved follow-up in a local UI fixture and return appointment request/status. | New sibling project; unattended runtime and production scheduling API gap are unverified. | Synthetic identifiers only. Selector or business errors return screenshot/error references; ambiguous writes are never retried automatically. |
| Queue: `HealthcareProviderAbnormalResultCases` | Canonical demo case, version-aware idempotency, current state, final outputs, exceptions, and ordered audit events. | New Orchestrator queue in the approved Playground parent `JD_Demos/demos`; not provisioned. | Least-privilege access, minimized specific content, masked traces, synthetic-only retention chosen during provisioning, and append-only amendment history. |
| Context index: `HealthcareProviderFollowUpPolicy` | Versioned synthetic local policy used by the inline agent. | New Context Grounding index; clinical-governance owner must approve any non-synthetic corpus; not provisioned. | Retrieval is pinned to `policyVersion`. Missing, expired, or uncited content suppresses the recommendation and forces facts-only review. |
| Synthetic context and coverage tools | Return limited recent result, active problem, existing order, contact restriction, ordering clinician, accepted covering clinician, and escalation-pool facts. | Local fixture-backed tools initially; EHR and identity/coverage owners are unverified. | Read-only and minimum necessary. Ownership ambiguity forces urgent manual escalation; agent output cannot resolve it. |
| Azure AI Foundry external-agent connection | Supplies the non-material showcase node with a constant message and discarded response. | Shared connection `0107247a-0197-42c9-b957-05d1b722b111`; verified enabled in Playground folder `JD_Demos/demos` on August 12, 2026. | No case or sensitive data, no business-variable mapping, fail-open continuation, and Flow-specific binding validation during implementation. |
| MCP template server | Exposes only `get_approved_followup_message_template(templateId, language, channel)`. | New demo server or local mock; communications/clinical owner approves templates; not provisioned. | No send or patient lookup capability. Failure creates a manual-draft task and leaves `deliveryStatus = "not_sent"`. |
| Coded action app: `Diagnostic Result Follow-Up Review` | Evidence-rich review UI with source/version, report spans, policy passages, ownership, recommendation, editable plan/due date, rationale, and named outcomes. | Deployed independently and referenced by Flow; not provisioned. | Role-restricted access, required rationale for edits/no action/escalation, immutable source facts, and no direct clinical-system credentials. |

### Solution boundary and layout

```text
healthcare-provider/abnormal-result-follow-up/
├── abnormal-result-follow-up-solution/
│   ├── healthcare-provider-abnormal-result-follow-up.uipx
│   ├── abnormal-result-follow-up-flow/
│   │   └── abnormal-result-follow-up-flow.flow
│   ├── diagnostic-result-gateway/
│   ├── legacy-follow-up-scheduler/
│   └── patient-follow-up-writer/
└── diagnostic-result-follow-up-review/
```

The solution has exactly one `.uipx` manifest and is independently deployable. The coded action app remains independently deployed if required by the platform, with its versioned contract pinned by solution configuration. Before packaging, run `uip solution resources refresh`, restore dependencies, and dry-run pack. Pull requests validate only the changed solution; publishing and deployment occur only after merge to `main` through `playground-deploy`, using an immutable package version and the approved Playground parent `JD_Demos/demos`.

## Human decisions

- **Reviewer and decision:** The ordering clinician or an explicitly accepted covering clinician owns `AcceptPlan`, `EditPlan`, `UrgentEscalation`, `NoActionWithRationale`, or `RequestContext`. The task shows source/version, report spans, deterministic flags, ownership, limited context, current policy passages, citations, confidence, missing context, and advisory plan.
- **Task experience:** The coded action app receives source facts and evidence as read-only inputs. Editable `inOut` fields are `approvedDisposition`, `approvedFollowUp`, `responsibleClinicianId`, and `dueAt`. `rationale` is required for edits, no action, and escalation. It returns the named outcome, edited values, reviewer ID, policy/result version acknowledgment, and timestamp.
- **Timeout and transfer:** `reviewTimeoutMinutes = 15` is a demo-only configuration, not a clinical SLA. Timeout sends an operational escalation to the synthetic covering pool, leaves `acknowledgedAt` empty, and keeps the case open. A covering clinician must explicitly accept ownership before completing the review.
- **Downstream routes:** `AcceptPlan` and `EditPlan` enable the mock task/order, scheduling, and draft branches; `UrgentEscalation` creates only the urgent manual path until a clinician records a plan; `NoActionWithRationale` writes the signed mock documentation receipt and closes without scheduling; `RequestContext` creates a coordinator task and returns a versioned context response to a fresh review.

## Controls and safety

| Control | Design decision | Evidence in the Flow or demo |
| --- | --- | --- |
| Deterministic safety routing | Source-critical flag, unresolved ownership, cancelled status, amendment state, extraction confidence, and policy availability are evaluated outside the agents. Agent confidence can never suppress escalation or clinician review. | Real expressions in segments 1–3 and exception paths below the happy path. |
| Clinical authority | Agents summarize and draft only. The clinician task owns acknowledgment, disposition, plan edits, due date, and no-action rationale. | Completion handle, returned outcome/field IDs, empty pre-task `acknowledgedAt`, and write paths gated by validated human output. |
| Access and data | Synthetic data, least-privilege identities, minimum necessary context, role-restricted tasks, managed connections, masked traces, and no direct patient identifier on the canvas. | Fixture inventory, tool schemas, task roles, connection bindings, and redaction assertions. |
| Grounding and amendments | Every material recommendation links to result and current-policy evidence. A new version marks earlier advice stale and requires new review. | Evidence/policy refs, source/prompt/model versions, amendment link, and five-case evaluation. |
| Communication safety | Only a clinician-approved plan and approved template may seed a preview. Restrictions are checked before drafting, and the demo has no send tool. | MCP trajectory, safety evaluator, `deliveryStatus = "not_sent"`, and held-preview route. |
| External showcase isolation | Azure AI Foundry receives constants only and cannot affect case state, evidence, routing, acknowledgment, plan, receipts, communication, or status. | No output mapping into business variables and off/on/failing isolation evaluation. |
| Resilience and receipt truth | Idempotent intake, bounded transient read retries, no automatic retry after ambiguous writes, owned exceptions, and required receipt reconciliation. | Versioned queue reference, typed exception array, merge, and final reconciliation expression. |

## Error paths and recovery

| Failure | Safe route | Recovery owner and condition |
| --- | --- | --- |
| Invalid schema, cancelled result, or unsupported version | Stop before agent invocation or writes and record the validation result. | Diagnostic follow-up coordinator corrects/rejects intake and resubmits with the same version-aware key when valid. |
| Low-confidence PDF extraction or structured/PDF conflict | Preserve both sources and open manual reconciliation before assessment. | Coordinator confirms the canonical fields and source references; a corrected case version resumes. |
| Source-critical flag or unresolved clinician ownership | Set `urgent_escalation`; do not wait on routine prioritization or allow confidence to downgrade the route. | Covering pool accepts ownership and completes the clinician task. |
| Missing/expired policy, unavailable context tool, or uncited agent output | Discard the recommendation and present source facts with `insufficient_evidence`. | Clinician may review facts; platform/clinical-policy owner restores approved evidence before advisory output resumes. |
| Clinician task timeout | Leave `acknowledgedAt` empty, keep the case open, and escalate to the covering pool. | Patient-safety lead monitors; accepted covering clinician resumes review. |
| API or RPA read failure | Retry transient reads at most twice, then set a typed `technical_exception`. | Platform/RPA owner restores the dependency and explicitly retries the failed activity. |
| Ambiguous or failed clinical-system/scheduling write | Do not retry automatically and never create a success receipt. Hold closure and communication preview. | Coordinator reconciles target state, records evidence, and explicitly resumes or completes the action. |
| Template or draft safety-check failure | Create a manual-draft task with `deliveryStatus = "not_sent"`; preserve other receipts. | Communications/clinical owner supplies approved copy. |
| Azure AI Foundry timeout, error, or unexpected response | Record only transient `externalAgentShowcaseStatus`, discard the response, and rejoin the same core route. | Demo owner may disable the branch; clinical users take no recovery action because case state is unchanged. |

## Observability and evaluation

During specification inspection on August 12, 2026, the installed CLI was version 1.199.0, exposed Flow validation/format/evaluation commands, and was authenticated as `james.dickson@uipath.com` to `uipathlabs / Playground`. The Azure AI Foundry node was tenant-available, and `JD_Demos/demos` is the approved deployment parent. These facts support planning only; no Flow, domain resource, or Flow-specific binding was created or validated by this specification task.

| Signal or test | What it proves | Expected result or threshold |
| --- | --- | --- |
| Correlation and audit | Result/version ownership, evidence, recommendation, reviewer changes/outcome, receipts, amendment, and recovery can be reconstructed. | Every trace, task, receipt, and audit event contains `organizationKey`, `resultId`, and `resultVersion`; no direct patient identifier or unmasked contact data appears in logs. |
| Flow route evaluator | Each fixture reaches the correct safe route and final status. | 5/5 initial synthetic cases match expected escalation/review route and status before promotion. |
| Safety-route evaluator | Source-critical and unresolved-owner inputs cannot be downgraded by model output. | 100% enter `urgent_escalation`; zero routine scheduling/message operations occur first. |
| Grounded-assessment evaluator | Advisory plans use only supplied result/context and the pinned synthetic policy. | 100% of material claims contain valid evidence and policy references; unsupported clinical claims occur zero times. |
| Tool-use/trajectory evaluator | Agents call only their required read-only context/policy or template tools. | Required calls in every applicable case; zero unapproved write, send, web-search, or patient-lookup calls. |
| External showcase isolation | The placeholder external agent cannot change a clinical business result. | With showcase off, on, or failing, identical `AbnormalResultCase`, route, decision, receipts, and final status; only transient showcase status may differ. |
| Receipt reconciliation | Closure never hides failed, ambiguous, or held work. | `closed` only when every outcome-required receipt is present and `succeeded`; failure fixture remains open with an owned exception. |

### Synthetic evaluation set

Dataset name: `healthcare-provider-abnormal-result-follow-up-v1`.

| Case | Expected route | Expected business output |
| --- | --- | --- |
| Abnormal radiology result with complete context | Advisory plan enters clinician review; clinician selects `AcceptPlan`. | Mock EHR task/order, scheduling receipt, non-sent grounded preview, acknowledgment, and `closed`. |
| Source-critical result with absent ordering clinician | Immediate covering-pool escalation before routine assessment/action. | `urgent_escalation`, empty pre-review acknowledgment, and no routine scheduling or communication. |
| Amended report | Prior assessment is marked stale and a new review is required for the new result version. | Version-linked audit history, no reuse of old approval, and new clinician decision. |
| Low-confidence scanned PDF | Manual reconciliation before agent assessment. | Corrected canonical fields/source refs and `review_required`; no silent extraction acceptance. |
| Mock EHR write failure after approval | Parallel branches merge into reconciliation exception. | No false receipt or closure, preview held, `technical_exception`, and coordinator recovery data. |

## Demo script

1. Show the four-segment canvas, the exception paths below it, and the disabled external-agent showcase branch.
2. Submit the complete-context synthetic radiology fixture and point out the version-aware queue reference and source-designated flags.
3. Open the assessment trace to show the minimum-context and pinned-policy tool calls, evidence references, missing-context array, confidence, and advisory-only boundary.
4. Open `Diagnostic Result Follow-Up Review`, compare the source conclusion and policy passage with the proposed plan, edit the follow-up window, add rationale, and select `EditPlan`.
5. Return to Flow and show mock EHR documentation/order, legacy scheduling, and non-sent patient-preview branches running independently and merging.
6. Open the queue record to show clinician acknowledgment, returned edits, policy/result/model/tool versions, mock receipts, final status, and ordered audit events.
7. Point out that the Azure AI Foundry node receives only a constant, discards its output, and produces identical clinical state whether disabled, successful, or failing.
8. Preview the source-critical/missing-owner fixture to show immediate covering-pool escalation and no routine action before human ownership.

## Success measures

- **Business proof:** The demo makes acknowledgment time, overdue results, documented-plan coverage, patient-notification status, follow-up-task completion, amendment re-review, and exception aging measurable without promising a target improvement.
- **Flow proof:** A viewer can see FHIR/PDF intake, document intelligence, deterministic safeguards, two material agent roles, the non-material Azure AI Foundry showcase, API/RPA contrast, human authority, parallel follow-up, merge, and safe recovery.
- **Demo proof:** In under ten minutes, a viewer can verify source/version provenance, evidence-linked advisory reasoning, clinician correction, downstream use of returned fields, reconciled mock receipts, and a critical-ownership exception.
- **Build proof:** Every project validates without warnings, the five-case evaluation set passes, bindings are recorded, `resources refresh` and dry-run pack succeed, and the immutable package can follow changed-solution CI into Playground after merge.

## Reference mapping

| Reference requirement | Domain-specific implementation | Evidence or gap |
| --- | --- | --- |
| 3–4 segment topology and canvas rules | Four blue sticky notes: Receive and normalize the result; Assemble evidence and ownership; Clinician owns the follow-up; Complete and prove the loop. Happy path is left-to-right, exceptions are below, and parallel branches are symmetric. | Fully specified; canvas implementation remains. |
| IXP/document intelligence, when relevant | `HealthcareProviderDiagnosticReportExtraction` handles three synthetic legacy-PDF fixtures with eight required fields and a `0.90` review threshold. | Contract and offline fixture fallback specified; project/model/folder remain to provision. |
| API workflow and RPA on the intended path | `diagnostic-result-gateway` normalizes, reads context, and records mock follow-up; `legacy-follow-up-scheduler` uses a UI-only local scheduler after approval. | Contracts specified; projects, endpoint/UI fixtures, and production API-gap validation remain. |
| Inline agent with a wired tool | `Follow-Up Evidence Synthesizer` uses minimum-context and pinned-policy tools and returns evidence, gaps, recommendation, and confidence. | Contract specified; model, tools, and context index remain unprovisioned. |
| Coded agent with visible value-add | `Patient Follow-Up Writer` retrieves an approved template, drafts a non-sent preview, and self-checks clinical/data boundaries. | Contract specified; coded agent and MCP server remain to build. |
| Shared external-agent showcase | Azure AI Foundry node/connection on a disabled-by-default branch with constant input, no `thread_id`, discarded output, and fail-open rejoin. | Shared connection verification is preserved and node availability is current; Flow binding and selected agent remain to validate. |
| Real business decision and safe exception | Critical/ownership/confidence/policy expressions plus named clinician outcomes; amendment, extraction, timeout, and receipt exceptions remain safe. | Fully specified; expressions remain to implement and evaluate. |
| Human decision and returned outcome data | Coded action app returns named outcome, approved/edited plan, responsible clinician, due date, rationale, reviewer/version acknowledgment, and timestamp; Flow consumes them downstream. | Contract specified; clinical authority and timing remain human-owned validations. |
| Purposeful parallelism and merge | Mock EHR task/order, UI-only scheduling, and communication preview run independently after approval and merge before reconciliation. | Fully specified; outcome-to-required-receipt matrix remains to encode. |
| Evaluation set and evaluator | Five fixtures plus route, safety, grounding, trajectory, isolation, and receipt checks with exact initial thresholds. | Fully specified; fixtures/evaluators remain to build. |
| Process-app variant | Not selected. The Orchestrator queue is the canonical demo record. | Later cross-domain selection may change the canonical record/app design. |
| Solution boundary and delivery contract | One `healthcare-provider-abnormal-result-follow-up` solution, exactly one `.uipx`, nested Flow project, resource refresh, immutable version, changed-solution CI, and deployment to approved Playground parent `JD_Demos/demos`. | Folder path approved; resources still require provisioning. |

## Open human decisions

These decisions refine implementation but do not block the synthetic, mock-backed build. They must be resolved before replacing a corresponding mock or using non-synthetic data.

| Decision | Owner | Resolution path |
| --- | --- | --- |
| Confirm the ambulatory-radiology scope and representative result subtype. | Clinical governance and radiology owners | Approve the synthetic incidental-finding scenario or revise fixtures, rules, and policy corpus for another result type. |
| Approve abnormal/critical rules, policy corpus, and operational thresholds. | Clinical governance and patient-safety owners | Supply versioned approved rules and replace the `0.90`, `0.85`, and 15-minute demo settings with sanctioned test configuration. |
| Confirm clinical ownership and accepted-transfer semantics. | Medical staff, ambulatory operations, and identity/coverage owners | Map ordering clinician, departure/leave, cross-coverage, explicit acceptance, and escalation pool to source-system fields. |
| Select the EHR/integration profile and writable resources. | Enterprise architect, EHR owner, and security owner | Choose vendor/sandbox, FHIR release/profile, event mechanism, scopes, patient-matching rules, and supported `Task`/`ServiceRequest`/communication operations. |
| Validate the scheduling API gap and RPA responsibility. | Scheduling product owner and enterprise architect | Retain RPA only if no governed scheduling API exists; otherwise choose another credible UI-only responsibility or approve a reference deviation. |
| Approve patient communication and result-release controls. | Clinical, privacy, legal, accessibility, and communications owners | Confirm channel consent, restrictions, language/accessibility, portal-release behavior, approved templates, and escalation channels. |
| Approve environment, model, privacy, and records controls. | Security, privacy, legal, AI governance, and records owners | Confirm agreements, synthetic-only scope, model/provider approval, access, retention, audit consumer, downtime, and incident ownership before broader testing. |
| Set pilot baselines and target measures. | Patient-safety and ambulatory-operations owners | Supply observed acknowledgment, overdue, notification, completion, amendment, and exception-aging baselines before benefit claims. |
| Decide whether healthcare provider is a Data Fabric/process-app variant. | Demo portfolio owner | Resolve in the later cross-domain selection issue; this specification currently marks it not selected. |

## Implementation tasks

1. Scaffold `healthcare-provider-abnormal-result-follow-up` with the nested Flow project and exactly one `.uipx` manifest.
2. Build the five-case fixture set, synthetic policy corpus, canonical queue contract, mock FHIR endpoint, and local scheduling UI.
3. Implement and validate `diagnostic-result-gateway`, `HealthcareProviderDiagnosticReportExtraction`, and `legacy-follow-up-scheduler` with typed failures and versioned receipts.
4. Build the inline evidence synthesizer, two read-only tools, deterministic safety rules, and grounding/safety evaluators.
5. Build the coded patient follow-up writer, least-privilege template MCP tool, and trajectory/output evaluators.
6. Build and deploy `Diagnostic Result Follow-Up Review`, then wire its completion handle, named outcomes, returned fields, accepted ownership, timeout, and context-request loop into Flow.
7. Author the four-segment Flow, amendment handling, exception routes, non-material Azure showcase, parallel completion branches, merge, and receipt reconciliation.
8. Bind and validate Azure AI Foundry connection `0107247a-0197-42c9-b957-05d1b722b111`; prove off/on/failing variants cannot change case data, evidence, route, acknowledgment, decision, actions, receipts, communication, or status.
9. Run project validation and the five-case Flow/agent evaluation set; resolve every validator warning and failed threshold without executing against live clinical systems.
10. Refresh solution resources, restore dependencies, dry-run pack, and register immutable-version deployment configuration for changed-solution CI.

## Quality rubric

| Dimension | Score (0–3) | Evidence and remaining gap | Owner and resolution path |
| --- | ---: | --- | --- |
| Enterprise credibility | 2 | Consequential safety workflow, accountable roles, data contract, controls, and evidence-backed measures are explicit; provider policy, systems, and baselines remain unverified. | Clinical, patient-safety, privacy, and system owners validate during discovery. |
| Flow differentiation | 3 | Four segments visibly coordinate IXP, deterministic safety logic, agents/tools, API, RPA, human review, parallel completion, merge, and recovery. | Flow implementer preserves the topology and validates registry definitions, route expressions, and bindings. |
| Demo clarity | 3 | The plan-edit hero journey plus critical/missing-owner exception have named proof points and a timed script. | Demo owner builds representative fixtures and rehearses after deployment. |
| Build feasibility | 2 | Inputs, outputs, mocks, fallbacks, solution boundary, evaluation, authenticated target, current CLI, and shared external-node readiness are recorded; domain resources remain unprovisioned. | Tenant administrator and implementers provision resources and record bindings. |
| **Total** | **10/12** | **Ready for implementation planning as a synthetic demo; not ready for production clinical integration, non-synthetic data, patient communication, or deployment until owned gaps are resolved.** | **Start with solution/fixtures, then close each human decision before replacing its mock.** |
