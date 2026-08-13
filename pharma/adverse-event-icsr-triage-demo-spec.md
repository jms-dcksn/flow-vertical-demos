# Pharma adverse-event ICSR triage demo specification

## Use case and narrative

- **Domain and solution:** `pharma/adverse-event-icsr-triage/adverse-event-icsr-triage-solution/`, with globally unique solution and package name `pharma-adverse-event-icsr-triage`.
- **Enterprise use case:** A pharmacovigilance team receives a synthetic post-market human-drug adverse-event report. Flow preserves the source and regulatory clock, extracts the report, checks the four minimum case elements, searches for potential duplicates, proposes evidence-linked seriousness and expectedness assessments, pauses for an authorised safety reviewer, and coordinates approved case write-back, follow-up, and mock ICH E2B(R3) output.
- **Why this use case:** It is the top-ranked candidate in the [Pharma opportunity research](agentic-workflow-opportunities.md), scoring 14/15. It has the strongest combination of document intelligence, bounded agent reasoning, deterministic regulatory controls, duplicate retrieval, human accountability, standardised output, parallel work, and a visible deadline. The public evidence defines the process at field and decision level; the other candidates need more sponsor- or site-specific data before their reasoning can be evaluated credibly.
- **Default demo scope:** A synthetic US post-market human-drug case received by email. The demo rule set covers minimum-case validity and the FDA 15-calendar-day serious-and-unexpected reporting scenario. The rule set is illustrative and versioned; an organisation's pharmacovigilance quality owner must approve it before any non-demo use.
- **Audience journey:** The demonstrator submits a synthetic report, follows four named canvas segments, opens an evidence-rich review task while the clock remains visible, corrects an agent's proposed seriousness criterion or duplicate disposition, and watches the returned decision drive mock safety-system, E2B(R3), and follow-up work. Flow is the right surface because one durable instance coordinates document extraction, deterministic rules, agent and tool calls, RPA, human waits, recoverable exceptions, and parallel completion while retaining per-case visibility.
- **Success outcome:** The case reaches a safe disposition with source-linked fields, versioned rules and context, an explicit authorised decision, reconciled mock receipts, and no real regulator submission or reporter contact.
- **Measurable value:** Pilot measures are intake-to-review-ready time, deadline adherence, minimum-element completeness, targeted follow-up completeness, potential-duplicate confirmation rate, reviewer override rate, validator rework, backlog age, and receipt reconciliation. The demo makes these measures observable but claims no improvement, capacity gain, or ROI without an organisation-specific baseline.
- **Out of scope:** Signal detection, aggregate benefit-risk analysis, clinical-trial SUSAR processing, literature surveillance at scale, real medical coding, production safety-database writes, real reporter contact, regulator transmission, causality automation, Data Fabric/process-app scope, and any claim that the demo is a validated GxP system.

## Personas

| Persona | Role in the demo | Authority boundary |
| --- | --- | --- |
| Pharmacovigilance intake specialist | Confirms source legibility, minimum-element evidence, and reporter contact restrictions. | May correct extracted facts and request information; cannot decide seriousness, expectedness, reportability, duplicate merge, or release. |
| Drug-safety physician or authorised medical reviewer | Owns the consequential case assessment and release decision. | Confirms or corrects seriousness criteria, expectedness/listedness, duplicate disposition, day zero, due date, report type, and release status. |
| Regulatory operations specialist | Reconciles mock E2B(R3) validation and submission-readiness artifacts. | May repair technical format errors after medical approval; cannot change the medical decision without returning the case to review. |
| Pharmacovigilance operations lead | Owns overdue work and technical-exception recovery. | Reassigns or escalates cases but does not silently override required review. |
| Demonstrator | Submits fixtures and narrates evidence, routing, review, and recovery. | Uses synthetic data and mock systems only. |

## Trigger and case contract

The first build uses a manual trigger for repeatable, connection-independent demonstration. It accepts a synthetic email package stored in an approved demo bucket. An Outlook email-received trigger may replace it after a monitored mailbox, connection, folder, and day-zero procedure are approved. `intakeId` is the immutable correlation and idempotency key; a replay reads the existing case and appends a replay audit event rather than creating another case.

