# Healthcare Provider Abnormal Result Follow-Up Spec Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Complete GitHub issue #23 with an implementation-ready specification for the recommended healthcare-provider Maestro Flow demo.

**Architecture:** Promote the top-ranked closed-loop abnormal diagnostic-result recommendation into one template-aligned domain specification, narrowed to a synthetic ambulatory-radiology demo. Keep the research as the evidence source and define explicit Flow, actor, data, clinical-review, safety, evaluation, external-agent isolation, solution-boundary, and delivery contracts in the new spec.

**Tech Stack:** Markdown, repository reference pattern, UiPath CLI 1.199.0 tenant/auth/Flow-capability inspection, GitHub issue #23 acceptance criteria.

## Global Constraints

- Scope changes to GitHub issue #23 and the healthcare-provider domain only.
- Preserve `healthcare-provider/agentic-workflow-opportunities.md` as the research and recommendation source.
- Design one independently deployable UiPath Solution with exactly one `.uipx` manifest and the CLI-supported nested Flow layout.
- Deploy the solution and its resources to the approved UiPath Labs Playground parent `JD_Demos/demos`.
- Use synthetic, de-identified clinical data and mock write targets; no model may diagnose, set urgency, select treatment, acknowledge for a clinician, send patient advice, or write to a live clinical system.
- Bind Azure AI Foundry connection `0107247a-0197-42c9-b957-05d1b722b111` only on a non-material showcase branch with static input, no `thread_id`, discarded output, and fail-open continuation that preserves core business state.
- Keep open human decisions separate from implementation tasks.
- Keep README changes concise and use no emojis in documentation.
- Do not execute a Flow, upload a solution, provision a tenant resource, or perform any external write while producing the specification.

---

### Task 1: Author the abnormal-result follow-up specification

**Files:**
- Create: `healthcare-provider/abnormal-diagnostic-result-follow-up-demo-spec.md`
- Modify: `healthcare-provider/README.md`
- Create: `docs/superpowers/plans/2026-08-12-healthcare-provider-abnormal-result-follow-up-spec.md`

**Interfaces:**
- Consumes: `healthcare-provider/agentic-workflow-opportunities.md`, `reference-solution/claims-process-flow.md`, `reference-solution/domain-demo-spec-template.md`, `commercial-banking/payment-exception-demo-spec.md`, `retail/product-recall-containment-demo-spec.md`, and GitHub issue #23.
- Produces: a complete healthcare-provider demo contract, a concise README entry point, and an exact validation/commit checklist for later review.

- [ ] **Step 1: Select and narrow the recommended use case**

  Select closed-loop abnormal diagnostic-result follow-up because it ranks first at 15/15 and has the clearest safety ownership, deadline/escalation, human-decision, and synthetic-demo story. Narrow the build to ambulatory radiology while keeping result subtype, local clinical policy, and production integration as explicitly owned human decisions.

- [ ] **Step 2: Write the full specification**

  Create `healthcare-provider/abnormal-diagnostic-result-follow-up-demo-spec.md` with the reason for selection, narrative, personas, typed trigger/case contract, four reference segments, actor/tool boundaries, resources/mocks, HITL contract, controls, recovery, observability, five-case evaluation set, demo script, success measures, reference mapping, separate human decisions/tasks, and quality rubric.

- [ ] **Step 3: Preserve external-agent isolation**

  Specify node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` with connection `0107247a-0197-42c9-b957-05d1b722b111`, the constant message `UiPath Flow external-agent connectivity showcase for healthcare provider`, no `thread_id`, discarded output, no case/clinical data, a disabled-by-default flag, short timeout/error continuation, and identical case state/routes for off, on, and failing variants.

- [ ] **Step 4: Link the specification**

  Add `Abnormal diagnostic result follow-up demo specification` to `healthcare-provider/README.md` beside the existing opportunity-research link.

- [ ] **Step 5: Validate placeholder-free template coverage**

  Run:

  ```bash
  rg '\[(Describe|State|Event source|System of record|Documents|Business result|Title|Responsibility|Schema|Project/model|Real business|Least privilege|Measurable|Mapping|Evidence/gap|Owner and resolution)|TBD|TODO' healthcare-provider/abnormal-diagnostic-result-follow-up-demo-spec.md
  rg '^## (Use case and narrative|Personas|Trigger and case contract|Flow topology|Agentic reasoning and tool use|Data and resources|Human decisions|Controls and safety|Error paths and recovery|Observability and evaluation|Demo script|Success measures|Reference mapping|Open human decisions|Implementation tasks|Quality rubric)$' healthcare-provider/abnormal-diagnostic-result-follow-up-demo-spec.md
  rg 'uipath\.connector\.uipath-microsoft-azureaifoundry\.execute-the-thread|0107247a-0197-42c9-b957-05d1b722b111|static|constant|discard|unchanged|isolation' healthcare-provider/abnormal-diagnostic-result-follow-up-demo-spec.md
  ```

  Expected: the first command returns no matches; all 16 required sections are present; and the external-agent command shows the node, connection, constant/static input, discarded output, unchanged state, and isolation proof.

- [ ] **Step 6: Validate issue scope and formatting**

  Run:

  ```bash
  git diff --check
  git status --short
  ```

  Expected: whitespace validation succeeds and `git status --short` lists only these files:

  ```text
  docs/superpowers/plans/2026-08-12-healthcare-provider-abnormal-result-follow-up-spec.md
  healthcare-provider/README.md
  healthcare-provider/abnormal-diagnostic-result-follow-up-demo-spec.md
  ```

- [ ] **Step 7: Review against GitHub issue #23**

  Confirm the specification directly explains the chosen use case; defines narrative, personas, trigger/input, outcome, and value; maps every reference element; specifies topology, tools, integrations/mocks, data/resources, HITL, error paths, observability, evaluation, and demo script; separates human decisions from implementation tasks; scores at least 10/12 with no zero; preserves the Azure showcase contract; and makes no production-readiness, clinical, or ROI claim beyond the evidence.

- [ ] **Step 8: Commit the scoped documentation**

  ```bash
  git add healthcare-provider/README.md healthcare-provider/abnormal-diagnostic-result-follow-up-demo-spec.md docs/superpowers/plans/2026-08-12-healthcare-provider-abnormal-result-follow-up-spec.md
  git commit -m "Specify healthcare provider result follow-up demo"
  ```
