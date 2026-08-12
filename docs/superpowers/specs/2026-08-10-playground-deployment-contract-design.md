# Playground deployment contract

## Purpose

Define the minimal authority required for GitHub Actions to deploy demo
solutions to UiPath. This design supersedes the original multi-environment
scope in issue #5: these demos have one deployment target.

## Deployment target

| Setting | Value |
| --- | --- |
| UiPath organization | UiPath Labs (`https://cloud.uipath.com`) |
| Tenant | `Playground` |
| Orchestrator folder | `JD_Demos/demos` |
| GitHub Environment | `playground-deploy` |
| Deployment trigger | Push to `main` only |

Pull-request workflows may validate a solution, but must not authenticate to
UiPath, publish a package, or deploy it.

## Authentication and ownership

GitHub Actions signs in with a dedicated OAuth client, not a person's account.
The UiPath Labs tenant administrator creates or identifies that client and
grants it only the permissions required to publish and deploy solutions in the
`JD_Demos/demos` folder. The repository administrator owns the GitHub Environment and
its secrets.

The `playground-deploy` environment contains these GitHub Actions secrets:

- `UIPATH_CLIENT_ID`
- `UIPATH_CLIENT_SECRET`

The client secret is never committed, printed in logs, or included in a pull
request. The client ID is also stored as an environment secret to keep the
workflow interface simple.

## Repository contract

The repository may contain non-secret deployment configuration: the UiPath URL,
tenant name, folder path, solution path, package name, version, and deployment
configuration. It must not contain OAuth tokens, OAuth client secrets, or any
other credential values.

Issue #4 owns the reusable GitHub Actions workflow. That workflow will select
the `playground-deploy` environment only for its push-to-`main` publish/deploy
job, read the two secrets above, sign in to `Playground`, and deploy changed
solutions to `JD_Demos/demos`.

## Failure handling

If the environment or either secret is absent, the deployment job fails before
attempting a UiPath operation. The job output must identify the missing
configuration by name without revealing secret values.

## Acceptance checks

1. The target URL, tenant, folder, and GitHub Environment are explicit.
2. A dedicated OAuth client is required and is scoped to `JD_Demos/demos`.
3. The required GitHub secrets and their role owners are explicit.
4. The repository's secret/non-secret boundary is explicit.
5. Issue #4 has an unambiguous deployment contract to implement.
