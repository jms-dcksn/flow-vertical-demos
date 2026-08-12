# P&C insurance catastrophe property-claim demo specification

## Use case and narrative

- **Domain and solution:** `p-c-insurance/catastrophe-property-claim/catastrophe-property-claim-solution/`, with globally unique solution and package name `p-c-insurance-catastrophe-property-claim`.
- **Enterprise use case:** A catastrophe claims adjuster reviews a synthetic residential wildfire claim, an evidence-linked coverage recommendation, and a rules-backed advance-payment proposal. The adjuster may correct the proposal, request information, escalate coverage, or assign an investigation before any consequential action. The hero moment is the reviewer editing an advance amount and investigation plan in a coded action app, then watching Flow use the returned fields to create only approved mock records and receipts.
- **Why this use case:** It is the top-ranked candidate in the [P&C insurance opportunity research](agentic-workflow-opportunities.md), scoring 14/15. The cited catastrophe volume, surge staffing, smoke-investigation obligations, time-bound handling, and advance-payment rules make the workflow credible, while documents, rules, agents, API and UI work, human authority, and recovery make Flow's value visible. Commercial submission triage scored 13/15 and remains the alternative for an underwriting-led audience.
- **Audience journey:** The demonstrator submits a synthetic wildfire claim packet, follows four named canvas segments, inspects cited evidence and deadline calculations, opens the adjuster review, edits the advance and assigns a smoke inspection, and then shows independent mock claim, vendor, and communication work merge into an audit-ready result. Flow is the right surface because the value is coordination across deterministic rules, document intelligence, bounded agents, RPA, APIs, and human judgment rather than a single model response.
- **Success outcome:** The case reaches the correct safe state with source-linked evidence, an authenticated human outcome, an authority-valid non-executing payment request when approved, required mock receipts, and a reconstructable audit trail. No agent denies coverage, sets a reserve, issues funds, contacts a claimant, or makes a fraud determination.
- **Measurable value:** Pilot measures are median and p90 time from report to acknowledgement, assignment, first approved action, and customer update; packet completeness; correct-route rate; human touch time; rework; missed-deadline count; exception age; specialist utilization; override rate; and duplicate or failed write-backs. The demo makes these measures observable but claims no improvement or ROI without a carrier baseline.
- **Out of scope:** Production policy interpretation, coverage denial, live disbursement, reserve setting, fraud accusation, real claimant or property data, production carrier or vendor connectivity, jurisdictional legal advice, customer delivery beyond a mock outbox, Data Fabric/process-app scope, and quantified benefit claims.

## Personas

| Persona | Role in the demo | Authority boundary |
| --- | --- | --- |
| Catastrophe claims adjuster | Reviews facts, evidence, proposed coverage position, advance amount, investigation, and communication. | May select named outcomes and edit allowlisted fields within configured authority; cannot issue funds or make an unsupported adverse decision. |
| Claims supervisor | Owns authority-limit, disputed-coverage, deadline-risk, and overdue cases. | May approve or return an escalation under the configured synthetic authority matrix; cannot bypass evidence and audit requirements. |
| Property specialist or vendor coordinator | Resolves smoke, environmental, structural, or mitigation work. | Receives only the approved task scope; cannot decide coverage or payment. |
| Demonstrator | Submits fixtures and narrates evidence, review, follow-up, and recovery. | Uses synthetic data, versioned rules, local mocks, and non-executing write targets only. |

## Trigger and case contract

The initial build uses a manual trigger for repeatable demonstration. A governed FNOL event can replace it after a carrier channel is selected. `<claimId>:<correlationId>` is the idempotency key. An exact replay reads the existing queue item and appends a replay event; it never creates a second review task, payment request, or vendor order.

