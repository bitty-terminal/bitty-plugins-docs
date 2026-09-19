# bitty-plugins-docs

`bitty-plugins-docs` is the canonical documentation repository for the Bitty
plugin ecosystem. It owns the English-language documentation for the plugin
SDK, plugin manifests and contracts, the plugin lifecycle, isolation and
capability rules, and per-plugin design notes.

**Current state: plugin-ecosystem corpus migrated from `bitty-docs` at
`c664214` (CTX-0001, parent bitty-docs CTX-0187 Phase 3).** The repository
carries the accepted and draft plugin contracts in theme topic trees
(`runtime/`, `sdk/`, `packaging/`, `architecture/`, `specifications/`,
`product/`, `extensibility/`) and the per-plugin page set under `docs/plugins/`.
Migrated documents remain design-stage unless they state their own evidence;
this README describes the repository contract, not shipped behavior.

## Scope

This repository owns:

- Plugin SDK documentation: public API surface, versioning, and compatibility.
- Plugin manifests, contracts, and schema references.
- Plugin lifecycle: installation, activation, update, disable, and removal.
- Isolation and capability model: what plugins may access and how limits are
  enforced.
- Per-plugin design notes for first-party and featured plugin candidates.
- Reference material derived from verified implementation evidence.

This repository does not own:

- Shared cross-project governance — decisions, the security corpus, findings,
  reviews, handoff, roadmap, releases, and project state — which stays in
  [bitty-docs](https://github.com/bitty-terminal/bitty-docs).
- Terminal-platform documentation, which lives in
  [bitty-terminal-docs](https://github.com/bitty-terminal/bitty-terminal-docs).
- AI-core documentation, which lives in
  [bitty-ai-docs](https://github.com/bitty-terminal/bitty-ai-docs).

Cross-project contracts and registers are linked, never copied.

## Composition

This repository is mounted at `bitty-plugins/docs` as a Git submodule
(branch `main`) of the
[bitty-plugins](https://github.com/bitty-terminal/bitty-plugins) composition
repository, which holds the plugin registry and the official plugins as pinned
submodules (`plugins/activity`, `plugins/palette`, `plugins/statusline`) plus
the `sdk`
([bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk)) and
`template`
([bitty-plugin-template](https://github.com/bitty-terminal/bitty-plugin-template))
submodules. The standalone documentation repository remains fully
self-contained and passes its own gates. The plugin-ecosystem corpus migrated
from `bitty-docs` at `c664214` with history; residual terminal and AI material
stays in the sibling documentation repositories.

## Structure

| Path                             | Purpose                                                            |
| -------------------------------- | ------------------------------------------------------------------ |
| `docs/README.md`                 | Documentation map and authority rules for this repository.         |
| `docs/development/`              | Contributor workflow and the normative documentation policy.       |
| `docs/plugins/`                  | Per-plugin index, template, and standard page sets.                |
| `runtime/`                       | Plugin host runtime, Lua runtime, and isolation and resource.      |
| `sdk/`                           | Public plugin SDK Lua surface contract.                            |
| `packaging/`                     | Package lifecycle, resolver, registry, and provider ecology.       |
| `architecture/`                  | Plugin ecosystem model, IPC boundary, UI extensibility, diagrams.  |
| `architecture/diagrams/`         | Glossary-driven diagram suite, Mermaid sources, vector exports.    |
| `specifications/`                | Platform contract and candidate-direction register index.          |
| `product/`                       | Plugin roadmap and product planning documents.                     |
| `extensibility/`                 | Pre-implementation plugin system and package-management contracts. |
| `TODO.md`                        | Work register for this repository.                                 |
| `AGENTS.md`                      | Agent scope, CarryCtx workflow, and local gate rules.              |
| `.github/scripts/check-docs.mjs` | Links, metadata, language, budgets, and hygiene checks.            |
| `justfile`                       | Pinned docs-quality commands; `just check` is the gate.            |

## Authority and status

- The
  [documentation workflow](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/docs/development/documentation-workflow.md)
  is normative for authoring, metadata, status, and review.
- Every canonical document carries the flat frontmatter schema and declares its
  own status; `just metadata` enforces the schema for `docs/**` and the root
  topic trees. Design intention must never read as implemented behavior.
- "Candidate" and "planned" are prose, not implementation claims. A plugin page
  must not imply shipped behavior it cannot support.
- When statements conflict, the canonical bitty-docs security corpus takes
  precedence. Implementation claims require evidence from the owning code
  repository.

## Local checks

```sh
just fmt           # format supported files
just check         # full local gate pipeline (same logical gates as CI)
```

`just check` verifies Prettier formatting, markdownlint, repository-local links,
frontmatter metadata, English-only content, file budgets, hygiene, SVG
well-formedness, and GitHub Actions syntax. JavaScript tooling runs through Bun
only; `npm`, `npx`, and `yarn` are not used here.

## Contributing

Read [CONTRIBUTING.md](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/CONTRIBUTING.md)
and [AGENTS.md](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/AGENTS.md)
before editing. The normal lifecycle is Issue, scoped CarryCtx task,
branch/worktree, commit, pull request, independent review plus CI, merge, then
task closure and a final checkpoint.

## Related repositories

| Repository                                                                       | Role                                           |
| -------------------------------------------------------------------------------- | ---------------------------------------------- |
| [bitty](https://github.com/bitty-terminal/bitty)                                 | Terminal platform implementation.              |
| [bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk)           | Plugin SDK.                                    |
| [bitty-plugin-template](https://github.com/bitty-terminal/bitty-plugin-template) | Plugin template.                               |
| [bitty-docs](https://github.com/bitty-terminal/bitty-docs)                       | Shared cross-project governance and registers. |
| [bitty-terminal-docs](https://github.com/bitty-terminal/bitty-terminal-docs)     | Terminal-platform documentation.               |
| [bitty-ai-docs](https://github.com/bitty-terminal/bitty-ai-docs)                 | AI-core documentation.                         |

## License

MIT — see [LICENSE](https://github.com/bitty-terminal/bitty-plugins-docs/blob/main/LICENSE).
