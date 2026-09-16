# Contributing to bitty-plugins-docs

This guide is for contributors to the `bitty-plugins-docs` repository. The
repository is documentation-first and pre-implementation: documents are the
contract source, and nothing here may be described as implemented product
behavior.

## Repository ground rules

- Read [AGENTS.md](AGENTS.md) before making any change. It defines scope,
  authority, the CarryCtx workflow, the documentation contract, and workspace
  hygiene that override convenience.
- Canonical plugin-ecosystem contracts live in this repository; shared
  cross-project governance lives in `bitty-docs` and is linked, never copied.
  Do not duplicate normative specifications across repositories.
- Every document under `docs/` uses the exact flat frontmatter schema defined
  in `docs/development/documentation-workflow.md`; its `title` must match the
  H1. English is the only canonical documentation language.
- Label statements as normative, accepted, proposed, experimental, implemented,
  or unverified; never turn a design intention into a shipped-behavior claim.
- Never commit, push, publish, or mutate remote state without explicit
  authorization from the owning task.

## Prerequisites

Toolchain expectations (tool versions are pinned in exactly one place, the
[justfile](justfile); never invoke formatters or linters by name):

- `just` — command runner owning all quality-gate invocations.
- `bun` / `bunx --bun` — JavaScript execution; the justfile runs pinned tools
  through `bunx --bun` and repository scripts through `bun`. Never use `npm`,
  `npx`, or `yarn` in any Bitty repository.
- `actionlint` 1.7.12 — GitHub Actions workflow validation, checked against the
  justfile pin.
- `xmllint` (libxml2-utils) — SVG well-formedness validation.
- `act` 0.2.x — optional local workflow dry-run (`act -n`).

## Development setup

1. Enter this repository before running Git, CarryCtx, or toolchain commands.
2. Confirm the gates run from a clone: `just check`.
3. Enable Git hooks (optional): `just hooks-install`.
4. Run all quality gates: `just check` (Prettier format check, markdownlint,
   repository-local links, frontmatter metadata, English-only content, agent
   file budgets, hygiene, SVG well-formedness, and Actions syntax). CI runs the
   same logical gates, and individual gates are available as `just fmt-check`,
   `just markdownlint`, `just links`, `just metadata`, `just language`,
   `just agents`, `just hygiene`, and `just svg`.
5. Record scoped work in CarryCtx (task, session, progress, checkpoint) and
   stop at review; independent review is required for acceptance.

## Delivery lifecycle

Changes follow Issue -> Branch -> Commit -> Pull Request -> Review -> Merge,
where independent review plus required CI must pass before merge. Every pull
request states its Issue and CarryCtx task links, impact areas, security and
privacy impact, reproducible gate evidence, and documentation synchronization
status. Labels (`feat`/`fix`/`docs`/`chore`, `P0`/`P1`/`P2`, `area:*`) and
milestone `v0.1.0` are kept in sync. Commits are Conventional Commits validated
against [commitlint.config.ts](commitlint.config.ts).

## Contributor branches

Branches are managed with CarryCtx. Official branches use
`ctx-XXXX/<type>-<slug>`, where `XXXX` is the owning CarryCtx task number,
`<type>` is one of `feat|fix|chore|docs`, and the slug is short kebab-case;
commander housekeeping branches may use `cmd/<slug>`. External contributors
must use a distinguishable prefix, for example `<github-handle>/<type>-<slug>`.
Worktrees live at `.worktrees/ctx-XXXX-<type>-<short-slug>`, mapping `/` to
`-`.

## Capabilities and privacy

Plugin packages, manifests, project files, and update channels are untrusted
until a narrow capability or policy grants access. Capabilities are deny by
default; plugins may alter presentation, never terminal truth, and receive no
ambient Lua or OS authority. Installation executes no package code. Never add a
temporary bypass API, broaden a capability implicitly through documentation, or
describe an unreviewed grant as automatic approval.

## Workflow snapshots

The engineering workflow snapshot lives in this repository on the branch
`refs/heads/carryctx-snapshots`. Merges run `just workflow-publish` (dry run:
`just workflow-publish-dry`) as part of the commander closeout; snapshots are
redacted publication artifacts and are never merged back. Fresh clones restore
with `just workflow-import` (`just workflow-import-dry`).

## Reporting

Report bugs and feature requests through the GitHub issue templates. Report
security issues privately per [SECURITY.md](SECURITY.md); never open a public
issue for a vulnerability.
