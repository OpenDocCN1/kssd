
# win32asm 第三十课：win32 调试 API 第三部分

> 原文：[`www.pediy.com/kssd/pediy10/64101.html`](https://www.pediy.com/kssd/pediy10/64101.html)

## Tutorial 30: Win32 Debug API part 3

第三十课：win32 调试 API 第三部分

---

In this tutorial, we continue the exploration of win32 debug api. Specifically, we will learn how to trace the debuggee.

Download the example.

在这一课中，我们继续探索 win32 调试 api。特别的，我们将学习如何跟踪被调试进程。

### Theory

If you have used a debugger before, you would be familiar with tracing. When you "trace" a program, the program stops after executing each instruction, giving you the chance to examine the values of registers/memory. Single-stepping is the official name of tracing.

The single-step feature is provided by the CPU itself. The 8th bit of the flag register is called trap flag. If this flag(bit) is set, the CPU executes in single-step mode. The CPU will generate a debug exception after each instruction. After the debug exception is generated, the trap flag is cleared automatically.

如果你曾经使用过调试器，你将会更加熟悉跟踪。当你跟踪一个程序时，在执行了每一条指令后程序都会停止运行，以便给你一个机会，让你能分析寄存器和内存。单步执行官方命名为跟踪。单步执行的特征由 CPU 自己提供。标志寄存器的第 8 位叫做陷阱标志。如果该位被设置，CPU 的运行就是单步执行模式。CPU 将在每条指令执行后产生一个调试异常。在调试异常产生之后，陷阱标志被自动清除。

We can also single-step the debuggee, using win32 debug api. The steps are as follows:

使用 win32 调试 API，我们同样能单步执行被调试进程，步骤如下：

1.  Call `GetThreadContext`, specifying `CONTEXT_CONTROL` in `ContextFlags`, to obtain the value of the flag register.

调用 `GetThreadContext`，在 `ContextFlags` 中指定 `CONTEXT_CONTROL` 标志，来获取标志寄存器的值。

2.  Set the trap bit in `regFlag` member of the `CONTEXT` structure

在 `CONTEXT` 结构的 `regFlag` 成员中设置陷阱位。

3.  call `SetThreadContext`

调用 `SetThreadContext`

4.  Wait for the debug events as usual. The debuggee will execute in single-step mode. After it executes each instruction, we will get `EXCEPTION_DEBUG_EVENT` with `EXCEPTION_SINGLE_STEP` value in `u.Exception.pExceptionRecord.ExceptionCode`

像通常一样等待调试事件。被调试进程将运行于单步执行模式。在它执行一条指令之后，`u.Exception.pExceptionRecord.ExceptionCode` 中得到的值为 `EXCEPTION_SINGLE_STEP` 和 `EXCEPTION_DEBUG_EVENT`

5.  If you need to trace the next instruction, you need to set the trap bit again.

如果你需要单步执行下一条指令，你需要再次的设置陷阱标志位。

### Example

例子：

```asm
.386
.model flat,stdcall
option casemap:none

include \masm32\include\windows.inc
include \masm32\include\kernel32.inc
include \masm32\include\comdlg32.inc
include \masm32\include\user32.inc

includelib \masm32\lib\kernel32.lib
includelib \masm32\lib\comdlg32.lib
includelib \masm32\lib\user32.lib

.data
AppName db "Win32 Debug Example no.4",0
ofn OPENFILENAME <>
FilterString db "Executable Files",0,"*.exe",0
             db "All Files",0,"*.*",0,0
ExitProc db "The debuggee exits",0Dh,0Ah
         db "Total Instructions executed : %lu",0
TotalInstruction dd 0

.data?
buffer db 512 dup(?)
startinfo STARTUPINFO <>
pi PROCESS_INFORMATION <>
DBEvent DEBUG_EVENT <>
context CONTEXT <>

.code
start:
mov ofn.lStructSize,SIZEOF ofn
mov ofn.lpstrFilter, OFFSET FilterString
mov ofn.lpstrFile, OFFSET buffer
mov ofn.nMaxFile,512
mov ofn.Flags, OFN_FILEMUSTEXIST or OFN_PATHMUSTEXIST or OFN_LONGNAMES or OFN_EXPLORER or OFN_HIDEREADONLY

invoke GetOpenFileName, ADDR ofn

.if eax==TRUE
    invoke GetStartupInfo,addr startinfo
    invoke CreateProcess, addr buffer, NULL, NULL, NULL, FALSE, DEBUG_PROCESS+ DEBUG_ONLY_THIS_PROCESS, NULL, NULL, addr startinfo, addr pi
    .while TRUE
       invoke WaitForDebugEvent, addr DBEvent, INFINITE
       .if DBEvent.dwDebugEventCode==EXIT_PROCESS_DEBUG_EVENT
          invoke wsprintf, addr buffer, addr ExitProc, TotalInstruction
          invoke MessageBox, 0, addr buffer, addr AppName, MB_OK+MB_ICONINFORMATION
          .break
       .elseif DBEvent.dwDebugEventCode==EXCEPTION_DEBUG_EVENT
          .if DBEvent.u.Exception.pExceptionRecord.ExceptionCode==EXCEPTION_BREAKPOINT
             mov context.ContextFlags, CONTEXT_CONTROL
             invoke GetThreadContext, pi.hThread, addr context
             or context.regFlag,100h
             invoke SetThreadContext,pi.hThread, addr context
             invoke ContinueDebugEvent, DBEvent.dwProcessId, DBEvent.dwThreadId, DBG_CONTINUE
             .continue
          .elseif DBEvent.u.Exception.pExceptionRecord.ExceptionCode==EXCEPTION_SINGLE_STEP
             inc TotalInstruction
             invoke GetThreadContext,pi.hThread,addr context
             or context.regFlag,100h
             invoke SetThreadContext,pi.hThread, addr context
             invoke ContinueDebugEvent, DBEvent.dwProcessId, DBEvent.dwThreadId,DBG_CONTINUE
             .continue
          .endif
       .endif
       invoke ContinueDebugEvent, DBEvent.dwProcessId, DBEvent.dwThreadId, DBG_EXCEPTION_NOT_HANDLED
    .endw
.endif

invoke CloseHandle,pi.hProcess
invoke CloseHandle,pi.hThread
invoke ExitProcess, 0
end start
```

### Analysis

分析：

The program shows the openfile dialog box. When the user chooses an executable file, it executes the program in single-step mode, couting the number of instructions executed until the debuggee exits.

程序显示了一个打开文件对话框。当用户选择一个可执行文件后，它以单步执行模式来运行这个程序，并记录被执行的指令数，直到被调试进程退出。

```asm
.elseif DBEvent.dwDebugEventCode==EXCEPTION_DEBUG_EVENT
   .if DBEvent.u.Exception.pExceptionRecord.ExceptionCode==EXCEPTION_BREAKPOINT
```

We take this opportunity to set the debuggee into single-step mode. Remember that Windows sends an `EXCEPTION_BREAKPOINT` just before it executes the first instruction of the debuggee.

我们利用这个机会来设置被调试进程为单步跟踪模式。注意：windows 仅在它执行被调试进程的第一条指令之前发送一 `EXCEPTION_BREAKPOINT` 消息。

```asm
mov context.ContextFlags, CONTEXT_CONTROL
invoke GetThreadContext, pi.hThread, addr context
```

We call `GetThreadContext` to fill the `CONTEXT` structure with the current values in the registers of the debuggee. More specifically, we need the current value of the flag register.

我们调用 `GetThreadContext` 函数，用被调试进程使用的当前寄存器的值来填充 `CONTEXT` 结构体。更准确的说，我们需要标志寄存器的当前值。

```asm
or context.regFlag,100h
```

We set the trap bit (8th bit) in the flag register image.

我们设置标志寄存器映像的陷阱标志（第八位）

```asm
invoke SetThreadContext,pi.hThread, addr context
invoke ContinueDebugEvent, DBEvent.dwProcessId, DBEvent.dwThreadId, DBG_CONTINUE
.continue
```

Then we call `SetThreadContext` to overwrite the values in the `CONTEXT` structure with the new one(s) and call `ContinueDebugEvent` with `DBG_CONTINUE` flag to resume the debuggee.

然后我们调用 `SetThreadContext` 函数重写在 `CONTEXT` 的值并用 `DBG_CONTINUE` 标识调用 `ContinueDebugEvent` 函数来恢复被调试进程的执行。

```asm
.elseif DBEvent.u.Exception.pExceptionRecord.ExceptionCode==EXCEPTION_SINGLE_STEP
inc TotalInstruction
```

When an instruction is executed in the debuggee, we receive an `EXCEPTION_DEBUG_EVENT`. We must examine the value of `u.Exception.pExceptionRecord.ExceptionCode`. If the value is `EXCEPTION_SINGLE_STEP`, then this debug event is generated because of the single-step mode. In this case, we can increment the variable `TotalInstruction` by one because we know that exactly one instruction was executed in the debuggee.

当被调试进程中的第一条指令被执行时，我们收到一 `EXCEPTION_DEBUG_EVENT` 调试事件，我们必须检查 `u.Exception.pExceptionRecord.ExceptionCode` 的值。如果这个值为 `EXCEPTION_SINGLE_STEP`，那么这个调试事件是由于单步执行模式而产生的。在这种情况下，我们将 `TotalInstruction` 变量的值增一，因为我们知道在调试进程中确实有一条指令被执行。

```asm
invoke GetThreadContext,pi.hThread,addr context
or context.regFlag,100h
invoke SetThreadContext,pi.hThread, addr context
invoke ContinueDebugEvent, DBEvent.dwProcessId, DBEvent.dwThreadId,DBG_CONTINUE
.continue
```
