# Commercial banking payment exception demo specification

## Use case and narrative

- **Domain and solution:** `commercial-banking/payment-exception/payment-exception-solution/`, with globally unique solution and package name `commercial-banking-payment-exception`.
- **Enterprise use case:** A payment-operations analyst resolves a synthetic ISO 20022 payment investigation. The Flow correlates the investigation with the original payment, network status, customer-risk context, screening result, and approved operating policy. The consequential decision is whether to request information, approve a routine repair, return or reject the payment, or escalate it to compliance. The hero moment is a reviewer correcting the proposed repair in a coded action app and the Flow consuming those returned fields to create mock network and legacy-system receipts.
- **Why this use case:** It is the top-ranked candidate in the [commercial-banking opportunity research](agentic-workflow-opportunities.md), scoring 14/15. It combines a time-sensitive operational exception, structured payment data, bounded agent reasoning, API and UI-system work, human authorization, external communication, and an auditable outcome in one legible Flow.
- **Audience journey:** The demonstrator submits a synthetic investigation, follows four named canvas segments, opens the evidence-rich review task, edits and approves a safe repair, and then shows parallel network response, legacy write-back, communication drafting, and audit completion. Flow is the right surface because the value lies in coordinating deterministic logic, agents, RPA, API work, and human authority while making normal and exception routes visible.
- **Success outcome:** The case reaches the correct safe status with cited evidence, an explicit human outcome where required, and reconciled mock receipts. No agent releases, recalls, repairs, returns, rejects, or clears a live payment.
- **Out of scope:** Live Swift, Fedwire, CHIPS, payment-hub, sanctions, customer-master, or case-system connectivity; real customer or watchlist data; autonomous movement of funds; production policy interpretation; cancellation/recall processing; Data Fabric/process-app scope; and quantified ROI without a design-partner baseline.

## Personas

| Persona | Role in the demo | Authority boundary |
| --- | --- | --- |
| Payment operations reviewer | Reviews evidence and selects the operational outcome. | May request information, approve an allowed repair, or return/reject only through the named human task; cannot clear a compliance hold. |
| Sanctions/AML reviewer | Resolves possible screening matches or incomplete high-risk party data. | May release the case back to operations, maintain the hold, or request further investigation. |
| Payment operations queue manager | Owns overdue and technical-exception queues. | Reassigns work and coordinates recovery; does not override required maker-checker or compliance decisions. |
| Demonstrator | Submits fixtures and narrates evidence, routing, review, and completion. | Uses synthetic data and mock write-backs only. |

## Trigger and case contract

The initial build uses a manual trigger for repeatable demonstration. A webhook can replace it after a non-production channel is selected. `caseId` plus `uetr` is the idempotency key; a repeated submission returns the existing case and appends a replay audit event.

| Item | Specification |
| --- | --- |
| Trigger | Manual Flow trigger accepting the `PaymentExceptionInput` object below. The API workflow validates the raw `camt.110`-like investigation and produces the canonical case. |
| Canonical record | Orchestrator queue `CommercialBankingPaymentExceptions`, with unique reference `<caseId>:<uetr>`. Queue specific-content stores the synthetic case and current state; output data stores the final decision and receipts. The deployed folder is a child of `JD_Demos/demos` chosen during implementation. |
| Required inputs | `caseId`, `uetr`, `investigationMessage`, `originalPayment`, `networkStatus`, `customerRiskContext`, `screeningResult`, `policyVersion`, and `receivedAt`. All values are synthetic; account and party identifiers are masked in tasks and logs. |
| Outputs | Classification, deterministic findings, cited evidence summary, advisory recommendation, reviewer outcome, `camt.111`-like response, mock write-back receipts, final status, and append-only audit events. |

### Input schema

