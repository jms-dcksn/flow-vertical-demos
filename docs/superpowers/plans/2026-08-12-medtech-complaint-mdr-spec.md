# Medtech Complaint-to-MDR Spec Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Complete GitHub issue #26 with an implementation-ready specification for the recommended Medtech Maestro Flow demo.

**Architecture:** Promote the top-ranked complaint-to-MDR recommendation into one template-aligned domain specification. Keep the research as the evidence source and define explicit Flow, actor, data, human-authority, safety, recovery, evaluation, external-agent isolation, solution-boundary, and delivery contracts without executing or deploying a Flow.

**Tech Stack:** Markdown, repository reference pattern, UiPath CLI 1.199.0 read-only tenant/resource inspection, GitHub issue #26 acceptance criteria.

## Global Constraints

- Scope changes to issue #26 and the Medtech domain only.
- Preserve `medtech/opportunity-research.md` as the evidence and recommendation source.
- Design one independently deployable UiPath Solution with exactly one `.uipx` manifest and the CLI-supported nested Flow layout.
- Use synthetic complaint, patient, reporter, device, policy, eQMS, and regulator data; no clinical conclusion, real communication, quality action, or regulator submission is permitted.
- Keep regulatory clock calculation deterministic and final reportability authority with a qualified human.
- Bind Azure AI Foundry connection `0107247a-0197-42c9-b957-05d1b722b111` only on a non-material showcase branch with static input, no `thread_id`, discarded output, and fail-open continuation.
- Keep open human decisions separate from implementation tasks.
- Preserve the shared Azure showcase isolation contract; do not modify reference-solution files.
- Deploy the solution and its resources under the approved Playground parent `JD_Demos/demos`.
- Keep README changes concise and use no emojis in documentation.
- Do not run `uip maestro flow debug`, upload, publish, deploy, or make external writes for this specification issue.

---

### Task 1: Author and validate the complaint-to-MDR demo specification

**Files:**
- Create: `medtech/complaint-to-mdr-demo-spec.md`
- Modify: `medtech/README.md`
- Create: `docs/superpowers/plans/2026-08-12-medtech-complaint-mdr-spec.md`

**Interfaces:**
- Consumes: `medtech/opportunity-research.md`, `reference-solution/claims-process-flow.md`, `reference-solution/domain-demo-spec-template.md`, GitHub issue #26, and the merged commercial-banking/retail specification patterns.
- Produces: a complete Medtech demo contract and a concise README entry point for later implementation.

- [ ] **Step 1: Record the recommendation and evidence boundary**

  Select complaint-to-MDR decision orchestration, cite its 14/15 research ranking and comparison with the next candidates, and state that public evidence supports the workflow/deadlines but not manufacturer-specific policy, readiness, ROI, or production authority.

- [ ] **Step 2: Write the implementation-ready specification**

  Create `medtech/complaint-to-mdr-demo-spec.md` with narrative, personas, typed trigger/input/output contracts, four reference segments, actor/tool contracts, mocked integrations, queue and solution data model, HITL return schema, safety controls, owned recovery, observability, five-case evaluation set, demo script, success measures, complete reference mapping, separate human decisions/tasks, and quality rubric.

- [ ] **Step 3: Preserve the external-agent isolation contract**

  Specify node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` and connection `0107247a-0197-42c9-b957-05d1b722b111` with constant Medtech showcase text, no `thread_id`, no case or sensitive data, discarded response, disabled-by-default flag, timeout/error continuation, and identical core case state/routes for off, on, and failing variants.

- [ ] **Step 4: Link the specification**

  Add `Complaint-to-MDR decision demo specification` to `medtech/README.md` beside the existing research link.

- [ ] **Step 5: Validate placeholders, required sections, mapping, and Azure isolation**

  Run:

  ```bash
  rg '\[(Describe|State|Event source|System of record|Documents|Business result|Title|Responsibility|Schema|Project/model|Real business|Least privilege|Measurable|Mapping|Evidence/gap|Owner and resolution)|TBD|TODO' medtech/complaint-to-mdr-demo-spec.md
  rg '^## (Use case and narrative|Personas|Trigger and case contract|Flow topology|Agentic reasoning and tool use|Data and resources|Human decisions|Controls and safety|Error paths and recovery|Observability and evaluation|Demo script|Success measures|Reference mapping|Open human decisions|Implementation tasks|Quality rubric)$' medtech/complaint-to-mdr-demo-spec.md
  rg '^\| (Use case and hero moment|3-4 segment topology and canvas rules|Trigger|IXP/document intelligence|API workflow and RPA on the intended path|Inline agent with a wired tool|Coded agent with visible value-add|Shared external-agent showcase|Real business decision and safe exception|Human decision and returned outcome data|Purposeful parallelism and merge|MCP server/tool|Data model|Evaluation set and evaluator|Process-app variant|Solution boundary and delivery contract) \|' medtech/complaint-to-mdr-demo-spec.md
  rg 'uipath\.connector\.uipath-microsoft-azureaifoundry\.execute-the-thread|0107247a-0197-42c9-b957-05d1b722b111|constant message|no `thread_id`|response is discarded|disabled by default|unchanged core route|off, on, or failing' medtech/complaint-to-mdr-demo-spec.md
  rg '\*\*Total\*\*.*10/12' medtech/complaint-to-mdr-demo-spec.md
  rg 'Complaint-to-MDR decision demo specification' medtech/README.md
  git diff --check
  git diff --name-only
  git status --short
  ```

  Expected: the placeholder scan exits 1 with no matches; all 16 required top-level sections and 16 mapping rows are printed; the Azure scan shows the exact node/connection plus constant-input, discarded-output, disabled/failure, and unchanged-route evidence; the rubric is 10/12 with no zero; the README link is present; whitespace is clean; and only the three issue-scoped files are changed.

- [ ] **Step 6: Review issue and evidence coverage**

  Confirm every issue #26 acceptance criterion has direct evidence; all reference actors have readiness/fallback contracts; clocks, human authority, recovery, evaluation thresholds, and demo proof are testable; `JD_Demos/demos` is stated directly as the approved deployment parent rather than an open decision; open human decisions are distinct from implementation tasks; all unavailable resources are explicit mocks/contracts; and no production-readiness, legal, incidence, causality, benefit, or ROI claim exceeds `medtech/opportunity-research.md`.

- [ ] **Step 7: Commit the explicit scope**

  ```bash
  git add medtech/README.md medtech/complaint-to-mdr-demo-spec.md docs/superpowers/plans/2026-08-12-medtech-complaint-mdr-spec.md
  git diff --cached --check
  git diff --cached --name-only
  git commit -m "Specify medtech complaint-to-MDR demo"
  ```

  Expected: the staged scope contains exactly the three listed files, the staged diff is whitespace-clean, and the commit succeeds on `agent/issue-26-medtech-spec`.
