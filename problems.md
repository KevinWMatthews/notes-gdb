# Problems to Solve

## "No source file named"

Error when setting a breakpoint:

```
No source file named
```

Reload and run the program.

## Reverse Debugging

Record:

if started using `record`, then `Ctrl-c` stops working!
It seems to recover if I execute one `rn` command, at least temporarily.

Crashing on missing instruction in system calls [bug report](https://bugzilla.redhat.com/show_bug.cgi?id=1450992). avx2 stuff?

```
107       return __printf_chk (__USE_FORTIFY_LEVEL - 1, __fmt, __va_arg_pack ());
(gdb) n
Process record does not support instruction 0xc5 at address 0x7ffff7e332a6.
Process record: failed to record execution log.

Program stopped.
__strchrnul_avx2 () at ../sysdeps/x86_64/multiarch/strchr-avx2.S:47
47      ../sysdeps/x86_64/multiarch/strchr-avx2.S: No such file or directory.
```

## Reloading

Error when changing the current symbol table:

```
warning: Probes-based dynamic linker interface failed.
Reverting to original interface.
```

This occurs when modifying and recompiling the executable (as opposed to a shared library).

GDB seems to choke when auto-reloading "position independent executables".
Specifically, it seems to have trouble reloading [Static Probe Points](https://sourceware.org/gdb/current/onlinedocs/gdb/Static-Probe-Points.html#Static-Probe-Points) for ld?
Run: `info probes` before and after; most probes are lost after the error.

One solution is to manually reload the file after every recompilation.

Another solution is to *link* using `-no-pie`.
This seems to fix auto-reload but *breaks* manual reloading using `file` (!!), at least in the gdb view; the program output is still correct.

* [SO explanation](https://stackoverflow.com/a/63207237/8807809)
* [SO: pie in gcc and ld](https://stackoverflow.com/questions/2463150/what-is-the-fpie-option-for-position-independent-executables-in-gcc-and-ld)
* [GDB source: error message](https://github.com/bminor/binutils-gdb/blob/4258df85f13dbaf9a1cb5b523bf2b9d390b25e69/gdb/solib-svr4.c#L2141)
* [GDB source: related?](https://github.com/bminor/binutils-gdb/blob/4258df85f13dbaf9a1cb5b523bf2b9d390b25e69/gdb/solib-svr4.c#L125)
* [GDB source: note on probes](https://github.com/bminor/binutils-gdb/blob/4258df85f13dbaf9a1cb5b523bf2b9d390b25e69/gdb/testsuite/gdb.base/solib-probes-nosharedlibrary.exp#L42)
* [Static Probe Points](https://sourceware.org/gdb/current/onlinedocs/gdb/Static-Probe-Points.html#Static-Probe-Points)
* [this is not my issue?](https://dustri.org/b/solving-warning-probes-based-dynamic-linker-interface-failed-in-gdb.html)
* [blog post](http://tromey.com/blog/?p=806)


## Reload the current file

Easy way to reload the current file. Someone on stack overflow had a python command to do it (roughly):

```
define make
    shell make
    python gdb.execute("file " + gdb.current_progspace().filename)
    # clear cache
    directory
end
```
but I'm not sure how to get this to give feedback/print to the console.
