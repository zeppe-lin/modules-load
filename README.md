OVERVIEW
========

`modules-load` is a lightweight, boot-safe utility for orchestrating
kernel module loading via declarative configuration files.

Designed for early-boot environments, it processes configuration files
from standard `modules-load.d` directories and invokes `modprobe(8)`
to reconcile system state.

The utility enforces a strict directory precedence and shadowing
model:

1. /etc/modules-load.d/ - Local administration
2. /run/modules-load.d/ - Runtime/volatile
3. /lib/modules-load.d/ - Vendor/package defaults

Files in higher-priority directories shadow (mask) files with the same
name in lower-priority directories.

---

FORMAT
======

- Configuration files must use the `.conf` extension.
- Each line contains a single bare module name.
- Lines beginning with `#` and empty lines are ignored.
- **Module parameters should be defined in `modprobe.d(5)`.**

---

RATIONALE
=========

For background on the transition from imperative scripting to
declarative orchestration, see [RATIONALE.md](RATIONALE.md).

---

REQUIREMENTS
============

Build-time
----------
  * POSIX `sh(1p)`, `make(1p)`, and "mandatory utilities"
  * `scdoc(1)` to generate manual pages

Runtime
-------
  * POSIX `sh(1p)` and "mandatory utilities"
  * `modprobe(8)` from `kmod`

---

INSTALLATION
============

To install:

```sh
# as root
make install
```

Configuration parameters, including installation paths, are defined in
`config.mk`.

---

QUICK START
===========

Create a configuration file in `/etc/modules-load.d/`:

```sh
# /etc/modules-load.d/example.conf
# Load sound and graphics modules at boot
snd_hda_intel
i915
```

Then run:

```sh
# as root
modules-load -v
```

This will read all `.conf` files from the precedence chain (`/etc`,
`/run`, `/lib`) and invoke `modprobe(8)` for each listed module.

---

DOCUMENTATION
=============

Manual pages are shipped in `/man` and installed under the system
manual hierarchy.

See `modules-load(8)` and `modules-load.d(5)` for usage and
configuration details.

---

LICENSE
=======

`modules-load` is licensed under the
[GNU General Public License v2 or later](https://gnu.org/licenses/gpl.html).

See `COPYING` for license terms and `COPYRIGHT` for notices.
