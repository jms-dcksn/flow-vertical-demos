# Consumer banking EFT dispute resolution demo specification

## Use case and narrative

- **Domain and solution:** `consumer-banking/eft-dispute-resolution/eft-dispute-resolution-solution/`, with globally unique solution and package name `consumer-banking-eft-dispute-resolution`.
- **Enterprise use case:** A disputes analyst resolves a synthetic electronic-fund-transfer error report covering debit-card, ACH, ATM, or P2P transactions. Flow assembles source-linked evidence, calculates the applicable demo deadline variant deterministically, presents an advisory investigation packet, stops for an authorised person, and then coordinates only the reviewed financial and communication actions.
- **Why this use case:** EFT dispute investigation and resolution is the top-ranked candidate in the [consumer-banking opportunity research](opportunity-research.md), scoring 14/15. It has the strongest combination of public evidence, a time-bound and consequential decision, multi-system evidence, bounded agent assistance, human authority, account action, consumer communication, and a safe synthetic implementation. Mortgage loss mitigation is similarly compelling but depends more heavily on investor-specific rules and foreclosure timing; complaint response is easier to mock but less differentiated; identity exceptions risk overstating what an agent may decide.
- **Audience journey:** The demonstrator submits a synthetic mailed dispute form and transaction fixture, follows four named canvas segments, opens a source-linked review experience, corrects the proposed route and credit instruction, and then shows mock financial, case, and notice work reconciling into one audit record. Flow is the right surface because it makes deterministic rules, IXP, API and RPA work, two bounded agents, human judgement, parallel actions, and recovery paths legible together.
- **Hero moment:** The disputes analyst compares the consumer statement, transaction and authentication evidence, deadline plan, agent rationale, and conflicts in the `EFT Dispute Review` coded action app; edits the proposed outcome; supplies a reason; and watches Flow consume the returned field IDs before any mock credit or notice action can occur.
- **Success outcome:** The case reaches the expected safe status with cited evidence, a deterministic deadline plan, an explicit human outcome, route-specific mock receipts, and a complete audit record. No agent determines legal coverage, approves or denies a claim, posts a live credit, files a fraud report, or sends an unreviewed consumer notice.
- **Out of scope:** Credit-card Regulation Z disputes; a universal interpretation of Regulation E; socially engineered but consumer-authorised payment adjudication; criminal-fraud or SAR decisions; live account, payment-rail, fraud, customer, notification, or records systems; production policy; real customer data; Data Fabric/process-app scope; and quantified return without a bank baseline.

## Personas

| Persona | Role in the demo | Authority boundary |
| --- | --- | --- |
| Consumer | Supplies the synthetic error notice and supporting evidence. | Provides facts but does not interact with internal tools or approve the resolution. |
| Contact-centre or branch representative | Captures the notice and confirms the referenced transactions. | May correct intake fields; cannot classify legal coverage or authorise account action. |
| Disputes analyst | Reviews the investigation packet and selects the operational outcome. | Owns `ApproveCredit`, `DenyWithReason`, `RequestInformation`, or `EscalateCompliance` through the human task; cannot override a compliance hold. |
| Compliance or quality reviewer | Resolves ambiguous coverage, clock, conflicting-evidence, or exception-to-policy cases. | May return the case to the analyst, request evidence, or maintain escalation; cannot let an agent substitute for bank policy. |
| Disputes queue manager | Owns timed-out and technical-exception work. | Reassigns and coordinates recovery; does not bypass required review or dual control. |
| Demonstrator | Runs fixtures and narrates the route, evidence, review, receipts, and audit proof. | Uses synthetic data, repository mocks, and sandbox notification previews only. |

## Trigger and case contract

The initial build uses a manual Flow trigger for repeatable demonstrations. A non-production intake webhook may replace it after its owning channel is selected. `caseId` is the correlation and idempotency key; a duplicate input reads the existing queue item, records a replay event, and never repeats a financial action.

| Item | Specification |
| --- | --- |
| Trigger | Manual Flow trigger accepting `EftDisputeInput`. The `dispute-evidence-gateway` API workflow validates the input, normalises transaction and evidence references, and produces the canonical case. |
| Canonical record | Orchestrator queue `ConsumerBankingEftDisputes`, unique reference `caseId`. Queue specific-content stores the synthetic case and state; output data stores the reviewed decision, receipts, notifications, and final audit summary. Deploy the solution to a child folder beneath the approved `JD_Demos/demos` parent. |
| Required inputs | Notice metadata, tokenised customer and account references, disputed transactions, untrusted consumer statement, and scanned synthetic attachments with immutable evidence metadata. |
| Outputs | Source-linked case summary, advisory coverage and risk assessments, deterministic deadline plan, human decision, optional credit instruction, notification records, system receipts, final status, and audit record. |