| Item | Specification |
| --- | --- |
| Trigger | Manual Flow trigger accepting `SafetyReportInput`; `safety-case-gateway` normalises the package before case creation. |
| Canonical record | Orchestrator queue `PharmaAdverseEventCases`, unique reference `intakeId`. Specific content stores the synthetic source references, extracted case, current state, review data, and audit events; output data stores final disposition and receipts. |
| Required inputs | `intakeId`, `receivedAtUtc`, `sourceChannel`, `sourceMarket`, `sourceDocument`, `reporter`, `patient`, `suspectProducts`, `reactions`, and `caseContext`. |
| Sensitive data | Synthetic health and contact data only. Direct identifiers remain in the protected source fixture and are excluded from agent prompts, showcase traffic, task summaries, traces, and evaluation exports. |
| Outputs | Validity, duplicate assessment, advisory medical assessment, reporting plan, reviewer decision, follow-up plan, mock safety-system receipt, mock E2B(R3) validation artifact, final status, and audit summary. |

### Input schema

| Field | Type | Required | Contract |
| --- | --- | --- | --- |
| `intakeId` | string | yes | Immutable synthetic correlation and idempotency key. |
| `receivedAtUtc` | ISO 8601 string | yes | Preserved source receipt time; proposed day zero remains subject to reviewer confirmation. |
| `sourceChannel` | enum | yes | Initial build uses `email`; contract also permits `web_form`, `call_transcript`, or `literature` for later fixtures. |
| `sourceMarket` | enum | yes | Initial build requires `US`; unsupported values route to manual assessment. |
| `sourceDocument` | object | yes | Bucket URI, filename, media type, SHA-256 hash, and immutable source ID for a synthetic email/PDF package. |
| `reporter` | object | no | Qualification, country, contactability, restrictions, and synthetic contact reference. |
| `patient` | object | no | At least one qualifying descriptor when present; synthetic age, sex, or initials only. |
| `suspectProducts` | array | no | Product, strength, dose, route, dates, indication, and batch when supplied. |
| `reactions` | array | no | Verbatim reaction, onset, outcome, and proposed synthetic terminology. |
| `caseContext` | object | no | Concomitant products, history, tests, pregnancy, medication error, and prior-case references when supplied. |

### Output schema

| Field | Type | Contract |
| --- | --- | --- |
| `validity` | object | Boolean presence of identifiable reporter, identifiable patient, suspect product, and suspected reaction; missing fields, evidence references, and reviewer status. |
| `duplicateAssessment` | object | `none`, `potential`, or `confirmed`; candidate IDs, match reasons, master case, new-information flag, and human confirmation. |
| `medicalAssessment` | object | Advisory seriousness criteria, expectedness/listedness result, synthetic term suggestions, rationale, evidence references, confidence, and reviewer corrections. |
| `reportingPlan` | object | Jurisdiction, report type, calculated and reviewer-confirmed day zero, due date, rule-set version, and `submit_mock`, `follow_up`, `retain_only`, or `escalate`. |
| `followUpPlan` | object | Missing clinically significant information, contact restrictions, approved questions, attempt history, and outcome. |
| `humanDecision` | object | Reviewer identity/role, `ApproveMockRelease`, `CorrectAssessment`, `RequestInformation`, or `Escalate`, reason, changed fields, and timestamp. |
| `regulatoryArtifact` | object | Mock E2B(R3) XML URI, validator status/errors, masking profile, watermark, and release status; it never represents a real submission. |
| `writeBackReceipts` | array | Mock system, operation, correlation ID, status, and timestamp. |
| `finalStatus` | enum | `awaiting_information`, `approved_for_mock_release`, `retained_incomplete`, `escalated`, `completed`, or `technical_exception`. |
| `auditEvents` | array | Actor, transition, evidence references, input/output hashes, rule/model/prompt/tool versions, exception, and timestamp. |

## Flow topology

Use four blue sticky notes with the titles below. The happy path runs left to right; invalid intake, low-confidence extraction, missing criteria, timeout, and dependency failures sit below their originating segment. The regulatory clock remains visible from intake through completion. Independent finalisation branches are visually symmetric and merge before receipt reconciliation. A separately labelled `External agent showcase` branch sits below segment 2 and never receives safety-case data.