| Item | Specification |
| --- | --- |
| Trigger | Manual Flow trigger accepting `CatastropheClaimInput`. The `catastrophe-claim-gateway` API workflow validates the envelope, reads synthetic policy/claim facts, and emits the canonical case. |
| Canonical record | Orchestrator queue `PCInsuranceCatastropheClaims`, unique reference `<claimId>:<correlationId>`. Specific content stores the input, evidence, current state, and audit events; output data stores the human decision, action plan, receipts, and final status. |
| Required inputs | Event, policy/loss, coverage, evidence, and jurisdiction fields defined below. All values and files are synthetic. |
| Sensitive data | Claimant contact and risk address are restricted PII/property-security data. They are masked in tasks and traces and excluded from agent inputs unless a minimum field is explicitly required by an allowlisted tool. |
| Outputs | Cited assessment, deterministic routing, reviewer outcome and edits, non-executing payment request, mock claim/vendor/communication receipts, exceptions, audit artifact, and final status. |

### Input schema

| Field | Type | Required | Contract |
| --- | --- | --- | --- |
| `claimId` | string | yes | Synthetic carrier claim identifier. |
| `correlationId` | UUID string | yes | Idempotency and trace key shared by every task, tool call, and receipt. |
| `reportedAt` | ISO 8601 string | yes | Starts acknowledgement and task-aging measures. |
| `channel` | enum | yes | `portal`, `phone`, or `agent`; used for audit only in the initial demo. |
| `catastropheEventId` | string | yes | Synthetic wildfire-event identifier used by the event relationship tool. |
| `policy` | object | yes | `policyNumber`, status, effective dates, risk address, dwelling/content/ALE limits, deductible, endorsements, and prior payments. Values originate from the mock policy API, not an agent. |
| `loss` | object | yes | Occurrence time, reported cause, damage description, occupancy/displacement facts, and requested help. |
| `evidenceFiles` | array | yes | Synthetic claim form plus optional notes, photos, estimates, receipts, contents list, and smoke/environmental report. Each item includes file name, type, size, and hash. |
| `jurisdictionConfig` | object | yes | `jurisdiction`, `ruleSetVersion`, notice/investigation deadlines, advance-payment rules, and authority limits. California is an illustrative versioned profile, not a universal rule. |
| `showExternalAgentShowcase` | boolean | yes | Defaults to `false`; controls only the isolated Azure AI Foundry showcase branch. |

### Output schema

| Field | Type | Contract |
| --- | --- | --- |
| `coverageAssessment` | object | Advisory `recommendation`, `coverageBasis`, `damageCategories`, `missingEvidence`, `contradictions`, `confidence`, `ruleSetVersion`, and source references. |
| `triage` | object | `severityBand`, `specialtiesNeeded`, `queue`, `dueAt`, `reasonCodes`, and `requiresSpecialist`; values driving routes are deterministically verified. |
| `review` | object | `outcome`, edited `approvedAdvanceAmount`, `approvedCoveragePosition`, investigation/vendor tasks, claimant-message selection, reviewer rationale/ID, authority result, and timestamps. |
| `paymentRequest` | object or null | Non-executing request ID, amount, category, authority reference, status, and timestamp; never a payment receipt. |
| `actionReceipts` | array | System, operation, approved scope, status, correlation ID, and timestamp for mock claim, vendor, and outbox work. |
| `exceptions` | array | Typed code, owner, retry safety, recovery condition, and state. |
| `finalStatus` | enum | `awaiting_information`, `awaiting_specialist`, `supervisor_review`, `approved_plan_recorded`, `action_incomplete`, or `technical_exception`. |
| `auditEvents` | array | Actor, action, input/output hashes, evidence refs, rule/model/prompt/tool versions, reviewer override, system response, and timestamp. |
| `auditArtifactUrl` | string | Mock or approved storage URI for the reconciled claim-plan artifact. |

## Flow topology

Use four blue sticky notes with the titles below. The happy path runs left to right. Validation, evidence, authority, timeout, and dependency exceptions sit below their originating segment. Post-decision work is visually symmetric and merges before reconciliation. A separately labelled `External agent showcase` branch sits below segment 2 and never receives claim data.

