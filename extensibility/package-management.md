---
title: Plugin package management
description: Pre-implementation contract for plugin manifests, package sources, updates, rollback, and trust
category: extensibility
audience: plugin-author
document_type: specification
status: draft
website_publish: true
sidebar_order: 20
---

# Plugin package management

> Status: pre-implementation architecture. A package-manager experience for
> first-party and third-party plugins is accepted working direction. State
> boundaries, command names, file names, schemas, and update policy are
> candidate contracts except where the security contract is normative. The
> integrity verification chain, staged activation lifecycle and safe rollback
> semantics are accepted in
> Package Lifecycle RFC
> (OQ-021, 2026-08-27) as normative for staged activation and rollback; real
> signature verification, registry service, and key-directory contracts remain
> draft under OQ-022 and OQ-026 through OQ-029.

## Purpose and scope

Bitty should treat plugin installation as package management, not as an
incidental side effect of loading Lua. Users should be able to declare,
reproduce, inspect, update, disable, and remove official or third-party plugins
through one model shared by CLI, future developer tools, and agent interfaces.

## Candidate state model

The plugin environment has three distinct forms of state:

1. A managed manifest records the desired plugin set and version constraints.
2. A lockfile records an exact, reproducible resolution.
3. The package store contains verified installed artifacts.

Runtime behavior configuration remains in Lua. The package manager must not
rewrite arbitrary `init.lua` or user module code because conditionals,
functions, comments, imports, and formatting make that unsafe and
nondeterministic.

Supply-chain requirements are normative in the
[security overview](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/overview.md) and tracked as R-015, R-016, and
R-022 in the [security risk register](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/security/risk-register.md). Package
design must not weaken those requirements.

The boundary is:

```text
Which packages are present?       How are packages configured?
bitty-plugins.toml                init.lua and Lua modules
          |                                  |
          v                                  v
  Package resolver                         ConfigPlan
          |
          v
bitty-plugins.lock -> package store -> plugin host
```

`bitty-plugins.toml` and `bitty-plugins.lock` are current candidate names, not
final file-format commitments.

## Candidate managed manifest

```toml
# Candidate syntax. This file may be managed by `bitty plugin`.
[plugins."bitty-terminal.tabs"]
version = "^1.0"
enabled = true

[plugins."xuepoo.markdown"]
git = "https://github.com/xuepoo/bitty-markdown"
version = "^0.8"
enabled = true
```

The lockfile should record enough material to restore and audit the resolution,
including:

- plugin ID and package source;
- requested constraint and resolved version;
- immutable revision when applicable;
- content checksum and manifest hash;
- dependency resolution;
- Bitty and Plugin API compatibility.

The lockfile belongs beside the user's configuration so it can be versioned
with dotfiles. Installed plugin code belongs under the platform data directory,
not the configuration directory. See
[Lua and XDG configuration](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/configuration/lua-and-xdg.md).

