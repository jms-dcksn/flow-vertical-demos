# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Authoritative working agreements

[AGENTS.md](AGENTS.md) holds the repository operating model: solution boundaries, naming, CI/CD rules, issue intake, and working agreements. Read it and treat it as the source of truth — do not restate or fork its rules here. This file covers the architecture and commands that require reading several files to reconstruct.

## What this repository currently is

A documentation-first monorepo of UiPath Maestro Flow demo designs across nine industry domains. As of now it contains **no implemented demo solutions** — only markdown research/specs plus one CI fixture (`test-fixtures/solution-ci-fixture/`). There is no `package.json`, no JavaScript, and no Python, so AGENTS.md's `npm test` and `uv` agreements only bind once such code is introduced.

Implementation is the next phase: turning the nine specs into real `.uipx` solutions.

## The content pipeline

Work in a domain flows through four artifact types, in order. Each stage cites the previous one as its evidence source.

1. **Reference baseline** — `reference-solution/claims-process-flow.md`. Derived from an inspected Studio Web export of the `ClaimsProcessFlow` FNOL solution (not vendored here; it holds tenant-bound resources). It separates *evidence found in that export* from the *stricter target standard* new demos must meet. Every demo's architecture and story derive from this.
2. **Domain research** — `<domain>/agentic-workflow-opportunities.md` (or `opportunity-research.md` / `research.md`; the name varies by lane). Ranks candidate use cases. Preserved as the evidence source; specs do not replace it.
3. **Demo spec** — `<domain>/<name>-demo-spec.md`, produced by copying `reference-solution/domain-demo-spec-template.md` and completing every bracketed field. The template ends in a 0–3 **quality rubric** across enterprise credibility, Flow differentiation, demo clarity, and build feasibility; a spec is implementation-ready only at ≥10/12 with no zero and an owner for every gap.
4. **Superpowers plan/spec** — `docs/superpowers/plans/YYYY-MM-DD-<slug>.md` and `docs/superpowers/specs/`. Checkbox task plans meant to be executed with the `superpowers:subagent-driven-development` or `superpowers:executing-plans` skills. Each plan states goal, architecture, global constraints, and per-task file/interface lists. Written work is recorded here, so check for an existing plan before designing one.

The nine domain lanes are `commercial-banking`, `consumer-banking`, `healthcare-payer`, `healthcare-provider`, `life-insurance`, `medtech`, `p-c-insurance`, `pharma`, `retail`. Keep changes inside one lane unless deliberately updating a shared reference pattern.

## Cross-cutting design contracts every spec inherits

These come from `reference-solution/claims-process-flow.md` and recur in all nine specs — preserve them when implementing:

- **Four-segment topology**: receive and understand → assess and enrich → decide and review → act and communicate. Happy path left-to-right, exceptions below, 3–4 named sticky notes, independent parallel branches merged before dependent work.
- **Actor contrast is the point**: at least one inline low-code agent with a wired tool, one coded agent with a narrow visible value-add, plus API workflow and RPA on the *intended path* (never as unconnected projects).
- **Shared external-agent showcase**: node `uipath.connector.uipath-microsoft-azureaifoundry.execute-the-thread` bound to connection `0107247a-0197-42c9-b957-05d1b722b111` (verified enabled in Playground `demos`, 2026-08-12). It sits on a labelled non-material branch: static non-sensitive input only, response discarded, no `thread_id`, and timeout or failure rejoins the unchanged core route. Re-validate the binding during implementation rather than trusting the recorded date.
- **Real routing, real review**: at least one decision on a genuine business value (confidence, risk, eligibility, human outcome) with both paths labelled in business language, and a human task whose returned outcome data is consumed downstream.
- **Evaluation**: one 3–5 point evaluation set per demo covering straight-through, review, exception, and a domain edge case, plus a Flow- or node-level evaluator.

## Solution layout and local validation

Planned layout is `<domain>/<demo>/<demo>-solution/`, one `.uipx` per solution, with the CLI-supported nested Flow layout `<solution>/<project>/<project>.flow`. Package and solution names are globally unique and domain-prefixed (e.g. `commercial-banking-payment-exception`).

Drive all solution operations through the `uip` CLI; never hand-edit a `.uipx`. `test-fixtures/solution-ci-fixture/CLAUDE.md` is a full `uip solution` reference (scaffolded snapshot — verify commands against `uip <group> --help` and correct it in place if stale).

```bash
uip solution resources refresh --solution-folder <solution-folder>   # sync bindings_v2.json into the inventory
uip solution restore <solution-folder>                               # resolve dependencies
uip solution pack <solution-folder> --dry-run                        # strict validation, no package produced
uip maestro flow validate <path-to.flow> --output json               # what CI runs per Flow
```

The deploy path is `pack` → `publish` → `deploy run`. `uip solution upload` targets Studio Web for editing/debugging only and is not a deployment path.

## CI/CD mechanics

`.github/workflows/uipath-solution-ci.yml` is the entry point; it calls two reusable workflows.

- **discover** locates every `*.uipx` via `find`, and treats a solution as changed when any path under its directory appears in `git diff --name-only BASE HEAD`. So a solution is picked up purely by having a `.uipx` at its root — no registration needed for validation.
- **validate** (pull requests only) runs credential-free: `jq empty` on the `.uipx` plus `uip maestro flow validate` on every `.flow`. It must never authenticate, publish, or deploy.
- **deploy** (push to `main` only) is gated three ways: the solution must be listed in `.github/uipath-deployments.json` (currently `{"solutions": []}` — so nothing deploys yet), the `playground-deploy` environment variable `UIPATH_DEPLOY_ENABLED` must be `true`, and `UIPATH_CLIENT_ID`/`UIPATH_CLIENT_SECRET` live only as environment secrets. Each registry entry supplies `solution_path`, `package_name`, `deployment_name`, `folder_name`, and `deployment_config`.
- Versions are `1.0.${{ github.run_number }}` and deployment names get the run number appended, satisfying the feed's immutable name/version requirement. Target is `https://cloud.uipath.com`, tenant `Playground`, parent folder `JD_Demos/demos`. See `.github/PLAYGROUND_DEPLOYMENT.md`.

Non-secret deployment configuration (URL, tenant, folder, paths, package names, versions) belongs in the repository; only OAuth credentials go in the GitHub Environment.

## Working style in this repo

- Issues are the intake and execution record; `.github/ISSUE_TEMPLATE/` defines four forms (research, demo specification, implementation, decision/access request), each requiring scope, acceptance criteria, dependencies as `blocked-by #<issue>`, and exactly one readiness label. See [CONTRIBUTING.md](CONTRIBUTING.md).
- Branches follow `<tool>/issue-<n>-<slug>` (`agent/…` for Claude Code, `codex/…` for Codex) and are developed in git worktrees under `.worktrees/` (gitignored) — expect many sibling checkouts of the whole tree there and exclude them from searches.
- Domain READMEs are deliberately short link hubs. Keep them that way, and use no emojis in documentation.
- For any UiPath claim, open the relevant UiPath skill and inspect the actual artifact, installed CLI behavior, and active auth target rather than asserting from memory.
