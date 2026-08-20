# Retail product recall containment demo specification

## Use case and narrative

- **Domain and solution:** `retail/product-recall-containment/product-recall-containment-solution/`, with globally unique solution and package name `retail-product-recall-containment`.
- **Enterprise use case:** A product-safety lead determines which synthetic retailer SKUs fall within a CPSC or supplier recall. Flow correlates authoritative notice data, extracted supplier evidence, catalog identifiers, and inventory exposure; applies precautionary blocks to exact matches; and stops ambiguous scope for human review. The hero moment is the reviewer comparing notice evidence with candidate SKU and lot/date-code data, correcting the proposed scope, and watching the returned decision drive simulated stop-sale, quarantine, notification, and remedy work.
- **Why this use case:** It is the top-ranked candidate in the [retail opportunity research](agentic-workflow-opportunities.md), scoring 14/15. It combines a consequential safety decision, public regulator data, document intelligence, bounded agent reasoning, API and UI-system work, human authority, parallel containment, and an auditable outcome. It wins the tie with returns because the safety and stop-sale controls make Flow's orchestration and fail-closed behavior more visible.
- **Audience journey:** The demonstrator submits a synthetic recall notice, follows four named canvas segments, opens the evidence-rich review task for an ambiguous candidate, edits and confirms the affected scope, then shows parallel simulated containment and remedy work merging into a reconciled audit summary. Flow is the right surface because the value lies in coordinating deterministic matching, IXP, agents, API and RPA work, and human judgment while keeping normal and exception routes legible.
- **Success outcome:** Every confirmed SKU and affected channel has the required simulated containment receipt, ambiguous products remain precautionarily blocked until reviewed, customer and remedy artifacts are previews only, and the case closes with source-to-decision-to-receipt lineage.
- **Measurable value:** Pilot measures are time to first stop-sale, percentage of affected offers blocked, reconciliation completeness, unresolved match count, customer-notification coverage, remedy completion, and overdue action count. The demo does not claim improvement or ROI without retailer baselines.
- **Out of scope:** Determining or filing a legal duty-to-report submission; regulator negotiation; production customer outreach or refunds; real POS, PIM, ERP, WMS, OMS, CRM, or quality-system writes; unapproved public recall language; non-synthetic customer data; and Data Fabric/process-app scope, which decision #56 confirmed is not selected for this domain.

## Personas

| Persona | Role in the demo | Authority boundary |
| --- | --- | --- |
| Product-safety lead | Reviews ambiguous product scope and selects the containment outcome. | May confirm or exclude candidate SKUs and request supplier data through the named human task; cannot publish a notice or remove an existing precautionary block alone. |
| Product-safety director | Owns overdue reviews and second approval for removing an existing block. | May approve an exclusion or reassign an overdue case; does not bypass evidence or receipt reconciliation. |
| Store and e-commerce operations | Act on approved containment scope and investigate technical exceptions. | Operate only on confirmed SKU/channel identifiers; cannot change recall scope. |
| Demonstrator | Submits fixtures and narrates evidence, review, containment, and recovery. | Uses public CPSC data plus synthetic notices, catalog, inventory, orders, and customer records only. |

## Trigger and case contract

The initial build uses a manual trigger for repeatability. A scheduled CPSC poll or supplier event may replace it after intake ownership is decided. `<recallId>:<sourceVersion>` is the idempotency key. An exact replay returns the existing case; a newer source version appends a version event and re-evaluates scope without erasing prior decisions or receipts.

| Item | Specification |
| --- | --- |
| Trigger | Manual Flow trigger accepting `RecallCaseInput`. The `recall-intake-gateway` API workflow validates input, retrieves or reads the authoritative source, and emits the canonical case. |
| Canonical record | Orchestrator queue `RetailProductRecallCases`, unique reference `<recallId>:<sourceVersion>`. Specific content stores input, evidence, candidate scope, and current state; output data stores the final result and receipts. |
| Required inputs | `recallId`, `sourceType`, `sourceUrl`, `sourceVersion`, `noticeFileName`, `supplierId`, `productIdentifiers`, `hazard`, `remedies`, and `effectiveAt`. At least one UPC, model, date code, or lot code is required. |
| Sensitive data | Public CPSC data and synthetic supplier, catalog, inventory, order, and customer fixtures. Customer identity is excluded from agent prompts and masked in tasks, logs, and evaluations. |
| Outputs | Confirmed and excluded SKU scope, evidence references, reviewer outcome, simulated containment/remedy receipts, preview communications, exception state, audit artifact, and final status. |