### Input schema

| Field | Type | Required | Contract |
| --- | --- | --- | --- |
| `caseId` | string | yes | Synthetic correlation and idempotency value. |
| `receivedAt` | ISO 8601 datetime | yes | Starts the case clock and preserves the source timezone. |
| `intakeChannel` | enum | yes | `phone`, `branch`, `digital`, or `mail`. |
| `noticeForm` | enum | yes | `oral` or `written`; supplied to deterministic confirmation logic. |
| `customerRef` | string | yes | Tokenised customer reference, never a raw identity value. |
| `accountRef` | string | yes | Tokenised deposit-account reference. |
| `statementSentAt` | ISO 8601 date or null | conditional | Required by the configured deadline rule when statement timing is material. |
| `firstDepositAt` | ISO 8601 date or null | conditional | Supports deterministic new-account variants; absence cannot be guessed by an agent. |
| `transactions` | array | yes | Each item contains transaction ID, timestamp, amount, currency, rail/type, tokenised counterparty, location, and allegation. |
| `consumerStatement` | string | yes | Preserved as untrusted source text; derived facts are stored separately with evidence IDs. |
| `attachments` | array | no | URI, media type, SHA-256 hash, classification, malware-scan state, and fixture version. |
| `policyVersion` | string | yes | Pins the synthetic dispute procedure and deterministic rule configuration. |
| `showExternalAgentShowcase` | boolean | yes | Defaults to `false`; controls only the non-material Azure showcase branch. |

### Output schema

| Field | Type | Contract |
| --- | --- | --- |
| `caseSummary` | object | Disputed total, source-linked facts, `missingEvidenceCount`, missing-information details, conflicts, and extraction confidence. |
| `coverageAssessment` | object | Candidate error type, rationale, evidence IDs, confidence, and `needsPolicyReview`; advisory only. |
| `deadlinePlan` | object | `status`, next action, due date, rule variant, calendar version, and source fields, produced only by deterministic logic. |
| `riskAssessment` | object | Authentication and transaction signals, evidence IDs, confidence, contradiction flags, and escalation reasons. |
| `reviewDecision` | object | Named outcome, corrected fields, rationale, reviewer ID, approval reference, and timestamp, set by an authorised person. |
| `creditInstruction` | object or null | Amount, `provisional` or `final` status, effective date, approval reference, and idempotency key; populated only on an allowed reviewed route. |
| `notifications` | array | Approved notice type, immutable template/content version, preview or sandbox delivery state, and receipt. |
| `systemReceipts` | array | Mock system, operation, status, transaction reference, and timestamp. |
| `finalStatus` | enum | `awaiting_information`, `provisional_credit_recorded`, `approved_credit_recorded`, `denied`, `compliance_review`, `completed`, or `technical_exception`. |
| `auditRecord` | object | Input and evidence hashes, rule/model/prompt/tool versions, agent tool calls, reviewer data, routes, timestamps, and receipts. |

## Flow topology

Use four blue sticky notes with the titles below. The happy path runs left to right; intake, evidence, review, timeout, and dependency exceptions sit directly below their originating segment. The post-review action branches are visually symmetric and merge before receipt reconciliation and audit closure. A separate grey `External agent showcase` note sits below segment 2 and cannot contribute data to the dispute route.

