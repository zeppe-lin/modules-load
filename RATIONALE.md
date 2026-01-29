# RATIONALE: Declarative Kernel Module Orchestration

### The Problem: Imperative Fragmentation

In the traditional CRUX model, `/etc/rc.modules` is a monolithic shell
script.  This creates three primary failure points for the system
administrator:

1. **Package-Script Coupling:** When a package (e.g., `fuse3`,
   `wireguard-tools`, `v4l2loopback`) requires a kernel module, it
   cannot fulfill its own dependency.  It must instead "instruct" the
   user to manually edit a core system script.
   This is a **weak link** in the deployment chain.
2. **Upgrade Fragility:** Manual edits to `/etc/rc.modules` are
   difficult to track.  During system upgrades or `prt-get` syncs,
   local logic can be clobberred or bypassed, leading to silent
   boot-time failures.
3. **Opaque State:** To determine which modules are intended to load
   at boot, an admin must parse shell logic (loops, conditionals, and
   variables) rather than reading a static state.

---

### The Solution: `modules-load(8)`

Zeppe-Lin moves module management from **Imperative Scripting** (how
to load) to **Declarative Policy** (what to load).

#### 1. Atomic Package Metadata

Packages now "own" their module requirements.  A package provides
`/lib/modules-load.d/<name>.conf`.

* **Result:** Installing the package automatically prepares the module
  for the next boot.  Removing the package automatically cleans up the
  requirement.  No manual script editing is required.

#### 2. The Shadowing Contract

Zeppe-Lin implements a strict directory precedence modeled after
`modprobe.d(5)`:

1. **/etc/modules-load.d/** (Local Administration - Highest Priority)
2. **/run/modules-load.d/** (Runtime/Volatile)
3. **/lib/modules-load.d/** (Vendor/Package Default - Lowest Priority)

This allows for **Non-Destructive Overrides**:

* To override a vendor default, copy the file to `/etc` and modify it.
* To **mask** (disable) a module entirely without uninstalling the
  package, symlink the `/etc` entry to `/dev/null`.

#### 3. Execution Integrity

The `modules-load` utility is written in pure POSIX `sh`.

* **Zero Dependencies:** It does not require `/usr` to be mounted.
* **Idempotency:** It can be run multiple times without side effects.
* **Auditability:** `modules-load -n` (dry-run) allows the admin to
  see exactly what would happen without touching the kernel state.

---

### The Hybrid Sovereignty Model

Zeppe-Lin does not deprecate `/etc/rc.modules`; it augments it.  We
maintain a "Manual Door" to ensure the administrator remains the
ultimate authority.

* **Non-Coercive Automation:** `modules-load(8)` is a utility, not a
  daemon.  It is invoked by init scripts.  An administrator can bypass
  the declarative system entirely by simply commenting out the
  modules-load call.
* **The "Safety Valve":** For hardware requiring hyper-specific
  loading sequences or complex logic, administrators can mix models:
  use `/etc/rc.modules` for "exotic" hardware and let `modules-load.d`
  handle standard package requirements.
* **Zero-Friction Migration:** A legacy `/etc/rc.modules` file remains
  fully functional.  Zeppe-Lin treats manual scripts as primary logic,
  while the declarative directories serve as a secondary, automated
  layer.

---

### Engineering Impact

By delegating to `modules-load`, the `/etc/rc.modules` script is
reduced from a maintenance liability to a simple delegation point.
This architecture respects the **KISS** principle by separating
**Policy** (the list of modules) from **Mechanism** (the `modprobe`
logic).

**In short: Zeppe-Lin replaces brittle manual edits with a
predictable, file-based orchestration layer.**

---

