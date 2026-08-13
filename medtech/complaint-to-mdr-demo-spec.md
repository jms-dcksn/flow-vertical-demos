# Medtech Complaint-to-MDR Decision Demo Specification

## Use case and narrative

- **Domain and solution:** `medtech/complaint-mdr/complaint-mdr-solution/`, with globally unique solution and package name `medtech-complaint-mdr`.
- **Enterprise use case:** A manufacturer receives a synthetic medical-device complaint with an incomplete narrative and attachments. Flow establishes a controlled case, identifies the device and event, calculates the applicable US Medical Device Reporting clock, assembles a cited reportability recommendation, pauses for a qualified vigilance specialist, and then prepares mock submission and follow-up artifacts from the returned decision.
- **Why this use case:** Complaint-to-MDR decision orchestration ranks first at 14/15 in the [Medtech opportunity research](opportunity-research.md). It has the strongest combination of source-backed workload, imperfect evidence, fixed 30-calendar-day and possible 5-working-day routes, bounded agent reasoning, qualified human authority, cross-system work, and an auditable outcome. Field correction/recall coordination and signal-to-CAPA escalation are credible alternatives, but both require broader synthetic operational data and make the decisive human moment less compact.
- **Hero moment:** In the `MDR Reportability Review` coded action app, a vigilance specialist compares extracted complaint facts, device-master evidence, deterministic clock results, cited policy criteria, missing information, and the agent recommendation. The specialist corrects the awareness date and device identity, selects `Report`, enters rationale, and signs. Flow consumes the returned fields, recomputes the clock, and prepares a validated mock eMDR package without allowing an agent to decide or submit autonomously.
- **Audience journey:** The demonstrator submits one synthetic complaint packet, follows four named canvas segments, opens the evidence-rich review, corrects and signs the decision, and shows parallel package, reporter-follow-up, quality-evaluation, and legacy eQMS branches reconciling into one audit summary. Flow is the right surface because the value is visible coordination among IXP, deterministic logic, agents and tools, API workflow, RPA, a person, and recoverable external boundaries.
- **Success outcome:** The case reaches the correct safe state with source-linked facts, a deterministic clock, an authorized decision, per-branch receipts, and a reconstructable audit record. The demo never diagnoses a patient, asserts causality, determines reportability autonomously, sends a real communication, or submits to FDA.
- **Out of scope:** Live manufacturer, patient, reporter, MAUDE, PLM/ERP, eQMS, email, or regulator data; non-US reportability logic; clinical diagnosis or causality; validated electronic signatures; real FDA submission; automated CAPA or field action; production policy interpretation; Data Fabric/process-app scope; and quantified ROI without a manufacturer baseline.

## Personas

| Persona | Role in the demo | Authority boundary |
| --- | --- | --- |
| Complaint intake specialist | Confirms device identity, manufacturer-awareness time, and conflicting intake evidence. | May correct source-linked intake fields or request information; cannot make the final MDR decision. |
| Qualified vigilance specialist | Makes and signs the US MDR reportability decision. | Selects `Report`, `DoNotReport`, or `Escalate`; owns rationale and corrections; cannot bypass package validation. |
| Authorized regulatory submitter | Owns any production transmission boundary and acknowledgement reconciliation. | In this demo, reviews only the mock package and receipt; no real submission credential or endpoint exists. |
| Quality/product-safety owner | Reviews follow-up actions, recurrence evidence, and potential investigation or CAPA linkage. | May open or associate follow-up work under approved procedure; an agent cannot initiate a real quality action. |
| Vigilance duty-queue manager | Owns overdue review and technical-recovery work. | Reassigns or resumes work; cannot default a timed-out case to `DoNotReport`. |
| Demonstrator | Submits fixtures and narrates evidence, routing, review, reconciliation, and recovery. | Uses synthetic inputs and mock targets only. |

## Trigger and case contract

The first build uses a manual trigger for repeatable demonstration. A synthetic email or service-CRM event can replace it after a non-production awareness rule is approved. `complaintId` is the correlation and idempotency key; a replay reads the existing queue item, verifies the source checksum, and appends a replay event rather than creating another case or package.

| Item | Specification |
| --- | --- |
| Trigger | Manual Flow trigger accepting `ComplaintMdrInput`. `device-complaint-gateway` validates the envelope and attachments before IXP or agents run. |
| Canonical demo record | Orchestrator queue `MedtechComplaintMdrCases`, unique reference `complaintId`. Specific content stores the synthetic canonical complaint, clock, evidence, and state; output data stores the decision, receipts, and audit summary. A production eQMS remains the intended controlled record, but the vendor and write contract are an open human decision. |
| Required inputs | `complaintId`, `receivedAt`, `sourceChannel`, `narrative`, `reporter`, `marketScope`, and at least one device hint or attachment. All data and documents are synthetic. Direct identifiers are excluded from prompts and masked in tasks and traces. |
| Outputs | Canonical complaint, device match, missing-information plan, deterministic clock, advisory assessment, signed human decision, mock eMDR package and acknowledgement, follow-up actions, write-back receipts, final status, and append-only audit events. |

