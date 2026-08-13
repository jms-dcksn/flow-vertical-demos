# Healthcare payer SNF prior-authorization demo specification

## Use case and narrative

- **Domain and solution:** `healthcare-payer/snf-prior-authorization/snf-prior-authorization-solution/`, with globally unique solution and package name `healthcare-payer-snf-prior-authorization`.
- **Enterprise use case:** A utilization-management nurse reviews a synthetic expedited request for post-acute skilled-nursing-facility admission. Flow correlates the submitted clinical packet, member and provider facts, applicable coverage evidence, extraction confidence, and cited agent evidence map. The consequential decision is whether to approve, request specific information, escalate, modify, or deny. The hero moment is a clinician comparing source-linked clinical facts and policy passages in a coded action app, correcting the proposed disposition, and watching those returned fields drive mock authorization, notice, and audit work.
- **Why this use case:** It is the top-ranked candidate in the [healthcare-payer research](research.md), scoring 14/15. It combines recent OIG evidence about initial SNF authorization quality, explicit CMS decision-time and denial-reason requirements, document-heavy evidence, bounded agent reasoning, human clinical authority, API and UI-system work, and an auditable outcome. No Surprises Act IDR tied the score, but SNF authorization provides a clearer clinical-evidence and appeal-prevention hero moment for a broad payer audience.
- **Audience journey:** The demonstrator submits a synthetic packet, follows four named canvas segments, opens an evidence-rich nurse review, refers an adverse candidate to a medical director, and then shows independent mock write-back and correspondence work merge into an audit-ready result. Flow is the right surface because the case is long-running, has multiple accountable actors and safe exception routes, and must preserve evidence, state, timing, and human outcomes across product boundaries.
- **Outcome and value:** The case reaches the correct safe state within the configured demo SLA, every material rationale cites supplied evidence, no adverse or reduced decision bypasses a medical director, and successful completion requires reconciled mock receipts. Pilot value is measured through timeliness, evidence completeness, reviewer edits, notice quality, and appeal-overturn signals; the demo makes no improvement or ROI promise without a payer baseline.
- **Out of scope:** Real member data; production medical-necessity determination; autonomous denial or reduction; live provider, EHR, claims, utilization-management, or correspondence connectivity; certified FHIR Prior Authorization API conformance; production policy interpretation; real notification; Data Fabric/process-app scope; and quantified benefit without a design-partner baseline.

## Personas

| Persona | Role in the demo | Authority boundary |
| --- | --- | --- |
| Utilization-management nurse | Reviews complete evidence, approves eligible care, requests information, or refers the case. | Owns `Approve`, `RequestInformation`, `ReferMedicalDirector`, and `EscalateOperations`; cannot issue a denial or reduced authorization. |
| Medical director | Resolves adverse, reduced, conflicting, or clinically ambiguous cases. | Owns `Approve`, `Modify`, `Deny`, `RequestInformation`, and `Escalate`; a specific cited rationale is mandatory for `Modify` or `Deny`. |
| UM operations coordinator | Owns invalid intake, overdue work, and technical recovery queues. | Reassigns and reconciles work; cannot substitute for clinical authority. |
| Provider authorization staff | Supplies requested information through a simulated channel. | May add evidence but cannot alter policy, reviewer decisions, or audit history. |
| Demonstrator | Submits fixtures and narrates extraction, evidence, review, action, and recovery. | Uses synthetic data and mock systems only. |

## Trigger and case contract

The initial build uses a manual trigger for repeatability. A non-production API trigger can replace it after an interoperability surface and standard version are approved. `correlationId` is the idempotency key; replay returns the existing case and appends a replay audit event without repeating a completed write.

| Item | Specification |
| --- | --- |
| Trigger | Manual Flow trigger accepting `SnfAuthorizationInput`. The `prior-auth-intake-gateway` API workflow validates and normalizes the request before clinical processing. |
| Canonical record | Orchestrator queue `HealthcarePayerSnfAuthorizations`, unique reference `correlationId`. Specific content stores the synthetic request, evidence, current state, and SLA timestamps; output data stores the final determination and receipts. The queue is not provisioned and uses a local fixture during build. |
| Required inputs | Request identifiers and timing, synthetic member/coverage and provider/facility facts, requested SNF dates and days, a packet manifest, versioned policy context, and demo controls. |
| Sensitive data | Synthetic health and identity data only. Member and provider identifiers are masked in tasks, traces, fixtures, and evaluation outputs. No PHI is sent to the external-agent showcase. |
| Outputs | Structured determination, cited reason, missing-information request when applicable, reviewer outcome, mock authorization and notice receipts, audit artifact, exception state, and final status. |

