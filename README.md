# GDB

Notes on using gdb's terminal app.
[Online docs](https://sourceware.org/gdb/current/onlinedocs/gdb/)

## Compilation

Compile the program with debug symbols using one of:

* `-gN`
  - debug symbols
  - N is 0-3, defaults to 2
* `-ggdbN`
  - gdb-specific debug symbols
  - N is 0-3, defaults to 2

Optimize using:

* `-Og`

for the most debug-friendly optmization.

## Setup

```sh
$ gdb
(gdb) file <path/to/file>

# (Re)start the program
(gdb) run <args>
(gdb) r <args>

# Place a temporary breakpoint at main, then (re)start the program
(gdb) start
```

Program arguments are stored and reused the next time the program is run.
Alter them using:

```sh
(gdb) set args <args>
# Clear arguments (set to empty)
(gdb) set args
```

[Docs: files](https://sourceware.org/gdb/current/onlinedocs/gdb/Files.html)
[Docs: run](https://sourceware.org/gdb/current/onlinedocs/gdb/Starting.html)

## Inspect Variable

```sh
(gdb) print <variable>
(gdb) p <var>
```

To resolve pointers:

```sh
(gdb) p *<var>
```

For arrays, specify the number of "repetitions" using `@N`:

```sh
# N is 1+ (0 is pointless)
# TODO does this need *
(gdb) p *argv@N
(gdb) p *argv@argc
```

Alter the print format using `/` and a "format letter":

```sh
(gdb) p/? <var>
# For example, hex
(gdb) p/x <pointer>
```

[Docs](https://sourceware.org/gdb/current/onlinedocs/gdb/Data.html)

## Location

```sh
# Current location, single line
(gdb) where
```

Show source:

```sh
# Show lines centered around current location
(gdb) list
# Repeated invocation shows the next lines!
(gdb) list

# Show next lines
(gdb) list +
# Show previous lines
(gdb) list -
```

TODO Learn about location spec

[Docs: location](https://sourceware.org/gdb/current/onlinedocs/gdb/Specify-Location.html)

## Stepping

Basic stepping:

```sh
# Next source line, current stack frame (step over)
(gdb) next
(gdb) n

# Next source line, any stack frame (step into)
(gdb) step
(gdb) s
```

Continuing:

```sh
# Resume execution
(gdb) continue
(gdb) c

# Finish executing a function and return to the calling frame
(gdb) finish

# **Cancel** a function
(gdb) return
(gdb) return <value>

# Run until past the current line (useful in loops)
(gdb) until
(gdb) u

# Run until the given location is reached OR the stack frame returns
# TODO learn to specify a function
(gdb) until <location>

# TODO learn more about this
(gdb) advance
(gdb) adv
# Can be used to exit stack frames
(gdb) adv +1
```

Instruction-level stepping:

```sh
(gdb) stepi
(gdb) nexti
```

[Docs](https://sourceware.org/gdb/current/onlinedocs/gdb/Continuing-and-Stepping.html)
[Docs: return](https://sourceware.org/gdb/current/onlinedocs/gdb/Returning.html)

## Breakpoints

gdb places breakpoints in the *current file*.
If program execution is paused (especially a multithreaded application),
note the current file before placing a breakpoint!

```sh
(gdb) break
(gdb) break <line>
(gdb) break <function>
(gdb) break <file:line>
(gdb) break <file:function>
```

To resume execution after a breakpoint:

```sh
(gdb) continue
(gdb) c
```

Place a one-time breakpoint:

```sh
(gdb) tbreak
```

Perform a custom operation when a breakpoint hits:

```sh
(gdb) command <breakpoint>
# do useful stuff
end
```

Breakpoints can be saved to and loaded from a file:

```sh
# TODO Try this!
(gdb) save breakpoints <filename>
(gdb) source <filename>
```

[Docs](https://sourceware.org/gdb/current/onlinedocs/gdb/Breakpoints.html)

### Breakpoint Options

```sh
(gdb) set breakpoint pending on
```

## Watchpoints

A special kind of breakpoint: pause execution whenever a value changes.

```sh
(gdb) watch <var>
# TODO try this: watch a memory location
# (use if local variable goes out of scope but memory is still valid)
(gdb) watch -location <var>
(gdb) watch -l <var>
```

[Docs](https://sourceware.org/gdb/current/onlinedocs/gdb/Set-Watchpoints.html)

## Signals

Signals have three effects in gdb:

* Stop
  - Pause program execution and drop to the gdb console
* Print
  - Print a message to the gdb console
* Pass
  - Automatically pass the signal to the program
  - If Stop is enabled, the signal is delivered when the program resumes

Inspect and modify behavior using:

```sh
(gdb) info signals [<signal>]
(gdb) handle <signal> <behavior>
```

[Docs](https://sourceware.org/gdb/current/onlinedocs/gdb/Signals.html)

## Backtraces

```sh
(gdb) backtrace
(gdb) bt
```

[Docs](https://sourceware.org/gdb/current/onlinedocs/gdb/Backtrace.html)

### Backtrace Options

```sh
(gdb) set print frame-arguments none
```

## Frames

Select a stack frame:

```sh
(gdb) frame <frame>
(gdb) f <frame>
# Move up/down the stack
(gdb) up <N>
(gdb) down <N>
```

This is useful in conjunction with a backtrace:

```sh
(gdb) backtrace
#0 ...
#1 ...
#2 ...
(gdb) frame N

# Execute until... the given stack frame?
(gdb) advance +1
```

[Docs](https://sourceware.org/gdb/current/onlinedocs/gdb/Selection.html)

## Dynamic Printf

Automatically create a breakpoint, print an expression, and resume execution:

```sh
(gdb) dprintf <location>, "Interesting message"
(gdb) dprintf <location>, "%d %s", some_int, some_string
```

[Docs](https://sourceware.org/gdb/current/onlinedocs/gdb/Dynamic-Printf.html)

## Info

```sh
(gdb) info args
(gdb) info program
```

## Reverse Debugging

Example:

```sh
(gdb) record
(gdb) step
(gdb) reverse-step
(gdb) next
(gdb) reverse-next
(gdb) record stop
```

Commands:
```sh
(gdb) reverse-next
(gdb) reverse-step
(gdb) reverse-finish
(gdb) reverse-continue
(gdb) set exec-direction reverse
(gdb) set exec-direction forward
```

TODO: this can cause `Ctrl-c` to have no effect.
It seems to recover after executing a "reverse" instruction?
Not sure what is happening. Is it recording signals? Does execting a reverse instruction pause recording?

NOTE: this has issue with IO?
NOTE: this fails on system calls (`printf`)?

[Docs: record](https://sourceware.org/gdb/current/onlinedocs/gdb/Process-Record-and-Replay.html)
[Docs: reverse](https://sourceware.org/gdb/current/onlinedocs/gdb/Reverse-Execution.html)

## Helpful Hints

gdb offers a powerful reverse search through the command history: `Ctrl-r`

```sh
(gdb) help <>
(gdb) info <>
(gdb) show <>
```

Exit gdb using:

```sh
(gdb) quit
(gdb) <Ctrl-d>
```

## Resources

### Videos

* [CppCon: Greg Law, 15 minutes](https://www.youtube.com/watch?v=PorfLSr3DDI)
* [CppCon: Greg Law, 1](https://www.youtube.com/watch?v=-n9Fkq1e6sg)
* [CppCon: Greg Law, 2](https://www.youtube.com/watch?v=V1t6faOKjuQ)
* [Debugging Embedded Devices](https://www.youtube.com/watch?v=FnfuxDVFcWE)