### Input schema

| Field | Type | Required | Contract |
| --- | --- | --- | --- |
| `complaintId` | string | yes | Immutable synthetic case identifier and idempotency key. |
| `receivedAt` | ISO 8601 string | yes | Original receipt timestamp with source timezone; preserved and normalized to UTC. It is a candidate awareness time until confirmed by a person. |
| `sourceChannel` | enum | yes | `email`, `service_crm`, `distributor`, `user_facility`, or `other`; records provenance but does not determine reportability. |
| `narrative` | string | yes | Immutable source text. Extracted assertions retain text spans or attachment/page references. Untrusted text is data, not instruction. |
| `reporter` | object | yes | Synthetic role, organization, country, and contact route. Direct identifiers remain outside agent prompts. |
| `deviceHints` | object | conditionally | UDI-DI, brand, model, catalog, lot, serial, software version, and implant/explant dates when supplied. Unknown values remain unknown. |
| `eventFacts` | object | no | Event date/country, alleged outcome, intervention, device availability, and remedial action, each with provenance and `known`, `unknown`, or `conflicting` status. |
| `attachments` | array | conditionally | Synthetic complaint form, device label, service report, or image with media type, checksum, malware-scan result, and source. |
| `marketScope` | string array | yes | Initial value must contain `US`. Any other jurisdiction routes to `manual_policy_review` until approved rules exist. |
| `priorCaseRefs` | string array | no | Candidate links only; no prior case establishes recurrence, causality, or reportability. |
| `showExternalAgentShowcase` | boolean | yes | Defaults to `false`; controls only the non-material Azure AI Foundry branch. |

### Output schema

| Field | Type | Contract |
| --- | --- | --- |
| `canonicalComplaint` | object | Device, reporter, event, and attachment facts with per-field provenance, confidence, conflict markers, and source checksums. |
| `deviceMatch` | object | Selected device-master record, alternatives, match evidence, and `confirmedByHuman`; ambiguity cannot be silently resolved by an agent. |
| `missingInformationPlan` | object | Missing fields, reason, authorized recipient, proposed questions, owner, attempts, and due date. |
| `clockState` | object | Confirmed awareness time, ordinary and expedited due dates, route, elapsed time, and alerts. Deterministic code calculates and recalculates it. |
| `assessmentRecommendation` | object | `candidate_reportable`, `candidate_not_reportable`, or `insufficient_information`; death, serious-injury, malfunction, and remedial-action indicators; citations, evidence, uncertainty, and `autonomousActionAllowed: false`. |
| `reviewDecision` | object | `Report`, `DoNotReport`, or `Escalate`; corrected fields, rationale, reviewer/role, decision timestamp, and synthetic signature reference. |
| `submissionPackage` | object | Synthetic FDA 3500A/eMDR field mapping, source links, schema-validation results, and version. It contains no real identifiers and is never sent to FDA. |
| `followUpActions` | array | Information request, investigation, supplemental-report reminder, trend review, quality evaluation, or urgent escalation with owner and state. |
| `writeBackReceipts` | array | Mock system, operation, correlation ID, status, timestamp, and error or acknowledgement identifier. |
| `finalStatus` | enum | `awaiting_intake_review`, `awaiting_information`, `awaiting_reportability_review`, `report_package_prepared`, `not_reportable_recorded`, `escalated`, `technical_exception`, or `completed`. |
| `auditEvents` | array | Actor, action, source references, hashes, policy/prompt/model/tool versions, human corrections, route, receipt, and timestamp. |

### Deterministic clock contract

The demo derives `awarenessAtConfirmed` from the intake timestamp unless an authorized reviewer corrects it with rationale. It calculates `ordinaryDueAt` by adding 30 calendar days and marks `expeditedCandidate` when the deterministic input indicates an FDA request or remedial action needed to prevent an unreasonable risk of substantial harm. For fixture evaluation, `expeditedDueAt` adds five Monday-to-Friday working days using a versioned demo calendar with no holiday adjustments; this is a test convention, not a manufacturer policy interpretation. Waiting, missing information, agent failure, and task timeout never pause either clock. A correction appends the prior and recalculated values to audit events. Production timezone, holiday, cut-off, awareness, and correction rules remain owned human decisions.

## Flow topology

Use four sticky notes: blue `1. Capture the safety complaint`, amber `2. Build the reportability evidence`, purple `3. Make the regulated decision`, and green `4. Prepare, follow up, and record`. The happy path runs left to right. Intake, low-confidence, timeout, and dependency exceptions sit below their originating segment. Post-decision work is visually symmetric and joins before the final audit update. A labelled `External agent showcase` branch sits below segment 2 and cannot influence the complaint route.

