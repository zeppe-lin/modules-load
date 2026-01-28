# RATIONALE

Historically, `/etc/rc.modules` was a shell script containing explicit
`modprobe` calls.  That model has several drawbacks:

* **Manual edits:** packages had to instruct users to add lines such
  as `/sbin/modprobe fuse` to `rc.modules`.  This couples package
  behavior to local script edits and is fragile across upgrades.

* **Duplication:** many packages repeated the same instructions
  instead of shipping machine-readable configuration.

* **Mixed concerns:** persistent module policy lived alongside one-off
  boot-time workarounds, obscuring the distinction between system
  intent and local experimentation.

* **Poor auditability:** determining which modules were expected to
  load at boot required reading and reasoning about shell code.

The `modules-load(8)` utility with configuration directories addresses
these issues:

* **Declarative configuration:** packages may ship `.conf` files in
  `/lib/modules-load.d` to express required modules directly.
  For example, a `fuse` package can provide
  `/lib/modules-load.d/fuse.conf` without requiring users to edit boot
  scripts.

* **Explicit precedence:** configuration is processed from `/etc`,
  `/run`, and `/lib` in descending priority.
  Vendor defaults can be overridden or disabled by shadowing files
  with the same name.
  This mirrors the behavior of `modprobe.d(5)` and other `*.d`
  configuration systems.

* **Simple format:** one module name per line, with `#` comments.
  No arguments or embedded logic.
  This keeps configuration readable, auditable, and easy to override.

* **Centralized logic:** ordering, precedence, diagnostics, dry-run,
  and error handling are implemented in a single utility.
  `rc.modules` is reduced to delegation.

* **Improved auditability:** listing the contents of
  `/etc/modules-load.d` shows which modules are expected to load at
  boot, without inspecting shell scripts.

* **Early-boot suitability:** the implementation is POSIX `sh` and
  depends only on `modprobe`, with no reliance on `/usr` or
  non-portable behavior.

* **Reduced maintenance:** `rc.modules` no longer requires
  modification when new drivers are introduced.
  Packages and administrators express policy by adding or overriding
  configuration files.

In short, `modules-load` replaces ad-hoc scripting with declarative
and auditable configuration.
Packages can express module requirements directly, administrators can
override them predictably, and init scripts remain small and focused.