### Input schema

| Field | Type | Required | Contract |
| --- | --- | --- | --- |
| `recallId` | string | yes | Public or synthetic recall identifier. |
| `sourceType` | enum | yes | `cpsc_api` or `supplier_notice`. |
| `sourceUrl` | HTTPS URL | yes | Allowlisted provenance for the notice. |
| `sourceVersion` | ISO 8601 string | yes | Prevents silent rollback and versions case evidence. |
| `noticeFileName` | string | conditional | Synthetic supplier PDF name when document extraction is used. |
| `supplierId` | string | yes | Synthetic supplier identifier. |
| `productIdentifiers` | object | yes | Arrays `upcs`, `models`, `dateCodes`, and `lotCodes`; at least one value is required. |
| `hazard` | string | yes | Source-grounded hazard summary. |
| `remedies` | array | yes | Source-grounded remedy types such as `refund`, `repair`, or `replacement`. |
| `effectiveAt` | ISO 8601 string | yes | Starts containment aging and escalation measurement. |

### Output schema

| Field | Type | Contract |
| --- | --- | --- |
| `status` | enum | `review_required`, `awaiting_supplier_data`, `contained`, `containment_incomplete`, or `technical_exception`. |
| `matchedSkus` | array | SKU, match type, confidence, evidence references, and review decision. |
| `excludedSkus` | array | SKU, evidence references, reviewer rationale, and second-approval reference when required. |
| `exposure` | object | Source-derived store, channel, on-hand, in-transit, and eligible-customer counts. |
| `review` | object | Named outcome, included/excluded IDs, edited lot/date boundaries, rationale, reviewer, approver when applicable, and timestamps. |
| `actionReceipts` | array | System, operation, SKU/channel scope, status, correlation ID, and timestamp. |
| `communicationPreviews` | array | Channel, approved template/version, grounded draft, citations, and safety-check status; never a send receipt. |
| `exceptions` | array | Typed code, owning role, recovery condition, and current state. |
| `auditEvents` | array | Actor, action, source references, input/output hashes, rule/model/prompt/tool versions, and timestamp. |
| `auditArtifactUrl` | string | Mock or approved storage URI for the reconciled closeout artifact. |

## Flow topology

Use four blue sticky notes with the titles below. The happy path runs left to right; validation, extraction, matching, timeout, and dependency exceptions sit below their originating segment. Independent actions are visually symmetric and merge before reconciliation. A separately labelled `External agent showcase` branch sits below segment 2 and never receives recall-case data.

| Reference segment | Domain-specific canvas title | Actors and business output | Branch or merge evidence |
| --- | --- | --- | --- |
| Receive and understand | **1. Detect and parse the recall** | Manual trigger invokes `recall-intake-gateway`; IXP extracts a synthetic supplier PDF when present; the queue item is created or read. Output: normalized `RecallCase`. | Invalid provenance, schema failure, source rollback, or missing identifiers routes to `insufficient_notice_data`; IXP confidence below `0.90` sets `requiresScopeReview = true`. |
| Assess and enrich | **2. Match products and exposure** | Deterministic rules verify exact identifiers. Inline `Recall Product Matcher` uses read-only catalog search for ambiguous descriptions. API workflow reads mock PIM, ERP, WMS, OMS, and customer eligibility data. | `allMatchesExact === true && extractionConfidence >= 0.90 && conflictingIdentifiers.length === 0` permits precautionary containment; all other candidate scope enters human review. The external showcase branch rejoins with `RecallCase` unchanged. |
| Decide and review | **3. Confirm the safety scope** | Exact matches retain a precautionary block. Ambiguous cases open `Product Recall Scope Review`, a coded action app for the product-safety lead. Output: included/excluded SKU IDs, boundaries, rationale, and named outcome. | `review.outcome` is `ConfirmScope`, `ExcludeCandidates`, or `RequestSupplierData`. Exclusion of an already blocked SKU requires `secondApproval.status === "approved"`. Timeout escalates while blocks remain. |
| Act and communicate | **4. Contain, remedy, and evidence** | In parallel: RPA applies simulated legacy-POS stop-sale; API workflow creates mock quarantine and remedy receipts; coded `Recall Communication Writer` creates grounded previews. The branches merge, receipts reconcile, and the queue item closes. | `requiredContainmentReceipts.every(receipt => receipt.status === "succeeded")` permits `contained`. A missing or ambiguous required receipt sets `containment_incomplete`, keeps the item open, and creates an owned recovery task. |