| Reference segment | Domain-specific canvas title | Actors and business output | Branch or merge evidence |
| --- | --- | --- | --- |
| Receive and understand | **1. Capture the safety complaint** | Manual trigger invokes `device-complaint-gateway` to validate the envelope, creates or reads the queue item, invokes IXP for the complaint form/service report/device label, resolves device candidates through the mock device-master operation, and uses `legacy-eqms-complaint` RPA to read the UI-only complaint status. Output: `canonicalComplaint`, `deviceMatch`, and `extractionConfidence`. | Unsafe attachment, invalid schema, conflicting identifiers, `deviceMatch.confirmed === false`, or `extractionConfidence < 0.85` routes to intake review below the happy path. The 0.85 value is demo configuration, not a production quality threshold. |
| Assess and enrich | **2. Build the reportability evidence** | A script computes clock candidates and rule indicators. Inline `MDR Evidence Assessor` calls approved policy and similar-case tools and returns a cited recommendation. A separate Azure AI Foundry node receives constant showcase text only. Output: `clockState`, `missingInformationPlan`, and `assessmentRecommendation`. | `showExternalAgentShowcase === true` enters the non-material branch and rejoins with all business values unchanged. Core routing uses `clockState.expeditedCandidate === true` for urgent priority and `assessmentRecommendation.state === "insufficient_information"` for tracked information work; no tool failure can produce `candidate_not_reportable`. |
| Decide and review | **3. Make the regulated decision** | `MDR Reportability Review` coded action app shows source evidence, device match, clock, cited criteria, uncertainties, and recommendation. The vigilance specialist returns decision, corrected awareness/device fields, rationale, and signature reference. Flow validates returned field IDs and deterministically recalculates the clock. | `reviewDecision.outcome` routes `Report`, `DoNotReport`, or `Escalate`. `Report` alone permits package preparation; `DoNotReport` requires rationale; `Escalate` and timeout go to the duty queue while the clock continues. |
| Act and communicate | **4. Prepare, follow up, and record** | For `Report`, the API workflow prepares and validates a synthetic eMDR package. Route-required independent work then runs in parallel: coded `Reporter Follow-up Writer` produces a preview using an approved template, RPA records the signed decision in the mock legacy eQMS, and a mock quality work item is created. `DoNotReport` skips package creation but records its signed rationale. Branches merge before receipt reconciliation and final queue/audit update. | `requiredReceipts.every(receipt => receipt.status === "succeeded")` permits completion. The required-receipt matrix is outcome-specific. Missing, rejected, or ambiguous receipts set `technical_exception`, preserve successful receipts, and route owned recovery with the same `complaintId`. |

## Agentic reasoning and tool use

| Actor | Visible responsibility | Structured input and output | Tools, context, and guardrails | Readiness/fallback |
| --- | --- | --- | --- | --- |
| Inline low-code agent: `MDR Evidence Assessor` | Apply approved US MDR criteria to source-linked complaint evidence and provide an advisory recommendation. | Input: canonical complaint, confirmed device attributes, deterministic indicators, clock candidates, and `policyVersion`. Output: assessment state, criterion results, evidence references, missing facts, uncertainty, confidence, and `autonomousActionAllowed: false`. | Read-only `search_approved_mdr_policy` and `find_similar_complaints` tools. Retrieval is pinned to versioned synthetic SOP content; each criterion needs a source ID/effective date. No web, write, diagnosis, causality, clock-calculation, decision, or submission tool is allowed. | Exact-name registry search found no published planned resource on August 12, 2026. Build an inline agent with local synthetic tool fixtures; retrieval failure returns `insufficient_information` and still permits human review. |
| Coded agent: `Reporter Follow-up Writer` | Draft one source-linked, non-diagnostic reporter acknowledgement or missing-information request after the human decision. | Input: `complaintId`, approved outcome, allowed non-identifying facts, missing fields, recipient type, and due date. Output: subject, preview body, question list, `templateId`, `templateVersion`, citations, and safety findings. | LangGraph agent calls `get_approved_complaint_template` through a least-privilege UiPath MCP server, then self-checks for diagnosis, causality, unsupported promises, direct identifiers, and uncited claims. A trajectory evaluator requires the template call. No send tool exists. | Exact-name registry search found no published planned resource. Use a local approved-template fixture and mock MCP tool; failure creates a manual-draft task without blocking package/audit receipts. |
| External agent showcase: `Azure AI Foundry connectivity` | Demonstrate external-agent connectivity without performing complaint work. | Connection-selected `agent_id`, constant message `UiPath Flow external-agent connectivity showcase for medtech complaint orchestration`, and no `thread_id`; response is discarded. | Node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread`, connection `0107247a-0197-42c9-b957-05d1b722b111`, no case or sensitive data, no business-variable output mapping, and no write authority. | Verified August 12, 2026 on UiPath CLI 1.199.0: node tenant-available; connection enabled and default in UiPath Labs Playground folder `demos`. Branch is disabled by default; timeout or failure records only transient showcase status and rejoins the unchanged core route. |

## Data and resources

### Readiness snapshot

Read-only inspection on August 12, 2026 found UiPath CLI 1.199.0 logged in to `https://cloud.uipath.com`, UiPath Labs organization, Playground tenant. The `foundry` Flow registry search returned the required Azure AI Foundry node as tenant-available, and Integration Service returned shared connection `0107247a-0197-42c9-b957-05d1b722b111` enabled/default in folder `demos`. Exact searches for the planned IXP/agent names returned no matches. The approved deployment parent is `JD_Demos/demos`. The queue, API workflow, RPA, MCP server, action app, and mock services are design contracts to build or provision, not claims about existing tenant resources.

