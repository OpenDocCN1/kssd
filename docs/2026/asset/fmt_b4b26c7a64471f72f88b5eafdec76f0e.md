
```
/*
* Module Name:
* antidebug_long_int3.cpp
*
* Abstract:
* Attempts to detect the presence of a debugger
* by issuing a multi-byte int 3 and inspecting
* page PTE mappings.
*
* Author:
* Nemanja (Nemi) Mulasmajic <nm@triplefault.io>
* http://triplefault.io
*/
#pragma warning(disable: 4710)
#pragma warning(push, 0)
#include <windows.h>
#include <stdio.h>
#include <Psapi.h>
#pragma warning(pop)
#pragma comment(lib, "psapi.lib")
// The size of an architecture page on x86/x64.
#define PAGE_SIZE 0x1000
// A pseudo-handle that represents the current process.
#define NtCurrentProcess() ((HANDLE)-1)
// Multi-byte int 3.
BYTE _LongInt3[] = { 0xCD, 0x03 };
int main(void)
{
int status = -1;
// We allocate 2 contiguous pages of executable virtual memory.
PBYTE Memory = (PBYTE)VirtualAlloc(NULL, (PAGE_SIZE * 2), (MEM_COMMIT | MEM_RESERVE), PAGE_EXECUTE_READWRITE);
if (!Memory)
{
fprintf(stderr, "[-] ERROR: Failed to allocate memory.\n");
goto Cleanup;
}
// The first page, "page 1", will have the multi-byte form of int 3
// embedded at the very end of the page. This is done so that the next
// byte immediately after the multi-byte int 3 will span into "page 2".
//
// e.g:
// | Page 1 | Page 2
// [.....................0xCD 0x03][............................]
PBYTE CodeLocation = &Memory[PAGE_SIZE - sizeof(_LongInt3)];
PBYTE DeadPageLocation = &Memory[PAGE_SIZE];
// Add the int 3 to the very end of page 1.
memcpy(CodeLocation, _LongInt3, sizeof(_LongInt3));
// Page 2 should never be accessed and therefore should not be
// present in our process' working set.
PSAPI_WORKING_SET_EX_INFORMATION wsi;
wsi.VirtualAddress = DeadPageLocation;
if (!QueryWorkingSetEx(NtCurrentProcess(), &wsi, sizeof(wsi)))
{
fprintf(stderr, "[-] ERROR: QueryWorkingSetEx failed with error: 0x%lx.\n", GetLastError());
goto Cleanup;
}
if (wsi.VirtualAttributes.Valid)
{
fprintf(stderr, "[-] ERROR: Page is expected to be invalid. Make sure you have not inadvertently accessed this page.\n");
goto Cleanup;
}
__try
{
// Invoke the long form of int 3.
((void(*)())CodeLocation)();
}
__except (EXCEPTION_EXECUTE_HANDLER) { }
// If a debugger caught the exception, even if it was passed to the
// application via "go unhandled", it will have, most likely, mapped
// page 2 into the application's memory space.
if (!QueryWorkingSetEx(NtCurrentProcess(), &wsi, sizeof(wsi)))
{
fprintf(stderr, "[-] ERROR: QueryWorkingSetEx failed with error: 0x%lx.\n", GetLastError());
goto Cleanup;
}
printf("[+] Debugger %s.\n", ((wsi.VirtualAttributes.Valid) ? "detected" : "not detected"));
status = 0;
Cleanup:
// Free allocated memory.
if (Memory)
{
VirtualFree(Memory, 0, MEM_FREE);
Memory = NULL;
}
// Wait for [ENTER] key press to terminate the program.
getchar();
return status;
}
```

# 通过滥用一个Windows中的糟糕假设来检测调试器

---

翻译 by 银雁冰

