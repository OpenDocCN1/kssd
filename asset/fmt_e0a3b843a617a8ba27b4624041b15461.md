
# 一个另类的 loader.

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-7.htm`](https://www.pediy.com/kssd/pediy09/pediy09-7.htm)

> 以前一直在潜水，从论坛上学到了很多东西，也没有为论坛做个贡献。看到大侠们都是用 `asm`、`c++`、`delphi` 写的 `loader`。

> 正巧在学习 `python`，看到有一个 `debug` 框架，于是用 `python` 写了个 `loader.py`。软件破解过程比较简单，就不献丑了。

> 下面是 `loader.py` 源码。

```python
#!/usr/bin/env python

#--*-- coding =utf-8 --*--

"""
Author: <nmweizi@gmail.com>, 2007.1.6
crack xxxxx 11.0.2
xxxxx must install at C:\\Program Files\\SlickEdit\\win\\vs.exe
(bp_address,mem_address,s_value,d_value,times,delay,begintime)

Usage:
    python loader.py
"""

import pydbg, time
import os, sys


def busca_pid(dbg, nombre):
    for pid, proc in dbg.enumerate_processes():
        if proc.lower() == nombre.lower():
            return pid
    return -1


def modificaDados(dbg):
    global bp_address, mem_address, s_value, d_value, times, delay, begintime
    if dbg.read_process_memory(mem_address, length=1) == s_value and \
       (time.clock() - begintime) >= delay:
        times = times - 1
        if times == 0:
            dbg.write_process_memory(mem_address, d_value, length=1)
            print 'Patched!'

            #TODO if use bp_set(bp_address) int 3 then del # at this line
            #dbg.bp_del(bp_address)
            dbg.bp_del_hw(bp_address)
            print "Remove breakpoint %s ok!" % hex(bp_address).upper()
    return pydbg.DBG_CONTINUE


def handler_breakpoint(dbg):
    if dbg.first_breakpoint:
        dbg.hide_debugger()

        #print "first_breakpoint"
    if dbg.exception_handler_single_step:
        modificaDados(dbg)
    return pydbg.DBG_CONTINUE


def handler_event(dbg):
    if dbg.event_handler_create_process:
        dbg.bp_set_hw(bp_address, 2, pydbg.HW_EXECUTE) #not sucessed ,why ?
        #print 'CREATE_PROCESS_DEBUG_EVENT'
    return pydbg.DBG_CONTINUE


if __name__ == '__main__':
    global dbg, bp_address, mem_address, s_value, d_value, times, delay, begintime

    bp_address, mem_address, s_value, d_value, times, delay, begintime = \
                (0x406dba, 0x406dbc, "\\x66", "\\x67", 1, 0, time.clock())
    dbg = pydbg.pydbg()
    path = "C:\\Program Files\\SlickEdit\\win\\vs.exe"
    cmdline = ""
    if not os.path.exists(path):
        print "File '%s' not exist!" % path
        sys.exit(1)

    dbg.set_callback(pydbg.EXCEPTION_BREAKPOINT, handler_breakpoint)

    #TODO if use bp_set(bp_address) int 3 then # this line
    dbg.set_callback(pydbg.CREATE_PROCESS_DEBUG_EVENT, handler_event)

    dbg.load(path, cmdline)

    #TODO if use bp_set(bp_address) int 3 then del # at this line
    #dbg.bp_set(bp_address)     #int 3

    #winsock_recv     = dbg.func_resolve("ws2_32",  "recv")
    #winsock_recvfrom = dbg.func_resolve("ws2_32",  "recvfrom")
    #dbg.bp_set(winsock_recvfrom)
    #dbg.set_callback(pydbg.USER_CALLBACK_DEBUG_EVENT, modificaDados)
    #dbg.set_callback(pydbg.EXCEPTION_SINGLE_STEP, modificaDados)

    dbg.run()
    #dbg.debug_event_loop()
```