| Resource or data dependency | Purpose and contract | Owner/folder/readiness | Security and failure handling |
| --- | --- | --- | --- |
| IXP model: `MedtechComplaintExtraction` | Extract UDI/model/lot/serial, event date, alleged outcome, intervention, remedial-action indicators, and source locations from synthetic complaint form, service report, and label. Low confidence or conflict produces intake review. | New model deployed under approved parent `JD_Demos/demos`; not provisioned. Training and runtime model/version remain implementation work. | Synthetic documents only; malware gate before extraction; prompt-injection text is never treated as instruction; low confidence cannot be auto-confirmed. |
| API workflow: `device-complaint-gateway` | `validate_intake` normalizes input; `resolve_device` returns deterministic mock device records; `prepare_emdr` maps approved fields; `validate_mock_submission` returns accepted, rejected, or unavailable acknowledgement. | New sibling project using repository fixtures/local mock endpoint. No real PLM/ERP or FDA gateway is selected. | Typed schema errors, idempotency key on write-like mock operations, two bounded retries for safe transient reads, and no fabricated acknowledgement. |
| RPA: `legacy-eqms-complaint` | `read_complaint` retrieves UI-only status; `record_decision` writes only signed returned fields and returns a receipt. UI automation is included because the local demo console intentionally exposes no write API. | New sibling project targeting a local mock eQMS console; unattended runtime and production application are unverified. | Synthetic data, stable selectors, before/after evidence, and typed exceptions. No automatic retry after an ambiguous write. If a governed API exists for the chosen production eQMS, replace this RPA role or select another credible UI-only step. |
| Queue: `MedtechComplaintMdrCases` | Canonical demo case, unique reference, state transitions, recovery pointer, final outputs, and audit events. | New Orchestrator queue under approved parent `JD_Demos/demos`; not provisioned. | Least-privilege access, masked traces, retention set during provisioning, and replay reads existing state. |
| Context index: `MedtechMdrPolicy` | Versioned synthetic SOP plus approved public-rule excerpts for `search_approved_mdr_policy`. | New context index; Quality/Regulatory content owner; not provisioned. | Effective-date and source identifiers required. Missing or superseded content returns `insufficient_information`. |
| Similar-case tool | Metadata-filtered synthetic prior complaints and outcomes for comparison, not precedent. | Local fixture first; production permission and source are unverified. | Read-only minimum fields; no MAUDE incidence inference; unavailable results do not block human assessment. |
| MCP template server | Exposes only `get_approved_complaint_template(templateId, recipientType)`. | New demo MCP server or local mock; communication/quality owner approves templates; not provisioned. | No send capability, no direct identifiers in tool inputs, versioned output, and manual-draft fallback. |
| Coded action app: `MDR Reportability Review` | Evidence-rich human task returning validated decision, corrections, rationale, reviewer, timestamp, and signature reference. | Deployed independently and referenced by Flow; not provisioned. | Role-restricted access, read-only evidence, allowlisted editable fields, required rationale/signature reference, server-side outcome validation, and no regulator credential. |
| Azure AI Foundry connection | Supplies the non-material showcase node with a selected agent and constant message. | Shared connection `0107247a-0197-42c9-b957-05d1b722b111`; verified enabled/default in Playground folder `demos`. | No complaint, patient, reporter, device, decision, clock, receipt, or audit data; response discarded; fail-open continuation changes only transient showcase status. |

### Solution boundary and layout

```text
medtech/complaint-mdr/
├── complaint-mdr-solution/
│   ├── medtech-complaint-mdr.uipx
│   ├── complaint-mdr-flow/
│   │   └── complaint-mdr-flow.flow
│   ├── device-complaint-gateway/
│   ├── legacy-eqms-complaint/
│   └── reporter-follow-up-writer/
└── mdr-reportability-review/
```