Shipped slice (`bitty` #483 `95c2b23`, CTX-0150, closes `bitty` #244):
`bitty plugin` manages exactly one `bitty-plugins.toml` beside `init.lua` as a
strict bounded TOML subset (per-record plugin id, manifest hash pin, enabled
flag, and granted capabilities; version `1`). Unknown sections or keys,
duplicate keys, malformed values, over-limit files, and unknown versions fail
closed before any mutation, and the Lua `init.lua` is never rewritten. The
lockfile and non-bundled sources were not implemented in that slice; the
package store and local-path external sources are implemented in the CTX-0406
slice below, and the candidate state model above otherwise remains the target.
See the [CLI reference](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/interfaces/cli.md) for the shipped verbs and consent
behavior.

Shipped slice (`bitty` CTX-0406, 2026-09-14): `bitty plugin install <path>`
installs or updates an externally authored package from a local directory into
the XDG data store at `$XDG_DATA_HOME/bitty/plugins/`:

- `packages/<plugin-id>/<version>/bitty-plugin.toml` (verified manifest body)
  and `packages/<plugin-id>/<version>/lua/` (module root), owner-only
  (`0700` directories, `0600` files) on Unix;
- `current.json`, the atomic (write-temp-then-rename) index mapping each plugin
  id to its resolved record: source class, owner-qualified id, version,
  store-relative root, consent-bound manifest hash, module-tree content digest,
  enabled flag, and granted capabilities.

Before anything is staged, the command verifies the manifest with the same
bounded reader the runtime uses, requires the `init.lua` entry point, re-checks
the ratified module-tree bounds (4096 files, 16 MiB, 1024-byte paths, native
artifacts rejected), evaluates `compat.bitty` and `compat.plugin-api` against
the running host, and diffs the requested capabilities against the recorded
grant. Any added capability blocks until explicit consent (`--yes` or an
interactive `[y/N]` prompt that fails closed on EOF); unchanged or narrowed
sets carry forward. Installation executes zero plugin code.

Installed local-directory packages carry `local-path` provenance. The staged
record root is store-relative and immutable, so a content-digest mismatch is a
fail-closed store integrity error; an absolute canonical root keeps the
read-only development-flow semantics (re-digested on load, drift reported as
unverified). A store record can never claim `bundled` provenance, and `--safe`
creates no VM for any third-party class.

`bitty plugin list|info` include installed packages (a `source` column and JSON
`source` field); `enable`/`disable` rewrite the store index atomically;
`remove --force` deletes the record and the staged tree; re-running `install`
with a newer version updates in place and retains the previous version. The
runtime resolves the store through `current.json`, re-verifies the manifest
hash and content digest before creating a VM, and enforces the recorded grant
at activation: a record that does not cover the manifest's declared
capabilities fails closed with no VM. Reload tears generation N down before
activating N+1 with grants still bound to the manifest hash. A committed update
is picked up at the next start; the live `bitty plugin reload`/IPC trigger
surface remains open under OQ-072. Remote `git` sources, the SDK
`bitty-plugin-lint` conformance check, the draft seven-stage pipeline, and
rollback commands remain unimplemented.

## Source model

Status: **accepted direction.**

Both first-party and third-party plugins are supported. Package identity is a
stable plugin ID, not a GitHub URL. Sources should include:

- bundled packages;
- a future registry;
- Git repositories with version or revision selection;
- local paths for plugin development.

Status: **candidate cli examples.**

```sh
bitty plugin add bitty-terminal.tabs
bitty plugin add xuepoo.markdown
bitty plugin add git:https://github.com/xuepoo/bitty-markdown@v0.8.2
bitty plugin add --path ../bitty-markdown
```

A registry can arrive later. Starting with Git and local paths must not prevent
stable identity, compatibility metadata, integrity verification, or a future
registry mapping from plugin ID to source.

## Candidate source-implementation direction (Git-first, network-isolated)

Status: **candidate direction, non-normative** (user architecture note,
bitty-docs CTX-0201 / bitty-docs#288,
[DIR-016](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/index.md)).
It refines the accepted source model above without changing any accepted
contract: the source classes, the seven-stage verification pipeline and
provenance separation in the
Package Follow-up RFC, and the
package-manager/host split below stay authoritative. No implementation claim:
`bitty-package` today models sources as the `PackageSource` data enum
(registry / git / local-path / bundled) with no fetch behavior, and only
local-path install has shipped (CTX-0406 slice above).

- **Design: `PluginSource` trait.** Sources implement `resolve` / `fetch` /
  `update` behind one trait. `LocalSource` and `GitSource` come first; later
  `RegistrySource`, `HttpArchiveSource`, and OCI sources extend the same seam.
- **v1 sources: local + system `git` only.** `GitSource` shells out to the
  system `git`: `git clone --filter=blob:none --depth=1`, then fetch/checkout
  by version or commit. The lockfile records source, version, and resolved
  revision (`rev`) for reproducibility. (The user note spells the lockfile
  `bitty.lock`; the managed-manifest and lockfile names remain open per the
  open questions below.)
- **Explicitly not v1 (plugin sources).** No curl-tarball fetching for
  _plugin_ sources (TLS/proxy/retry/checksum/cache/auth matrix), no `git2` /
  libgit2 (heavier dependencies, and it loses the system gitconfig,
  SSH-agent, credential-helper, proxy, and CA handling), and no `reqwest`
  in Core. Rule: do not reimplement Git (Unix philosophy).
- **Shipped exception (component sources).** The manager-seed slice
  (bitty#1791, shipped as bitty#1870) added `bitty component install`:
  fixed-argv system `curl` fetches a hash-pinned release bundle from the
  pinned CDN host, system `tar` unpacks it, SHA-256 is verified before
  staging. This curl-tarball path is v1 for _components only_; the
  consent/egress/digest/fetch-environment policy around it is tracked in
  bitty#1905 (egress + consent), bitty#1906 (digest arbitration),
  bitty#1907 (curl floor + CA/proxy) — this document does not pre-decide
  those levels.
- **Graceful degradation.** `bitty` runs without git, curl, or network.
  `bitty plugin install` without `git` fails closed with a diagnostic that
  points at manual placement under `$XDG_DATA_HOME/bitty/plugins/`.
- **Registry hosts metadata/discovery only, never plugin binaries.** The
  candidate distribution mechanism for the registry itself is Git: clone/fetch
  the registry repository into the XDG cache, search local TOML records, and
  refresh only on an explicit `registry update`. This mirrors the CarryCtx sync
  philosophy. It is recorded as a candidate because the accepted
  Package Follow-up RFC (OQ-028)
  specifies an HTTPS index snapshot fetch; reconciling the two mechanisms
  needs an RFC amendment, and this section does not weaken that contract.
- **Later native HTTP.** When Git cannot serve a need, native HTTP runs in
  the bitty-network crates, out of process as the `bitty-net` native
  component (see [Component packages](#component-packages)), so Core stays
  network-free. (`bitty-plugin-manager` is a name from the user note; the
  current workspace has `bitty-package` only. `bitty-net` now names the
  component executable under [DIR-030](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/index.md), not a Core crate.)

## Command semantics

Status: **accepted direction.**

Plugin lifecycle should be manageable without requiring ordinary users to edit
Lua.

Status: **candidate contract.**

CLI, GUI, and agent adapters should invoke one package-manager service. They
should not implement independent file mutation or resolution logic.

The four central verbs have deliberately different semantics:

- `add`: change desired state to include a plugin, resolve it, and update the
  environment.
- `remove`: change desired state to exclude a plugin; optional purge of its
  persistent state is a separate choice.
- `update`: select newer versions allowed by constraints and create a new lock
  resolution.
- `sync`: restore the exact manifest/lock environment without opportunistically
  selecting newer versions.

Enable/disable is not install/remove. A disabled plugin remains declared,
installed, locked, and configured, but does not load. Removing a plugin should
preserve its state by default and report where it remains; an explicit purge
may remove it.

Status: **candidate command set.**

```text
P0: add, remove, list, info, enable, disable, update, sync
P1: search, outdated, pin, unpin, clean, doctor, rollback
P2: registry, audit, signature, publish, graphical management,
    background update checks
```

P0/P1/P2 are planning candidates, not an implementation roadmap approved by
this document. `add`/`remove` are preferred over duplicated
`install`/`uninstall` aliases unless user research shows a need.

## Updates, transactions, and rollback

Status: **accepted direction.**

An update must not leave a half-updated environment. The target flow is:

```text
resolve
  -> stage downloads
  -> validate manifests and dependency graph
  -> verify compatibility and integrity
  -> require review for any capability increase
  -> commit lock state
  -> atomically switch active package set
```

Installation and update execute no package code. Runtime activation is a
separate transaction; an activation failure must preserve or restore the
previous working environment. The package manager should retain enough prior
lock information to support full or per-plugin rollback.

Status: **candidate version behavior.**

- `bitty plugin update` updates all unpinned plugins within declared version
  constraints.
- major-version movement outside the constraint requires an explicit command or
  manifest edit.
- `pin` and `unpin` control whether ordinary updates may move a package.
- `outdated` distinguishes current, wanted, and latest versions.

Automatic update checks may be enabled, but automatic background upgrades
should be off by default. A terminal is a production tool; activation changes
should be explicit and recoverable.

## Package store

Status: **accepted direction.**

Package contents are installed under the platform data root. Configuration,
cache, and persistent plugin state remain separate.

One candidate layout is:

```text
$XDG_DATA_HOME/bitty/plugins/
├── packages/
│   ├── xuepoo.markdown/
│   │   ├── 0.8.1/
│   │   └── 0.9.0/
│   └── bitty-terminal.tabs/
│       └── 1.4.1/
└── current/
```

A content-addressed store may improve deduplication and atomic switching later,
but Nix-like storage is not a first-stage requirement.

## Component packages

Status: **accepted direction ([DIR-030](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/decisions/index.md))**,
including the v1 component commands below (DIR-030 refinement of
2026-10-02); the manifest `[components]` grammar is **accepted** per the
2026-10-02 amendment to the
[Plugin Manifest and Capability Grammar
Authority](../specifications/manifest-capability-authority.md#amendment-2026-10-02-accepting-components-and-networkegress)
(it is also part of the [Plugin Platform
RFC](../specifications/plugin-platform-rfc.md#accepted-manifest-schema)
accepted manifest schema); the remaining package-manager install/resolution
spelling below is candidate. The process, install, and authority model is
defined in the [Native Component Boundary](https://github.com/bitty-terminal/bitty-docs/blob/main/docs/development/native-component-boundary.md); this section records only the package-manager
consequences.

A native component is a second package class beside Lua plugin packages: an
independently installed, single-purpose executable (for example the `net`
component, executable `bitty-net`) that Core spawns on demand as a stdio
coprocess and shares across every plugin that declares it. It is not a plugin:
it carries no Lua, never loads into a plugin VM, and never receives authority
of its own; Core issues every grant.

Install layout, separate from the plugin store:

```text
$XDG_DATA_HOME/bitty/components/
└── net/
    ├── current                 # plain text: the active version
    └── 0.0.1/
        ├── bitty-component.toml
        └── bitty-net           # bitty-net.exe on Windows
```

The component descriptor `bitty-component.toml` records `name`, `version`
(semver), `protocol` (supported wire protocol range), `executable` (a bare
file name, no path separators), and `sha256` (lowercase hex digest of the
executable). The package manager verifies the descriptor and the digest when
it installs a component, and Core verifies them again before every spawn; any
mismatch fails closed with a diagnostic. Installation executes no component
code.

A plugin declares component dependencies in its manifest:

```toml
[components]
net = "^0.0.1"
```

Version requirements use semver caret matching with Cargo semantics:
`^0.0.1` admits exactly `0.0.1`, and `^0.1` admits `>=0.1.0, <0.2.0`.

A plugin that uses the `net` component to reach a specific destination also
declares the structured egress entry so Core can compute its grant (the
intersection of granted `network.connect:HOST[:PORT]` capabilities and
`[[network.egress]]` declarations, per the [Plugin Manifest and Capability
Grammar
Authority](../specifications/manifest-capability-authority.md#amendment-2026-10-02-accepting-components-and-networkegress)):

```toml
[[network.egress]]
host = "api.example.com"
ports = [443]
```

Rules:

- **No `PATH` discovery.** Components resolve only from the component root
  (or the developer-only `BITTY_COMPONENTS_DIR` override used by tests); an
  executable that happens to be on `PATH` is never used.
- **Missing component.** `bitty plugin install` resolves the plugin's
  `[components]` table; a missing or incompatible component makes the install
  fail with a diagnostic naming the component, the range, and the
  `bitty component add` / `bitty component install` commands to run.
  Automatic download of the missing component follows the installer-trust
  policy tracked in bitty#1905 (egress + consent) and bitty#1906 (digest
  arbitration) — no silent fetching outside that policy. At runtime an
  unavailable component makes the capability unavailable, never ambient.
- **Sources (v1).** Local path plus the shipped CDN seed, through the DIR-030
  component commands: `bitty component add <dir-or-executable>` (computes the
  digest and writes `bitty-component.toml` and `current`),
  `bitty component install <name>` (hash-pinned CDN bundle; see the shipped
  exception above), `bitty component list`,
  `bitty component remove <name> [<version>]`, and `bitty component clean`
  (removes versions not referenced by `current` and components no installed
  plugin requires). Registry and download sources are a follow-up.
- **No automatic cascade uninstall.** Removing the last plugin that depends on
  a component does not remove the component; the manager reports it as
  unused and removal stays an explicit user action (`remove` or `clean`).

No component command, install, resolution, or `[components]` validation is
implemented in the package manager yet. The plugin-facing request surface is
the [bitty.net candidate](../sdk/net-request-surface-candidate.md).

## Package manager versus runtime host

Status: **candidate contract.**

Separating these subsystems is a candidate architecture. The per-plugin
isolated VM and restricted-authority requirement shown for the host is a
normative P0 security baseline, not an optional part of this candidate split.

| Package manager                      | Plugin host                      |
| ------------------------------------ | -------------------------------- |
| sources and downloads                | load/unload                      |
| dependency resolution                | per-plugin isolated Lua VMs      |
| integrity verification               | capabilities                     |
| manifest and lock state              | lazy activation                  |
| package store and update transaction | lifecycle and resource ownership |

The host consumes a resolved, verified package graph. It must not perform
network resolution during ordinary startup. Runtime isolation, services, and
conflict rules are specified in [Plugin system](plugin-system.md).

## Plugin-provided CLI

Status: **accepted direction.**

Installed plugins may expose commands, but the core CLI namespace remains
stable and cannot be overridden. A collision-free route qualified by plugin
identity must remain available.

Status: **candidate CLI example.**

The current candidate spelling for that route is:

```sh
bitty x xuepoo.markdown render README.md
```

Status: **candidate contract.**

A plugin manifest may suggest a short alias:

```toml
[cli]
alias = "markdown"
```

That could make `bitty markdown ...` available. It is only an alias: conflicts
are diagnosed and users can always use the qualified plugin identity. In this
candidate grammar, that route uses `bitty x`. Built-in commands win and may
never be shadowed. Help should display extension commands in a separate section.

PATH-discovered executables are not used for native components: components
resolve only from the component root and are digest-verified before spawn (see
[Component packages](#component-packages)). Whether a separate CLI-extension
class may follow Cargo-style external executables (for example discovering
`bitty-benchmark` on `PATH` as `bitty benchmark`) stays open. If it exists, it
is a distinct class from both runtime plugins and native components, with its
own trust, distribution, and capability model, and must not be conflated with
either.

Static manifest metadata should supply command names, argument schemas, help,
completion, lazy triggers, and ownership without starting plugin Lua VMs. The
shared CLI model is described in [CLI](https://github.com/bitty-terminal/bitty-terminal-docs/blob/main/interfaces/cli.md).

## Security and trust

Status: **accepted direction.**

Third-party does not mean trusted. Lock and checksum validation are required for
installation and update. Installation must show requested sensitive
capabilities, and a capability increase blocks an update pending explicit
review. A package declaring network, process, terminal read/input, filesystem
write, protocol, or runtime-control access needs a clear decision surface.

Checksums are a baseline integrity primitive, not a complete supply-chain
strategy. Registry provenance, signatures, publisher identity, revocation,
audit, dependency health, and abandoned packages remain staged design work.

Local-path development packages need visibly different trust and reproducibility
semantics from immutable registry or Git revisions.

## Open points

- What are the final managed manifest and lockfile names and formats?
- Is one version of a plugin ID allowed in an environment, and can service
  interfaces support side-by-side dependency versions?
- What is the exact constraint grammar and prerelease/yanked-version policy?
- What static validation can occur before the atomic switch without executing
  package code, and how does the separate activation transaction roll back?
- How many prior environments are retained for rollback, and where?
- How are local-path changes represented in lock and integrity status?
- Which package sources are allowed before a registry exists?
- How are signatures, publisher identity, revocation, and audit introduced?
- Are top-level plugin aliases worth the ambiguity, or should `bitty x` be the
  only plugin command namespace?
- Should a separate CLI-extension class of external executables exist, and if
  so, should it be managed by this package manager or discovered from `PATH`?
  This question does not apply to native components, which never use `PATH`
  discovery (DIR-030).
- How are components installed from a registry once registry sources exist?
