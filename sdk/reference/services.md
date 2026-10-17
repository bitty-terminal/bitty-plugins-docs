---
title: bitty.services Reference
description: Draft service reference separating historical v1 authority from the implemented runtime backend and fail-closed host defaults
category: reference
audience: plugin-author
document_type: reference
status: draft
website_publish: true
sidebar_order: 120
---

# bitty.services Reference

> Status: **draft**. The historical accepted v1 surface lives in the
> [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md).
> Current runtime observations below are **Implemented-only**, pinned to
> Core `1c68de2545c961790d0a2011340e4a3c04564ab7`; they are not Verified,
> compatible, or release-ready claims and accept no new manifest/API contract.

## Purpose and scope

`bitty.services` resolves provided services and registers provider
functions. This reference separates the accepted minimal v1 spellings
(LUA-OQ-8), current draft/v2 composition direction, and the pinned Core
implementation. Implementation evidence does not amend the frozen v1
contract or accept the host's experimental manifest forms.

At the pinned revision, `crates/bitty-lua/src/host.rs` installs get/provide
callbacks. `PluginHostServices` in
`crates/bitty-runtime/src/plugin_runtime/services.rs` overrides
`service_provide_check`, `service_resolve`, and `service_call`;
`plugin_runtime/mod.rs` wires the runtime-shared service directory and
ledger before activation. These are callable in-process backend paths.
The `HostServices` trait defaults still fail closed with
`E_NOT_IMPLEMENTED`, as does the runtime seam without a directory. Those
defaults do not describe the wired product runtime. Older route-table
`not_implemented` labels describe the earlier reference snapshot and are
not current backend evidence.

## Signature

The accepted v1 spellings are also installed by the pinned Lua bridge:

```lua
bitty.services.get(iface, opts) -> service | nil
bitty.services.provide(iface, impl) -> handle
```

The runtime makes them conditional on manifest declarations, backing-head
grants, lifecycle state, and bridge validation. Provider registration is
activation-time capture, not permission to publish new authority.

## Params

The consumer takes an interface name and options with `version` and
`optional`. In the pinned host, `opts` and `opts.version` may be omitted;
the host then uses the caller manifest's matching `services.required`
requirement. An explicit version overrides only the requirement text.
Neither it nor `optional = true` substitutes for that declaration.
An owner-qualified lookup needs its matching qualified declaration; a
short declaration does not authorize the qualified lookup.

