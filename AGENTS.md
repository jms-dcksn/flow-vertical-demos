# Flow Vertical Demos

## Project goal

Build a curated set of UiPath Maestro Flow demos that prove Flow is a modern,
developer-friendly, powerful agentic workflow runtime for enterprise use cases.

The work starts by documenting one existing representative, best-in-class Flow
solution. That reference becomes the common architecture and storytelling model
for deep, parallel domain research. Every domain folder must ultimately contain:

- a recommended enterprise use case;
- an initial demo design/spec; and
- an explicit mapping from that design to the reference solution's structure.

## Repository model

This is one GitHub monorepo. Git is the shared source of truth; UiPath Solution
boundaries remain independent deployment boundaries.

- Each demo is one UiPath Solution with exactly one `.uipx` manifest.
- Each solution can include one or more Flow projects and the resources required
  by that demo.
- Do not put unrelated demos in one solution; they must be independently
  versioned, packaged, and deployed.
- Place demos below their domain: `<domain>/<demo>/<demo>-solution/`.
- A Flow project must retain the CLI-supported nested layout:
  `<solution>/<project>/<project>.flow`.
- Use globally unique, domain-prefixed solution/package names, for example
  `commercial-banking-payment-exception`.

## Domain deliverables

The top-level domain folders are the research lanes:

- `commercial-banking`
- `consumer-banking`
- `healthcare-payer`
- `healthcare-provider`
- `life-insurance`
- `medtech`
- `p-c-insurance`
- `pharma`
- `retail`

Keep domain research, recommendations, and demo specs in the relevant domain
folder. Avoid cross-domain changes unless a reusable reference pattern is being
updated deliberately.

## UiPath Solution and CI/CD rules

- Use the `uip` CLI; do not hand-roll UiPath REST calls for solution operations.
- Before packaging, run `uip solution resources refresh` for the target
  solution, then restore/pack as appropriate.
- The deploy path is `pack` -> `publish` -> `deploy run`; `solution upload` is
  only for Studio Web editing/debugging and is not a deployment path.
- Deploy demo solutions to the UiPath Labs `Playground` tenant, under the
  `JD_Demos/demos` parent folder. Do not create environment-specific solutions.
- CI must operate per changed solution folder using GitHub Actions path filters
  or a changed-solution matrix.
- Pull requests validate affected solutions. Publishing and deployment run only
  after a push to `main`, through the `playground-deploy` GitHub Environment.
- Store `UIPATH_CLIENT_ID` and `UIPATH_CLIENT_SECRET` in the
  `playground-deploy` GitHub Environment. Keep the UiPath URL, tenant, folder,
  and other non-secret deployment configuration in the repository.
- Use an immutable, unique package version for every publish; the solution feed
  rejects duplicate name/version pairs.

## Work management

GitHub Issues are the intake and execution record for all work.

- Every issue must state its domain/solution scope, acceptance criteria,
  dependencies, and whether it is ready for an agent or a human.
- Use `blocked-by #<issue>` in the issue body for dependencies and maintain a
  `blocks` list in the parent issue. Labels communicate state; issue links
  communicate ordering.
- `ready for agent` means scope, inputs, and acceptance criteria are sufficient
  for autonomous implementation. `ready for human` means a product, technical,
  security, or external-access decision is required.
- The reference-solution documentation is the prerequisite for all domain demo
  specs. Domain research may proceed in parallel after that reference is stable.
- Keep a small number of focused issues. Use parent/child issues or a GitHub
  project only when the dependency structure adds value.

## Working agreements

- Always run `npm test` after modifying JavaScript files.
- Always use `uv` for Python package management. Never use `pip`.
- Keep README files concise.
- Do not use emojis in documentation.
- For UiPath work, open the relevant UiPath skill, inspect the actual artifact,
  installed CLI behavior, and active auth target before making claims.
- Prefer simple, demo-grade implementations unless production hardening is
  required for the demo to work.