| Field | Type | Required | Contract |
| --- | --- | --- | --- |
| `caseId` | string | yes | Synthetic case identifier. |
| `uetr` | UUID string | yes | Correlates payment and investigation events. |
| `investigationMessage` | object | yes | Raw and parsed `camt.110`-like reason, requester, requested fields, and timestamps. |
| `originalPayment` | object | yes | Synthetic `pacs.008`-like amount, currency, parties, agents, remittance data, and value date. |
| `networkStatus` | object | yes | Mock status, timestamps, and prior-message references. |
| `customerRiskContext` | object | yes | Synthetic customer ID, expected-activity summary, risk tier, and jurisdiction flags. |
| `screeningResult` | object | yes | Mock status, possible-match reason, list version, and timestamp. |
| `policyVersion` | string | yes | Pins the approved operating-policy corpus used by retrieval. |
| `receivedAt` | ISO 8601 string | yes | Starts case aging and timeout measurement. |

### Output schema

| Field | Type | Contract |
| --- | --- | --- |
| `classification` | enum | `missing_information`, `data_mismatch`, `status_request`, `possible_duplicate`, `screening_escalation`, or `other`. |
| `validationFindings` | array | Rule ID, affected field, actual value, expected constraint, and pass/fail. |
| `evidenceSummary` | object | Bounded narrative, source IDs, policy section citations, and confidence. |
| `requiresCompliance` | boolean | Forces the compliance route when true. |
| `recommendedAction` | enum | `respond_with_status`, `request_information`, `approve_repair`, `return_or_reject`, or `escalate_compliance`; advisory only. |
| `reviewOutcome` | object | Reviewer ID, named outcome, edited repair fields, rationale, and timestamp. |
| `responseMessage` | object | Synthetic `camt.111`-like response or information request. |
| `writeBackReceipts` | array | Mock system, operation ID, status, and timestamp. |
| `finalStatus` | enum | `awaiting_information`, `repair_approved`, `returned_or_rejected`, `compliance_hold`, `completed`, or `technical_exception`. |
| `auditEvents` | array | Actor, action, source references, input/output hashes, rule/model/prompt version, and timestamp. |

## Flow topology

Use four blue sticky notes with the titles below. The happy path runs left to right. Schema, matching, compliance, timeout, and dependency exceptions sit below their originating segment. Independent completion work is visually symmetric and merges before the final case update. A separately labelled `External agent showcase` branch sits below segment 2 so the Azure AI Foundry icon and connection are visible without making the placeholder agent part of the payment investigation.

| Reference segment | Domain-specific canvas title | Actors and business output | Branch or merge evidence |
| --- | --- | --- | --- |
| Receive and understand | **1. Receive and correlate** | Manual trigger invokes `payment-message-gateway` API workflow to validate and normalize the message, creates or reads the queue item, and invokes `legacy-payment-console` RPA to retrieve the UI-only status. Output: canonical `PaymentExceptionCase`. | Invalid schema, duplicate `<caseId>:<uetr>`, unmatched UETR, or RPA read failure routes to manual intake or technical exception below the happy path. |
| Assess and enrich | **2. Investigate the payment** | Deterministic script evaluates field and status rules. Inline `Payment Exception Investigator` uses read-only case, policy-search, customer-context, and screening tools to create the cited evidence summary and recommendation. A separate Azure AI Foundry node demonstrates external-agent connectivity with static metadata only. | `showExternalAgentShowcase === true` enters the non-material showcase branch and rejoins with the canonical case unchanged. Core routing then uses `requiresCompliance === true`; otherwise, `classification === "status_request" && evidenceSummary.confidence >= 0.85` takes the safe status-response path and all other cases continue to operations review. The confidence threshold is demo configuration, not a production policy claim. |
| Decide and review | **3. Control the resolution** | Safe status enquiries use a bounded no-repair route. Other cases open the `Payment Exception Review` coded action app with facts, findings, citations, recommendation, and editable repair fields. A separate quick-form compliance task handles screening cases. | Named outcomes route with `reviewOutcome.outcome`: `RequestInformation`, `ApproveRepair`, `ReturnOrReject`, or `EscalateCompliance`. Low confidence always enters operations review. A 30-minute demo timeout escalates to the queue manager. Only human outcomes enable write operations. |
| Act and communicate | **4. Respond and close** | In parallel: API workflow produces a mock network receipt; RPA records an approved repair in the mock legacy console when applicable; coded `Payment Update Writer` retrieves an approved template and drafts a safe status message. The branches merge, receipts reconcile, and the queue item closes. | `requiredReceipts.every(receipt => receipt.status === "succeeded")` completes the case. Any missing or failed required receipt sets `technical_exception`, leaves the item open, and routes it to the queue manager. |

