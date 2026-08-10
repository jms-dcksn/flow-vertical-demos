# Playground deployment setup

Demo solutions deploy to UiPath Labs at `https://cloud.uipath.com`, tenant
`Playground`, under the `JD/demos` parent folder.

## One-time setup

1. A UiPath Labs tenant administrator creates or identifies a dedicated OAuth
   client and grants it only the permissions needed to publish and deploy
   solutions in `JD/demos`.
2. A repository administrator opens the `playground-deploy` GitHub Environment
   and adds these environment secrets:
   - `UIPATH_CLIENT_ID`
   - `UIPATH_CLIENT_SECRET`

Do not commit either value or share the client secret in a pull request or log.

## Workflow contract

The reusable workflow from issue #4 uses `playground-deploy` only in its
publish/deploy job. That job runs after a push to `main`, logs in to the
`Playground` tenant, and deploys changed solutions to `JD/demos`.

Pull-request jobs validate only. They must not access this environment,
authenticate to UiPath, publish a package, or deploy a solution.

The repository may contain the UiPath URL, tenant name, folder path, solution
paths, package names, versions, and deploy configuration. OAuth credentials and
other secret values belong only in GitHub Environment secrets.

## Enabling deployment

Live deployment remains disabled until #34 is complete. Set the
`UIPATH_DEPLOY_ENABLED` variable to `true` in `playground-deploy` only after
the client credentials and environment protections are verified. Add each real
solution to `.github/uipath-deployments.json` with a stable package,
deployment-folder name, and deploy-config path. The workflow retains the
published package and command logs for 30 days.
