# Contributing

Use GitHub Issues as the intake and execution record for all work. Read
[AGENTS.md](AGENTS.md) for the complete repository working agreements.

## Choose an intake form

- **Research** — investigate a domain, market, or capability before selecting a demo.
- **Demo specification** — define a focused demo design and its reference-solution mapping.
- **Implementation** — build or change a defined demo solution or shared capability.
- **Decision or access request** — obtain a product, technical, security, or access decision from a human owner.

Every form requires a domain or shared-area scope, solution scope, acceptance
criteria, dependencies, and readiness state. Record each dependency as
`blocked-by #<issue>` or write `None`.

## Triage

Triage confirms the required fields are complete, then applies one `type:`
label, the relevant domain or area label when applicable, a `priority:` label,
and exactly one readiness label:

- `ready for agent` — scope, inputs, and acceptance criteria support autonomous work.
- `ready for human` — a product, technical, security, or external-access decision is needed.

Use `blocked` when unresolved work cannot begin. Keep `blocked-by #<issue>` in
the blocked issue body and maintain a `blocks` list in its parent issue. Labels
communicate state; issue links communicate ordering.
