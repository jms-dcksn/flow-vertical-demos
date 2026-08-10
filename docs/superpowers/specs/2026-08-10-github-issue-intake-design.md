# GitHub Issue Intake Design

## Goal

Make GitHub Issues the repository's consistent intake and execution record.

## Scope

Add four GitHub issue forms under `.github/ISSUE_TEMPLATE`:

- **Research** for a domain, market, or capability investigation.
- **Demo specification** for a proposed demo design.
- **Implementation** for work on an identified domain solution or shared repository capability.
- **Decision or access request** for work that requires a human decision, approval, credential, or access grant.

Each form will require scope, acceptance criteria, dependencies, and a readiness selection. Scope will capture a domain lane or shared area, plus a solution name when known. Dependencies will use `blocked-by #<issue>` or `None` so issue ordering remains explicit.

## Contributor Guidance

`CONTRIBUTING.md` will explain the intake types, readiness meanings, triage order, labels, and dependency convention without restating the repository's full operating instructions. It will direct contributors to `AGENTS.md` for the comprehensive working agreements.

Forms will apply their stable `type:` labels automatically. The implementation form requires a new `type: implementation` label; all other labels already exist. Form authors select a readiness state in the required field, and triage applies either `ready for agent` or `ready for human` after confirming the record is complete.

## Non-goals

This change does not add automation, modify CI/CD, create demo solutions, or duplicate the README that issue #3 owns.

## Validation

Review each form for GitHub issue-form schema validity, required fields, and clear copy. Run `git diff --check`, inspect the staged diff, and verify the GitHub label exists before opening the PR.