## Agentic reasoning and tool use

| Actor | Visible responsibility | Structured input and output | Tools, context, and guardrails | Readiness/fallback |
| --- | --- | --- | --- | --- |
| Inline low-code agent: `Payment Exception Investigator` | Correlate evidence and produce a bounded, cited recommendation. | Input: canonical case, deterministic findings, `policyVersion`. Output: `classification`, `evidenceSummary`, `requiresCompliance`, `recommendedAction`, `confidence`. | Read-only tools: `get_customer_risk_summary`, `get_screening_result`, and `search_payment_policy`. Policy search is pinned to the approved version. Every material claim needs a source ID; unavailable evidence yields `insufficient_evidence`; web search and write tools are prohibited. | Not provisioned. Build with local synthetic tool responses and a repository policy fixture; bind approved tenant resources later. Retrieval failure bypasses the recommendation and still permits manual review. |
| Coded agent: `Payment Update Writer` | Draft one internal or customer-safe case update after the human decision. | Input: case ID, outcome, non-restricted facts, recipient type, and response deadline. Output: subject, body, `templateId`, `templateVersion`, and safety findings. | LangGraph agent calls `get_approved_payment_message_template` through a least-privilege UiPath MCP server, then self-checks for restricted sanctions rationale, unsupported promises, and missing required fields. A trajectory evaluator requires the template call. | Not provisioned. Use a local approved-template fixture and mock MCP tool during build. Tool failure routes to a manual-draft task; no message is sent automatically. |
| External agent showcase: `Azure AI Foundry connectivity` | Display the Azure AI Foundry node and prove that Flow can bind an external-agent connection; it performs no payment work. | Inputs are a connection-selected `agent_id` and the constant `message` `UiPath Flow external-agent connectivity showcase for commercial banking`; omit `thread_id`. The response is discarded and is never mapped into `PaymentExceptionCase`. | Node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` with Azure AI Foundry connection `0107247a-0197-42c9-b957-05d1b722b111`. Do not pass `caseId`, UETR, payment, customer, screening, policy, reviewer, or receipt data. The node has no authority to influence routing or write-backs. | Verified on August 12, 2026: the node is tenant-available and the connection is enabled and default in UiPath Labs Playground folder `demos`. Each Flow must still validate its binding. The branch is disabled by default, and timeout or failure records only a transient showcase status before rejoining the unchanged core route. |

## Data and resources

| Resource or data dependency | Purpose and contract | Owner/folder/readiness | Security and failure handling |
| --- | --- | --- | --- |
| IXP/document intelligence | Not used: the chosen intake is structured ISO 20022-like data, so adding document extraction would be ornamental. Attachments remain out of scope. | Not applicable. | If future investigation attachments become material, define fields and confidence review before adding IXP. |
| API workflow: `payment-message-gateway` | Operation `parse_investigation` validates and normalizes input; operation `send_response` writes a mock `camt.111`-like record and returns a receipt. Flow invokes it in segments 1 and 4. | New sibling project in the solution; connector/HTTP target is a local mock endpoint. Tenant connection and target folder are unverified. | Schema errors return typed validation findings. Calls use the idempotency key, two bounded retries for transient failures, and a technical-exception result without fabricated success. |
| RPA: `legacy-payment-console` | Operation `read_status` retrieves the UI-only payment status; `record_repair` writes only human-approved repair fields and returns a receipt. UI automation is purposeful because the demo console exposes no API. | New sibling project targeting a local mock console; production application and unattended runtime are unverified. | Synthetic data only. Selector or application errors return a typed exception and screenshot reference; no retry occurs after an ambiguous write. |
| Queue: `CommercialBankingPaymentExceptions` | Canonical demo case, unique reference, state transitions, and final outputs. | New Orchestrator queue beneath the solution deployment folder; not provisioned. | Least-privilege robot access, masked identifiers in logs, and retention set during environment provisioning. Duplicate references read the existing record. |
| Context index: `CommercialBankingPaymentPolicy` | Versioned synthetic operating procedure used by `search_payment_policy`. | New context index; corpus owner is the payment-policy owner; not provisioned. | Only approved synthetic policy content. Retrieval failure produces `insufficient_evidence` and forces human review. |
| Static screening and customer tools | Return versioned synthetic risk and screening summaries. | Local mocks initially; compliance and customer-data owners must approve any real connection. | Read-only, minimum fields, no raw watchlist data, and compliance hold on ambiguity or failure. |
| Azure AI Foundry external-agent connection | Supplies `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` on the non-material showcase branch. Its request contains only a selected agent and constant message; its response is discarded. | Shared connection `0107247a-0197-42c9-b957-05d1b722b111`; verified enabled and default in UiPath Labs Playground folder `demos` on August 12, 2026. | No case or sensitive data, no business-variable mapping, and fail-open continuation to the same merge point. Validate the node binding during Flow implementation. |
| MCP template server | Exposes only `get_approved_payment_message_template(templateId, recipientType)`. | New demo MCP server or local mock; messaging owner approves templates; not provisioned. | No send capability. Tool inputs exclude raw account identifiers and screening rationale; failure creates a manual-draft task. |
| Coded action app: `Payment Exception Review` | Evidence-rich task UI with before/after values, findings, citations, recommendation, and returned outcome contract. | Deployed independently and referenced by Flow; not provisioned. | Role-restricted access, masked identifiers, required rationale, server-side outcome validation, and no direct payment-system credentials. |

### Solution boundary and layout

```text
commercial-banking/payment-exception/
├── payment-exception-solution/
│   ├── commercial-banking-payment-exception.uipx
│   ├── payment-exception-flow/
│   │   └── payment-exception-flow.flow
│   ├── payment-message-gateway/
│   ├── legacy-payment-console/
│   └── payment-update-writer/
└── payment-exception-review/
```

The solution has exactly one `.uipx` manifest and is an independent deployment boundary. The coded action app remains independently deployed if required by the platform, while its contract and version are pinned by the solution configuration. Before packaging, run `uip solution resources refresh`, restore dependencies, and run a dry-run pack. Pull requests validate only the changed solution. Publishing and deployment occur after merge to `main` through `playground-deploy`, using an immutable package version and a child folder beneath `JD_Demos/demos`.

## Human decisions

- **Reviewer and decision:** A payment operations reviewer decides `RequestInformation`, `ApproveRepair`, `ReturnOrReject`, or `EscalateCompliance`. The task shows original and proposed values, deterministic findings, payment/network status, policy citations, confidence, and agent rationale.
- **Task experience:** The coded action app receives read-only case facts and evidence; exposes only policy-allowed repair fields as editable `inOut` values; requires reviewer rationale; and returns outcome, edits, reviewer ID, and timestamp. High-risk fields are not editable. A second-person approval field is required when the selected policy configuration marks the repair maker-checker controlled.
- **Compliance task:** Possible matches, incomplete high-risk party data, or an explicit escalation create a role-restricted quick form. Outcomes are `ReleaseToOperations`, `MaintainHold`, or `InvestigateFurther`; the agent cannot select or execute release.
- **Timeout and resumption:** A 30-minute demo timeout assigns the case to the queue manager and leaves it open. On completion, Flow resumes from the task handle, validates the returned outcome and field IDs, persists the decision, and routes only from the returned value.
- **Downstream routes:** `RequestInformation` creates a mock information request and sets `awaiting_information`; `ApproveRepair` enables the RPA repair branch; `ReturnOrReject` creates a mock return/reject response; `EscalateCompliance` or `MaintainHold` sets `compliance_hold`; `ReleaseToOperations` returns to the operations task without performing a write.

## Controls and safety

| Control | Design decision | Evidence in the Flow or demo |
| --- | --- | --- |
| Routing safety | Deterministic validation precedes agent reasoning. Compliance or confidence rules force review; write paths require a validated human outcome. | Decision expressions use `requiresCompliance`, `evidenceSummary.confidence`, and `reviewOutcome.outcome`; exception paths are below the happy path. |
| Access and data | Use synthetic data, least-privilege service identities, masked identifiers, read-only enrichment tools, role-restricted tasks, and mock write targets. | Connection bindings, task roles, mock endpoint configuration, and redacted trace fields are visible during implementation review. |
| Agent boundaries | Domain agents recommend and draft only. They cannot clear screening, authorize an outcome, mutate a payment, or send a message. The Azure AI Foundry showcase agent receives only constants, and its output is discarded. Claims need source IDs; missing evidence triggers manual review. | Structured output schemas, allowlisted tools, prohibited-action prompt rules, required human task handles, and no output edge from the showcase node into case variables. |
| Resilience | Use idempotency, typed errors, bounded transient retries, no retry after ambiguous writes, receipt reconciliation, and an owned technical-exception queue. | `<caseId>:<uetr>` queue reference, retry configuration, receipt merge, and queue-manager route. |

## Error paths and recovery

| Failure | Safe route | Recovery owner and condition |
| --- | --- | --- |
| Invalid message schema or unmatched UETR | Create a manual-intake item; do not invoke agents or write systems. | Payment operations corrects or rejects intake, then resubmits with the same idempotency key. |
| Policy retrieval unavailable or uncited agent output | Set `insufficient_evidence` and bypass the recommendation. | Payment operations reviews raw facts; platform owner restores the index before agent use resumes. |
| Screening ambiguity or dependency failure | Set `requiresCompliance = true` and create the compliance task. | Sanctions/AML reviewer selects a named outcome. |
| Human task timeout | Keep the queue item open and assign it to the queue manager. | Queue manager reassigns or cancels according to the demo runbook. |
| API or RPA read failure | Retry transient reads at most twice, then set `technical_exception`. | Platform or RPA owner resolves the dependency and retries from the failed activity. |
| Ambiguous or failed write | Do not retry automatically and do not create a success receipt. | Queue manager reconciles target state and explicitly resumes or closes the case. |
| Communication template or safety-check failure | Create a manual-draft task; other successful receipts remain recorded. | Messaging owner supplies an approved draft before merge completion. |
| Azure AI Foundry showcase timeout, error, or unexpected response | Record only `externalAgentShowcaseStatus` in transient trace data, discard the response, and rejoin the same core route. | Demo owner may disable the branch; payment operations takes no recovery action because business state is unchanged. |

## Observability and evaluation

| Signal or test | What it proves | Expected result or threshold |
| --- | --- | --- |
| Correlation and audit | Case state, deterministic findings, agent citations, reviewer edits/outcome, tool versions, and receipts can be reconstructed. | Every trace, task, and receipt contains `caseId` and `uetr`; the queue item contains an ordered audit-event list with no unmasked account identifier. |
| Flow route evaluator | The principal business claim: each case reaches the correct safe route and final status. | 5/5 initial synthetic cases match the expected review route and final status before promotion. |
| Evidence evaluator | Recommendations are bounded by supplied facts and the pinned policy. | 100% of material claims contain valid source IDs; prohibited unsupported actions occur 0 times. |
| Tool-use/trajectory evaluator | The coded writer uses the approved-template tool and no send tool. | Required template-tool call in 5/5 applicable runs; no unapproved tool calls. |
| External showcase isolation | The placeholder external agent cannot change a business decision or case result. | With the showcase flag off, on, or failing, the same fixture produces identical `PaymentExceptionCase`, route, receipts, and final status; only transient `externalAgentShowcaseStatus` may differ. |
| Receipt reconciliation | Completion never hides failed or ambiguous work. | `completed` occurs only when every required receipt is present and `succeeded`; dependency-failure cases end in `technical_exception`. |

### Synthetic evaluation set

Dataset name: `commercial-banking-payment-exception-v1`.

| Case | Expected route | Expected business output |
| --- | --- | --- |
| Complete status enquiry | Deterministic safe status-response route; bypass operations review. | Cited status summary, mock response receipt, safe message draft, and `completed`. |
| Missing creditor identifier | Operations review with `recommendedAction = request_information`. | Named requested field, due date, mock information request, and `awaiting_information`. |
| Policy-allowed remittance repair | Operations review with editable low-risk field and `ApproveRepair`. | Reviewer edit consumed, API and RPA receipts reconciled, and `repair_approved`. |
| Possible sanctions match | Compliance route regardless of recommendation confidence. | No repair/write action, compliance task created, and `compliance_hold` until human disposition. |
| Payment-hub timeout | Technical exception below segment 4. | No false receipt, queue item remains open, and queue-manager recovery data is present. |

## Demo script

1. Show the four-segment canvas and explain that exception routes sit below the happy path.
2. Submit the policy-allowed remittance-repair fixture and point out the shared `caseId` and UETR.
3. Open the investigation trace to show deterministic findings, read-only tool calls, policy citations, confidence, and the advisory recommendation.
4. Open `Payment Exception Review`, compare original and proposed remittance values, edit the proposed value, enter rationale, and select `ApproveRepair`.
5. Return to the Flow and show the API response, RPA repair, and message-drafting branches running independently and merging.
6. Open the queue record to show the reviewer edit, template-tool call, mock receipts, final status, and ordered audit events.
7. Point out the disabled `External agent showcase` branch and its Azure AI Foundry icon. Show that its input is static, its output is discarded, and enabling or failing the branch cannot change the payment route.
8. Run or preview the sanctions fixture to show that `requiresCompliance` overrides confidence and prevents payment write-back.

## Success measures

- **Business proof:** The demo makes reduced evidence-assembly and handoff time measurable through median/p90 resolution time, queue age, human touch time, correct-triage rate, override/reopen rate, and evidence completeness. These are pilot measures, not promised savings.
- **Flow proof:** A viewer can see deterministic checks, two material agent roles, the non-material Azure AI Foundry showcase, API and RPA responsibilities, business-value routing, human authority, parallel follow-up, merge, and recovery without opening generic plumbing nodes.
- **Demo proof:** In under ten minutes, a viewer can verify the trigger, cited recommendation, reviewer correction, downstream consumption, reconciled receipts, and compliance exception.
- **Build proof:** Each project validates, the five-case evaluation set passes, every resource binding is recorded, `resources refresh` and dry-run pack succeed, and the solution is deployable with an immutable version through the changed-solution CI path.

## Reference mapping

| Reference requirement | Domain-specific implementation | Evidence or gap |
| --- | --- | --- |
| 3-4 segment topology and canvas rules | Four blue sticky notes: Receive and correlate, Investigate the payment, Control the resolution, Respond and close; happy path left-to-right and exceptions below. | Fully specified; canvas implementation remains. |
| IXP/document intelligence, when relevant | Not used because the selected intake is structured ISO 20022-like data and attachments are out of scope. | Rationale recorded; revisit only if attachments become a required input. |
| API workflow and RPA on the intended path | `payment-message-gateway` parses/sends; `legacy-payment-console` reads status and records an approved repair. | Contracts specified; mock endpoint and UI remain to build. |
| Inline agent with a wired tool | `Payment Exception Investigator` uses customer, screening, and pinned-policy tools and returns branch-driving fields. | Contract specified; resources are unverified. |
| Coded agent with visible value-add | `Payment Update Writer` retrieves an approved template, drafts a safe update, and self-checks it. | Contract specified; coded agent and template server remain to build. |
| Shared external-agent showcase | Node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` uses connection `0107247a-0197-42c9-b957-05d1b722b111` on a demo-flag branch with a static message, discarded output, and fail-open rejoin. | Node availability and connection state are verified in Playground; the Flow-specific binding and agent selection remain to validate during implementation. |
| Real business decision and safe exception | Compliance/confidence expression plus named human outcomes; schema, hold, timeout, and technical routes are explicit. | Fully specified; expressions remain to implement and evaluate. |
| Human decision and returned outcome data | Coded action app returns named outcome, edits, rationale, reviewer, and timestamp; Flow consumes them downstream. | Contract specified; maker-checker policy is an open human decision. |
| Purposeful parallelism and merge | Network response, applicable RPA write-back, and safe message draft run independently, then merge before closure. | Fully specified; receipt requirements vary by outcome and remain to encode. |
| Evaluation set and evaluator | Five synthetic cases, route/evidence/trajectory/reconciliation evaluators, and exact initial thresholds. | Fully specified; fixtures and evaluators remain to build. |
| Solution boundary and delivery contract | `commercial-banking/payment-exception/payment-exception-solution/`, one `.uipx`, nested Flow layout, resource refresh, immutable version, changed-solution CI. | Fully specified; deployment folder child and resources require provisioning. |

