# Flow Vertical Demos

This repository curates enterprise UiPath Maestro Flow demos. Git is the shared
source of truth; each demo is an independently versioned, packaged, and
deployed UiPath Solution.

Start with the [operating instructions](AGENTS.md) and the
[reference solution](reference-solution/README.md), which provides the common
architecture and storytelling model.

## Domain lanes

- [Commercial banking](commercial-banking/README.md)
- [Consumer banking](consumer-banking/README.md)
- [Healthcare payer](healthcare-payer/README.md)
- [Healthcare provider](healthcare-provider/README.md)
- [Life insurance](life-insurance/README.md)
- [Medtech](medtech/README.md)
- [P&C insurance](p-c-insurance/README.md)
- [Pharma](pharma/README.md)
- [Retail](retail/README.md)

## Local validation

For an affected solution, refresh resources before packaging. Restore resolves
dependencies early; `pack --dry-run` validates without creating a package.

```bash
uip solution resources refresh --solution-folder <solution-folder>
uip solution restore <solution-folder>
uip solution pack <solution-folder> --dry-run
```

## Delivery

Pull requests validate changed Flow and solution manifests without UiPath
credentials. After merge to `main`, the `playground-deploy` Environment can
refresh, package, publish, deploy, and verify explicitly registered solutions
to UiPath Labs Playground. See [the deployment guide](.github/PLAYGROUND_DEPLOYMENT.md).
