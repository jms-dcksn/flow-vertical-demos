# Commercial banking agentic-workflow opportunities

## Decision summary

**Recommendation:** build a payment exception investigation and resolution demo.
It combines high-value payment operations, structured ISO 20022 data, policy-bound
reasoning, legacy-system work, a consequential human decision, external messaging,
and an auditable outcome in one legible Maestro Flow.

This is public-industry research, not discovery inside a bank. Workflow descriptions,
scores, and the proposed design are explicitly inferences from the cited sources.
No source establishes a particular bank's volumes, current process, automation
coverage, savings, or return on investment. Those values must be measured with a
design partner before implementation.

## Evidence base

The evidence indicates large transaction scale, time-sensitive exception work, and
strong control obligations:

- The Fedwire Funds Service originated 217,296,700 transfers in 2025, averaging
  869,187 transfers and $4.593 trillion per day. These are network totals, not an
  addressable volume for any one bank. ([Federal Reserve Financial Services,
  2025 annual statistics](https://www.frbservices.org/resources/financial-services/wires/volume-value-stats/annual-stats.html))
- Fedwire completed its ISO 20022 migration in July 2025. The Federal Reserve says
  richer structured data supports straight-through processing, fraud-risk mitigation,
  sanctions compliance, and AML compliance. ([Federal Reserve Financial Services,
  July 16, 2025](https://www.frbservices.org/news/fed360/issues/071625/wires-iso-20022-implementation-complete-fedwire-funds-service))
- Swift reports that roughly 2–5% of payments generate enquiries on a given day and
  operations teams average 3–4 minutes per payment-instruction investigation. This is
  a Swift industry estimate, not a measured baseline for this repository or a specific
  institution. ([Swift, payment exceptions use case](https://www.swift.com/standards/iso-20022/supercharge-your-payments-business/chapter-6))
- Swift's current migration plan requires institutions to be able to receive
  `camt.110` investigation requests in November 2026 and moves investigation requests
  and responses to `camt.110`/`camt.111` through Case Management in November 2027.
  ([Swift, ISO 20022 exceptions and investigations FAQ](https://www.swift.com/standards/iso-20022/iso-20022-faqs/iso-20022-exceptions-and-investigations))
- FFIEC examination procedures require banks to evaluate fund-transfer risk using
  frequency, value, jurisdiction, and their role in the transfer, and to address
  missing, meaningless, incomplete, or suspicious payment information.
  ([FFIEC BSA/AML Manual, Funds Transfers](https://bsaaml.ffiec.gov/manual/RisksAssociatedWithMoneyLaunderingAndTerroristFinancing/07_ep))
- FFIEC guidance says payments such as funds transfers and letters of credit should
  be checked against OFAC lists before execution, with risk-based procedures for
  transaction parties and jurisdictions. ([FFIEC BSA/AML Manual, OFAC](https://bsaaml.ffiec.gov/manual/OfficeOfForeignAssetsControl/01))
- Customer due diligence must support a customer risk profile and ongoing monitoring,
  with documented analysis and resolution when information is insufficient or
  inaccurate. ([FFIEC BSA/AML Manual, Customer Due Diligence](https://bsaaml.ffiec.gov/manual/AssessingComplianceWithBSARegulatoryRequirements/02_ep))
- FinCEN received 4.8 million SARs in fiscal 2025, including 2.8 million from
  depository institutions. This establishes compliance workload scale across the
  industry but does not equal alert volume or commercial-banking volume.
  ([FinCEN, Year in Review](https://www.fincen.gov/about-fincen/fincen-year-review))
- The 2025 Shared National Credit portfolio covered 6,857 borrowers and $6.9 trillion
  in commitments; 8.6% of commitments were non-pass, and leveraged loans accounted
  for 81% of non-pass loans. ([Federal Reserve, FDIC, and OCC, 2025 SNC report
  release](https://www.federalreserve.gov/newsevents/pressreleases/bcreg20260112a.htm))
- OCC commercial-loan guidance expects systems that monitor compliance with loan
  covenants. ([OCC, Comptroller's Handbook: Commercial Loans](https://www.occ.treas.gov/publications-and-resources/publications/comptrollers-handbook/files/commercial-loans/pub-ch-commercial-loans.pdf))
- ICC reports that approximately 65–80% of documentary credits are refused on first
  presentation because of discrepancies, while noting that many are resolved quickly.
  This figure is a global range, not a bank-specific reject rate. ([ICC Academy,
  April 8, 2025](https://academy.iccwbo.org/trade-finance/article/25-tips-to-avoid-common-documentary-credit-issues/))
- ADB's 2025 survey of more than 110 trade-finance providers estimates a $2.5 trillion
  global trade-finance gap and identifies accelerated digitalization as a priority.
  ([Asian Development Bank, 2025 Global Trade Finance Gap Survey](https://www.adb.org/publications/adb-global-trade-finance-gap-survey))

## Candidate ranking

Scores are editorial inferences on a 1–5 scale. They compare suitability for a
short, synthetic UiPath demo; they are not business cases or delivery estimates.
Each dimension is equally weighted.

| Rank | Candidate | Demo value | Enterprise credibility | Feasibility | Total /15 | Evidence and rationale |
| --- | --- | ---: | ---: | ---: | ---: | --- |
| 1 | Payment exception investigation and resolution | 5 | 5 | 4 | **14** | Payment data and messages create a strong intake-to-decision story. Fedwire scale, Swift enquiry/touch-time estimates, near-term `camt.110`/`camt.111` milestones, and FFIEC transfer controls make the workflow timely. A synthetic ISO-message/API mock is feasible; real network connectivity is not assumed. |
| 2 | Documentary trade-finance discrepancy review | 5 | 4 | 4 | **13** | Document comparison, policy retrieval, sanctions screening, human waiver/refusal, and notice drafting showcase IXP and agents. ICC's 65–80% first-presentation refusal range signals a material workflow, but the statistic is global and product complexity can overwhelm a short demo. |
| 3 | Commercial-customer AML alert investigation | 4 | 5 | 3 | **12** | FinCEN's 2.8 million depository-institution SARs and FFIEC's CDD requirements support enterprise relevance. The evidence-gathering and investigator decision fit Flow well, but realistic typologies, SAR confidentiality, and safe synthetic data raise demo risk. |
| 4 | Commercial-loan covenant breach triage | 4 | 4 | 4 | **12** | OCC expects covenant-monitoring systems, while the 2025 SNC review shows material credit-monitoring exposure. Financial-statement intake, covenant calculation, relationship-manager context, and credit-officer review are compelling; representative loan-system integration and accounting edge cases need careful scoping. |
| 5 | Event-driven commercial KYC refresh | 4 | 5 | 3 | **12** | FFIEC requires risk-based ongoing monitoring and updates when material customer information changes. Orchestrating registry evidence, ownership documents, screening, outreach, and compliance approval is credible, but public evidence does not provide a reliable workload baseline and entity-resolution quality is difficult to demonstrate safely. |

### Candidate workflow evidence

| Candidate | Pain point and inferred workflow | Actors and systems | Controls | Outcomes to measure in a pilot |
| --- | --- | --- | --- | --- |
| Payment exception | **Inference:** an operations analyst correlates payment, customer, screening, and network-message evidence; identifies missing or inconsistent data; requests information, repairs a non-material field, escalates, or rejects/returns under policy. | Payment operations, sanctions/AML reviewer, relationship manager; payment hub, Swift/Fedwire gateway, screening platform, customer master, case/message store. | Maker-checker authorization, sanctions hold, deterministic validation, evidence provenance, immutable event history, no agent-authorized release of funds. | Median and p90 resolution time, queue age, human touch time, percentage correctly auto-triaged, reopen rate, policy/escalation accuracy. |
| Trade-finance discrepancy | **Inference:** classify a letter-of-credit package, extract fields, compare documents to credit terms and UCP/ISBP policy, screen parties, and send discrepancies to a document checker for accept/waive/refuse. | Trade operations, applicant, issuing/confirming bank; document repository, trade platform, sanctions screening, customer channel. | Exact-source citations, extraction-confidence gate, checker approval, deadline tracking, separation of discrepancy detection from legal interpretation. | First-pass review time, extraction accuracy, discrepancy precision/recall, reviewer override rate, notice timeliness. |
| AML alert | **Inference:** assemble account, customer, ownership, transaction, and prior-case evidence; summarize a typology; and route an investigator to close, monitor, or escalate for SAR consideration. | AML investigator and manager; transaction monitoring, KYC/customer master, case management, screening, document store. | SAR confidentiality, least privilege, five-year supporting-document retention where applicable, human filing decision, reason codes, no customer-facing disclosure. | Investigation cycle time, evidence completeness, reviewer agreement, false-positive disposition, overdue cases; never optimize only for fewer SARs. |
| Covenant breach | **Inference:** ingest borrower statements and a loan agreement, calculate deterministic covenant values, explain variances, retrieve exposure and collateral context, and route a credit officer to waive, amend, escalate, or request information. | Credit analyst, relationship manager, credit officer; loan origination/servicing, financial spreading, document store, CRM, risk-rating system. | Deterministic calculations, approved definitions, dual approval for waivers, versioned statements/agreements, audit trail, no autonomous risk-rating change. | Time from statement receipt to decision, overdue reviews, calculation accuracy, exception aging, rework, early-warning escalation timeliness. |
| KYC refresh | **Inference:** react to a material customer event, collect registry and customer evidence, compare beneficial ownership and expected activity to the current profile, then obtain compliance approval. | KYC analyst, relationship manager, customer, compliance approver; CRM, KYC platform, registry/data provider, screening, document store. | Event-driven/risk-based scope, source provenance, privacy and retention, identity-match thresholds, human approval of risk-profile changes, documented insufficient-information route. | Review cycle time, evidence completeness, outreach count, stale-profile age, match precision, reviewer overrides. |

No public evidence was found for a working internal automation that could be
replicated across a specific bank. This is therefore a greenfield shortlist, not a
claim that one candidate is a proven replicable model.

## Strongest candidate: payment exception investigation and resolution

### Demo narrative and hero moment

A synthetic inbound ISO 20022 payment investigation arrives with a UETR. Flow
correlates the original payment, customer profile, screening result, network status,
and operating policy. Deterministic checks identify malformed or missing data; an
agent produces a cited evidence summary and bounded recommendation. A payment
operations reviewer sees the source fields, mismatches, policy evidence, and proposed
action in one task, then chooses **Request information**, **Approve repair**,
**Return/reject**, or **Escalate compliance**. The hero moment is the reviewer
correcting a proposed repair and Flow using the returned fields to complete both the
network response and the case audit record.

The demo must never imply that an LLM releases, recalls, rejects, or repairs a live
payment. The synthetic scenario ends in mock write-back receipts.

### Input and output contract

#### Input

| Field | Type | Required | Purpose and handling |
| --- | --- | --- | --- |
| `caseId` | string | yes | Synthetic case identifier and idempotency key. |
| `uetr` | UUID string | yes | Correlates all payment and investigation events. |
| `investigationMessage` | object | yes | Synthetic `camt.110`-like payload: reason code, requester, requested information, timestamps. Preserve raw and parsed forms. |
| `originalPayment` | object | yes | Synthetic `pacs.008`-like fields: amount/currency, debtor/creditor and agents, remittance data, value date. |
| `networkStatus` | object | yes | Mock status, timestamps, and prior message references. |
| `customerRiskContext` | object | yes | Synthetic customer ID, expected activity summary, risk tier, jurisdiction flags. Exclude real PII. |
| `screeningResult` | object | yes | Mock status, possible-match reason, list version, timestamp; never raw live watchlist data. |
| `policyVersion` | string | yes | Pins the approved operating-procedure snapshot used by retrieval. |
| `receivedAt` | ISO 8601 timestamp | yes | Starts SLA and aging measures. |

#### Output

| Field | Type | Purpose |
| --- | --- | --- |
| `caseId`, `uetr` | string | Correlation and audit. |
| `classification` | enum | `missing_information`, `data_mismatch`, `status_request`, `possible_duplicate`, `screening_escalation`, or `other`. |
| `validationFindings` | array | Deterministic rule ID, affected field, actual value, expected constraint, pass/fail. |
| `evidenceSummary` | object | Bounded narrative plus source IDs and policy section citations. |
| `recommendedAction` | enum | `request_information`, `approve_repair`, `return_or_reject`, `escalate_compliance`. Advisory only. |
| `reviewOutcome` | object | Reviewer, selected outcome, edited repair fields, rationale, timestamp. |
| `responseMessage` | object | Synthetic `camt.111`-like response or information request. |
| `writeBackReceipts` | array | Mock payment-hub, network, and case-store operation IDs and statuses. |
| `finalStatus` | enum | `awaiting_information`, `repair_approved`, `returned_or_rejected`, `compliance_hold`, `completed`, `technical_exception`. |
| `auditEvents` | array | Actor, action, input/output hashes, source references, model/prompt version where applicable, timestamp. |

### Proposed Flow mapped to the reference solution

| Reference segment | Payment-exception segment | Actors and visible output | Branch/merge evidence |
| --- | --- | --- | --- |
| Receive and understand | **Receive and correlate** | Webhook/manual trigger; API workflow validates and parses the message; RPA retrieves a status from a mock legacy operations console. Output is a canonical exception case. | Invalid schema, duplicate `caseId`, or unmatched UETR routes below the happy path to technical/manual intake. |
| Assess and enrich | **Investigate the payment** | Deterministic validator; inline agent with read-only customer, screening, and policy tools; narrowly scoped coded agent creates the evidence bundle. | `requiresCompliance == true` or `confidence < configuredThreshold` forces escalation. Threshold remains a configuration decision. |
| Decide and review | **Control the resolution** | Action Center coded action app for a payment-operations reviewer; separate compliance task for screening cases. | Named outcomes drive separate routes. Only a person can approve a repair, return/reject, or remove a hold. Timeout escalates to the queue manager. |
| Act and communicate | **Respond and close** | In parallel, API workflow writes a mock network response, RPA records the approved repair in the mock legacy console, and the coded agent drafts an internal/customer-safe status message; merge before closing the case. | Any failed write-back leaves the case open in `technical_exception`; successful receipts merge into an audit-ready completion. |

Canvas presentation should use four left-to-right sticky notes matching the segment
titles above, with exception paths below. This deliberately mirrors the repository's
claims reference while substituting a real payment-data decision for literal routing.

### Integrations and demo mocks

| Dependency | Demo implementation | Production question or fallback |
| --- | --- | --- |
| ISO 20022 investigation channel | Local synthetic `camt.110`-like JSON fixture through a webhook or manual trigger; generate a `camt.111`-like result. | Confirm Swift Case Management or gateway API/channel, schemas, test environment, certification, and 2026/2027 migration plan. Keep local fixtures if unavailable. |
| Payment hub / core | Mock REST API for original payment and write-back receipt. | Confirm system of record, supported actions, idempotency, entitlements, and whether write-back is API-based. |
| Legacy operations console | Purposeful RPA path reads status and records an approved repair in a local mock UI. | Use RPA only if the selected bank lacks a suitable API; otherwise replace this actor with the API and document why. |
| Sanctions/AML screening | Static, synthetic screening response with list-version metadata. | A real connection requires security and compliance approval. Failure or ambiguity always creates a compliance hold. |
| Customer master / KYC | Read-only mock API returning a synthetic risk summary. | Minimize fields and verify data residency, purpose limitation, retention, and access controls. |
| Operating policy | Versioned repository documents in a context index. | Policy owner approves the exact corpus and citation granularity. Retrieval failure blocks agent recommendation but not manual review. |
| Case/audit store | Local mock endpoint or queue item containing canonical status and event receipts. | Confirm record system, retention, legal hold, audit access, and reconciliation ownership. |
| Notification | Draft-only message displayed to reviewer or written to a mock outbox. | Confirm approved templates and channels; never disclose a sanctions/AML rationale to the customer. |

Resource and connection readiness is **unverified**. The local CLI is installed, but
the research session was not authenticated to UiPath Automation Cloud, so no tenant
resource, connector, folder, or deployment claim is made.

### Human decisions

| Decision | Role | Evidence shown | Returned data and route |
| --- | --- | --- | --- |
| Repair routine data | Payment operations reviewer with maker-checker authority | Original and proposed values, deterministic findings, payment/network status, cited policy, agent rationale. | Edited fields and rationale; mock write-back, second-person authorization if policy requires, then respond. |
| Request information | Payment operations reviewer | Missing fields, requester/beneficiary context, response deadline, approved request template. | Requested fields and due date; send mock request and wait for correlated response. |
| Return or reject | Authorized payment operations reviewer | Reason code, policy evidence, payment state, downstream impact. | Reason and rationale; mock return/reject response and communication. |
| Escalate or maintain hold | Sanctions/AML reviewer | Screening result, party/jurisdiction evidence, risk context, full provenance. | `release_to_operations`, `maintain_hold`, or `investigate_further`; agent cannot select or execute release. |

### Risks, compliance, and controls

| Risk | Required demo control |
| --- | --- |
| Hallucinated policy or facts | Agents receive only structured case data and an approved versioned corpus; every material claim carries a source ID; missing evidence produces `insufficient_evidence`. |
| Unauthorized movement of funds | Tool permissions are read-only until after a named human outcome. The demo writes only to mocks and separates recommendation, authorization, and execution. |
| Sanctions/AML harm | Possible matches and incomplete party data route to compliance hold. No agent suppresses or clears an alert. Customer messages omit restricted rationale. |
| Incorrect repair | Schema and business-field checks are deterministic. Reviewer sees before/after values; high-risk fields require second-person authorization. |
| Duplicate or replayed message | `caseId` plus UETR is the idempotency key; repeated events return the existing case and append an audit event. |
| Sensitive-data leakage | Use synthetic data; minimize fields; mask account identifiers in tasks/logs; prohibit external web search and unapproved tools. |
| Tool or dependency failure | Timeout, bounded retry, circuit-breaker-style stop, and explicit technical-exception queue. Never infer a successful write-back without a receipt. |
| Weak accountability | Record source versions, rule IDs, agent/model/prompt versions, recommendations, reviewer edits/outcomes, and operation receipts. |
| Model drift | Maintain synthetic evaluation cases and approval thresholds; version prompts/models; require regression evaluation before promotion. |

### Evaluation and measurable outcomes

Use only synthetic cases. The initial evaluation set should contain:

1. complete status enquiry that routes to a response without repair;
2. missing creditor identifier that routes to request information;
3. safe, policy-allowed remittance repair that requires reviewer approval;
4. possible sanctions match that always routes to compliance hold; and
5. payment-hub timeout that ends in technical exception without a false receipt.

Evaluate deterministic classification and route correctness, required policy/source
citations, prohibited tool/action absence, human-outcome consumption, and write-back
receipt reconciliation. Success thresholds must be agreed during implementation; none
are invented here.

A design partner should baseline and then measure median/p90 resolution time, queue
age, human touch time, correct auto-triage rate, reviewer override/reopen rate,
technical-exception recovery, and evidence completeness. Swift's 2–5% enquiry and
3–4 minute investigation estimates are context, not the demo's target or ROI model.

## Unresolved assumptions for human review

- Is the target rail Swift cross-border payments, Fedwire, CHIPS, or an internal
  commercial-payment hub? The message contract and controls change by rail.
- Which exception types are in scope: information request, status, duplicate,
  repair, cancellation/recall, return/reject, screening, or reconciliation?
- Does a design partner have measured volumes, resolution times, queue aging,
  rework, and escalation rates by exception type?
- Which fields may operations repair, which require maker-checker approval, and which
  may never change after acceptance?
- Who owns the final decision for routine repair, payment return/reject, sanctions
  disposition, and customer communication?
- Is Swift Case Management connectivity available in a non-production environment,
  or should the demo remain entirely synthetic?
- Which payment hub, screening, customer, case, and policy systems are representative,
  and which offer APIs? Is RPA genuinely required for one UI-only responsibility?
- What data classifications, residency, retention, legal-hold, and log-redaction rules
  apply to payment and screening data?
- What is the approved policy corpus, who signs it off, and how frequently does it
  change?
- What response deadlines, timeout escalations, and service-level measures should the
  Flow enforce?
- Is commercial banking one of the three future Data Fabric/process-app variants?
  This research does not select it.
- What success thresholds and reviewer-agreement standard make the demo credible?

## Method and limitations

Research was conducted on August 10, 2026 using public primary or authoritative
industry sources from the Federal Reserve, FFIEC, FinCEN, OCC, Swift, ICC, and ADB.
The analysis looked for workflow friction, scale, roles, systems, controls, human
judgement, measurable outcomes, and alignment to the repository's reference Flow.

The evidence does not reveal any individual bank's actual process or technology
estate. Product-provider figures are identified as such. Scores are comparative
judgement, and no pack-hours, implementation complexity band, benefit forecast, or
ROI is supplied. A bank-side process walk-through and anonymized operational sample
are required before a build specification or business case is treated as validated.
