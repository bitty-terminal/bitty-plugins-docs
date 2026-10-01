# Frequently asked questions

This document provides answers to common architectural, security, and integration
questions regarding Bitty's plugin ecosystem, sandboxing, external tool usage,
and lifecycle management.

## Security and sandboxing

### How does Bitty's plugin sandbox differ from WezTerm, Kitty, Neovim, and VSCode?

Traditional terminal emulators and extensible editors provide little or no physical
process-level security boundaries for plugins:

- **WezTerm and Neovim**: Plugins run directly inside the main host process with ambient operating system authority. Any plugin script can invoke `io.popen`, `os.execute`, load C dynamic shared libraries (`package.loadlib`), read sensitive user files (such as SSH private keys or cloud credentials), or make outbound network requests.
- **Kitty (Kittens)**: Kittens run as Python scripts. While some run out-of-process, Python provides full ambient access to the standard library (`os`, `subprocess`, `socket`), granting plugins unrestricted local authority unless sandboxed externally.
- **VSCode**: Extensions execute in an out-of-process Node.js runtime, but Node.js retains ambient access to `child_process`, `fs`, and `net`. Malicious extensions regularly compromise developer machines via npm dependency attacks.
- **Bitty**: Plugins run inside Phodopus, a pure-Rust sandbox derived from Piccolo. The runtime has zero C dependencies and enforces Zero Ambient Authority (ZAA):
  - Standard I/O libraries (`io`, `os`, `package.loadlib`) are physically omitted from the Lua environment.
  - Syscalls, raw file access, socket creation, and arbitrary process execution cannot occur because the underlying virtual machine simply lacks the capability to emit them.
  - All access to external resources requires explicit capability declarations in `plugin.toml` and runtime mediation by the host.

### How does Fuel instruction bounding prevent rogue plugins from freezing Bitty?

In traditional runtimes, an infinite loop (`while true do end`) or deeply recursive algorithm in a plugin thread can lock the event loop and freeze the UI.

- Bitty uses instruction-level **Fuel budgeting** provided by the Phodopus engine.
- Every instruction execution consumes a unit of fuel. When a plugin's allocated fuel for the current render tick is exhausted, the VM automatically yields control back to the host event loop.
- If a plugin continuously fails to complete or exceeds wall-clock execution limits, the host gracefully suspends or terminates the plugin without interrupting terminal rendering, PTY processing, or sibling plugins.

## External tools and host services

### Can plugins invoke local CLI tools like `rg`, `fd`, `git`, or `matugen`?

Yes, through **Layer 2 External Tool Invocation**:

- If a plugin requires a specialized system binary (for example, `ripgrep` for content searching, `fd` for file listing, or `matugen` for wallpaper color extraction), it declares the binary requirement in its manifest:

  ```toml
  [tools.ripgrep]
  command = "rg"
  required = true
  version = ">=14.0.0"

  [capabilities]
  required = ["process.spawn:rg"]
  ```

- At runtime, the plugin invokes the tool via the host process service:

  ```lua
  local proc = bitty.process.spawn("rg", {
    args = { "--json", "-e", query },
    cwd = workspace_path,
  })
  ```

- **Execution rules**:
  - The host executes the binary directly using argument vector passing (`execve` style). Shell interpreters (`sh -c` or `cmd.exe /c`) are prohibited to prevent command injection vulnerabilities.
  - The host enforces strict execution timeouts and cancellation tokens.
  - All spawned child processes are tracked in process groups and automatically terminated if the plugin exits or encounters a runtime error.

### Why doesn't Bitty allow plugins to embed fuzzy matchers like `nucleo` or `skim`?

Bitty follows the architectural rule: **"Reuse below, compose above"**.

- Bundling native C/Rust search libraries inside Lua plugins introduces heavy binary dependencies, inconsistent compilation across platforms, and memory safety risks.
- Instead, Bitty provides a built-in **Host Fuzzy Service** (`bitty.services:get("fuzzy")`):
  - The plugin acts as a pipeline orchestrator: it spawns a fast discovery tool (`fd` or `git ls-files`) and streams the output lines into the host fuzzy service.
  - The host executes zero-copy SIMD-accelerated scoring and filtering directly in Rust.
  - The host renders the filtered results inside a native GPU-accelerated overlay panel with consistent keybindings, theming, and accessibility.

### How do palette and theme generators like `matugen` work with Bitty?

Theme plugins do not write raw terminal escape sequences or attempt to manipulate graphics protocols directly:

- A theme generator plugin spawns `matugen` via the Layer 2 tool bridge to analyze wallpaper images and generate a structured JSON palette.
- The plugin decodes the color tokens and passes the table to Bitty's theme engine:

  ```lua
  bitty.theme.apply({
    background = palette.surface,
    foreground = palette.on_surface,
    palette = palette.colors,
  })
  ```

- The host validates color contracts and updates the terminal color map dynamically across all windows and panels.

## Trust boundaries and lifecycle

### Why does `bitty plugin add` use system `git`, but plugins cannot run `git` without permission?

This reflects the distinct separation between the **Bitty CLI** and the **Plugin Sandbox**:

- `bitty plugin add` is executed by the human developer in their interactive terminal shell. As a host CLI utility, it operates under the user's explicit interactive authority and invokes the system `git` binary (per DIR-016) to clone or inspect repositories.
- Conversely, a plugin is downloaded third-party code executing inside an automated terminal session. Allowing plugins ambient access to `git` would permit silent exfiltration of git credentials, private repositories, or SSH keys.
- Therefore, a plugin cannot spawn `git` unless the user explicitly grants the `process.spawn:git` capability during installation.

### What happens when an external tool crashes or hangs?

The host acts as a supervisor for all spawned child processes:

- Child processes are launched in isolated process groups.
- If an external tool hangs, the host enforces the declared execution timeout (default 30 seconds) and triggers a tree-kill (`SIGTERM` followed by `SIGKILL` on Unix, Job Objects on Windows).
- If the plugin itself crashes or triggers a Lua error, the host catches the error, isolates the failure to that plugin panel, and immediately cleans up all associated child processes, temporary pipes, and UI allocations without affecting the core terminal emulator.

## Related documentation

- [Extensibility FAQ](extensibility/faq.md)
- [Plugin system](extensibility/plugin-system.md)
- [Plugin package management](extensibility/package-management.md)
- [Plugin reuse and providers](packaging/plugin-reuse-and-providers.md)
- [Lua UI component model](extensibility/lua-ui-component-model-candidate.md)
- [Terminal platform documentation](https://github.com/bitty-terminal/bitty-terminal-docs)
- [AI core documentation](https://github.com/bitty-terminal/bitty-ai-docs)
