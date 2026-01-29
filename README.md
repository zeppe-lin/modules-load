OVERVIEW
========

`modules-load` is a lightweight, boot-safe utility for orchestrating
kernel module loading via declarative configuration files.

Designed for early-boot environments, it processes configuration
artifacts from standard `modules-load.d` directories and invokes
`modprobe(8)` to reconcile system state.

The utility implements a strict directory precedence and shadowing
model:

1. /etc/modules-load.d/ (Local Administration)
2. /run/modules-load.d/ (Runtime/Volatile)
3. /lib/modules-load.d/ (Vendor/Package Defaults)

Files in higher-priority directories shadow (mask) files with the same
name in lower-priority directories.


FORMAT
======

- Configuration files must use the `.conf` extension.
- Format: One bare module name per line.
- Comments: Lines starting with `#` and empty lines are ignored.
- Policy: Module parameters should be defined in `modprobe.d(5)`.


RATIONALE
=========

For architectural background on the transition from imperative
scripting to declarative orchestration, see
[RATIONALE.md](RATIONALE.md).


REQUIREMENTS
============

Build-time
----------
  * POSIX `sh(1p)`, `make(1p)` and "mandatory utilities"
  * `scdoc(1)` to build manual pages

Runtime
-------
  * POSIX `sh(1p)` and "mandatory utilities"
  * `modprobe(8)` provided by `kmod`


INSTALLATION
============

To build and install:

    make install

Configuration parameters, including installation paths, are defined in
`config.mk`.


DOCUMENTATION
=============

Manual pages are provided in the `/man` directory and installed under
the system manual hierarchy.

See `modules-load(8)` and `modules-load.d(5)` for full usage and
configuration details.


LICENSE
=======

`modules-load` is licensed under the
[GNU General Public License v2 or later](https://gnu.org/licenses/gpl.html).

See `COPYING` for license terms and `COPYRIGHT` for notices.
