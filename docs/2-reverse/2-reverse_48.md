

# ollydbg 自动附加

> 原文：[`www.pediy.com/kssd/pediy10/67852.html`](https://www.pediy.com/kssd/pediy10/67852.html)

代码:

```
@set ollydbg=c:\\olly\\ollydbg.exe
@set victim=fuckee.exe
@tasklist /fi "imagename eq %victim%" /fo "list" | find "PID" > autoattach.lst
@for /f "tokens=2" %%i in (autoattach.lst) do echo %%i > autoattach.pid
@for /f %%i in (autoattach.pid) do start %ollydbg% -p %%i
@del autoattach.lst
@del autoattach.pid

```

对于有些程序需要加一个 sysrun 之类的给 ollydbg 提权，也可能是插件 anti-anti 的处理有问题