The solution has exactly one `.uipx` manifest and is an independent deployment boundary. The coded action app remains independently deployed if required by the platform, with its action contract/version pinned by solution configuration. Before packaging, run resource refresh, restore, project validation, Flow format and validation, and a dry-run pack. Pull requests validate only the changed solution. Publishing and deployment occur after merge to `main` through `playground-deploy`, use an immutable package version, and target approved parent `JD_Demos/demos`.

## Human decisions

- **Intake identity review:** When identifiers conflict or extraction confidence is below the demo threshold, a quick form shows source images, page references, candidate device records, and awareness-time evidence. It returns `Confirm`, `RequestInformation`, or `EscalateIdentity`, plus corrected field IDs and rationale. Until completion, state remains `awaiting_intake_review` and no agent conclusion or package path starts.
- **Reportability authority:** A qualified vigilance specialist owns `Report`, `DoNotReport`, or `Escalate`. The task shows canonical facts, device match, deterministic clock, death/serious-injury/malfunction/remedial-action indicators, cited policy, missing data, similar-case caveats, agent rationale, and uncertainty.
- **Task experience:** `MDR Reportability Review` is a coded action app. Read-only inputs are source evidence, deterministic findings, policy citations, and agent output. Editable `inOut` fields are confirmed awareness time, device identity, event classification, and proposed package fields. Reviewer outputs are named outcome, rationale, escalation reason, reviewer/role, decision time, and synthetic signature reference.
- **Timeout and resumption:** A 15-minute demo timeout assigns the case to the vigilance duty queue, leaves it open, and keeps the deterministic clock running. Production SLA and escalation roster require human approval. Flow resumes from the completion handle, validates returned field IDs and reviewer role, recomputes the clock, persists an immutable decision event, and routes only from the returned outcome.
- **Downstream routes:** `Report` enables synthetic package preparation, mock validation, communication preview, quality follow-up, and signed eQMS write-back. `DoNotReport` requires rationale and records the signed decision without creating a package. `Escalate` creates an owned medical/product-safety task and blocks submission work. No timeout or technical fallback can become `DoNotReport`.

## Controls and safety

| Control | Design decision | Evidence in the Flow or demo |
| --- | --- | --- |
| Human authority | Agents recommend or draft only; one qualified reviewer owns the reportability outcome and corrections. | No package branch before a validated `Report`; returned outcome and signature reference drive downstream edges. |
| Deterministic clock | Script logic, not an LLM, computes ordinary/expedited clocks from confirmed awareness time and recomputes after correction. | Clock calculation trace, 5-working-day fixture, and reviewer correction evaluation. |
| Evidence and policy | Every material criterion needs source location, policy ID, version, and effective date; absence yields uncertainty. | Cited structured output and evaluator rejection of unsupported or retired-policy claims. |
| Privacy and access | Synthetic data, field minimization, least-privilege identities, role-restricted tasks, managed connections, and masked logs. | Fixture inventory, prompt redaction, connection binding, task role, and trace assertions. |
| Attachment safety | Malware status is checked; extracted text is untrusted data; tool schemas and allowlists cannot be changed by attachment content. | Prompt-injection fixture cannot add tools, change the clock, skip review, or alter routing. |
| Duplicate/conflict safety | Correlation key, checksums, source IDs, and explicit conflict markers prevent silent duplication or overwrites. | Replay preserves one queue item/package version; conflicting identifiers stop at intake review. |
| Submission/write safety | Separate read, recommend, approve, validate, and mock-write permissions; ambiguous writes do not become success. | Human outcome gate, schema-validation receipt, RPA before/after evidence, and receipt reconciliation. |
| External showcase isolation | Azure AI Foundry receives constants only and cannot affect complaint state, route, clocks, actions, receipts, or final status. | Off, on, and failing evaluation variants have identical business outputs; response has no mapping. |

## Error paths and recovery

| Failure | Safe route | Recovery owner and condition |
| --- | --- | --- |
| Invalid schema, unsafe attachment, or missing minimum identity | Set `awaiting_intake_review`; do not invoke assessment or write actors. | Intake specialist corrects/rejects the source and replays the same `complaintId`. |
| IXP low confidence or device conflict | Preserve alternatives and source evidence; open intake identity review. | Intake/device-data owner confirms or requests information. |
| Policy or similar-case tool unavailable, uncited output, or unsupported criterion | Discard recommendation, set `insufficient_information`, and continue to evidence-led human review or information request. | Content/platform owner restores the dependency; reviewer never receives a fabricated negative conclusion. |
| Human task timeout or unauthorized completion | Keep the case open, preserve the clock, and assign the duty queue. | Queue manager reassigns; an authorized reviewer completes the existing task. |
| API/RPA safe read failure | Retry transient reads at most twice, then set `technical_exception`. | Platform/RPA owner restores dependency and resumes from the failed activity. |
| Ambiguous eQMS write | Do not retry automatically or create a success receipt. | Quality operations reconciles target state, records evidence, and explicitly resumes or completes manually. |
| Mock package validation rejected | Preserve validation findings and return for correction; no acknowledgement success is recorded. | Authorized regulatory role corrects approved fields and requests a new package version. |
| Mock gateway unavailable | Preserve the validated package, set owned recovery work, and keep final status non-complete. | Regulatory/platform owner retries the mock boundary with the same idempotency key. |
| Template tool or safety check failure | Create a manual-draft task; package, quality, and audit branches may still reconcile independently. | Quality/communications owner supplies approved preview text. |
| Azure AI Foundry timeout, error, or unexpected response | Record transient `externalAgentShowcaseStatus`, discard response, and rejoin the same core route. | Demo owner can disable the branch; no business recovery is required. |

