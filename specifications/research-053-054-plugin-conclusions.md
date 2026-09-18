---
title: Research 053 and 054 plugin-side conclusions
description: Draft candidate distillation of the plugin-side conclusions of research records 053 and 054
category: provenance
audience: plugin-author
document_type: research
status: draft
website_publish: false
sidebar_order: 50
---

# Research 053 and 054 plugin-side conclusions

> Status: **draft**, research-derived design input distilling the plugin-side
> conclusions of two research records — record 053 (public plugin contracts,
> layered Lua frameworks) and record 054 (Lux-based Lua dependency management,
> self-contained artifacts). This page is **not** an accepted contract, an RFC,
> or an implementation claim.

## 1. Purpose, status, and attribution

- This page is built only from the trimmed English summaries of research
  records 053 and 054 in the workspace research archive
  ([bitty-terminal/research](https://github.com/bitty-terminal/research)). The
  archive marked records 044 through 055 Captured and renamed the originals
  with a `.completed` suffix on 2026-09-18 under an owner directive; the
  archive states the `*-docs` corpora are the working corpus and it records
  discussion provenance. Record 053 has no recorded discussion date; record
  054's archive receipt is dated 2026-09-18. This page asserts no independent
  verification of canonical capture for either record.
- Only the plugin-side conclusions are distilled here. Model, provider, tool,
  agent, context, registry, SDK-packaging, and governance conclusions are
  recorded as owner-pending pointers in section 4, not as decisions of this
  corpus.
- A parallel in-review capture (CTX-0027) covers the 053 plugin-service and
  framework-layering direction in the specifications tree. This page is scoped
  as a complementary single-entry distillation built only from the two
  summaries named above; it introduces no competing normative wording. Where an
  accepted document already decides a point, this page links it instead of
  restating it, and accepted contracts always win over proposals recorded here.
- Documentation capture only: no SDK, framework, transport, registry, or
  product feature is implemented by this page.

## 2. Public plugin contracts and layered frameworks (053, plugin slice)

Proposal direction from research record 053. Every item below is a candidate
unless it cites an accepted document.

- **Private modules versus public contracts.** Ordinary `require()` is for
  private modules inside one plugin. Cross-plugin consumers use declared
  public services or mediated proxies instead of reaching into another
  repository's source by relative path or module-search-path mutation.
  Repositories are publication boundaries; versioned service contracts are
  architectural boundaries.
- **Typed, schema-backed contracts.** Service interfaces carry explicit
  request, response, and event shapes with versioning plus Lua tooling and
  type hints, so contracts are checkable rather than undocumented tables. One
  interface may have several conforming implementations; consumers depend on
  neither the provider's repository layout nor its implementation language.
- **Install dependencies versus service requirements.** A package dependency
  requires installation of a particular plugin; a service requirement needs a
  compatible provider of an interface. The two declarations stay distinct so an
  official, user-supplied, or other authorized provider can be substituted
  without rewriting consumers. A declared service requirement is not itself a
  permission grant.
- **Location transparency is a proposal, not fake synchrony.** A common
  service-facing abstraction could mediate same-runtime and cross-process
  calls, but local shortcuts must neither bypass enforcement nor present
  remote calls as ordinary synchronous calls that block the UI or event loop.
  The direction is async-first calls with standardized streaming; any accepted
  contract must also define cancellation and lifecycle behavior explicitly.
  Recorded await, promise, coroutine, callback, and stream-iteration spellings
  are discussion alternatives, not accepted SDK methods.
- **Four layers.** Rust enforcement and mechanisms (rendering, font shaping,
  input and IME, scheduling, resource lifecycles) lead to a narrow public Lua
  SDK (stable general wrappers over explicitly admitted host operations),
  then replaceable optional framework plugins (widgets, layouts, themes,
  dashboards, workflows), then application and user plugins. Ordinary plugins
  never bind to raw Rust internal APIs, and public contracts insulate
  consumers from host-internal change as a design aim, not an unconditional
  compatibility guarantee.
- **Small core and SDK.** Only justified, stable public abstractions belong in
  the SDK; there is no second heavyweight Lua Core. Frameworks evolve
  separately while the reviewed public boundary stays small.
- **Optional, competing frameworks.** UI composition and higher-level
  authoring live in independently versioned plugins rather than one mandatory
  official framework. Reactive, immediate-mode, terminal, canvas, and
  dashboard-oriented approaches are illustrative alternatives, not approved
  packages, APIs, or a roadmap.
- **Authority and lifecycle take precedence.** Accepted security requirements
  and lifecycle schemas override the record's speculative service methods,
  coroutine and stream interfaces, custom event vocabulary, and SDK facility
  lists. Process execution, agent spawning, IPC, networking, and credentials
  are illustrative mechanism needs, never ambient access. Framework wrappers
  cannot bypass host enforcement.

The accepted plugin-composition baseline lives in
[Plugin system](../extensibility/plugin-system.md): per-plugin isolated VMs,
no cross-plugin private imports, and versioned host-mediated services. Service
multiplicity, provider selection, and capability grants stay host-controlled.

## 3. Lua dependency management and self-contained artifacts (054, plugin slice)

Proposal direction from research record 054. Every item below is a candidate
unless it cites an accepted document.

- **Two graphs behind one CLI.** The Bitty plugin graph (lifecycle,
  permissions, services, compatibility; managed by Bitty) and the Lua package
  graph (sources, resolution, semver, lockfiles; resolved with Lux tooling)
  stay separate resolvers behind unified install, update, remove, and list
  commands. Plugin management never equates the Bitty plugin system with the
  Lux Lua package manager: plugins carry lifecycle, permission, service,
  panel, UI, and compatibility concerns that plain Lua packages do not.
- **Lux as reference and build-time backend.** Manifest, lockfile with
  integrity, git, local, and workspace dependencies, vendoring, LuaRocks
  compatibility, and semver resolution follow Lux tooling, without a hard
  runtime ABI dependency on its pre-1.0 embedding API.
- **Host-owned runtime module loader.** The host implements a resolver chain
  (plugin-local, plugin dependencies, framework packages, stdlib, vendored
  libraries) over the sandboxed Lua runtime, banning native extensions and C
  loading rather than emulating a full traditional Lua runtime.
- **Self-contained installable artifacts.** A plugin ships its manifest, Lua
  sources, vendored pure-Lua dependencies, and assets with dependencies
  pre-resolved, favoring reproducibility, startup speed, security, offline
  install, sandboxing, and version consistency over on-device resolution.
- **Build-time versus runtime split.** Lux is confined to development and
  packaging (declare, resolve, vendor, pack); end-user installs need no Lux
  toolchain.
- **Sandbox-compatible dependency subset.** Only defined pure-Lua dependencies
  are admitted: no native code, no arbitrary process execution, no dynamic
  native loading. Native dependencies are rejected at the manager level;
  native needs such as storage route through host Rust services.
- **Separate manifests per level.** The Bitty-owned plugin manifest
  (identity, entrypoint, host and harness compatibility, provided and required
  services, plugin dependencies, permissions) stays separate from the Lua
  dependency manifest; there is no mega-manifest across abstraction levels.
- **Unified CLI over internal steps.** Add, update, sync, doctor, list, enable
  and disable, info, and permissions commands run internal resolve, verify,
  unpack, register, and loader-configure steps, with two lock levels: a
  user-side plugin lock plus a build-side Lua lock baked into the artifact.
- **No auto-install on repository entry.** Project composition files never
  auto-install or execute plugins when entering a repository; declarations
  prompt for explicit user trust first. This is a supply-chain boundary.
- **Three layers.** A Bitty-implemented Plugin Manager, a host-controlled Lua
  Resolver (Lux at build time), and a Service Registry for plugin wiring over
  the Lua runtime: host network dependencies stay out of install and resolve
  paths, and the plugin registry and index serve as the distribution point.

The accepted package-management baseline lives in
[Plugin package management](../extensibility/package-management.md): the
manager-versus-host split, the staged verify-then-activate transaction model,
and the installation-executes-no-code rule. This page weakens none of them.

## 4. Owner-pending pointers (not captured here)

The following conclusions from the two summaries belong to other owners and
are pointers only, not decisions of this corpus.

| Theme                                                                                       | Disposition                                                                                         |
| ------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| 053 model APIs, provider substitution, normalized model streams                             | Pending AI and Wheel owner capture; only the generic bounded-stream direction is noted in section 2 |
| 053 tool discovery and registry, schema and permission DSL, host execution wrapper          | Pending AI and Wheel owner capture; no tool contract is accepted here                               |
| 053 agent, context, workflow, and multi-agent orchestration                                 | Pending AI and Wheel owner capture; no agent API is accepted here                                   |
| 053 host enforcement and SDK boundaries                                                     | Core and SDK documentation owners; linked, never copied                                             |
| 053 UI mechanism and composition boundaries                                                 | Terminal-platform documentation owner; linked, never copied                                         |
| 054 plugin-manager ownership, manifest and permission model, resolver and loader boundaries | Core and plugin documentation owners; open approval                                                 |
| 054 Lua subset and build-time Lux flow                                                      | SDK and packaging owners; open approval                                                             |
| 054 registry and index role                                                                 | Plugin-ecosystem owners; open approval                                                              |
| Shared governance, security corpus, and decisions                                           | Canonical [bitty-docs](https://github.com/bitty-terminal/bitty-docs) corpus; linked, never copied   |

Sibling corpora: [bitty-docs](https://github.com/bitty-terminal/bitty-docs)
(shared governance),
[bitty-terminal-docs](https://github.com/bitty-terminal/bitty-terminal-docs)
(terminal platform), and
[bitty-ai-docs](https://github.com/bitty-terminal/bitty-ai-docs) (AI core).

## 5. Open items and acceptance path

- Owner approval of the manager and Lux split, artifact format, manifest
  fields, lockfile shapes, trust-prompt UX, and registry scope (054).
- Contract schemas and version compatibility, provider selection and
  substitution, dependency declarations, async, cancellation, and streaming
  semantics, and the framework-versus-SDK boundary where existing
  specifications do not already decide them (053).
- The AI and Wheel owner-pending rows in section 4. The archive marked records
  053 and 054 Captured under the 2026-09-18 owner directive, but this corpus
  does not assert owner-verified capture of the model, tool, and agent
  conclusions those rows route elsewhere.
- This page asserts no capture claim or destination link beyond the plugin-side
  distillation recorded here.
