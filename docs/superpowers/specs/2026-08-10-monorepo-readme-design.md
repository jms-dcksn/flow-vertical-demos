# Monorepo README Design

## Goal

Provide a concise repository entry point and a working index for every vertical demo lane.

## Root README

The root `README.md` will state that this repository curates independently deployable UiPath Solutions containing Maestro Flow demos. It will explain that Git is the shared source of truth while every demo solution remains its own deployment boundary.

It will link to `AGENTS.md`, the documented reference solution, and each top-level domain lane. It will include a short local-validation sequence—refresh resources, restore, then pack the affected solution—and the intended CI/CD flow: pull requests validate changed solutions; merges to `main` publish once and promote through `dev`, `staging`, and `prod` GitHub Environments.

## Domain Lanes

Each existing empty domain directory will receive a one-line `README.md` that identifies it as the research and demo-specification lane for that domain and links back to the repository README. This makes all root README links valid without inventing demo details.

## Non-goals

This issue will not create demo solutions, modify the reference-solution documentation, duplicate `AGENTS.md`, or add CI workflows.

## Validation

Confirm every README link resolves to a tracked target, check Markdown whitespace with `git diff --check`, and verify that the root README includes the required project goal, boundary, operating-instruction, domain, local-validation, and CI/CD statements.