## Open human decisions

These decisions refine implementation but do not block building the synthetic, mock-backed demo.

| Decision | Owner | Resolution path |
| --- | --- | --- |
| Choose the representative payment rail and final message subset. | Commercial payments product owner | Approve Swift cross-border, Fedwire, CHIPS, or internal-hub framing before replacing the generic fixtures. |
| Confirm repairable fields and maker-checker rules. | Payment operations control owner | Review the editable-field allowlist and identify fields requiring second-person approval. |
| Confirm sanctions disposition roles and customer-safe wording. | Sanctions/AML and legal owners | Approve task outcomes, restricted rationale rules, and message templates. |
| Confirm response deadlines and escalation SLA. | Payment operations owner | Replace the 30-minute demo timeout with the approved test policy for the selected exception types. |
| Choose representative systems and prove whether UI automation is necessary. | Enterprise architect and application owners | Review API availability; retain RPA only for a demonstrably UI-only responsibility. |
| Approve policy corpus, retention, residency, and redaction. | Policy, privacy, and records owners | Sign off the synthetic corpus and environment controls before non-synthetic testing. |
| Select the exact `JD_Demos/demos` child folder and provision remaining resources. | UiPath tenant administrator | Reuse the verified Azure AI Foundry connection, discover or create the other approved resources, and record every binding in the solution. |
| Decide whether commercial banking is one of the three Data Fabric/process-app variants. | Demo portfolio owner | Resolve in the later cross-domain selection issue; this initial spec marks the variant not selected. |

