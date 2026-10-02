---
title: bitty.net Lua Request Surface (Candidate)
description: Candidate contract for the non-blocking bitty.net Lua request surface over the DIR-030 net native component with result events errors and per-plugin bounds
category: specifications
audience: plugin-author
document_type: specification
status: draft
website_publish: false
sidebar_order: 30
---

# bitty.net Lua Request Surface (Candidate)

> Status: **draft candidate** — not **Accepted**, not **Implemented**, not
> **Verified**, and not normative. This page defines the plugin-facing Lua
> spelling for outbound HTTP requests served by the `net` native component of
> the accepted
> [DIR-030 Native Component Boundary](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/development/native-component-boundary.md).
> No host implements `bitty.net` today: the Core component broker exists in
> the `bitty` repository but is not wired to any Lua surface, and the SDK
> surface still excludes every network entry point. Constants named here are
> proposed values; they become contract only when this page is accepted.

## Purpose and scope

DIR-030 fixes the process, install, and authority model for native
components and leaves the Lua surface to this corpus, with one constraint: a
request handle plus a response event, never blocking a Lua callback. This page
specifies that surface so the host bridge, the SDK surface file, and plugin
authors share one spelling:

- the `bitty.net` namespace: `bitty.net.request` and `bitty.net.cancel`;
- the request options, their validation, and their bounds;
- the four result events `net.response`, `net.body`, `net.done`, and
  `net.error`, their payloads, ordering, and delivery rules;
- the capability and manifest requirements;
- the synchronous and asynchronous error codes;
- per-plugin concurrency and buffering bounds.

Out of scope: the HTTP backend behavior inside the component (owned by the
bitty-network repository), the wire byte layout (owned by the
`bitty-network-wire` codec), the component process lifecycle and install
commands (owned by DIR-030), WebSocket messages (reserved by DIR-030, not
specified), and secrets or authenticated-request helpers.

## Normative sources this specification must not weaken

- [DIR-030 Native Component Boundary](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/development/native-component-boundary.md):
  Core is the policy authority, the component only executes and never widens
  a grant, Core never reads `PATH`, and the per-plugin grant is the granted
  `network.connect:HOST[:PORT]` capabilities intersected with the manifest
  `[[network.egress]]` declarations.
- [Plugin API v1 Lua Surface RFC](plugin-api-v1-lua-surface-rfc.md): the
  read-only `bitty` table, one spelling per concept, typed fail-closed
  denials (`E_CAPABILITY_DENIED`, `runtime` class), small generation-owned
  integer handles that are not host objects, no colon-method variants, and
  registration only during `init.lua` activation.
- [Plugin Platform RFC](../specifications/plugin-platform-rfc.md): the
  capability grammar (`network.connect:DESTINATION`, no ambient sockets), the
  manifest validation rules, and the event pipeline (OQ-013), including the
  rule that silent event loss is not permitted.
- [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md): bounded
  host calls, per-plugin caps that refuse rather than queue silently, and the
  RC-5 event queue budgets this page does not loosen for existing classes.
