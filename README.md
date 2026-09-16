# qwts-agent-docs-gov

The `docs-gov` capability of the qwts fleet: deterministic documentation-governance checks that gate docs the way code is gated, the reusable GitHub Actions workflow that runs them in every consumer repository, and the on-demand docs-eval loop that measures whether the docs work for agents. Extracted from [qwts/agent-sop](https://github.com/qwts/agent-sop) at [ed5c5d8](https://github.com/qwts/agent-sop/commit/ed5c5d8f7aadba6eefa41a7fd17b076601530848) under the fleet split ([qwts/agent-sop#371](https://github.com/qwts/agent-sop/issues/371), [qwts/agent-sop#372](https://github.com/qwts/agent-sop/issues/372)): agent-sop keeps the rules, procedures, and guides; each capability lives in its own repository and is consumed at a pinned commit SHA.

## Adopting the gate

A consumer adds one job and pins it to a commit:

```yaml
jobs:
  docs-gov:
    uses: qwts/qwts-agent-docs-gov/.github/workflows/docs-governance.yml@<40-hex sha>
    with:
      tooling-ref: <40-hex sha>
```

`<40-hex sha>` is the pin recorded as the `docs-gov` capability in [qwts/qwts-agent-org](https://github.com/qwts/qwts-agent-org)'s `org.json`, and it is written twice on purpose: a called workflow cannot read its own commit from the `github` context (that context is the caller's; the reusable workflow's commit exists only as the `job_workflow_sha` OIDC claim), so `tooling-ref` names the commit the checks are fetched from, and the job fails closed when it is empty rather than drifting to the default branch. Optional inputs: `config` (path to the consumer's config, default `docs-gov.config.json`), `runs-on` (JSON array of runner labels, default `["ubuntu-latest"]`), and `use-local-tooling` (this repository's own self-test). The job uploads `docs-gov-report.json` as an artifact.

The consumer writes a `docs-gov.config.json` at its root naming what to scan and how strictly; this repository's own [config](docs-gov.config.json) is a minimal example, and [documentation governance](docs/documentation-governance.md) explains every rule and setting. The same check, locally:

```bash
node tools/docs-gov/docs-gov.mjs                       # this repository
node tools/docs-gov/docs-gov.mjs --root ../other-repo  # any checkout that has its own config
```

## Docs

- [Documentation governance](docs/documentation-governance.md) — the `docs-gov` rule catalog, the conventions the gate enforces, running it locally, and consuming the reusable workflow.
- [Documentation evaluation](docs/docs-evaluation.md) — the `docs-eval` benchmark, runner, and held-out validation gate; on demand, never a merge gate.

## Layout

- `tools/docs-gov/` — the zero-dependency CLI (`tools/docs-gov/docs-gov.mjs`), its check library, tests, and fixtures.
- `tools/docs-eval/` — the benchmark (`tools/docs-eval/benchmark.json`), runner, comparator, tests, and committed results.
- `.github/workflows/docs-governance.yml` — the reusable workflow consumers pin.
- `.github/workflows/ci.yml` — this repository's CI: tests, the gate against its own docs, markdownlint.
- `docs-gov.config.json` — this repository's own docs-gov config.

## Developing

```bash
npm ci                  # markdownlint-cli only; the tools themselves have no dependencies
npm test                # node --test over tools/docs-gov and tools/docs-eval
npm run docs:gov        # the gate, against this repository's docs
npm run lint:markdown   # markdownlint over docs/, AGENTS.md, README.md
```

Runtime dependencies stay at zero (ENG-0004): a consumer runs the CLI from a bare checkout with no install, so a change to the workflow's inputs or the CLI's flags is a contract change for every pinned consumer.

## Governing decisions

The decisions stay in agent-sop and are not restated here:

- [ENG-0009](https://github.com/qwts/agent-sop/blob/main/docs/decisions/ENG-0009-documentation-governance-gate.md) — documentation is gated like code: deterministic checks first, evaluation later.
- [ENG-0010](https://github.com/qwts/agent-sop/blob/main/docs/decisions/ENG-0010-docs-evaluation-loop.md) — the docs evaluation loop, adopted with tight bounds.

Agent context for this repository: [AGENTS.md](AGENTS.md).