[**原文链接**](<https://bbs.kanxue.com/elink@2b2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4c8J5K9i4m8D9k6h3k6S2N6h3I4@1i4K6u0W2K9h3!0Q4x3V1j5J5x3o6p5%4i4K6u0r3x3o6S2Q4x3V1k6V1k6i4c8W2j5%4c8A6L8X3N6Q4x3X3c8V1k6h3u0#2k6$3N6W2M7Y4y4Q4x3X3c8T1P5g2)9J5k6r3q4T1N6i4y4A6L8X3N6Q4x3X3c8T1j5h3c8Q4x3X3g2Z5N6r3#2D9>)

这篇博客将会审视一个微软在十多年前处理软件断点时所用的假设，利用这一点可以检测到大部分(市面上所有的？)用户态和内核态调试器。

`x86`架构可以用多种方式编码一条特定的汇编指令。举个例子，将两个寄存器`eax`和`ebx`累加，并且将结果存储到`eax`中可采用如下助记符：`add eax, ebx`。这可以被编码成字节序列`0x03 0xC3`或`0x01 0xD8`。这两组机器码代表同一个汇编操作。

如果你只关心反调试技巧(并不想结合上下文理解它的原理)，请将进度条滚动到这篇博客的最下面。对于那些想要勇敢地读完整篇文章的人，请系好安全带(老司机要开车了)。

---

## `int 3`的漫长历史

一个`int 3`指令可以被编码成一个单字节指令`0xCC`，也可以通过不常见的方式被编码成多指令序列`0xCD 0x03`：

![](https://bbs.kanxue.com/upload/attach/201709/628056_05df473nrbnac4k.jpg)

来自[《Intel指令手册》](<https://bbs.kanxue.com/elink@214K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8$3k6@1N6$3q4J5k6g2)9J5k6h3W2F1N6r3g2D9i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3q4J5N6r3W2U0L8r3g2K6i4K6u0r3K9h3&6@1k6h3I4Q4x3X3c8K6k6r3@1>)(第2卷，第3章，第3.2节)

所以，如果Windows遇到一个多字节的`int 3`指令会发生什么呢？我们写一个简单的C++程序来看一下：

```
/*

*    Module Name:

*        int3.cpp

*

*    Abstract:

*        Examines the difference in operation between a 

*        multi-byte int 3 (0xCD 0x03) and a single-byte 

*        int 3 (0xCC).

*

*    Author:

*        Nemanja (Nemi) Mulasmajic <nm@triplefault.io>

*            2b3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8J5K9i4m8D9k6h3k6S2N6h3I4@1i4K6u0W2K9h3)9`.

*/

#pragma warning(disable: 4710)



#pragma warning(push, 0)

#include <Windows.h>

#include <stdio.h>

#pragma warning(pop)



// The size of an architecture page on x86/x64.

#define PAGE_SIZE 0x1000



// Single-byte int 3 stub.

BYTE _Int3[] = 

{ 

    0xCC,            /* int 3*/

    0xC3            /* ret */

};



// Multi-byte int 3 stub.

BYTE _LongInt3[] =

{

    0xCD, 0x03,        /* int 3 */

    0xC3            /* ret */

};



/*

*    Handles exception processing for our int 3s.

*/

DWORD WINAPI ExceptionFilter(_In_ PEXCEPTION_POINTERS ExceptionInformation)

{

    // Malformed exception information.

    if (!ExceptionInformation || !ExceptionInformation->ExceptionRecord || !ExceptionInformation->ContextRecord)

        return EXCEPTION_EXECUTE_HANDLER;



    // This is the only type of exception we should see...

    if (ExceptionInformation->ExceptionRecord->ExceptionCode != EXCEPTION_BREAKPOINT)

        return EXCEPTION_EXECUTE_HANDLER;



    printf("[+] ExceptionRecord->ExceptionAddress: 0x%p.\n", ExceptionInformation->ExceptionRecord->ExceptionAddress);



#if defined(_M_AMD64)

#define IP Rip

#elif defined(_M_IX86)

#define IP Eip

#else

#error "Compiling for unhandled architecture."

#endif



    const BYTE* InstructionPointer = (const BYTE*)ExceptionInformation->ContextRecord->IP;

    printf("[+] ContextRecord->IP: 0x%p.\n", InstructionPointer);

    printf("\t@ [0: 0x%X...]\n", InstructionPointer[0]);



#undef IP



    return EXCEPTION_EXECUTE_HANDLER;

}



/*

*    The entry point.

*/

int main(void)

{

    int status = -1;



    // Allocate executable memory.

    PBYTE Memory = (PBYTE)VirtualAlloc(NULL, PAGE_SIZE, (MEM_COMMIT | MEM_RESERVE), PAGE_EXECUTE_READWRITE);

    if (!Memory)

    {

        fprintf(stderr, "[-] ERROR: Failed to allocate memory.\n");

        goto Cleanup;

    }



    typedef void(*Function)();



    // Place our simple breakpoint stubs in executable memory.



    // Our single-byte int 3 will be at the start of this region.

    Function Int3 = (Function)&Memory[0];

    size_t BufferSize = sizeof(_Int3);

    memcpy(Int3, _Int3, BufferSize);



    printf("[+] Single-byte 'int 3' buffer: 0x%p.\n", Int3);



    // Our multi-byte int 3 will be immediately after.

    Function LongInt3 = (Function)&Memory[BufferSize];

    BufferSize = sizeof(_LongInt3);

    memcpy(LongInt3, _LongInt3, BufferSize);



    printf("[+] Multi-byte 'int 3' buffer: 0x%p.\n", LongInt3);



    // Execute both variants of int 3.



    printf("\n[+] Executing single-byte variant.\n");



    __try

    {

        Int3();

    }

    __except (ExceptionFilter(GetExceptionInformation())) {    }



    printf("\n[+] Executing multi-byte variant.\n");



    __try

    {

        LongInt3();

    }

    __except (ExceptionFilter(GetExceptionInformation())) {}



    status = 0;



Cleanup:



    // Free allocated memory.

    if (Memory)

    {

        VirtualFree(Memory, 0, MEM_FREE);

        Memory = NULL;

    }



    // Wait for [ENTER] key press to terminate the program.

    getchar();



    return status;

}
```

在你运行这个程序之后，你应该看到与之类似的输出:

![](https://bbs.kanxue.com/upload/attach/201709/628056_1jli5aong6i8kxq.jpg)

一个单字节的`int 3`（`0xCC`）和预期的一致。代码块开始处位于`0x000001BE94B90000`。当这段代码被执行后，异常处理例程启动，我们看到`_EXCEPTION_RECORD.ExceptionAddress`和`_CONTEXT.Rip`都位于`0x000001BE94B90000`，这就是`int 3`指令的开始处。完美！

多字节的`int 3`（`0xCD 0x03`）的代码块开始处位于`0x000001BE94B90002`。当这段代码被执行后，异常处理例程认为`_EXCEPTION_RECORD.ExceptionAddress`和`_CONTEXT.Rip`都位于`0x000001BE94B90003`。这是`int 3`指令的中间位置。为什么？哪里出问题了？

---

## 那个假设