| Reference segment | Domain-specific canvas title | Actors and business output | Branch or merge evidence |
| --- | --- | --- | --- |
| Receive and understand | **1. Receive and extract the report** | Manual trigger invokes `safety-case-gateway`; IXP extracts the synthetic email/PDF; queue create-or-read preserves the immutable source and clock. Output: canonical `SafetyCase`. | Invalid package or hash routes to intake repair. `extractionConfidence < 0.90 || extractionConflicts.length > 0` forces extraction review. The threshold is demo configuration, not medical confidence. |
| Assess and enrich | **2. Validate and assess the case** | Deterministic script checks the four minimum elements; `safety-case-gateway` calculates a proposed due date from versioned rules. `legacy-safety-console` RPA performs read-only duplicate retrieval. Inline `Safety Case Assessor` uses approved-label and case-rule tools to propose evidence-linked seriousness and expectedness. | `validity.allMinimumElementsPresent === false` routes to follow-up/retention review. `duplicateAssessment.status === "potential" || medicalAssessment.confidence < 0.85` flags mandatory reviewer attention. The external showcase rejoins with `SafetyCase` unchanged. |
| Decide and review | **3. Review the safety decision** | `Safety Case Medical Review`, a coded action app, shows the source, evidence spans, clock, missing elements, duplicate candidates, label excerpt, proposed assessment, and editable decision fields. | Flow waits for the completion handle and routes on `humanDecision.outcome`: `ApproveMockRelease`, `CorrectAssessment`, `RequestInformation`, or `Escalate`. No autonomous release path exists. Timeout escalates while preserving case state. |
| Act and communicate | **4. Finalise and evidence the case** | After a validated human outcome, API workflow generates/validates a mock E2B(R3) artifact, RPA writes the approved state to the local safety UI, and coded `Safety Follow-up Writer` creates an approved follow-up preview when needed. Branches merge before receipt reconciliation and queue completion. | `requiredReceipts.every(receipt => receipt.status === "succeeded")` permits completion. Missing or ambiguous receipts set `technical_exception`; validator errors return to regulatory operations; no external submission or message send occurs. |

### Intended node sequence

1. Manual trigger receives `SafetyReportInput` and calls the normalisation operation in `safety-case-gateway`.
2. Queue create-or-read enforces `intakeId` idempotency and starts the visible case clock.
3. `PharmaSafetyReportExtraction` IXP extracts field values, evidence spans, and confidence; until the model is published, a contract-faithful mock supplies fixture output.
4. A deterministic script verifies the four minimum elements; `safety-case-gateway` calculates the illustrative US due date from versioned rules.
5. `legacy-safety-console` searches the local mock UI for duplicate candidates and returns typed match evidence.
6. Inline `Safety Case Assessor` calls only read-only label and rule tools and returns structured recommendations.
7. A decision sends incomplete, potential-duplicate, low-confidence, and supported complete cases into the appropriate view of the same coded action app.
8. The completed human task returns named outcome and corrected fields; a switch controls all downstream routes.
9. Approved or information-request routes fan out to the applicable mock E2B, safety-system, and preview branches.
10. A merge waits for required branches, then a deterministic reconciliation script sets the final status and closes or retains the queue item.

## Agentic reasoning and tool use

