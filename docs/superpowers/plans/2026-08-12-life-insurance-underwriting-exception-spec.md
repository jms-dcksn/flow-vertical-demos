# Life-insurance Underwriting Evidence-exception Spec Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Complete GitHub issue #25 with an implementation-ready specification for the recommended life-insurance Maestro Flow demo.

**Architecture:** Promote the research's top-ranked underwriting evidence-exception recommendation into one template-aligned domain specification. Keep research evidence separate from the design, use typed synthetic contracts and explicit mocks for unverified resources, and define Flow, actor, consent, review, safety, evaluation, Azure-isolation, solution-boundary, and delivery contracts in the new spec.

**Tech Stack:** Markdown, repository reference pattern, UiPath CLI 1.199.0 read-only tenant/resource inspection, and GitHub issue #25 acceptance criteria.

## Global Constraints

- Scope changes to GitHub issue #25, `life-insurance/`, and this plan file only.
- Preserve `life-insurance/agentic-workflow-opportunities.md` as the evidence and recommendation source.
- Select the top-ranked underwriting evidence-exception orchestrator without another approval gate; keep carrier-owned policies and authorities as open human decisions.
- Design one independently deployable UiPath Solution with exactly one `.uipx` manifest and the CLI-supported nested Flow layout.
- Deploy the solution under the approved UiPath Labs Playground parent `JD_Demos/demos`.
- Use synthetic application, medical, consumer-report, carrier-rule, and policy-admin data; no real evidence acquisition, applicant decision, system write, or communication is permitted.
- Treat every unprovisioned tenant, vendor, carrier, model, tool, and system dependency as a named mock or owned readiness gap.
- Bind node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` to connection `0107247a-0197-42c9-b957-05d1b722b111` only on a disabled-by-default, non-material showcase branch with a connection-selected `agent_id`, constant static message, no `thread_id`, discarded output, and fail-open continuation to an unchanged core route.
- Keep open human decisions separate from implementation tasks.
- Keep the README concise and use no emojis in documentation.
- Do not run or debug a Flow, provision resources, publish, deploy, or perform external writes while executing this documentation plan.

---

### Task 1: Author and validate the underwriting evidence-exception specification

**Files:**
- Create: `life-insurance/underwriting-evidence-exception-demo-spec.md`
- Modify: `life-insurance/README.md`
- Create: `docs/superpowers/plans/2026-08-12-life-insurance-underwriting-exception-spec.md`

**Interfaces:**
- Consumes: `life-insurance/agentic-workflow-opportunities.md`, `reference-solution/claims-process-flow.md`, `reference-solution/domain-demo-spec-template.md`, merged `commercial-banking/payment-exception-demo-spec.md` and `retail/product-recall-containment-demo-spec.md` patterns, GitHub issue #25, and read-only UiPath CLI evidence from `uipathlabs / Playground`.
- Produces: a complete life-insurance demo contract, an explicit implementation sequence, and a concise README entry point for later implementation.

- [ ] **Step 1: Confirm recommendation and evidence boundaries**

  Record underwriting evidence-exception orchestration as the selected use case because it ranks first at 4.75/5 and combines documents, consent-aware evidence, deterministic rules, bounded agents, API/RPA work, human authority, parallel completion, and auditability. State that sector evidence does not establish a carrier baseline, target, system, policy, or production readiness.

- [ ] **Step 2: Write the typed demo contract**

  Create `life-insurance/underwriting-evidence-exception-demo-spec.md` with the selected use case and trade-off, narrative, personas and authority boundaries, `UnderwritingApplicationInput`, typed output schema, `applicationId` idempotency, synthetic-data classification, success outcome, measurable signals, and explicit out-of-scope items.

- [ ] **Step 3: Map the full reference topology**

  Define four blue canvas segments named `Receive and validate application`, `Gather and reconcile evidence`, `Recommend and underwrite`, and `Record decision and notify`. Put consent, intake, extraction, evidence, review, timeout, and write errors below the happy path. Specify the real straight-through expression, named human outcomes, distinct adverse control, parallel completion branches, merge, and receipt reconciliation.

- [ ] **Step 4: Specify actors, tools, and readiness**

  Document `LifeUnderwritingPacketExtraction`, `underwriting-evidence-gateway`, `legacy-underwriting-console`, `Underwriting Evidence Reconciler`, LangGraph `Medical Evidence Chronology`, its least-privilege MCP evidence tool, `Underwriting Evidence Review`, `LifeUnderwritingCases`, the synthetic evidence services, the guide index, and their exact inputs, outputs, authority limits, typed failures, owners, mock fallbacks, and readiness gaps.

- [ ] **Step 5: Preserve and prove the Azure showcase contract**

  Specify node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` and connection `0107247a-0197-42c9-b957-05d1b722b111` with the constant message `UiPath Flow external-agent connectivity showcase for life insurance`, connection-selected `agent_id`, omitted `thread_id`, discarded response, no case or sensitive data, a disabled-by-default flag, short timeout/error continuation, and identical business state/routes for off, on, and failing variants.

