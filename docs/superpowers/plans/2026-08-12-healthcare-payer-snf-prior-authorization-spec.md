# Healthcare Payer SNF Prior-Authorization Spec Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Complete GitHub issue #20 with an implementation-ready specification for the recommended healthcare-payer Maestro Flow demo.

**Architecture:** Promote the research's top-ranked post-acute SNF prior-authorization recommendation into one template-aligned domain specification. Keep the research as the evidence source and define explicit Flow topology, typed contracts, actor/tool boundaries, two-level clinical review, safe recovery, evaluation, external-agent isolation, solution boundary, and delivery readiness in the new spec.

**Tech Stack:** Markdown, repository reference pattern, UiPath CLI 1.199.0 tenant/resource inspection, GitHub issue acceptance criteria.

## Global Constraints

- Scope changes to GitHub issue #20 and the healthcare-payer domain only.
- Preserve `healthcare-payer/research.md` as the evidence and recommendation source.
- Design one independently deployable UiPath Solution with exactly one `.uipx` manifest and the CLI-supported nested Flow layout.
- Use synthetic member, provider, clinical, policy, and system data; no production clinical decision or notification is permitted.
- A utilization-management nurse owns approval and information requests; a medical director owns every denial or reduction.
- Bind Azure AI Foundry connection `0107247a-0197-42c9-b957-05d1b722b111` only on a non-material showcase branch with static input, no `thread_id`, discarded output, and fail-open continuation.
- Keep open human decisions separate from implementation tasks.
- Record the repository `JD/demos` versus authenticated CLI `JD_Demos/demos` discrepancy without silently choosing a deployment path.
- Keep README changes concise and use no emojis in documentation.

---

### Task 1: Author and validate the SNF prior-authorization demo specification

**Files:**
- Create: `healthcare-payer/snf-prior-authorization-demo-spec.md`
- Modify: `healthcare-payer/README.md`
- Create: `docs/superpowers/plans/2026-08-12-healthcare-payer-snf-prior-authorization-spec.md`

**Interfaces:**
- Consumes: `healthcare-payer/research.md`, `reference-solution/claims-process-flow.md`, `reference-solution/domain-demo-spec-template.md`, GitHub issue #20, and the merged commercial-banking and retail domain-spec patterns.
- Produces: a complete healthcare-payer demo contract, a concise README entry point, and a reviewable execution checklist for later implementation work.

- [ ] **Step 1: Write the specification**

  Create `healthcare-payer/snf-prior-authorization-demo-spec.md` with the selected use case and rationale, narrative, personas, typed trigger/case contract, four reference segments, agent/tool boundaries, data/resources, two-level HITL, controls, recovery, observability, five-case evaluation set, demo script, success measures, reference mapping, separate human decisions/tasks, and quality rubric.

- [ ] **Step 2: Preserve the shared external-agent contract**

  Specify node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` with connection `0107247a-0197-42c9-b957-05d1b722b111`, a connection-selected `agent_id`, constant healthcare-payer showcase text, no `thread_id`, discarded output, no case data, a disabled-by-default flag, a short timeout/error continuation, and identical business state/routes for off, on, and failing variants.

- [ ] **Step 3: Separate implementation work from human authority decisions**

  Put tenant provisioning, artifact construction, Flow authoring, evaluation, and packaging under `Implementation tasks`. Put payer product, clinical authority, policy, SLA, privacy, system/API, deployment-folder, baseline, and process-app choices under `Open human decisions`, with an owner and resolution path for every decision.

- [ ] **Step 4: Link the specification**

  Add `SNF prior-authorization demo specification` to `healthcare-payer/README.md` beside the existing research link.

- [ ] **Step 5: Validate completeness and external-agent isolation**

  Run:

  ```bash
  rg '\[(Describe|State|Event source|System of record|Documents|Business result|Title|Responsibility|Schema|Project/model|Real business|Least privilege|Measurable|Mapping|Evidence/gap|Owner and resolution)|T[B]D|T[O]DO|im[p]lement later|fi[l]l in details' healthcare-payer/snf-prior-authorization-demo-spec.md
  rg '^## (Use case and narrative|Personas|Trigger and case contract|Flow topology|Agentic reasoning and tool use|Data and resources|Human decisions|Controls and safety|Error paths and recovery|Observability and evaluation|Demo script|Success measures|Reference mapping|Open human decisions|Implementation tasks|Quality rubric)$' healthcare-payer/snf-prior-authorization-demo-spec.md
  rg 'uipath\.connector\.uipath-microsoft-azureaifoundry\.execute-the-thread|0107247a-0197-42c9-b957-05d1b722b111|agent_id|thread_id|constant|discard|unchanged|isolation' healthcare-payer/snf-prior-authorization-demo-spec.md
  rg 'medical director|Medical director|medical-director|nurse|denial|reduction' healthcare-payer/snf-prior-authorization-demo-spec.md
  ```

  Expected: the first command returns no matches; all required sections are listed; the Azure contract has direct evidence for every isolation rule; and clinical-authority language is explicit throughout the design.

- [ ] **Step 6: Validate issue scope and repository hygiene**

  Run:

  ```bash
  git diff --check
  git diff --name-only
  git status --short
  ```

  Expected: `git diff --check` exits successfully, and only these three files are changed:

  ```text
  docs/superpowers/plans/2026-08-12-healthcare-payer-snf-prior-authorization-spec.md
  healthcare-payer/README.md
  healthcare-payer/snf-prior-authorization-demo-spec.md
  ```

- [ ] **Step 7: Review against GitHub issue #20**

  Confirm the spec selects and justifies one use case from issue #8; specifies narrative, personas, trigger/input contract, outcome/value, topology, actors/tools, integrations/mocks, data/resources, HITL, recovery, observability, evaluation, and demo script; maps every reference element; separates human decisions from implementation tasks; preserves the shared Azure contract; and scores at least 10/12 with no zero.

- [ ] **Step 8: Commit the scoped documentation**

  ```bash
  git add healthcare-payer/README.md healthcare-payer/snf-prior-authorization-demo-spec.md docs/superpowers/plans/2026-08-12-healthcare-payer-snf-prior-authorization-spec.md
  git diff --cached --name-only
  git commit -m "Specify healthcare payer SNF authorization demo"
  ```

  Expected: the staged-file list contains exactly the three issue-scoped documentation files, and the commit succeeds without JavaScript changes or an `npm test` requirement.
