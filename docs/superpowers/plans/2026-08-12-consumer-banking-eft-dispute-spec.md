# Consumer Banking EFT Dispute Spec Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Complete GitHub issue #17 with an implementation-ready specification for the recommended consumer-banking Maestro Flow demo.

**Architecture:** Promote the top-ranked EFT dispute recommendation into one template-aligned domain specification. Keep the research as the evidence source and define explicit Flow, actor, input/output, human-review, safety, recovery, evaluation, external-agent isolation, solution-boundary, and delivery contracts in the new spec.

**Tech Stack:** Markdown, repository reference pattern, UiPath CLI 1.199.0 tenant/resource inspection, GitHub issue acceptance criteria.

## Global Constraints

- Scope changes to issue #17 and exactly three consumer-banking documentation files.
- Preserve `consumer-banking/opportunity-research.md` as the evidence and recommendation source.
- Design one independently deployable UiPath Solution with exactly one `.uipx` manifest and the CLI-supported nested Flow layout.
- Use synthetic dispute, transaction, authentication, document, and account data; no live financial or customer action is permitted.
- Keep deterministic deadline and policy logic outside agents and require an authorised human outcome before any mock credit, denial, or notice action.
- Bind Azure AI Foundry connection `0107247a-0197-42c9-b957-05d1b722b111` only on a non-material showcase branch with a connection-selected `agent_id`, static input, no `thread_id`, discarded output, and fail-open continuation.
- Keep open human decisions separate from implementation tasks, including the repository `JD/demos` versus tenant `JD_Demos/demos` folder mismatch.
- Keep the README concise and use no emojis in documentation.

---

### Task 1: Author and validate the EFT dispute resolution demo specification

**Files:**
- Create: `consumer-banking/eft-dispute-resolution-demo-spec.md`
- Modify: `consumer-banking/README.md`
- Create: `docs/superpowers/plans/2026-08-12-consumer-banking-eft-dispute-spec.md`

**Interfaces:**
- Consumes: `consumer-banking/opportunity-research.md`, `reference-solution/claims-process-flow.md`, `reference-solution/domain-demo-spec-template.md`, GitHub issue #17, and the merged commercial-banking and retail specification patterns.
- Produces: a complete consumer-banking demo contract, a concise README entry point, and an executable documentation plan for review and handoff.

- [ ] **Step 1: Confirm the selected use case and evidence boundary**

  Read the ranked candidates and select `EFT dispute investigation and resolution`, preserving the 14/15 comparative score and the research caveats: public sources establish relevance but do not establish bank volume, production process, policy, ROI, or implementation readiness.

- [ ] **Step 2: Record current CLI and tenant evidence without executing a Flow**

  Run:

  ```bash
  uip --version
  uip login status --output json
  uip user --output json
  uip maestro flow registry search "foundry" --output json --output-filter "[*].{NodeType:NodeType,DisplayName:DisplayName,AvailableOnTenant:AvailableOnTenant}"
  uip is connections list uipath-microsoft-azureaifoundry --all-folders --connection-id 0107247a-0197-42c9-b957-05d1b722b111 --output json --output-filter "[*].{Id:Id,Name:Name,ConnectorKey:ConnectorKey,State:State,IsDefault:IsDefault,Folder:Folder,FolderKey:FolderKey}"
  uip or folders list --all --name demos --limit 50 --output json --output-filter "[*].{Key:Key,Name:Name,Path:Path,Type:Type,ParentKey:ParentKey}"
  ```

  Expected: CLI `1.199.0`; logged-in `uipathlabs/Playground` user; exact Azure node tenant-available; shared connection enabled/default in folder `demos`; and folder listing showing path `JD_Demos/demos`. Treat every other tenant resource as unverified and use a mock or fixture fallback.

- [ ] **Step 3: Write the specification**

  Create `consumer-banking/eft-dispute-resolution-demo-spec.md` with the selection rationale, narrative, personas, typed input/output contracts, four reference segments, complete actor inventory, agent/tool boundaries, data/resources, canonical lifecycle, HITL contracts, controls, recovery, observability, five-case evaluation set, demo script, success measures, complete reference mapping, separate human decisions, implementation tasks, and quality rubric.

- [ ] **Step 4: Preserve the external-agent showcase contract**

  Specify node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` and connection `0107247a-0197-42c9-b957-05d1b722b111` with a connection-selected `agent_id`, constant consumer-banking showcase message, no `thread_id`, no case or sensitive data, discarded response, disabled-by-default flag, short timeout/error continuation, and identical core data/routes for off, on, and failing variants.

- [ ] **Step 5: Link the specification from the domain README**

  Add `EFT dispute resolution demo specification` as a bullet beside the existing opportunity-research link in `consumer-banking/README.md`.

- [ ] **Step 6: Validate completeness, isolation, issue coverage, and scope**

  Run:

  ```bash
  rg '\[(Describe|State|Event source|System of record|Documents|Business result|Title|Responsibility|Schema|Project/model|Real business|Least privilege|Measurable|Mapping|Evidence/gap|Owner and resolution)|TBD|TODO' consumer-banking/eft-dispute-resolution-demo-spec.md
  rg 'TBD|TODO|implement later|fill in details|add appropriate error handling|write tests for the above|similar to Task' docs/superpowers/plans/2026-08-12-consumer-banking-eft-dispute-spec.md | rg -v '^  rg '
  rg '^## ' consumer-banking/eft-dispute-resolution-demo-spec.md
  rg '0107247a-0197-42c9-b957-05d1b722b111|connection-selected|static|constant|no `thread_id`|discarded|unchanged|identical|isolation' consumer-banking/eft-dispute-resolution-demo-spec.md
  rg '## Open human decisions|## Implementation tasks|## Demo script|## Error paths and recovery|## Observability and evaluation|## Reference mapping' consumer-banking/eft-dispute-resolution-demo-spec.md
  git diff --check
  git diff --name-only
  ```

  Expected: the placeholder scan returns no matches; every required section and external-agent isolation term is present; whitespace is clean; and only the three issue-scoped files changed. Review every issue #17 acceptance criterion against a named section and confirm the quality score is at least 10/12 with no zero.

- [ ] **Step 7: Commit the scoped documentation**

  ```bash
  git add consumer-banking/README.md consumer-banking/eft-dispute-resolution-demo-spec.md docs/superpowers/plans/2026-08-12-consumer-banking-eft-dispute-spec.md
  git commit -m "Specify consumer banking EFT dispute demo"
  ```