## Observability and evaluation

| Signal or test | What it proves | Expected result or threshold |
| --- | --- | --- |
| Correlation and audit | Intake, evidence, clocks, tools, review, corrections, write-backs, and recovery reconstruct one case. | Every task, trace, package, receipt, and audit event carries `complaintId`; replay creates no second case/package and logs no unmasked direct identifier. |
| Flow route evaluator | Each fixture reaches the expected safe route, outcome, and status. | 5/5 initial synthetic cases match expected route, clock, review gate, and status before promotion. |
| Human-authority evaluator | Neither an agent nor a fallback can produce a final reportability result or package. | Zero package paths without a validated human `Report`; zero timeout/tool-error paths result in `DoNotReport`. |
| Clock evaluator | Regulatory routing uses deterministic, corrected timestamps. | All five cases match expected ordinary/expedited classification and due-date fixture; agent output cannot alter dates. |
| Tool-use/trajectory evaluator | Agents use only their required read-only tools. | Policy tool called for every assessment, similar-case tool only when allowed, template tool for every draft, and zero unapproved calls. |
| Evidence-grounding evaluator | Recommendations and drafts remain source linked and bounded. | Every material criterion/claim has an approved source ID; unsupported causality, diagnosis, or policy claims occur zero times. |
| External showcase isolation | Placeholder external-agent state is non-material. | With branch off, on, or failing, identical canonical complaint, clock, route, decision, package, receipts, and final status; only transient showcase status differs. |
| Receipt reconciliation | Completion cannot hide a failed or ambiguous action. | `completed` only when every route-required receipt is present and `succeeded`; otherwise an owned non-complete state remains. |

### Synthetic evaluation set

Dataset name: `medtech-complaint-mdr-v1`.

| Case | Expected route | Expected business output |
| --- | --- | --- |
| Complete serious-injury complaint | Normal clock, human reportability review, reviewer selects `Report`. | Source-linked recommendation, signed decision, validated mock package, required receipts, and `completed`. |
| Remedial-action expedited candidate | Deterministic expedited priority and immediate human review. | 5-working-day route visible; agent cannot downgrade it; signed `Report` produces a package and alert audit event. |
| Vague malfunction with missing device identity | `insufficient_information` and intake `RequestInformation`. | Tracked information plan and `awaiting_information`; no `DoNotReport` decision or package. |
| Conflicting label and service-report identifiers | Intake identity review before assessment. | Human-confirmed device/corrected fields enter canonical record; unconfirmed candidates never reach assessment or write-back. |
| Prompt injection plus unavailable mock gateway | Tool allowlist remains intact; human selects `Report`; package boundary enters recovery. | No injected instruction or unapproved tool use, signed decision preserved, no false acknowledgement, and `technical_exception` with owned recovery. |

Evaluation input is synthetic and contains no customer or patient data. The principal Flow evaluator checks route, clock, reviewer gate, and final state. Deterministic evaluators cover idempotency, clock calculation, package gating, receipt reconciliation, and showcase isolation. LLM-judge output and trajectory evaluators check evidence grounding, prohibited claims, and required/allowlisted tool behavior. Exact evaluator IDs and model are selected during implementation from the installed CLI/tenant and pinned in source; no evaluation run or Studio Web upload is part of this specification issue.

## Demo script

1. Show the four coloured canvas segments, exception paths below the happy path, the parallel post-decision branches, and the labelled external-agent showcase.
2. Submit the complete serious-injury fixture and point out `complaintId`, source checksums, attachment provenance, IXP confidence, and device match.
3. Open the assessment trace to show deterministic clock logic, cited policy-tool calls, similar-case caveat, recommendation, and explicit lack of decision authority.
4. Open `MDR Reportability Review`, compare the source facts and proposed package fields, correct awareness time and device identity, enter rationale/signature reference, and select `Report`.
5. Return to Flow and show clock recomputation plus mock package validation, reporter-preview, legacy eQMS, and quality branches running independently and merging.
6. Open the queue record to show the signed human event, template-tool trajectory, source-linked package, per-branch receipts, final status, and ordered audit events.
7. Point out the disabled external-agent branch, constant input, absent `thread_id`, discarded response, and lack of complaint-variable mappings.
8. Preview the prompt-injection/gateway-outage fixture to show unchanged tool policy, preserved signed decision, no false acknowledgement, and owned recovery.

