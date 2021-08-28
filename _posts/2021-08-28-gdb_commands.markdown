---
layout: post
title:  "Usefull GDB commands"
date:   2021-08-28 19:20:38 +0200
categories: cpp gdb debugging
---
# Usefull commands
Some usefull commands that should get you started with debugging your program

 * Just pressing enter -- Repeat last command

 * `help`
 * `help <command-or-category>`

 * `info locals` -- Inspect your locals
 * `where` -- Shows the current stack trace
 * `list` or `l` -- List lines where execution is at

 * `continue` or `fg` or `c` -- Continue to run the program
 * `step` or `s` -- Step program, will step into functions
 * `next` or `n` -- Step programto next line, will step over functions

 * `break`, `brea`, `bre`, `br` or `b` -- Set breakpoint, use as `break <sourcefile>:<line-number>`
 * `delete`, `del`, `d` -- Delete all or some breakpoints
 * `watch` -- Set a watchpoint for an expression, stops execution if the value of an expression changes

# Set arbitrary breakpoint

```
$ gdb ./build/application/binary-to-debug
...

(gdb) break Main.cpp:31
Breakpoint 1 at 0x344e: file /home/user/repos/program/application/Main.cpp, line 31.
(gdb) run
Starting program: /home/user/repos/program/build/application/binary-to-debug

Breakpoint 1, main () at /home/user/repos/program/application/Main.cpp:31
31          std::cout << "add breakpoint here" << std::endl;
(gdb) where
#0  main () at /home/user/repos/program/application/Main.cpp:31
```

# Breaking on any exception
You can use the `catch throw` command to break on any exception.
```
$ gdb ./build/application/binary-to-debug

...

(gdb) catch throw
Catchpoint 1 (throw)
(gdb) run
Starting program: /home/user/repos/program/build/application/binary-to-debug

Catchpoint 1 (exception thrown), 0x00007ffff7e0a87e in __cxa_throw ()
   from /usr/lib/gcc/x86_64-pc-linux-gnu/10.3.0/libstdc++.so.6
(gdb) where
#0  0x00007ffff7e0a87e in __cxa_throw () from /usr/lib/gcc/x86_64-pc-linux-gnu/10.3.0/libstdc++.so.6
#1  0x00005555555574d0 in main () at /home/user/repos/program/application/Main.cpp:35
```