## Implementation tasks

1. Scaffold `commercial-banking-payment-exception` with the nested Flow project and one `.uipx` manifest.
2. Build the synthetic fixture set, canonical queue contract, mock payment endpoint, and mock legacy console.
3. Implement and validate the API workflow and RPA operations with typed errors and receipts.
4. Build the inline investigator, versioned policy context, and structured-output evaluation.
5. Build the coded update writer, least-privilege template MCP tool, and trajectory/safety evaluators.
6. Build and deploy the coded action app, then wire its completion handle and returned fields into Flow.
7. Author the four-segment Flow, exception routes, non-material Azure AI Foundry showcase branch, parallel completion branches, merge, and audit updates.
8. Bind and validate Azure AI Foundry connection `0107247a-0197-42c9-b957-05d1b722b111`; prove the showcase branch cannot change case data, routing, receipts, or final status.
9. Run project validation and the five-case evaluation set; resolve all warnings and failed thresholds.
10. Refresh solution resources, restore, dry-run pack, and register immutable-version deployment configuration.

## Quality rubric

| Dimension | Score (0-3) | Evidence and remaining gap | Owner and resolution path |
| --- | ---: | --- | --- |
| Enterprise credibility | 2 | Consequential role, decision, data contract, controls, and pilot measures are explicit; no design-partner baseline or verified estate exists. | Product and operations owners validate the workflow and baseline during discovery. |
| Flow differentiation | 3 | Four segments visibly coordinate deterministic logic, agents, API, RPA, human decisions, parallel work, merge, and safe recovery. | Flow implementer preserves the specified layout and validates route expressions. |
| Demo clarity | 3 | The repair hero journey and sanctions exception have named, observable proof points and a timed script. | Demo owner rehearses both fixtures after deployment. |
| Build feasibility | 2 | Inputs, outputs, mocks, fallbacks, solution boundary, evaluation, and delivery are specified; tenant resources are not verified. | UiPath tenant administrator and implementers provision and record resource readiness. |
| **Total** | **10/12** | **Ready for implementation planning as a synthetic demo; not ready for production integration or deployment until owned resource gaps are resolved.** | **Start with solution and fixture scaffolding, then close the listed human decisions before replacing mocks.** |
