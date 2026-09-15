# bitty-plugins-docs agent guide

## Scope and authority

- This file governs only the independent `bitty-plugins-docs` Git repository,
  not the workspace umbrella directory or other Bitty repositories.
- All formal Bitty repositories belong under <https://github.com/bitty-terminal>.
- This repository owns canonical documentation for the Bitty plugin ecosystem:
  SDK, manifests, lifecycle, isolation and capabilities, and per-plugin design
  notes. Shared cross-project governance stays in [bitty-docs](https://github.com/bitty-terminal/bitty-docs) and is
  linked, never copied. Verify other repository boundaries from the owning repository.

## Current phase

- Documentation and project foundations come before product implementation.
- The plugin-ecosystem corpus was migrated from `bitty-docs` at `c664214`
  (CTX-0001, parent bitty-docs CTX-0187); this repository is published as
  [bitty-plugins-docs](https://github.com/bitty-terminal/bitty-plugins-docs) and mounted at `docs`
  in [bitty-plugins](https://github.com/bitty-terminal/bitty-plugins) as a Git submodule (branch `main`).
- Never describe a planned, proposed, or unverified feature as implemented.
  The migrated corpus is design-stage unless its own evidence says otherwise.

## Read before acting

1. Read this file and the task's relevant files under `.carryctx/rules/`.
2. Read the relevant contract documents and the repository state.
3. For source analysis, read narrowly: outline first, then read only the needed
   ranges; use `rg` to locate symbols and imports, and keep command output trimmed.

## CarryCtx workflow

- CarryCtx is the durable project record; the external harness runs agents.
- Start/resume a named session, inspect task/team context, then claim and start
  only the assigned task; record progress, risks, blockers, decisions, and
  checkpoints as work proceeds.
- Every parallel task declares a non-overlapping scope. Prefer one Git worktree
  per implementation task; shared-checkout work requires explicit disjoint
  scope.
- Branches use `ctx-XXXX/<type>-<short-slug>` (`XXXX` is the owning task number;
  `<type>` is `feat|fix|chore|docs`; the slug is kebab-case) with worktrees at
  `.worktrees/ctx-XXXX-<type>-<short-slug>`; one branch per task, while
  commander housekeeping may use `cmd/<slug>`.
- Subagents perform scoped implementation and stop at `in_review`; the
  commander plans, dispatches, reads CarryCtx state back, reviews diffs, and an
  independent reviewer accepts completion.
- Do not commit, merge, publish, or change external state unless explicitly
  asked.
- Fresh clones have no CarryCtx state DB. Once snapshot publication exists, the
  redacted snapshot on `refs/heads/carryctx-snapshots` restores with
  `carryctx import --from-git refs/remotes/origin/carryctx-snapshots`
  (`--dry-run` first). Never merge snapshots back; rotate any leaked secret at
  the source.

The lifecycle is GitHub Issue, CarryCtx task, branch/worktree, commit, pull
request, independent review plus CI, merge, then Issue closure and a final
checkpoint. Before the first commit, shared-checkout work with disjoint scopes
is the only allowed exception; do not invent a branch, commit, or PR that
cannot yet exist.

### Issue hygiene (labels and milestones)

- Every GitHub Issue and PR must have appropriate `labels` and the owning
  `milestone` when one applies. Write issues with a clear title, description,
  acceptance criteria, and links to the CarryCtx task and any RFC/OQ.
- Use `gh issue create --label "docs,area:docs,P1" --milestone "v0.1.0"` and
  keep labels in sync with `gh issue edit` / `gh pr edit`.
- Every CarryCtx task carries a `Priority: P0/P1/P2 | Area: ... | Labels: ... |
Milestone: ... | RFC: ... | Task: CTX-XXXX` header; a missing header is
  `NEEDS-FIX`.

| Label group | Values                                                                                                 |
| ----------- | ------------------------------------------------------------------------------------------------------ |
| Type        | `docs`, `feat`, `fix`, `chore`                                                                         |
| Priority    | `P0` (highest), `P1` (high), `P2` (medium)                                                             |
| Area        | `area:docs` (default), `area:plugin`, `area:lua`, `area:package`, `area:security`, `area:architecture` |
| Maintenance | `dependencies`                                                                                         |

### Local gates before push (mandatory)

- Before pushing any branch, run `just check` locally with 0 issues and
  validate `.github/workflows/ci.yml` with `act -n`; `act` only checks syntax,
  so `just check` must still pass. Never push with known local failures.
- CodeQL and cargo gates are intentionally not configured for this
  documentation-only repository; local `just check` and the Docs quality
  workflow are the merge gates.

### Remote monitoring and merge

- This repository is docs-only: local `just check` is the gate. After push,
  `gh pr checks` is informational; merge when locally green and
  `mergeable==MERGEABLE` with `gh pr merge --squash`. When waiting is desired,
  prefer `HTTPS_PROXY=$NETWORK_PROXY gh pr checks <PR> --watch --interval 15`
  via `pty_spawn` over `sleep` loops.
- Every completed task receives independent review of its diff before
  acceptance; record defects via CarryCtx and convert them to follow-up tasks.

## Documentation contract

- English is the only canonical documentation language. Do not add CJK content,
  translations, locale directories, or multilingual routes; i18n is deferred.
- Every `docs/**/*.md` file uses the exact flat frontmatter schema in
  `docs/development/documentation-workflow.md`; its `title` matches the H1.
- Separate normative requirements, accepted decisions, proposals, and current
  implementation status; cross-link one authoritative definition instead of
  copying divergent wording, and record unresolved questions rather than
  silently choosing across a public contract boundary.
- Per-plugin pages use the standard page set (README, design, schemas,
  evidence) and create only pages with real content.
- Update affected architecture, security, specification, and reference
  documents together when their shared contract changes; synchronization is
  part of the definition of done.

## Security baseline

- Plugin packages, manifests, project files, and update channels are untrusted
  until a narrow capability or policy grants access.
- P0 trust boundaries are release blockers; do not add temporary bypass APIs.
- Plugins may alter presentation, never terminal truth; they do not enter the
  input, parser, or render hot paths and receive no ambient Lua or OS
  authority.
- Installation executes no package code; preserve checksum validation, bounded
  parsing, transactional activation, and a no-third-party-plugin safe startup
  path. Capability grants block updates pending review; document grant and
  budget semantics rather than implying automatic approval.

## Workspace hygiene

- The umbrella workspace root is not a Git repository; run Git and CarryCtx in
  the named repository.
- Durable scratch data lives under `recording/`; use `/tmp/bitty/` only for
  ephemeral data. Reference clones are untrusted, read-only evidence; do not run cloned
  scripts, hooks, binaries, or installers without explicit need and review.
- Avoid `rm` and `rmdir`.
  Preserve unrelated and untracked changes in a shared checkout.

## Verification and handoff

- Check the diff stays inside task scope and contains no generated or temporary
  artifacts.
- Run repository-specific formatting, link, schema, language, and hygiene gates
  in proportion to the change; record exact evidence in CarryCtx.
- Report changed files, verification, unresolved risks, and remaining work. A
  partial or pre-feature check is not evidence that the broader project is
  done.
