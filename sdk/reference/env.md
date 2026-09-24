---
title: bitty.env Reference
description: Not-implemented host environment read surface with per-key env grant gate and typed denials
category: reference
audience: plugin-author
document_type: reference
status: draft
website_publish: true
sidebar_order: 110
---

# bitty.env Reference

> Status: **draft**. Normative surface detail lives in the accepted
> [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md) and
> [ADR 0006](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0006-os-env-policy.md);
> executable behavior lives in the `bitty` repository
> (`crates/bitty-lua/src/host.rs`, `bitty@7da6d6f`). This page states no
> behavior beyond what those sources pin.

## Purpose and scope

`bitty.env` is the accepted host-mediated surface for reading host
environment variables without ambient OS authority (the RFC references the
ADR 0006 contract instead of redefining it). Its current status is
`not_implemented`: the `bitty` route table marks it `not_implemented`
([Lua API Reference](README.md)), and every call fails closed until a host
backend lands (`crates/bitty-lua/src/host.rs`, `bitty@7da6d6f`). Do not
treat its functions as usable.

## Signature

There are no implemented Lua entry points in this namespace. The accepted
RFC vocabulary names `bitty.env.get` / `bitty.env.has`
(RFC "Notifications and environment"), but neither is callable behavior:
the `HostServices` trait defaults fail closed with `E_NOT_IMPLEMENTED`
(`crates/bitty-lua/src/host.rs`, `bitty@7da6d6f`) — `env_get` at
`host.rs:533` (`Err(BridgeError::not_implemented("bitty.env.get"))`) and
`env_has` at `host.rs:544`
(`Err(BridgeError::not_implemented("bitty.env.has"))`), both constructed by
`BridgeError::not_implemented` at `host.rs:444`.

## Params

The following describes the expected contract shapes only, as derived from
the Rust side — not callable behavior. Keys are validated by the caller
shape `[A-Za-z_][A-Za-z0-9_]*` of `1..128` bytes
(`crates/bitty-lua/src/host.rs` `env_get` docs, `bitty@7da6d6f`). An
over-bound key is rejected fail-closed with `E_DEF_LIMIT` before any grant
check (`ENV_KEY_MAX_BYTES = 128`, `host.rs:81-86`). A shape-invalid key is
rejected with `E_DEF_INVALID` (pinned by `env_bridge_rejects_malformed_keys_before_grants` in `crates/bitty-lua/tests/lua_parity.rs`).

## Returns

The following describes the expected contract shapes only, as derived from
the Rust side — not callable behavior. A granted-but-absent key resolves
to `Ok(None)`, i.e. Lua `nil` (`crates/bitty-lua/src/host.rs` `env_get`
docs, `bitty@7da6d6f`). With a valid grant, a read for a non-allowlisted
key returns `nil`, indistinguishable from an unset variable (RFC
"Notifications and environment").

## Errors

Denials arrive as catchable Lua error tables (`class` / `code` / `message`,
per `BridgeError::to_error` in `crates/bitty-lua/src/host.rs`,
`bitty@7da6d6f`); match on `code`.

| `code`              | `class`      | When                                                                                                                                                                                                                                                                                            |
| ------------------- | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `E_NOT_IMPLEMENTED` | `runtime`    | Every call on the default host: no env backend is wired (`host.rs:533`, `host.rs:544`). An ungranted key shares this same code, so callers cannot probe which keys exist (`host.rs` `env_get` docs; `env_bridge_denies_ungranted_keys_without_leak` in `crates/bitty-lua/tests/lua_parity.rs`). |
| `E_DEF_INVALID`     | `validation` | Shape-invalid key at the bridge (caller shape above; `env_bridge_rejects_malformed_keys_before_grants` in `crates/bitty-lua/tests/lua_parity.rs`).                                                                                                                                              |
| `E_DEF_LIMIT`       | `validation` | Over-bound key (`> 128` bytes) at the bridge, before any grant check (`host.rs:81-86`).                                                                                                                                                                                                         |

## Capability

`CapabilityFamily::Env` (`crates/bitty-plugin-host/src/capability.rs`,
`Env` variant: host-mediated environment reads, `env.read:<KEY>`). The
expected gate is one grant per key: the trait docs require an
`env.read:<KEY>` grant for `key`
(`crates/bitty-lua/src/host.rs` `env_get` docs, `bitty@7da6d6f`), and the
RFC extension-level split lists the surface as gated by `env:<KEY>` (RFC
extension table). `bitty.env` is absent from the VM unless the manifest
declares the grant (the ADR-0006 carve-out in RFC "Notifications and
environment" and LUA-OQ-2); grants, families, and lifecycle belong to the
capability contract and are never redefined here
([Lua Reference Overview](overview.md)).

## Example

Denial shape, verified against `crates/bitty-lua/tests/lua_parity.rs`
(`env_get_has_are_not_implemented`, default seam with no env backend):

```lua
local ok, err = pcall(bitty.env.get, "HOME")
-- ok == false; err.code == "E_NOT_IMPLEMENTED"; err.class == "runtime"
-- (the default HostServices::env_get fails closed; host.rs:533)
```

The granted path has no passing test behind a wired backend; any
granted-path claim beyond this denial shape is follow-up work.

## Limits

- Not implemented: this namespace has no callable signatures, and every
  statement above about params, returns, and gates describes the expected
  contract only. All of it is subject to change when a host backend lands.
- Key-shape checks (`E_DEF_INVALID` / `E_DEF_LIMIT`) run before any grant
  check; oversize input never reaches the allowlist (`host.rs:81-86`).
- Ungranted keys are deliberately indistinguishable from unimplemented ones
  (same `E_NOT_IMPLEMENTED` code), so key presence cannot be probed
  (`host.rs` `env_get` docs).
- Ambient reads stay denied: values cross only as bounded strings through
  the host boundary, never through ambient `os.getenv`
  (`crates/bitty-plugin-host/src/capability.rs`, `Env` docs).
- Out of scope (no source): enumeration of keys, writes to the host
  environment, and any delivery, caching, or reload semantics.

## References

- [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md)
  (Accepted; "Notifications and environment" spellings, `env:<KEY>` gate,
  LUA-OQ-2 absent-unless-declared carve-out)
- [ADR 0006](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/adrs/ADR-0006-os-env-policy.md)
  (accepted `bitty.env.*` contract; referenced, not redefined)
- [Lua Reference Overview](overview.md) (error contract, capability gating,
  executability rule, L1/L2 split)
- [Lua API Reference](README.md) (route table; `env.md` row,
  `not_implemented`)
- `bitty` `crates/bitty-lua/src/host.rs` (`BridgeError::not_implemented`
  at `host.rs:444`; `env_get` default at `host.rs:533`; `env_has` default
  at `host.rs:544`; `ENV_KEY_MAX_BYTES` at `host.rs:81-86`)
- `bitty` `crates/bitty-plugin-host/src/capability.rs`
  (`CapabilityFamily::Env`, `env.read` closed identifier)
- `bitty` `crates/bitty-lua/tests/lua_parity.rs`
  (`env_get_has_are_not_implemented`,
  `env_bridge_denies_ungranted_keys_without_leak`,
  `env_bridge_rejects_malformed_keys_before_grants`)
