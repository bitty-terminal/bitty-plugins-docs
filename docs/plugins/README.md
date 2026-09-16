---
title: Plugin documentation
description: Per-plugin documentation partition with the standard plugin page set
category: project
audience: plugin-author
document_type: index
status: draft
website_publish: false
sidebar_order: 10
---

# Plugin documentation

This tree holds the per-plugin documentation for Bitty's official plugins and
featured candidates. Each documented plugin gets its own directory
`docs/plugins/<plugin>/` with the standard page set: status, design, schemas
and contracts, and evidence and links. The reusable starting point is
[`TEMPLATE.md`](TEMPLATE.md); the normative description lives in the
[documentation workflow](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/development/documentation-workflow.md#per-plugin-documentation-page-set).

## Registered official plugins

These plugins have independent repositories under `bitty-terminal` and a
published registry entry in `bitty-plugins`. The `palette` and `statusline`
page sets below are implemented in this tree; `file-manager` and `git-panel`
page sets are pending (separate task) but their packages and registry entries
already exist. Each is implemented and tested headlessly in its owning
repository; none is verified, compatible, or shipped, and no product release
contains them.

| Plugin       | Repository                                       | Plugin id                     | Stage                                                                                                                                                   | Documentation                      |
| ------------ | ------------------------------------------------ | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------- |
| activity     | <https://github.com/bitty-terminal/activity>     | `bitty-featured.activity`     | Implementation present, host integration landing                                                                                                        | [activity](activity/README.md)     |
| palette      | <https://github.com/bitty-terminal/palette>      | `bitty-terminal.palette`      | Package implemented headlessly, host bridge landing                                                                                                     | [palette](palette/README.md)       |
| statusline   | <https://github.com/bitty-terminal/statusline>   | `bitty-terminal.statusline`   | Package implemented headlessly, host bridge landing                                                                                                     | [statusline](statusline/README.md) |
| file-manager | <https://github.com/bitty-terminal/file-manager> | `bitty-terminal.file-manager` | Independent package implemented headlessly, panel presentation deferred pending the panel-provider contract; registry entry published; page set pending | Pending                            |
| git-panel    | <https://github.com/bitty-terminal/git-panel>    | `bitty-terminal.git-panel`    | Independent package implemented headlessly, panel presentation deferred pending the panel-provider contract; registry entry published; page set pending | Pending                            |

## Candidate plugins

The list below records documentation candidates from draft planning. None of
these candidates has a page set or an accepted contract; the
[Plugin Roadmap](../../product/plugin-roadmap.md) (draft) is the planning
source, and names, identifiers, and batches may change before any plugin
contract is accepted.

| Plugin        | Batch | Documentation status | Planning notes                                               |
| ------------- | ----- | -------------------- | ------------------------------------------------------------ |
| scratchpad    | First | Not started          | Ephemeral per-directory notes.                               |
| peek          | First | Not started          | Hover and preview anchored to semantic zones or rich blocks. |
| pet           | First | Not started          | Non-blocking companion overlay.                              |
| browser-panel | Later | Not started          | Browser view and panel (`bitty-terminal.browser-panel`).     |
| ai-panel      | Later | Not started          | Agent panel surface (`bitty-terminal.ai-panel`).             |
| mail-panel    | Later | Not started          | Mail triage panel (`bitty-terminal.mail-panel`).             |

## Creating a plugin directory

1. Copy [`TEMPLATE.md`](TEMPLATE.md) to `docs/plugins/<plugin>/README.md` and
   fill in the identity, stage, owning repository, and links.
2. Add `design.md`, `schemas.md`, and `evidence.md` when the plugin has real
   content for each; do not create empty placeholder pages.
3. Keep the flat frontmatter schema exact; a metadata mismatch fails
   `just metadata`.
4. Link cross-project contracts from the shared directories instead of
   restating them.

## Related

- [Project documentation partition](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/projects/README.md)
- [Bundled plugin split decision](../../product/bundled-plugin-split-decision.md) (OQ-053)
- [Plugin system](../../extensibility/plugin-system.md)
- [Plugin Roadmap](../../product/plugin-roadmap.md) (draft)
- [Default Distribution RFC](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/specifications/default-distribution-rfc.md)
