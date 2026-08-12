# Retail agentic-workflow opportunities

Research current as of August 10, 2026. This is public-source discovery for a
representative US omnichannel retailer, not a finding about any specific
company. Operational baselines, ROI, delivery estimates, UiPath resource
readiness, and legal conclusions must be supplied or validated by the adopting
retailer.

## Recommendation

Build a **consumer-product recall containment and remedy** demo. A public or
supplier recall notice enters Flow; deterministic matching and an agent scope
the affected catalog and inventory; exact matches are contained immediately;
ambiguous matches stop for a product-safety lead; and approved containment,
customer notification, remedy, and reporting work proceeds in parallel before
an audit-ready closeout.

The hero moment is a product-safety lead reviewing an explainable proposed match
between a regulator notice and several retailer SKUs. The reviewer confirms the
affected SKU/lot/date-code boundary, and Flow visibly fans the decision out to
e-commerce/POS stop-sale, inventory isolation, targeted notification, and
remedy operations. This makes agentic reasoning useful without allowing an
agent to make the safety or regulatory decision.

This recommendation is grounded in the following evidence:

- Manufacturers, importers, distributors, and retailers must immediately report
  specified hazardous or noncompliant product information; CPSC says a company
  generally has 24 hours after obtaining reportable information. Late reporting
  can lead to civil or criminal penalties. [CPSC duty-to-report
  guidance](https://www.cpsc.gov/Business--Manufacturing/Recall-Guidance/Duty-to-Report-to-CPSC-Rights-and-Responsibilities-of-Businesses)
- Federal law prohibits selling or offering recalled products for sale, and
  retailers are responsible for monitoring recalls. [CPSC retailer and reverse
  logistics guidance](https://www.cpsc.gov/Business--Manufacturing/Recall-Guidance/Guidelines-for-Retailers-and-Reverse-Logistics-Providers)
- CPSC's recall checklist explicitly spans affected UPC/model/date-code
  identification, inventory isolation, distribution-chain notification,
  remedy selection, reverse logistics, customer communications, and monthly
  progress reporting. [CPSC recall
  checklist](https://www.cpsc.gov/Business--Manufacturing/Recall-Guidance/Recall-Checklist)
- The operational surface is material: in FY2024, CPSC reported more than three
  million e-commerce platform screenings, over 56,000 takedown requests for
  banned or previously recalled products, and removal of more than 58,000
  product units from e-commerce. [CPSC FY2024 Annual Performance Report, pp.
  69-70](https://www.cpsc.gov/s3fs-public/FY-2024-APR-508-compliant.pdf)
- CPSC exposes public recall data as JSON or XML through a REST API, which gives
  a demo a credible live intake/enrichment dependency without tenant-bound
  credentials. [CPSC Recalls API
  information](https://www.cpsc.gov/Recalls/CPSC-Recalls-Application-Program-Interface-API-Information)

**Inference:** these duties and workflow steps create a strong Maestro Flow
story because work must cross regulator data, product records, transactional
systems, human safety judgment, and auditable downstream actions under time
pressure. The cited sources do not establish a retailer's current cycle time,
error rate, or savings.

## Evidence-backed opportunity landscape

### 1. Consumer-product recall containment and remedy

- **Pain point and workflow:** a retailer must recognize reportable hazards,
  identify affected product identifiers, stop distribution, isolate inventory,
  communicate a remedy, and monitor completion. A Fast-Track corrective action
  is intended to begin within 20 working days, while reportable information may
  require notice to CPSC within 24 hours. [CPSC recall
  guidance](https://www.cpsc.gov/Business--Manufacturing/Recall-Guidance/How-to-Conduct-a-Recall)
  [CPSC duty-to-report
  guidance](https://www.cpsc.gov/Business--Manufacturing/Recall-Guidance/Duty-to-Report-to-CPSC-Rights-and-Responsibilities-of-Businesses)
- **Actors:** product safety, legal/compliance, merchandising/catalog, store and
  distribution-center operations, e-commerce operations, customer care,
  finance, supplier, regulator, and affected customers.
- **Systems and evidence:** CPSC API, supplier notices and certificates,
  product information management (PIM), product lifecycle or quality system,
  ERP/WMS, OMS, POS/e-commerce catalog, CRM/loyalty data, contact center, and
  remedy/refund systems. CPSC asks retailers to obtain and review applicable
  Children's Product Certificates or General Certificates of Conformity and to
  retain identifying product information. [CPSC retailer
  responsibilities](https://www.cpsc.gov/FAQ/Retailers-Product-Safety-and-Your-Responsibilities)
- **Controls:** authoritative notice provenance; deterministic identifier
  matching before semantic matching; fail-closed stop-sale; separation between
  recommendation and approval; least-privilege writes; versioned approved
  communications; PII minimization; immutable correlation and decision logs.
  CPSC says its staff must review notices disseminated in connection with a
  recall or recall alert, including recall webpages. [CPSC website notification
  guide](https://www.cpsc.gov/Business--Manufacturing/Recall-Guidance/Website-Notification-Guide-for-Recalling-Companies)
- **Measurable outcomes:** time from notice to first stop-sale; percentage of
  identified affected SKUs and locations blocked; unresolved match count;
  affected/on-hand/in-transit units reconciled; customer-notification coverage;
  remedy participation and completion; overdue action count; report
  completeness. These are proposed measures, not observed baselines.

### 2. Return and refund exception resolution

- **Pain point and workflow:** the 2025 NRF/Happy Returns survey projects $849.9
  billion in retail returns, estimates 19.3% of online sales will be returned,
  and identifies 9% of returns as fraudulent. It also reports that 82% of
  consumers consider free returns important when shopping online. [2025 Retail
  Returns Landscape](https://nrf.com/research/2025-retail-returns-landscape)
  Exceptions require policy interpretation, receipt/order and item evidence,
  fraud signals, disposition, refund authorization, and customer explanation.
- **Actors:** customer, store/contact-center associate, loss prevention, returns
  operations, finance, fraud team, and carrier/reverse-logistics partner.
- **Systems:** OMS, POS, payment/refund gateway, return-management system, fraud
  platform, WMS, carrier tracking, CRM, and product policy/knowledge base.
- **Controls:** deterministic policy gates; reason codes and evidence
  provenance; amount/velocity thresholds; human authorization for denials or
  high-value refunds; payment-data minimization. PCI SSC describes PCI DSS as
  the standard for safe handling of cardholder information. [PCI DSS document
  library](https://www.pcisecuritystandards.org/document_library/?class=pcidss&doc=pci_dss)
- **Measurable outcomes:** exception cycle time, first-contact resolution,
  manual touches, refund-posting time, policy override rate, loss rate, and
  customer recontact rate. These are proposed measures.

### 3. Payment-dispute evidence assembly

- **Pain point and workflow:** dispute response requires condition-specific
  evidence distributed across transaction, delivery, device, identity, and
  customer systems. Visa's merchant guidance lists permissible evidence such as
  delivery to an AVS-matched address, verified customer-profile access, device
  history, and prior undisputed transactions. [Visa Dispute Management
  Guidelines, pp. 51-55](https://by.visa.com/dam/VCOM/global/support-legal/documents/merchants-dispute-management-guidelines.pdf)
- **Actors:** payment operations, fraud analyst, customer service, fulfillment,
  payment processor/acquirer, and card network.
- **Systems:** payment gateway, order/fulfillment systems, carrier proof of
  delivery, identity/device-risk platform, CRM, and acquirer dispute portal.
- **Controls:** dispute-condition-specific evidence allowlist; deadline timer;
  masked account data; source links rather than generated facts; human approval
  before submission; retention aligned to network and legal requirements.
- **Measurable outcomes:** time to complete an evidence package, deadline
  compliance, evidence completeness, manual source lookups, representment rate,
  and outcome rate by dispute reason. Proposed measures must not be represented
  as guaranteed recovery.

### 4. Food traceability and withdrawal scoping

- **Pain point and workflow:** FDA's Food Traceability Rule adds recordkeeping
  requirements for entities that manufacture, process, pack, or hold foods on
  the Food Traceability List. In February 2026, FDA described active work with
  retail food establishments and warehouses on lot-level tracking challenges.
  [FDA February 2026
  update](https://www.fda.gov/food/hfp-constituent-updates/fda-takes-several-actions-related-food-traceability-rule)
  Congress directed FDA not to enforce before July 20, 2028 while the agency
  considers implementation flexibilities. [FDA lot-level tracking discussion
  paper, March 2026](https://www.fda.gov/media/192696/download)
- **Actors:** food-safety lead, supplier, distribution center, store operations,
  quality, regulator, and customer communications.
- **Systems:** supplier EDI/ASN, ERP/WMS, lot/traceability repository, store
  inventory, POS/e-commerce, FDA feeds, and communications.
- **Controls:** key data element and critical tracking event validation; lot
  provenance; exception quarantine; human approval of withdrawal scope; access
  and retention policies; explicit rule-versioning because implementation
  guidance remains active.
- **Measurable outcomes:** trace query response time, lot-event completeness,
  unresolved supplier records, stores/lots contained, units reconciled, and
  notification coverage. These are proposed measures.

### 5. Delayed or unfulfillable order resolution

- **Pain point and workflow:** for covered mail, internet, or telephone sales,
  the FTC rule requires shipment within the advertised time or, absent a stated
  time, within 30 days. A delay requires customer notice and a choice to accept
  or cancel; certain cancellations require a prompt refund. [FTC business guide
  to the Mail, Internet, or Telephone Order Merchandise
  Rule](https://www.ftc.gov/business-guidance/resources/business-guide-ftcs-mail-internet-or-telephone-order-merchandise-rule)
- **Actors:** order management, fulfillment, inventory allocation, customer
  care, finance, carrier, and customer.
- **Systems:** OMS, inventory/ATP, WMS, carrier data, payment/refund gateway,
  notification service, and CRM.
- **Controls:** deterministic promise-date and applicability calculations;
  recorded customer consent; full refund calculation; deadline timers;
  idempotent cancellation/refund; human review for partial or conflicting
  states.
- **Measurable outcomes:** exceptions detected before deadline, response/consent
  capture time, overdue notices, duplicate refunds, customer contacts, and
  order-to-resolution time. These are proposed measures.

## Ranked demo candidates

Scores use a 1-5 scale. Demo value measures visual actor contrast and a clear
hero moment; enterprise credibility measures consequence, named owners,
controls, and evidence; feasibility measures availability of synthetic inputs,
public/mock interfaces, bounded decisions, and a demo-grade build. Equal
weights are intentional. All scores are design inferences from the evidence
above, not empirical ROI or delivery estimates.

| Rank | Candidate | Demo value | Enterprise credibility | Feasibility | Total /15 | Why it ranks here |
| --- | --- | ---: | ---: | ---: | ---: | --- |
| 1 | Consumer-product recall containment and remedy | 5 | 5 | 4 | 14 | Consequential safety decision, public API, documents, real stop-sale branch, human review, parallel containment and communication. ERP/POS interfaces require mocks. |
| 2 | Return and refund exception resolution | 5 | 4 | 5 | 14 | Familiar, high-volume journey with clear customer payoff and easy synthetic data. Policy and fraud decisions vary by retailer, reducing portability. |
| 3 | Food traceability and withdrawal scoping | 4 | 5 | 3 | 12 | Strong cross-company coordination and regulatory narrative. Lot data and evolving implementation guidance make a convincing mock harder. |
| 4 | Payment-dispute evidence assembly | 4 | 4 | 4 | 12 | Excellent evidence-gathering agent and deadline story. Network rules and access to an acquirer portal make the demo narrower and version-sensitive. |
| 5 | Delayed or unfulfillable order resolution | 3 | 4 | 5 | 12 | Highly feasible, measurable orchestration with a clear consumer control. It is less differentiated from conventional rules-based workflow. |

The tie between the first two is resolved in favor of recalls because the
safety, reporting, and stop-sale controls make human authority and Flow's
cross-system coordination more visible. **Inference:** returns may be the better
fallback if the demo audience values customer experience and transaction volume
over safety governance.

## Strongest candidate specification

### Use case and narrative

- **Domain and solution:**
  `retail/product-recall-containment/product-recall-containment-solution/`
- **Enterprise use case:** a product-safety lead must decide which retailer SKUs
  are within a CPSC or supplier recall and prove that affected inventory and
  offers were contained before remedy operations proceed.
- **Audience journey:** the demonstrator submits a recall notice; Flow retrieves
  authoritative recall data, extracts product identifiers, compares them with a
  synthetic catalog, automatically contains exact matches, asks a human to
  resolve ambiguous matches, runs independent containment/remedy work, then
  presents a reconciled audit summary.
- **Out of scope:** determining whether the company has a legal duty to report;
  filing a real Section 15 report; regulator negotiation; generating unapproved
  public recall language; production customer outreach; actual refunds; and
  real POS, ERP, or WMS writes.

### Input contract

The manual demo trigger accepts the following fields. `recallId` is the
correlation and idempotency key; repeat submissions return the existing case
unless `sourceVersion` changed.

```json
{
  "recallId": "26-065",
  "sourceType": "cpsc_api",
  "sourceUrl": "https://www.cpsc.gov/Recalls/...",
  "sourceVersion": "2026-07-16T00:00:00Z",
  "noticeFileName": "recall-notice.pdf",
  "supplierId": "SUP-1042",
  "productIdentifiers": {
    "upcs": ["001234567890"],
    "models": ["D310800CN01"],
    "dateCodes": ["2025-09..2025-10"],
    "lotCodes": []
  },
  "hazard": "fire and burn",
  "remedies": ["refund"],
  "effectiveAt": "2026-07-16T14:00:00Z"
}
```

Required validation: `recallId`, `sourceType`, `sourceUrl`, `sourceVersion`, at
least one product identifier, hazard, remedy, and effective time. Files are
treated as confidential business information in an enterprise implementation;
the demo uses synthetic notice files and public CPSC data only.

### Output contract

Flow returns and persists one `RecallCaseResult`:

```json
{
  "recallId": "26-065",
  "status": "contained",
  "sourceVersion": "2026-07-16T00:00:00Z",
  "matchedSkus": [
    {
      "sku": "RTL-88431",
      "matchType": "model_and_date_code",
      "confidence": 1.0,
      "evidenceRefs": ["notice:model", "catalog:model", "catalog:dateCode"],
      "reviewDecision": "confirmed"
    }
  ],
  "excludedSkus": [],
  "actions": {
    "offersBlocked": 3,
    "locationsQuarantined": 5,
    "unitsOnHand": 41,
    "unitsInTransit": 7,
    "customersEligibleForNotice": 28,
    "notificationsPrepared": 28,
    "remedyCasesOpened": 0
  },
  "exceptions": [],
  "review": {
    "outcome": "confirm_scope",
    "reviewerRole": "product_safety_lead",
    "rationale": "Model and sale date are within notice scope",
    "completedAt": "2026-07-16T14:17:00Z"
  },
  "auditArtifactUrl": "mock://recalls/26-065/audit-summary.json",
  "completedAt": "2026-07-16T14:22:00Z"
}
```

No customer message is sent and no refund is executed in the demo; those nodes
produce preview artifacts and simulated receipts. Counts must be derived from
the synthetic systems, never generated by an agent.

### Flow topology

| Reference segment | Canvas title and actors | Business output | Branch or merge evidence |
| --- | --- | --- | --- |
| Receive and understand | **1. Detect and parse recall**: manual trigger; API workflow reads CPSC JSON; IXP extracts supplier PDF | Normalized recall scope with source links and extraction confidence | Invalid source or missing identifier enters `insufficient_notice_data` below the happy path. |
| Assess and enrich | **2. Match products and exposure**: inline agent uses catalog-search tool; deterministic function verifies identifiers; API workflow queries mock ERP/WMS/OMS | Candidate SKU list, evidence references, confidence, stores, channels, on-hand and in-transit counts | `allExactMatches && extractionConfidence >= 0.90`; anything else goes to human review. |
| Decide and review | **3. Confirm safety scope**: coded action app for product-safety lead; real decision node consumes outcome | Confirmed/excluded SKUs, rationale, and next step | `confirm_scope`, `exclude_candidates`, or `request_supplier_data`; timeout escalates to product-safety director and retains precautionary block. |
| Act and communicate | **4. Contain, notify, and evidence**: parallel RPA stop-sale, inventory quarantine API, coded communication agent, remedy-case API; merge; audit writer | Simulated stop-sale/quarantine receipts, approved message drafts, remedy cases, and reconciled closeout | Three independent branches merge; any failed receipt routes to `containment_incomplete`, not success. |

Use four left-to-right sticky notes with one color per segment. Put validation,
dependency, and timeout exceptions below the happy path. Keep the three
post-decision branches visually symmetric and merge them before reconciliation.

### Actors and tool boundaries

| Actor | Responsibility and contract | Guardrail | Readiness or fallback |
| --- | --- | --- | --- |
| Trigger | Receives `RecallCaseInput`; correlation is `recallId + sourceVersion`. | Reject missing source/provenance. | Manual trigger is sufficient for the demo. |
| IXP | Extracts model, UPC, date/lot code, hazard, remedy, and effective date from a synthetic supplier PDF. | Confidence below 0.90 or conflicting identifiers forces review. | Project/model and deployed folder are unverified; fallback uses fixture JSON with the same schema. |
| API workflow | Reads CPSC JSON and calls mock PIM/ERP/WMS/OMS endpoints. | Read-only until an approved scope exists; responses carry source and retrieval time. | CPSC API is public; retailer endpoints are mocks. |
| RPA | Applies a simulated stop-sale in a legacy POS administration UI and returns per-SKU/channel receipts. | Only confirmed SKU IDs; no free-text navigation; screenshot/error receipt on failure. | UI fixture required. Use RPA because this actor represents a legacy system with no usable API. |
| Inline low-code agent | Maps notice language to candidate catalog items and returns `sku`, `confidence`, `matchReasons`, `evidenceRefs`, `ambiguities`. | Search tool is read-only; cannot approve, block, message, or invent identifiers; deterministic validator overrides it. | Agent/model/tool resources unverified; a fixture output supports offline narration. |
| Coded agent | Produces channel-specific drafts from approved CPSC/supplier language and confirmed scope. | Retrieval-only template library; citations required; no new safety claims; legal/product-safety approval required before use. | Framework, MCP server, and model unverified; fallback uses approved static templates. |
| External agent | Not used. It adds no essential actor contrast until a working vendor connection is verified. | N/A | Record a real connector only during implementation discovery. |
| Human task | Coded action app shows notice excerpts, side-by-side SKU attributes, inventory exposure, confidence, and evidence. Editable scope/rationale; outcomes defined below. | Only designated product-safety roles can complete; all changes logged. | App deployment and action schema unverified; quick form is a reduced fallback. |
| MCP/catalog tool | Least-privilege `search_catalog(recallId, identifiers)` returns source-linked catalog rows only. | No writes; result cap; exact input/output logging; tool-call evaluator. | MCP server unverified; mock API workflow can implement the same contract. |

### Integrations and mocks

| Dependency | Demo contract | Demo implementation | Enterprise replacement |
| --- | --- | --- | --- |
| CPSC Recalls API | Query by recall/product terms; return JSON plus retrieval timestamp. | Live read with cached fixture fallback. | Same API behind approved egress/proxy and monitoring. |
| Supplier notice | Synthetic PDF containing model/UPC/date-code/hazard/remedy. | Fixture file passed to IXP or fixture extractor response. | Supplier portal, email intake, EDI, or quality system. |
| PIM/catalog | Search and read SKU identifiers, descriptions, supplier, and sale dates. | Mock HTTP/API workflow dataset. | Retailer's PIM or master-data API. |
| ERP/WMS/OMS | Read inventory/order exposure; return simulated quarantine/remedy receipts. | Mock HTTP endpoints with deterministic data. | Scoped Integration Service connection or retailer APIs. |
| Legacy POS administration | Stop-sale by confirmed SKU/channel and return receipt. | Local web fixture driven by RPA. | Attended/unattended RPA only if no governed API exists. |
| CRM/notification | Select eligible synthetic customers and create draft jobs. | Mock data and preview-only outbox. | Consent-aware CRM and communication platform. |
| Case/audit store | Persist input version, evidence, decisions, receipts, exceptions, and final result. | Mock service or Orchestrator-backed artifact store. | Retailer quality/recall system of record. |

No tenant resources or connections were verified during research. The local
UiPath CLI is 1.198.0, but the active environment is not authenticated. All
resource names and mock choices above are therefore design contracts, not
claims of deployment readiness.

### Human decisions

The coded action app is the hero experience.

- **Reviewer:** product-safety lead; product-safety director receives timeout
  escalation.
- **Read-only inputs:** recall source/version, extracted notice evidence,
  candidate and exact matches, inventory/order exposure, agent rationale,
  confidence, source links, and current precautionary blocks.
- **Editable `inOut` fields:** included SKU IDs, excluded SKU IDs, lot/date-code
  boundaries, requested supplier information, and decision rationale.
- **Outcomes:** `confirm_scope` runs containment/remedy branches;
  `exclude_candidates` records exclusions but requires a rationale and second
  approval before removing an existing precautionary block;
  `request_supplier_data` opens a supplier task and keeps precautionary blocks.
- **Timeout:** after the retailer-defined safety SLA, escalate while leaving
  affected/ambiguous offers blocked. The demo must not invent this SLA.

### Risk, compliance, and agent controls

| Risk | Required control and visible evidence |
| --- | --- |
| False negative leaves a recalled product for sale | Exact identifiers take precedence; ambiguity fails closed; product-safety review; synthetic case tests include near-match SKU/date codes; stop-sale receipt required before `contained`. |
| False positive blocks safe inventory | Evidence-linked rationale; editable scope; exclusion requires reviewer rationale and, for an existing block, second approval. |
| Agent fabricates identifiers or safety language | Structured schema; catalog tool results are the only allowed identifiers; deterministic validation; approved-language retrieval; citation and tool-trajectory evaluators. |
| Stale or tampered notice | Allowlisted HTTPS source; capture source URL, retrieval time, content hash, and version; reject version rollback; cached fixture clearly labelled. |
| Unauthorized operational write | Service accounts limited by system/action; agent has no write tools; Flow invokes writes only after rule/human gate; secrets stay in managed connections. |
| Customer privacy exposure | Query only confirmed purchasers; minimize fields; separate identity from agent prompt; consent/suppression controls; redact traces and evaluation fixtures. |
| Incomplete containment | Per-system receipts, retry limits, compensating manual task, owner, and `containment_incomplete` state; no optimistic merge. |
| Unapproved public statement | Agent output remains a draft; approved source text and versioned templates; legal/product-safety approval before any enterprise send. |
| AI accountability gap | Log model/prompt/tool versions, input evidence, output, reviewer changes, and outcome. NIST AI RMF calls for documented human-AI roles, oversight, testing, and third-party contingencies. [NIST AI RMF Core](https://airc.nist.gov/airmf-resources/airmf/5-sec-core/) |

This is a workflow-control design, not legal advice. Product-safety counsel must
confirm reporting, notice, retention, privacy, consumer-remedy, and jurisdiction
requirements.

### Observability, evaluation, and success measures

Every node writes `recallId`, `sourceVersion`, actor, start/end time, status,
evidence references, and exception code. The case audit artifact records the
original source hash, extraction, match candidates, deterministic checks,
reviewer edits/outcome, and all action receipts.

Use a synthetic evaluation set named `retail-recall-containment-v1`:

| Case | Expected route and result |
| --- | --- |
| Exact SKU/model/date-code | Straight-through exact match; automatic precautionary containment; all action receipts merge to `contained`. |
| Ambiguous family/model description | Human review; reviewer confirms one SKU and excludes one with rationale; only confirmed SKU proceeds. |
| Missing lot/date code | `request_supplier_data`; ambiguous offers remain blocked; supplier task created; no success state. |
| CPSC API unavailable | Cached fixture is identified as stale/fallback; case requires review; no unsupported live-source claim. |
| POS RPA receipt missing | Other branches may complete, but merge routes to `containment_incomplete` with an owned retry/manual task. |

Evaluation thresholds must be calibrated on the completed implementation. At
minimum, deterministic assertions require zero unconfirmed SKU writes, correct
route and final status for all five cases, and exact reconciliation between
mock source counts and output counts. A tool-trajectory evaluator requires the
catalog-search call when semantic matching is used; an LLM judge checks that
the draft communication contains only approved facts and citations.

Proposed business measures are time to first stop-sale, percentage of affected
offers blocked, reconciliation completeness, unresolved exceptions, customer
notification coverage, remedy completion, and overdue action count. The demo
proves orchestration by showing source-to-receipt lineage for one exact, one
human-reviewed, and one failed-dependency case; it does not claim target
improvements without retailer baselines.

### Reference-solution mapping

| Reference requirement | Retail implementation | Evidence or gap |
| --- | --- | --- |
| Four named segments and canvas rules | Detect and parse; Match products and exposure; Confirm safety scope; Contain, notify, and evidence. | Designed; no `.flow` artifact yet. |
| IXP/document intelligence | Synthetic supplier recall PDF; seven required fields; confidence below 0.90 forces review. | Model/folder unverified; fixture fallback defined. |
| API workflow and RPA on intended path | API reads CPSC and mock retail systems; RPA performs simulated legacy POS stop-sale. | CPSC API available; fixtures and projects not built. |
| Inline agent with wired tool | Product matcher calls least-privilege catalog search and returns structured evidence. | Agent/MCP resources unverified. |
| Coded agent with visible value-add | Evidence-constrained multi-channel communication drafts. | Framework/model unverified; static-template fallback. |
| Real business decision and safe exception | Exact-match expression; ambiguity and failures fail closed. | Expression specified; not implemented. |
| Human decision and returned outcome data | Coded action app returns scope, rationale, and one of three named outcomes. | App/action schema not deployed; quick-form fallback. |
| Purposeful parallelism and merge | Stop-sale, quarantine, communication/remedy branches merge before reconciliation. | Designed; missing receipt blocks success. |
| Evaluation set and evaluator | Five cases; deterministic route/count checks, tool-call evaluator, grounded-draft judge. | Threshold calibration awaits implementation. |
| Process-app variant | Not selected. | Selection belongs to a later cross-domain decision. |
| Solution boundary and delivery | One globally unique `retail-product-recall-containment` package; exactly one `.uipx`; nested Flow project; resource refresh before restore/pack; immutable version; changed-solution CI; publish/deploy only after `main` to Playground `JD_Demos/demos`. | Repository contract inherited; solution not created in this research issue. |

### Design quality rubric

| Dimension | Score /3 | Evidence and remaining gap | Owner and resolution path |
| --- | ---: | --- | --- |
| Enterprise credibility | 2 | Named product-safety decision, regulatory evidence, data and control contract; retailer policy and systems remain unknown. | Product-safety/legal and enterprise architect validate obligations, SLA, systems, and system of record. |
| Flow differentiation | 2 | Four segments, real branch, human review, agent tools, RPA/API contrast, parallel containment and safe merge. | Flow builder verifies supported node contracts and resources. |
| Demo clarity | 2 | One notice-to-receipt narrative with exact, ambiguous, and failure paths. | Demo owner selects fixtures and validates runtime. |
| Build feasibility | 2 | Public API and complete mock contracts; tenant resources and connections unverified. | UiPath developer authenticates to Playground, discovers resources, and builds fixtures. |
| **Total** | **8/12** | **Not implementation-ready because all four dimensions require verified resource or stakeholder evidence to reach the repository's 10/12 threshold.** | **Resolve assumptions below before implementation.** |

## Unresolved assumptions for human review

1. Confirm the target retail segment and jurisdictions. CPSC and FTC evidence is
   US-specific; grocery, apparel, marketplace, and general-merchandise operating
   models differ.
2. Confirm that product-recall containment should outrank returns for the target
   audience; the two candidates tie on the scoring model but tell different
   stories.
3. Name the enterprise recall system of record and owners for catalog, ERP/WMS,
   OMS, POS/e-commerce, CRM, and remedy operations.
4. Provide the retailer's recall/stop-sale policy, decision authority,
   escalation SLA, second-approval rules, and record-retention schedule. No
   timing beyond cited external requirements should be invented.
5. Decide whether exact authoritative identifier matches may initiate a
   precautionary stop-sale before human review and which channels support it.
6. Supply representative, sanitized notice/catalog/inventory/order data and
   edge cases. Confirm whether lot/date-code information is actually available
   at SKU, location, and order level.
7. Confirm allowed customer-contact basis, consent/suppression behavior, PII
   fields, languages, accessibility, and legal approval workflow.
8. Verify UiPath Playground authentication, deployed IXP models, agent/model
   availability, MCP or API catalog tool, action-app deployment, and Integration
   Service connections. The research session was not authenticated.
9. Decide whether a live CPSC call is allowed during demos; retain a clearly
   versioned fixture for network failure and repeatability.
10. Validate that RPA is justified by a real API gap. If the target POS/catalog
    has a governed stop-sale API, use it and choose a different credible UI-only
    step or omit RPA with an approved deviation from the reference pattern.
11. Select business baselines and targets for time to stop-sale, reconciliation,
    notification, remedy completion, and exception aging. No ROI or improvement
    claim is supported yet.
12. Confirm whether this domain is one of the three later Data Fabric/process-app
    variants. This proposal currently marks that variant not selected.

## Research method and source limits

The research reviewed regulator guidance and data from CPSC, FDA, FTC, the NIST
AI RMF, Visa merchant rules, PCI SSC standards, and original 2025 retail-return
survey results from NRF and Happy Returns. Government and standards-owner
sources were preferred for workflows and controls. NRF findings are
survey-based industry estimates, not government statistics or a forecast for a
specific retailer. Vendor marketing benchmarks, internal metrics, ROI,
pack-hours, and invented system readiness were excluded.

Because no retailer's internal systems or behavioral data were in scope, this
report identifies strategic/design opportunities rather than the discovery
skill's Tier 1-3 internal findings or replicable models. The ranked scores and
the conclusion that Flow is a good fit are explicitly design inferences. Before
implementation, the adopting retailer must validate each workflow against its
own policy, data, controls, volumes, and outcome baselines.