| Reference segment | Domain-specific canvas title | Actors and business output | Branch or merge evidence |
| --- | --- | --- | --- |
| Receive and understand | **1. Capture the disputed EFT** | Manual trigger invokes `dispute-evidence-gateway`, creates or reads the queue item, sends a scanned fixture to `ConsumerBankingDisputeIntakeExtraction`, and uses `legacy-deposit-adjustment-console.read_account_status` for a purposeful UI-only mock lookup. Output: canonical `EftDisputeCase`. | Invalid schema, unsafe attachment, extraction confidence below `0.85`, missing required clock inputs, duplicate case, or RPA read failure routes below the happy path to corrected intake or technical review. The `0.85` extraction threshold is demo configuration, not bank policy. |
| Assess and enrich | **2. Build the investigation packet** | Deterministic logic calculates the deadline plan. Inline `Dispute Evidence Organizer` uses read-only policy and transaction tools to create the cited advisory coverage assessment. Coded `Dispute Packet Reconciler` calls one read-only MCP tool to reconcile authentication and transaction signals. | `coverageAssessment.needsPolicyReview === true || caseSummary.missingEvidenceCount > 0 || riskAssessment.contradictionFlags.length > 0` forces the compliance/evidence route. `showExternalAgentShowcase === true` enters a non-material Azure branch that rejoins with `EftDisputeCase` unchanged. |
| Decide and review | **3. Review the resolution** | Cases with a complete deterministic plan open the `EFT Dispute Review` coded action app. Coverage or clock ambiguity first opens a role-restricted compliance quick form, then returns to analyst review or remains escalated. | Flow routes only on `reviewDecision.outcome`: `ApproveCredit`, `DenyWithReason`, `RequestInformation`, or `EscalateCompliance`. A 30-minute demo timeout assigns the open case to the queue manager; this is demonstrator configuration, not a regulatory SLA. Only a validated reviewed outcome can enable a write branch. |
| Act and communicate | **4. Resolve, notify, and evidence** | In parallel, the RPA records any approved mock credit, the API workflow writes case status, and an approved-template operation creates a notice preview. Branches merge; required receipts reconcile; then the sandbox notification and audit closure run. | `requiredReceipts.every(receipt => receipt.status === "succeeded")` permits completion. A missing, duplicate, failed, or ambiguous required receipt sets `technical_exception`, leaves the item open, and routes to the queue manager without fabricating success. |

## Actor inventory

| Actor | Contract and readiness |
| --- | --- |
| Trigger | Manual `EftDisputeInput` event; `caseId` is correlation/idempotency; target is a child folder beneath the approved `JD_Demos/demos` parent. |
| IXP | `ConsumerBankingDisputeIntakeExtraction` consumes a scanned synthetic dispute form and extracts consumer/account tokens, transaction IDs/dates/amounts, received date, allegation, and signature-presence flag with evidence locations. Confidence below `0.85` or a required-field conflict forces corrected intake. No model is provisioned; a versioned extraction fixture is the build fallback. |
| API workflow | `dispute-evidence-gateway` operations `normalise_intake`, `get_transaction_evidence`, `write_case_status`, `render_notice`, and `deliver_notice_preview` return typed results and receipts. It is a new sibling project backed by repository mocks; no live bank connection is assumed. |
| RPA | `legacy-deposit-adjustment-console` operations `read_account_status` and `record_credit` target a local legacy-screen simulator with no API. `record_credit` accepts only reviewed instruction fields and returns a receipt; ambiguous writes are never retried automatically. The production API gap is unverified. |
| Inline agent | `Dispute Evidence Organizer` receives the canonical case, deterministic findings, and policy version; calls `search_dispute_procedure` and `lookup_transaction_evidence`; and returns `coverageAssessment`. It is advisory, must cite evidence IDs, and has no write, web-search, credit, or decision tool. It is not provisioned; synthetic local responses are the fallback. |
| Coded agent | LangGraph `Dispute Packet Reconciler` receives tokenised transaction and channel evidence, calls `get_authentication_signal_summary`, and returns `riskAssessment`. It cannot label fraud truth or decide coverage. The project and evaluation set are not provisioned; a deterministic mock tool supports local build. |
| External agent | `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` uses shared connection `0107247a-0197-42c9-b957-05d1b722b111`, a connection-selected `agent_id`, constant message `UiPath Flow external-agent connectivity showcase for consumer banking`, and no `thread_id`. The response is discarded. On August 12, 2026, CLI 1.199.0 confirmed the node tenant-available and the connection enabled/default in Playground folder `demos`; each Flow binding still requires validation. |
| Human task | `EFT Dispute Review` coded action app shows source-linked facts and allows controlled edits; a separate compliance quick form owns policy ambiguity. Neither is provisioned. A quick-form analyst fallback may unblock Flow wiring, but the coded action app remains the hero contract. |
| MCP server/tool | Least-privilege `get_authentication_signal_summary(transactionIds, accountRef, asOf)` returns versioned synthetic device, login, MFA, and channel signals. A trajectory evaluator requires exactly one bounded call in each reconciliation case. It has no mutation or raw-identity access and is not provisioned. |

## Agentic reasoning and tool use

