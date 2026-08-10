# GitHub Issue Intake Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add unambiguous GitHub Issue intake forms and concise contributor triage guidance for issue #2.

**Architecture:** Four YAML issue forms capture the required structured intake data and apply stable type labels. A root-level contributor guide defines the triage contract, label taxonomy, readiness states, and dependency notation without duplicating `AGENTS.md`.

**Tech Stack:** GitHub Issue Forms (YAML), GitHub labels, Markdown, Ruby/Psych YAML parser, GitHub CLI.

## Global Constraints

- Keep guidance concise and refer to `AGENTS.md` instead of duplicating it.
- Require scope, acceptance criteria, dependencies, and readiness in every form.
- Use `blocked-by #<issue>` for dependencies and `None` when there are none.
- Use the existing domain labels exactly; create only `type: implementation`.
- Do not change the README; issue #3 owns repository entry-point documentation.

---

## File Structure

- Create: `.github/ISSUE_TEMPLATE/research.yml` — structured research intake.
- Create: `.github/ISSUE_TEMPLATE/demo-specification.yml` — structured demo-design intake.
- Create: `.github/ISSUE_TEMPLATE/implementation.yml` — structured implementation intake.
- Create: `.github/ISSUE_TEMPLATE/decision-access-request.yml` — structured decision or access intake.
- Create: `.github/ISSUE_TEMPLATE/config.yml` — disables blank issues and directs contributors to the four forms.
- Create: `CONTRIBUTING.md` — concise filing and triage conventions.

### Task 1: Add the implementation type label

**Files:**
- Modify: GitHub repository label configuration for `jms-dcksn/flow-vertical-demos`.

**Interfaces:**
- Produces: GitHub label `type: implementation` with description `Demo or repository implementation work`.
- Consumed by: `.github/ISSUE_TEMPLATE/implementation.yml` as its automatic label.

- [ ] **Step 1: Confirm the label is absent**

Run: `gh label list --repo jms-dcksn/flow-vertical-demos --limit 100 | rg '^type: implementation\\b'`

Expected: no output.

- [ ] **Step 2: Create the label**

Run: `gh label create 'type: implementation' --color 1D76DB --description 'Demo or repository implementation work' --repo jms-dcksn/flow-vertical-demos`

- [ ] **Step 3: Verify label metadata**

Run: `gh label list --repo jms-dcksn/flow-vertical-demos --limit 100 | rg '^type: implementation\\s+Demo or repository implementation work\\s+#1D76DB$'`

Expected: one matching line.

### Task 2: Add issue-form configuration and intake forms

**Files:**
- Create: `.github/ISSUE_TEMPLATE/config.yml`
- Create: `.github/ISSUE_TEMPLATE/research.yml`
- Create: `.github/ISSUE_TEMPLATE/demo-specification.yml`
- Create: `.github/ISSUE_TEMPLATE/implementation.yml`
- Create: `.github/ISSUE_TEMPLATE/decision-access-request.yml`

**Interfaces:**
- Consumes: existing `domain:` labels and `type: implementation` from Task 1.
- Produces: four selectable GitHub Issue Forms with required scope, acceptance criteria, dependencies, and readiness fields.
- Consumed by: repository contributors and the triage guidance in Task 3.

- [ ] **Step 1: Create `config.yml`**

Set `blank_issues_enabled: false` and add a contact link to `AGENTS.md` for repository working agreements.

- [ ] **Step 2: Create four forms with focused prompts**

For every form, include required controls for:

```yaml
- type: dropdown
  id: scope
  attributes:
    label: Domain or shared area
    options:
      - commercial-banking
      - consumer-banking
      - healthcare-payer
      - healthcare-provider
      - life-insurance
      - medtech
      - p-c-insurance
      - pharma
      - retail
      - reference-solution
      - repository-wide
  validations:
    required: true
```

