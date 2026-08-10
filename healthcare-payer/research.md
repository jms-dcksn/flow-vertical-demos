# Healthcare-payer agentic workflow research

## Scope and method

This is a public-industry research scan, not an assessment of one payer's
operations. It uses current government evidence available on August 10, 2026,
to identify workflows where a UiPath Maestro Flow demo could show coordinated
document work, bounded agentic reasoning, deterministic controls, human
decisions, and system action.

The automation-discovery method was adapted because no internal systems or
behavioral data were authorized or available. Consequently:

- no existing automation is treated as a proven replicable model;
- volumes and rates below are source facts, not estimates for a prospective
  customer;
- candidate scores are comparative design judgments, not ROI predictions; and
- proposed success measures are pilot metrics that need a payer baseline.

## Evidence-backed opportunity landscape

### Prior authorization and appeals

Beginning in 2026, the CMS Interoperability and Prior Authorization Final Rule
requires impacted payers, except QHP issuers on the FFEs for the timeframe
provision, to send decisions within 72 hours for expedited requests and seven
calendar days for standard requests. It also requires a specific denial reason
and annual public metrics such as approval, denial, approval-after-appeal, and
decision-time measures. The API requirements generally begin in 2027
([CMS final-rule fact sheet](https://www.cms.gov/newsroom/fact-sheets/cms-interoperability-prior-authorization-final-rule-cms-0057-f),
[CMS Prior Authorization API FAQ](https://www.cms.gov/initiatives/burden-reduction/overview/interoperability/frequently-asked-questions/prior-authorization-api)).

The quality risk is material. In a June 2024 sample covering 19 Medicare
Advantage organizations (MAOs), HHS OIG found that 12% of skilled-nursing
facility (SNF) admission requests were denied. Only 18% of those denials were
appealed, but MAOs overturned 95% of appealed denials. Denial rates varied from
0.4% to 23%, and requests for nursing-home residents were denied 40% of the time
versus 11% for other enrollees
([HHS OIG, 2026](https://oig.hhs.gov/reports/all/2026/medicare-advantage-organizations-overturned-nearly-all-appealed-prior-authorization-denials-for-skilled-nursing-facility-admission-raising-concerns-about-initial-denials/)).
An earlier OIG review found that 13% of sampled denied prior-authorization
requests met Medicare coverage rules and that some denials resulted from
missing-document assertions even when the medical record was sufficient
([HHS OIG, 2022](https://oig.hhs.gov/reports/all/2022/some-medicare-advantage-organization-denials-of-prior-authorization-requests-raise-concerns-about-beneficiary-access-to-medically-necessary-care/)).

Affected actors include utilization-management nurses, medical directors,
provider authorization staff, members and representatives, appeals teams, and
compliance leaders. Typical systems include EHR/provider portals, utilization-
management platforms, coverage-policy repositories, document stores, member
and provider master data, and correspondence systems.

### Claims-payment denial correction

The same 2022 OIG review found that 18% of sampled payment requests denied by
MAOs met both Medicare coverage and MAO billing rules. OIG attributed most of
those erroneous denials to human error during manual review, such as overlooking
a document, and system-processing errors
([HHS OIG, 2022](https://oig.hhs.gov/reports/all/2022/some-medicare-advantage-organization-denials-of-prior-authorization-requests-raise-concerns-about-beneficiary-access-to-medically-necessary-care/)).
This is evidence for a document-completeness and rules-reconciliation workflow,
not evidence that every payer has the same error rate.

Affected actors include claims examiners, payment-integrity analysts, provider
dispute staff, configuration teams, and compliance reviewers. Typical systems
include claims-adjudication platforms, document imaging, contract and policy
repositories, provider data, and payment/correspondence systems.

### No Surprises Act payment disputes

The Federal Independent Dispute Resolution (IDR) process resolves eligible
out-of-network payment disputes after unsuccessful open negotiation. Through
July 2025, 3,743,767 disputes had been initiated and 3,344,616 closed; 630,428
closures were ineligibility determinations
([CMS IDR reports](https://www.cms.gov/nosurprises/policies-and-resources/Reports)).
In the first half of 2024, non-initiating parties challenged eligibility for
45% of initiated disputes, and 18% of closed disputes were found ineligible.
CMS described eligibility determination as a primary cause of delay. Providers
or facilities prevailed in about 84% of payment determinations during that
period
([CMS supplemental background](https://www.cms.gov/files/document/supplemental-background-federal-idr-puf-january-1-june-30-2024-march-18-2025.pdf)).

Affected actors include payer IDR analysts, legal/compliance staff, network and
pricing teams, providers or their representatives, and certified IDR entities.
Typical systems include claims, provider/network data, qualifying-payment-
amount calculations, negotiation correspondence, the Federal IDR portal, and
evidence repositories.

### Risk-adjustment evidence validation

CMS reported a fiscal-year 2025 Medicare Part C improper-payment rate of 6.09%,
or $23.67 billion, and stated that most Part C improper payments involved
supporting documentation that did not substantiate submitted diagnosis data
([CMS FY 2025 fact sheet](https://www.cms.gov/newsroom/fact-sheets/fiscal-year-2025-improper-payments-fact-sheet)).
CMS describes Risk Adjustment Data Validation (RADV) as its primary mechanism
for addressing MAO overpayments and confirms sampled diagnoses against medical
records
([CMS RADV program](https://www.cms.gov/data-research/monitoring-programs/medicare-risk-adjustment-data-validation-program)).

Affected actors include risk-adjustment coders, medical-record retrieval teams,
compliance and audit staff, clinicians, and CMS audit liaisons. Typical systems
include encounter and diagnosis data, EHR or chart-retrieval platforms, coding
tools, audit workspaces, and CMS submission systems.

### Provider-directory remediation

GAO reviewed a generalizable sample of 342 behavioral-health listings in the
TRICARE network directories and estimated that most listings contained an error
in at least one item, such as location or phone number. The two contractors'
reported accuracy for their overall directories was about 82%. The directories
contained nearly 130,000 behavioral-health listings and more than one million
provider listings overall
([GAO-24-106588](https://www.gao.gov/products/gao-24-106588)).
This evidence is specific to TRICARE and should not be generalized as an error
rate for commercial or Medicare Advantage plans.

Affected actors include provider-data stewards, network managers, credentialing
teams, provider offices, member services, and compliance staff. Typical systems
include credentialing, contracting, provider master data, NPPES or other
reference sources, plan directories, and outreach channels.

## Ranked demo candidates

Each dimension is scored from 1 (weak) to 5 (strong). **Demo value** measures
how clearly Flow can show multi-actor orchestration and a consequential hero
moment. **Enterprise credibility** reflects the strength and currency of public
evidence. **Feasibility** reflects whether a persuasive synthetic demo can use
bounded mocks without pretending to make a live regulated decision.

| Rank | Candidate | Demo value | Enterprise credibility | Feasibility | Total / 15 | Why it ranks here |
| --- | --- | ---: | ---: | ---: | ---: | --- |
| 1 | Post-acute SNF prior-authorization decision quality and appeal prevention | 5 | 5 | 4 | 14 | Recent OIG evidence, explicit decision deadlines, document-heavy clinical evidence, real human authority, and visible approve/request-information/adverse routes make the orchestration legible. |
| 2 | No Surprises Act IDR eligibility and response assembly | 5 | 5 | 4 | 14 | High documented volume and complex eligibility create an excellent evidence-assembly story; the legal/pricing rules require careful scoping and a human-submitted portal step. |
| 3 | Claims-payment denial correction before provider notification | 4 | 5 | 4 | 13 | OIG directly identifies overlooked documents and processing errors. The story is credible but less visibly agentic than a clinical review unless the demo emphasizes root-cause reconciliation. |
| 4 | Medicare Advantage RADV evidence-package validation | 4 | 5 | 3 | 12 | Strong financial and audit evidence with rich document work, but coding specificity, long audit cycles, and specialized source data make a faithful demo harder. |
| 5 | Provider-directory discrepancy remediation | 3 | 4 | 4 | 11 | Easy to mock and well suited to parallel outreach, but lower decision consequence and TRICARE-specific evidence make it a weaker flagship. |

There is no Tier 1 replication finding because this public scan found no
organization-specific working automation to replicate. The top-ranked candidate
is therefore the greenfield headline. This ranking is an inference from the
cited evidence and the repository's reference-demo rubric.

The minimum control boundary and pilot measure for each candidate are:

| Candidate | Control boundary | Proposed measure, requiring a local baseline |
| --- | --- | --- |
| SNF authorization | Agent organizes and cites evidence; a qualified clinician owns every adverse or reduced decision. | Decision timeliness, additional-information rate, adverse rate, and appeal-overturn rate. |
| No Surprises Act IDR | Deterministic eligibility rules and provenance checks precede legal/pricing review; a human releases any portal submission. | Eligibility-challenge rate, ineligible-closure rate, submission completeness, and cycle time. |
| Payment-denial correction | Agent flags contradictions or missing evidence; a claims examiner approves reprocessing and system configuration changes remain separate. | Pre-notice correction rate, examiner edit rate, repeat-error rate, and recovery cycle time. |
| RADV evidence package | Agent may link diagnosis claims to chart evidence but cannot create diagnosis support; a certified coder or auditor attests the package. | Unsupported-diagnosis catch rate, citation precision, package completeness, and audit rework. |
| Provider directory | Conflicts remain pending until provider-data stewardship resolves identity and network status; no autonomous provider termination. | Verified-field rate, outreach response time, stale-listing rate, and repeat discrepancy rate. |

## Recommended demo: post-acute SNF authorization quality

### Narrative and hero moment

A hospital submits an expedited SNF admission request with a discharge summary,
therapy evaluation, medication list, and supporting clinical notes. Flow creates
one correlated case, extracts and checks the packet, gathers the applicable
coverage evidence, and asks a bounded agent to produce a structured evidence
map. Deterministic logic can route a complete, policy-supported case toward
approval, but any proposed denial, conflicting evidence, low confidence, or
high-risk condition goes to a utilization-management clinician.

The hero moment is the clinician seeing the requested service, source-linked
clinical facts, applicable policy passages, missing or contradictory evidence,
and a recommendation in one review experience. The clinician—not the agent—owns
the consequential decision. Flow then performs independent write-back and
communication work, merges the branches, and records an audit-ready case
summary.

This design aims to prevent avoidable adverse decisions at initial review. It
does not predict the 95% appeal-overturn finding will apply to a particular
payer, and it does not promise a reduction without a measured pilot.

### Input and output contract

All demo inputs are synthetic. The schema deliberately separates identifiers,
clinical evidence, policy evidence, and orchestration state.

| Contract area | Fields |
| --- | --- |
| Trigger | `requestId`, `correlationId`, `receivedAt`, `urgency` (`expedited` or `standard`), `requestedService` (`SNF admission`), `requestedStartDate`, `requestedDays` |
| Member and coverage | synthetic `memberId`, plan/product, coverage-effective dates, transition-of-care flag, prior authorization history token |
| Provider and facility | synthetic requesting-provider ID, hospital ID, proposed-SNF ID, network status, contact endpoint |
| Clinical packet | discharge summary, therapy evaluation, current functional status, diagnoses, medications, prior level of function, requested level of care, attached-note manifest |
| Policy context | policy version and effective date, applicable Medicare coverage source IDs, plan criteria only where permitted, provenance URL or repository key |
| Orchestration state | extraction confidence, missing-document list, evidence conflicts, SLA due time, assigned reviewer, retry count |

The Flow returns one structured result:

```json
{
  "requestId": "PA-DEMO-1007",
  "status": "approved | information_requested | denied | escalated",
  "authorizedDays": 7,
  "decisionReasonCodes": ["POLICY_CRITERIA_MET"],
  "specificReason": "Synthetic demo rationale with evidence references",
  "evidenceReferences": ["therapy-eval:p2", "policy-v3:section-4.2"],
  "missingInformation": [],
  "reviewerOutcome": "Approve",
  "reviewerRationale": "Synthetic clinician rationale",
  "decisionAt": "2026-08-10T14:00:00Z",
  "slaState": "within_target",
  "communicationIds": ["provider-notice-1007"],
  "auditRecordId": "audit-1007"
}
```

`authorizedDays` is nullable unless approved. `specificReason` and evidence
references are required for a denial. No agent may populate `reviewerOutcome`
or `reviewerRationale`.

### Integrations and demo mocks

The repository reference proves the actor breadth and topology, but this
research did not validate live healthcare connections. Use explicit mocks until
owners verify resources.

| Responsibility | Proposed actor | Demo dependency or mock |
| --- | --- | --- |
| Receive request | API workflow | Local synthetic JSON endpoint shaped like a prior-authorization request; do not claim certification or conformance to a production FHIR implementation. |
| Classify and extract packet | IXP/document intelligence | Synthetic PDF packet and a demo extraction model; confidence and missing-field outputs drive routing. |
| Verify coverage/provider facts | API workflow | Mock member, benefit, and network endpoints with fixed fixtures and failure cases. |
| Retrieve policy | Context-grounding index/tool | Curated, versioned public CMS excerpts plus synthetic plan policy; every returned passage includes provenance and effective date. |
| Build evidence map | Inline low-code agent | Tool access limited to the case packet and approved policy corpus; JSON-schema output, no free-form decision field. |
| Enter authorization | RPA | Mock legacy utilization-management desktop with no API; returns transaction ID or typed exception. This is the only UI-automation justification. |
| Review case | Action Center coded action app, or quick form if necessary | Synthetic clinician queue; no real member data. |
| Draft specific notice | Narrow coded agent | Generates plain-language draft only from the final human outcome, reason codes, and cited evidence; deterministic template validation follows. |
| Notify and archive | API workflow plus mock correspondence/document store | Independent provider-notice and audit-write branches, then merge. |

No external agent is required for the first demo. Adding one solely to match the
reference would weaken feasibility unless a real, approved connection and a
non-duplicative responsibility are available.

### Human decisions

The primary reviewer is a utilization-management nurse; a medical director is
the escalation authority for adverse or clinically ambiguous cases. The review
experience shows read-only case facts, extracted-source snippets, policy
passages and versions, evidence conflicts, the agent's structured summary, and
the SLA clock. Editable fields are approved level of care, authorized days,
reason code, specific rationale, and requested information.

Named outcomes are:

- `Approve`: continue to authorization write-back and notice;
- `Modify`: require medical-director confirmation before a reduced scope or
  duration is written;
- `Request information`: send a specific request and wait or safely expire;
- `Deny`: require medical-director decision and a validated specific reason;
- `Escalate`: transfer ownership without issuing a decision.

The task must time out before the regulatory deadline, not at it. Timeout routes
to an operational escalation queue. Exact buffer, reviewer qualifications, and
state- or product-specific rules require owner approval.

### Reference-solution mapping

| Reference segment | Demo segment | Actors and output | Visible route or merge |
| --- | --- | --- | --- |
| Receive and understand | Receive SNF request | API intake, packet classification, IXP extraction; canonical case and confidence | Invalid packet or low confidence goes to information/review path. |
| Assess and enrich | Build clinical evidence map | Coverage/provider API, policy context tool, bounded inline agent; cited facts, conflicts, and completeness | Structured `evidenceState` and `adverseCandidate` fields drive routing. |
| Decide and review | Make accountable determination | Real decision expression, nurse action app, medical-director escalation; final outcome and rationale | Any denial, reduction, conflict, or low confidence requires human review; safe exception below happy path. |
| Act and communicate | Authorize, notify, and record | RPA write-back and coded-agent notice draft run as independent branches; deterministic validation and audit write | Branches merge before case completion; failure creates an owned recovery item. |

The canvas should use four named sticky notes matching the demo segments, keep
the happy path left to right, and place missing-information, dependency-failure,
and escalation routes below it. A practical branch expression is based on
structured fields such as `packetComplete`, `extractionConfidence`,
`policyConflict`, and `adverseCandidate`, never a literal constant.

### Controls, risks, and compliance

| Risk | Required demo control |
| --- | --- |
| Inappropriate denial or delay | No autonomous denial or reduction. Require qualified human review, surface the SLA clock, escalate before expiry, and preserve all evidence and edits. CMS requires specified decision timeframes and specific denial reasons for impacted payers ([CMS](https://www.cms.gov/newsroom/fact-sheets/cms-interoperability-prior-authorization-final-rule-cms-0057-f)). |
| Stale or inapplicable coverage criteria | Version and date every policy source; retrieve Medicare NCD/LCD or other applicable criteria first; block the decision when sources conflict. CMS limits when MA plans may use internal criteria and requires annual utilization-management committee review ([CMS-4201-F](https://www.cms.gov/newsroom/fact-sheets/2024-medicare-advantage-and-part-d-final-rule-cms-4201-f)). |
| Hallucinated clinical fact or rationale | Agent output must cite packet page and policy passage IDs. Schema validation rejects uncited assertions; reviewers can open the source. The agent recommends evidence organization, not medical necessity. |
| PHI exposure | Synthetic data only in the demo. For any future pilot, apply role-based access, encryption, retention limits, business-associate review, and minimum-necessary policies. HHS states that health plans are HIPAA covered entities and must generally limit PHI use or disclosure to the minimum necessary ([HHS covered entities](https://www.hhs.gov/hipaa/for-professionals/faq/190/who-must-comply-with-hipaa-privacy-standards/index.html), [HHS minimum necessary guidance](https://www.hhs.gov/hipaa/for-professionals/privacy/guidance/minimum-necessary-requirement/index.html)). |
| Prompt injection or malicious attachment | Treat packet content as untrusted data, disable open-web tools, restrict tools to read-only case and policy scopes, scan files, and keep system instructions outside retrieved content. |
| Bias or inconsistent treatment | Test equivalent synthetic cases across demographic proxies, exclude protected traits from decision logic unless legally required, compare routes and rationales, and require compliance review before any live use. |
| Incorrect system write-back | Use idempotent `correlationId`, preview human-approved values, require RPA read-back, and compensate or create a recovery task on mismatch. |
| Weak adverse notice | Generate only after the human outcome; validate required fields and evidence references deterministically; retain the rendered notice with template and policy versions. |

### Measurable outcomes and evaluation

These are proposed measures, not forecast improvements:

- percentage of decisions completed within the applicable deadline;
- median and 90th-percentile time from complete packet to decision;
- percentage of cases requiring additional information, with reason;
- initial adverse-decision rate and appeal-overturn rate by service type,
  reviewer path, and contractor;
- percentage of notices with a specific reason and valid evidence references;
- extraction accuracy and evidence-citation precision on a labeled synthetic
  set;
- human edit rate for agent summaries and notice drafts; and
- write-back/read-back mismatch and recovery-task rates.

The synthetic evaluation set should include at least five cases:

1. complete, policy-supported expedited request that is approved;
2. missing therapy evidence that requests specific information;
3. apparently adverse case that reaches nurse and medical-director review;
4. conflicting packet and policy evidence that safely escalates; and
5. unavailable policy or legacy system dependency that takes the designed
   fallback without losing the case.

The principal Flow evaluator should assert route, deadline state, required human
outcome, and final status. A trajectory evaluator should verify policy-tool use
and source citations. Deterministic checks should verify that no denial bypasses
medical-director review and no system write completes without read-back.

## Unresolved assumptions for human review

- Confirm the target payer product and jurisdiction. Medicare Advantage is the
  evidence-backed default; Medicaid, CHIP, Marketplace, commercial, and
  state-specific requirements differ.
- Confirm whether SNF admission is the desired flagship service or whether
  imaging, rehabilitation, durable medical equipment, or another service better
  matches the audience.
- Name the authoritative coverage-policy corpus, its owner, versioning method,
  and precedence rules. Synthetic policy is acceptable only for the demo.
- Decide which cases, if any, may be straight-through approved. The proposed
  baseline prohibits autonomous adverse decisions and reductions.
- Confirm nurse and medical-director authority, escalation buffer, after-hours
  coverage, and exact outcome vocabulary.
- Decide whether this domain is one of the three later Data Fabric/process-app
  variants. Until selected, the canonical case store remains a mock.
- Verify availability and ownership of IXP, context grounding, Action Center,
  correspondence, and the legacy-system test surface in `JD/demos`.
- Confirm whether a coded action app is worth its independent deployment
  boundary or whether a quick form is sufficient for the research demo.
- Confirm a production interoperability standard and version only after the
  payer's implementation is known; the demo endpoint must not be presented as
  a certified FHIR Prior Authorization API.
- Establish current process baselines before setting any improvement target or
  business case. This research intentionally provides no ROI or delivery-effort
  estimate.
