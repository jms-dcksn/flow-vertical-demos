# Pharma agentic-workflow opportunities

## Recommendation

Build a post-market adverse-event intake and ICSR triage demo. A safety case
arrives through an unstructured channel, Flow extracts the report, checks the
minimum case criteria and potential duplicates, proposes seriousness and
reportability, pauses for a pharmacovigilance reviewer, and then coordinates
the approved write-back, follow-up request, and regulatory-format output.

The visible hero moment is the reviewer correcting or approving an agent's
evidence-linked recommendation while the regulatory clock remains visible.
This is a credible use of agents because language understanding helps assemble
and explain a case, while deterministic rules and accountable people retain
control of medical and regulatory decisions.

This is a public-evidence quick scan completed on August 10, 2026, not a company
automation audit. It uses regulator, standards-body, and government registry
sources rather than vendor ROI claims. No
pharmaceutical company's internal volumes, cycle times, error rates, ROI,
existing automations, or UiPath resource readiness were available. The ranking
is therefore a comparative demo recommendation, not a quantified business
case. No pack-hours or delivery estimate is supplied.

## Evidence base

### Post-market safety case processing

- EudraVigilance received more than 1.7 million adverse-drug-reaction reports
  in 2024; 63% originated outside the EEA. EMA also reviewed 1,254 potential
  safety signals, 74% of which originated from EudraVigilance monitoring. This
  establishes material case scale and cross-border coordination, not the
  workload of any one company. [EMA Annual Report 2024: human medicines](https://www.ema.europa.eu/assets/en/annual-report/2024/human-medicines/index.html)
- FDA requires serious and unexpected post-market adverse drug experiences to
  be reported as soon as possible and no later than 15 calendar days after
  initial receipt; follow-up reports have their own 15-day clock. The same
  source requires written surveillance, receipt, evaluation, and reporting
  procedures. [FDA expedited safety reporting requirements](https://www.fda.gov/science-research/clinical-trials-and-human-subject-protection/expedited-safety-reporting-requirements-human-drug-and-biological-products)
- EMA's GVP Module VI says a valid ICSR needs an identifiable reporter, one
  identifiable patient, a suspected product, and a suspected adverse reaction.
  An incomplete report must be retained, followed up with due diligence, and
  have those attempts documented. It also covers reports from literature,
  medical-information services, company-controlled digital media, patient
  programmes, and other sources. [EMA GVP Module VI, sections VI.B.1–VI.B.3](https://www.ema.europa.eu/en/documents/regulatory-procedural-guideline/guideline-good-pharmacovigilance-practices-gvp-module-vi-collection-management-submission-reports-suspected-adverse-reactions-medicinal-products-rev-2_en.pdf)
- Duplicate handling is a real business workflow rather than a simple lookup:
  the EMA process calls for searching at data entry, validating a potential
  match, determining whether new information exists, updating a master case,
  and deciding whether a follow-up report is warranted. [EMA GVP Module VI Addendum I](https://www.ema.europa.eu/en/documents/regulatory-procedural-guideline/guideline-good-pharmacovigilance-practices-gvp-module-vi-addendum-i-duplicate-management-suspected-adverse-reaction-reports_en.pdf)
- FDA accepts post-market ICSRs in ICH E2B(R3), requires E2B(R3) for IND safety
  reports as of April 1, 2026, and says post-market submissions through ESG
  NextGen must use E2B(R3) beginning October 1, 2026. FDA provides regional
  business rules and a validator, making a structured payload and validation
  response credible demo outputs. [FDA AEMS electronic submissions](https://www.fda.gov/drugs/fda-adverse-event-monitoring-system-aems/fda-adverse-event-monitoring-system-aems-electronic-submissions)
- The July 2025 EU masking guidance identifies 13 ICSR fields that senders
  should always mask and 11 that should be left blank, while preserving other
  pseudonymised fields needed for processing, duplicate detection, and signal
  management. [EMA GVP Module VI Addendum II](https://www.ema.europa.eu/en/documents/regulatory-procedural-guideline/guideline-good-pharmacovigilance-practices-gvp-module-vi-addendum-ii-masking-personal-data-individual-case-safety-reports-submitted-eudravigilance_en.pdf)

### Clinical-trial oversight

- ClinicalTrials.gov listed 593,857 studies on July 14, 2026, including 65,004
  recruiting studies and 219,299 registered studies involving a drug or
  biologic. These are ecosystem-scale figures and do not represent a sponsor's
  active workload. [ClinicalTrials.gov trends and charts](https://clinicaltrials.gov/about-site/trends-charts)
- ICH E6(R3), adopted by FDA in September 2025, says sponsors should define
  trial-specific criteria for important protocol deviations and use timely,
  proportionate action, root-cause analysis, CAPA, and documented escalation
  when noncompliance may affect participant safety or reliable results.
  [FDA E6(R3) Good Clinical Practice guidance](https://www.fda.gov/media/169090/download?attachment=)
- FDA's December 2024 protocol-deviation guidance says consistent
  classification, reporting, and documentation are important, but it remains
  draft guidance and should not be presented as a binding rule.
  [FDA draft protocol-deviation guidance](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/protocol-deviations-clinical-investigations-drugs-biological-products-and-devices)

### Manufacturing quality and supply continuity

- FDA says current good manufacturing practice includes detecting and
  investigating product-quality deviations. It also notes that a manufacturer
  may test only a sample, such as 100 tablets from a two-million-tablet batch,
  so testing alone cannot assure the whole batch's quality.
  [FDA facts about CGMP](https://www.fda.gov/drugs/pharmaceutical-quality-resources/facts-about-current-good-manufacturing-practice-cgmp)
- ICH Q9(R1) says quality-risk decisions should be based on scientific
  knowledge, link ultimately to patient protection, and use effort, formality,
  and documentation proportionate to risk. It warns that digitalisation can
  reduce risk but can also introduce risks that need control.
  [ICH Q9(R1) Quality Risk Management](https://database.ich.org/sites/default/files/ICH_Q9%28R1%29_Guideline_Step4_2023_0126_0.pdf)
- FDA reports that more than 90% of inspected pharmaceutical facilities have
  acceptable CGMP compliance; when a Form FDA 483 is issued, a firm generally
  has 15 business days to respond. This supports risk-based exception handling
  rather than treating every quality event as equally severe.
  [FDA pharmaceutical inspections and compliance](https://www.fda.gov/drugs/guidance-compliance-regulatory-information/pharmaceutical-inspections-and-compliance)
- FDA and manufacturers prevented 283 drug shortages in 2024 and identified 15
  new shortages. FDA describes earlier manufacturer notification as giving it
  more time to preserve treatment options, and separately identifies
  manufacturing quality issues as the most common reason for shortages.
  [FDA 2024 Drug Shortages Report to Congress](https://www.fda.gov/media/189325/download?attachment=),
  [FDA drug-shortage FAQ](https://www.fda.gov/drugs/drug-shortages/frequently-asked-questions-about-drug-shortages)

## Candidate ranking

Scores are comparative judgement on a 1–5 scale: `5` is strongest. They are
not measured operational performance. Demo value rewards a legible multi-actor
story and human decision; enterprise credibility rewards regulatory evidence
and a clear owner; feasibility rewards bounded synthetic data and mockable
systems. Equal weighting avoids implying an unsupported economic model.

| Rank | Candidate and actors | Evidence-backed workflow and measurable proof | Demo value | Enterprise credibility | Feasibility | Total |
| --- | --- | --- | --- | ---: | ---: | ---: | ---: |
| 1 | Post-market adverse-event intake and ICSR triage; intake specialist, drug-safety associate, medical reviewer, regulatory operations | Multichannel report -> minimum-criteria validation -> duplicate/follow-up -> seriousness and expectedness recommendation -> human release -> E2B(R3) validation. Measure intake-to-valid-case time, deadline adherence, missing-field follow-up, duplicate confirmation, validator rejection, and reviewer override. The [1.7 million EU reports](https://www.ema.europa.eu/assets/en/annual-report/2024/human-medicines/index.html) and [15-day FDA clock](https://www.fda.gov/science-research/clinical-trials-and-human-subject-protection/expedited-safety-reporting-requirements-human-drug-and-biological-products) establish scale and consequence. | 5 | 5 | 4 | 14 |
| 2 | Clinical-trial protocol-deviation triage; site coordinator, CRA, sponsor quality lead, medical monitor, IRB/regulatory staff | Site report -> protocol/version grounding -> important-deviation recommendation -> safety/data-integrity review -> CAPA and notifications. Measure time to classification/escalation, backlog age, reclassification rate, CAPA closure, and repeat deviation. The [65,004 recruiting studies](https://clinicaltrials.gov/about-site/trends-charts) show ecosystem scale; [E6(R3)](https://www.fda.gov/media/169090/download?attachment=) supplies the risk and escalation model. | 5 | 5 | 3 | 13 |
| 3 | Manufacturing deviation and CAPA evidence pack; operator, investigator, quality unit, batch disposition authority | Deviation/equipment/log intake -> batch and SOP context -> risk hypothesis -> human investigation scope -> parallel CAPA and batch-impact work. Measure triage lead time, overdue investigations, right-first-time evidence completeness, CAPA recurrence, and disposition time. FDA reports [more than 90% of inspections found acceptable CGMP compliance](https://www.fda.gov/drugs/guidance-compliance-regulatory-information/pharmaceutical-inspections-and-compliance), supporting exception-based triage, but representative MES/LIMS data are harder to mock well. | 4 | 5 | 3 | 12 |
| 4 | Drug-shortage early-warning and response; supply planner, site quality, regulatory affairs, medical affairs, shortage authority | Supply/quality signal -> affected-product and patient-risk assessment -> human response decision -> parallel authority notification and allocation communication. Measure alert-to-assessment time, notification timeliness/completeness, response latency, and mitigated supply gap. FDA's [283 prevented and 15 new shortages in 2024](https://www.fda.gov/media/189325/download?attachment=) support the value of coordinated early action. | 4 | 4 | 3 | 11 |

### Why the first candidate wins

The ICSR workflow has the best balance of document intelligence, agentic
interpretation, deterministic rules, duplicate retrieval, an accountable human
decision, standardized output, parallel follow-up, and an auditable deadline.
Its sources define the process at field and decision level. The other candidates
are credible, but realistic manufacturing and supply-chain demos need more
site-specific master data, and a clinical-deviation demo needs a trial protocol
and sponsor-specific classification scheme before its reasoning can be tested.

No proven internal automation was available to replicate. This is a greenfield
recommendation and must not be described as a Tier 1 replicable model or as an
ROI claim.

## Strongest candidate: demo contract

### Scope and narrative

The demo covers one synthetic post-market human-drug report from receipt through
approved case disposition. Signal detection, aggregate benefit-risk assessment,
clinical-trial SUSAR processing, and real submission to a regulator are out of
scope. The target user is a pharmacovigilance operations team; the consequential
decision belongs to an authorised medical/safety reviewer.

### Input contract

| Field | Type | Purpose and validation |
| --- | --- | --- |
| `intakeId` | string, required | Immutable correlation and idempotency key. |
| `receivedAtUtc` | timestamp, required | Preserved source receipt time; proposed regulatory day zero must be reviewer-confirmed. |
| `sourceChannel` | enum | `email`, `web_form`, `call_transcript`, or `literature`. |
| `sourceMarket` | string, required | Drives jurisdictional rules; demo supports configured US and EU scenarios only. |
| `sourceDocument` | file, required | Synthetic email, form, transcript, or article plus attachments; original is retained read-only. |
| `reporter` | object | Qualification, country, contactability, and consent/contact restrictions. Direct identifiers stay in the protected source zone. |
| `patient` | object | At least one qualifying descriptor; synthetic initials/age/sex only in the demo. |
| `suspectProducts[]` | array | Product, strength, dose, route, dates, indication, and batch when relevant. |
| `reactions[]` | array | Verbatim reaction, onset, outcome, and proposed controlled terminology. |
| `caseContext` | object | Concomitant products, history, tests, pregnancy, medication-error, literature, and prior case identifiers when supplied. |

IXP produces extracted fields with source spans and field confidence. Extraction
confidence controls document-review routing only; it must never be treated as
medical confidence or regulatory validity.

### Output contract

| Field | Type | Meaning |
| --- | --- | --- |
| `caseId` / `intakeId` | strings | Safety-system ID and original correlation key. |
| `validity` | object | Presence of the four minimum elements, missing fields, evidence spans, and reviewer status. |
| `duplicateAssessment` | object | `none`, `potential`, or `confirmed`; candidate IDs, match reasons, master case, and human confirmation. |
| `medicalAssessment` | object | Proposed seriousness criteria, expectedness, listedness source/version, causality notes, rationale, and reviewer corrections. |
| `reportingPlan` | object | Jurisdiction, report type, proposed day zero, due date, due-date rule version, and `submit`, `follow_up`, `retain_only`, or `escalate`. |
| `followUpPlan` | object | Missing clinically significant information, reporter-contact restriction, questions, attempts, and outcome. |
| `humanDecision` | object | Reviewer ID/role, `approve`, `correct`, `request_information`, or `escalate`, reason, timestamp, and changed fields. |
| `regulatoryArtifact` | object | Mock E2B(R3) XML location, validator status/errors, masking profile, and release status; never reports a real submission in the demo. |
| `auditSummary` | object | Model/prompt/rule versions, tool calls, sources, state transitions, exceptions, and timestamps. |

### Flow topology and reference mapping

| Reference segment | Pharma canvas title | Actors and output | Visible route |
| --- | --- | --- | --- |
| Receive and understand | `1. Receive safety report` | Email/form trigger, API intake workflow, IXP extraction, immutable source capture | Missing document or unreadable extraction goes below the happy path to intake repair. |
| Assess and enrich | `2. Validate and assess case` | Inline triage agent uses approved product-label context and a duplicate-search tool; coded narrative-quality agent checks evidence coverage; API product lookup and, only if required, RPA safety-system lookup return structured assessments | `minimumElementsComplete && extractionReviewComplete` controls case creation; potential duplicates require confirmation. |
| Decide and review | `3. Review reportability` | Coded action app shows source evidence, clock, duplicate candidate, and proposed medical/reporting assessment to an authorised reviewer | Named outcomes are `Approve`, `Correct`, `Request information`, and `Escalate`; no autonomous regulatory release. |
| Act and communicate | `4. Finalise and follow up` | In parallel, create/validate mock E2B(R3) output and draft a targeted reporter follow-up; write the approved state to the mock safety system; merge before completion | Validator failure, contact restriction, or write-back failure enters a recoverable exception route. |

The design deliberately puts an API workflow and an RPA activity on the intended
path: the API workflow normalises inbound reports and product data; RPA performs
a read-only duplicate lookup in a mock legacy safety UI when no supported API is
available. If the selected safety system has an adequate API, remove RPA rather
than automate its UI merely to satisfy a checklist.

### Integrations and demo substitutes

| Dependency | Intended responsibility | Demo implementation and readiness gap |
| --- | --- | --- |
| Outlook or monitored intake mailbox | Receive report and attachments with original timestamp/message ID | Use an Integration Service connection only after tenant verification; otherwise use a manual trigger with exported synthetic messages. |
| IXP | Extract the safety form, narrative, and attachments with evidence spans | Train/deploy a small synthetic document model in the target folder; project and connection are not yet verified. |
| Product/label repository | Ground product identity and expectedness/listedness evidence | Use versioned synthetic product and label records in a context index. A real regulatory information-management source is not selected. |
| Safety database | Search potential duplicates and persist an approved case | Mock Oracle Argus/Veeva-style API and UI contracts. Vendor APIs, credentials, and licences are not verified. |
| Controlled terminology | Suggest reaction and product codes | Use a clearly labelled synthetic subset. MedDRA access and licensing are unresolved; do not bundle licensed content. |
| Coded action app / Action Center | Present evidence and capture accountable reviewer changes/outcome | Contract is specified above; deployment and returned field IDs require implementation verification. |
| E2B(R3) validator and gateway | Validate structure and return acknowledgement/error | Generate synthetic XML and mock acknowledgement/error responses. Real FDA ESG NextGen or EudraVigilance transmission is explicitly disabled. |
| Notification channel | Send targeted follow-up after reviewer approval | Use a test mailbox; redact sensitive fields from logs and notification metadata. |

No external agent is required for the first build. Add one only when a real,
approved connector and a distinct responsibility can be demonstrated; never
ship a placeholder connection ID.

### Human decisions

- A pharmacovigilance intake specialist confirms poor extraction and reporter
  contact restrictions before the case enters medical assessment.
- An authorised drug-safety physician or designated medical reviewer owns
  seriousness, expectedness/listedness, causality commentary, report type, day
  zero, due date, duplicate disposition, and regulatory release. The agent may
  recommend and cite evidence but cannot make or submit these decisions.
- The review app shows the immutable source, extracted evidence spans, missing
  minimum elements, rule version, deadline, duplicate candidates, label excerpt,
  and all proposed fields. Corrections require a rationale.
- `Request information` records targeted questions and waits; `Escalate` routes
  to the configured safety lead. A due-date threshold must also notify the
  operational owner rather than waiting indefinitely for a task timeout.

### Risk and compliance controls

| Risk | Required control and demo evidence |
| --- | --- |
| Missed or incorrect regulatory clock | Preserve source receipt time, separate system-calculated and reviewer-confirmed day zero, version jurisdiction rules, show due date/SLA, and escalate ageing work. |
| Hallucinated medical or regulatory judgement | Constrain agents to extraction, recommendation, and cited rationale; use deterministic validity/deadline rules; require authorised review before release. |
| False duplicate merge | Retrieval only proposes candidates; reviewer confirms the master case; retain both source records and merge rationale. |
| Incomplete report | Evaluate the four minimum elements explicitly, retain incomplete reports, generate targeted follow-up, and record unsuccessful attempts. |
| Personal or health-data exposure | Use synthetic data; segregate original identifiers; apply role-based least privilege, encryption, redacted logs, retention policy, and jurisdiction-specific masking. Test the 2025 EU field-masking profile. |
| Terminology or label drift | Pin terminology and label versions in every assessment; route missing/outdated context to review; do not imply access to licensed terminology. |
| Silent tool/model failure | Time out tools, validate structured output, reject unknown enums, preserve tool/model versions and evidence, and route failures to a recoverable queue. |
| Unapproved external submission | Hard-disable production gateway credentials, watermark artifacts `DEMO — NOT SUBMITTED`, and return synthetic acknowledgements only. |

### Evaluation and measurable outcomes

Use a synthetic five-case set: complete non-serious listed case, serious and
unexpected expedited case, missing reporter/patient detail requiring follow-up,
potential duplicate with new information, and a case with a validator or lookup
failure. Evaluation should measure:

- exact minimum-element and route agreement with an approved expected result;
- recall of serious-case escalation, with any false negative failing the demo;
- exact due-date calculation for the configured synthetic rule set;
- duplicate recommendation and human-confirmation contract, never autonomous
  merge;
- presence of source evidence for every proposed medical field;
- expected tool calls and safe fallback when a dependency fails;
- masked-field and synthetic E2B(R3) validation results; and
- intake-to-review-ready time, reviewer edits, follow-up completeness, backlog
  age, and validator rework as observed demo measures, without inventing target
  improvements.

## Unresolved assumptions for human review

1. Choose the regulatory scope: US post-market, EU post-market, or a deliberately
   limited dual-jurisdiction ruleset. The rules, clocks, masking, and evaluation
   oracle depend on this choice.
2. Confirm which role is authorised to decide seriousness, expectedness,
   causality, duplicate disposition, day zero, and release in the target
   organisation; titles and delegation vary.
3. Select the safety database and verify whether its API supports source
   attachment storage, duplicate search, case creation, and audit history. Use
   RPA only for a proven UI-only gap.
4. Confirm licensed terminology access and permitted demo distribution. Until
   then, use synthetic terms and avoid representing them as MedDRA-coded output.
5. Decide whether the hero report is email, call transcript, literature, or web
   form. Each source has different day-zero, reporter-identifiability, and
   attachment behaviour.
6. Verify the target UiPath folder, IXP deployment, mailbox connection, context
   index, action-app contract, model availability, and any safety-system
   connection before drafting an implementation spec.
7. Obtain privacy, security, pharmacovigilance quality, and validation approval
   for the proposed data path, retention, masking profile, audit trail, and use
   of generative AI in a regulated process.
8. Decide whether this domain will be one of the three Data Fabric/process-app
   variants. If selected, the safety case must have one canonical record and
   the review app must not compete with the validated safety database as the
   source of truth.
9. Establish organisation-specific baselines and targets for cycle time,
   timeliness, completeness, rework, duplicate rate, reviewer overrides, and
   backlog age before making ROI or capacity claims.

## Next step

After the first six assumptions are resolved, copy the repository's domain demo
specification template and turn this contract into one independently deployable
solution under `pharma/<demo>/<demo>-solution/`. Keep all submission endpoints
mocked until regulated-system owners approve and verify a test connection.