- [Plugin Manifest and Capability Grammar Authority](../specifications/manifest-capability-authority.md):
  the closed capability grammar; see [Affected contracts](#affected-contracts)
  for the manifest fields this page depends on.
- The shared
  [security overview](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/overview.md)
  and
  [threat model](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/threat-model.md)
  (the component is the L3 native sidecar; plugins hold no ambient authority).

## Terminology

| Term          | Meaning                                                                                                                    |
| ------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Request id    | Positive Lua integer returned by `bitty.net.request`; owned by `(PluginId, generation)`; never reused within a generation. |
| Live request  | A request id that has not yet produced its terminal event and has not been cancelled.                                      |
| Terminal      | `net.done` or `net.error`; exactly one per live request that is not cancelled.                                             |
| Result event  | One of the four `net.*` events; a result of the plugin's own request, delivered to that plugin only.                       |
| Plugin grant  | The Core-computed host and port set for one plugin (DIR-030), attached to every wire request; the component re-checks it.  |
| Pending bytes | Body bytes of result events that Core has queued for a plugin and the plugin's handlers have not yet received.             |

## Namespace and functions

`bitty.net` is always present in the `bitty` table, like every v1 sub-table;
it is not subject to the `bitty.env` absence carve-out. Every call from a
plugin that lacks the requirements in [Capability and manifest](#capability-and-manifest)
fails closed with `E_CAPABILITY_DENIED` before any side effect.

```lua
bitty.net.request(opts) -> request_id
bitty.net.cancel(request_id) -> boolean
```

### `bitty.net.request(opts)`

Validates `opts`, checks the destination against the plugin grant, reserves
a per-plugin slot, enqueues the request for the Core component broker, and
returns a request id. It never waits for the component: resolution, digest
verification, spawn, handshake, and network I/O all happen off the Lua
thread, and every outcome after the return value arrives as a result event.
The call is valid at any time after activation (from command handlers, event
handlers, timers, and tasks); it is not a registration call.

| `opts` field     | Type    | Required | Rule                                                                                                                                                                                   |
| ---------------- | ------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `url`            | string  | yes      | Absolute `https` or `http` URL, at most `NET_URL_MAX_BYTES`; no userinfo (`user:pass@`), no fragment; the host is a DNS name or IP literal; the effective port defaults to 443 or 80.  |
| `method`         | string  | no       | One of `GET`, `POST`, `PUT`, `DELETE`, `HEAD`, `OPTIONS`, `PATCH` (upper case, the wire method set); default `GET`.                                                                    |
| `headers`        | table   | no       | Map of header name to string value; at most `NET_HEADERS_MAX` entries and `NET_HEADER_BYTES_MAX` total name plus value bytes; names are HTTP tokens; values contain no CR, LF, or NUL. |
| `body`           | string  | no       | Request body bytes (a Lua string is a byte string), at most `NET_REQUEST_BODY_MAX_BYTES`.                                                                                              |
| `timeout_ms`     | integer | no       | Positive; default `NET_DEFAULT_TIMEOUT_MS`; values above `NET_MAX_TIMEOUT_MS` are clamped, not rejected.                                                                               |
| `max_body_bytes` | integer | no       | Positive response body budget; default `NET_DEFAULT_MAX_BODY_BYTES`; values above `NET_MAX_BODY_BYTES` are clamped, not rejected.                                                      |

Validation rules:

1. `opts` must be a plain table; an unknown key, a wrong type, a non-integer
   number where an integer is required, or a non-positive `timeout_ms` or
   `max_body_bytes` fails with `E_DEF_INVALID`.
2. Header names `host`, `content-length`, `transfer-encoding`, `connection`,
   `upgrade`, `te`, `trailer`, `keep-alive`, and `proxy-authorization` (ASCII
   case-insensitive) are owned by the component and fail with
   `E_DEF_INVALID`. Duplicate names that differ only in case fail with
   `E_DEF_INVALID`.
3. A URL, header set, or body over its bound fails with `E_DEF_LIMIT`.
4. A destination host or effective port outside the plugin grant fails with
   `E_CAPABILITY_DENIED`. Host comparison is exact after ASCII lower-casing;
   there are no wildcards, matching the accepted egress host rule.
5. A ninth live request (`NET_MAX_LIVE_REQUESTS_PER_PLUGIN`), or a request
   whose body would push the plugin's in-flight request body bytes over
   `NET_PLUGIN_REQUEST_BODY_MAX_BYTES`, fails with `E_BUDGET_NET` and is never
   queued.
6. The clamped `timeout_ms` and `max_body_bytes` are the values Core forwards
   on the wire and uses for its own deadline and budget (DIR-030 refinements
   for the Core deadline and the body budget ceiling).

### `bitty.net.cancel(request_id)`

Cancels a live request. Returns `true` when the id named a live request of
the calling generation, `false` otherwise (already terminal, already
cancelled, unknown, or from a disposed generation). A non-integer argument
fails with `E_DEF_INVALID`. After `true` is returned, Core discards every
undelivered result event of that request, sends the wire `Cancel`, releases
the slot and the request body bytes, and delivers no further event for that
id; a cancelled request has no terminal event. Cancellation never blocks.

## Result events

The four result names join the closed event-name set as an additive minor
version of the plugin API line. They form a new **Result** event class:
host-produced results of the plugin's own asynchronous request, delivered to
the requesting plugin only, like the Lifecycle class. A plugin receives them
through the accepted `bitty.events.subscribe(name, handler)` call during
`init.lua`; the envelope is the accepted
`{ kind = string, sequence = integer, payload = table }`.

| Kind           | Payload                                                | Notes                                                                                                       |
| -------------- | ------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| `net.response` | `{ request_id, status, headers }`                      | At most one per request; `status` is the HTTP status integer; `headers` is an array of `{ name, value }`.   |
| `net.body`     | `{ request_id, data }`                                 | Zero or more, in byte order, only after `net.response`; `data` is a 1 to `NET_BODY_EVENT_MAX_BYTES` string. |
| `net.done`     | `{ request_id, status, body_bytes }`                   | Terminal success; `body_bytes` is the total delivered (or, without a `net.body` subscriber, received).      |
| `net.error`    | `{ request_id, kind, code, message, retry_after_ms? }` | Terminal failure; may follow `net.response` and partial `net.body`; see [Errors](#errors).                  |

Payload rules:

- Response header names are ASCII-lowercased by Core; duplicates and order
  are preserved in the array. The header set is bounded by the wire limits
  (`NET_HEADERS_MAX` entries, `NET_HEADER_BYTES_MAX` bytes).
- Every string is bounded and is untrusted network data: plugins render it
  with host-owned components and never as markup.
- `message` is a bounded, host-authored or component-authored diagnostic
  (at most `NET_ERROR_MESSAGE_MAX_BYTES`) that never echoes request headers or
  body; match on `code`, never on `message`.
- Unknown future payload fields are ignored, per the accepted compatibility
  policy.

Delivery rules (Result class; proposed addition to the OQ-013 pipeline):

1. **Owner only.** Result events are delivered only to the generation that
   issued the request. Subscribing to a `net.*` name is valid only for a
   plugin whose manifest declares `[components] net`; otherwise it is a
   registration error. `net.*` names are not valid `[lazy].events` triggers,
   because a result can exist only after activation.
2. **One ordered queue per plugin.** Core places all result events of one
   plugin in a single FIFO result queue rather than four
   `(plugin, event-type)` queues, so the events of one request are delivered
   in production order: `net.response`, then `net.body` chunks, then the
   terminal. There is no ordering across requests.
3. **Unsubscribed kinds are not queued.** An event whose kind the plugin did
   not subscribe to is dropped at production and is not counted as event
   loss; in particular, body bytes are never buffered for a plugin without a
   `net.body` subscriber. Budget accounting (`max_body_bytes`) still applies
   to the bytes the component sends.
4. **Never coalesced, never DropOldest.** Result events are not coalescable
   and are excluded from the DropOldest overflow policy. Instead, admission is
   bounded per plugin by `NET_PLUGIN_PENDING_MAX_BYTES` and
   `NET_PLUGIN_PENDING_MAX_EVENTS`. When a `net.response` or `net.body` event
   would exceed either bound, Core fails that request: it discards the
   request's undelivered `net.body` events, sends the wire `Cancel`, and
   enqueues `net.error` with code `E_NET_BUDGET`. Terminal events are always
   admitted (one reserved slot per live request), so every live request ends
   with exactly one terminal.
5. **Payload and batch bounds.** A result event payload is bounded by
   `NET_EVENT_MAX_BYTES`, distinct from the observation `EVENT_MAX_BYTES`; Core
   re-chunks component body frames (up to 192 KiB on the wire) into
   `net.body` events of at most `NET_BODY_EVENT_MAX_BYTES`. One executor
   wakeup delivers at most `NET_BATCH_MAX_EVENTS` result events or
   `NET_BATCH_MAX_BYTES` of result payload, whichever is smaller.
6. **Handler budgets.** Result handlers are ordinary observation-style
   handlers: the return value is ignored, the accepted soft-limit and
   violation accounting applies, and a handler error is attributed to the
   plugin without affecting the request.
7. **Generation end.** Suspension or disposal of a generation cancels each of
   its live requests exactly as `bitty.net.cancel` does; no result event is
   delivered to a later generation, and request ids from the old generation
   are invalid.

## Capability and manifest

A request is admitted only when all of the following hold; otherwise
`bitty.net.request` fails with `E_CAPABILITY_DENIED`:

1. The manifest declares the component requirement:

   ```toml
   [components]
   net = "^0.0.1"
   ```

   The package manager refuses to install a plugin whose requirement is
   unmet (DIR-030 component install sources). Caret matching follows Cargo
   semantics: `^0.0.1` admits exactly `0.0.1`, and `^0.1` admits
   `>=0.1.0, <0.2.0`.

2. The plugin holds at least one granted `network.connect:HOST[:PORT]`
   capability whose host has a matching `[[network.egress]]` entry:

   ```toml
   [plugin.capabilities]
   required = ["network.connect:api.example.com:443"]

   [[network.egress]]
   host = "api.example.com"
   ports = [443]
   ```

3. The request destination is inside the Core-computed plugin grant: a
   `network.connect:HOST` capability admits every declared egress port of
   `HOST`; a `network.connect:HOST:PORT` capability admits only `PORT`, and
   only when an egress entry for `HOST` declares it. Hosts without a matching
   egress entry contribute nothing. No method restriction is derived.

The grant is computed by Core from the persisted consent at request time and
attached to the wire request; the plugin never sees or supplies it. The
component re-checks it and reports a mismatch as `net.error` with kind
`denied`, which indicates a Core or component defect rather than a plugin
error.

## Errors

Synchronous errors are raised as the accepted BridgeError table
`{ class, code, message }` before any side effect:

| Code                  | Class        | Raised when                                                                                             | New in this page |
| --------------------- | ------------ | ------------------------------------------------------------------------------------------------------- | ---------------- |
| `E_CAPABILITY_DENIED` | `runtime`    | Missing `[components] net`, no usable `network.connect` grant, or destination outside the plugin grant. | no               |
| `E_DEF_INVALID`       | `validation` | Shape-invalid `opts` or `request_id` (see validation rules).                                            | no               |
| `E_DEF_LIMIT`         | `validation` | URL, header, or request body bound exceeded.                                                            | no               |
| `E_BUDGET_NET`        | `budget`     | Live-request cap or in-flight request body bytes exceeded.                                              | yes              |
| `E_NOT_IMPLEMENTED`   | `runtime`    | The host has no component broker wired to the Lua bridge (the current state of every host).             | no               |

Asynchronous errors arrive as the terminal `net.error` event. `kind` is a
closed string set: the eight wire `ErrorKind` values plus one Core-only
value. `code` is the stable identifier to match on.

| `kind`           | `code`                 | Origin                                                                                                                                                                                 | New in this page |
| ---------------- | ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| `denied`         | `E_CAPABILITY_DENIED`  | Component re-check rejected the grant (defense in depth).                                                                                                                              | no               |
| `offline`        | `E_NET_OFFLINE`        | Empty grant or unreachable destination (DNS, connect, proxy failure).                                                                                                                  | yes              |
| `timeout`        | `E_TIMEOUT`            | The component deadline or the Core request deadline expired.                                                                                                                           | no               |
| `budget`         | `E_NET_BUDGET`         | Response body exceeded `max_body_bytes`, or the pending result bounds were exceeded.                                                                                                   | yes              |
| `tls`            | `E_NET_TLS`            | TLS refused the connection (certificate, protocol, or handshake failure).                                                                                                              | yes              |
| `protocol`       | `E_NET_PROTOCOL`       | HTTP-level protocol violation by the server, or a wire protocol violation reported for the request.                                                                                    | yes              |
| `component_lost` | `E_NET_COMPONENT_LOST` | The component process crashed, failed its handshake, or was stopped while the request was in flight.                                                                                   | yes              |
| `internal`       | `E_NET_INTERNAL`       | Component-internal failure.                                                                                                                                                            | yes              |
| `unavailable`    | `E_NET_UNAVAILABLE`    | Core-only, never on the wire: component not installed, descriptor or digest verification failed, spawn failed, crash backoff, crash latch, or the broker-wide in-flight bound is full. | yes              |

`retry_after_ms` is present only on `unavailable` during crash backoff or
when the broker-wide in-flight bound is full; a plugin may retry after that
delay. After the crash latch (DIR-030: five crashes in five minutes) it is
absent, and retries fail the same way until the next Bitty start.

## Bounds and constants

| Constant                            | Value   | Source or rationale                                                                              |
| ----------------------------------- | ------- | ------------------------------------------------------------------------------------------------ |
| `NET_MAX_LIVE_REQUESTS_PER_PLUGIN`  | 8       | Keeps one plugin to one eighth of the broker-wide wire bound.                                    |
| `NET_PLUGIN_REQUEST_BODY_MAX_BYTES` | 16 MiB  | Aggregate in-flight request body bytes held by Core for one plugin.                              |
| `NET_REQUEST_BODY_MAX_BYTES`        | 8 MiB   | Wire `MAX_REQUEST_BODY_BYTES`.                                                                   |
| `NET_URL_MAX_BYTES`                 | 8 KiB   | Wire `MAX_URL_BYTES`.                                                                            |
| `NET_HEADERS_MAX`                   | 64      | Wire `MAX_HEADERS`.                                                                              |
| `NET_HEADER_BYTES_MAX`              | 16 KiB  | Wire `MAX_HEADER_BYTES`.                                                                         |
| `NET_DEFAULT_TIMEOUT_MS`            | 30 000  | DIR-030 Core deadline default (30 s).                                                            |
| `NET_MAX_TIMEOUT_MS`                | 300 000 | DIR-030 Core deadline ceiling (300 s); larger values are clamped.                                |
| `NET_DEFAULT_MAX_BODY_BYTES`        | 8 MiB   | Wire `DEFAULT_MAX_BODY_BYTES`.                                                                   |
| `NET_MAX_BODY_BYTES`                | 64 MiB  | DIR-030 response body budget ceiling; larger values are clamped.                                 |
| `NET_BODY_EVENT_MAX_BYTES`          | 64 KiB  | Per `net.body` event `data`.                                                                     |
| `NET_EVENT_MAX_BYTES`               | 80 KiB  | Per result event payload: one body chunk, or a full response head (16 KiB headers) plus framing. |
| `NET_PLUGIN_PENDING_MAX_BYTES`      | 1 MiB   | Undelivered result payload bytes per plugin.                                                     |
| `NET_PLUGIN_PENDING_MAX_EVENTS`     | 256     | Undelivered result events per plugin, excluding reserved terminal slots.                         |
| `NET_BATCH_MAX_EVENTS`              | 32      | Same event count as the accepted observation batch bound.                                        |
| `NET_BATCH_MAX_BYTES`               | 256 KiB | Result payload per executor wakeup.                                                              |
| `NET_ERROR_MESSAGE_MAX_BYTES`       | 1 KiB   | Wire `MAX_ERROR_MESSAGE_BYTES`.                                                                  |

The wire-derived values are owned by the `bitty-network-wire` codec and are
repeated here only so plugin authors see one table; if the codec changes,
the codec wins and this table follows.

## Example

```lua
-- init.lua: subscriptions are registration calls, valid only here.
local chunks = {}

bitty.events.subscribe("net.response", function(ev)
  chunks[ev.payload.request_id] = {}
end)

bitty.events.subscribe("net.body", function(ev)
  local parts = chunks[ev.payload.request_id]
  if parts then
    parts[#parts + 1] = ev.payload.data
  end
end)

bitty.events.subscribe("net.done", function(ev)
  local body = table.concat(chunks[ev.payload.request_id] or {})
  chunks[ev.payload.request_id] = nil
  bitty.notify.show({ title = "Fetched", body = #body .. " bytes" })
end)

bitty.events.subscribe("net.error", function(ev)
  chunks[ev.payload.request_id] = nil
  bitty.notify.show({ title = "Request failed", body = ev.payload.code })
end)

bitty.commands.register({
  id = "fetch",
  title = "Fetch status",
  run = function()
    local ok, id = pcall(bitty.net.request, {
      url = "https://api.example.com/status",
      headers = { accept = "application/json" },
      timeout_ms = 10000,
    })
    if not ok then
      return { error = id.code }
    end
    return { request_id = id }
  end,
})
```

The example is illustrative and maps to no executable test yet.

## Security review

- **No ambient authority.** The namespace is inert without `[components] net`,
  a granted `network.connect` capability, and a matching egress declaration;
  the destination is checked in Core before the request leaves the plugin
  host, and again in the component.
- **Grant integrity.** The plugin never constructs the grant; Core computes it
  per request from persisted consent, so a revoked capability takes effect on
  the next request. In-flight requests are not retroactively cancelled by a
  revocation in this candidate; this is an open point.
- **Credentials.** URL userinfo is rejected, `proxy-authorization` is
  component-owned, and error messages never echo request headers or bodies.
  Plugin-supplied `authorization` headers are allowed and are the plugin's
  own secret; a host-managed secrets surface is out of scope.
- **No blocking.** Every call returns without waiting on the component, the
  network, or file I/O, so a slow or hostile endpoint cannot stall a Lua
  callback or the executor (T-07 availability).
- **Resource exhaustion.** Live requests, request body bytes, response
  budgets, pending result bytes and events, payload size, and batch size are
  all bounded; overflow refuses or fails the one request, never queues
  silently and never drops a terminal.
- **Untrusted data.** Response status, headers, body, and messages are
  untrusted network input delivered as bounded immutable copies.
- **Residual risk.** The component is an unsandboxed L3 sidecar until the
  DIR-030 sandboxing follow-up lands; a compromised component can reach any
  destination the user's account can. This page does not reduce that
  residual risk; the Core pre-check limits what an honest plugin can ask for.

## Verification plan

Required before this page can move beyond draft, and before any host claims
the surface:

1. **Host parity.** `bitty.net.request` and `bitty.net.cancel` map to the Core
   broker; the four event names round-trip through the host event-kind parser;
   the SDK surface file adds the namespace and drops the `bitty.network`
   exclusion only when the host verdict is wired.
2. **Negative tests.** Missing `[components] net`, missing capability,
   undeclared egress host, undeclared port, userinfo URL, forbidden header,
   oversized URL, headers, and body, the ninth live request, and the
   aggregate request body bound each fail with the documented code before any
   wire frame.
3. **Delivery tests.** Ordering within a request; exactly one terminal per
   uncancelled request; no events after `cancel` returns `true`; pending-bound
   overflow yields `E_NET_BUDGET` and a wire `Cancel`; unsubscribed kinds are
   not buffered; generation disposal cancels live requests.
4. **Clamping tests.** `timeout_ms` above `NET_MAX_TIMEOUT_MS` and
   `max_body_bytes` above `NET_MAX_BODY_BYTES` are clamped on the wire and in
   the Core deadline and budget.
5. **Error mapping.** Each wire `ErrorKind` maps to the documented `kind` and
   `code`; each Core-side broker failure maps to `unavailable` or
   `component_lost` as documented.
6. **Non-blocking evidence.** A test component that never answers does not
   delay the calling handler, and the request ends with `E_TIMEOUT` at the
   Core deadline.
7. **Documentation.** `just check` passes; the security reviewer, the
   category owner, and the docs curator review this page.

## Alternatives considered

- **Callback form** (`bitty.net.request(opts, on_result)`, like
  `bitty.timers.create(delay_ms, callback)`). Rejected. A timer produces one
  result; a request produces a head, a stream of chunks, and a terminal, so a
  callback form needs either several closures per request or a multiplexed
  callback that re-implements event dispatch. Timer callbacks already deliver
  through the event path, so the events form adds no new delivery machinery,
  reuses the accepted subscription, handler budget, violation, and
  generation-disposal rules, and retains no per-request closure in the host.
  DIR-030 also names a request handle plus a response event.
- **Handle object with `id` and `cancel()`.** Rejected. The accepted Lua
  surface rule is that handles are small generation-owned integers, not host
  objects, and that there are no colon-method variants. The integer request
  id is also the correlation key in every result event, and
  `bitty.net.cancel(request_id)` mirrors `bitty.tasks.cancel` and
  `bitty.timers.cancel`.
- **Synchronous `bitty.http.get`.** Rejected: it blocks a Lua callback on the
  network, which DIR-030 forbids. The earlier sketch in the plugin system
  candidate is superseded by this page.
- **One buffered `net.done` carrying the whole body.** Rejected: an 8 MiB
  default budget (64 MiB ceiling) in one event would break every payload and
  batch bound; chunked `net.body` events keep memory proportional to the
  pending bound.
- **Reusing DropOldest for result events.** Rejected: dropping a body chunk
  silently corrupts the response and dropping a terminal leaks a slot. Failing
  the one request with `E_NET_BUDGET` is visible and bounded.
- **Namespace `bitty.network` or `bitty.http`.** Rejected: `bitty.net` matches
  the DIR-030 component name `net`, and the excluded `bitty.network` spelling
  belonged to the removed in-process binding.

## Affected contracts

- [Plugin API v1 Lua Surface RFC](plugin-api-v1-lua-surface-rfc.md): an
  additive minor version adds the `bitty.net` namespace, the Result event
  class, and the four event names to the closed set.
- [Plugin Platform RFC](../specifications/plugin-platform-rfc.md): the
  accepted manifest schema must admit `[components]` and `[[network.egress]]`,
  and the event pipeline gains the Result class delivery rules above.
- [Plugin Manifest and Capability Grammar Authority](../specifications/manifest-capability-authority.md):
  section 6 currently classifies `[[network.egress]]` as rejected, while
  DIR-030 and the Core grant computation depend on it. That classification
  must be amended by a reviewed change before this surface can be accepted.
- [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md): the
  per-plugin network bounds above become a new resource-ceiling row.
- The SDK surface file (`bitty-plugin-api-v1.json` in the bitty-plugin-sdk
  repository): adds the namespace, types, events, and error codes once the
  host verdict is wired; until then the `bitty.network` exclusion stays.
- [DIR-030 Native Component Boundary](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/development/native-component-boundary.md):
  links this page as the Lua request surface.

## Open points

- Amend the manifest authority section 6 and the Plugin Platform RFC manifest
  schema to accept `[components]` and `[[network.egress]]` (owner: plugin
  platform contract owners).
- Whether revoking a `network.connect` grant cancels in-flight requests of
  that plugin, or only refuses new ones.
- Whether the Result class needs a cross-plugin global pending bound in
  addition to the per-plugin bound.
- Response trailers, redirects policy (follow, limit, cross-host), and
  decompression are component behavior owned by the bitty-network
  repository; this page surfaces their results but does not fix them.
- WebSocket and streaming request bodies are not specified.

## Acceptance criteria

- The manifest schema conflict in [Open points](#open-points) is resolved.
- The category owner, the docs curator, and a security reviewer approve the
  namespace, events, errors, and bounds.
- The verification plan items have named owning tasks in the `bitty`
  repository.
- `just check` passes.

## P0 Review Sign-off

Not signed off. This page is a draft candidate and carries no P0 approval.

## References

- [DIR-030 Native Component Boundary](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/development/native-component-boundary.md)
- [Plugin API v1 Lua Surface RFC](plugin-api-v1-lua-surface-rfc.md)
- [Plugin Platform RFC](../specifications/plugin-platform-rfc.md)
- [Plugin Manifest and Capability Grammar Authority](../specifications/manifest-capability-authority.md)
- [Isolation and Resource RFC](../runtime/isolation-resource-rfc.md)
- [Plugin system](../extensibility/plugin-system.md)
- [Package management](../extensibility/package-management.md)
- [bitty-network](https://github.com/bitty-terminal/bitty-network) — owner of
  the `bitty-network-wire` codec and the `net` component.
