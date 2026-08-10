# Monorepo README Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create the concise repository README and working index of the nine vertical demo lanes required by issue #3.

**Architecture:** A root README provides the entry point, boundaries, instructions, index, validation, and delivery flow. One concise README in each pre-existing empty domain directory establishes a tracked link target without fabricating domain designs.

**Tech Stack:** Markdown, Git link-target checks.

## Global Constraints

- Keep every README concise.
- Do not duplicate `AGENTS.md` or modify the reference-solution documentation.
- Do not create demo solutions or CI workflows.
- Link every listed domain lane to a tracked repository file.
- Describe the prescribed `uip` resource-refresh, restore, and pack validation sequence accurately.

---

## File Structure

- Create: `README.md` — repository entry point and demo-lane index.
- Create: `<domain>/README.md` in each of `commercial-banking`, `consumer-banking`, `healthcare-payer`, `healthcare-provider`, `life-insurance`, `medtech`, `p-c-insurance`, `pharma`, and `retail` — concise tracked lane targets.

### Task 1: Create the root repository README

**Files:**
- Create: `README.md`

**Interfaces:**
- Consumes: `AGENTS.md`, `reference-solution/README.md`, and the nine lane README paths from Task 2.
- Produces: the repository entry point used by contributors and GitHub visitors.

- [ ] **Step 1: State the project goal and deployment boundary**

Describe the repository as a curated collection of enterprise UiPath Maestro Flow demos. State that Git is the shared source of truth and each demo remains an independently versioned, packaged, and deployed UiPath Solution.

- [ ] **Step 2: Link the repository guidance and reference pattern**

Add direct relative links to `AGENTS.md` and `reference-solution/README.md`.

- [ ] **Step 3: Add the domain-lane index**

Create a compact bullet list linking to all nine `<domain>/README.md` files.

- [ ] **Step 4: Add local validation and CI/CD summaries**

Document this local sequence for an affected solution:

```bash
uip solution resources refresh --solution-folder <solution-folder>
uip solution restore <solution-folder>
uip solution pack <solution-folder> --dry-run
```

State that pull requests validate changed solutions, while merged `main` changes publish once and promote the same package through `dev`, `staging`, and `prod` GitHub Environments.

### Task 2: Create the domain-lane README targets

**Files:**
- Create: `commercial-banking/README.md`
- Create: `consumer-banking/README.md`
- Create: `healthcare-payer/README.md`
- Create: `healthcare-provider/README.md`
- Create: `life-insurance/README.md`
- Create: `medtech/README.md`
- Create: `p-c-insurance/README.md`
- Create: `pharma/README.md`
- Create: `retail/README.md`

**Interfaces:**
- Consumes: the lane names in `AGENTS.md`.
- Produces: valid target paths for the root README index and a minimal description of each lane's purpose.

- [ ] **Step 1: Add a title and purpose to every lane README**

Use the human-readable domain name as the heading. State that the folder contains research, recommendations, and demo specifications for that domain.

- [ ] **Step 2: Link each lane back to the root README**

Include `[Repository overview](../README.md)` in every lane README.

### Task 3: Validate and publish the issue #3 change

**Files:**
- Verify: `README.md`, all nine lane READMEs, and `docs/superpowers/specs/2026-08-10-monorepo-readme-design.md`.

**Interfaces:**
- Consumes: Tasks 1–2.
- Produces: a focused pull request that closes issue #3.

- [ ] **Step 1: Validate Markdown links resolve to tracked paths**

Run:

```bash
for path in AGENTS.md reference-solution/README.md commercial-banking/README.md consumer-banking/README.md healthcare-payer/README.md healthcare-provider/README.md life-insurance/README.md medtech/README.md p-c-insurance/README.md pharma/README.md retail/README.md; do
  test -f "$path"
done
for path in commercial-banking consumer-banking healthcare-payer healthcare-provider life-insurance medtech p-c-insurance pharma retail; do
  rg -q '\[Repository overview\]\(\.\./README\.md\)' "$path/README.md"
done
```

Expected: exit status 0.

- [ ] **Step 2: Verify the root README contract**

Run:

```bash
rg -n 'Maestro Flow|independently|AGENTS.md|reference-solution|resources refresh|solution restore|solution pack|pull requests|dev|staging|prod' README.md
git diff --check
```

Expected: all concepts appear and no whitespace errors are reported.

- [ ] **Step 3: Review, commit, and push**

Run:

```bash
git add README.md commercial-banking consumer-banking healthcare-payer healthcare-provider life-insurance medtech p-c-insurance pharma retail docs/superpowers
git diff --cached --check
git diff --cached
git commit -m 'docs: add monorepo readme and demo index'
git push -u origin codex/issue-3-monorepo-readme
```

- [ ] **Step 4: Open the pull request**

Create a draft pull request titled `docs: add monorepo readme and demo index` against `main`. Its body must summarize the repository entry point and domain lanes, report the Markdown checks, and end with `Closes #3`.
