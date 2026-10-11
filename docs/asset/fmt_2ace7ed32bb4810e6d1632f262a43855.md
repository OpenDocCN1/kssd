
# 让程序崩溃时自动创建 minidump

> 原文：`www.pediy.com/kssd/pediy11/106446.html` ([www.pediy.com/kssd/pediy11/106446.html](https://www.pediy.com/kssd/pediy11/106446.html))

创建 Minidump 实际上相当的简单。主要是调用了 `dbghelp.dll` 提供的 API `MiniDumpWriteDump`。

函数原型是：

```
BOOL MiniDumpWriteDump (

HANDLE hProcess,

DWORD ProcessId,

HANDLE hFile,

MINIDUMP_TYPE DumpType,

PMINIDUMP_EXCEPTION_INFORMATION ExceptionParam,

PMINIDUMP_USER_STREAM_INFORMATION UserStreamParam,

PMINIDUMP_CALLBACK_INFORMATION CallbackParam

);
```

前两个参数分别是当前程序的句柄和 ID，分别用 `GetCurrentProcess()`, `GetCurrentProcessId()` 这两个函数就可以获得了。第三个参数是文件句柄，一般我们用 `CreateFile` 来创建一个 `XXX.dmp` 的文件，而返回的句柄就是我们所需要的。`DumpType` 是 Minidump 的类型，一般来说设置为 `MiniDumpNormal` 就可以了。接下来就是最关键的一个参数 `ExceptionParam`，这是一个指向 `MINIDUMP_EXCEPTION_INFORMATION` 结构的指针。如果不填充这个结构那么我们的 minidump 就不能达到理想的效果，我会在后面谈到怎么填充这个结构。最后两个参数我认为在这里没有多大用处，我们可以简单的赋值为 `NULL`。

接下来的问题就是，我们怎么在程序崩溃的时候自动的创建 Minidump。这里我们就要用到一个 `Kernel32.dll` 提供的 API `SetUnhandledExceptionFilter`。如果读过《软件调试》等书籍的朋友肯定知道，这个函数是用来设置最后的异常处理函数的。也就是当程序产生异常的时候，如果程序自身没有处理就会调用这个函数设置的异常处理函数来处理这个异常。

有了这个函数的帮助，我们就可以在程序开始的时候设置最后的异常处理函数。并在这个函数中调用 `MiniDumpWriteDump`。这样就可以达到理想的效果。

不过我还是觉得这个不够方便，因为我每次创建一个程序，都需要调用这个函数。能不能有一个更好的方法呢？的确是有的，我们可以写一个创建 minidump 的类，然后在类的构造函数中调用 `SetUnhandledExceptionFilter` 来设置异常处理函数。最后的效果就是只需要我们加入事先写好的 `.h` 和 `.cpp` 文件，然后什么也不用做，直接编译运行就能达到理想的效果了。

再来看看我们所生成的 Minidump 的效果，将生产的 dmp 文件拖入 `windbg`，然后 `!analyze -v`，可以看到以下信息:

代码:

```
FAULTING_IP:
test!main+28 [c:\documents and settings\administrator\XXX\test\test\test.cpp @ 6]
004115d8 c70000000000    mov     dword ptr [eax],0

EXCEPTION_RECORD: ffffffff -- (.exr 0xffffffffffffffff)
ExceptionAddress: 004115d8 (test!main+0x00000028)
   ExceptionCode: c0000005 (Access violation)
ExceptionFlags: 00000000
NumberParameters: 2
   Parameter[0]: 00000001
   Parameter[1]: 00000000
Attempt to write to address 00000000

DEFAULT_BUCKET_ID: NULL_POINTER_WRITE

PROCESS_NAME: test.exe

ERROR_CODE: (NTSTATUS) 0xc0000005 - "0x%08lx"

EXCEPTION_CODE: (NTSTATUS) 0xc0000005 - "0x%08lx"

EXCEPTION_PARAMETER1: 00000001

EXCEPTION_PARAMETER2: 00000000

WRITE_ADDRESS: 00000000

FOLLOWUP_IP:
test!main+28 [c:\documents and settings\administrator\×à??\test\test\test.cpp @ 6]
004115d8 c70000000000    mov     dword ptr [eax],0

FAULTING_THREAD: 00000fd4

PRIMARY_PROBLEM_CLASS: NULL_POINTER_WRITE

BUGCHECK_STR: APPLICATION_FAULT_NULL_POINTER_WRITE

LAST_CONTROL_TRANSFER: from 00413ab8 to 004115d8

STACK_TEXT:
0012ff68 00413ab8 00000001 003929d0 00393138 test!main+0x28 [c:\documents and settings\administrator\×à??\test\test\test.cpp @ 6]
0012ffb8 004138ff 0012fff0 7c817077 80000001 test!__tmainCRTStartup+0x1a8 [f:\dd\vctools\crt_bld\self_x86\crt\src\crtexe.c @ 582]
0012ffc0 7c817077 80000001 04a3d9e4 7ffd5000 test!mainCRTStartup+0xf [f:\dd\vctools\crt_bld\self_x86\crt\src\crtexe.c @ 399]
0012fff0 00000000 004111bd 00000000 00000000 kernel32!BaseProcessStart+0x23


STACK_COMMAND: ~0s; .ecxr ; kb

FAULTING_SOURCE_CODE:
     2:
     3: void main()
     4: {
     5: int *p = 0;
>    6: *p = 0;
     7: }


SYMBOL_STACK_INDEX: 0

SYMBOL_NAME: test!main+28

FOLLOWUP_NAME: MachineOwner

MODULE_NAME: test

IMAGE_NAME: test.exe

DEBUG_FLR_IMAGE_TIMESTAMP: 4b66ae0d

FAILURE_BUCKET_ID: NULL_POINTER_WRITE_c0000005_test.exe!main

BUCKET_ID: APPLICATION_FAULT_NULL_POINTER_WRITE_test!main+28

WATSON_STAGEONE_URL: http://watson.microsoft.com/StageOne/test_exe/0_0_0_0/4b66ae0d/test_exe/0_0_0_0/4b66ae0d/c0000005/000115d8.htm?Retriage=1

Followup: MachineOwner
\---------
```

最后附上这个类的代码，至于用法非常简单，只需要把类的两个文件加入你的工程中就行了。

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | | `test.rar` |
