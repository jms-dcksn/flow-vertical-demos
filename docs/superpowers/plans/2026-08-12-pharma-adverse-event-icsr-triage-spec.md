# Pharma Adverse-Event ICSR Triage Spec Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Complete GitHub issue #28 with an implementation-ready specification for the recommended Pharma Maestro Flow demo.

**Architecture:** Promote the top-ranked adverse-event intake and ICSR triage recommendation into one template-aligned domain specification. Keep research evidence separate and define explicit Flow, actor, data, review, regulatory-safety, recovery, evaluation, external-agent isolation, solution-boundary, and delivery contracts in the new spec.

**Tech Stack:** Markdown, repository reference pattern, UiPath CLI 1.199.0 tenant/resource inspection, GitHub issue acceptance criteria.

## Global Constraints

- Scope changes to GitHub issue #28 and exactly three files: the Pharma specification, Pharma README, and this plan.
- Preserve `pharma/agentic-workflow-opportunities.md` as the evidence and recommendation source.
- Design one independently deployable UiPath Solution with exactly one `.uipx` manifest and the CLI-supported nested Flow layout.
- Use a synthetic US post-market human-drug case, synthetic labels/terms/rules, local mocks, and preview-only communication; no real case processing, reporter contact, safety-system write, or regulator submission is permitted.
- Bind Azure AI Foundry connection `0107247a-0197-42c9-b957-05d1b722b111` only on a non-material showcase branch with constant input, no `thread_id`, discarded output, and fail-open continuation.
- Keep open human decisions separate from implementation tasks.
- Preserve `reference-solution/claims-process-flow.md`, `reference-solution/domain-demo-spec-template.md`, and every non-Pharma domain unchanged.
- Keep README changes concise and use no emojis in documentation.

---

### Task 1: Author and validate the adverse-event ICSR triage specification

**Files:**
- Create: `pharma/adverse-event-icsr-triage-demo-spec.md`
- Modify: `pharma/README.md`
- Create: `docs/superpowers/plans/2026-08-12-pharma-adverse-event-icsr-triage-spec.md`

**Interfaces:**
- Consumes: `pharma/agentic-workflow-opportunities.md`, `reference-solution/claims-process-flow.md`, `reference-solution/domain-demo-spec-template.md`, GitHub issue #28, and the merged commercial-banking/retail specification patterns.
- Produces: a complete Pharma demo contract and a concise README entry point for later implementation.

- [ ] **Step 1: Write the specification**

  Create `pharma/adverse-event-icsr-triage-demo-spec.md` with the selected use case and rationale, narrative, personas, typed trigger/case contract, four reference segments, intended node sequence, actor/tool boundaries, resources/mocks, HITL, controls, recovery, observability, five-case evaluation set, demo script, success measures, full reference mapping, separate human decisions/tasks, and quality rubric.

- [ ] **Step 2: Record verified readiness without overclaiming**

  Record `uip` 1.199.0 and authenticated target `uipathlabs / Playground`; target folder `JD_Demos/demos`; the absence of a matching Pharma IXP/ICSR/safety-case resource in targeted searches; and the presence of Outlook connector capability without silently selecting a connection. Treat every unverified domain resource as new or mocked.

- [ ] **Step 3: Preserve the shared external-agent contract**

  Specify node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` with connection `0107247a-0197-42c9-b957-05d1b722b111`, constant Pharma showcase text, no `thread_id`, discarded output, no case/sensitive data, a disabled-by-default flag, short timeout/error continuation, and identical business state/routes for off, on, and failing variants.

- [ ] **Step 4: Link the specification**

  Add `Adverse-event ICSR triage demo specification` to `pharma/README.md` beside the existing research link.

- [ ] **Step 5: Validate placeholders, sections, mapping, Azure isolation, and scope**

  Run:

  ```bash
  test "$(find pharma -maxdepth 1 -name '*-demo-spec.md' -print | wc -l | tr -d ' ')" = 1
  ! rg '\[(Describe|State|Event source|System of record|Documents|Business result|Title|Responsibility|Schema|Project/model|Real business|Least privilege|Measurable|Mapping|Evidence/gap|Owner and resolution)|TBD|TODO|<PLACEHOLDER>' pharma/adverse-event-icsr-triage-demo-spec.md
  rg '^## (Use case and narrative|Personas|Trigger and case contract|Flow topology|Agentic reasoning and tool use|Data and resources|Human decisions in the Flow|Controls and safety|Error paths and recovery|Observability and evaluation|Demo script|Success measures|Reference mapping|Open human decisions|Implementation tasks|Quality rubric)$' pharma/adverse-event-icsr-triage-demo-spec.md
  rg '^\| (3-4 segment topology and canvas rules|IXP/document intelligence|API workflow and RPA on the intended path|Inline agent with a wired tool|Coded agent with visible value-add|Shared external-agent showcase|Real business decision and safe exception|Human decision and returned outcome data|Purposeful parallelism and merge|Evaluation set and evaluator|Process-app variant|Solution boundary and delivery contract) \|' pharma/adverse-event-icsr-triage-demo-spec.md
  rg 'uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread|0107247a-0197-42c9-b957-05d1b722b111|constant message|omit `thread_id`|Discard the response|unchanged|off, on, and failing' pharma/adverse-event-icsr-triage-demo-spec.md
  git diff --check
  git status --short
  git diff --name-only
  ```

  Expected: exactly one Pharma demo spec exists; placeholder scan exits zero through `!`; every required section and all twelve mapping rows are present; Azure isolation terms are present; whitespace is clean; and only the three issue-scoped files changed.

- [ ] **Step 6: Review evidence, decisions, and safety boundaries**

  Confirm every issue #28 acceptance criterion has direct evidence, the quality score is at least 10/12 with no zero, human decisions are distinct from implementation tasks, the Azure branch cannot affect business state, unverified resources are mocks or owned gaps, and the spec makes no regulated-use, submission, production-readiness, benefit, or ROI claim beyond the research.

- [ ] **Step 7: Commit only the explicit files**

  ```bash
  git add pharma/README.md pharma/adverse-event-icsr-triage-demo-spec.md docs/superpowers/plans/2026-08-12-pharma-adverse-event-icsr-triage-spec.md
  git diff --cached --check
  git diff --cached --name-only
  git commit -m "Specify pharma adverse-event ICSR triage demo"
  ```
