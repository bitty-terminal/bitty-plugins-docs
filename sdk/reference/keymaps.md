---
title: bitty.keymaps Reference
description: Live keymap reads and key-binding suggestion capture with bounded paging and typed denials
category: reference
audience: plugin-author
document_type: reference
status: draft
website_publish: true
sidebar_order: 90
---

# bitty.keymaps Reference

> Status: **draft**. The accepted [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md) defines the `suggest` spelling and its precedence model. The live `list` read is documented here from Core `bitty@7d98f53a` and the `plugin_keymaps` runtime tests. This page states implementation evidence, not a compatibility promise for a future API revision.

## Purpose and scope

`bitty.keymaps` has two different surfaces:

- `bitty.keymaps.list(options)` is a bounded, read-only view of the live
  resolved keymap registry. It reads user and shipped-default bindings through
  the `runtime.keymaps.read` capability.
- `bitty.keymaps.suggest(def)` is the separate registration-time surface for
  capturing a plugin's proposed binding. A suggestion is not a row in
  `list`, and capture is not publication or application of a binding.

The list read contains registry data only. It never exposes configuration
paths, raw configuration text, arguments, environment values, URLs, terminal
content, clipboard content, plugin-store values, or host diagnostics. Rows are
read from the live source at call time; a reload or republished table is
visible on the next call rather than through a cached page.

## Signature

```lua
bitty.keymaps.list(options?) -> page
bitty.keymaps.suggest(def) -> handle
```

