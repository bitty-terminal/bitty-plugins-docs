---
title: File manager plugin schemas and contracts
description: Manifest surface version ranges capabilities commands and events for the file manager plugin
category: project
audience: plugin-author
document_type: contract
status: draft
website_publish: false
sidebar_order: 44
---

# File manager plugin schemas and contracts

## Purpose and scope

Describe observation-only policy at registry-pinned file-manager
`bf3a17ea6aa1ea0fcc3a45a13ceaa069e83b71d4`. Registry eligibility is separate
from filesystem and panel usability.

## Normative sources

The [Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md)
owns grants and manifest validation. The
[settings reference](../../../sdk/reference/settings.md) owns the current
declared-key behavior; it accepts no table-shaped `entries` kind.

## Manifest

| Field                 | Value                         |
| --------------------- | ----------------------------- |
| `[plugin].id`         | `bitty-terminal.file-manager` |
| `[plugin].name`       | File Manager                  |
| `[plugin].version`    | 0.1.0                         |
| `[compat].bitty`      | `>=0.0.24,<0.1`               |
| `[compat].plugin-api` | `^1.0`                        |

## Capabilities

`terminal.semantic-read` only. Deny by default; unknown identifiers fail
validation and there is no allow-all identifier. No `panel.*`, `fs.*`,
process, network, clipboard, or terminal-input authority is requested.

## Lazy surface

| Trigger  | Values                                                            |
| -------- | ----------------------------------------------------------------- |
| Commands | `bitty-terminal.file-manager:open`, `:preview`, `:rename`         |
| Events   | `terminal.cwd-changed`, `terminal.title-changed`, `focus.changed` |

Static triggers let the host register the plugin without creating a VM; the
observation events refresh the cached snapshot-derived state. `:rename` is
policy-only composition (rename preview and selection with no host filesystem
mutation); any host mutation stays deferred pending the `fs.write:PATTERN`
grant and the panel-provider contract (`bitty-docs` `CTX-0181`, OQ-058),
consistent with the file-manager candidate in the Plugin Matrix.

## Configuration keys

The source attempts two guarded reads through `bitty.settings.get`, but the
pinned manifest has no `[settings]`. Both are denied with
`E_SETTINGS_UNDECLARED` on the strict host:

| Key       | Type                   | Purpose                                                                   |
| --------- | ---------------------- | ------------------------------------------------------------------------- |
| `root`    | string candidate       | Undeclared; commands use explicit root or cached snapshot cwd.            |
| `entries` | table-shaped candidate | Undeclared and unaccepted; explicit command paths or an empty list serve. |

Both are inputs to the root-parameterized, fail-closed scope; neither grants
authority beyond `terminal.semantic-read`.

Declaration and shape decisions remain in
[issue #90](https://github.com/bitty-terminal/bitty-plugins/issues/90).
Do not declare `entries` as a scalar or string array merely to bypass the
gate. Neither candidate settings input enumerates the host filesystem.

## Policy bounds

Host-free Lua constants bound the composed listings: at most 128 entries per
listing, names at 128 code points, paths at 4096 bytes, selection at 64, and
the total listing payload at 8192 bytes (`8 KiB`).

## Security review

The package does not call filesystem or panel APIs. `open` shapes supplied
paths, `preview` shapes path-derived data, and `rename` returns a validated
pair without mutating a file. Pure path tests do not prove real-path,
symlink, or device isolation of a future host filesystem backend.

## Verification

The canonical manifest contract is the
[Plugin Platform RFC](../../../specifications/plugin-platform-rfc.md) (OQ-012).
The plugin manager and the host parse the manifest independently before any
plugin code runs, and `bitty-plugin-lint` from
[bitty-plugin-sdk](https://github.com/bitty-terminal/bitty-plugin-sdk) is the
authoritative validator. The owning repository holds the manifest file and its
validation evidence.
