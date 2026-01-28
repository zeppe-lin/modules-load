OVERVIEW
========

`modules-load` is a small, boot-safe utility for loading kernel
modules from configuration files.

It is intended for use early at boot, where it reads configuration
files from the `modules-load.d` directories and loads the listed
kernel modules using `modprobe(8)`.

Configuration files follow a simple, declarative format:

- Files must end with `.conf`
- Each line contains one module name.
- Empty lines and lines beginning with `#` are ignored.
- Only bare module names are accepted.
  Module parameters must be set in `modprobe.d(5)`.

The directory precedence and shadowing behavior mirrors `modprobe.d`,
allowing local administrator overrides and vendor defaults to coexist
predictably.


RATIONALE
=========

For background on why `modules-load` replaces manual `modprobe` calls
in `rc.modules`, see [RATIONALE.md](RATIONALE.md).


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