`list` accepts no argument, `nil`, or an options table. `suggest` accepts the
definition described in [Suggestion capture](#suggestion-capture). The two
functions have separate capability and lifecycle behavior; do not use one as
evidence that the other is available.

## Params

### `list` options

The only recognized options are integer `offset` and integer `limit`.
Unknown fields are ignored. The options table is validated by the Lua bridge
before the host source is called.

| Option   | Default | Accepted range | Meaning                                         |
| -------- | ------- | -------------- | ----------------------------------------------- |
| `offset` | `0`     | `0..=1024`     | Zero-based start in resolved-table order.       |
| `limit`  | `256`   | `1..=256`      | Maximum number of rows requested for this page. |

`offset` and `limit` are paging bounds, not filters. The API has no path,
command-name, URL, pattern, or free-form string option.

### Suggestion capture

The accepted RFC defines:

```lua
def = {
  chord = string,
  command = string,
  when? = string,
}
```

`command` names a command registered by the same plugin generation. In v1,
`when` is absent or `"global"`; the shipped configuration chord grammar and
the `(when, normalized chord)` identity apply when suggestions are evaluated.
The current bridge validates the shape and captures the definition during
`init.lua` activation. Host-side grammar validation and binding application
are separate stages.

## Returns

### `list`

`page` is a table with these fields:

| Field       | Type    | Meaning                                                                                              |
| ----------- | ------- | ---------------------------------------------------------------------------------------------------- |
| `entries`   | array   | Canonical rows in resolved-table order.                                                              |
| `total`     | integer | Number of addressable rows, capped at `1024`; this is not the number returned in `entries`.          |
| `offset`    | integer | The validated offset echoed by the response.                                                         |
| `truncated` | boolean | Whether an unreturned tail exists because of the page, addressable-row, or serialized-payload bound. |

An offset at or past `total` returns an empty `entries` array and
`truncated = false`. A page is also bounded to a 64 KiB canonical serialized
payload; whole rows are dropped from the tail to stay within that bound.

Each entry has the following canonical fields:

| Field     | Presence | Meaning                                                                                            |
| --------- | -------- | -------------------------------------------------------------------------------------------------- |
| `chord`   | required | Canonical chord spelling, such as `alt+h` or `ctrl+shift+v`.                                       |
| `action`  | required | Canonical Core action spelling, or `command:owner:command` for a plugin command binding.           |
| `source`  | required | `"default"` for a shipped default or `"user"` for a user override.                                 |
| `context` | required | `"global"` in the current implementation.                                                          |
| `title`   | optional | Bounded static Core action description; present for a Core action and absent for a `command:` row. |
| `command` | optional | The validated qualified `owner:command` target; present only when `action` is `command:<command>`. |

`title` and `command` are mutually exclusive. A malformed canonical row is
dropped rather than echoed; an optional title may be sanitized to one bounded
line, while the binding identity fields remain strict.

### `suggest`

The current bridge returns a positive integer handle: the 1-based position of
the captured suggestion within that plugin generation. It has no host-side
binding meaning and there is no removal function. Capture is generation-scoped
and is disposed with the generation.

## Errors

Denials arrive as catchable Lua error tables with `class`, `code`, and
`message`; match on the stable `code`, not message text.

### `list` read errors

| `code`                | `class`      | When                                                                                                                                                           |
| --------------------- | ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `E_CAPABILITY_DENIED` | `runtime`    | The plugin lacks `runtime.keymaps.read`, or safe policy denies the VM. The read grant is independent of `runtime.inspect`, `runtime.configure`, and UI grants. |
| `E_NOT_IMPLEMENTED`   | `runtime`    | The host has no keymap source. The namespace remains callable, but the default host fails closed instead of returning an empty registry.                       |
| `E_VALUE_TYPE`        | `validation` | `options` is not a table, or `offset`/`limit` is not an integer. Validation happens before the host call.                                                      |
| `E_DEF_LIMIT`         | `validation` | `offset` is outside `0..=1024`, `limit` is outside `1..=256`, the serialized page exceeds 64 KiB, or the per-generation call budget is exhausted.              |

An `E_CAPABILITY_DENIED` result does not reveal whether a keymap source exists.
Likewise, a host without a source reports `E_NOT_IMPLEMENTED` rather than
silently returning an empty page.

### `suggest` capture errors

| `code`             | `class`      | When                                                                                                                           |
| ------------------ | ------------ | ------------------------------------------------------------------------------------------------------------------------------ |
| `E_DEF_INVALID`    | `validation` | `def` is not a table, a required `chord` or `command` is missing/empty/over-bound, or `when` is neither absent nor `"global"`. |
| `E_DEF_LIMIT`      | `validation` | The 129th suggestion exceeds the 128-suggestion generation capture bound.                                                      |
| Registration error | `validation` | Capture is attempted after activation, or `command` does not name a command registered by the same generation.                 |

## Capability

`bitty.keymaps.list` requires the manifest capability:

```toml
[capabilities]
runtime.keymaps.read = true
```

This is a narrow read head. It does not grant `runtime.inspect`,
`runtime.configure`, UI capabilities, or access to any other runtime data.
Conversely, `runtime.inspect` does not imply `runtime.keymaps.read`.

`bitty.keymaps.suggest` is the accepted RFC's L1 Control suggestion surface
and is described as `none (suggestion only)` rather than as a registry-read
capability. Its capture is still constrained by activation lifecycle and
generation ownership.

## Example

The following read and paging shapes are covered by
`crates/bitty-runtime/tests/plugin_keymaps.rs` in Core `7d98f53a`:

```lua
local ok, page = pcall(bitty.keymaps.list, { offset = 2, limit = 2 })
if not ok then
  return page.code
end

for _, row in ipairs(page.entries) do
  -- Required: chord, action, source, context.
  -- Optional: title for a Core action, command for a plugin command row.
  print(row.chord, row.action, row.source, row.context)
end

-- page.total is the bounded addressable total, not #page.entries.
-- page.offset is 2; page.truncated reports an unreturned tail.
```

The tests also prove that the source is read live: replacing the published
table is visible on the next call. They cover grant denial, canonical row
fields, offset/limit paging, malformed options, the addressable-row cap, and
the per-generation call budget. The bridge implementation bounds the serialized
payload by dropping whole rows from the tail to stay within `64 KiB`, and the
host `fits_budget()` guard checks the payload budget. The bridge tests cover the
default `E_NOT_IMPLEMENTED` host seam.

Suggestion capture is separately covered by the Lua parity tests
`keymaps_suggest_captures_with_global_default`,
`keymaps_suggest_rejects_malformed`, and
`keymaps_suggest_enforces_cap_at_bridge`.

## Limits

- A single page requests at most `256` rows.
- At most `1024` rows are addressable through the read API. Rows beyond that
  cap are not reachable; `total` is capped and `truncated` reports the tail.
- The canonical serialized page is at most `64 KiB`; tail rows are removed as
  whole rows to preserve canonical entries.
- `offset` is `0..=1024`; `limit` is `1..=256`.
- A plugin generation may make at most `16` keymap reads per second. Further
  calls fail with `E_DEF_LIMIT` rather than being silently accepted.
- Rows are converted from typed Core bindings at call time. The read API
  intentionally omits plugin suggestions; it is not a combined view of every
  possible dispatch source.
- Suggestion capture allows at most `128` suggestions per generation. The
  129th call fails with `E_DEF_LIMIT`.
- Suggestions are valid only during activation, are owned by the plugin
  generation, and have no removal surface.

## Compatibility and versioning

This page is draft and carries no compatibility promise. The accepted RFC's
compatibility policy applies to the accepted `suggest` surface: additions are
minor-version changes within `1.x`, while removals or narrowings require a
major version and manifest compatibility-range review. The `list` read and
`runtime.keymaps.read` capability are current Core implementation evidence at
`7d98f53a`; their presence here does not turn the separate candidate or
follow-up publication behavior for suggestions into an accepted stable
contract.

## References

- [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md)
  (accepted `suggest` spelling, definition shape, chord grammar, precedence,
  and `none (suggestion only)` capability classification)
- [Lua Reference Overview](overview.md) (error table, capability gating,
  executability rule, and L1/L2 split)
- [Lua API Reference](README.md) (namespace route table and reference status)
- `bitty@7d98f53a` `crates/bitty-lua/src/keymaps.rs` (query bounds, canonical
  row fields, page fields, payload and call bounds)
- `bitty@7d98f53a` `crates/bitty-lua/src/host.rs` (options validation, grant
  bridge, typed failures, and `suggest` capture)
- `bitty@7d98f53a` `crates/bitty-runtime/src/plugin_runtime/services.rs`
  (live source conversion, `runtime.keymaps.read` gate, and exclusion of
  suggestions from the read registry)
- `bitty@7d98f53a` `crates/bitty-runtime/tests/plugin_keymaps.rs` (real-plugin
  read, paging, grants, canonical rows, bounds, and budget evidence)
- `bitty` `crates/bitty-lua/tests/lua_parity.rs` (suggestion capture and bridge
  error evidence)
