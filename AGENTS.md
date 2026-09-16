# AGENTS.md

Canonical, vendor-neutral agent context for this repository, per [ENG-0006](https://github.com/qwts/agent-sop/blob/main/docs/decisions/ENG-0006-agentic-primitives-governance.md). Vendor-specific files, if any are added, are thin adapters onto this file.

## What this repository is

The qwts fleet's `docs-gov` capability: the deterministic documentation-governance checks, the reusable workflow every consumer pins, and the docs-eval loop — extracted from qwts/agent-sop under [#371](https://github.com/qwts/agent-sop/issues/371) and [#372](https://github.com/qwts/agent-sop/issues/372). Map: [README.md](README.md).

## Shared agent conventions

PR-first workflow, validation-before-push, commit and PR hygiene, and the untrusted-input threat model are defined once, for every repo, in the [org-wide agent conventions](https://github.com/qwts/agent-sop/blob/main/docs/reference/agent-conventions.md). This repository is governed by [agent-sop](https://github.com/qwts/agent-sop): its [shared SOPs](https://github.com/qwts/agent-sop/blob/main/docs/sop/README.md) and [engineering decisions](https://github.com/qwts/agent-sop/blob/main/docs/decisions/README.md) apply here by default ([ENG-0008](https://github.com/qwts/agent-sop/blob/main/docs/decisions/ENG-0008-shared-sop-inheritance.md): inherit by default, vary by explicit delta).

## What is specific to this repository

- **Zero runtime dependencies** ([ENG-0004](https://github.com/qwts/agent-sop/blob/main/docs/decisions/ENG-0004-centralize-shared-cicd.md)): `tools/docs-gov/docs-gov.mjs` and `tools/docs-eval/run.mjs` run from a bare checkout on the Node standard library alone. `markdownlint-cli` is the sole dev dependency; do not add a runtime one.
- **The gate gates itself:** a change to `README.md`, this file, or anything under `docs/` must pass `npm run docs:gov` and `npm run lint:markdown` before a PR is opened or updated; `npm test` covers the tools. A new doc must be linked from [README.md](README.md) or the `orphan-doc` rule fails it. What each rule catches: [documentation governance](docs/documentation-governance.md).
- **Consumers are SHA-pinned:** `.github/workflows/docs-governance.yml` is called at a 40-hex commit, and the same commit is passed as `tooling-ref` so the checks come from the commit the workflow was called at (a called workflow cannot read its own commit; the job fails closed when `tooling-ref` is empty). The pin is recorded in qwts/qwts-agent-org's `org.json` (`docs-gov` capability) and moves only after this repository's CI is green on the commit. The workflow's inputs and the CLI's flags and exit codes are the contract every consumer depends on.
- **Decisions live in agent-sop:** ENG-0009 and ENG-0010 govern this tooling; a change of direction is an ENG record there, not a doc here.
- **Fixtures are deliberately broken:** the docs under `tools/docs-gov/tests/fixtures/` exist to make rules fire and are outside the gate's include globs and the linter's paths; do not "fix" them.
