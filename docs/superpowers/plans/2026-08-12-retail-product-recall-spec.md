# Retail Product Recall Spec Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Complete GitHub issue #29 with an implementation-ready specification for the recommended retail Maestro Flow demo.

**Architecture:** Promote the existing top-ranked product-recall recommendation into one template-aligned domain specification. Keep the research as the evidence source and define explicit Flow, actor, data, review, safety, evaluation, external-agent isolation, solution-boundary, and delivery contracts in the new spec.

**Tech Stack:** Markdown, repository reference pattern, UiPath CLI 1.199.0 tenant/resource inspection, GitHub issue acceptance criteria.

## Global Constraints

- Scope changes to issue #29 and the retail domain.
- Preserve `retail/agentic-workflow-opportunities.md` as the research and recommendation source.
- Design one independently deployable UiPath Solution with exactly one `.uipx` manifest and the CLI-supported nested Flow layout.
- Use public CPSC information plus synthetic supplier, product, inventory, order, and customer data; no real operational or customer action is permitted.
- Bind Azure AI Foundry connection `0107247a-0197-42c9-b957-05d1b722b111` only on a non-material showcase branch with static input, discarded output, and fail-open continuation.
- Keep open human decisions separate from implementation tasks.
- Keep README changes concise and use no emojis in documentation.

---

### Task 1: Author and validate the product-recall demo specification

**Files:**
- Create: `retail/product-recall-containment-demo-spec.md`
- Modify: `retail/README.md`
- Create: `docs/superpowers/plans/2026-08-12-retail-product-recall-spec.md`

**Interfaces:**
- Consumes: `retail/agentic-workflow-opportunities.md`, `reference-solution/claims-process-flow.md`, `reference-solution/domain-demo-spec-template.md`, GitHub issue #29, and the merged PR #46 pattern.
- Produces: a complete retail demo contract and a concise README entry point for later implementation.

- [ ] **Step 1: Write the specification**

  Create `retail/product-recall-containment-demo-spec.md` with the selected use case, narrative, personas, typed trigger/case contract, four reference segments, actor/tool boundaries, resources, HITL, controls, recovery, observability, five-case evaluation set, demo script, success measures, reference mapping, separate human decisions/tasks, and quality rubric.

- [ ] **Step 2: Preserve the shared external-agent contract**

  Specify `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` with connection `0107247a-0197-42c9-b957-05d1b722b111`, constant retail showcase text, no `thread_id`, discarded output, no case data, a disabled-by-default flag, and identical business state/routes for off, on, and failing variants.

- [ ] **Step 3: Link the specification**

  Add `Product recall containment demo specification` to `retail/README.md` beside the existing research link.

- [ ] **Step 4: Validate completeness and scope**

  Run:

  ```bash
  rg '\[(Describe|State|Event source|System of record|Documents|Business result|Title|Responsibility|Schema|Project/model|Real business|Least privilege|Measurable|Mapping|Evidence/gap|Owner and resolution)|TBD|TODO' retail/product-recall-containment-demo-spec.md
  rg '^## ' retail/product-recall-containment-demo-spec.md
  rg '0107247a-0197-42c9-b957-05d1b722b111|static|discard|unchanged|isolation' retail/product-recall-containment-demo-spec.md
  git diff --check
  git diff --name-only
  ```

  Expected: the placeholder scan has no matches; required sections and external-agent isolation evidence are present; whitespace is clean; and only the three issue-scoped files changed.

- [ ] **Step 5: Review against issue #29**

  Confirm every acceptance criterion has direct evidence, the quality score is at least 10/12 with no zero, open human decisions are distinct from implementation tasks, and no production-readiness or ROI claim exceeds the evidence.

- [ ] **Step 6: Commit**

  ```bash
  git add retail/README.md retail/product-recall-containment-demo-spec.md docs/superpowers/plans/2026-08-12-retail-product-recall-spec.md
  git commit -m "Specify retail product recall containment demo"
  ```
