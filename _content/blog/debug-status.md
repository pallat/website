---
title: Debugging Go code (a status report)
date: 2010-11-02
by:
- Luuk van Dijk
tags:
- debug
- gdb
summary: What works and what doesn't when debugging Go programs with GDB.
---


When it comes to debugging, nothing beats a few strategic print statements
to inspect variables or a well-placed panic to obtain a stack trace.
However, sometimes you’re missing either the patience or the source code,
and in those cases a good debugger can be invaluable.
That's why over the past few releases we have been improving the support
in Go’s gc linker (6l,
8l) for GDB, the GNU debugger.

In the latest release (2010-11-02), the 6l and 8l linkers emit DWARF3 debugging
information when writing ELF (Linux,
FreeBSD) or Mach-O (Mac OS X) binaries.
The DWARF code is rich enough to let you do the following:

  - load a Go program in GDB version 7.x,
  - list all Go, C, and assembly source files by line (parts of the Go runtime are written in C and assembly),
  - set breakpoints by line and step through the code,
  - print stack traces and inspect stack frames, and
  - find the addresses and print the contents of most variables.

There are still some inconveniences:

  - The emitted DWARF code is unreadable by the GDB version 6.x that ships with Mac OS X.
    We would gladly accept patches to make the DWARF output compatible with
    the standard OS X GDB,
    but until that’s fixed you’ll need to download,
    build, and install GDB 7.x to use it under OS X.
    The source can be found at [http://sourceware.org/gdb/download/](http://sourceware.org/gdb/download/).
    Due to the particulars of OS X you’ll need to install the binary on a
    local file system with `chgrp procmod` and `chmod g+s`.
  - Names are qualified with a package name and,
    as GDB doesn't understand Go packages, you must reference each item by its full name.
    For example, the variable named `v` in package `main` must be referred to
    as `'main.v'`, in single quotes.
    A consequence of this is that tab completion of variable and function names does not work.
  - Lexical scoping information is somewhat obfuscated.
    If there are multiple variables of the same name,
    the nth instance will have a suffix of the form ‘#n’.
    We plan to fix this, but it will require some changes to the data exchanged
    between the compiler and linker.
  - Slice and string variables are represented as their underlying structure
    in the runtime library.
    They will look something like `{data = 0x2aaaaab3e320, len = 1, cap = 1}.` For slices,
    you must dereference the data pointer to inspect the elements.

Some things don't work yet:

  - Channel, function, interface, and map variables cannot be inspected.
  - Only Go variables are annotated with type information; the runtime's C variables are not.
  - Windows and ARM binaries do not contain DWARF debugging information and, as such, cannot be inspected with GDB.

Over the coming months we intend to address these issues,
either by changing the compiler and linker or by using the Python extensions to GDB.
In the meantime, we hope that Go programmers will benefit from having better
access to this well-known debugging tool.

P.S. The DWARF information can also be read by tools other than GDB.
For example, on Linux you can use it with the sysprof system-wide profiler.