## Success measures

- **Business proof:** The demo makes deadline compliance, time to signed decision, first-review completeness, recommendation concordance by class, evidence traceability, recovery rate, and authorized-decision coverage measurable without claiming an improvement target. Demo governance requires 100% of final decisions to have reviewer, role, rationale, timestamp, and signature reference.
- **Flow proof:** A viewer can see IXP, deterministic clock logic, two bounded agent roles, API/RPA contrast, the isolated Azure showcase, real business expressions, human authority, purposeful parallelism, merge, and recoverable exceptions.
- **Demo proof:** In under ten minutes, a viewer can verify complaint provenance, evidence-linked reasoning, reviewer correction, downstream consumption of returned fields, package gating, receipts, and technical recovery.
- **Build proof:** Each project validates without warnings, five evaluation cases pass their stated thresholds, resource bindings are recorded, `resources refresh` and dry-run pack succeed, and an immutable package can follow changed-solution CI into Playground after merge.

## Reference mapping

| Reference requirement | Domain-specific implementation | Evidence or gap |
| --- | --- | --- |
| Use case and hero moment | Qualified vigilance specialist corrects evidence and signs the consequential US MDR decision. | Fully specified; manufacturer roles/procedure remain to validate. |
| 3-4 segment topology and canvas rules | Four coloured sticky notes: Capture the safety complaint; Build the reportability evidence; Make the regulated decision; Prepare, follow up, and record. Happy path is left-to-right, exceptions below, parallel work symmetric, and merge explicit. | Fully specified; canvas implementation remains. |
| Trigger | Manual synthetic packet with `ComplaintMdrInput`, deployment under approved parent `JD_Demos/demos`, and `complaintId` idempotency. | Contract specified; deployment parent approved. |
| IXP/document intelligence | `MedtechComplaintExtraction` extracts device/event fields with page evidence; low confidence/conflict forces intake review. | Contract specified; model, runtime, and threshold calibration remain to provision. |
| API workflow and RPA on the intended path | `device-complaint-gateway` validates/resolves/prepares mock package; `legacy-eqms-complaint` reads and records UI-only case state. | Contracts specified; projects and mocks remain to build; production API gap requires validation. |
| Inline agent with a wired tool | `MDR Evidence Assessor` calls approved policy and similar-case tools and returns branch-relevant evidence/uncertainty. | Contract specified; inline project/tools remain to build. |
| Coded agent with visible value-add | `Reporter Follow-up Writer` retrieves an approved template through MCP and creates a bounded preview. | Contract specified; coded agent, MCP server, and evaluator remain to build. |
| Shared external-agent showcase | Verified Azure node/connection on disabled-by-default, constant-input, discarded-output, fail-open branch. | Node/connection tenant-ready; selected agent and Flow binding remain to validate. |
| Real business decision and safe exception | Confidence/device confirmation, expedited flag, insufficient-information state, human outcome, and receipt expressions drive named paths. | Exact expressions specified; implementation/evaluation remain. |
| Human decision and returned outcome data | Coded action app returns decision, corrected fields, rationale, reviewer/role/time, and signature reference; Flow validates and consumes them. | Contract specified; authority, electronic signature, and task resource remain human/provisioning gaps. |
| Purposeful parallelism and merge | Package validation, reporter preview, eQMS write-back, and quality work run independently then merge before reconciliation. | Fully specified; route-required receipt matrix remains to encode. |
| MCP server/tool | Template-only server has least-privilege input, no send capability, and required trajectory evaluation. | Contract specified; server ownership/content approval remain. |
| Data model | Typed input/output, queue canonical demo record, correlation key, eQMS write-back, read-back receipts, audit events, and synthetic/minimized data. | Fully specified for demo; production source of truth/retention remain human decisions. |
| Evaluation set and evaluator | Five cases plus route, human-authority, clock, trajectory, grounding, isolation, and reconciliation checks. | Initial thresholds specified; evaluator resource IDs/model remain to select and pin. |
| Process-app variant | Not selected; Orchestrator queue is the canonical demo record and a production eQMS is the intended controlled record. | Later cross-domain decision may change this design. |
| Solution boundary and delivery contract | One `medtech-complaint-mdr` solution, one `.uipx`, nested Flow layout, resource refresh, immutable version, changed-solution CI, Playground target. | Fully specified; folder/resource provisioning and package validation remain. |

## Open human decisions

These decisions refine implementation or production substitution but do not block the synthetic, mock-backed build.

