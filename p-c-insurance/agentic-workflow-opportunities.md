# P&C insurance agentic-workflow opportunities

## Recommendation

Build the first P&C demo around **catastrophe property-claim coverage and
advance-payment orchestration**. A claims adjuster should see a newly reported
wildfire loss become a complete, evidence-linked recommendation, then approve,
edit, request information, or escalate it before any consequential payment or
coverage action occurs.

This is a greenfield public-research recommendation. No carrier's internal
process, system access, volumes, baselines, or existing automations were
available. Scores, architecture choices, and the ordering below are therefore
explicit design inferences, not measured carrier findings. No ROI, delivery
hours, or internal performance targets are asserted.

## Evidence and operating context

### Scale and pain points

- Catastrophe intake arrives in bursts and spans multiple product types. As of
  March 2026, the California Department of Insurance reported 38,835 residential
  property claims, 2,965 commercial-property claims, and 7,466 commercial and
  personal auto claims from the 2025 Palisades and Eaton fires. Payments shown
  in the same dataset exceed $23.8 billion across those categories. The affected
  teams are contact-centre staff, desk and field adjusters, catastrophe teams,
  claims supervisors, finance, and vendor managers. ([California Department of
  Insurance wildfire claims data, March 2026](https://www.insurance.ca.gov/01-consumers/180-climate-change/upload/Auto-Residential-and-Commercial-Property-Claims-Information-as-of-March-3rd-Palisades-and-Eaton-Fires.pdf))
- Surge capacity is an operational design problem, not merely a document task.
  Slide Insurance states that it uses inside and outside adjusters during
  high-volume catastrophe events while retaining reserving and payment authority;
  Travelers describes dedicated and enterprise response teams that can be
  deployed when event volume exceeds dedicated capacity. ([Slide 2025 Form
  10-K](https://www.sec.gov/Archives/edgar/data/1886428/000119312526083277/slde-20251231.htm),
  [Travelers 2025 Form 10-K](https://www.sec.gov/Archives/edgar/data/86312/000008631226000065/trv-20251231.htm))
- Property damage is not a binary image-classification problem. California's
  insurance regulator directed insurers to investigate smoke claims fully and
  fairly, including professional testing when warranted, rather than summarily
  denying them. The operational handoff can include an adjuster, environmental
  tester, remediation vendor, coverage specialist, and claimant. ([California
  Department of Insurance smoke-claim guidance, March 2025](https://www.insurance.ca.gov/0400-news/0100-press-releases/2025/release023-2025.cfm))
- Commercial underwriting has a different but credible intake bottleneck. A
  2024 survey of 201 insurance underwriters reported that commercial-lines
  underwriters spent 41% of their time on administrative activities and 33% on
  core underwriting activities. Lloyd's now emphasizes process simplification,
  common data standards, and incremental modernization for its market. A broker,
  underwriting assistant, underwriter, risk engineer, and pricing team all touch
  that journey. ([Capgemini World Property and Casualty Insurance Report 2024,
  pp. 8-9](https://www.sogeti.com/wp-content/uploads/sites/3/2024/10/wpcir_2024_web.pdf),
  [Lloyd's market modernization update, March 2026](https://www.lloyds.com/about-lloyds/blueprint-two))
- Recovery and fraud are material downstream workflows. Mercury General reported
  that subrogation reduced its 2025 catastrophe losses by about $586 million,
  while the National Insurance Crime Bureau (NICB) says its 2025 work included
  catastrophe, contractor-fraud, questionable-claim network, and investigative
  workflow initiatives. NICB separately projected a 49% rise during 2025 in
  insurance fraud linked to identity theft, based on its analysis of thousands
  of questionable claims submitted from 2022 through June 2025. The affected
  roles include recovery specialists, claims counsel, adjusters, special
  investigation units (SIUs), and law-enforcement liaisons. ([Mercury General
  2025 Form 10-K](https://www.sec.gov/Archives/edgar/data/64996/000006499626000005/mcy-20251231.htm),
  [NICB 2025 annual report](https://nicb.org/annual-reports/2025-annual-report),
  [NICB identity-theft fraud analysis, September 2025](https://www.nicb.org/news/news-releases/nicb-projects-49-rise-insurance-fraud-linked-identity-theft-2025))

### Controls and credible automation precedents

- Claims handling must remain prompt, fair, explainable, and evidence based. The
  NAIC model act identifies failures to acknowledge communications, implement
  reasonable investigation standards, make prompt and equitable settlements, or
  investigate before refusal as unfair practices. Actual obligations vary by
  jurisdiction. ([NAIC Unfair Claims Settlement Practices Act, Model 900](https://content.naic.org/sites/default/files/model-law-900.pdf))
- California illustrates the time-bound nature of claims operations: its consumer
  guidance says insurers generally must acknowledge a claim and begin
  investigation within 15 days, accept or deny within 40 days after proof of
  claim, and pay within 30 days after settlement. Those figures are evidence for
  a configurable jurisdictional deadline service, not universal U.S. rules.
  ([California Department of Insurance fair-claims guidance](https://www.insurance.ca.gov/01-consumers/105-type/95-guides/01-auto/hadaccident.cfm))
- Catastrophe rules can alter the ordinary workflow. California required certain
  wildfire advance payments, including 30% of the dwelling limit up to $250,000
  for contents and at least four months of living expenses. This supports a
  rules-backed, human-approved advance-payment route. ([California Department of
  Insurance claims tracker announcement, January 2025](https://www.insurance.ca.gov/0400-news/0100-press-releases/2025/release011-2025.cfm))
- Automation is already replicable in parts of the sector. Safety Insurance says
  it implemented a generative-AI application that extracts data from claim PDFs
  and uses robotics to enter it into core systems. Lemonade reported that its
  claims bot took 96% of first notices of loss and that roughly 55% of claims
  were automated at year-end 2025, with concerns and unauthorized settlements
  routed to human specialists. These are carrier-specific precedents, not
  forecast results for this demo. ([Safety Insurance 2025 Form 10-K](https://www.sec.gov/Archives/edgar/data/1172052/000117205226000005/saft-20251231x10k.htm),
  [Lemonade 2025 Form 10-K](https://www.sec.gov/Archives/edgar/data/1691421/000169142126000016/lmnd-20251231.htm))
- If AI supports a consumer-impacting decision, governance is part of the
  workflow. The NAIC model bulletin calls for a written AI-systems program,
  risk-based governance, testing, documentation, data lineage, bias analysis,
  third-party oversight, and compliance with unfair-trade-practice laws.
  Adoption and legal effect vary by state. ([NAIC Model Bulletin on AI Systems
  Used by Insurers](https://content.naic.org/sites/default/files/inline-files/2023-12-4%20Model%20Bulletin_Adopted_0.pdf))

## Ranked demo candidates

Scores are design judgments on a 1-5 scale. `Demo value` measures how clearly a
short demonstration can show orchestration, agent/tool specialization, business
routing, human judgment, and recovery. `Enterprise credibility` measures the
strength of the cited problem, accountable roles, controls, and measurable
outcomes. `Feasibility` measures whether representative inputs, rules, systems,
and exceptions can be mocked without pretending to make a production-grade
insurance decision.

| Rank | Candidate and hero moment | Evidence and workflow | Demo value | Enterprise credibility | Feasibility | Total |
| --- | --- | --- | ---: | ---: | ---: | ---: |
| 1 | Catastrophe property-claim coverage and advance-payment orchestration. An adjuster reviews a cited recommendation, edits the advance, and chooses the safe next action. | CAT volume, surge staffing, smoke investigation, time-bound handling, and advance-payment rules are documented above. Actors: claimant, intake team, adjuster, supervisor, field/vendor network, finance. Systems: policy and claims administration, document store, catastrophe/geospatial data, payments, communications. | 5 | 5 | 4 | **14** |
| 2 | Commercial P&C submission triage. An underwriter receives normalized submission data, appetite evidence, missing-information questions, and a route recommendation rather than re-keying a packet. | Underwriter administrative time and market data-standard modernization are documented above. Actors: broker, underwriting assistant, underwriter, risk engineer, pricing/referral authority. Systems: email/portal, document extraction, policy administration, rating and third-party risk data. | 5 | 4 | 4 | **13** |
| 3 | Catastrophe subrogation opportunity referral. A recovery specialist sees a claim-cluster hypothesis, supporting evidence, preservation tasks, and limitation-date controls before referral. | Mercury's reported catastrophe subrogation demonstrates material recovery relevance. Actors: adjuster, recovery specialist, counsel, experts, liable-party carrier. Systems: claims, payments, evidence store, legal matter management, external records. | 4 | 4 | 3 | **11** |
| 4 | Questionable-claim and contractor-network SIU referral packet. An SIU analyst reviews linked indicators and decides whether to investigate, return to ordinary handling, or refer externally. | NICB documents catastrophe and contractor-fraud programs, questionable-claim networks, and privacy-first model work. Actors: adjuster, SIU analyst, compliance, counsel, NICB/law enforcement. Systems: claims, entity/network data, document store, SIU case management. | 4 | 4 | 3 | **11** |

Candidate 1 wins because it combines a large, observable workload with documents,
rules, system lookups, specialized agents, UI automation, a consequential human
decision, parallel follow-up, and a safe exception route. Candidate 2 may be the
better choice if the intended audience is underwriting-led or wants distance from
the repository's auto-FNOL reference story.

## Strongest candidate: catastrophe property claim orchestration

### Scope and narrative

The demo starts when a synthetic residential wildfire claim is reported. It ends
after an authorized adjuster chooses an outcome and the Flow updates a mock claims
record, creates any required investigation or vendor tasks, and prepares an
approved customer communication. It does not autonomously deny coverage, issue a
payment, set a production reserve, detect fraud, or replace licensed claims
judgment.

The hero moment is the review experience: one screen presents policy facts,
extracted loss evidence, applicable rule citations, unresolved contradictions,
the agent's rationale, and proposed next actions. The reviewer can correct the
recommendation and the Flow demonstrably uses the returned values.

### Input contract

| Input group | Required fields | Classification and validation |
| --- | --- | --- |
| Event envelope | `claimId`, `correlationId`, `reportedAt`, `channel`, `catastropheEventId` | IDs must be unique; replay with the same correlation ID must not create a second work item. |
| Policy and loss | `policyNumber`, `policyStatus`, `effectiveFrom`, `effectiveTo`, `lossOccurredAt`, `riskAddress`, `reportedCause`, `claimantContact` | Policy facts come from the mock policy API, not the agent. Address and contact data are restricted PII. Date and status contradictions take the exception route. |
| Coverage facts | `dwellingLimit`, `contentsLimit`, `additionalLivingExpenseLimit`, `deductible`, `endorsements`, `priorPayments` | Monetary values and policy wording are read-only inputs. The implementation must use synthetic values and a versioned policy/rule set. |
| Evidence | claim form, adjuster notes, photos, estimates, receipts, optional contents list, smoke or environmental report | Files may contain PII and property-security details. Malware scan, type/size checks, document classification, extraction confidence, and source-page references are required. |
| Jurisdiction configuration | `jurisdiction`, `ruleSetVersion`, `noticeDeadline`, `investigationDeadline`, `advancePaymentRules`, `authorityLimits` | A claims/compliance owner supplies this configuration. The California evidence above is only the illustrative demo profile. |

### Output contract

| Output | Fields and consumer | Safety contract |
| --- | --- | --- |
| Canonical assessment | `coverageRecommendation`, `coverageBasis[]`, `damageCategories[]`, `missingEvidence[]`, `contradictions[]`, `confidence`, `ruleSetVersion` for adjuster review | Every conclusion links to policy text, a rule, or a source page. `recommendation` is never a final coverage decision. |
| Triage and assignment | `severityBand`, `specialtiesNeeded[]`, `queue`, `dueAt`, `reasonCodes[]` for claims operations | Routing uses configured deterministic conditions. Missing or conflicting facts route to review, not straight-through action. |
| Human decision | `outcome`, `approvedAdvanceAmount`, `approvedCoveragePosition`, `editedFields[]`, `reviewerRationale`, `reviewerId`, `decidedAt` for downstream Flow branches | Only an authenticated role within authority limits can approve. An authority breach routes to a supervisor. |
| Action plan | `investigationTasks[]`, `vendorTasks[]`, `paymentRequest`, `customerMessage`, `nextReviewAt` for claims, vendor, finance, and communications systems | A proposed payment remains non-executing in the base demo. Customer text is sent only after reviewer approval. |
| Audit record | input hashes, evidence references, prompt/model/tool versions, agent rationale, rule evaluations, overrides, timestamps, and system responses | Append-only demo audit log; redact secrets and minimize PII in traces. |

### Intended integrations and demo mocks

| Component | Intended responsibility | Demo implementation and failure route |
| --- | --- | --- |
| Claims/policy API workflow | Read policy and claim facts; persist approved status, tasks, and decision. | Versioned local JSON-backed mock with realistic response/error schemas. Timeout or mismatch creates a retryable integration incident; no agent guesses missing policy facts. |
| IXP document processing | Classify packet pages and extract loss, estimate, receipt, and environmental-report fields with page references. | Synthetic packet and a deployed test extractor when available; otherwise a recorded extraction response clearly labelled as a mock. Low-confidence or contradictory fields become reviewer tasks. |
| Policy/rule context tool | Retrieve only the applicable policy clauses and jurisdiction rules. | Curated, versioned synthetic policy corpus plus the public rules cited above. Return source IDs and effective dates; absence forces review. |
| Catastrophe/geospatial enrichment | Confirm event and location relationship and attach hazard context. | Deterministic fixture or public-data adapter. `unknown`, timeout, or disputed perimeter is a visible non-adverse fallback, not a denial signal. |
| Vendor portal RPA | Request inspection, smoke testing, mitigation, or temporary-housing support where no suitable API exists. | Mock portal with synthetic data. It returns confirmation ID or screenshot-backed exception. RPA is invoked only after human authorization. |
| Coded evidence agent | Reconcile evidence, policy clauses, and rule results into a cited structured assessment; run a self-check for unsupported conclusions. | Narrow schema-bound agent. It cannot alter records, calculate a final payment, or communicate externally. Invalid schema or missing citation routes to human review. |
| Coded action app / Action Center | Show facts and evidence; capture edits, authority-sensitive decisions, and rationale. | Primary review surface. Named outcomes: `Approve plan`, `Request information`, `Escalate coverage`, and `Assign investigation`. Timeout routes to the claims-supervisor queue. |
| Notification API | Prepare and, after approval, send acknowledgement or information-request text. | Mock outbox by default. Delivery failure records an incident and leaves the claim open for retry. |

### Human decisions

1. A claims adjuster validates extracted facts, the coverage recommendation,
   applicable advance-payment rule, proposed investigation, and customer message.
2. The adjuster may edit permitted fields and must provide rationale when
   overriding a recommendation or choosing an exception route.
3. A claims supervisor decides cases beyond the adjuster's configured authority,
   deadline-risk cases, or disputed coverage. A specialist reviews smoke,
   environmental, fraud, or legal issues; no such flag produces an automatic
   adverse outcome.
4. The Flow resumes from the named outcome and returned field IDs. `Approve plan`
   writes the approved plan and starts independent follow-up; `Request
   information` creates claimant tasks; `Escalate coverage` creates a supervisor
   task; `Assign investigation` creates specialist/vendor tasks.

### Flow topology and reference mapping

| Reference segment | P&C canvas segment | Actors and branch evidence |
| --- | --- | --- |
| Receive and understand | **1. Register catastrophe loss** | Trigger -> claims/policy API workflow -> IXP. Output is a correlated claim packet. Invalid IDs, file checks, and extraction uncertainty run below the happy path. |
| Assess and enrich | **2. Build cited claim assessment** | Inline triage agent uses the event/location tool; the coded evidence agent uses policy/rule context and produces schema-bound recommendations. `missingEvidence`, `contradictions`, and configuration-owned confidence gates drive routing. |
| Decide and review | **3. Exercise claims authority** | A real decision routes ordinary review, specialist review, or integration exception. The action app returns the named outcome, edited values, reviewer identity, and rationale. |
| Act and communicate | **4. Coordinate recovery** | After approval, vendor/RPA work, claim write-back, and customer-message preparation run in parallel where independent; a merge records their results before completion or incident handling. |

The implementation boundary should be
`p-c-insurance/catastrophe-property-claim/catastrophe-property-claim-solution/`
with one `.uipx` and a nested Flow project. The exact actor/resource readiness,
package name, and process-app selection remain implementation decisions. This
research does not claim that any UiPath tenant resource or external connection is
currently configured.

### Risks and controls

| Risk | Control and observable evidence |
| --- | --- |
| Incorrect or unfair coverage action | Agent output is advisory; cited evidence is required; adverse, uncertain, conflicting, or out-of-authority cases require a qualified human. Persist recommendation, human outcome, reason, and override. |
| Missed jurisdictional deadline | Compute deadlines from a compliance-owned rule version, expose due times on every task, escalate before breach, and audit pauses/retries. Do not hard-code California timing as universal. |
| Hallucinated policy or rule | Retrieval is restricted to approved versioned sources. Output schema requires source IDs and effective dates. A missing or invalid citation fails closed to review. |
| Bias or proxy discrimination | Exclude protected characteristics and unnecessary proxies from prompts and routing; test outcome and override patterns; retain data lineage, model/prompt versions, and third-party validation evidence consistent with the NAIC bulletin. |
| PII or property-security exposure | Synthetic demo data; least-privilege connections; encryption; retention limits; trace redaction; no secrets, full documents, or exact property details in prompts unless required and approved. |
| Fraud flag harms claimant | Treat anomaly/fraud signals only as investigation referrals. Do not deny, delay, or accuse automatically; restrict SIU details and record authorized access. |
| Automation or vendor failure | Idempotent correlation, bounded retry, explicit incident state, manual work-queue fallback, and reconciliation of ambiguous write results. |
| Uncontrolled payment | The base demo creates a payment request, not a live disbursement. Production payment would require authority checks, segregation of duties, duplicate detection, and finance reconciliation. |

### Measurable outcomes and evaluation

Establish carrier baselines before setting targets. Credible business measures are
median and tail time from notice to acknowledgement, initial triage, adjuster
assignment, first approved action, and customer update; packet-completeness and
correct-routing rates; manual touch time; rework/supplement rate; missed-deadline
count; exception aging; specialist utilization; override rate; customer-contact
volume; and duplicate or failed write-backs. Compare by jurisdiction, peril,
channel, and protected-class-safe monitoring cohorts where legally approved.

The demo evaluation set should use 4-5 synthetic cases:

| Case | Expected route and proof |
| --- | --- |
| Complete covered loss | Assessment cites policy/rules, adjuster approves, independent actions merge, audit summary completes. |
| Missing or contradictory policy fact | No coverage conclusion or payment request; request-information or integration-exception route. |
| Smoke-only claim | Investigation/testing recommendation reaches specialist review; it is not summarily denied. |
| Authority-limit escalation | Adjuster edit beyond configured authority routes to supervisor and resumes with returned decision. |
| Dependency failure or replay | Tool/API failure creates an incident without duplicate work; replayed correlation ID returns the existing case. |

Implementation should define, with claims/compliance owners, the extraction and
routing thresholds and the evaluator pass criteria. Required evaluators are: a
deterministic route/contract evaluator, a grounded-citation evaluator, a
tool-use/trajectory evaluator for policy/rule retrieval, and an end-to-end check
that no payment or adverse decision occurs without the required human outcome.

## Unresolved assumptions for human review

1. Which carrier segment, product, peril, and jurisdiction should anchor the
   demo? The current recommendation assumes a synthetic California residential
   wildfire profile for evidence richness.
2. Is the target audience claims-led or underwriting-led? An underwriting-led
   audience may prefer candidate 2 even though candidate 1 ranks highest overall.
3. Which core claims/policy platform and vendor portal should be represented,
   and may their UI/brand be shown? Until confirmed, use neutral mocks.
4. Which policy form, endorsements, authority matrix, deadline rules, and
   advance-payment rules will claims and compliance approve for the synthetic
   corpus?
5. Is the payment step allowed to remain a non-executing request, as recommended,
   or must the demo show a sandbox payment integration?
6. Are IXP, Action Center/coded action app, external agent, MCP/context source,
   and notification connections available in the target folder? No readiness has
   been verified in this research issue.
7. Should this demo be one of the three Data Fabric-backed process-app variants?
   If yes, Data Fabric becomes the canonical claim record and the review resumes
   from record-change events rather than a detached task.
8. What baselines and targets can the carrier substantiate for cycle time,
   touch time, routing accuracy, rework, deadline compliance, and overrides?
9. What retention, residency, model-provider, privacy, explainability, and SIU
   access policies apply to prompts, traces, documents, images, and audit data?
10. Who owns sign-off for claims legality, AI governance, security, model risk,
    and the boundary between recommendation and authorized action?

## Research limits

Research was performed on public U.S. sources available on August 10, 2026. It
uses regulator publications, insurer SEC filings, industry-standard bodies, and
identified industry research. Public evidence demonstrates cross-carrier pain
and precedent but does not prove any one carrier's process or opportunity size.
There was no internal replicable model to inspect; the Safety Insurance and
Lemonade examples are external precedents and must not be treated as transferable
performance promises. Regulatory sources cited include models and one state's
rules; counsel must determine the rules applicable to an implementation.
