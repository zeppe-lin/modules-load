RATIONALE: modules-load(8)
==========================

Purpose
-------

`modules-load(8)` provides a deterministic, declarative mechanism for
loading Linux kernel modules during early boot.  It replaces ad-hoc
imperative scripting with a stable configuration model.

The goal is not feature expansion.  The goal is reduction of mutation
pressure on core boot scripts and improvement of configuration
legibility.

Design Principles
-----------------

**1. Separation of Policy and Mechanism**

Module selection (policy) is expressed as static configuration files:

```
/lib/modules-load.d/
/run/modules-load.d/
/etc/modules-load.d/
```

Module loading (mechanism) is implemented once in `modules-load(8)`,
which invokes modprobe(8).

`rc.modules` remains an orchestration point and does not embed
directory traversal, precedence logic, or parsing rules.

This keeps boot scripts thin and stable.

**2. Declarative State**

Each configuration file contains a plain list of module names.
No shell constructs, conditionals, or execution logic are permitted.

The intended module state can be determined by inspection of
configuration files alone.

This reduces cognitive re-entry cost and avoids hidden behavior.

**3. Explicit Precedence Model**

Configuration follows a strict, documented order:

```
/etc   (administrator)
/run   (volatile/runtime)
/lib   (vendor defaults)
```

Higher-precedence directories shadow lower-precedence entries.

This hierarchy defines authority clearly:

```
Vendor < Runtime < Administrator
```

Override behavior does not require editing packaged files.
Disabling a vendor module can be achieved without uninstalling its
package.

**4. Deterministic Execution**

`modules-load(8)`:

- Has minimal dependencies (requires **modprobe** only).
- Does not depend on `/usr` being mounted.
- Is idempotent.
- Provides dry-run (`-n`) and verbose (`-v`) modes for inspection.

The dry-run mode exists to make boot-time module state auditable
without mutating kernel state.

**5. Scope Limitation**

`modules-load(8)` does not attempt:

- Automatic hardware detection
- Dependency resolution beyond modprobe semantics
- Runtime module management
- Event-driven behavior

It executes once during boot and exits.

This utility is not a daemon and does not introduce persistent state.

Relationship to rc.modules
--------------------------

`rc.modules` invokes `modules-load(8)` if present.

Manual `modprobe` calls remain possible for exceptional hardware or
debugging scenarios.  However, declarative configuration under
`modules-load.d` is the canonical path.

This establishes a single predictable mechanism while preserving
escape hatches for unusual cases.

Maintenance Considerations
--------------------------

The introduction of `modules-load(8)`:

- Moves precedence and parsing logic out of `rc.modules`.
- Prevents repeated reinvention of module-loading logic.
- Reduces the likelihood of local modifications to core boot scripts.
- Encodes configuration hierarchy explicitly rather than implicitly.

The additional code surface (~100 lines of POSIX shell) is intentional
infrastructure.  It is small, self-contained, and expected to remain
stable.

The net effect is a reduction in long-term boot-script complexity and
a decrease in configuration ambiguity.

Conclusion
----------

`modules-load(8)` formalizes kernel module loading as a declarative,
inspectable, and precedence-aware mechanism.

It exists to increase legibility and reduce mutation in core boot
scripts, not to expand system scope.