| Decision | Owner | Resolution path |
| --- | --- | --- |
| Confirm US-only scope and how other jurisdictions route. | Regulatory affairs and demo portfolio owner | Approve US-only fixtures and `manual_policy_review`, or provide approved jurisdiction rules before expanding inputs. |
| Define manufacturer awareness and correction semantics. | Vigilance process owner and regulatory counsel | Approve which event starts the clock, acceptable correction reasons, and timer audit behavior. |
| Confirm decision, signature, package, and escalation authority. | Regulatory/vigilance owner | Name roles, segregation rules, signature standard, duty roster, and production SLA. |
| Select the controlled system of record and representative integrations. | Quality-system owner and enterprise architect | Choose eQMS, service CRM, PLM/ERP, repository, and gateway; confirm authoritative fields and APIs. |
| Validate the eQMS API gap and RPA responsibility. | eQMS owner and enterprise architect | Retain RPA only if no governed API satisfies the visible read/write responsibility; otherwise select a credible UI-only step or approve a reference deviation. |
| Approve policy/context content and lifecycle. | Quality and Regulatory content owners | Supply versioned synthetic SOP, approved interpretations, risk-file boundaries, effective dates, and retirement controls. |
| Approve data classification and model residency. | Privacy, security, legal, and data-governance owners | Classify patient/reporter/product fields, approve minimization/redaction, retention, regions, and non-synthetic model access. |
| Calibrate extraction and assessment thresholds. | Vigilance SMEs and IXP/AI owners | Evaluate representative sanitized documents and expert-labelled cases before changing demo thresholds into policy. |
| Decide whether prior cases may inform recommendations. | Quality, privacy, and regulatory owners | Approve searchable fields, precedent caveats, superseded-decision handling, and access controls. |
| Select a regulator test boundary. | Regulatory operations and security owners | Approve a validated non-production gateway or retain the schema validator/mock acknowledgement only. |
| Set manufacturer baselines and target measures. | Vigilance operations owner | Supply observed completeness, timing, concordance, rework, recovery, and on-time submission baselines before benefit claims. |
| Decide whether Medtech is a Data Fabric/process-app variant. | Demo portfolio owner | Resolve in the later cross-domain selection issue; this spec currently marks it not selected. |

## Implementation tasks

1. Scaffold `medtech-complaint-mdr` with exactly one `.uipx` and the nested `complaint-mdr-flow/complaint-mdr-flow.flow` project.
2. Build the five-case synthetic packet set, typed schemas, queue contract, versioned SOP/template fixtures, mock device master, mock regulator gateway, and local eQMS UI.
3. Implement and validate `device-complaint-gateway`, `MedtechComplaintExtraction`, deterministic clock/idempotency scripts, and `legacy-eqms-complaint` with typed errors and receipts.
4. Build the inline assessor, approved policy/similar-case tools, structured output, source/version guardrails, and grounding/trajectory evaluators.
5. Build the coded follow-up writer, template-only MCP tool, output safety checks, and required tool-call evaluator.
6. Build and deploy `MDR Reportability Review`, then wire completion handle, returned field IDs, role validation, corrected clock/device values, named outcomes, and timeout route.
7. Author the four-segment Flow, intake exceptions, urgent/insufficient-information expressions, non-material Azure showcase, outcome routes, parallel post-decision work, merge, and receipt reconciliation.
8. Validate the Azure binding and prove that branch off, on, timeout, error, and unexpected output cannot change any business value or final status.
9. Run all project validations and the five-case evaluation set; resolve every warning, failed threshold, prohibited claim, unauthorized route, and receipt mismatch.
10. Refresh solution resources, restore, dry-run pack, and register immutable changed-solution deployment configuration for approved parent `JD_Demos/demos`.

## Quality rubric

| Dimension | Score (0-3) | Evidence and remaining gap | Owner and resolution path |
| --- | ---: | --- | --- |
| Enterprise credibility | 2 | Consequential regulated decision, personas, typed data, clocks, evidence controls, and measures are explicit; manufacturer procedure, authority, systems, and baselines remain unverified. | Regulatory/Quality and system owners validate during discovery. |
| Flow differentiation | 3 | Four segments visibly coordinate IXP, deterministic logic, inline/coded/external agents, API, RPA, human decisions, parallel work, merge, and safe recovery. | Flow implementer preserves topology and validates real expressions. |
| Demo clarity | 3 | Signed-report hero journey, missing-identity and prompt-injection/gateway exceptions, observable receipts, and a timed script make the story verifiable. | Demo owner builds representative fixtures and rehearses after deployment. |
| Build feasibility | 2 | Inputs, outputs, mocks, fallbacks, solution boundary, evaluation, authenticated target, and verified Azure connection are recorded; most resources remain unprovisioned. | Tenant administrator and implementers provision resources, close owned decisions, and record bindings. |
| **Total** | **10/12** | **Ready for implementation planning as a synthetic demo; not ready for production policy, non-synthetic data, electronic signature, or regulator submission.** | **Start with solution/fixtures, then close owned human decisions before replacing mocks.** |
