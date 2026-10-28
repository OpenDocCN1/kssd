
原因在于，我们在 `WriteFile()` API 的起始地址处设置了断点，被调试者（`notepad.exe`）内部调用 `WriteFile()` 时，会在起始地址 `752335B0` 处遇到 `INT3(0xCC)` 指令。执行该指令（`BreakPoint-INT3`）时，`EIP` 的值会增加 1 个字节（`INT3` 指令的长度）。然后控制权会转移给调式器（`hook.exe`）（因为在“调式器-被调试器者”关系中，被调试者中发生的 `EXCEPTION_BREAKPOINT` 异常需要由调式器处理）。修改覆写了数据缓冲区的内容后，`EIP` 的值被重新更改为 `WriteFile()` API 的起始地址，继续运行。

#### “脱钩”&“钩子”

另一个问题是，若只将执行流程返回到 `WriteFile()` API 起始位置，在遇到的 `INT3` 指令时，就会陷入无限循环（发生 `EXCEPTION_BREAKPOINT`）。为了不致于陷入这种境地，应该去除设置在 `WriteFile()` API 起始地址处的断点。即，将 `0xCC` 更改为 original byte（`0x6A`）（original byte 在钩取 API 前已保存）。这一操作称为“脱钩”，就是取消对 API 的钩取。

覆写好数据缓冲区并正常返回 `WriteFile()` API 代码后，`EIP` 值恢复为 `WriteFile()` API 的地址，修改后的字符串最终保存到文件。这就是 `hook.exe` 的工作原理。

若只需要钩取一次，到这儿就结束了。但是需要不断钩取，就要再次设置断点。

### 源代码分析

这一节分析 `hook.exe` 的源代码

```
#include "windows.h"
#include "stdio.h"

LPVOID g_pfWriteFile = NULL;
CREATE_PROCESS_DEBUG_INFO g_cpdi;
BYTE g_chINT3 = 0xCC, g_chOrgByte = 0;

BOOL OnCreateProcessDebugEvent(LPDEBUG_EVENT pde)
{
    // 获取WriteFile() API 地址
    g_pfWriteFile = GetProcAddress(GetModuleHandleA("kernel32.dll"), "WriteFile");

    // API Hook - WriteFile()
    //  更改第一个字节为 0xCC（INT3）
    //  originalbyte 是 g_ch0rgByte 备份
    memcpy(&g_cpdi, &pde->u.CreateProcessInfo, sizeof(CREATE_PROCESS_DEBUG_INFO));
    ReadProcessMemory(g_cpdi.hProcess, g_pfWriteFile, &g_chOrgByte, sizeof(BYTE), NULL);
    WriteProcessMemory(g_cpdi.hProcess, g_pfWriteFile, &g_chINT3, sizeof(BYTE), NULL);

    return TRUE;
}

BOOL OnExceptionDebugEvent(LPDEBUG_EVENT pde)
{
    CONTEXT ctx;
    PBYTE lpBuffer = NULL;
    DWORD dwNumOfBytesToWrite, dwAddrOfBuffer, i;
    PEXCEPTION_RECORD per = &pde->u.Exception.ExceptionRecord;

    // 是断点异常(INT3)时
    if (EXCEPTION_BREAKPOINT == per->ExceptionCode)
    {
        // 断点地址为 WriteFile() API 地址时
        if (g_pfWriteFile == per->ExceptionAddress)
        {
            // #1. Unhook
            //  将0xCC 恢复为 original byte
            WriteProcessMemory(g_cpdi.hProcess, g_pfWriteFile, &g_chOrgByte, sizeof(BYTE), NULL);

            // #2. 获取线程上下文
            ctx.ContextFlags = CONTEXT_CONTROL;
            GetThreadContext(g_cpdi.hThread, &ctx);

            // #3. 获取WriteFil() 的 param 2、3 值
            //  函数参数存在于相应进程的栈
            //  param 2 : ESP + 0x8
            //  param 3 : ESP + 0xC
            ReadProcessMemory(g_cpdi.hProcess, (LPVOID)(ctx.Esp + 0x8), &dwAddrOfBuffer, sizeof(DWORD), NULL);
            ReadProcessMemory(g_cpdi.hProcess, (LPVOID)(ctx.Esp + 0xC), &dwNumOfBytesToWrite, sizeof(DWORD), NULL);

            // #4. 分配领事缓冲区
            lpBuffer = (PBYTE)malloc(dwNumOfBytesToWrite + 1);
            memset(lpBuffer, 0, dwNumOfBytesToWrite + 1);

            // #5. 复制 WriteFile() 缓冲区到临时缓冲区
            ReadProcessMemory(g_cpdi.hProcess, (LPVOID)dwAddrOfBuffer, lpBuffer, dwNumOfBytesToWrite, NULL);
            printf("\n### original string ###\n%s\n", lpBuffer);

            // #6. 将小写字母转换为大写字母
            for (i = 0; i < dwNumOfBytesToWrite; i++)
            {
                if (0x61 <= lpBuffer[i] && lpBuffer[i] <= 0x7A)
                    lpBuffer[i] -= 0x20;
            }

            printf("\n### converted string ###\n%s\n", lpBuffer);

            // #7. 将变换后的缓冲区复制到WriteFile()缓冲区
            WriteProcessMemory(g_cpdi.hProcess, (LPVOID)dwAddrOfBuffer, lpBuffer, dwNumOfBytesToWrite, NULL);

            // #8.释放临时缓冲区
            free(lpBuffer);

            // #9. 将线程上下文的 EIP 更改为 WriteFile()首地址
            //  当前为 WriteFile()+1 位置，INT3命令之后
            ctx.Eip = (DWORD)g_pfWriteFile;
            SetThreadContext(g_cpdi.hThread, &ctx);

            // #10. 运行被调试进程
            ContinueDebugEvent(pde->dwProcessId, pde->dwThreadId, DBG_CONTINUE);
            Sleep(0);

            // #11. API Hook
            WriteProcessMemory(g_cpdi.hProcess, g_pfWriteFile, &g_chINT3, sizeof(BYTE), NULL);

            return TRUE;
        }
    }

    return FALSE;
}

void DebugLoop()
{
    DEBUG_EVENT de; //描述调试事件
    DWORD dwContinueStatus;

    // 等待被调试者发生事件
    while (WaitForDebugEvent(&de, INFINITE))//等待正在调试的进程中发生调试事件。
    {
        dwContinueStatus = DBG_CONTINUE;

        // 被调试进程生成或者附加事件
        if (CREATE_PROCESS_DEBUG_EVENT == de.dwDebugEventCode)
        {
            OnCreateProcessDebugEvent(&de);
        }
        // 异常事件
        else if (EXCEPTION_DEBUG_EVENT == de.dwDebugEventCode)
        {
            if (OnExceptionDebugEvent(&de))
                continue;
        }
        // 被调试进程终止事件
        else if (EXIT_PROCESS_DEBUG_EVENT == de.dwDebugEventCode)
        {
            // 被调试者终止---调试器终止
            break;
        }

        // 再次运行被调试者
        ContinueDebugEvent(de.dwProcessId, de.dwThreadId, dwContinueStatus);
    }
}

int main(int argc, char* argv[])
{
    DWORD dwPID;

    if (argc != 2)
    {
        printf("\nUSAGE : hookdbg.exe <pid>\n");
        return 1;
    }

    // Attach Process
    dwPID = atoi(argv[1]);
    if (!DebugActiveProcess(dwPID))
    {
        printf("DebugActiveProcess(%d) failed!!!\n"
            "Error Code = %d\n", dwPID, GetLastError());
        return 1;
    }

    // 调试器循环
    DebugLoop();

    return 0;
}
```

