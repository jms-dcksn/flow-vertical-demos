# Healthcare provider agentic-workflow opportunities

**Research date:** 2026-08-10
**Scope:** Public evidence for United States healthcare-provider workflows; no
provider's internal data, policies, volumes, or technology estate was available.

## Recommendation

Build the first healthcare-provider demo around **closed-loop abnormal diagnostic
result follow-up**. The hero moment is a clinician reviewing an evidence-linked,
policy-constrained follow-up recommendation for an abnormal result, changing or
accepting the plan, and watching Flow safely complete and audit the downstream
work.

This is an inference from public evidence, not a claim about any specific health
system. AHRQ reports that prior studies found 6.8–62% of abnormal laboratory
tests and 1.0–35.7% of abnormal imaging results were not followed up, while also
warning that the exact burden remains uncertain across settings. AHRQ identifies
result routing and tracking as persistent safety concerns.
([AHRQ diagnostic safety review](https://www.ahrq.gov/diagnostic-safety/resources/issue-briefs/dxsafety-current-state3.html))
The current federal SAFER guide recommends explicit ownership, coverage during
clinician absence, escalation of unacknowledged abnormal results, reminders for
overdue actions, and measurement of acknowledgment and patient follow-up.
([2025 SAFER Test Results Reporting and Follow-Up guide](https://healthit.gov/wp-content/uploads/2025/06/SAFER-Guide-8.-Test-Results-Reporting-Final.pdf))

No ROI, hours-saved estimate, implementation effort, or production performance
target is asserted here. Those require a provider baseline and its approved
delivery-sizing catalogue.

## Evidence-backed opportunity landscape

| Workflow and pain point | Actors and systems | Required controls | Observable outcome |
| --- | --- | --- | --- |
| Abnormal result follow-up can fail through unclear ownership, inbox coverage, routing, acknowledgment, and patient notification. AHRQ's cited study ranges above establish material but variable risk. | Ordering/covering clinician, diagnostic service, clinical support staff, patient; EHR inbox, LIS/RIS/PACS, FHIR interface, portal, scheduling. | The SAFER guide assigns responsibility to the ordering clinician until an accepted transfer, calls for alternate-provider escalation, and recommends monitoring acknowledgment and patient follow-up. | Time to clinician acknowledgment; results overdue; documented plan; time to patient notification; completed follow-up action. These are measurement candidates, not promised improvements. |
| Provider prior-authorization teams assemble evidence, respond to requests for information, and challenge questionable denials. HHS OIG found 13% of sampled denied Medicare Advantage prior-authorization requests met Medicare coverage rules. ([HHS OIG](https://oig.hhs.gov/reports/all/2022/some-medicare-advantage-organization-denials-of-prior-authorization-requests-raise-concerns-about-beneficiary-access-to-medically-necessary-care/)) | Ordering clinician, authorization specialist, utilization management, payer; EHR, document store, payer portal, fax, X12/FHIR APIs. | Preserve source evidence and payer criteria; clinician validates medical necessity; deadline and appeal-state controls; no autonomous submission of unsupported assertions. | First-pass completeness; requests for more information; decision turnaround; approval/denial/appeal disposition. |
| Discharge planning spans clinical readiness, patient preference, post-acute placement, information transfer, education, and follow-up. CMS requires hospitals to involve patients/caregivers and send necessary information to post-acute and outpatient providers. ([CMS discharge planning rule](https://www.cms.gov/newsroom/fact-sheets/cms-discharge-planning-rule-supports-interoperability-and-patient-preferences)) | Hospitalist, RN/case manager, social worker, pharmacist, patient/caregiver, post-acute provider; EHR, bed/placement network, payer portal, pharmacy, transport. | Qualified staff own the plan; preserve patient goals and freedom of choice; verify medication and information transfer; human approval before discharge. | Barriers resolved; time medically ready to discharge; completed handoffs and follow-up; 30-day unplanned readmission. CMS uses 30-day measures in HRRP. ([CMS HRRP](https://www.cms.gov/medicare/quality/value-based-programs/hospital-readmissions)) |
| Ambulatory referrals can break between order, scheduling, specialist consultation, report receipt, plan communication, and patient follow-up. AHRQ's PSNet describes a nine-step loop and recommends interoperable records, standardized handoffs, explicit accountability, and patient communication. ([AHRQ PSNet](https://psnet.ahrq.gov/issue/closing-loop-guide-safer-ambulatory-referrals-ehr-era)) | Referring clinician, referral coordinator, specialist, patient; EHR, referral workqueue, directory, scheduling, fax/document exchange, portal. | Confirm referral acceptance and ownership; track aging and missing reports; escalate urgent or overdue cases; document patient communication. | Referral accepted; appointment scheduled; consult report received; plan acknowledged; loop closed. |

The 2024 CMS interoperability rule strengthens the feasibility case for digital
prior authorization: impacted payers must support FHIR prior-authorization APIs
beginning in 2027 and operational provisions generally began in 2026. It also
creates provider adoption measures, but payer coverage and implementation timing
must be checked for any selected demo.
([CMS final-rule fact sheet](https://www.cms.gov/newsroom/fact-sheets/cms-interoperability-prior-authorization-final-rule-cms-0057-f))

## Ranked demo candidates

Scores use a 1–5 ordinal scale. **Demo value** measures visible orchestration and
actor contrast; **enterprise credibility** measures consequence, ownership, and
evidence; **feasibility** measures whether a convincing synthetic demo can be
built with bounded mocks. Equal weighting avoids implying a financial model.

| Rank | Candidate | Demo value | Enterprise credibility | Feasibility | Total | Why it ranks here |
| ---: | --- | ---: | ---: | ---: | ---: | --- |
| 1 | Closed-loop abnormal diagnostic-result follow-up | 5 | 5 | 5 | **15** | Strong safety evidence, clear accountable clinician, natural deadlines/escalation, and a compact FHIR/PDF-to-human-to-action story. |
| 2 | Prior-authorization exception and denial rescue | 5 | 5 | 4 | **14** | Consequential document reasoning and payer interaction with timely regulatory momentum; payer-specific rules and portal variance reduce portability. |
| 3 | Inpatient discharge barrier and post-acute transition orchestration | 5 | 5 | 3 | **13** | Excellent cross-team orchestration and measurable outcomes, but many live dependencies make a faithful first demo harder. |
| 4 | Ambulatory referral-loop closure | 4 | 4 | 4 | **12** | Credible multi-party follow-up and escalation; less visually distinctive than result follow-up without a carefully chosen urgent referral. |

The ranking is a demo recommendation, not a prevalence or ROI ranking. No proven
internal automation was available to replicate, so the automation-discovery
method's replication tier is intentionally empty.

## Strongest candidate: closed-loop abnormal result follow-up

### Audience journey and scope

A patient-safety or ambulatory-operations audience sees a new radiology result
arrive, Flow create a correlated case, reconcile structured data and the source
report, summarize the finding against an approved local follow-up policy, route
it by safety state, and pause for the responsible clinician. The clinician sees
the report evidence, provenance, policy passage, patient context, and proposed
plan; after the clinician decides, independent documentation, scheduling, and
patient-communication work runs in parallel and rejoins at an auditable close.

The demo does **not** diagnose, independently establish urgency, choose treatment,
or send clinical advice before clinician authorization. Critical-result flags
from the diagnostic source and configured institutional rules remain
deterministic safety inputs.

### Trigger, input, and output contract

Use only synthetic, de-identified demo records.

| Contract area | Fields |
| --- | --- |
| Trigger | New or amended `DiagnosticReport` event, or synthetic report upload. `caseId = organization + resultId + resultVersion` is the correlation/idempotency key. |
| Identity and ownership | `patientKey`, `encounterId?`, `orderId`, `resultId`, `resultVersion`, `orderingClinicianId`, `coveringClinicianId?`, `careTeamIds[]`. No direct identifier is needed on the Flow canvas. |
| Result | `resultType`, `sourceSystem`, `issuedAt`, `status`, `sourceAbnormalFlag`, `sourceCriticalFlag`, `conclusion`, `observationRefs[]`, `reportFile?`, `amendsResultId?`. |
| Context | `recentRelevantResults[]`, `activeProblems[]`, `followUpOrders[]`, `contactPreferences`, `communicationRestrictions`, `language`, `localPolicyVersion`. Minimize these fields to the approved purpose. |
| Agent assessment | `summary`, `evidenceRefs[]`, `policyRefs[]`, `missingContext[]`, `proposedDisposition`, `proposedFollowUp`, `confidence`, `requiresClinicianReview=true`. |
| Clinician decision | `outcome` (`Accept plan`, `Edit plan`, `Urgent escalation`, `No action with rationale`, `Request context`), `approvedPlan`, `rationale`, `responsibleClinicianId`, `dueAt`. |
| Final output | `caseStatus`, `disposition`, `acknowledgedAt`, `followUpTaskIds[]`, `serviceRequestIds[]`, `patientCommunicationStatus`, `ehrWriteStatus`, `exceptions[]`, and immutable audit references for source, agent, policy, and reviewer versions. |

FHIR is a credible interface shape rather than a guarantee of a target EHR's
capabilities. HL7 defines `ServiceRequest` for ordered services and distinguishes
it from `Task`, which tracks administrative fulfillment.
([HL7 FHIR ServiceRequest](https://hl7.org/fhir/servicerequest.html))

### Reference-solution mapping

| Reference segment | Healthcare-provider canvas segment | Actors and business output | Branch/merge evidence |
| --- | --- | --- | --- |
| Receive and understand | **Receive and normalize result** | FHIR/webhook or upload trigger; API workflow gets structured report; IXP extracts a legacy PDF only when structured content is unavailable. Output is a versioned canonical result. | Duplicate/version check and extraction-confidence exception. |
| Assess and enrich | **Assess follow-up context** | Inline low-code agent summarizes the report and uses approved context grounding for local policy; API workflow retrieves minimal EHR context. A narrowly scoped coded agent prepares an evidence-linked plan object. | Deterministic route on source critical flag, missing ownership, and minimum confidence; agent text alone never suppresses escalation. |
| Decide and review | **Clinician owns the decision** | Coded action app shows read-only evidence and editable plan; ordering or covering clinician accepts, edits, escalates, records no-action rationale, or requests context. | Real expression combines source flags, ownership, confidence, amendment state, and returned human outcome. Timeout escalates to the configured covering pool. |
| Act and communicate | **Close the loop** | In parallel: API workflow writes the EHR task/order, coded agent drafts plain-language communication from the approved plan, and RPA schedules through a UI-only mock. Merge records completion or a recoverable exception. | Symmetric branches merge only after required acknowledgments; patient message remains held if communication restrictions or write-back failure exist. |

This deliberately uses the reference solution's multi-actor contrast. An external
agent is omitted until a real connector and clinical purpose are verified; a
fake connection would weaken the demo. The process-app/Data Fabric variant is
not selected by this research and remains a portfolio decision.

### Integrations and demo mocks

| Dependency | Production intent | Demo implementation | Readiness and fallback |
| --- | --- | --- | --- |
| EHR clinical data | Read `DiagnosticReport`/`Observation`, responsible practitioner and limited context; write `Task`, `ServiceRequest`, and communication/audit references where supported. | Versioned synthetic FHIR JSON served by a mock endpoint. | Vendor, FHIR version, scopes, and write support unverified. API failure enters a human-owned exception queue. |
| Legacy result document | Handle text-based radiology/pathology PDF or scanned result. | Three synthetic PDFs: abnormal, amended, and low-confidence scan; IXP project/model to be created. | No IXP resource verified. Low confidence requires manual reconciliation, never silent completion. |
| Approved follow-up policy | Ground recommendations in institution-approved, versioned content. | Small synthetic policy corpus with explicit effective dates and citations. | No policy owner or Context Grounding index verified. Missing/expired policy forces clinician review without recommendation. |
| Clinician coverage | Resolve ordering clinician, accepted delegate, and escalation pool. | Synthetic roster and on-call schedule. | No identity/roster system verified. Unresolved owner escalates to a staffed pool. |
| Scheduling | Create approved follow-up appointment when no suitable API exists. | Mock scheduling web UI exercised by RPA. | UI automation is justified only for the UI-only path; selector or business error returns to staff with context. |
| Patient portal/message | Send the clinician-approved message and retain delivery status. | Mock connector or outbox; drafts are visible before release. | No live connection verified. Failure does not roll back the clinical task and is separately escalated. |

The local `uip` CLI was inspected at research time (version 1.198.0). It found an
ancestor auth file, but `uip login status` could not acquire the auth-file lock,
so no tenant, folder, connection, IXP model, or Context Grounding resource is
claimed ready.

### Human decisions and safety controls

| Decision/control | Design |
| --- | --- |
| Clinical accountability | The ordering clinician owns follow-up until a covering clinician explicitly accepts transfer, reflecting the SAFER recommendation. The agent cannot acknowledge on the clinician's behalf. |
| Critical and overdue handling | Source-designated critical results bypass normal prioritization. Unacknowledged cases escalate to an alternate clinician; overdue follow-up stays open and visible. |
| Evidence and amendments | Every conclusion links to source spans and policy passages. A new result version reopens the case, marks prior recommendations stale, and requires re-review where material. |
| Patient communication | Only the clinician-approved plan can seed the message. Respect channel/language/restriction fields, record delivery state, and never treat “sent” as patient comprehension. |
| Privacy and security | Apply role-based access, encryption, retention limits, and audit controls. HHS says cloud services that process/store ePHI require a HIPAA-compliant BAA and risk analysis. ([HHS cloud guidance](https://www.hhs.gov/hipaa/for-professionals/special-topics/health-information-technology/cloud-computing/index.html)) HHS's audit protocol calls for mechanisms that record and examine activity in systems using ePHI. ([HHS HIPAA audit protocol](https://www.hhs.gov/hipaa/for-professionals/compliance-enforcement/audit/protocol/index.html)) |
| AI boundary | Treat model output as decision support. Show model/prompt/policy versions, tool calls, evidence, confidence, edits, and final human author. Do not train on case data or send it to an unapproved model/service. |
| Recovery | Idempotent intake; bounded retries for transient calls; no automatic retry of a clinical decision; dead-letter/exception queue with owner, reason, and replay guard. |

### Measurement and evaluation

The SAFER guide explicitly recommends monitoring notification response, time to
acknowledgment, patient follow-up, and results overdue for review. It offers two
business days for ambulatory review and 12 hours for inpatient review as example
measurement points, while noting more urgent results need shorter handling.
These are external guide examples, not adopted demo SLAs; a clinical governance
owner must configure local thresholds.

| Measure/test | Demo evidence |
| --- | --- |
| Intake integrity | Each unique result version creates one case; duplicate delivery is idempotent; an amendment reopens/stales the prior plan. |
| Safety routing | 100% of synthetic source-critical and missing-owner cases enter urgent/manual escalation; no agent confidence can override that route. |
| Acknowledgment | Trace records assigned owner, acknowledgment time, transfer acceptance, and overdue escalation. |
| Follow-up completion | Dashboard/trace distinguishes plan approved, order/task created, patient notified, appointment scheduled, exception open, and fully closed. |
| Agent grounding | Evaluator verifies that every proposed plan cites the supplied result and current policy, and that unsupported clinical claims fail the case. |
| Tool trajectory | Evaluator verifies required context lookup and EHR-write mock calls, while the unavailable-tool case takes the designed fallback. |

Synthetic evaluation set:

1. Abnormal radiology finding with complete context: clinician accepts, all
   required follow-up branches complete, and the case closes.
2. Source-critical result with absent ordering clinician: immediate covering-pool
   escalation; no routine message or scheduling occurs first.
3. Amended report: prior recommendation is stale, the case reopens, and a new
   clinician decision is required.
4. Low-confidence scanned PDF: manual reconciliation occurs before assessment.
5. EHR write failure after approval: the clinical case remains open, patient
   communication is held as configured, and a recoverable exception is audited.

## Unresolved assumptions for human review

1. Which clinical setting and result type is the demo's narrow scope (for
   example, ambulatory radiology incidental findings versus laboratory results)?
2. Who owns clinical governance, defines critical/abnormal rules and local time
   thresholds, approves the policy corpus, and accepts residual model risk?
3. Which EHR, FHIR release/profile, event mechanism, writable resources, patient
   matching rules, and vendor sandbox are available?
4. How are ordering-provider departure, leave, cross-coverage, and explicit
   acceptance represented in the source systems?
5. Which patient-contact restrictions, language/accessibility needs, portal
   release rules, and escalation channels apply? AHRQ notes that the 21st Century
   Cures Act mandates access to electronic health information including test
   results; the workflow must not become a release-delay mechanism.
   ([AHRQ diagnostic safety review](https://www.ahrq.gov/diagnostic-safety/resources/issue-briefs/dxsafety-current-state3.html))
6. Is IXP needed for the chosen source, or can the demo remain structured-FHIR
   first and use one legacy-document exception solely to show document handling?
7. Is healthcare provider one of the three portfolio domains selected for the
   Data Fabric/coded process-app variant?
8. What demo-time SLAs, success thresholds, retention period, audit consumer,
   downtime procedure, and incident owner are approved?
9. Are UiPath services and any model/cloud providers covered by the required
   agreements and approved for the intended synthetic-only demo or later ePHI use?

## Evidence limitations

- Public studies establish workflow risk, not a target provider's volumes,
  failure rate, baseline, or likely savings.
- Several prevalence ranges summarized by AHRQ derive from older studies; they
  support the existence and variability of the problem, not a current national
  point estimate.
- Regulations and federal guidance define obligations and practices but do not
  prove that a particular EHR or payer exposes the needed API operations.
- Resource readiness, clinical policy, and organizational authority require
  validation in the intended UiPath tenant and provider environment before an
  implementation-ready specification can be approved.