| Actor | Visible responsibility | Structured input and output | Tools, context, and guardrails | Readiness/fallback |
| --- | --- | --- | --- | --- |
| Inline low-code agent: `Safety Case Assessor` | Propose evidence-linked seriousness criteria and expectedness/listedness for reviewer consideration. | Input: redacted extracted facts, validity, duplicate candidates, product ID, label/rule versions. Output: `seriousnessCriteria`, `expectedness`, `confidence`, `rationale`, `evidenceRefs`, `missingEvidence`. | Read-only `get_approved_product_label(productId, version)` and `get_safety_case_rule(ruleId, version)` tools. Every proposed field needs an evidence reference; unknown or conflicting context returns `insufficient_evidence`. It cannot decide validity, day zero, reportability, duplicate merge, release, or write data. | Not provisioned. Use versioned synthetic label/rule fixtures behind matching tool contracts. Missing context ignores the recommendation and forces human review. |
| Coded agent: `Safety Follow-up Writer` | Draft targeted reporter questions or an internal case-completion note after the reviewer outcome. | Input: outcome, approved missing fields, non-identifying facts, contact restrictions, and template ID. Output: subject, body, `templateId`, `templateVersion`, cited fields, and safety findings. | LangGraph agent calls `get_approved_safety_followup_template` through a least-privilege UiPath MCP server, then self-checks for unapproved medical advice, unsupported claims, direct identifiers, and prohibited contact. It has no send tool. | Not provisioned. Use an approved static-template fixture if the MCP tool is unavailable; failure creates a manual-draft task without fabricating communication. |
| External agent showcase: `Azure AI Foundry connectivity` | Display external-agent connectivity without performing pharmacovigilance work. | Connection-selected `agent_id` and constant message `UiPath Flow external-agent connectivity showcase for pharma`; omit `thread_id`. Discard the response. | Node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread`, connection `0107247a-0197-42c9-b957-05d1b722b111`. Do not pass intake, source, reporter, patient, product, reaction, assessment, reviewer, clock, artifact, or receipt data. | Verified August 12, 2026 in `uipathlabs / Playground`: node tenant-available and connection default, enabled, and pinged active in folder `demos`. The branch is disabled by default; timeout/error records transient status and rejoins the unchanged core route. |

## Data and resources

| Resource or data dependency | Purpose and contract | Owner/folder/readiness | Security and failure handling |
| --- | --- | --- | --- |
| IXP project: `PharmaSafetyReportExtraction` | Extract reporter, patient, suspect product, reaction, dates, dose, narrative, and prior-case IDs from a synthetic email/PDF with field confidence and source spans. | New model under the solution's child folder beneath `JD_Demos/demos`; no Pharma-specific published IXP node was found in the August 12 targeted registry search. | Synthetic documents only. Until publication, use a `core.logic.mock` fixture with the same output contract. Low confidence, missing evidence, or conflicts force review. |
| API workflow: `safety-case-gateway` | Operations `normalize_intake`, `calculate_reporting_plan`, `build_mock_e2b`, and `validate_mock_e2b`; Flow invokes it in segments 1, 2, and 4. | New sibling project using local versioned rules and mock endpoints; no matching published ICSR/safety-case resource was found. | Typed schema and validator errors; idempotent calls; at most two retries for transient reads; no submission endpoint or credential exists. |
| RPA: `legacy-safety-console` | Operations `search_duplicates` and `record_approved_case` against a local mock safety UI with no API. Returns candidates or an unambiguous write receipt. | New sibling project; unattended runtime not provisioned. The UI-only gap is a demo premise that the safety-system owner must validate before any real-system design. | Read uses synthetic data. Ambiguous writes are never retried or treated as success; screenshot/error reference routes to reconciliation. |
| Queue: `PharmaAdverseEventCases` | Canonical demo case, idempotency, state transitions, human decision, receipts, and ordered audit events. | New Orchestrator queue in the deployed solution folder beneath `JD_Demos/demos`; not provisioned. | Least-privilege identities, synthetic data, masked logs, approved retention, and replay-safe unique reference. |
| Synthetic product and rule corpus | Versioned label sections, seriousness criteria, minimum-case rules, and illustrative US reporting clock used by tools and deterministic logic. | Repository fixtures approved by the demo's pharmacovigilance quality owner; not yet authored. | Clearly labelled synthetic content; no licensed MedDRA content; missing/stale version forces review. |
| Outlook connector | Optional later trigger and test-mailbox follow-up transport. Connector activities and an enabled non-default connection are visible in Playground folder `demos`; no connection has been selected or pinged for this Flow. | Mailbox owner and tenant administrator must select the connection and approve the monitored folder. Manual trigger and preview-only follow-up are the initial defaults. | Do not bind another owner's connection silently. Preserve original receipt metadata, redact logs, and keep send disabled until approved. |
| Azure AI Foundry connection | Supplies the non-material external-agent showcase with a constant message and discarded response. | Shared connection `0107247a-0197-42c9-b957-05d1b722b111`; default, enabled, and pinged active in Playground folder `demos`. | No case or sensitive data, no business-variable mappings, short timeout, fail-open continuation, and Flow-specific binding validation during implementation. |
| MCP template server | Exposes only `get_approved_safety_followup_template(templateId, purpose)`. | New demo MCP server or local mock; pharmacovigilance communications owner approves templates. | No patient lookup or send capability. Inputs contain no direct identifiers; failure creates a manual-draft task. |
| Coded action app: `Safety Case Medical Review` | Evidence-linked review UI with immutable source facts, editable assessment fields, required rationale, named outcomes, and clock. | Deployed independently and referenced by Flow; not provisioned. | Role-restricted access, field-level change log, server-side outcome validation, no submission credentials, and safe timeout escalation. |

### Solution boundary and layout

```text
pharma/adverse-event-icsr-triage/
├── adverse-event-icsr-triage-solution/
│   ├── pharma-adverse-event-icsr-triage.uipx
│   ├── adverse-event-icsr-triage-flow/
│   │   └── adverse-event-icsr-triage-flow.flow
│   ├── safety-case-gateway/
│   ├── legacy-safety-console/
│   └── safety-follow-up-writer/
└── safety-case-medical-review/
```

The solution has exactly one `.uipx` manifest and is an independent deployment boundary. The coded action app remains independently deployed if required by the platform, with its contract/version pinned by solution configuration. Before packaging, run `uip solution resources refresh`, restore dependencies, validate every project and Flow warning, and dry-run pack. Pull requests validate only the changed solution. Publish and deploy occur only after merge to `main` through `playground-deploy`, using an immutable package version and a child folder beneath the approved `JD_Demos/demos` deployment parent.

## Human decisions in the Flow

- **Minimum-element/extraction review:** The intake specialist sees immutable source references, extracted values, evidence spans, missing minimum elements, extraction confidence, and conflicts. Corrections require rationale and retain original values.
- **Medical and regulatory review:** The authorised reviewer sees the clock, versioned rule result, duplicate candidates, label excerpt, agent recommendations, missing evidence, and all proposed medical/reporting fields. Editable values are seriousness criteria, expectedness/listedness, duplicate disposition/master case, day zero, due date, report type, follow-up fields, and rationale.
- **Named outcomes:** `ApproveMockRelease` permits only mock finalisation; `CorrectAssessment` persists changes and then permits the selected mock route; `RequestInformation` creates a preview and sets `awaiting_information`; `Escalate` assigns the safety lead and performs no release work.
- **Timeout and resumption:** The demo timeout is configuration pending organisational policy. Expiry notifies the pharmacovigilance operations lead, preserves the case and clock, and does not infer a decision. Flow resumes from the task completion handle, validates outcome and returned field IDs, records every change, and routes only from the returned value.
- **Release boundary:** No agent, API workflow, RPA process, or task button can transmit an ICSR. `ApproveMockRelease` authorises generation and storage of watermarked synthetic artifacts only.

## Controls and safety

| Control | Design decision | Evidence in the Flow or demo |
| --- | --- | --- |
| Regulatory clock | Preserve source receipt, keep calculated and reviewer-confirmed day zero separate, version rules, expose due date/age, and escalate overdue cases. | Canonical fields, clock display, audit event, timeout route, and exact due-date evaluator. |
| Minimum-case validity | Evaluate the four minimum elements deterministically; retain incomplete reports and document follow-up attempts. | Four explicit booleans, missing-field route, review task, and follow-up plan. |
| Medical judgement | Agents recommend with cited evidence only; an authorised person confirms every consequential field and mock release. | Structured recommendations, evidence-reference assertions, task completion handle, and no autonomous release edge. |
| Duplicate safety | RPA retrieves candidates; deterministic fields describe matches; a reviewer confirms merge/master case and new information. | Candidate table, named returned fields, audit rationale, and zero autonomous merge writes. |
| Privacy and access | Synthetic data, protected source zone, least privilege, role-restricted task, masked traces, retention configuration, and no direct identifiers in prompts. | Fixture manifest, tool contracts, task schema, redaction assertions, and access review. |
| Terminology/context drift | Use a synthetic terminology subset and pin label, rule, prompt, and model versions. | Version fields in every assessment and missing/stale-context exception. |
| External showcase isolation | Azure AI Foundry receives constants only and cannot influence validity, assessment, routing, task data, clocks, artifacts, receipts, or status. | No business-variable mappings and an off/on/failing isolation evaluation. |
| Submission prevention | No regulator credential or live endpoint; watermark all artifacts `DEMO - NOT SUBMITTED`; synthetic acknowledgements only. | Environment configuration, artifact assertion, endpoint allowlist, and no send/submission node. |

## Error paths and recovery

| Failure | Safe route | Recovery owner and condition |
| --- | --- | --- |
| Invalid package, missing source file, or hash mismatch | Set `technical_exception`; do not extract or assess. | Intake specialist repairs the fixture and resumes with the same `intakeId`. |
| IXP unavailable, low confidence, or conflicting evidence | Use contract-faithful fixture only in demo mode; otherwise force extraction review. | IXP owner restores the model or intake specialist corrects source-linked values. |
| Missing one or more minimum elements | Retain the report, open review, generate approved follow-up preview, and record attempts. | Intake specialist or reviewer supplies missing information; no case is silently discarded. |
| Duplicate lookup failure | Mark duplicate status `unknown` and require reviewer acknowledgement; do not merge. | RPA/platform owner restores lookup or reviewer documents manual search evidence. |
| Uncited, malformed, or low-confidence agent output | Ignore recommendation and show deterministic/source facts in review. | Agent owner repairs tool/context/prompt; reviewer retains decision authority. |
| Human task timeout | Preserve state and clock; escalate to pharmacovigilance operations lead. | Lead reassigns or escalates under the demo runbook. |
| Mock E2B validation failure | Store errors, keep case open, and route to regulatory operations. | Specialist corrects technical mapping without changing medical fields, then reruns validation. |
| Ambiguous or failed safety-system write | Do not retry automatically or create a success receipt; set `technical_exception`. | RPA owner reconciles target state and resumes explicitly. |
| Follow-up template/tool safety failure | Create manual-draft task; no reporter message is sent. | Communications owner supplies approved text. |
| Azure AI Foundry timeout, error, or unexpected response | Record transient `externalAgentShowcaseStatus`, discard the response, and rejoin the identical core route. | Demo owner may disable the branch; pharmacovigilance users take no recovery action. |

## Observability and evaluation

| Signal or test | What it proves | Expected result or threshold |
| --- | --- | --- |
| Correlation and audit | Source, extraction, rules, tool calls, reviewer changes, receipts, and recovery can be reconstructed. | Every trace/task/receipt carries `intakeId`; ordered audit events contain versions and hashes; no direct identifiers appear in traces. |
| Flow route evaluator | Each fixture reaches its approved safe route and final status. | 5/5 initial synthetic cases match expected route/status before promotion. |
| Minimum-element evaluator | The four validity booleans and missing-field list match the approved oracle. | Exact match for all five cases; any false valid case fails promotion. |
| Serious-case safety evaluator | A serious/unexpected candidate can never bypass medical review. | 100% review recall; any false negative fails the demo gate. |
| Due-date evaluator | Illustrative US clock calculation is deterministic and versioned. | Exact due-date match for every applicable fixture. |
| Evidence-grounding evaluator | Every proposed medical field cites source or approved context. | 100% evidence-reference presence; unknown context yields `insufficient_evidence`. |
| Tool trajectory evaluator | Material agents call only required read-only tools. | Label/rule tools called for every assessment; template tool called for every preview; zero unapproved calls. |
| External showcase isolation | Placeholder external-agent execution cannot change business output. | Branch off, on, and failing produce identical case, route, decision, clock, artifacts, receipts, and final status; only transient showcase status may differ. |
| Receipt reconciliation | Completion cannot hide missing or ambiguous finalisation work. | `completed` only when all outcome-required receipts are present and `succeeded`. |

### Synthetic evaluation set

Dataset name: `pharma-adverse-event-icsr-triage-v1`.

| Case | Expected route | Expected business output |
| --- | --- | --- |
| Complete non-serious listed report | Extraction and deterministic validity, then mandatory medical review and mock finalisation. | Four elements present, listed/non-serious recommendation, approved mock artifact, reconciled receipts. |
| Serious and unexpected report | Prominent clock and mandatory authorised review. | Seriousness criterion cited, exact illustrative due date, no bypass, watermarked mock E2B(R3) artifact. |
| Missing reporter contactability and patient descriptor | Incomplete-case review and `RequestInformation`. | Missing elements preserved, targeted preview generated, status `awaiting_information`, no release receipt. |
| Potential duplicate with new information | Duplicate confirmation view and corrected master-case decision. | Reviewer-confirmed master case/new-information flag in audit; no autonomous merge. |
| Duplicate lookup or validator failure | Recoverable technical exception after safe review stage. | No fabricated result/receipt; owned recovery data and non-completed status. |

## Demo script

1. Show the four-segment canvas, visible regulatory clock, exception routes below the happy path, and final parallel branches.
2. Submit the serious-and-unexpected fixture and point out `intakeId`, immutable source hash, receipt time, and synthetic-data label.
3. Open the extraction and validity trace to show source spans, four minimum-element checks, rule version, and proposed due date.
4. Show duplicate candidates and the inline assessor's read-only label/rule tool calls, cited seriousness criterion, expectedness recommendation, and confidence.
5. Open `Safety Case Medical Review`, correct one proposed field or duplicate disposition, confirm day zero/due date, enter rationale, and select `CorrectAssessment` or `ApproveMockRelease`.
6. Return to Flow and show mock E2B generation/validation, safety-UI write-back, and applicable follow-up-preview work running independently and merging.
7. Open the queue/audit summary to show reviewer changes, versions, watermarked artifact, per-branch receipts, and final status.
8. Point out the disabled Azure showcase branch and prove that its constant input, discarded response, timeout, or failure cannot affect safety-case variables.
9. Preview the lookup/validator-failure fixture to show safe recovery without a false submission or completion claim.

## Success measures

- **Business proof:** The demo exposes intake-to-review-ready time, deadline adherence, completeness, follow-up quality, duplicate confirmation, override, validator rework, backlog age, and receipt reconciliation without inventing targets.
- **Flow proof:** A viewer can see IXP, deterministic rules, two material agent roles, API/RPA contrast, the non-material Azure showcase, real business routing, human authority, parallel work, merge, and recoverable exceptions.
- **Demo proof:** In under ten minutes, a viewer can verify immutable intake, source-linked recommendations, a reviewer correction, downstream consumption, artifact watermarking, receipt reconciliation, and a failed-dependency route.
- **Build proof:** Every project validates, the five-case evaluation set passes, all Flow warnings are resolved, bindings are recorded, `resources refresh` and dry-run pack succeed, and immutable packaging can follow changed-solution CI into Playground after merge.

## Reference mapping

| Reference requirement | Domain-specific implementation | Evidence or gap |
| --- | --- | --- |
| 3-4 segment topology and canvas rules | Four blue sticky notes: Receive and extract the report; Validate and assess the case; Review the safety decision; Finalise and evidence the case. Happy path is left-to-right, exceptions below, parallel branches symmetric. | Fully specified; canvas implementation remains. |
| IXP/document intelligence | `PharmaSafetyReportExtraction` extracts required fields, source spans, and confidence; low confidence/conflict forces review. | Contract and fixture fallback specified; model/folder remain to provision because no Pharma-specific IXP node was found. |
| API workflow and RPA on the intended path | `safety-case-gateway` normalises/rules/builds mock E2B; `legacy-safety-console` searches and writes the UI-only mock safety system. | Contracts specified; projects and UI fixture remain to build; real API gap requires human validation. |
| Inline agent with a wired tool | `Safety Case Assessor` calls approved-label and case-rule tools and returns structured cited recommendations. | Contract specified; agent, context, and tool resources remain to build. |
| Coded agent with visible value-add | `Safety Follow-up Writer` retrieves an approved template and produces a restricted, evidence-linked preview with safety findings. | Contract specified; coded agent/MCP server remain to build. |
| Shared external-agent showcase | Verified Azure AI Foundry node/connection on a constant-input, discarded-output, fail-open branch with no business mappings. | Node tenant availability and connection health verified; Flow binding and selected `agent_id` remain to validate. |
| Real business decision and safe exception | Minimum elements, duplicate status, and confidence drive visible routes; missing evidence, timeout, validator, and write failures remain safe. | Expressions and outcomes specified; implementation/evaluation remain. |
| Human decision and returned outcome data | Coded action app returns assessment corrections, duplicate disposition, clock/reporting fields, rationale, reviewer, timestamp, and named outcome. | Contract specified; role delegation, SLA, and deployed app require approval/provisioning. |
| Purposeful parallelism and merge | Outcome-required mock E2B, safety-UI, and follow-up-preview branches merge before receipt reconciliation. | Fully specified; required-receipt matrix remains to encode. |
| Evaluation set and evaluator | Five cases plus route, validity, safety, due-date, grounding, trajectory, isolation, and receipt evaluators. | Exact initial gates specified; fixtures/evaluators remain to build. |
| Process-app variant | Not selected. Queue is the canonical demo record. | Later cross-domain selection may replace the record/app design. |
| Solution boundary and delivery contract | One `pharma-adverse-event-icsr-triage` solution, one `.uipx`, nested Flow layout, resource refresh, immutable version, and changed-solution CI. | Fully specified; deployment child folder/resources remain to provision. |

## Open human decisions

These decisions refine or approve implementation but do not block the synthetic, mock-backed default described above.

| Decision | Owner | Default and resolution path |
| --- | --- | --- |
| Approve or replace the US post-market demo scope and illustrative rule set. | Pharmacovigilance quality and regulatory affairs | Default is synthetic US post-market human-drug intake; approve versioned rules or revise fixtures, clock oracle, and task fields before build sign-off. |
| Confirm delegated reviewer roles and field authority. | Pharmacovigilance quality/medical governance | Default separates intake corrections from authorised medical/regulatory decisions; map organisational roles and maker-checker requirements. |
| Approve the email/PDF hero source and day-zero procedure. | Intake process owner and pharmacovigilance quality | Default is manual submission of a synthetic email package with preserved receipt time; approve mailbox/event semantics before enabling Outlook. |
| Select the monitored mailbox/Outlook connection or retain manual intake. | Mailbox owner and UiPath tenant administrator | Manual trigger is default. If email intake is selected, choose and ping an approved connection rather than silently binding the visible non-default connection. |
| Validate the safety-system API gap and RPA responsibility. | Safety-system owner and enterprise architect | Local UI-only fixture is default. Retain RPA only for a proven API gap; otherwise choose another credible UI-only step or approve a reference-pattern deviation. |
| Approve product-label, terminology, and rule content. | Medical information, terminology, and regulatory owners | Use synthetic versioned labels/terms initially; do not represent them as licensed MedDRA or production rules. |
| Approve privacy, security, retention, and generative-AI controls. | Privacy, security, pharmacovigilance quality, and validation owners | Synthetic data and no external writes are mandatory until the data path, masking, access, audit, and validation approach are approved. |
| Set human-task escalation timing and pilot baselines. | Pharmacovigilance operations lead | Keep timeout configurable and make no improvement claim until observed baselines/targets are supplied. |
| Select the solution child folder and provision resources. | UiPath tenant administrator | Create the solution child folder beneath the approved `JD_Demos/demos` deployment parent, then provision the queue, IXP, agents, action app, runtime, and bindings. |
| Decide whether Pharma is a later Data Fabric/process-app variant. | Demo portfolio owner | This spec marks it not selected; a later cross-domain decision may change the canonical record and review experience. |

## Implementation tasks

1. Scaffold `pharma-adverse-event-icsr-triage` with the nested Flow project and exactly one `.uipx` manifest.
2. Author the five synthetic intake packages, expected route/output oracle, versioned US demo rules, synthetic label/terminology corpus, queue contract, mock E2B endpoint, and local safety UI.
3. Implement and validate `safety-case-gateway`, `PharmaSafetyReportExtraction` or its contract-faithful mock, and `legacy-safety-console` with typed errors and receipts.
4. Build the inline assessor, read-only label/rule tools, deterministic validity/clock checks, structured-output validation, and grounding/safety evaluators.
5. Build the coded follow-up writer, least-privilege template MCP tool, and trajectory/preview-safety evaluators.
6. Build and deploy the coded action app, then wire its task handle, returned field IDs, named outcomes, corrections, and timeout escalation into Flow.
7. Author the four-segment Flow, exception routes, real decision expressions, non-material Azure showcase, outcome-specific parallel branches, merge, and receipt reconciliation.
8. Bind and validate Azure connection `0107247a-0197-42c9-b957-05d1b722b111`; prove off/on/failing showcase runs cannot change case data, routing, decisions, clocks, artifacts, receipts, or final status.
9. Run project validation and `pharma-adverse-event-icsr-triage-v1`; resolve every warning and failed threshold without executing live external writes.
10. Refresh solution resources, restore, dry-run pack, and register immutable deployment configuration for changed-solution CI.

## Quality rubric

| Dimension | Score (0-3) | Evidence and remaining gap | Owner and resolution path |
| --- | ---: | --- | --- |
| Enterprise credibility | 2 | Regulated decision, roles, data contract, controls, evidence, measures, and safe scope are explicit; organisation-specific rules, authority, and validation remain unapproved. | Pharmacovigilance quality, medical governance, privacy, and system owners approve during discovery. |
| Flow differentiation | 3 | Four segments visibly coordinate IXP, deterministic rules, agents, API, RPA, human decisions, parallel work, merge, clock, and recovery. | Flow implementer preserves topology, port-level error routes, and business expressions. |
| Demo clarity | 3 | Serious-case hero journey, duplicate correction, external-agent isolation, failed-dependency route, and observable proof points form a timed script. | Demo owner authors fixtures and rehearses after deployment. |
| Build feasibility | 2 | Inputs, outputs, mocks, fallbacks, solution boundary, evaluations, authenticated target, and verified Azure connection are recorded; domain resources remain unprovisioned. | Tenant administrator and implementers provision resources and record every binding. |
| **Total** | **10/12** | **Ready for implementation planning as a synthetic, mock-backed demo; not ready for regulated use, real case processing, reporter contact, or submission.** | **Start with fixtures/contracts, then close owned human decisions before replacing mocks or enabling connectors.** |