Also include required textarea fields headed `Solution scope`, `Acceptance criteria`, and `Dependencies`; the dependency prompt must require either `blocked-by #<issue>` entries or `None`. Include a required readiness dropdown with `ready for agent` and `ready for human` as its two options. Apply each form's matching `type:` label; apply `type: implementation` to implementation.

- [ ] **Step 3: Validate all YAML files parse**

Run:

```bash
ruby -e 'require "yaml"; Dir[".github/ISSUE_TEMPLATE/*.yml"].each { |f| YAML.load_file(f); puts "valid: #{f}" }'
```

Expected: five `valid:` lines and exit status 0.

- [ ] **Step 4: Verify the required form contract**

Run:

```bash
for f in .github/ISSUE_TEMPLATE/{research,demo-specification,implementation,decision-access-request}.yml; do
  printf '%s\n' "$f"
  rg -n 'id: (scope|solution_scope|acceptance_criteria|dependencies|readiness)|required: true' "$f"
done
```

Expected: each form reports all five IDs and their required validations.

### Task 3: Add concise contributor triage guidance

**Files:**
- Create: `CONTRIBUTING.md`

**Interfaces:**
- Consumes: the four form names from Task 2 and existing GitHub labels.
- Produces: a concise contract for filing, triaging, labeling, and ordering issues.

- [ ] **Step 1: Document how to select each intake form**

Describe the intended use of Research, Demo specification, Implementation, and Decision or access request in one line each.

- [ ] **Step 2: Document the triage contract**

Require scope, acceptance criteria, dependencies, and readiness before triage. State that triage applies one `type:` label, one domain or area label where applicable, a priority label, and exactly one readiness label. Define `ready for agent` and `ready for human` using the repository's existing label descriptions.

- [ ] **Step 3: Document dependency notation and escalation**

Require `blocked-by #<issue>` in the issue body and a `blocks` list in parent issues. Direct contributors to mark unresolved work `blocked`, and point readers to `AGENTS.md` for the full working agreements.

- [ ] **Step 4: Check scope and Markdown rendering hygiene**

Run: `git diff --check && rg -n 'AGENTS.md|blocked-by #<issue>|ready for agent|ready for human|type:' CONTRIBUTING.md`

Expected: no whitespace errors and all required concepts reported.

### Task 4: Verify and publish the issue #2 change

**Files:**
- Verify: all files listed above and `docs/superpowers/specs/2026-08-10-github-issue-intake-design.md`.

**Interfaces:**
- Consumes: completed Tasks 1–3.
- Produces: a focused pull request that closes issue #2.

- [ ] **Step 1: Review the complete diff and working-tree scope**

Run: `git status -sb && git diff --check && git diff --stat main...HEAD && git diff --stat && git diff main...HEAD && git diff`

Expected: the committed design artifact plus only the issue #2 forms and contributor guide in the working tree, with no whitespace errors.

- [ ] **Step 2: Run final YAML and content validation**

Run:

```bash
ruby -e 'require "yaml"; Dir[".github/ISSUE_TEMPLATE/*.yml"].each { |f| YAML.load_file(f) }'
for f in .github/ISSUE_TEMPLATE/{research,demo-specification,implementation,decision-access-request}.yml; do
  rg -q 'id: scope' "$f" && rg -q 'id: solution_scope' "$f" && rg -q 'id: acceptance_criteria' "$f" && rg -q 'id: dependencies' "$f" && rg -q 'id: readiness' "$f"
done
```

Expected: exit status 0.

- [ ] **Step 3: Commit and push the branch**

Run:

```bash
git add .github CONTRIBUTING.md docs/superpowers
git diff --cached --check
git diff --cached
git commit -m 'docs: add issue intake forms and triage guidance'
git push -u origin codex/issue-2-intake-templates
```

- [ ] **Step 4: Open the pull request**

Create a draft pull request titled `docs: add issue intake forms and triage guidance` against `main` with a body that lists the forms, triage guide, implementation label, and validation commands, ending with `Closes #2`.