- [ ] **Step 6: Specify safety, recovery, evaluation, and the demo proof**

  Add authorization-before-acquisition, adverse-outcome, grounding, access, agent-boundary, receipt-truth, and Azure-isolation controls. Add typed recovery paths, dataset `life-insurance-underwriting-exceptions-v1` with five exact cases, route/authorization/grounding/MCP/adverse/isolation/receipt evaluators, an eight-step demo script, success measures, and a complete reference-mapping table.

- [ ] **Step 7: Separate human decisions from build work**

  Add an owned human-decision table for carrier scope, rules/thresholds, authorization/privacy, systems of record, API gap, vendors/models, reviewer authority/SLA, resource provisioning, pilot measures, and process-app selection. Follow it with a distinct ordered implementation-task list and a quality score of at least 10/12 with no zero.

- [ ] **Step 8: Link the specification**

  Add `Underwriting evidence-exception demo specification` to `life-insurance/README.md` beside the existing research link.

- [ ] **Step 9: Validate completeness, isolation, and exact scope**

  Run:

  ```bash
  rg '\[(Describe|State|Event source|System of record|Documents|Business result|Title|Responsibility|Schema|Project/model|Real business|Least privilege|Measurable|Mapping|Evidence/gap|Owner and resolution)|T[B]D|T[O]DO' life-insurance/underwriting-evidence-exception-demo-spec.md
  rg '^## ' life-insurance/underwriting-evidence-exception-demo-spec.md
  rg '0107247a-0197-42c9-b957-05d1b722b111|static|thread_id|discard|unchanged|isolation' life-insurance/underwriting-evidence-exception-demo-spec.md
  rg 'UnderwritingApplicationInput|applicationId|ApproveEdited|ApproveAdverse|life-insurance-underwriting-exceptions-v1' life-insurance/underwriting-evidence-exception-demo-spec.md
  git diff --check
  git diff --name-only
  ```

  Expected: the placeholder scan has no matches; every required section, typed contract, named human result, external-agent isolation term, and evaluation contract is present; whitespace is clean; and only the three issue-scoped files changed.

- [ ] **Step 10: Review against issue #25 and the reference checklist**

  Confirm every acceptance criterion has direct evidence; all four reference segments and every actor type are mapped; the quality score is at least 10/12 with no zero; open human decisions are distinct from implementation tasks; all readiness claims identify their evidence date and target; mocks replace unverified resources; and no production-readiness, legal, carrier-performance, or ROI claim exceeds the evidence.

- [ ] **Step 11: Commit the exact files**

  ```bash
  git add life-insurance/README.md life-insurance/underwriting-evidence-exception-demo-spec.md docs/superpowers/plans/2026-08-12-life-insurance-underwriting-exception-spec.md
  git diff --cached --check
  git diff --cached --name-only
  git commit -m "Specify life underwriting evidence exception demo"
  ```

  Expected: the staged scope contains exactly the three listed files and the commit succeeds on `agent/issue-25-life-insurance-spec`.