| Actor | Visible responsibility | Structured input and output | Tools, context, and guardrails | Readiness/fallback |
| --- | --- | --- | --- | --- |
| Inline low-code agent: `Dispute Evidence Organizer` | Organise evidence and propose a cited candidate error category without making the legal or financial decision. | Input: `EftDisputeCase`, validation findings, transaction evidence, `policyVersion`. Output: `coverageAssessment` with candidate type, rationale, evidence IDs, confidence, and `needsPolicyReview`. | Read-only `search_dispute_procedure` and `lookup_transaction_evidence`; context corpus pinned to `policyVersion`. Consumer text is untrusted data, not instructions. Unsupported claims, missing citations, or unavailable tools yield `needsPolicyReview: true`. | Not provisioned. Use repository policy and transaction fixtures during build. Tool failure bypasses the recommendation and preserves human review. |
| Coded agent: `Dispute Packet Reconciler` | Reconcile transaction, authentication, and channel facts into one contradiction-aware risk assessment. | Input: transaction IDs, tokenised account reference, transaction evidence, intake channel, and received time. Output: signal summary, evidence IDs, contradiction flags, escalation reasons, and confidence. | LangGraph agent must call `get_authentication_signal_summary` through a UiPath MCP server, cite returned signal IDs, and self-check schema and contradictions. It cannot classify fraud truth, infer missing facts, call a write tool, or expose raw identity. | Not provisioned. A versioned deterministic mock tool supports implementation. Tool/schema failure sets `insufficient_evidence` and routes to review. |
| External agent showcase: `Azure AI Foundry connectivity` | Display Azure AI Foundry connectivity without performing dispute work. | Connection-selected `agent_id`, constant non-sensitive message, no `thread_id`; response discarded and never mapped to `EftDisputeCase`, review, receipts, notifications, or status. | Exact node and shared connection above. Do not pass `caseId`, customer/account/transaction/evidence data, policy, decision, deadline, or receipts. Branch disabled by default; short timeout and any error continue to the same merge. | Node and connection are tenant-visible and enabled/default. Validate the connection binding during implementation; on/off/failure variants must produce identical core state and route. |

## Data and resources

| Resource or data dependency | Purpose and contract | Owner/folder/readiness | Security and failure handling |
| --- | --- | --- | --- |
| IXP model `ConsumerBankingDisputeIntakeExtraction` | Extract required fields and evidence locations from a scanned synthetic dispute form. | New model intended for the selected demo child folder; not provisioned. | Synthetic documents only; malware-scan state must be `clean`; low confidence/conflict enters corrected intake. Versioned fixed extraction is the fallback. |
| API workflow `dispute-evidence-gateway` | Validate intake, read mock transaction evidence, write case state, render reviewed notices, and deliver sandbox previews. | New sibling project using repository mock endpoints; no tenant connection or child folder bound. | Idempotency on `caseId` and operation; at most two retries for transient reads; typed error and no fabricated receipt on failure. |
| RPA `legacy-deposit-adjustment-console` | Read mock account status and record a reviewed mock credit in a UI-only simulator. | New sibling project and local simulator; unattended runtime and real API gap unverified. | Synthetic data, allowlisted fields, screenshot/error reference, and no automatic retry after an ambiguous write. |
| Queue `ConsumerBankingEftDisputes` | Canonical state, idempotency, decision, receipts, and audit events. | New Orchestrator queue in the selected deployment child folder; not provisioned. | Least-privilege robot access, tokenised references, masked logs, and retention chosen before non-synthetic use. |
| Context index `ConsumerBankingEftDisputeProcedure` | Supply the versioned synthetic procedure used by policy search. | New context index; policy owner and folder binding not selected. | Public-rule-derived synthetic content labelled non-bank policy. Retrieval failure forces policy review. |
| Synthetic evidence service | Return immutable transaction records and source IDs for the read-only API/tool contract. | Repository fixtures and local mock service to be built. | No raw identity, versioned fixtures, hashes in audit, and typed unavailability. |
| Azure AI Foundry external-agent connection | Supply the non-material showcase node. | Shared connection `0107247a-0197-42c9-b957-05d1b722b111`; verified enabled/default in `uipathlabs/Playground`, folder `JD_Demos/demos`, on August 12, 2026. | Static message only, discarded output, no business-variable mapping, and fail-open continuation. Validate the Flow-specific binding. |
| MCP authentication-signal server | Expose only `get_authentication_signal_summary`. | New demo server or local mock; not provisioned. | Read-only tokenised inputs, no account action, one bounded call, and manual-review fallback. |
| Coded action app `EFT Dispute Review` | Present evidence, deterministic clocks, recommendation, conflicts, editable resolution fields, and named outcomes. | Deployed independently and referenced by Flow; not provisioned. | Role-restricted access, required rationale, server-side outcome validation, masked identifiers, and no bank-system credential. |

