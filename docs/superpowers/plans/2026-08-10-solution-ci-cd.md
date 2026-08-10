# UiPath Solution CI/CD Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans (recommended) or superpowers:subagent-driven-development to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add reusable GitHub Actions workflows that validate changed UiPath Solutions on pull requests and can safely deploy explicitly configured solutions after merge to `main`.

**Architecture:** A dispatcher discovers changed `.uipx` roots and produces validation and deployment matrices. Reusable workflows isolate validation from deployment: validation runs only dry-run lifecycle commands; deployment is guarded by the `playground-deploy` Environment and deploy-enable variable, and uses an explicit solution registry so the fixture can never be published.

**Tech Stack:** GitHub Actions YAML, Bash, `jq`, UiPath CLI, Ruby YAML parser.

## Global Constraints

- A pull request must never access UiPath credentials, publish a package, or deploy.
- Run `uip solution resources refresh` before restore and packing.
- Publish and deploy the same immutable package version only after a push to `main`.
- `playground-deploy` holds `UIPATH_CLIENT_ID`, `UIPATH_CLIENT_SECRET`, and the `UIPATH_DEPLOY_ENABLED` variable; the credentials are never committed or logged.
- Real solutions opt into deployment through `.github/uipath-deployments.json`; the fixture remains validation-only.
- Retain packages and CLI logs, including on failure.

---

### Task 1: Add a fixture and deploy registry

**Files:**
- Create: `test-fixtures/solution-ci-fixture/solution-ci-fixture.uipx`
- Create: `test-fixtures/solution-ci-fixture/AGENTS.md`
- Create: `test-fixtures/solution-ci-fixture/CLAUDE.md`
- Create: `.github/uipath-deployments.json`

**Interfaces:**
- Produces: an empty but valid solution root accepted by `uip solution resources refresh`, `restore`, and `pack --dry-run`.
- Produces: a JSON registry with `solutions` entries carrying `solution_path`, `package_name`, `deployment_name`, `folder_name`, and `deployment_config`.

- [ ] **Step 1: Generate the fixture with the supported CLI**

Run: `uip solution init test-fixtures/solution-ci-fixture --output json`

- [ ] **Step 2: Verify the generated fixture and start with an empty deployment registry**

Create `.github/uipath-deployments.json`:

```json
{ "solutions": [] }
```

Run: `uip solution resources refresh --solution-folder test-fixtures/solution-ci-fixture --output json && uip solution restore test-fixtures/solution-ci-fixture --output json && uip solution pack test-fixtures/solution-ci-fixture /tmp/solution-ci-fixture --dry-run --output json`

Expected: every command returns `Result: Success`; no package is created by the dry run.

- [ ] **Step 3: Commit the fixture and registry**

```bash
git add test-fixtures/solution-ci-fixture .github/uipath-deployments.json
git commit -m "test: add UiPath solution CI fixture"
```

### Task 2: Add reusable validation and deployment workflows

**Files:**
- Create: `.github/workflows/validate-uipath-solution.yml`
- Create: `.github/workflows/deploy-uipath-solution.yml`

**Interfaces:**
- Consumes: `solution_path` for validation.
- Consumes: `solution_path`, `package_name`, `package_version`, `deployment_name`, `folder_name`, and `deployment_config` for deployment.
- Produces: validation command logs; package archives and deployment logs.

- [ ] **Step 1: Create the reusable validation workflow**

Use `on.workflow_call` with required `solution_path`. Checkout the caller revision, install `@uipath/cli`, and run the following commands with `tee` log files:

```bash
uip solution resources refresh --solution-folder "${{ inputs.solution_path }}" --output json
uip solution restore "${{ inputs.solution_path }}" --output json
uip solution pack "${{ inputs.solution_path }}" "$RUNNER_TEMP/package" --dry-run --output json
```

Upload `validation-logs` with `if: always()` and 14-day retention. Do not configure an environment, authentication, publish, or deploy step.

- [ ] **Step 2: Create the guarded reusable deployment workflow**

Use `on.workflow_call` with all deployment inputs. Its sole job uses `environment: playground-deploy` and has a job-level condition `vars.UIPATH_DEPLOY_ENABLED == 'true'`. Login without exposing values:

