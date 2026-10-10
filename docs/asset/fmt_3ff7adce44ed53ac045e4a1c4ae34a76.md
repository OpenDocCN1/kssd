
# 汇编代码片段

```
.text:00401528                jmp     short @loop1_done
.text:0040152A @loop2_wait:                   ; 延时等待
.text:0040152A                lodsb
.text:0040152B                or      al, al
.text:0040152D                jnz     short @loop2_wait
.text:0040152F                inc     [ebp+var_108_service_count]
.text:00401535                cmp     byte ptr [esi], 0
.text:00401538                jz      short @loop1_done
.text:0040153A                jmp     short @loop1
.text:0040153C @loop1_done:
.text:0040153C
.text:0040153C                jmp     short @ret_false
.text:0040153E @wm_default:                   ; 默认消息处理
.text:0040153E                push    [ebp+lParam]    ; lParam
.text:00401541                push    [ebp+wParam]    ; wParam
.text:00401544                push    [ebp+Msg]       ; Msg
.text:00401547                push    [ebp+hWnd]      ; hWnd
.text:0040154A                call    DefWindowProcA
.text:0040154F                pop     ecx
.text:00401550                pop     esi
.text:00401551                pop     edi
.text:00401552                leave
.text:00401553                retn    10h
.text:00401556 @ret_false:
.text:00401556                xor     eax, eax
.text:00401558                pop     ecx
.text:00401559                pop     esi
.text:0040155A                pop     edi
.text:0040155B                leave
.text:0040155C                retn    10h
.text:0040155C lpfnWndProc     endp
```

# HBQQXX.dll 说明

最后顺便提一下 `HBQQXX.dll`：

`HBQQXX.dll` 提拱 3 个输出函数：

- `ServiceRouteEx`：钩子函数，简单地将传入的参数交给 `CallNextHookEx` 处理；
- `StartServiceEx`：通过调用 `SetWindowsHookExA` 装载钩子函数 `ServiceRouteEx`
- `StopServiceEx`：通过调用 `UnhookWindowsHookEx` 来停止系统服务 `"HBKernel32"`

# 小结

本解法菜就菜在系统机制完全搞不懂，但确实能解决问题。发送消息 `iMessage=0x10` 或 `iMessage=0x11` 能够停止系统服务 `"HBKernel32"`，但不能停止设备驱动 `"HBKernel32 Driver"`。系统服务 `"HBKernel32"` 停止后，可以删除 `HBKernel32.sys` 文件，但不能删除 `HBQQXX.dll` 文件，而只能将 `HBQQXX.dll` 文件更名或移动。感觉好像是可以删除驱动程序的磁盘文件 `HBKernel32.sys`，但其内存映像不能清除，重启系统后因驱动实体 `HBKernel32.sys` 已不复存在，因而可以将其清除之。如果有办法（可能是所谓的 ring0 编程，本人完全不懂这个）停止内存中的 `HBKernel32.sys` 进程（他确实是被真正的系统进程 System 锁定了）。再次声明一下我这里只是停掉了木马 `System.exe` 进程，而不是系统的 `System` 进程。

发送 `WM_CLOSE`（`0x10`）或是 `WM_QUERYENDSESSION`（`0x11`）时，执行的代码都是

引用:
```
call    dlls__StopServiceEx
push    [ebp+hWnd]      ; hWnd
call    DestroyWindow
push    0               ; nExitCode
call    PostQuitMessage
```

这里并没有停止服务的动作，只是取消了全局鼠标 hook，因此驱动没有停止

抱歉理解能力有限还是请教一下，不太理解楼主所说的

引用: 能够停止系统服务"HBKernel32"，但不能停止设备驱动"HBKernel32 Driver"。系统服务"HBKernel32"停止后，可以删除 HBKernel32.sys 文件

木马只有一个服务，也就是"HBKernel32"，"HBKernel32 Driver"只是一个 DisplayName，这里又能停止又不能停止都看糊涂了![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心")

至于 DLL 文件不能删除很好解释，因为此时该 DLL 还被其他进程加载了，所以文件不能直接删除除非先从其他进程中全部 unload 该 dll 模块

内存中并没有 `HBKernel32.sys` 进程，只是在 `SYSTEM` 进程中有一个 `HBKernel32.sys` 模块及其创建的系统线程在执行重复 hook 以及刷新注册表键值的无限循环

这个问题的根本应该是在于对 `HBKernel32.sys` 的锁定并不是在 `System.exe` 木马进程中，所以无法理解楼主提到关闭这个木马进程 `System.exe` 就能删除 `HBKernel32.sys` 文件

楼主在顶楼就已经说过是操作系统是 2000，我太粗心了![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")

试了一下，2000 下驱动果然没有任何保护，什么都不用做 `HBKernel32.sys` 都可以直接删，瀑布汗一个。。。![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心")