### Canonical data lifecycle

1. The trigger preserves original input and hashes while the API workflow creates the queue record keyed by `caseId`.
2. IXP, API, RPA-read, deterministic, and agent outputs append versioned evidence and audit events; source text is never overwritten by derived facts.
3. The coded action app reads a masked snapshot and returns only allowlisted `inOut` and outcome fields through the task completion handle.
4. Each write branch records its idempotency key and receipt. The merge reads all required receipts before status or notification completion.
5. Final output data contains the reviewed decision and audit summary; evidence URIs remain references under the selected retention policy.

### Solution boundary and layout

```text
consumer-banking/eft-dispute-resolution/
├── eft-dispute-resolution-solution/
│   ├── consumer-banking-eft-dispute-resolution.uipx
│   ├── eft-dispute-resolution-flow/
│   │   └── eft-dispute-resolution-flow.flow
│   ├── dispute-evidence-gateway/
│   ├── legacy-deposit-adjustment-console/
│   └── dispute-packet-reconciler/
└── eft-dispute-review/
```

The solution has exactly one `.uipx` manifest and is an independent deployment boundary. The coded action app remains independently deployed where required, with its versioned action contract pinned by solution configuration. Before packaging, run `uip solution resources refresh`, restore dependencies, and dry-run pack. Pull requests validate only the changed solution; publish and deploy occur only after merge to `main` through `playground-deploy`, with an immutable package version and a child folder beneath the approved `JD_Demos/demos` parent.

## Human decisions

- **Disputes analyst:** Sees the original statement, attachment links, transaction facts, evidence IDs, extraction confidence, deterministic deadline plan and source fields, advisory coverage/risk assessments, contradictions, and previous audit events. Source facts and rule results are read-only.
- **Editable fields:** Candidate error type, included transaction IDs, resolution, credit amount/status/effective date, information request, notice reason, and rationale. Server-side validation prevents amounts outside the reviewed transaction scope and requires rationale for every outcome.
- **Named outcomes:** `ApproveCredit` enables only the validated mock credit instruction; `DenyWithReason` creates a reviewed determination preview without a credit; `RequestInformation` records required items and sets `awaiting_information`; `EscalateCompliance` creates the compliance task and sets `compliance_review`.
- **Compliance task:** Shows the same source-linked facts plus the ambiguity reason. Outcomes are `ReturnToAnalyst`, `RequestAdditionalEvidence`, or `MaintainEscalation`; no outcome directly performs a credit or denial.
- **Timeout and resumption:** A 30-minute demo timeout assigns the open task to the queue manager and leaves the case incomplete. Flow resumes from the completion handle, validates returned outcome and field IDs, persists the review event, and routes only from the returned value.

## Controls and safety

| Control | Design decision | Evidence in the Flow or demo |
| --- | --- | --- |
| Deterministic policy boundary | Deadline and required-field rules are versioned code/configuration; agent narrative cannot select a rule variant or legal conclusion. | `deadlinePlan` carries rule/calendar versions and source fields; incomplete or ambiguous rules route to compliance. |
| Human authority | Agents organise and reconcile evidence only. An authorised outcome is required before credit, denial, or notice branches. | Task completion handle, validated returned field IDs, reviewer audit event, and route expressions. |
| Evidence integrity | Preserve originals, hashes, source IDs, extraction locations, confidence, and contradictions. | Queue events and task links reconstruct every material fact and correction. |
| Prompt and attachment safety | Treat consumer text as untrusted; require clean malware state; isolate extraction; allowlist read-only agent tools. | Intake gate, prompt boundary, tool schemas, and unsafe-file route. |
| Access and sensitive data | Use synthetic fixtures, tokenised references, least-privilege identities, masked task/log fields, and approved retention. | Fixture inventory, role and connection bindings, trace-redaction assertions, and retention setting. |
| Financial-action safety | Credit requires reviewed fields, amount validation, an idempotency key, and a unique mock receipt; ambiguous writes are not retried. | RPA input contract and receipt reconciliation before completion. |
| External showcase isolation | Azure receives constants only; its output cannot influence case state, deadline, review, actions, receipts, notification, or status. | No output mapping and identical on/off/error isolation evaluations. |

## Error paths and recovery