This is an **experimental host declaration gate**, not accepted v1 schema:
the accepted
[host-only forms classification](../../specifications/manifest-capability-authority.md#6-host-only-manifest-forms-classification)
still rejects `[services.required]` as a stable field. Historical v1 puts
plugin/service dependencies in `[dependencies]`; that field alone does
not satisfy current `service_resolve`. Resolving this contract difference
belongs to the owning manifest/API evolution, not this reference.

The provider takes a short interface name and a plain `impl` table of
functions. The manifest's `[services.provided]` table supplies a concrete
version, non-empty distinct closed backing capability `heads`, and bounded
`args_schema`/`result_schema` for schema-validating consumers. The
[backing-head authority](../../specifications/manifest-capability-authority.md#7-provided-service-backing-heads-dynapi-gating)
supersedes historical unbacked examples; string shorthand cannot express
the required heads. A declaration does not grant any of them.

## Returns

Resolution builds a Lua table of callable method closures over a snapshot
`ServiceRoute`, pinning provider, generation, interface, and version. A
republished replacement cannot hijack that route. Arguments and results
cross VM boundaries as copied values; they never expose live VM objects.
Declared argument schemas are checked before provider execution and result
schemas before returning to the consumer.

For a declared lookup, `optional = true` returns Lua `nil` when no visible
provider satisfies the requirement, including when backing-head grants
hide the provider. It does not suppress missing declarations, malformed
names, or ambiguous short-name resolution. These are pinned host mechanics,
not a new v1 or cross-process contract.

## Errors

Denials arrive as catchable Lua error tables (`class` / `code` / `message`,
per `BridgeError::to_error` in `crates/bitty-lua/src/host.rs`); match on
`code`. The following records the pinned implementation's denial paths.

| `code`                 | `class`      | When                                                                                                                                    |
| ---------------------- | ------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| `E_NOT_IMPLEMENTED`    | `runtime`    | Host trait defaults or runtime seam without a service directory, after bridge shape validation; not a universal product-runtime denial. |
| `E_SERVICE_RESOLUTION` | `resolution` | Missing required declaration, invalid or ambiguous reference, or no visible compatible provider for a non-optional lookup.              |
| `E_SERVICE_GONE`       | `runtime`    | Provider unavailable, stale generation/version route, unpublished method, or call-time consumer grant cover missing.                    |
| `E_SERVICE_INVALID`    | `validation` | Declared argument or result schema violated at the cross-VM boundary.                                                                   |
| `E_SERVICE_UNDECLARED` | `validation` | Undeclared, unbacked, or invalid provide-side interface.                                                                                |
| `E_CAPABILITY_DENIED`  | `runtime`    | Provider lacks a declared backing head.                                                                                                 |
| `E_SERVICE_FAILED`     | `runtime`    | Provider VM is busy or its method execution fails.                                                                                      |

## Capability

The historical minimal v1 extension table lists no single consumer-side
service capability and keeps provider execution under the callee's grants.
It must not be read as permission for grant-free consumption in the current
host. The later accepted backing-head authority and pinned implementation
require **both** provider and consumer grant cover. There is no new
`services` capability family.

A provided interface declares existing Core capability heads in the manifest
(`[services.provided].*.heads`, non-empty distinct closed ids,
`MAX_SERVICE_BACKING_HEADS` 8). The provider must hold a grant for every
head. Provision with an ungranted head fails closed with a capability
denial; an unbacked declaration has a naming diagnostic. The consumer
resolves and calls only when its own grant covers every head: uncovered
records are hidden as absent at resolution, and calls fail as gone. This
consume-side denial does not expose ungranted provider availability.
Brokered calls record `{ provider, iface, method, heads }` into the
runtime-shared observe-only `ServiceCallLedger` (`SERVICE_CALL_LEDGER_CAPACITY`
256, overflow drops newest and counts the drop). Full grammar in the
[manifest and capability authority](../../specifications/manifest-capability-authority.md#7-provided-service-backing-heads-dynapi-gating).

## Example

Default-seam denial shape from `crates/bitty-lua/tests/lua_parity.rs`
(`services_get_provide_are_not_implemented`). This example assumes a host
using the trait defaults without a service backend:

```lua
local ok, err = pcall(bitty.services.get, "iface", {})
-- ok == false; err.code == "E_NOT_IMPLEMENTED"; err.class == "runtime"
-- The wired PluginRuntime instead runs declaration and resolution gates.
```

Real-runtime evidence is separate. At the pinned revision,
`crates/bitty-runtime/tests/plugin_services.rs` activates provider and
consumer plugin VMs against the shared directory. Its sources include:

- `provide_resolve_call_round_trip_with_schemas`: callable methods,
  copied values, and argument/result schema refusals.
- `suspend_resume_dispose_fail_closed_with_gone` and
  `optional_missing_resolves_nil_while_provider_suspended`: lifecycle and
  optional-absence behavior.
- `unbacked_provide_rejected_at_install_with_naming_diagnostic` and
  `head_spoofing_provide_naming_ungranted_head_rejected_at_record`:
  unbacked and ungranted provision refusal.
- `confused_deputy_consume_without_grant_fails_closed_indistinguishable_from_absent`
  and `granted_calls_record_provider_owner_and_backing_heads`: consumer
  grant-cover denial versus a granted call and observe-only broker metadata.

These test sources exercise a wired backend, rather than only documenting
trait vocabulary. This reference update inspects them read-only; it does
not report a fresh execution or independent Verified acceptance.

## Limits

- The pinned runtime backend is Implemented-only; hosts retaining the
  trait defaults or lacking directory wiring still deny calls. Neither
  test source presence nor callable callbacks establish release acceptance.
- A republished replacement never hijacks a pinned route, and a stale
  generation fails closed (`service_call` implementation).
- Only table-form `[services.provided]` entries are resolvable by
  schema-validating consumers (RFC "Services").
- Backed provision only (accepted authority and pinned host): only backed
  providers (non-empty `heads`) record at all; unbacked, unknown-head,
  duplicate-head, or over-limit provisions are rejected with a naming
  diagnostic. Grant cover is fail-closed in both directions; ungranted
  consumption is indistinguishable from absent at resolution or gone at
  call time.
- Broker ledger (host evidence, `bitty` #2037): runtime-shared
  observe-only `ServiceCallLedger`, capacity 256, drops newest on
  overflow; a missing ledger never blocks a call.
- Provider ecology beyond this minimal contract is explicitly deferred
  (RFC "Services").
- Current draft/v2 provider ecology, async completion, and cross-process
  transport remain candidates. The pinned host's version/arbitration
  mechanics do not accept those contracts. No retry, caching, transport,
  or concrete service schema is defined by this page.

## Compatibility and versioning

Draft reference: historical v1 spellings, current draft/v2 direction, and
Implemented-only host mechanics have separate standing. In particular,
`services.required` remains experimental despite being enforced by this
host; neither it nor service multiplicity becomes accepted v1 schema here.
The host's `API_VERSION` still reports `1.0.0` at the pinned revision; this
is not evidence that a v2 contract shipped or that the experimental forms
are compatible. The target v1 contract is the
[Compatibility policy](../plugin-api-v1-lua-surface-rfc.md#compatibility-policy)
in the accepted Plugin API v1 Lua Surface RFC: the surface is stable within
`1.x`, additions ship as minor versions, and removals or narrowings require a
major version, gated by the manifest `compat.plugin-api` range against the
runtime `bitty.api_version`.

## References

- [Plugin API v1 Lua Surface RFC](../plugin-api-v1-lua-surface-rfc.md)
  (Accepted; "Services" spellings, `opts` schema, `E_SERVICE_RESOLUTION` /
  `E_SERVICE_GONE` vocabulary, LUA-OQ-8, deferred provider ecology)
- [Lua Reference Overview](overview.md) (error contract, capability gating,
  executability rule, L1/L2 split)
- [Lua API Reference](README.md) (historical route-table snapshot; the
  earlier `not_implemented` row is not current runtime wiring evidence)
- Core `1c68de2545c961790d0a2011340e4a3c04564ab7` supplies all executable
  source observations on this page:
  `crates/bitty-lua/src/host.rs` (`HostServices` defaults, installed
  callbacks, `ServiceRoute`, `API_VERSION`)
- `bitty` `crates/bitty-lua/tests/lua_parity.rs`
  (`services_get_provide_are_not_implemented`,
  `not_implemented_constructor_shape`)
- [Manifest and capability authority](../../specifications/manifest-capability-authority.md#7-provided-service-backing-heads-dynapi-gating)
  (accepted backing-head grammar, grant-cover rule, broker ledger)
- `bitty` `crates/bitty-plugin-host/src/manifest.rs` (same Core pin;
  `ProvidedService.heads`, `MAX_SERVICE_BACKING_HEADS`, unbacked naming
  diagnostic) and `crates/bitty-runtime/src/plugin_runtime/services.rs`
  (grant-cover checks, `ServiceCallRecord`, `ServiceCallLedger`,
  `SERVICE_CALL_LEDGER_CAPACITY` 256)
- `bitty` `crates/bitty-runtime/src/plugin_runtime/mod.rs` (same Core pin;
  shared directory and ledger wiring before activation)
- `bitty` `crates/bitty-runtime/tests/plugin_services.rs` (same Core pin;
  real-VM conformance sources named above)
- [Cross-Plugin Composition (Candidate)](../../extensibility/cross-plugin-composition-candidate.md)
  (draft composition direction, async/cross-process exclusions)