### Input schema

| Field | Type | Required | Contract |
| --- | --- | --- | --- |
| `requestId` | string | yes | Synthetic prior-authorization request identifier. |
| `correlationId` | UUID string | yes | Idempotency value shared by Flow, queue, tasks, receipts, and traces. |
| `receivedAt` | ISO 8601 string | yes | Starts case aging and deadline calculation. |
| `urgency` | enum | yes | `expedited` or `standard`; fixtures use expedited for the hero path. |
| `requestedService` | enum | yes | Fixed to `snf_admission` in the initial demo. |
| `requestedStartDate` | ISO date string | yes | Proposed admission date. |
| `requestedDays` | integer | yes | Positive requested authorization duration. |
| `memberCoverage` | object | yes | Synthetic member ID, product, effective dates, transition-of-care flag, and authorization-history token. |
| `requestingProvider` | object | yes | Synthetic provider and hospital IDs plus contact endpoint. |
| `proposedFacility` | object | yes | Synthetic SNF ID, network state, and contact endpoint. |
| `packet` | object | yes | File manifest containing discharge summary, therapy evaluation, medication list, functional status, and clinical notes. |
| `policyContext` | object | yes | Corpus version, effective date, applicable source IDs, and provenance keys. |
| `showExternalAgentShowcase` | boolean | yes | Defaults to `false`; enables only the non-material Azure branch. |

### Output schema

| Field | Type | Contract |
| --- | --- | --- |
| `status` | enum | `approved`, `information_requested`, `modified`, `denied`, `escalated`, or `technical_exception`. |
| `authorizedDays` | integer or null | Populated only for `approved` or `modified`; copied from the validated human outcome. |
| `decisionReasonCodes` | string array | Deterministic allowlisted codes consistent with the human outcome. |
| `specificReason` | string | Required for `information_requested`, `modified`, or `denied`; cites supplied clinical and policy evidence. |
| `evidenceReferences` | string array | Packet page IDs and versioned policy passage IDs supporting the outcome. |
| `missingInformation` | object array | Document or field, reason, requested owner, and response due time. |
| `review` | object | Reviewer role and ID, named outcome, edited days, rationale, escalation reference, and timestamps. |
| `sla` | object | Due time, warning time, state, and elapsed duration. |
| `writeBackReceipts` | object array | Mock system, operation, transaction ID, read-back state, and timestamp. |
| `communicationArtifacts` | object array | Notice type, template/version, draft URI, validation state, and mock archive receipt. |
| `exceptions` | object array | Typed code, owning role, recovery condition, and current state. |
| `auditEvents` | object array | Actor, action, evidence IDs, input/output hashes, policy/model/prompt/tool versions, and timestamp. |
| `auditArtifactUrl` | string | Mock storage URI for the reconciled case summary. |

## Flow topology

Use four blue sticky notes with the titles below. The happy path runs left to right. Validation, low-confidence, missing-information, clinical-escalation, timeout, and dependency routes sit below their originating segment. The two completion branches are visually symmetric and merge before final reconciliation. A labelled `External agent showcase` branch sits below segment 2 and never receives authorization-case data.