| Reference segment | Domain-specific canvas title | Actors and business output | Branch or merge evidence |
| --- | --- | --- | --- |
| Receive and understand | **1. Register the catastrophe loss** | Manual trigger invokes `catastrophe-claim-gateway`, creates or reads the queue item, scans and validates file metadata, and invokes `PropertyClaimPacketExtraction` IXP. Output: canonical `CatastropheClaimCase` and extraction confidence. | Invalid envelope, policy-date contradiction, unsafe file, or missing required evidence routes below the happy path. `extractionConfidence < 0.90` sets `requiresSpecialist = true`; it never produces an adverse conclusion. |
| Assess and enrich | **2. Build the cited claim assessment** | A deterministic script calculates deadlines and candidate advance limits from the pinned configuration. Inline `Catastrophe Claim Triage` uses a read-only event/location tool. Coded `Coverage Evidence Analyst` retrieves approved policy/rule passages and produces a cited schema-bound assessment. | `requiresSpecialist === true` when damage includes smoke/environmental concerns, evidence contradicts policy facts, required evidence is missing, extraction confidence is below `0.90`, or assessment confidence is below `0.85`. The thresholds are demo configuration pending owner approval. The external showcase branch rejoins with the case unchanged. |
| Decide and review | **3. Exercise claims authority** | Every valid case opens `Catastrophe Claim Plan Review`, a coded action app. Specialist-needed cases show the required specialty and safe investigation route. Output: named outcome, edits, rationale, identity, authority result, and completion handle. | `review.outcome` is `ApprovePlan`, `RequestInformation`, `EscalateCoverage`, or `AssignInvestigation`. `ApprovePlan` proceeds only when `review.authorityResult === "within_limit"`; otherwise it routes to supervisor review. Timeout leaves the claim open and escalates. |
| Act and communicate | **4. Coordinate approved recovery** | After an authorized outcome, independent branches invoke the API workflow to persist the plan/create a non-executing payment request, RPA to submit an approved vendor request in a mock UI, and the API workflow to place approved text in a mock outbox. The branches merge before queue finalization. | `requiredActionReceipts.every(receipt => receipt.status === "succeeded")` permits `approved_plan_recorded`. Missing or ambiguous required receipts set `action_incomplete`, preserve completed receipts, and create an owned reconciliation task. |

## Agentic reasoning and tool use