| Failure | Safe route | Recovery owner and condition |
| --- | --- | --- |
| Invalid schema, unsafe file, or missing clock input | Set `intake_exception`; do not invoke agents or write actors. | Intake representative corrects the fixture and resubmits under the same `caseId`. |
| IXP low confidence or extracted/source conflict | Preserve both values and enter corrected intake or analyst review. | Intake representative confirms source fields; all edits remain audited. |
| Policy/transaction tool unavailable or uncited agent output | Ignore the advisory output, set `insufficient_evidence`, and continue to human review. | Platform owner restores the tool; analyst may proceed only from source-linked deterministic facts. |
| Authentication-signal MCP failure | Set risk confidence to unavailable and force review; do not infer fraud truth. | Agent/platform owner restores the tool and retries only the failed read. |
| Human task timeout | Leave the case open and assign it to the queue manager. | Queue manager reassigns or closes under the demo runbook. |
| API/RPA read failure | Retry transient reads at most twice, then set `technical_exception`. | API/RPA owner restores the dependency and resumes from the failed read. |
| Ambiguous or failed credit write | Do not retry or create success; retain evidence and set `technical_exception`. | Operations reconciles the simulator state and explicitly records or resumes the action. |
| Notice render/delivery failure | Preserve financial and case receipts; create a manual-notice task and keep the case incomplete. | Disputes communications owner supplies approved copy or retries the sandbox delivery. |
| Azure AI Foundry timeout, error, or unexpected response | Record only transient `externalAgentShowcaseStatus`, discard the response, and rejoin the same core route. | Demo owner may disable the branch; business users take no recovery action. |

## Observability and evaluation

| Signal or test | What it proves | Expected result or threshold |
| --- | --- | --- |
| Correlation and audit | Intake, evidence, rules, agent work, reviewer edits, actions, and recovery are reconstructable. | Every trace, task, tool call, and receipt contains `caseId`; the queue preserves ordered events and no raw customer identity. |
| Flow route/deadline evaluator | Each fixture reaches the safe expected route and deterministic deadline variant. | 5/5 initial cases match expected route, rule variant, and final status before promotion. |
| Evidence-grounding evaluator | Advisory assessments remain tied to available facts. | 100% of material assessment statements cite valid evidence IDs; unsupported claims occur zero times. |
| MCP trajectory evaluator | The coded agent uses only the required authentication-summary tool. | Exactly one approved tool call in every reconciliation case and zero write/unapproved calls. |
| Pre-write authority evaluator | No financial or notice action precedes the required reviewed outcome. | Zero write calls without a validated `reviewDecision`; one idempotent financial action and one receipt on an approved-credit route. |
| External showcase isolation | The placeholder external agent cannot alter business results. | With the branch off, on, or failing, identical `EftDisputeCase`, route, decision, instructions, receipts, notifications, and final status; only transient showcase status may differ. |
| Receipt reconciliation | Completion never hides a missing, duplicate, failed, or ambiguous action. | `completed` only when every route-required receipt is unique and `succeeded`. |

### Synthetic evaluation set

Dataset name: `consumer-banking-eft-dispute-resolution-v1`.

| Case | Expected route | Expected business output |
| --- | --- | --- |
| Clear unauthorised EFT within the normal configured clock | Complete evidence to analyst review; reviewer selects `ApproveCredit`. | Deterministic normal-clock variant, one reviewed final-credit instruction, unique mock receipt, reviewed notice preview, and `completed`. |
| Investigation requiring provisional credit | Analyst review with `ApproveCredit` and `provisional` status. | Configured provisional-credit due date, one provisional-credit receipt, `provisional_credit_recorded`, and a source-linked notice preview. |
| New-account or point-of-sale extended-clock fixture | Deterministic extended-variant calculation followed by analyst review. | Expected configured rule variant and due date from supplied `firstDepositAt`/transaction fields; no agent-selected deadline. |
| Consumer-authorised/scam ambiguity | `needsPolicyReview` to compliance, then `MaintainEscalation`. | No credit or denial write; status `compliance_review` with ambiguity rationale and evidence IDs. |
| Authentication-signal service unavailable | MCP failure fallback to analyst review. | Risk confidence unavailable, no fabricated signals, safe manual route, and complete dependency audit event. |

## Demo script