| Reference segment | Domain-specific canvas title | Actors and business output | Branch or merge evidence |
| --- | --- | --- | --- |
| Receive and understand | **1. Receive and extract the request** | Manual trigger invokes `prior-auth-intake-gateway`; IXP extracts the synthetic clinical packet; the queue item is created or read. Output: normalized `SnfAuthorizationCase` with packet completeness and extraction confidence. | Invalid schema or duplicate completed request follows the intake exception route. `extractionConfidence < 0.90 || missingRequiredDocuments.length > 0` forces information/review handling. The threshold is demo configuration, not clinical policy. |
| Assess and enrich | **2. Build the clinical evidence map** | API workflow returns mock coverage, provider, and network facts. Inline `SNF Evidence Mapper` uses read-only packet and policy tools to produce cited facts, conflicts, completeness, and an advisory disposition. The separate Azure showcase branch receives constants only. | `showExternalAgentShowcase === true` enters the non-material branch and rejoins with `SnfAuthorizationCase` unchanged. Core routing uses `policyConflict === true || evidenceConflicts.length > 0 || adverseCandidate === true` for mandatory medical-director referral; all other cases continue to nurse review. |
| Decide and review | **3. Make the accountable determination** | `SNF Authorization Review` coded action app presents the source-linked evidence to a UM nurse. Any adverse, reduced, or unresolved case opens a medical-director task. Output: validated named outcome, rationale, authorized days, and reviewer identity. | Nurse outcomes are `Approve`, `RequestInformation`, `ReferMedicalDirector`, or `EscalateOperations`. Medical-director outcomes are `Approve`, `Modify`, `Deny`, `RequestInformation`, or `Escalate`. Only the medical-director task can return `Modify` or `Deny`; timeout escalates before the configured decision deadline. |
| Act and communicate | **4. Record and communicate the outcome** | In parallel, `legacy-utilization-management` RPA records and reads back the human outcome while coded `Authorization Notice Writer` retrieves an approved template and produces a grounded draft that `prior-auth-intake-gateway` validates and mock-archives. The branches merge and the queue record is finalized. | `requiredReceipts.every(receipt => receipt.state === "succeeded")` permits final success. Missing, failed, or ambiguous write/read-back sets `technical_exception`, keeps the case open, and creates an owned recovery item. |

## Agentic reasoning and tool use

