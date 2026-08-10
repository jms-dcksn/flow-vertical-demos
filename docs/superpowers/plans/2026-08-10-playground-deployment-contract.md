# Playground Deployment Contract Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Establish one GitHub Environment and a concise repository contract for automated deployment of demo solutions to UiPath Playground.

**Architecture:** A GitHub Environment named `playground-deploy` isolates the OAuth credentials outside the repository. Repository instructions and a short setup guide fix the non-secret target (`https://cloud.uipath.com`, `Playground`, `JD/demos`) and tell the future reusable workflow in issue #4 when it may use that environment.

**Tech Stack:** GitHub Environments, GitHub Actions secrets, UiPath CLI contract, Markdown.

## Global Constraints

- Deploy only after a push to `main`; pull requests never authenticate to UiPath or deploy.
- Use one `playground-deploy` GitHub Environment with no deployment approval gate.
- The target is `https://cloud.uipath.com`, tenant `Playground`, folder `JD/demos`.
- Store `UIPATH_CLIENT_ID` and `UIPATH_CLIENT_SECRET` only as GitHub Environment secrets.
- The repository may contain only non-secret target and package configuration.

---

### Task 1: Create and verify the GitHub Environment

**Files:**
- Modify: remote GitHub repository setting for `jms-dcksn/flow-vertical-demos`

**Interfaces:**
- Consumes: authenticated `gh` session with permission to administer repository environments.
- Produces: GitHub Environment `playground-deploy`, with no required reviewers or wait timer.

- [ ] **Step 1: Verify GitHub CLI authentication**

Run: `gh auth status`

Expected: an authenticated account with access to `jms-dcksn/flow-vertical-demos`.

- [ ] **Step 2: Create the environment idempotently**

Run: `gh api --method PUT repos/jms-dcksn/flow-vertical-demos/environments/playground-deploy`

Expected: JSON describing the `playground-deploy` environment.

- [ ] **Step 3: Verify the environment configuration**

Run: `gh api repos/jms-dcksn/flow-vertical-demos/environments/playground-deploy`

Expected: the response name is `playground-deploy`; protection rules do not require approvals or a wait timer.

- [ ] **Step 4: Record the outstanding secret setup**

Do not create fake secrets. The repository administrator adds `UIPATH_CLIENT_ID` and `UIPATH_CLIENT_SECRET` in the `playground-deploy` environment after the UiPath Labs tenant administrator supplies the dedicated OAuth client credentials.

### Task 2: Align repository CI/CD instructions with the single-target model

**Files:**
- Modify: `AGENTS.md`
- Create: `.github/PLAYGROUND_DEPLOYMENT.md`

**Interfaces:**
- Consumes: GitHub Environment `playground-deploy` from Task 1 and the target contract in `docs/superpowers/specs/2026-08-10-playground-deployment-contract-design.md`.
- Produces: one source of truth for human setup and unambiguous instructions for issue #4's reusable workflow.

- [ ] **Step 1: Update the CI/CD rules in `AGENTS.md`**

Replace the multi-environment promotion wording with the single `Playground` target, the `JD/demos` parent folder, and `playground-deploy` Environment. Keep the required `resources refresh`, `pack`, `publish`, and `deploy run` lifecycle. State that only pushes to `main` can publish or deploy.

- [ ] **Step 2: Add `.github/PLAYGROUND_DEPLOYMENT.md`**

Document the exact target, the two secret names, their role owners, the safe/non-secret boundary, and a short GitHub UI setup procedure. State that future workflow jobs use `environment: playground-deploy` only in a deploy job triggered by a push to `main`.

- [ ] **Step 3: Verify the documentation contract**

Run: `rg -n 'playground-deploy|Playground|JD/demos|UIPATH_CLIENT_ID|UIPATH_CLIENT_SECRET|push to `main`' AGENTS.md .github/PLAYGROUND_DEPLOYMENT.md`

Expected: all required target and secret strings appear in the repository instructions and setup guide.

- [ ] **Step 4: Verify repository text formatting**

Run: `git diff --check`

Expected: exit code 0 and no whitespace errors.

### Task 3: Prepare the pull request

**Files:**
- Modify: `AGENTS.md`
- Create: `.github/PLAYGROUND_DEPLOYMENT.md`
- Create: `docs/superpowers/specs/2026-08-10-playground-deployment-contract-design.md`
- Create: `docs/superpowers/plans/2026-08-10-playground-deployment-contract.md`

**Interfaces:**
- Consumes: Tasks 1 and 2 verification output.
- Produces: a draft pull request targeting `main` with `Closes #5` in its body.

- [ ] **Step 1: Inspect the final branch diff**

Run: `git diff origin/main...HEAD --check && git diff --stat origin/main...HEAD`

Expected: only the safety ignore rule, deployment contract documentation, plan, and GitHub setup guide/instructions are present.

- [ ] **Step 2: Commit the implementation changes**

Run: `git add AGENTS.md .github/PLAYGROUND_DEPLOYMENT.md docs/superpowers/plans/2026-08-10-playground-deployment-contract.md && git commit -m "docs: define Playground deployment authority"`

Expected: a focused commit on `agent/issue-5-deployment-contract`.

- [ ] **Step 3: Push and open a draft pull request**

Run: `git push -u origin agent/issue-5-deployment-contract`

Expected: the remote branch is created. Open a draft PR to `main` with `Closes #5` and state that the two OAuth secrets must be added before the future deployment workflow can run.