#### `main()`

```
#include "windows.h"
#include "stdio.h"

LPVOID g_pfWriteFile = NULL;
CREATE_PROCESS_DEBUG_INFO g_cpdi;
BYTE g_chINT3 = 0xCC, g_ch0rgByte = 0;

int main(int agc, char* argv[])
{
    DWORD dwPID;

    if (argc != 2)
    {
        printf("\n USEAGE : %s  <PID>\n", argv[0], argv[1]);
        return 1;
    }

    // Attach Process
    dwPID = atoi(argv[1]);
    if (!DebugActiveProcess(dwPID))
    {
        printf("DebugActiveProcess(%d) failed !!!\n"
            "Error Code = %d\n", dwPID, GetLastError());
        return 1;
    }

    // 调试器
    DebugLoop();
    return 0;
}
```

```
BOOL DebugActiveProcess(
  DWORD dwProcessId
);
// 使调试器能够附加到活动进程并对其进行调试。

Parameters
dwProcessId
// 要调试的进程的标识符。
// 调试器被授予对进程的调试访问权限，
// 就像它使用 DEBUG_ONLY_THIS_PROCESS 标志创建进程一样。
```

`main()` 函数的代码非常简单，以程序运行参数的形式接受要钩取 API 的进程 PID。然后通过 `DebugActiveProcess()` API 将调试器附加到该运行的进程上，开始调试（上面输入的 PID 作为参数传入函数）。

然后进入 `DebugLoop()` 函数，处理来自被调试者的调试信息。

```
void DebugLoop()
{
    DEBUG_EVENT de; //描述调试事件
    DWORD dwContinueStatus;

    // 等待被调试者发生事件
    while (WaitForDebugEvent(&de, INFINITE))//等待正在调试的进程中发生调试事件。
    {
        dwContinueStatus = DBG_CONTINUE;

        // 被调试进程生成或者附加事件
        if (CREATE_PROCESS_DEBUG_EVENT == de.dwDebugEventCode)
        {
            OnCreateProcessDebugEvent(&de);
        }
        // 异常事件
        else if (EXCEPTION_DEBUG_EVENT == de.dwDebugEventCode)
        {
            if (OnExceptionDebugEvent(&de))
                continue;
        }
        // 被调试进程终止事件
        else if (EXIT_PROCESS_DEBUG_EVENT == de.dwDebugEventCode)
        {
            // 被调试者终止---调试器终止
            break;
        }

        // 再次运行被调试者
        ContinueDebugEvent(de.dwProcessId, de.dwThreadId, dwContinueStatus);
    }
}
```