| Actor | Visible responsibility | Structured input and output | Tools, context, and guardrails | Readiness/fallback |
| --- | --- | --- | --- | --- |
| Inline low-code agent: `Catastrophe Claim Triage` | Explain event relationship and recommend severity/specialties without deciding coverage. | Input: synthetic event ID, coarse geospatial fixture key, damage categories, displacement facts, and evidence summary. Output: `eventRelationship`, `severityBand`, `specialtiesNeeded`, `reasonCodes`, `evidenceRefs`, and `confidence`. | Read-only `get_catastrophe_event_relationship(eventId, locationFixtureKey)` tool. It returns fixture provenance and `confirmed`, `disputed`, or `unknown`; `unknown` is non-adverse and forces review. No policy, payment, write, web-search, or claimant-contact tool is allowed. | No matching tenant resource was found by the August 12, 2026 registry inspection. Build a new inline agent with a versioned local tool fixture. Tool failure returns `unknown` and preserves the human-review route. |
| Coded agent: `Coverage Evidence Analyst` | Reconcile extracted evidence with approved policy clauses and jurisdiction rules, then self-check every material conclusion. | Input: extracted fields/references, deterministic rule results, `policyVersion`, and `ruleSetVersion`. Output: `recommendation`, `coverageBasis`, `missingEvidence`, `contradictions`, `confidence`, citations, and self-check findings. | LangGraph agent calls least-privilege MCP tools `get_policy_clause(policyVersion, clauseId)` and `get_claim_rule(ruleSetVersion, ruleId)`. It cannot alter records, calculate a final payment, deny coverage, or communicate externally. Missing/invalid citations discard the recommendation and force review. | Not provisioned. Use a synthetic policy/rule corpus and mock MCP server with identical schemas. Invalid schema, tool failure, or failed self-check produces `insufficient_evidence`. |
| External agent showcase: `Azure AI Foundry connectivity` | Display external-agent connectivity without performing claims work. | Connection-selected `agent_id` and constant message `UiPath Flow external-agent connectivity showcase for P&C insurance`; omit `thread_id`. Discard the response. | Node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` and connection `0107247a-0197-42c9-b957-05d1b722b111`. Do not pass claim, claimant, property, policy, evidence, assessment, reviewer, payment-request, or receipt data. | Reverified August 12, 2026 in `uipathlabs / Playground`: the node is tenant-available and the connection is enabled, default, and active in folder `demos`. The branch is disabled by default; timeout/error records transient showcase status and rejoins the unchanged core route. |

## Data and resources

Tenant registry searches on August 12, 2026 returned no matches for `catastrophe claim`, `PropertyClaimPacketExtraction`, or `legacy vendor portal`. The solution does not exist yet, so no local sibling resources can be discovered. The domain actors below are therefore build targets with contract-compatible mocks; no unverified live resource is assumed.

| Resource or data dependency | Purpose and contract | Owner/folder/readiness | Security and failure handling |
| --- | --- | --- | --- |
| IXP project: `PropertyClaimPacketExtraction` | Classifies the synthetic packet and extracts loss date/cause, occupancy/displacement, damage categories, estimate/receipt values, environmental findings, and page references. | New model; deploy beneath the approved `JD_Demos/demos` parent; not provisioned. | Synthetic files only. Malware/type/size failure rejects intake. Confidence below `0.90`, missing values, or conflicts force review. A recorded extraction fixture is explicitly marked `mock`. |
| API workflow: `catastrophe-claim-gateway` | Operations `register_claim`, `read_policy`, `record_plan`, `create_payment_request`, and `stage_message` use versioned JSON-backed claims/policy/outbox mocks and return typed receipts. | New sibling project; neutral local mock endpoint; not provisioned. | Idempotency on `<claimId>:<correlationId>`, at most two retries for safe transient reads, no retry after ambiguous write, and no fabricated success. |
| RPA: `legacy-vendor-portal` | Operation `request_service` enters only the human-approved inspection, smoke testing, mitigation, or temporary-housing task into a local UI fixture and returns a confirmation or screenshot-backed exception. | New sibling project targeting a neutral local portal; unattended runtime not provisioned. | RPA is retained only to demonstrate a verified UI-only fixture. No free-text navigation or claim decision; an ambiguous write is not retried and routes to reconciliation. |
| Queue: `PCInsuranceCatastropheClaims` | Canonical demo case, idempotency, state transitions, outputs, exception ownership, and ordered audit events. | New Orchestrator queue beneath the approved `JD_Demos/demos` parent; not provisioned. | Least-privilege access, masked PII, configured retention, and duplicate-reference read-back. |
| Context and MCP corpus | Versioned synthetic homeowners policy, endorsements, authority matrix, and illustrative jurisdiction rules, exposed only by ID/version lookup. | New demo corpus and MCP server; claims/compliance owners approve content; not provisioned. | No open web retrieval at runtime. Missing version, ID, or effective date forces review and is visible in citations. |
| Catastrophe/geospatial fixture | Returns synthetic event/location relationship and provenance. | New deterministic fixture; no live geospatial provider selected. | Coarse synthetic locations only. `unknown`, timeout, or disputed perimeter is a review signal, never a denial signal. |
| Azure AI Foundry connection | Supplies the non-material external-agent node with constant input and discarded output. | Shared connection `0107247a-0197-42c9-b957-05d1b722b111`; active/default in Playground folder `demos`. | No case/sensitive data or business-variable output mapping; fail-open continuation to the same core route. Flow-specific binding and `agent_id` remain to validate. |
| Coded action app: `Catastrophe Claim Plan Review` | Side-by-side policy facts, extraction evidence/pages, rule citations, contradictions, recommendation, deadlines, proposed advance, and investigation plan. | Deployed independently and referenced by Flow; not provisioned. | Role-restricted access, allowlisted edits, required rationale, server-side authority validation, masked PII, and no operational credentials. |

### Solution boundary and layout

```text
p-c-insurance/catastrophe-property-claim/
├── catastrophe-property-claim-solution/
│   ├── p-c-insurance-catastrophe-property-claim.uipx
│   ├── catastrophe-property-claim-flow/
│   │   └── catastrophe-property-claim-flow.flow
│   ├── catastrophe-claim-gateway/
│   ├── legacy-vendor-portal/
│   └── coverage-evidence-analyst/
└── catastrophe-claim-plan-review/
```

The solution has exactly one `.uipx` manifest and is independently deployable. The coded action app remains independently deployed if required by the platform, with its versioned contract pinned by solution configuration. Before packaging, run `uip solution resources refresh`, restore dependencies, and dry-run pack. Pull requests validate only the changed solution. Publishing and deployment occur only after merge to `main` through `playground-deploy`, with an immutable package version beneath the approved `JD_Demos/demos` parent folder.

## Human decisions

- **Adjuster review:** The adjuster sees read-only policy facts, source-page evidence, rule citations/effective dates, contradictions, confidence, deadlines, recommendation, and prior payments. Editable `inOut` values are `approvedAdvanceAmount`, `approvedCoveragePosition`, investigation/vendor tasks, and message-template selection.
- **Named outcomes:** `ApprovePlan` authorizes mock downstream work only after authority validation; `RequestInformation` creates required-evidence tasks and stages an approved request; `EscalateCoverage` creates a supervisor task; `AssignInvestigation` creates specialist/vendor work while leaving coverage and payment undecided.
- **Rationale and overrides:** Rationale is mandatory for an override, escalation, or investigation assignment. Source facts, rule calculations, citations, and receipts are not editable. The app returns field IDs, old/new values, reviewer ID, outcome, rationale, and timestamp.
- **Authority and specialist review:** An amount above the configured adjuster limit, disputed coverage, or deadline risk routes to the claims supervisor. Smoke/environmental issues route to a qualified specialist and cannot yield automatic denial or accusation.
- **Timeout and resumption:** A 30-minute demo timeout assigns the case to the claims supervisor and leaves it open; the production SLA remains a human decision. Flow resumes from the completion handle, validates the returned outcome/field IDs, persists the decision, and routes only from the returned values.
- **Downstream authority:** Agents remain advisory. No downstream branch can issue funds, deny coverage, set reserves, send external communication, or create vendor work without the required authenticated human outcome.

## Controls and safety

| Control | Design decision | Evidence in the Flow or demo |
| --- | --- | --- |
| Claims fairness and routing | Deterministic checks precede agent reasoning. Uncertain, conflicting, smoke/environmental, adverse, or out-of-authority cases remain with qualified humans. | Real route expressions, exception paths below the happy path, named human outcomes, and no autonomous denial branch. |
| Grounding and governance | Policy/rule retrieval is version pinned; every material conclusion requires a valid source ID/effective date; prompt/model/tool versions and overrides are retained. | Structured coded-agent output, MCP trajectory, citation evaluator, and append-only audit events. |
| Data and access | Synthetic data, least-privilege identities, masked PII/property details, role-restricted tasks, managed connections, and retention controls. | Fixture inventory, connection bindings, task roles, trace-redaction assertions, and no secrets in Flow variables. |
| Payment and agent boundaries | The demo creates a payment request, never a disbursement. Agents cannot decide coverage, authorize amounts, mutate records, contact claimants, or act on fraud indicators. | Tool allowlists, task completion handle, authority check, output schemas, and mock-only write targets. |
| Receipt truth | A write without a confirmed receipt is not success and is not retried automatically when outcome ambiguity exists. | Merge plus deterministic receipt reconciliation gates final status. |
| External showcase isolation | Azure AI Foundry receives constants only and cannot affect case data, routes, human decisions, writes, receipts, or status. | No business-variable mappings and an off/on/failing isolation evaluation. |

## Error paths and recovery

| Failure | Safe route | Recovery owner and condition |
| --- | --- | --- |
| Invalid envelope, unsafe file, or policy/date contradiction | Set `technical_exception` or `awaiting_information`; do not invoke coverage or action actors. | Claims intake corrects/resubmits with the same idempotency key, or a platform owner clears the file incident. |
| Low-confidence extraction or missing/contradictory evidence | Preserve source references and force adjuster/specialist review. | Adjuster requests information or assigns investigation; corrected evidence resumes the same case. |
| Event/location tool unavailable or disputed | Record `eventRelationship = unknown`; do not infer non-coverage. | Adjuster reviews available evidence; platform owner may restore the fixture/tool and retry the read. |
| Policy/rule MCP failure, invalid citation, or coded-agent schema failure | Discard the recommendation, set `insufficient_evidence`, and present deterministic/source facts to the adjuster. | Claims/compliance owner corrects corpus/configuration; platform owner retries only the failed read/analysis. |
| Human task timeout | Leave the case open and assign it to the claims supervisor. | Supervisor reassigns, reviews, or closes under the demo runbook. |
| API/RPA transient read failure | Retry safe reads at most twice, then create `technical_exception`. | Platform or RPA owner restores the dependency and resumes at the failed actor. |
| Ambiguous or failed claim/vendor/outbox write | Do not retry automatically or create success; preserve confirmed receipts and set `action_incomplete`. | Claims operations reconciles target state and explicitly resumes or records manual completion. |
| Azure AI Foundry timeout, error, or unexpected response | Record transient `externalAgentShowcaseStatus`, discard the response, and rejoin the unchanged core route. | Demo owner may disable the branch; business users take no recovery action. |

## Observability and evaluation

| Signal or test | What it proves | Expected result or threshold |
| --- | --- | --- |
| Correlation and audit | Input, evidence, rule evaluation, agent rationale, reviewer edits, system work, and recovery can be reconstructed. | Every trace, task, tool call, exception, and receipt contains `claimId` and `correlationId`; no unmasked claimant contact or exact address appears in traces. |
| Flow route evaluator | Each fixture reaches the expected safe review route and final state. | 5/5 initial synthetic cases match expected route and state before promotion. |
| Grounded-citation evaluator | Coverage evidence never invents policy or jurisdiction support. | Every material conclusion has a valid source ID, effective date, and version; unsupported adverse conclusions occur zero times. |
| Tool-use/trajectory evaluator | The two agents call only their required read-only tools. | Event tool called in every triage case; policy/rule MCP called for every assessment; zero unapproved calls. |
| Human-authority evaluator | No payment request, vendor task, adverse decision, or external message bypasses the human outcome and authority rule. | Zero unauthorized downstream actions across all cases; above-limit edits always route to supervisor. |
| External showcase isolation | The placeholder external agent cannot change business results. | With the branch off, on, or failing, identical `CatastropheClaimCase`, review route, approved scope, required receipts, and final status; only transient showcase status may differ. |
| Receipt reconciliation | Completion never hides missing or ambiguous work. | `approved_plan_recorded` occurs only when every required receipt is present and `succeeded`. |

### Synthetic evaluation set

Dataset name: `p-c-insurance-catastrophe-property-claim-v1`.

| Case | Expected route | Expected business output |
| --- | --- | --- |
| Complete covered wildfire loss | Standard adjuster review; reviewer edits and selects `ApprovePlan` within authority. | Cited assessment, returned edits consumed, payment request and applicable mock receipts reconciled, `approved_plan_recorded`. |
| Contradictory policy status | Information/integration exception before any action. | No coverage conclusion or payment request; contradiction and owned recovery recorded. |
| Smoke-only damage | Specialist-needed review; reviewer selects `AssignInvestigation`. | Environmental testing/vendor task only, no denial or payment request, `awaiting_specialist`. |
| Advance exceeds adjuster authority | Adjuster returns `ApprovePlan`, authority expression diverts to supervisor. | No downstream write until supervisor outcome; final state `supervisor_review` until completion. |
| Claim gateway timeout followed by replay | Bounded read retries, then technical exception; exact replay reads the existing case. | One queue item, no duplicate task/write, preserved incident evidence, `technical_exception`. |

## Demo script

1. Show the four-segment canvas, exception routes below the happy path, and the separately labelled Azure showcase branch.
2. Submit the complete wildfire-loss fixture and point out `claimId`, `correlationId`, policy/rule versions, and evidence hashes.
3. Open the assessment trace to show IXP page references, deadline/advance calculations, the event tool, policy/rule MCP calls, citations, contradictions, and advisory confidence.
4. Open `Catastrophe Claim Plan Review`, compare source evidence with the recommendation, change the advance amount, add an inspection task, enter rationale, and select `ApprovePlan`.
5. Return to Flow and show the mock claim/payment-request, vendor-portal RPA, and outbox branches running independently and merging.
6. Open the queue item to show returned edits, authority result, receipts, final status, prompt/tool versions, and ordered audit events.
7. Point out the disabled Azure AI Foundry branch; show its constant message, discarded output, and isolation from all claim variables.
8. Preview the smoke-only fixture to show specialist assignment without automatic denial, then the authority-limit fixture to show supervisor escalation before any write.

## Success measures

- **Business proof:** The demo exposes time to first approved action, packet completeness, correct-route rate, override behavior, specialist handoff, deadline risk, and write reconciliation as measurable pilot signals without promising a target improvement.
- **Flow proof:** A viewer can see IXP, deterministic rules, two material agent roles, the non-material Azure AI Foundry showcase, API/RPA contrast, a real business route, human authority, parallel follow-up, merge, and safe recovery.
- **Demo proof:** In under ten minutes, a viewer can verify evidence provenance, cited recommendation, reviewer correction, downstream use of returned data, mock receipts, and smoke/authority exceptions.
- **Build proof:** Every project validates, the five-case evaluation set passes, every binding is recorded, `resources refresh` and dry-run pack succeed, and the immutable package can follow changed-solution CI into Playground after merge.

## Reference mapping

| Reference requirement | Domain-specific implementation | Evidence or gap |
| --- | --- | --- |
| Use case and hero moment | Adjuster corrects a cited catastrophe-claim plan and advance before mock action. | Top-ranked 14/15 research candidate; implementation remains. |
| 3-4 segment topology and canvas rules | Four blue sticky notes: Register the catastrophe loss; Build the cited claim assessment; Exercise claims authority; Coordinate approved recovery. | Fully specified; canvas remains to author and format. |
| IXP/document intelligence | `PropertyClaimPacketExtraction` extracts claim, estimate, receipt, and environmental fields with page references. | Contract and mock fallback specified; model/folder remain to provision. |
| API workflow and RPA on the intended path | `catastrophe-claim-gateway` reads/writes mock claim data; `legacy-vendor-portal` submits human-approved UI-only work. | Contracts specified; projects/fixtures remain to build and the real-world API gap remains a human validation. |
| Inline agent with a wired tool | `Catastrophe Claim Triage` calls `get_catastrophe_event_relationship` and returns branch-relevant specialty/evidence fields. | Contract specified; inline project/tool remain to build. |
| Coded agent with visible value-add | `Coverage Evidence Analyst` retrieves policy/rule evidence and self-checks cited structured output. | Contract specified; coded agent and MCP server remain to build. |
| Shared external-agent showcase | Tenant-available Azure node and active shared connection on a constant-input, discarded-output, fail-open branch. | Connection readiness verified; Flow binding and connection-selected `agent_id` remain to validate. |
| Real business decision and safe exception | Specialist/confidence/contradiction expression plus outcome and authority branches; no autonomous adverse route. | Expressions and recovery are specified; implementation/evaluation remain. |
| Human decision and returned outcome data | Coded action app returns named outcome, edited amount/plan, rationale, reviewer, authority result, and timestamp; Flow consumes them downstream. | Contract specified; authority matrix and production SLA remain human decisions. |
| Purposeful parallelism and merge | Human-approved claim/payment-request, vendor RPA, and outbox branches merge before receipt reconciliation. | Fully specified; outcome-specific required-receipt matrix remains to encode. |
| Evaluation set and evaluator | Five fixtures plus route, citation, trajectory, authority, isolation, and reconciliation evaluators with exact initial thresholds. | Fully specified; fixtures/evaluators remain to build. |
| Process-app variant | Not selected. The queue is the canonical demo record and the coded action app is the review surface. | Later cross-domain selection may change this design. |
| Solution boundary and delivery contract | One `p-c-insurance-catastrophe-property-claim` solution, one `.uipx`, nested Flow layout, resource refresh, immutable version, changed-solution CI, and deployment beneath `JD_Demos/demos`. | Deployment parent approved; domain resources still require provisioning. |

## Open human decisions

These decisions refine implementation but do not block a synthetic, mock-backed build.

| Decision | Owner | Resolution path |
| --- | --- | --- |
| Confirm the carrier segment, property product, peril, and jurisdictions. | Demo portfolio owner and claims counsel | Approve the synthetic California residential-wildfire profile or update rules, corpus, and fixtures. |
| Confirm coverage/advance rules and authority matrix. | Claims, compliance, and finance owners | Approve the policy form, endorsements, editable fields, advance calculations, maker-checker points, and escalation thresholds. |
| Confirm response deadlines and escalation SLA. | Claims operations and compliance owners | Replace the 30-minute demo timeout and illustrative jurisdiction deadlines with approved test policy. |
| Select representative core systems and validate the vendor-portal API gap. | Enterprise architect and application owners | Keep neutral mocks; retain RPA only if a credible UI-only responsibility exists, otherwise select another purposeful UI step. |
| Decide whether a non-executing payment request is sufficient. | Claims and finance owners | Retain the safe base contract or separately approve a sandbox-only payment integration with segregation and reconciliation. |
| Approve policy/rule corpus and model governance. | Claims legal, compliance, and model-risk owners | Review sources, versions, prompts, evaluations, citations, bias monitoring, and third-party controls. |
| Approve data, privacy, retention, and trace controls. | Privacy, security, and records owners | Confirm allowed fields, residency, masking, image/document handling, access roles, and retention before non-synthetic testing. |
| Provision the remaining UiPath resources. | UiPath tenant administrator | Deploy beneath the approved `JD_Demos/demos` parent, reuse the verified Azure connection, provision IXP, queue, agents, MCP, action app, and runtime, and record every binding. |
| Establish pilot baselines and target measures. | Catastrophe claims operations owner | Supply observed timing, routing, touch-time, rework, deadline, override, and failure baselines before making benefit claims. |
| Decide whether P&C insurance is a Data Fabric/process-app variant. | Demo portfolio owner | Resolve in the later cross-domain selection issue; this spec currently marks it not selected. |

## Implementation tasks

1. Scaffold `p-c-insurance-catastrophe-property-claim` with the nested Flow project and one `.uipx` manifest.
2. Build the five-case fixture set, queue contract, synthetic policy/rule corpus, JSON-backed claims/policy/outbox mock, event fixture, and local vendor UI.
3. Implement and validate the API workflow, IXP contract, and RPA operation with idempotency, typed errors, and receipts.
4. Build the inline triage agent, read-only catastrophe-event tool, structured output, and trajectory evaluation.
5. Build the coded coverage-evidence agent, least-privilege policy/rule MCP tools, citation self-check, and grounding evaluation.
6. Build and deploy the coded action app, then wire its completion handle, returned field IDs, outcomes, edits, rationale, and authority result into Flow.
7. Author the four-segment Flow, exception routes, non-material Azure showcase, human-outcome branches, parallel approved actions, merge, and deterministic reconciliation.
8. Bind and validate Azure AI Foundry connection `0107247a-0197-42c9-b957-05d1b722b111`; prove the showcase cannot change case data, review route, decisions, actions, receipts, or status.
9. Run project validation and the five-case evaluation set; resolve every warning and failed threshold.
10. Refresh solution resources, restore, dry-run pack, and register immutable-version deployment configuration.

## Quality rubric

| Dimension | Score (0-3) | Evidence and remaining gap | Owner and resolution path |
| --- | ---: | --- | --- |
| Enterprise credibility | 2 | Public evidence supports catastrophe scale, claims controls, smoke investigation, deadlines, and advance rules; carrier policy, systems, authority, and baselines remain unverified. | Claims/legal and system owners validate before replacing fixtures. |
| Flow differentiation | 3 | Four segments visibly coordinate IXP, deterministic rules, agents/tools, API, RPA, human authority, parallel work, merge, and safe recovery. | Flow implementer preserves the topology and validates real expressions. |
| Demo clarity | 3 | The edited-advance hero journey plus smoke and authority exceptions have named, observable proof points and a timed script. | Demo owner rehearses the five fixtures after deployment. |
| Build feasibility | 2 | Inputs, outputs, mocks, fallbacks, authenticated tenant target, verified Azure connection, solution boundary, evaluation, and delivery are specified; domain resources remain to build. | Tenant administrator and implementers provision resources and record bindings. |
| **Total** | **10/12** | **Ready for implementation planning as a synthetic demo; not ready for production integration, coverage action, payment, or claimant communication.** | **Start with solution/fixtures, then close owned human decisions before replacing mocks.** |
