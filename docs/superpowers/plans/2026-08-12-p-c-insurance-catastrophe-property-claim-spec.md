# P&C Insurance Catastrophe Property-Claim Spec Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Complete GitHub issue #27 with an implementation-ready specification for the recommended P&C insurance Maestro Flow demo.

**Architecture:** Promote the top-ranked catastrophe property-claim and advance-payment recommendation into one template-aligned domain specification. Keep public research as the evidence source and define explicit Flow, actor, data, review, safety, recovery, evaluation, external-agent isolation, solution-boundary, and delivery contracts in the new specification.

**Tech Stack:** Markdown, repository reference pattern, UiPath CLI 1.199.0 tenant/resource inspection, GitHub issue acceptance criteria.

## Global Constraints

- Scope changes to GitHub issue #27 and exactly three files: the P&C insurance specification, domain README, and this plan.
- Preserve `p-c-insurance/agentic-workflow-opportunities.md` as the research and recommendation source.
- Design one independently deployable UiPath Solution with exactly one `.uipx` manifest and the CLI-supported nested Flow layout.
- Use synthetic claim, claimant, property, policy, evidence, rule, and system data; no live coverage, payment, vendor, or communication action is permitted.
- Treat all unverified domain resources as build targets with contract-compatible mocks.
- Bind Azure AI Foundry connection `0107247a-0197-42c9-b957-05d1b722b111` only on a non-material showcase branch with static input, discarded output, and fail-open continuation.
- Preserve the shared Azure showcase's isolation from case state, review routing, human decisions, downstream actions, receipts, and final status.
- Keep open human decisions separate from implementation tasks.
- Keep README changes concise and use no emojis in documentation.

---

### Task 1: Author the catastrophe property-claim specification

**Files:**
- Create: `p-c-insurance/catastrophe-property-claim-demo-spec.md`
- Modify: `p-c-insurance/README.md`
- Create: `docs/superpowers/plans/2026-08-12-p-c-insurance-catastrophe-property-claim-spec.md`

**Interfaces:**
- Consumes: `p-c-insurance/agentic-workflow-opportunities.md`, `reference-solution/claims-process-flow.md`, `reference-solution/domain-demo-spec-template.md`, GitHub issue #27, and the merged commercial-banking/retail specification patterns.
- Produces: a complete P&C insurance demo contract, an implementation sequence, and a concise README entry point.

- [ ] **Step 1: Record the selected use case and evidence boundary**

  Select the 14/15 catastrophe property-claim coverage and advance-payment candidate. State why it is stronger than the 13/15 underwriting alternative, cite the existing research rather than duplicating it, and distinguish public evidence/design inference from carrier-verified facts and ROI.

- [ ] **Step 2: Define the complete business and data contract**

  Write the narrative, personas, hero moment, success outcome, measurable pilot signals, typed `CatastropheClaimInput`, canonical queue/idempotency contract, typed outputs, data classifications, and explicit non-goals. Use `claimId` and `correlationId` in every task, trace, exception, and receipt.

- [ ] **Step 3: Map the four-segment Flow and every reference actor**

  Specify the four named sticky-note segments, left-to-right happy path, exception paths below it, deterministic specialist/authority/receipt expressions, IXP, API workflow, purposeful UI-only RPA, inline agent with read-only tool, coded agent with MCP tools, coded action app, parallel approved actions, merge, and canonical end states.

- [ ] **Step 4: Preserve the shared external-agent contract**

  Specify node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` with connection `0107247a-0197-42c9-b957-05d1b722b111`, constant P&C showcase text, no `thread_id`, discarded output, no claim data, a disabled-by-default flag, and identical business case/routes for off, on, and failing variants.

- [ ] **Step 5: Define implementation safety and proof**

  Add contract-compatible mocks for unverified resources, HITL outcomes and authority boundaries, recovery owners/conditions, observability, exact initial evaluator thresholds, a five-case synthetic set, an under-ten-minute demo script, success measures, the complete reference mapping, and a quality score of at least 10/12 with no zero.

- [ ] **Step 6: Separate decisions from work and link the specification**

  Put carrier, legal, operational, security, resource, and process-app choices under `Open human decisions`; put build steps under `Implementation tasks`. Add `Catastrophe property-claim demo specification` to `p-c-insurance/README.md` beside the research link.

- [ ] **Step 7: Validate placeholders, required sections, mapping, and Azure isolation**

  Run:

  ```bash
  rg '\[(Describe|State|Event source|System of record|Documents|Business result|Title|Responsibility|Schema|Project/model|Real business|Least privilege|Measurable|Mapping|Evidence/gap|Owner and resolution)|TBD|TODO' p-c-insurance/catastrophe-property-claim-demo-spec.md
  rg '^## (Use case and narrative|Personas|Trigger and case contract|Flow topology|Agentic reasoning and tool use|Data and resources|Human decisions|Controls and safety|Error paths and recovery|Observability and evaluation|Demo script|Success measures|Reference mapping|Open human decisions|Implementation tasks|Quality rubric)$' p-c-insurance/catastrophe-property-claim-demo-spec.md
  rg '^\| (Use case and hero moment|3-4 segment topology and canvas rules|IXP/document intelligence|API workflow and RPA on the intended path|Inline agent with a wired tool|Coded agent with visible value-add|Shared external-agent showcase|Real business decision and safe exception|Human decision and returned outcome data|Purposeful parallelism and merge|Evaluation set and evaluator|Process-app variant|Solution boundary and delivery contract) \|' p-c-insurance/catastrophe-property-claim-demo-spec.md
  rg 'uipath\.connector\.uipath-microsoft-azureaifoundry\.execute-the-thread|0107247a-0197-42c9-b957-05d1b722b111|constant|discard|unchanged|isolation' p-c-insurance/catastrophe-property-claim-demo-spec.md
  git diff --check
  git status --short
  ```

  Expected: the placeholder scan returns no matches; all 16 required top-level sections and all 13 reference-mapping rows are present; Azure node/connection, static-data, discarded-output, unchanged-route, and isolation evidence are present; whitespace is clean; and status lists only the three issue-scoped files.

- [ ] **Step 8: Review issue #27 and delivery claims**

  Confirm every issue acceptance criterion has direct evidence, human decisions are separate from implementation tasks, no unverified domain resource is presented as ready, no production-readiness or ROI claim exceeds the evidence, and the Azure showcase is the only verified tenant resource claim.

- [ ] **Step 9: Commit the explicit files**

  ```bash
  git add p-c-insurance/catastrophe-property-claim-demo-spec.md p-c-insurance/README.md docs/superpowers/plans/2026-08-12-p-c-insurance-catastrophe-property-claim-spec.md
  git commit -m "Specify P&C catastrophe property-claim demo"
  ```
