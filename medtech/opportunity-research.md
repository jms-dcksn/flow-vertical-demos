# Medtech agentic-workflow opportunity research

## Executive recommendation

Build a **medical-device complaint-to-MDR decision** demo. A manufacturer receives
an incomplete complaint, identifies the device and event, gathers missing evidence,
assesses US Medical Device Reporting (MDR) criteria, sends the consequential
reportability decision to a qualified vigilance reviewer, and prepares the approved
submission and follow-up work.

This is the strongest candidate because it combines unstructured intake, regulated
deadlines, evidence retrieval, bounded agent reasoning, a real human decision, and
auditable cross-system action. FDA says it receives several hundred thousand medical
device reports of suspected deaths, serious injuries, and malfunctions each year, while
also warning that the reports may be incomplete, inaccurate, untimely, unverified, or
biased. That creates a credible orchestration problem without implying that an agent
can decide causality or reportability by itself. [FDA openFDA MAUDE overview](https://open.fda.gov/data/maude/)

The research is a greenfield, public-source assessment dated 2026-08-10. It does not
observe one manufacturer's operations, prove a replicable internal automation, or
support an ROI estimate. Rankings and solution designs below are explicitly labeled
as inferences where they go beyond the cited facts.

## Evidence base

| Evidence-backed pain point or obligation | Workflow, actors, systems, and controls established by the source | Measurable fact or outcome |
| --- | --- | --- |
| Adverse-event inputs are high-volume and imperfect. | FDA's MAUDE data comes from manufacturers, importers, user facilities, clinicians, patients, and consumers. FDA cautions that passive reports can be incomplete, inaccurate, untimely, unverified, or biased and cannot establish incidence or causality alone. Affected functions include complaint handling, product safety, quality, and regulatory affairs; MAUDE is a downstream surveillance system. [FDA openFDA MAUDE overview](https://open.fda.gov/data/maude/) | FDA receives **several hundred thousand** medical-device reports each year. A safe automation outcome is therefore field completeness and evidence traceability, not an automated causal conclusion. |
| Manufacturer reportability has fixed clocks and requires investigation. | Manufacturers report deaths, serious injuries, and qualifying malfunctions within 30 calendar days; a 5-working-day route applies when remedial action is needed to prevent unreasonable risk of substantial harm or FDA requests it. Manufacturers must investigate, seek reasonably known information, explain missing data, and later submit newly obtained required information. [FDA MDR requirement summary](https://www.fda.gov/medical-devices/guidance-documents-medical-devices-and-radiation-emitting-products/attachment-c-summary-mdr-reporting-requirements) | **30 calendar days**, **5 working days**, and a supplemental report within **one month of receiving** required follow-up information are deterministic timers suitable for orchestration and audit. |
| Individual adverse-event reports are electronic. | Manufacturers and importers generally submit individual MDRs electronically; FDA's Unified Submission Portal replaced legacy WebTrader in April 2025. The submission boundary is a regulated external dependency, not a safe place for an unconstrained agent. [FDA eMDR overview](https://www.fda.gov/medical-devices/mandatory-reporting-requirements-manufacturers-importers-and-device-user-facilities/emdr-electronic-medical-device-reporting) | An observable outcome is accepted/rejected submission status and reconciliation to the approved complaint record. The public demo should mock submission because tenant and regulator credentials are not available. |
| Post-market surveillance must feed quality and risk processes continuously. | Current European guidance describes active, systematic collection across device life, updates to technical documentation and risk management, defined indicators and thresholds, complaint-investigation methods, communication procedures, and criteria that can initiate nonconformance, CAPA, or field safety corrective action. Actors include PMS, quality, clinical, risk, regulatory, and management review. [European Commission MDCG 2025-10](https://health.ec.europa.eu/document/download/a9ad86b7-1b8e-4bae-beb4-48b2b3ed2f05_en?filename=mdcg_2025-10_en.pdf) | The guidance calls for measurable values such as batch quantity, use frequency, devices in use or sold, exposures, complaint rate, and severity, plus defined action thresholds. It does not prescribe a universal threshold. |
| The US quality-system framework changed recently. | FDA's Quality Management System Regulation (QMSR) became effective on 2026-02-02 and incorporates ISO 13485:2016 by reference for finished-device manufacturers. Any demo representing an operational quality record must preserve controlled procedures and records rather than treating generated text as the system of record. [FDA QMSR overview](https://www.fda.gov/medical-devices/postmarket-requirements-devices/quality-management-system-regulation-qmsr) | Effective date: **2026-02-02**. A measurable demo control is whether every recommendation, override, and write-back is correlated to one controlled complaint record. |
| Corrections and removals require coordinated data and communication. | A reportable correction or removal under 21 CFR Part 806 must capture device identity, lots/serials, distribution, consignees, injuries, actions, and communications. Manufacturers and importers own the report; unreported actions still require records. [FDA recalls, corrections, and removals](https://www.fda.gov/medical-devices/postmarket-requirements-devices/recalls-corrections-and-removals-devices) | A report is due within **10 working days** of initiating a reportable correction or removal; extensions to more lots or batches also require amendment within **10 working days**. |
| Connected-device cybersecurity spans the product lifecycle. | FDA's February 2026 final guidance covers cybersecurity design, labeling, submission documentation, and section 524B expectations for cyber devices. The broader response involves product security, safety/risk, engineering, regulatory, and customer communications. [FDA cybersecurity guidance](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/cybersecurity-medical-devices-quality-management-system-considerations-and-content-premarket) | Credible outcomes are time to assess, approved remediation route, affected-device traceability, and communication completion. No universal remediation-time target is asserted here. |
| Device master data is distributed and has both manual and machine submission paths. | FDA notes that GUDID preparation may require collection and validation across multiple systems and locations. Labelers can enter a few records manually or submit HL7 SPL XML, which requires technical setup and testing, for larger sets. [FDA GUDID preparation](https://www.fda.gov/medical-devices/global-unique-device-identification-database-gudid/prepare-gudid), [FDA GUDID submission](https://www.fda.gov/medical-devices/global-unique-device-identification-database-gudid/submit-data-gudid) | Observable outcomes include first-pass validation, reconciliation of approved device attributes, and reduction of manual re-entry. No baseline or improvement percentage is available from these sources. |

## Ranked demo candidates

Scores use a 1–5 research rubric: **demo value** rewards a legible multi-actor
hero moment; **enterprise credibility** rewards authoritative evidence of a
consequential workflow; **feasibility** rewards a safe synthetic-data demo with
mockable boundaries. The scores are design judgments, not measured business value.

| Rank | Candidate | Demo value | Enterprise credibility | Feasibility | Total | Why it ranks here |
| --- | --- | ---: | ---: | ---: | ---: | --- |
| 1 | Complaint-to-MDR decision orchestration | 5 | 5 | 4 | **14/15** | The 30-day/5-day routes, imperfect narrative and attachments, required investigation, qualified human decision, and electronic submission make all four reference segments visible. FDA evidence directly supports the workflow and scale. Submission and enterprise systems can be mocked. [FDA MDR requirements](https://www.fda.gov/medical-devices/postmarket-requirements-devices/mandatory-reporting-requirements-manufacturers-importers-and-device-user-facilities) |
| 2 | Field correction and recall coordination | 5 | 5 | 3 | **13/15** | It offers a strong command-center story across product safety, quality, supply chain, regulatory, distributors, and customers. The 10-working-day report, lot/serial scope, consignee list, and communication evidence are concrete. Feasibility is lower because realistic distribution reconciliation and regulator submission require several mocks. [FDA corrections and removals](https://www.fda.gov/medical-devices/postmarket-requirements-devices/recalls-corrections-and-removals-devices) |
| 3 | Post-market signal-to-CAPA escalation | 4 | 5 | 4 | **13/15** | Continuous evidence collection, complaint-rate/severity indicators, risk thresholds, technical-document updates, and CAPA/FSCA criteria support a credible human-governed signal review. The harder demo problem is creating synthetic longitudinal data that looks meaningful without implying a clinically valid signal algorithm. [MDCG 2025-10](https://health.ec.europa.eu/document/download/a9ad86b7-1b8e-4bae-beb4-48b2b3ed2f05_en?filename=mdcg_2025-10_en.pdf) |
| 4 | Connected-device vulnerability response | 5 | 5 | 3 | **13/15** | A vulnerability can trigger coordinated product-security, safety, engineering, regulatory, patch, and customer-communication branches. Current FDA guidance makes the lifecycle obligation credible. Feasibility depends on a believable software bill of materials, device fleet, exploitability evidence, and approved risk method. [FDA cybersecurity guidance](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/cybersecurity-medical-devices-quality-management-system-considerations-and-content-premarket) |
| 5 | UDI and GUDID change-impact orchestration | 3 | 4 | 5 | **12/15** | Cross-system device data, approvals, validation, XML submission, and reconciliation are highly feasible, and FDA explicitly recognizes multi-system coordination. It ranks lower because the consequential human decision and agentic reasoning are less visually compelling than a safety case. [FDA GUDID preparation](https://www.fda.gov/medical-devices/global-unique-device-identification-database-gudid/prepare-gudid) |

**Inference:** no public evidence identifies an existing automation to replicate,
so this is a greenfield ranking rather than a Tier 1 replication finding. The
complaint-to-MDR candidate is promoted because it has the strongest combination of
source-backed workload, deadline pressure, heterogeneous evidence, safe human
authority, and Flow-visible exception handling.

## Leading candidate: complaint-to-MDR decision orchestration

### Enterprise workflow and hero moment

A complaint arrives by email with a free-text narrative and attachments. Flow creates
a controlled case, extracts device and event facts, reconciles the device against a
master record, identifies missing information, and assembles a source-linked US MDR
assessment. A vigilance specialist sees the evidence, clock, uncertainties, prior
similar complaints, and the agent's rationale in one review experience. The reviewer
selects `Report`, `Do not report`, or `Escalate`, records rationale, and can correct
proposed fields. Flow then prepares the appropriate submission or documented decision,
starts follow-up work, and writes an audit summary.

The agent's output is advisory. It must not diagnose the patient, infer causality as
fact, make the final reportability determination, submit to a regulator without
approval, or suppress a safety escalation.

### Input contract

The demo uses synthetic data only.

| Input | Type | Required | Contract and handling |
| --- | --- | --- | --- |
| `complaintId` | string | yes | Immutable correlation and idempotency key; duplicate receipt resumes or links to the existing case. |
| `receivedAt` | ISO 8601 timestamp | yes | Source for the deterministic regulatory clock; preserved with source timezone and normalized to UTC. |
| `sourceChannel` | enum | yes | `email`, `service_crm`, `distributor`, `user_facility`, or `other`; drives provenance, not reportability. |
| `narrative` | string | yes | Original, immutable source text. Extracted assertions retain spans or attachment/page references. |
| `reporter` | object | yes | Synthetic role, organization, country, and contact route; direct identifiers are restricted and excluded from prompts unless needed. |
| `deviceHints` | object | no | UDI-DI, brand, model, catalog, lot, serial, software version, and implant/explant dates when supplied. Unknown values remain unknown. |
| `eventFacts` | object | no | Event date, country, alleged outcome, intervention, device availability, and remedial action. Values include provenance and `known`, `unknown`, or `conflicting` status. |
| `attachments` | array | no | Synthetic PDFs/images/service logs with media type, checksum, malware-scan status, and source. Untrusted content is data, not instructions. |
| `marketScope` | array | yes | Demo value is `US`; other jurisdictions route to `manual_policy_review` until their approved rules are configured. |
| `priorCaseRefs` | array | no | Candidate links only; matching does not establish recurrence, causality, or reportability. |

### Output contract

| Output | Type | Contract |
| --- | --- | --- |
| `canonicalComplaint` | object | Normalized device, reporter, event, and attachment facts with per-field provenance, confidence, and conflict markers. |
| `deviceMatch` | object | Selected device-master record, candidate alternatives, match evidence, and `confirmedByHuman` flag. |
| `missingInformationPlan` | object | Required or useful missing fields, reason, authorized recipient, question draft, owner, attempt history, and due date. |
| `assessmentRecommendation` | object | `candidate_reportable`, `candidate_not_reportable`, or `insufficient_information`; death/serious-injury/malfunction and 5-day-remedial-action indicators; policy citations; evidence; uncertainty; and prohibited autonomous action flag. |
| `clockState` | object | Awareness timestamp, ordinary and expedited due dates, selected route, elapsed time, pause policy if any, and SLA alerts. Clock computation is deterministic code, never an LLM calculation. |
| `reviewDecision` | object | Reviewer identity/role, `Report`, `Do not report`, or `Escalate`, corrected fields, rationale, decision time, and electronic signature reference. |
| `submissionPackage` | object | Approved synthetic FDA 3500A/eMDR field mapping, validation results, version, and source-record links; no submission occurs in the demo. |
| `followUpActions` | array | Investigation, information request, supplemental-report reminder, trend review, CAPA evaluation, or urgent safety escalation with owner and status. |
| `auditSummary` | object | Case state transitions, prompts/model version, tool calls, source checksums, human overrides, write-backs, and mock-submission acknowledgement. |

### Integrations and demo mocks

| Component | Purpose | Demo implementation | Production question |
| --- | --- | --- | --- |
| Email/service CRM trigger | Receive complaint and attachments. | Local synthetic event and fixture files. | Which channel establishes manufacturer awareness, and how are duplicates reconciled? |
| IXP document extraction | Classify attachments and extract device/event fields with page evidence. | Synthetic service report, complaint form, and device label; low confidence routes to review. | Which deployed IXP model, folder, supported languages, and confidence rules are approved? |
| Device master API workflow | Resolve UDI/model/lot against PLM/ERP and return controlled attributes. | Mock REST endpoint with deterministic device records and errors. | Which system is authoritative for each attribute and what connector is available? |
| eQMS complaint record | Persist controlled case state, investigation, decision, and CAPA link. | Mock API for read; RPA writes approved fields into a mock legacy web UI to visibly prove the no-API path. | Is TrackWise, MasterControl, another eQMS, or a custom system authoritative; is API write access permitted? |
| SOP and regulatory context | Ground the assessment in approved, versioned procedures and public rules. | Synthetic SOP corpus plus cited FDA pages; retrieval returns document version and passage identifier. | Who approves content, jurisdiction rules, effective dates, and retirement of superseded procedures? |
| Similar-case search | Provide candidate prior complaints and outcomes to the reviewer. | Synthetic indexed complaint set with deterministic metadata filters. | Which data may be searched, and may prior decisions be used as precedent? |
| FDA submission boundary | Validate and transmit an approved eMDR package. | Schema validator and mock regulator endpoint returning accepted, rejected, or unavailable. | Which validated gateway, certificate, acknowledgement, reconciliation, and business-continuity process applies? |
| Action app | Present evidence and collect the reportability decision. | Coded action app contract; implementation is a later issue. | Which roles have decision/signature authority and what is the escalation roster? |

The current workspace has UiPath CLI 1.198.0. Live resource readiness was not
verified: `uip login status` returned `Refresh Failed` because the auth-file lock
could not be acquired. Consequently, all resource names and connections above remain
design contracts or mocks, not claims about an authenticated tenant.

### Human decisions and escalation

| Decision | Human owner | Information shown | Outcomes and downstream route |
| --- | --- | --- | --- |
| Confirm device and awareness date | Complaint/vigilance intake specialist | Source evidence, candidate device records, conflicting dates, duplicate candidates | `Confirm` continues; `Request information` creates a tracked request; `Escalate identity` routes to device-data owner. |
| Determine US MDR reportability and clock | Qualified regulatory/vigilance specialist | Canonical facts, cited policy/SOP, missing data, recommendation, similar cases, clock, and all uncertainties | `Report` locks approved fields; `Do not report` requires rationale and controlled record; `Escalate` routes to medical/product-safety authority. |
| Approve external package | Authorized regulatory submitter | Final field mapping, validation results, differences from source and prior review | `Approve mock submission`, `Return for correction`, or `Hold and escalate`. No auto-submit path. |
| Decide quality follow-up | Quality engineer/product-safety owner | Investigation evidence, recurrence candidates, risk-file references, and reviewer outcome | Open/associate investigation, evaluate CAPA or safety action, request more evidence, or document no further action under procedure. |

Timeouts never default to `Do not report`. They escalate to a named duty queue while
the deterministic clock continues and the audit record preserves notifications.

### Flow topology and reference mapping

| Reference segment | Medtech canvas segment | Intended-path actors and outputs | Route evidence |
| --- | --- | --- | --- |
| Receive and understand | **1. Capture the safety complaint** | Trigger creates `complaintId`; IXP extracts the complaint form, service report, and device label; API workflow resolves device master data. Output: canonical complaint and confidence. | Missing identity, conflicting dates, unsafe attachment, or low confidence routes below the happy path to intake review. |
| Assess and enrich | **2. Build the reportability evidence** | Inline low-code agent uses approved SOP/context search and similar-case lookup to return the structured assessment. A narrowly scoped coded agent drafts source-linked follow-up questions; deterministic code calculates clocks. Output: recommendation, missing-information plan, and due dates. | `expeditedCandidate`, `informationComplete`, and `assessmentRecommendation` drive real business branches. Tool failures yield `insufficient_information`, never a negative decision. |
| Decide and review | **3. Make the regulated decision** | Coded action app presents evidence to the vigilance specialist and returns corrected fields, outcome, rationale, and signature reference. Output: locked review decision or escalation. | `Report`, `Do not report`, and `Escalate`; timeout routes to duty queue. The Flow consumes returned fields rather than ending at the form. |
| Act and communicate | **4. Prepare, follow up, and record** | Parallel branches prepare/validate the mock eMDR package, create reporter follow-up, and open quality evaluation. RPA writes approved data to the mock legacy eQMS. Merge writes the audit summary and status. | A merge waits for all required branches. Rejection, RPA exception, or mock endpoint outage opens a recoverable work item with the same correlation ID. |

Canvas intent: four left-to-right sticky notes in blue, amber, purple, and green;
exception paths below the happy path; symmetric post-review branches; one merge before
completion. Node names use business actions, such as `Assess MDR criteria (agent)` and
`Decide reportability (vigilance review)`.

### Agent boundaries and controls

| Risk or compliance concern | Required control | Observable demo evidence |
| --- | --- | --- |
| Patient and reporter privacy | Synthetic data only; field-level minimization; encryption and least privilege in a real deployment; do not send direct identifiers to a model unless explicitly approved and necessary. | Prompt/tool trace shows redacted payload; access-denied and retention behavior are tested. |
| Incorrect reportability or causality conclusion | Agent output is a recommendation with citations, provenance, uncertainty, and an explicit `insufficient_information` state. Qualified human owns the decision and signature. | No external package exists before an approved decision; human overrides and rationale are immutable audit events. |
| Missed expedited route or deadline | Deterministic rules compute clocks from confirmed awareness time. Agent cannot change deadlines. Urgent indicators and review timeout escalate to a duty queue. | Synthetic 5-day case enters the expedited branch and produces alerts. |
| Hallucinated or superseded policy | Retrieval is restricted to approved, versioned content. Every criterion returns a source ID/effective date; absent support produces uncertainty, not a fabricated rule. | Evaluation rejects unsupported citations and use of retired SOP versions. |
| Prompt injection or malicious attachment | Files are scanned; extracted document text is treated as untrusted data; agent tools are allow-listed and have schema validation and least privilege. | Injection fixture cannot alter tool policy or bypass review. |
| Duplicate or conflicting records | `complaintId`, checksums, source IDs, and duplicate candidates support idempotency; conflicts remain explicit until a person resolves them. | Replay does not create a second case or mock submission. |
| Unsafe write-back or submission | Separate read, draft, approve, and execute permissions; schema validation; human approval; mock external endpoint; acknowledgement reconciliation. | Rejected validation and endpoint outage take a recoverable exception path. |
| Bias from prior complaints | Similar cases are context, not ground truth; retrieval is metadata-filtered and the reviewer sees source quality limitations. | Assessment remains possible without prior cases and never derives incidence from MAUDE counts. |
| Auditability under QMSR | Preserve source evidence, controlled procedure version, agent/model version, tool calls, human decisions, corrections, and final write-backs under the eQMS retention policy. | One correlation ID reconstructs the full case timeline. |

### Evaluation and measurable outcomes

No improvement target is claimed without a manufacturer baseline. A pilot should first
measure current performance, agree target thresholds with Quality and Regulatory, and
then compare like-for-like synthetic or historical cases.

| Measure | Definition |
| --- | --- |
| Deadline compliance | Percentage of cases with a documented human decision and, when required, accepted submission before the applicable clock. |
| Time to decision | Median and 90th percentile from confirmed awareness time to signed reportability decision, segmented by ordinary and expedited routes. |
| First-review completeness | Percentage of required review fields supported by source evidence at the first vigilance review. |
| Recommendation concordance | Agreement with a qualified-expert gold set, reported separately for `report`, `do not report`, and `insufficient information`; false-negative safety errors are shown explicitly. |
| Evidence traceability | Percentage of populated regulated fields linked to an immutable source location or a named human correction. |
| Rework and recovery | Returned-for-correction rate, duplicate rate, failed write-backs, unacknowledged mock submissions, and mean time to recover. |
| Human governance | Percentage of final decisions with authorized reviewer, rationale, timestamp, and complete override history; target for the demo is 100%. |

Minimum synthetic evaluation set:

1. A complete non-expedited serious-injury complaint reaches `Report`, human approval,
   validated mock package, and audit completion.
2. A remedial-action case raises `expeditedCandidate` and the 5-working-day route; the
   agent cannot downgrade it.
3. A vague malfunction with missing device identity returns `insufficient_information`
   and a tracked request rather than `Do not report`.
4. Conflicting label and service-report identifiers stop at human identity review.
5. A prompt-injection attachment plus unavailable mock submission proves tool
   restrictions, human authority, and recoverable exception handling.

The principal evaluator checks expected route, clock, reviewer outcome, and presence
of source evidence. A trajectory evaluator checks that the inline agent used only the
approved policy and similar-case tools. Deterministic tests cover clock calculations,
idempotency, schema validation, and the prohibition on submission without approval.

### Proposed solution boundary

- Solution: `medtech/complaint-mdr/complaint-mdr-solution/`
- Globally unique package name: `medtech-complaint-mdr`
- One `.uipx` manifest with nested Flow, API-workflow, RPA, and agent projects.
- Coded action app deployed independently and referenced through its action contract.
- Process-app/Data Fabric variant: **not selected** at research stage; the canonical
  production record is assumed to remain the manufacturer's eQMS complaint record.
- Delivery must refresh solution resources before restore/pack, use an immutable
  package version, validate each project, and scope CI to this solution folder.

## Unresolved assumptions for human review

1. Is the first demo intentionally US-only, or must it make a simultaneous EU/other
   jurisdiction decision? The proposed safe default is US-only with manual routing for
   unconfigured markets.
2. What event starts the awareness clock under the manufacturer's approved procedure,
   and can the intake timestamp ever be corrected? This determines timer semantics.
3. Which roles may decide reportability, electronically sign, submit, and approve
   quality follow-up? The demo needs role names and timeout escalation owners.
4. Which eQMS, service CRM, PLM/ERP, document repository, and submission gateway are
   representative, and which have usable APIs? Current integrations are mocks.
5. Which approved SOPs, regulatory interpretations, device risk files, and controlled
   terminology may ground the agent? Public rules alone are not an operational policy.
6. What fields are patient information, personal data, protected health information,
   confidential business information, or export-controlled data, and which model/data
   residency choices are approved?
7. What confidence thresholds require extraction review? These must be calibrated with
   representative documents and cannot be invented from public research.
8. May similar complaints and past human decisions be retrieved, and how should
   superseded or inconsistent precedents be handled?
9. Is RPA into a legacy eQMS a useful part of the target story, or does the selected
   system provide an approved write API? Both should not be included merely for breadth.
10. What manufacturer baseline and agreed targets apply to completeness, cycle time,
    concordance, rework, and on-time submissions? No ROI or improvement percentage is
    supportable yet.
11. Which regulator test environment or validated submission component, if any, can be
    demonstrated? Until confirmed, external submission remains a mock.
12. Should this become one of the three Data Fabric-backed process-app demos? Doing so
    would change the canonical-record design and needs a repository-level decision.

## Research boundaries

- Sources are public FDA and European Commission materials, not internal behavioral
  data. No employee single point of failure or department automation coverage can be
  established.
- Regulatory guidance is summarized for demo discovery, not legal or regulatory
  advice. The manufacturer's qualified functions must approve jurisdiction-specific
  logic and controlled content.
- MAUDE report counts cannot establish incidence, comparative safety, causality, or a
  manufacturer's addressable volume; FDA explicitly states these limitations.
- No internal metric, benefit percentage, ROI, delivery hours, or pack-hours is
  estimated. Those require manufacturer evidence and, for effort sizing, the
  user-supplied authoritative matrices and catalogue.
- Resource and connection readiness remains unknown because the active UiPath auth
  target could not be verified during research.
