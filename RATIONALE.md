# RATIONALE: Declarative Kernel Module Orchestration

### The Problem: Imperative Fragmentation

Traditionally, CRUX used `/etc/rc.modules` as a single shell script.
This approach introduces several issues:

1. **Package-Script Coupling:**
   Packages that require kernel modules (e.g., `fuse`,
   `wireguard-tools`, `v4l2loopback`) cannot declare their own needs.
   It must instead "instruct" the user to manually edit a core system
   script, through the README files.
   This is a **weak link** in the deployment chain.

2. **Upgrade Fragility:**
   Manual edits to `/etc/rc.modules` are hard to track.
   During system upgrades or `prt-get` updates, local changes can be
   clobberred or bypassed, leading to silent boot-time failures.

3. **Opaque State:**
   Determining intended modules requires parsing shell logic (loops,
   conditionals, variables) rather than reading a simple list.

---

### The Solution: `modules-load(8)`

Zeppe-Lin moves from **imperative scripting** (how to load) to
**declarative policy** (what to load).

#### 1. Package Metadata

Packages provide `/lib/modules-load.d/<name>.conf` to declare required
modules.

- Installing a package prepares its modules for the next boot.
- Removing a package cleans up automatically.
- No manual script editing is needed.

#### 2. Directory Precedence

Configuration follows a strict precedence model, similar to
`modprobe.d(5)`:

1. `/etc/modules-load.d/` - local administration
2. `/run/modules-load.d/` - runtime/volatile
3. `/lib/modules-load.d/` - vendor/package defaults

This allows overrides without destructive edits:

* Copy a file to `/etc` to override defaults.
* Symlink to `/dev/null` to disable a module without uninstalling its
  package.

#### 3. Execution Integrity

* **Minimal Dependencies:** requires only `modprobe(8)`; no `/usr` mount.
* **Idempotent:** safe to run multiple times.
* **Auditable:** `modules-load -nv` shows intended actions without
  changing kernel state.

---

### Hybrid Model

`modules-load` augments rather than replaces `/etc/rc.modules`.
Administrators retain full control:

- It is a utility, not a daemon; invoked by init scripts and easily
  bypassed.
- Complex or unusual hardware can still be handled in
  `/etc/rc.modules`.
- Legacy scripts remain valid, while declarative configs provide a
  cleaner default path.

---

### Engineering Impact

Delegating to `modules-load` reduces `/etc/rc.modules` to a simple
entry point.  This separation of **policy** (module lists) and
**mechanism** (`modprobe` logic) improves clarity and reliability.

**In short: `modules-load` replaces fragile manual edits with a
predictable, file‑based configuration model.**