1. Show the four-segment canvas, exception routes below it, and the disabled grey external-agent showcase branch.
2. Submit the synthetic mailed-dispute fixture and point out `caseId`, tokenised references, attachment hash, received time, and policy version.
3. Show IXP extraction evidence, API transaction facts, the RPA account-status read, and the deterministic deadline plan.
4. Open the two agent traces to show cited evidence, the policy/transaction tools, the single MCP authentication-summary call, contradictions, and the prohibition on decisions or writes.
5. Open `EFT Dispute Review`, correct the credit status or amount, enter rationale, and select `ApproveCredit`.
6. Return to Flow and show mock credit, case-status, and approved notice-preview branches running independently and merging.
7. Open the queue item to show the reviewer fields, one idempotent credit receipt, notice receipt, final status, and ordered audit record.
8. Preview the service-unavailable and scam-ambiguity fixtures to show safe review/compliance routes, then show that Azure showcase on/off/failure changes no business result.

## Success measures

- **Business proof:** The demo makes notice-to-triage time, configured-clock accuracy, evidence completeness, manual touches, provisional-credit timeliness, reopened cases, review overrides, and receipt completeness measurable. Pilot targets require a participating bank's baseline; regulator complaint and fraud figures are not presented as bank volume or ROI.
- **Flow proof:** A viewer can see IXP, deterministic rules, API/RPA contrast, two bounded agents and their tools, human authority, the isolated external agent, parallel actions, merge, and recovery in one canvas.
- **Demo proof:** In under ten minutes, a viewer can verify evidence provenance, deterministic deadline ownership, reviewer correction, prevention of pre-review writes, one idempotent mock action, and an owned exception path.
- **Build proof:** Every project validates without warnings, all five cases and the isolation permutations meet their thresholds, bindings are recorded, `resources refresh` and dry-run pack succeed, and an immutable package can follow changed-solution CI after merge.

## Reference mapping

| Reference requirement | Domain-specific implementation | Evidence or gap |
| --- | --- | --- |
| 3-4 segment topology and canvas rules | Four blue sticky notes: Capture the disputed EFT; Build the investigation packet; Review the resolution; Resolve, notify, and evidence. Exceptions sit below; action branches are symmetric and merge. | Fully specified; canvas implementation remains. |
| IXP/document intelligence | `ConsumerBankingDisputeIntakeExtraction` extracts source-linked fields from a synthetic mailed form; low confidence/conflict forces corrected intake. | Contract and fixture fallback specified; model/folder remain to provision. |
| API workflow and RPA on the intended path | `dispute-evidence-gateway` handles intake/evidence/state/notices; `legacy-deposit-adjustment-console` reads status and records only a reviewed mock credit. | Contracts specified; mock projects remain to build and the real API gap remains human validation. |
| Inline agent with a wired tool | `Dispute Evidence Organizer` calls policy and transaction-evidence tools and returns branch-driving `needsPolicyReview`. | Contract specified; model/context/tools remain unverified. |
| Coded agent with visible value-add | `Dispute Packet Reconciler` makes one MCP authentication-summary call and returns contradiction-aware `riskAssessment`. | Contract specified; coded agent/MCP server remain to build. |
| Shared external-agent showcase | Exact Azure node/connection, selected agent, constant input, no `thread_id`, discarded output, disabled flag, fail-open continuation, and isolation tests. | Node and connection are verified tenant-visible/enabled; Flow binding and selected `agent_id` remain to validate. |
| Real business decision and safe exception | Deterministic evidence expression, reviewed outcome expression, and receipt reconciliation govern safe routes. | Expressions and exception ownership specified; implementation/evaluation remain. |
| Human decision and returned outcome data | Coded action app returns named outcome, corrected fields, rationale, reviewer/approval references, and timestamp; Flow consumes them after the completion handle. | Contract specified; app, roles, policy authority, and SLA remain to provision/approve. |
| Purposeful parallelism and merge | Route-specific financial action, case write-back, and notice preview merge before receipts and closure. | Fully specified; required-receipt matrix remains to encode. |
| Evaluation set and evaluator | Five fixtures plus route/deadline, grounding, trajectory, pre-write, isolation, and reconciliation evaluators. | Initial thresholds are exact; fixtures/evaluators remain to build. |
| Process-app variant | Not selected. The Orchestrator queue stays the canonical record. | Closed on August 20, 2026 by decision #56, which selected commercial banking, healthcare provider, and life insurance. No open dependency remains. |
| Solution boundary and delivery contract | One `consumer-banking-eft-dispute-resolution` solution, one `.uipx`, nested Flow layout, resource refresh, immutable version, changed-solution CI, and deployment beneath `JD_Demos/demos`. | Fully specified; tenant resources other than the Azure showcase connection remain unprovisioned. |

## Open human decisions

