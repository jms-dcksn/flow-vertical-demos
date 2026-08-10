# UiPath Solution CI/CD Design

## Scope

Issue #4 adds reusable GitHub Actions workflows for this monorepo. A pull
request validates only the UiPath Solutions it changes. A merge to `main`
can package, publish, deploy, and verify those same solutions, but only after
the `playground-deploy` GitHub Environment has been provisioned under #34.

## Architecture

The repository workflow discovers changed `*.uipx` manifests and produces a
matrix of solution roots. It invokes a reusable validation workflow on pull
requests and a separate reusable deployment workflow only for pushes to
`main`. The deployment workflow creates one immutable package version per
solution from `github.run_number`, retains the package and command logs, and
uses the `playground-deploy` Environment exclusively in its deployment job.

The validation workflow never authenticates to UiPath, reads environment
secrets, publishes, or deploys. It runs `uip solution resources refresh`,
`restore`, and `pack --dry-run` for every changed solution. The deployment
workflow runs the same preparation steps, packs once, publishes that exact
archive, deploys it below `JD/demos`, and queries deployment status using the
returned pipeline deployment id.

## Fixture and inputs

A minimal, committed fixture solution exercises discovery and dry-run packing.
The reusable workflows accept `solution_path`, `package_name`,
`package_version`, and `deployment_config`; callers own matrix construction.
Solutions are excluded from live deployment unless their path is explicitly
listed in the deployment matrix, so the fixture can never publish.

## Guardrails and failure behavior

The dispatcher uses path filters for workflow execution and compares the
event range to find changed manifests. It exits successfully with no matrix
when an eligible solution is not changed. Pull requests have read-only
permissions and call only the validation workflow. Deployment requires a push
to `main`, `playground-deploy`, an explicit environment enable variable, and
the two OAuth secrets. Command output is written to per-solution log files and
both package and logs are uploaded on success or failure.

Live deployment remains blocked by #34. Each real solution must supply a
stable package name, deploy configuration, and folder-name convention before
being added to the deployment matrix. `uip solution deploy run` creates its
solution folder beneath `JD/demos`; it is not an in-place update of an
arbitrary existing folder.

## Verification

Validate every workflow as YAML, verify expected trigger and guard strings,
and run the exact validation sequence against the fixture. Live publish and
deploy are intentionally not run from this branch because the required
environment credentials and human approval are external prerequisites.