| Actor | Visible responsibility | Structured input and output | Tools, context, and guardrails | Readiness/fallback |
| --- | --- | --- | --- | --- |
| Inline low-code agent: `SNF Evidence Mapper` | Organize packet and policy evidence into a cited map for human review. | Input: extracted packet fields, source page IDs, mock benefit facts, and `policyContext`. Output: `clinicalFacts`, `policyFindings`, `evidenceConflicts`, `missingRequiredDocuments`, `adverseCandidate`, `advisoryDisposition`, and `confidence`. | Read-only `get_case_document(documentId)` and `search_snf_coverage_policy(query, policyVersion)` tools. Every material claim requires a packet page or policy passage ID. No open web, write, member-search, authorization, or reviewer-outcome tool is allowed; retrieved content is untrusted data, not instruction. | No exact tenant resource resolved by name on August 12, 2026. Build an inline agent with versioned local packet and policy fixtures. Tool failure returns `insufficient_evidence` and forces human review. |
| Coded agent: `Authorization Notice Writer` | Draft a specific, plain-language notice only after a validated human outcome. | Input: request ID, outcome, authorized days, reason codes, specific rationale, evidence references, recipient type, and template ID. Output: subject, body, `templateId`, `templateVersion`, citations, and safety findings. | LangGraph agent calls `get_approved_authorization_template(templateId, noticeType)` through a least-privilege UiPath MCP server, then checks outcome consistency, required reason text, unsupported claims, and prohibited sensitive fields. It has no send or decision tool. | No exact tenant resource resolved by name. Use a local approved-template fixture and mock MCP tool during build. Tool or safety-check failure creates a manual-draft task without changing the clinical outcome. |
| External agent showcase: `Azure AI Foundry connectivity` | Display external-agent connectivity without performing healthcare work. | Connection-selected `agent_id` and constant message `UiPath Flow external-agent connectivity showcase for healthcare payer`; omit `thread_id`. Discard the response and do not map it into `SnfAuthorizationCase`. | Node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` with connection `0107247a-0197-42c9-b957-05d1b722b111`. Do not pass request, member, provider, facility, clinical, policy, reviewer, decision, notice, or receipt data. | Reverified August 12, 2026 with UiPath CLI 1.199.0 in `uipathlabs / Playground`: node tenant-available; connection enabled and default in `JD_Demos/demos`. The branch is disabled by default; a short timeout or error records only transient showcase status and rejoins the unchanged core route. |

## Data and resources

| Resource or data dependency | Purpose and contract | Owner/folder/readiness | Security and failure handling |
| --- | --- | --- | --- |
| IXP project: `HealthcarePayerSnfPacketExtraction` | Extract requested service/dates/days, diagnoses, functional status, therapy findings, prior level of function, medications, discharge disposition, and document/page references from synthetic PDFs. | New project/model; deploy beneath approved parent `JD_Demos/demos`; no model binding verified. | Synthetic files only. Confidence below `0.90`, a missing required document, or conflicting extracted values forces human review; versioned fixture JSON preserves the output contract offline. |
| API workflow: `prior-auth-intake-gateway` | Operations `normalize_request`, `read_coverage_context`, `request_information`, and `validate_and_archive_notice` return typed synthetic results and receipts. Flow invokes it in segments 1, 2, and 4. | New sibling project; production interoperability standard, endpoints, and connections are unverified. | Local mock endpoints are the default. Calls use `correlationId`, at most two retries for clearly transient reads, no automatic retry after an ambiguous write, and typed failure results that never fabricate success. |
| RPA: `legacy-utilization-management` | Operation `record_determination` enters only the validated human outcome in a local legacy UM desktop fixture and returns a transaction ID plus read-back values. | New sibling project; the local fixture deliberately has no API. A real application/API gap and unattended runtime are unverified. | Synthetic data only. The activity compares read-back to approved values. Selector failure or ambiguous write creates a screenshot/error reference and routes to reconciliation without automatic replay. |
| Queue: `HealthcarePayerSnfAuthorizations` | Canonical demo case, idempotency, state transitions, final output, and ordered audit events. | New Orchestrator queue beneath the deployment folder; not provisioned. Use a local fixture until provisioned. | Least-privilege robot access, masked identifiers, implementation-time retention selection, and replay-safe unique references. |
| Context index: `HealthcarePayerSnfCoveragePolicy` | Versioned public CMS excerpts plus synthetic plan policy used by `search_snf_coverage_policy`. | New context index; policy owner and precedence rules require human approval. | Each passage carries source ID, version, effective date, and provenance. Retrieval failure or conflicting sources blocks agent reliance and forces review. |
| Azure AI Foundry connection | Supplies the required non-material external-agent node. | Shared connection `0107247a-0197-42c9-b957-05d1b722b111`; enabled/default in Playground folder path `JD_Demos/demos` on August 12, 2026. | Constant non-sensitive message only, no `thread_id`, discarded output, no business-variable mapping, and fail-open continuation to the same core merge. Validate the Flow-specific binding during implementation. |
| MCP template server | Exposes only `get_approved_authorization_template(templateId, noticeType)`. | New demo MCP server or local mock; correspondence and compliance owners approve templates. | No send, member lookup, or free-form policy tool. Tool inputs omit raw clinical text and identifiers; failure creates a manual-draft task. |
| Coded action app: `SNF Authorization Review` | Shows packet evidence, policy passages and versions, conflicts, missing items, SLA clock, and the advisory evidence map; returns named outcomes and edits. | Deployed independently and referenced by Flow; no exact tenant resource resolved by name. | Role restricted, synthetic data, source facts read-only, rationale required, outcome allowlist enforced, and adverse outcomes limited to the medical-director task. |

### Solution boundary and layout

```text
healthcare-payer/snf-prior-authorization/
├── snf-prior-authorization-solution/
│   ├── healthcare-payer-snf-prior-authorization.uipx
│   ├── snf-prior-authorization-flow/
│   │   └── snf-prior-authorization-flow.flow
│   ├── prior-auth-intake-gateway/
│   ├── legacy-utilization-management/
│   └── authorization-notice-writer/
└── snf-authorization-review/
```

The solution has exactly one `.uipx` manifest and is independently deployable. The coded action app remains independently deployed if required by the platform, with its versioned contract pinned by solution configuration. Before packaging, run `uip solution resources refresh`, restore dependencies, and dry-run pack. Pull requests validate only the changed solution; publish and deploy occur only after merge to `main` through `playground-deploy`, with an immutable package version under the approved Playground parent folder `JD_Demos/demos`.

## Human decisions

- **Nurse review:** The UM nurse sees request facts, packet-page excerpts, extraction confidence, missing documents, mock benefit/network facts, policy passages and versions, conflicts, the agent's cited evidence map, and the SLA clock. Source facts and policy text are read-only.
- **Nurse edits and outcomes:** Editable values are authorized days, specific information requested, rationale, and referral note. Outcomes are `Approve`, `RequestInformation`, `ReferMedicalDirector`, and `EscalateOperations`. The nurse cannot return `Modify` or `Deny`.
- **Medical-director review:** The medical director receives the nurse record plus adverse/conflict evidence. Editable values are authorized days, reason code, specific rationale, requested information, and escalation detail. Outcomes are `Approve`, `Modify`, `Deny`, `RequestInformation`, and `Escalate`.
- **Timeout and resumption:** The demo warning and timeout are configuration values chosen before build. Warning occurs before, never at, the applicable decision deadline. Timeout assigns the case to UM operations or medical-director escalation and preserves the open case. Flow resumes from the task completion handle, validates outcome and returned field IDs, persists the decision, and routes only from returned values.
- **Downstream authority:** Only a validated nurse approval or medical-director outcome enables write-back. `RequestInformation` creates a specific mock request and sets `information_requested`; `Modify` or `Deny` requires a medical-director ID, reason code, rationale, and citations; `Escalate` produces no determination write.

## Controls and safety

| Control | Design decision | Evidence in the Flow or demo |
| --- | --- | --- |
| Clinical authority | Agents organize evidence and draft notices only. A nurse owns approval/information requests; a medical director owns every denial or reduction. | Outcome-specific human task handles, returned reviewer role, and a guard expression that rejects `modified` or `denied` without medical-director evidence. |
| Routing safety | Deterministic schema, completeness, confidence, conflict, and adverse-candidate checks precede agent output and human routing. | Expressions use `missingRequiredDocuments`, `extractionConfidence`, `policyConflict`, `evidenceConflicts`, and `adverseCandidate`; exceptions remain below the happy path. |
| Evidence grounding | Every material clinical or policy statement carries a packet page or versioned policy passage ID. Uncited output is ignored. | Structured schemas, evidence validator, clickable task citations, and source/version hashes in audit events. |
| Access and data | Synthetic data, least privilege, masked identifiers, approved policy fixtures, role-restricted tasks, and mock write targets. | Fixture manifest, connection bindings, task roles, redaction assertions, and no case mapping to the external branch. |
| Agent boundaries | Inline tools are read-only and scoped to one case/corpus; the coded agent has a template retrieval tool but no decision or send tool; the external agent receives constants only. | Tool allowlists, prompt prohibitions, structured outputs, trajectory evaluators, and absent write mappings. |
| Resilience | Use idempotency, typed failures, bounded transient read retries, no retry after ambiguous writes, early SLA escalation, and receipt reconciliation. | Queue reference, implicit error-port routes, timeout path, read-back comparison, merge, and owned recovery item. |

## Error paths and recovery

| Failure | Safe route | Recovery owner and condition |
| --- | --- | --- |
| Invalid schema, unsupported service, or completed replay | Do not invoke clinical actors or write systems; create an intake exception or return the existing result. | UM operations corrects intake or accepts the replay result using the same `correlationId`. |
| Missing document or IXP confidence below `0.90` | Preserve extracted evidence and enter nurse review with `RequestInformation` available; no adverse write. | Nurse specifies the missing item; provider staff submits a new version under the same case. |
| Policy retrieval unavailable, conflicting source, or uncited agent output | Set `insufficient_evidence`, ignore the advisory disposition, and require human review/escalation. | Policy/platform owner restores the corpus; clinician may proceed only from available source-linked evidence. |
| Human task warning or timeout | Keep the case open and escalate before the configured decision deadline. | UM operations or medical-director queue reassigns and completes the named task. |
| API or RPA read failure | Retry a clearly transient read at most twice, then set `technical_exception`. | Platform/RPA owner restores the dependency and explicitly resumes the failed activity. |
| Ambiguous determination write or read-back mismatch | Do not retry automatically or create a success receipt; keep the case open. | UM operations reconciles target state and authorizes resume or manual completion. |
| Template retrieval or notice validation failure | Create a manual-draft task; preserve the clinical outcome and successful receipts. | Correspondence/compliance owner supplies approved text before case closure. |
| Azure AI Foundry timeout, error, or unexpected response | Record only transient `externalAgentShowcaseStatus`, discard the response, and rejoin the same core route. | Demo owner may disable the branch; clinical and operations users take no recovery action because case state is unchanged. |

## Observability and evaluation

| Signal or test | What it proves | Expected result or threshold |
| --- | --- | --- |
| Correlation and audit | Intake, extraction, evidence, policy, reviewer edits, determinations, receipts, and recovery can be reconstructed. | Every trace, task, artifact, and receipt contains `correlationId`; ordered audit events include source/model/prompt/tool versions and no unmasked identity. |
| Flow route evaluator | Each fixture reaches the correct safe route, required reviewer, and final status. | 5/5 initial synthetic cases match the expected route, human-role requirement, and status before promotion. |
| Evidence evaluator | Agent and notice claims are supported by supplied packet and policy evidence. | 100% of material claims have valid source IDs; invented facts and unsupported policy claims occur zero times. |
| Clinical-authority evaluator | No adverse or reduced determination bypasses qualified human authority. | Zero `modified` or `denied` outputs without a completed medical-director task, reason code, rationale, and citations. |
| Tool-use/trajectory evaluator | Agents use only the evidence and approved-template tools required by their responsibilities. | Required evidence tools are called in every mapper run; template tool is called in every applicable notice run; unapproved tool calls occur zero times. |
| External showcase isolation | The placeholder external agent cannot change a case, decision, action, or status. | With the flag off, on, or failing, the same fixture produces identical `SnfAuthorizationCase`, route, review requirements, receipts, and final status; only transient showcase status may differ. |
| Receipt reconciliation | Completion never hides a failed or ambiguous system action. | Final success occurs only when every required mock receipt is present, `succeeded`, and read-back matches approved values. |

### Synthetic evaluation set

Dataset name: `healthcare-payer-snf-prior-authorization-v1`.

| Case | Expected route | Expected business output |
| --- | --- | --- |
| Complete policy-supported expedited request | Nurse review with `Approve`. | Approved days come from the nurse task; authorization and notice receipts reconcile; status is `approved`. |
| Missing therapy evaluation | Nurse selects `RequestInformation`. | Specific missing document and due time are recorded; no determination write occurs; status is `information_requested`. |
| Adverse-candidate request | Nurse selects `ReferMedicalDirector`; medical director selects `Deny`. | Specific cited reason, reason code, both reviewer records, and validated notice artifact are present; status is `denied`. |
| Conflicting packet and policy evidence | Mandatory medical-director route with `Escalate`. | No authorization write, conflict references preserved, escalation owner recorded, and status is `escalated`. |
| Legacy UM read-back mismatch | Human approval reaches segment 4, but RPA reconciliation fails. | No false success receipt; case remains open with `technical_exception` and owned recovery data. |

## Demo script

1. Show the four-segment canvas, exception routes below the happy path, and the disabled external-agent showcase branch.
2. Submit the adverse-candidate fixture and point out `correlationId`, expedited due time, packet manifest, and extraction confidence.
3. Open the evidence-map trace to show read-only packet/policy tools, page and passage citations, conflicts, and the advisory disposition.
4. Open `SNF Authorization Review` as the UM nurse, inspect evidence, add a referral note, and select `ReferMedicalDirector`.
5. Open the medical-director task, enter the specific cited rationale and reason code, and select `Deny`.
6. Return to Flow and show the mock legacy write-back and grounded notice/archive branches running independently and merging.
7. Open the queue record to show both human outcomes, template-tool call, read-back receipt, notice validation, final status, and ordered audit events.
8. Point out the Azure node's constant input, omitted `thread_id`, discarded response, and isolation. Preview the read-back-mismatch fixture to show `technical_exception` and owned recovery.

## Success measures

- **Business proof:** The demo exposes decision timeliness, additional-information rate and reasons, adverse rate, appeal-overturn follow-up, evidence completeness, reviewer edit rate, notice specificity, and write-back mismatch as measurable pilot signals without promising improvement.
- **Flow proof:** A viewer can see IXP, deterministic checks, two material agent roles, the non-material Azure AI Foundry showcase, API/RPA contrast, real routing, two levels of human authority, independent work, merge, and safe recovery.
- **Demo proof:** In under ten minutes, a viewer can verify evidence provenance, nurse-to-medical-director escalation, cited consequential determination, downstream use of returned fields, reconciled artifacts, and a technical exception.
- **Build proof:** Every project validates, the five-case evaluation set passes, all resource bindings are recorded, `resources refresh` and dry-run pack succeed, and the immutable package can follow changed-solution CI into Playground under `JD_Demos/demos`.

## Reference mapping

| Reference requirement | Domain-specific implementation | Evidence or gap |
| --- | --- | --- |
| 3-4 segment topology and canvas rules | Four blue sticky notes: Receive and extract the request; Build the clinical evidence map; Make the accountable determination; Record and communicate the outcome. | Fully specified; canvas implementation remains. |
| Trigger and canonical record | Manual `SnfAuthorizationInput` trigger and queue record keyed by `correlationId`. | Contract specified; queue remains to provision. |
| IXP/document intelligence | `HealthcarePayerSnfPacketExtraction` extracts clinical facts and page references; low confidence or missing evidence forces review. | Contract and fixture fallback specified; model/folder remain unverified. |
| API workflow and RPA on the intended path | `prior-auth-intake-gateway` handles normalization, enrichment, information requests, and notice archive; `legacy-utilization-management` performs human-approved UI write/read-back. | Contracts specified; mock projects and real API-gap validation remain. |
| Inline agent with a wired tool | `SNF Evidence Mapper` uses case-document and pinned-policy tools and returns branch-relevant evidence fields. | Contract specified; inline project and tools remain to build. |
| Coded agent with visible value-add | `Authorization Notice Writer` retrieves an approved template and drafts a grounded post-decision notice. | Contract specified; coded agent and template server remain to build. |
| Shared external-agent showcase | Required Azure node/connection uses a constant message, no `thread_id`, discarded output, disabled-by-default flag, short error continuation, and unchanged business state. | Node, connection state, and folder were reverified; Flow binding and connection-selected agent remain to validate. |
| Real business decision and safe exception | Completeness/conflict/adverse expressions plus nurse and medical-director outcome switches; timeouts and dependency failures remain safe. | Fully specified; expressions remain to implement and evaluate. |
| Human decision and returned outcome data | Coded action app returns role-bound outcomes, edited days, reason, rationale, reviewer, and timestamp; Flow consumes them downstream. | Contract specified; payer authority and SLA values remain human decisions. |
| Purposeful parallelism and merge | Legacy UM write/read-back runs independently from grounded notice validation/archive, then both merge before reconciliation. | Fully specified; outcome-specific receipt matrix remains to encode. |
| MCP server/tool | Least-privilege `get_approved_authorization_template` supports the coded writer; trajectory evaluation requires the call. | Contract specified; server or mock remains to build. |
| Evaluation set and evaluator | Five fixtures plus route, evidence, authority, trajectory, isolation, and receipt checks with exact initial thresholds. | Fully specified; fixtures and evaluators remain to build. |
| Process-app variant | Not selected. | Later cross-domain selection may change the canonical record and app design. |
| Solution boundary and delivery contract | One `healthcare-payer-snf-prior-authorization` solution, one `.uipx`, nested Flow layout, resource refresh, immutable version, changed-solution CI, and approved `JD_Demos/demos` deployment parent. | Fully specified; remaining resources require provisioning. |

## Open human decisions

These decisions refine implementation but do not block the synthetic, mock-backed build.

| Decision | Owner | Resolution path |
| --- | --- | --- |
| Confirm the payer product, jurisdiction, and representative SNF policy. | Payer product, clinical-policy, and legal owners | Approve the Medicare Advantage framing or revise policy sources, timing, and fixtures for the selected product. |
| Confirm clinical authority and outcome vocabulary. | UM medical director and operations owner | Approve nurse and medical-director roles, editable fields, referral rules, reason codes, and whether any straight-through approval is permitted; the initial build requires nurse approval. |
| Set deadline warning, task timeout, and after-hours escalation values. | UM operations and compliance owners | Configure values that escalate before the applicable deadline and document coverage ownership. |
| Approve the coverage-policy corpus and precedence rules. | Clinical-policy and compliance owners | Version the public and synthetic sources, approve effective dates and conflict handling, and name the corpus owner. |
| Confirm required packet documents and extraction thresholds. | UM clinical and document-intelligence owners | Review the synthetic taxonomy, labeled fixtures, `0.90` demo threshold, and minimum evidence for each outcome. |
| Validate production API surfaces and the legacy UM API gap. | Enterprise architect and application owners | Retain RPA only if no governed write/read-back API exists; otherwise choose another credible UI-only responsibility or approve a reference deviation. |
| Approve privacy, security, retention, and notice controls. | Privacy, security, records, legal, and accessibility owners | Approve allowed fields, role access, redaction, retention, languages, and templates before non-synthetic testing. |
| Provision tenant resources beneath the approved deployment parent. | UiPath tenant administrator | Use `JD_Demos/demos`, reuse the verified Azure connection, provision remaining resources, and record each binding. |
| Set pilot baselines and target measures. | UM analytics and operations owners | Supply observed timing, information-request, adverse, appeal-overturn, notice, and recovery baselines before making benefit claims. |
| Decide whether healthcare payer is a Data Fabric/process-app variant. | Demo portfolio owner | Resolve in the later cross-domain selection issue; this spec currently marks it not selected. |

## Implementation tasks

1. Scaffold `healthcare-payer-snf-prior-authorization` with the nested Flow project and exactly one `.uipx` manifest.
2. Build the five-case synthetic packet/policy fixture set, canonical queue contract, local API mocks, and legacy UM desktop fixture.
3. Implement and validate `prior-auth-intake-gateway`, the IXP extraction contract, and `legacy-utilization-management` with typed failures and read-back receipts.
4. Build the inline evidence mapper, read-only packet/policy tools, deterministic route logic, and structured-output evaluator.
5. Build the coded notice writer, least-privilege template MCP tool, and trajectory/grounding evaluators.
6. Build and deploy `SNF Authorization Review`, then wire nurse and medical-director task handles, outcomes, edits, timeout, and role validation into Flow.
7. Author the four-segment Flow, exception routes, external Azure showcase, parallel completion branches, merge, reconciliation, and audit updates.
8. Bind and validate Azure connection `0107247a-0197-42c9-b957-05d1b722b111`; prove off, on, and failing showcase runs cannot change case data, routing, human requirements, receipts, or status.
9. Run project validation and the five-case evaluation set; resolve every warning and failed threshold.
10. Refresh solution resources, restore, dry-run pack, and register immutable deployment configuration for `JD_Demos/demos`.

## Quality rubric

| Dimension | Score (0-3) | Evidence and remaining gap | Owner and resolution path |
| --- | ---: | --- | --- |
| Enterprise credibility | 2 | Consequential clinical decision, current public evidence, roles, data contract, controls, and pilot measures are explicit; payer policy, systems, and baselines remain unverified. | Clinical-policy, operations, legal, and system owners validate during discovery. |
| Flow differentiation | 3 | Four segments visibly coordinate IXP, deterministic logic, agents, API, RPA, two-level human authority, parallel work, merge, and safe recovery. | Flow implementer preserves the topology and validates each real expression and error edge. |
| Demo clarity | 3 | The nurse-to-medical-director adverse journey and read-back exception have named, observable proof points and a timed script. | Demo owner rehearses both fixtures after deployment. |
| Build feasibility | 2 | Inputs, outputs, mocks, fallbacks, solution boundary, evaluation, CLI/auth target, approved deployment parent, and verified Azure dependency are recorded; domain resources remain unprovisioned. | Tenant administrator and implementers provision the remaining resources beneath `JD_Demos/demos`. |
| **Total** | **10/12** | **Ready for implementation planning as a synthetic demo; not ready for production clinical use, non-synthetic data, or deployment until owned decisions and bindings are resolved.** | **Start with solution and fixtures, then close human decisions before replacing mocks.** |