```bash
uip login --client-id env.UIPATH_CLIENT_ID --client-secret env.UIPATH_CLIENT_SECRET --tenant Playground --output json
```

Run refresh, restore, and pack into `package/` with the supplied name and version. Publish the exact resulting zip, run `uip solution deploy run` with `JD/demos`, supplied deployment and folder names, and the supplied config. Extract `PipelineDeploymentId` from the run JSON and call `uip solution deploy status` with it. Upload the exact zip and all logs with 30-day retention, including on failure.

- [ ] **Step 3: Verify reusable workflow syntax**

Run: `ruby -e 'require "yaml"; ARGV.each { |path| YAML.safe_load_file(path, aliases: true); puts "valid: #{path}" }' .github/workflows/validate-uipath-solution.yml .github/workflows/deploy-uipath-solution.yml`

Expected: both workflow files report `valid`.

- [ ] **Step 4: Commit reusable workflows**

```bash
git add .github/workflows/validate-uipath-solution.yml .github/workflows/deploy-uipath-solution.yml
git commit -m "ci: add reusable UiPath solution workflows"
```

### Task 3: Add dispatcher, checks, and activation documentation

**Files:**
- Create: `.github/workflows/uipath-solution-ci.yml`
- Modify: `.github/PLAYGROUND_DEPLOYMENT.md`
- Modify: `README.md`

**Interfaces:**
- Consumes: event before/after SHAs and `.github/uipath-deployments.json`.
- Produces: `validation_matrix` containing every changed solution and `deployment_matrix` containing only changed registry entries.
- Invokes: `validate-uipath-solution.yml` for every pull request matrix entry and `deploy-uipath-solution.yml` after a `main` push.

- [ ] **Step 1: Create the dispatcher workflow**

Trigger only on `pull_request` and `push` to `main`, with path filters covering workflow files, the deploy registry, and source below all demo and fixture solution folders. Give the workflow `contents: read` permission. A discovery job checks out full history, compares the event range, searches for every `.uipx`, and emits both matrices through `$GITHUB_OUTPUT` with `jq`. A solution is changed when any changed file begins with its root path. Validation receives all changed roots. Deployment receives only changed roots with a matching registry entry; an empty registry emits `{"include":[]}`.

Use the reusable validation workflow for pull requests only. Use the reusable deployment workflow for pushes to `main` only. Both jobs should skip cleanly for an empty matrix.

- [ ] **Step 2: Document enablement and artifact behavior**

Update `.github/PLAYGROUND_DEPLOYMENT.md` to name `UIPATH_DEPLOY_ENABLED=true` as the final activation control, explain the explicit registry, stable deployment-folder names, and 30-day package/log retention. Update the README delivery section to link to that guide and clarify that a repository has no live deployment until #34 is complete and a solution is registry-listed.

- [ ] **Step 3: Run static checks**

Run:

```bash
ruby -e 'require "yaml"; ARGV.each { |path| YAML.safe_load_file(path, aliases: true); puts "valid: #{path}" }' .github/workflows/*.yml
rg -n 'pull_request:|resources refresh|restore|pack|publish|deploy run|deploy status|playground-deploy|UIPATH_DEPLOY_ENABLED|retention-days' .github/workflows .github/PLAYGROUND_DEPLOYMENT.md README.md
git diff --check main...HEAD
```

Expected: every workflow parses, each required contract string is present in the appropriate workflow, and the diff check has no output.

- [ ] **Step 4: Commit dispatcher and documentation**

```bash
git add .github/workflows/uipath-solution-ci.yml .github/PLAYGROUND_DEPLOYMENT.md README.md
git commit -m "ci: dispatch changed UiPath solutions"
```

### Task 4: Final verification and pull request

**Files:**
- Verify: all files from Tasks 1–3

**Interfaces:**
- Produces: a branch that closes GitHub issue #4.

- [ ] **Step 1: Re-run fixture validation and static checks**

Run the Task 1 fixture lifecycle commands and the Task 3 static checks. Confirm `git status --short` is empty.

- [ ] **Step 2: Publish the branch and open a draft pull request**

Push `codex/issue-4-solution-cicd`, then open a draft pull request with `Closes #4`. Summarize that live deployment awaits #34 and has not been attempted.