## Agentic reasoning and tool use

| Actor | Visible responsibility | Structured input and output | Tools, context, and guardrails | Readiness/fallback |
| --- | --- | --- | --- | --- |
| Inline low-code agent: `Recall Product Matcher` | Map ambiguous notice language to candidate catalog rows and explain the match. | Input: normalized notice identifiers/text and catalog-search scope. Output: `sku`, `confidence`, `matchReasons`, `evidenceRefs`, and `ambiguities`. | Read-only `search_retail_catalog(recallId, identifiers, query)` tool; identifiers must come from tool rows; no block, exclusion, message, refund, or invented-identifier authority. Deterministic rules override the recommendation. | Not provisioned. Use versioned fixture responses with the same schema. Tool or grounding failure returns `insufficient_evidence` and forces review. |
| Coded agent: `Recall Communication Writer` | Produce channel-specific preview copy from confirmed scope and approved recall language. | Input: recall ID, confirmed product/remedy facts, channel, audience type, and template ID. Output: subject/body, `templateId`, `templateVersion`, citations, and safety findings. | LangGraph agent calls `get_approved_recall_template` through a least-privilege UiPath MCP server, then checks for unsupported safety claims, missing remedy terms, and prohibited customer data. It has no send tool. | Not provisioned. Use approved static templates when the tool is unavailable; failure creates a manual-draft task without blocking already completed containment. |
| External agent showcase: `Azure AI Foundry connectivity` | Display external-agent connectivity without performing recall work. | Connection-selected `agent_id` and constant message `UiPath Flow external-agent connectivity showcase for retail`; omit `thread_id`. Discard the response. | Node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread`, connection `0107247a-0197-42c9-b957-05d1b722b111`. Do not pass recall, supplier, product, inventory, customer, reviewer, or receipt data. | Reverified August 12, 2026 in `uipathlabs / Playground`: node available; connection enabled and default in folder `demos`. The branch is disabled by default; timeout or error records transient status and rejoins the unchanged route. |

## Data and resources

| Resource or data dependency | Purpose and contract | Owner/folder/readiness | Security and failure handling |
| --- | --- | --- | --- |
| IXP project: `RetailRecallNoticeExtraction` | Extract UPC, model, date/lot codes, hazard, remedy, and effective date from a synthetic supplier PDF. | New project/model; target child folder beneath `JD_Demos/demos` not chosen; not provisioned. | Synthetic files only. Confidence below `0.90`, missing identifiers, or conflict with CPSC data forces review; fixture JSON preserves the output contract offline. |
| API workflow: `recall-intake-gateway` | Operations `normalize_notice`, `read_exposure`, and `record_actions` read public/fixture data and return deterministic mock receipts. | New sibling project; CPSC live-read permission and retail mock endpoint remain unverified. | Capture source URL, timestamp, hash, and version. Use cached fixture fallback marked as stale; typed failures never fabricate live evidence or success receipts. |
| RPA: `legacy-pos-stop-sale` | Operation `apply_stop_sale` writes confirmed SKU/channel scope to a local POS-admin fixture and returns per-scope receipts. | New sibling project targeting a local web fixture; unattended runtime not provisioned. | UI automation is used only to demonstrate a legacy API gap. No free-text navigation; ambiguous writes are not retried; screenshot/error reference routes to reconciliation. |
| Queue: `RetailProductRecallCases` | Canonical demo case, idempotency, state transitions, outputs, and ordered audit events. | New Orchestrator queue beneath the solution deployment folder; not provisioned. | Least-privilege robot access, masked customer identifiers, retention chosen during provisioning, and version-aware duplicate handling. |
| Mock retail systems | Versioned synthetic PIM/catalog, ERP/WMS/OMS, and customer-eligibility rows plus deterministic quarantine/remedy endpoints. | Repository fixtures and local mock service to be built. | Counts come only from source rows; agents cannot change them. Failures create typed exceptions and preserve precautionary blocks. |
| Azure AI Foundry connection | Supplies the non-material external-agent node with a constant message and discarded response. | Shared connection `0107247a-0197-42c9-b957-05d1b722b111`; enabled/default in Playground folder `demos`. | No case or sensitive data, no output mapping into business variables, fail-open continuation, and Flow-specific binding validation during implementation. |
| MCP template server | Exposes only `get_approved_recall_template(templateId, channel, remedyType)`. | New demo MCP server or local mock; product-safety/legal owner approves templates; not provisioned. | No send or customer lookup capability. Failure produces a manual-draft task. |
| Coded action app: `Product Recall Scope Review` | Side-by-side notice evidence, candidate catalog rows, exposure, editable scope/boundaries, rationale, and named outcomes. | Deployed independently and referenced by Flow; not provisioned. | Role restricted, all edits logged, no operational credentials, and second-approval contract for removing an existing block. |

### Solution boundary and layout

```text
retail/product-recall-containment/
├── product-recall-containment-solution/
│   ├── retail-product-recall-containment.uipx
│   ├── product-recall-containment-flow/
│   │   └── product-recall-containment-flow.flow
│   ├── recall-intake-gateway/
│   ├── legacy-pos-stop-sale/
│   └── recall-communication-writer/
└── product-recall-scope-review/
```

The solution has exactly one `.uipx` manifest and is independently deployable. The coded action app remains independently deployed if required by the platform, with its versioned contract pinned by solution configuration. Before packaging, run `uip solution resources refresh`, restore dependencies, and dry-run pack. Pull requests validate only the changed solution; publish and deploy occur only after merge to `main` through `playground-deploy`, with an immutable package version and a child folder beneath `JD_Demos/demos`.

## Human decisions

- **Scope review:** The product-safety lead sees source/version/hash, notice excerpts, extracted fields/confidence, deterministic matches, agent candidates/rationale, catalog evidence, exposure counts, and current precautionary blocks.
- **Editable values:** Included and excluded SKU IDs, lot/date-code boundaries, requested supplier fields, and rationale. Source facts, counts, and receipts remain read-only.
- **Outcomes:** `ConfirmScope` authorizes actions for included SKUs; `ExcludeCandidates` records exclusions and requires second approval before removing an existing block; `RequestSupplierData` opens a supplier task and retains precautionary blocks.
- **Timeout and resumption:** The timeout value is demo configuration pending retailer policy. Expiry escalates to the product-safety director and keeps blocks in place. Flow resumes from the task completion handle, validates returned IDs/outcome, persists reviewer edits, and routes only from the returned value.
- **Downstream authority:** Only validated confirmed scope enables RPA/API writes. No agent can approve scope, remove a block, publish a notice, contact a customer, or issue a refund.

## Controls and safety

| Control | Design decision | Evidence in the Flow or demo |
| --- | --- | --- |
| Fail-closed matching | Exact authoritative identifiers take precedence; ambiguity, low extraction confidence, conflicts, and missing evidence force review while precautionary blocks remain. | Deterministic expressions, exception routes below the happy path, and near-match evaluation fixtures. |
| Human authority | Agents recommend or draft only. Scope changes require named task outcomes; removal of an existing block requires rationale and second approval. | Completion handle, returned field IDs, outcome decision, and approver reference. |
| Data and access | Synthetic retail/customer data, least-privilege identities, read-only agent tools, managed connections, task roles, and masked traces. | Fixture inventory, connection bindings, tool schemas, task roles, and redaction assertions. |
| Grounded communication | Drafts use approved source text and versioned templates; citations and safety checks are mandatory; no send tool exists. | MCP trajectory evaluator, grounded-output evaluator, and preview-only artifact type. |
| Receipt truth | Required containment actions return per-scope receipts; an ambiguous write is never treated as success or retried automatically. | Receipt merge and reconciliation expression gate `contained`. |
| External showcase isolation | Azure AI Foundry receives constants only and cannot affect recall state, scope, routing, actions, receipts, or status. | No business-variable mappings from the node and an on/off/failing isolation evaluation. |

## Error paths and recovery

| Failure | Safe route | Recovery owner and condition |
| --- | --- | --- |
| Invalid source, schema, version rollback, or missing identifier | Set `insufficient_notice_data`; do not invoke match or write actors. | Product-safety intake corrects or rejects the notice, then resubmits with the same version-aware key. |
| IXP low confidence or CPSC/supplier conflict | Preserve evidence and force scope review. | Product-safety lead resolves scope or requests supplier data. |
| Catalog tool unavailable or uncited agent output | Set `insufficient_evidence`; ignore the recommendation and use deterministic facts in review. | Platform owner restores the tool; reviewer may proceed from source-linked facts. |
| Human task timeout | Keep precautionary blocks and assign the case to the product-safety director. | Director reassigns, approves, or closes under the demo runbook. |
| API/RPA read failure | Retry transient reads at most twice, then set `technical_exception`. | Platform/RPA owner restores the dependency and retries from the failed activity. |
| Ambiguous or failed containment write | Do not retry automatically or create a success receipt; set `containment_incomplete`. | Operations reconciles target state and explicitly resumes or completes manual action. |
| Template or draft safety-check failure | Create a manual-draft task; containment receipts remain recorded. | Product-safety/legal owner supplies approved copy. |
| Azure AI Foundry timeout, error, or unexpected response | Record transient `externalAgentShowcaseStatus`, discard response, and rejoin the same core route. | Demo owner may disable the branch; business users take no recovery action. |

## Observability and evaluation

| Signal or test | What it proves | Expected result or threshold |
| --- | --- | --- |
| Correlation and audit | Notice version, evidence, matching, reviewer changes, actions, and recovery can be reconstructed. | Every trace, task, and receipt contains `recallId` and `sourceVersion`; the queue preserves ordered audit events and no unmasked customer identity. |
| Flow route evaluator | Each fixture reaches the correct safe route and final status. | 5/5 initial synthetic cases match expected review route and status before promotion. |
| Match safety evaluator | No unconfirmed or excluded SKU reaches a write actor. | Zero unconfirmed SKU writes across all cases; exact source/output count reconciliation. |
| Tool trajectory evaluator | Semantic matching and communication use only their required read-only tools. | Catalog tool called for every semantic-match case; template tool called for every draft; zero unapproved calls. |
| Grounded draft evaluator | Preview text contains only approved facts and citations. | Every material claim cites notice/template evidence; unsupported safety claims and customer sends occur zero times. |
| External showcase isolation | The placeholder external agent cannot change business results. | With the branch off, on, or failing, identical `RecallCase`, route, scope, receipts, and status; only transient showcase status may differ. |
| Receipt reconciliation | Completion never hides missing or ambiguous containment. | `contained` only when every required per-scope receipt is present and `succeeded`. |

### Synthetic evaluation set

Dataset name: `retail-recall-containment-v1`.

| Case | Expected route | Expected business output |
| --- | --- | --- |
| Exact model and date code | Deterministic exact-match path with precautionary containment. | Required mock stop-sale/quarantine receipts reconcile and status is `contained`. |
| Ambiguous product family | Scope review; reviewer includes one SKU and excludes one with rationale. | Only confirmed SKU reaches action branches; returned edits appear in audit events. |
| Missing lot/date code | `RequestSupplierData` from review. | Supplier task created, ambiguous offers remain blocked, and status is `awaiting_supplier_data`. |
| CPSC API unavailable | Clearly marked cached-fixture fallback and required review. | No unsupported live-source claim; evidence records fallback version/hash. |
| Missing POS RPA receipt | Reconciliation exception after other branches finish. | No false success; status is `containment_incomplete` with owned recovery data. |

## Demo script

1. Show the four-segment canvas and the exception routes below the happy path.
2. Submit the ambiguous-family fixture and point out `recallId`, `sourceVersion`, source hash, and extracted confidence.
3. Open the match trace to show deterministic checks, the read-only catalog tool, evidence references, ambiguity, and exposure counts.
4. Open `Product Recall Scope Review`, compare the notice and catalog attributes, include one SKU, exclude one, refine the date-code boundary, enter rationale, and select `ConfirmScope`.
5. Return to Flow and show simulated POS stop-sale, inventory quarantine/remedy, and grounded communication-preview branches running independently and merging.
6. Open the queue record to show reviewer edits, template-tool call, per-scope receipts, final status, and ordered audit events.
7. Point out the disabled `External agent showcase` branch. Show its constant input, discarded response, and isolation from recall variables.
8. Preview the missing-receipt fixture to show `containment_incomplete` and the owned manual-reconciliation route.

## Success measures

- **Business proof:** The demo exposes time to first stop-sale, percentage of affected offers blocked, count reconciliation, unresolved scope, notification coverage, remedy completion, and action aging as measurable pilot signals without promising a target improvement.
- **Flow proof:** A viewer can see document intelligence, deterministic checks, two material agent roles, the non-material Azure AI Foundry showcase, API/RPA contrast, real business routing, human authority, parallel actions, merge, and safe recovery.
- **Demo proof:** In under ten minutes, a viewer can verify notice provenance, evidence-linked matching, reviewer correction, downstream scope consumption, per-system receipts, and an incomplete-containment exception.
- **Build proof:** Every project validates, the five-case evaluation set passes, bindings are recorded, `resources refresh` and dry-run pack succeed, and the immutable package can follow changed-solution CI into Playground after merge.

## Reference mapping

| Reference requirement | Domain-specific implementation | Evidence or gap |
| --- | --- | --- |
| 3-4 segment topology and canvas rules | Four blue sticky notes: Detect and parse the recall; Match products and exposure; Confirm the safety scope; Contain, remedy, and evidence. | Fully specified; canvas implementation remains. |
| IXP/document intelligence | `RetailRecallNoticeExtraction` extracts seven required fields; low confidence or conflict forces review. | Contract and fixture fallback specified; model/folder remain to provision. |
| API workflow and RPA on the intended path | `recall-intake-gateway` handles notice/exposure/actions; `legacy-pos-stop-sale` applies simulated UI-only blocks. | Contracts specified; projects, fixtures, and API-gap validation remain. |
| Inline agent with a wired tool | `Recall Product Matcher` calls `search_retail_catalog` and returns branch-relevant evidence and ambiguities. | Contract specified; model/tool resources remain unverified. |
| Coded agent with visible value-add | `Recall Communication Writer` retrieves an approved template and produces grounded previews with safety findings. | Contract specified; coded agent and MCP server remain to build. |
| Shared external-agent showcase | Verified Azure AI Foundry node/connection on a constant-input, discarded-output, fail-open branch. | Node and connection are tenant-ready; Flow binding and selected agent remain to validate. |
| Real business decision and safe exception | Exact-match/conflict expression plus named human outcomes; missing evidence, timeout, and failed receipt remain safe. | Fully specified; expressions remain to implement and evaluate. |
| Human decision and returned outcome data | Coded action app returns included/excluded IDs, boundaries, rationale, reviewer, approval, and outcome; Flow consumes them downstream. | Contract specified; retailer authority/SLA remain human decisions. |
| Purposeful parallelism and merge | Stop-sale, quarantine/remedy, and communication-preview branches merge before receipt reconciliation. | Fully specified; required receipt matrix remains to encode. |
| Evaluation set and evaluator | Five fixtures plus route, match-safety, trajectory, grounded-draft, isolation, and reconciliation checks. | Exact initial thresholds specified; fixtures/evaluators remain to build. |
| Process-app variant | Not selected. The Orchestrator queue stays the canonical demo record. | Closed on August 20, 2026 by decision #56, which selected commercial banking, healthcare provider, and life insurance. No open dependency remains. |
| Solution boundary and delivery contract | One `retail-product-recall-containment` solution, one `.uipx`, nested Flow layout, resource refresh, immutable version, changed-solution CI. | Fully specified; deployment child folder and remaining resources require provisioning. |

## Open human decisions

These decisions refine implementation but do not block a synthetic, mock-backed build.

| Decision | Owner | Resolution path |
| --- | --- | --- |
| Confirm the target retail segment and jurisdictions. | Demo portfolio owner and product-safety counsel | Approve the US general-merchandise framing or update regulator sources, policy, and fixtures. |
| Confirm precautionary-block and exclusion authority. | Product-safety and operations-control owners | Approve exact-match auto-block rules, editable fields, second approval, and unblock conditions. |
| Confirm escalation timing and system of record. | Product-safety owner and enterprise architect | Replace the unset demo timeout and approve the queue or enterprise recall system as canonical. |
| Validate source data and lot/date-code availability. | Catalog, supply-chain, and quality owners | Supply sanitized edge cases and confirm identifier availability across product, location, and order data. |
| Validate the POS API gap and RPA responsibility. | POS owner and enterprise architect | Retain RPA only if no governed stop-sale API exists; otherwise choose another credible UI-only action or approve a reference deviation. |
| Approve customer/privacy and communication controls. | Privacy, legal, CRM, and accessibility owners | Confirm eligible fields, consent/suppression, languages, templates, and retention before non-synthetic testing. |
| Choose live CPSC versus cached-fixture demo behavior. | Security/network and demo owners | Approve egress or make the versioned fixture the default with optional live refresh. |
| Select the exact `JD_Demos/demos` child folder and provision resources. | UiPath tenant administrator | Reuse the verified Azure connection; provision IXP, queue, agents, action app, runtime, and record every binding. |
| Set pilot baselines and target measures. | Product-safety operations owner | Supply observed timing, reconciliation, coverage, remedy, and aging baselines before making benefit claims. |

## Implementation tasks

1. Scaffold `retail-product-recall-containment` with the nested Flow project and one `.uipx` manifest.
2. Build the five-case fixture set, canonical queue contract, retail-system mocks, and local POS-admin UI.
3. Implement and validate the API workflow, IXP extraction contract, and RPA stop-sale operation with typed errors and receipts.
4. Build the inline matcher, read-only catalog tool, deterministic verifier, and structured-output evaluation.
5. Build the coded communication writer, least-privilege template MCP tool, and trajectory/grounding evaluators.
6. Build and deploy the coded action app, then wire its completion handle, outcomes, edits, and second approval into Flow.
7. Author the four-segment Flow, exception routes, precautionary-block logic, non-material Azure showcase, parallel action branches, merge, and reconciliation.
8. Validate the Azure binding and prove the showcase cannot change case data, scope, routing, actions, receipts, or final status.
9. Run project validation and the five-case evaluation set; resolve every warning and failed threshold.
10. Refresh solution resources, restore, dry-run pack, and register immutable deployment configuration.

## Quality rubric

| Dimension | Score (0-3) | Evidence and remaining gap | Owner and resolution path |
| --- | ---: | --- | --- |
| Enterprise credibility | 2 | Consequential safety decision, roles, data contract, controls, and measures are explicit; retailer policy, authority, systems, and baselines remain unverified. | Product-safety/legal and system owners validate during discovery. |
| Flow differentiation | 3 | Four segments visibly coordinate IXP, deterministic logic, agents, API, RPA, human decisions, parallel containment, merge, and safe recovery. | Flow implementer preserves the specified topology and validates expressions. |
| Demo clarity | 3 | Ambiguous-scope hero journey, exact-match path, failed-receipt exception, and observable proof points form a timed script. | Demo owner selects representative fixtures and rehearses after deployment. |
| Build feasibility | 2 | Inputs, outputs, mocks, fallbacks, solution boundary, evaluation, authenticated target, and one verified connection are recorded; most resources remain unprovisioned. | Tenant administrator and implementers provision resources and record bindings. |
| **Total** | **10/12** | **Ready for implementation planning as a synthetic demo; not ready for production integration, public communication, or real containment.** | **Start with solution/fixtures, then close owned human decisions before replacing mocks.** |