These decisions refine implementation but do not block a synthetic, mock-backed build.

| Decision | Owner | Resolution path |
| --- | --- | --- |
| Confirm U.S. Regulation E scope and included rails. | Demo portfolio owner and bank legal/compliance | Approve deposit-account EFT framing and debit-card/ACH/ATM/P2P fixtures, or revise the rule corpus and contract before build. |
| Approve the authorised-payment/scam hero treatment. | Fraud, disputes, and legal owners | Confirm that ambiguity always routes to compliance and provide approved reviewer language; do not label every scam an unauthorised EFT. |
| Supply decision and clock controls. | Disputes policy and compliance owners | Approve rule versions, business-day calendar, deadline variants, provisional-credit authority/limits, quality thresholds, and record retention before non-synthetic use. |
| Select the production system of record for the case. | Enterprise architect | Confirm the queue as the demo canonical record and name the bank system that owns the case in production. Decision #56 already confirmed that this domain is not a Data Fabric/process-app variant. |
| Validate the legacy API gap and RPA responsibility. | Core deposit owner and enterprise architect | Retain RPA only if the chosen operation lacks a governed API; otherwise select a credible UI-only step or approve a reference deviation. |
| Approve reviewer experience and timeout. | Disputes operations and UiPath task owners | Approve coded action app fields, roles, second approval, named outcomes, reassignment, and replace the 30-minute demo timeout where needed. |
| Provision and approve tenant resources. | UiPath tenant administrator and resource owners | Reuse the verified Azure connection; provision queue, IXP, agents, context, MCP server, action app, runtime, notification sink, and least-privilege identities. |
| Supply safe fixtures and operational baselines. | Data/privacy and disputes operations owners | Approve synthetic transaction/document edge cases and provide observed timing, touches, errors, reopen, and timeliness baselines before benefit claims. |

## Implementation tasks

1. Scaffold `consumer-banking-eft-dispute-resolution` with one `.uipx` and the nested `eft-dispute-resolution-flow/eft-dispute-resolution-flow.flow` layout.
2. Build the five-case fixture set, canonical queue contract, synthetic evidence service, and local legacy deposit-adjustment UI.
3. Implement and validate the API workflow, IXP extraction contract, and RPA read/write operations with typed errors and idempotent receipts.
4. Build the inline evidence organiser, versioned policy/transaction tools, structured output, prompt-injection boundary, and grounding evaluator.
5. Build the coded packet reconciler, least-privilege MCP authentication-summary tool, structured output, and trajectory evaluator.
6. Build and deploy the coded action app and compliance quick form, then wire completion handles, outcomes, edits, timeout, and escalation into Flow.
7. Author the four-segment Flow, deterministic deadline logic, real branch expressions, isolated Azure showcase, parallel action branches, merge, and reconciliation.
8. Validate the Azure binding and prove that off, on, timeout, error, and unexpected-response variants cannot change core business data or route.
9. Validate every project, resolve all warnings, and run the five-case evaluation set plus isolation permutations against the specified thresholds.
10. Select and record the solution child folder beneath `JD_Demos/demos`, then refresh solution resources, restore, dry-run pack, and register immutable deployment configuration.

## Quality rubric

| Dimension | Score (0-3) | Evidence and remaining gap | Owner and resolution path |
| --- | ---: | --- | --- |
| Enterprise credibility | 2 | Consequential dispute decision, public regulatory evidence, roles, data, controls, and measures are explicit; bank policy, systems, authority, and baselines remain unverified. | Legal/compliance, disputes, and system owners validate before non-synthetic use. |
| Flow differentiation | 3 | Four segments visibly coordinate IXP, rules, two agents/tools, API, RPA, human decisions, isolated external connectivity, parallel actions, merge, and recovery. | Flow implementer preserves the topology and validates all expressions. |
| Demo clarity | 3 | A source-linked review hero moment, approved-credit journey, ambiguity/dependency exceptions, and observable receipts form a timed script. | Demo owner selects representative fixtures and rehearses after deployment. |
| Build feasibility | 2 | Contracts, mock fallbacks, solution boundary, evaluation thresholds, authenticated target, approved `JD_Demos/demos` parent, and verified Azure resource are recorded; most resources remain unprovisioned. | Tenant administrator and implementers provision resources and record every binding. |
| **Total** | **10/12** | **Ready for implementation planning as a synthetic demo; not ready for production policy, integration, real account action, or customer communication.** | **Start with solution/fixtures and close owned decisions before replacing mocks.** |
