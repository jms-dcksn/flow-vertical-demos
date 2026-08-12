# Commercial Banking Payment Exception Spec Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Complete GitHub issue #16 with an implementation-ready specification for the recommended commercial-banking Maestro Flow demo.

**Architecture:** Create one template-aligned domain specification that selects the top-ranked payment-exception use case and turns the existing research into explicit Flow, actor, data, human-review, safety, evaluation, and delivery contracts. Keep research evidence separate and link the new spec from the domain README.

**Tech Stack:** Markdown, repository reference pattern, GitHub issue acceptance criteria.

## Global Constraints

- Scope changes to issue #16, its commercial-banking specification, and the deliberately reusable external-agent rule in the reference pattern and template.
- Preserve the research document as the evidence and recommendation source.
- Design one independently deployable UiPath Solution with exactly one `.uipx` manifest.
- Use synthetic payment and screening data; no live payment action is permitted.
- Bind the shared Azure AI Foundry connection `0107247a-0197-42c9-b957-05d1b722b111` only on a non-material showcase branch with static input, discarded output, and fail-open continuation.
- Keep open human decisions separate from implementation tasks.

---

### Task 1: Author and validate the payment-exception demo specification

**Files:**
- Create: `commercial-banking/payment-exception-demo-spec.md`
- Modify: `commercial-banking/README.md`
- Create: `docs/superpowers/plans/2026-08-12-commercial-banking-payment-exception-spec.md`
- Modify: `reference-solution/claims-process-flow.md`
- Modify: `reference-solution/domain-demo-spec-template.md`

**Interfaces:**
- Consumes: `commercial-banking/agentic-workflow-opportunities.md`, `reference-solution/claims-process-flow.md`, and `reference-solution/domain-demo-spec-template.md`.
- Produces: a complete domain demo contract and a README entry point for later implementation work.

- [ ] **Step 1: Write the specification**

  Fill every template section with explicit payment-exception contracts, including the four reference segments, actor boundaries, real branch expressions, human outcomes, evaluation cases, solution layout, readiness gaps, and owners. Add the shared Azure AI Foundry connection as a display-only branch that cannot affect core case data or routing.

- [ ] **Step 2: Link the specification from the domain README**

  Add `Payment exception demo specification` beside the existing research link.

- [ ] **Step 3: Check scope and completeness**

  Run:

  ```bash
  rg '\[(Describe|State|Event source|System of record|Documents|Business result|Title|Responsibility|Schema|Project/model|Real business|Least privilege|Measurable|Mapping|Evidence/gap|Owner and resolution)|TBD|TODO' commercial-banking/payment-exception-demo-spec.md
  rg '^## ' commercial-banking/payment-exception-demo-spec.md
  git diff --check
  ```

  Expected: the placeholder scan returns no matches, every required section is present, and `git diff --check` exits successfully.

- [ ] **Step 4: Review against issue #16**

  Confirm each acceptance criterion has direct evidence in the specification and that open human decisions are not mixed with implementation tasks.

- [ ] **Step 5: Commit**

  ```bash
  git add commercial-banking/payment-exception-demo-spec.md commercial-banking/README.md docs/superpowers/plans/2026-08-12-commercial-banking-payment-exception-spec.md
  git commit -m "Specify commercial banking payment exception demo"
  ```
