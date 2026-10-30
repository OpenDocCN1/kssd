
```cpp
#include "SRMFuzzy.h"
#include "Thread.h"
#include "Pages.h"

SRMFuzzy::SRMFuzzy(char *szFilePath, DWORD dwFuncBegin, DWORD dwFuncEnd):Fuzzy(szFilePath)
{
    this->dwFuncBegin = dwFuncBegin;
    this->dwFuncEnd = dwFuncEnd;
    this->bOrgFuncBegin = 0;
    this->bOrgFuncEnd = 0;
    this->bInt3 = 0xCC;
}

void SRMFuzzy::BeginFuzzy()
{
    STARTUPINFO si = { 0 };
    PROCESS_INFORMATION pi = { 0 };
    DEBUG_EVENT DbgEvt = { 0 }; // 保存调试事件的数据结构
    bool bContinue = true; // 是否继续
    DWORD dwContinueStatus = DBG_CONTINUE; // 恢复继续执行用的状态代码
    DWORD dwSize = 4; // 要修改的数据大小
    PVOID pInputAddr = NULL; // 被被调试进程中申请的数据地址
    CONTEXT cxt = { 0 };
    bool bFirstPointer = true; // 记录是否是第一次触发函数起始地址的软件断点

    si.cb = sizeof(si);
    if (!CreateProcess(GetFilePath(), NULL, NULL, NULL, FALSE, CREATE_NEW_CONSOLE | DEBUG_PROCESS | DEBUG_ONLY_THIS_PROCESS, NULL, NULL, &si, &pi))
    {
        ShowError("CreateProcess");
        goto exit;
    }

    printf("Fuzzy Begin...\n");
    while (bContinue && dwSize < MAX_PATH)
    {
        memset(&DbgEvt, 0, sizeof(DbgEvt));

        // 等待调试事件发生
        bContinue = WaitForDebugEvent(&DbgEvt, INFINITE);
        if (!bContinue)
        {
            ShowError("WaitForDebugEvent");
            break;
        }

        witch (DbgEvt.dwDebugEventCode)
        {
        case EXCEPTION_DEBUG_EVENT:
        {
            switch (DbgEvt.u.Exception.ExceptionRecord.ExceptionCode)
            {
            // 捕获到异常，打印出来
            case EXCEPTION_ACCESS_VIOLATION:
            {
                printf(".....Catch Access Violation...\n");
                printf("Address: 0x%X\n", DbgEvt.u.Exception.ExceptionRecord.ExceptionAddress);
                printf("dwSize:%d\n", dwSize);
                goto exit;
            }
            case EXCEPTION_BREAKPOINT:
            {
                // 触发断点则判断触发int 3断点的位置
                DWORD dwPointAddr = (DWORD)DbgEvt.u.Exception.ExceptionRecord.ExceptionAddress;
                if (dwPointAddr == this->dwFuncBegin)
                {
                    // 断点位置是函数起始地址
                    // 是否是第一次触发函数起始处的软件断点
                    if (bFirstPointer)
                    {
                        // 保存主线程的状态
                        cxt.ContextFlags = CONTEXT_FULL;
                        if (!GetThreadContext(pi.hThread, &cxt))
                        {
                            ShowError("GetThreadContext");
                            goto exit;
                        }

                        // 将EIP重新指向函数的起始地址
                        cxt.Eip -= sizeof(this->bInt3);

                        // 这一次的保存是为了主线程加入到全局变量中的时候
                        // EIP可以指向函数起始地址
                        if (!SetThreadContext(pi.hThread, &cxt))
                        {
                            ShowError("SetThreadContext");
                            goto exit;
                        }

                        // 保存线程CONTEXT与内存状态到全局变量中
                        if (!SaveThreadContext(pi.dwProcessId) || !SavePages(pi.hProcess))
                        {
                            goto exit;
                        }

                        bFirstPointer = false;
                    }

                    // 恢复函数起始的字节
                    if (!WriteProcessMemory(pi.hProcess, (PVOID)this->dwFuncBegin, &(this->bOrgFuncBegin), sizeof(this->bOrgFuncBegin), NULL))
                    {
                        ShowError("WriteProcessMemory");
                        goto exit;
                    }

                    // 函数末尾设置为软件断点
                    if (!WriteProcessMemory(pi.hProcess, (PVOID)this->dwFuncEnd, &(this->bInt3), sizeof(this->bInt3), NULL))
                    {
                        ShowError("WriteProcessMemory");
                        goto exit;
                    }

                    // 在被调试进程中申请并初始化输入数据
                    pInputAddr = GetData(pi.hProcess, dwSize);
                    if (!pInputAddr)
                    {
                        goto exit;
                    }

                    // 设置输入数据地址为随机产生的数据地址
                    if (!WriteProcessMemory(pi.hProcess, (PVOID)(cxt.Esp + 4), (PVOID)&pInputAddr, sizeof(pInputAddr), NULL))
                    {
                        ShowError("WriteProcessMemory");
                        goto exit;
                    }

                    // 不是第一次触发断点，此时就要修改主线程CONTEXT的EIP
                    if (!bFirstPointer)
                    {
                        if (!SetThreadContext(pi.hThread, &cxt))
                        {
                            ShowError("SetThreadContext");
                            goto exit;
                        }
                    }
                }
                else if (dwPointAddr == this->dwFuncEnd)
                {
                    // 执行到函数末尾
                    // 释放掉申请的内存
                    if (!FreeData(pi.hProcess, pInputAddr, dwSize))
                    {
                        goto exit;
                    }

                    // 重新设定申请的内存大小
                    pInputAddr = NULL;
                    dwSize += 4;

                    // 恢复线程CONTEXT与内存状态
                    if (!RestoreThreadContext() || !RestorePages(pi.hProcess))
                    {
                        goto exit;
                    }

                    // 修改函数起始地址中的字节为软件断点
                    if (!WriteProcessMemory(pi.hProcess, (PVOID)this->dwFuncBegin, &(this->bInt3), sizeof(this->bInt3), NULL))
                    {
                        ShowError("WriteProcessMemory");
                        goto exit;
                    }
                }
                break;
            }
            default:
            {
                break;
            }
            }
            break;
        }
        case CREATE_PROCESS_DEBUG_EVENT:
        {
            // 被调试进程创建的时候将函数的原始字节读取出来
            // 随后写入软件断点
            if (DbgEvt.dwProcessId == pi.dwProcessId)
            {
                if (!ReadProcessMemory(pi.hProcess, (PVOID)this->dwFuncBegin, &(this->bOrgFuncBegin), sizeof(this->bOrgFuncBegin), NULL))
                {
                    ShowError("ReadProcessMemory");
                    goto exit;
                }

                if (!WriteProcessMemory(pi.hProcess, (PVOID)this->dwFuncBegin, &(this->bInt3), sizeof(this->bInt3), NULL))
                {
                    ShowError("WriteProcessMemory");
                    goto exit;
                }

                if (!ReadProcessMemory(pi.hProcess, (PVOID)this->dwFuncEnd, &(this->bOrgFuncEnd), sizeof(this->bOrgFuncEnd), NULL))
                {
                    ShowError("ReadProcessMemory");
                    goto exit;
                }

                if (!WriteProcessMemory(pi.hProcess, (PVOID)this->dwFuncEnd, &(this->bInt3), sizeof(this->bInt3), NULL))
                {
                    ShowError("WriteProcessMemory");
                    goto exit;
                }

                printf("Init BeakPoint ok...\n");
            }
            break;
        }
        default:
        {
            break;
        }
        }

        // 恢复被调试进程继续运行
        bContinue = ContinueDebugEvent(DbgEvt.dwProcessId, DbgEvt.dwThreadId, dwContinueStatus);;
        if (!bContinue)
        {
            ShowError("ContinueDebugEvent");
            break;
        }
    }

exit:
    printf("Fuzzy End...\n");
}
```

运行结果如下：

![avatar](https://bbs.kanxue.com/upload/attach/202202/835440_MV52CH6JW3HXA6A.png)

模糊测试器成功捕获到错误，而且此时只需要启动一次被测程序就可以完成任务。不需要在反复经历`Init->Decrypt->test->Vuln`这些流程，降低了性能的开销。

# 六.参考资料

> - 《模糊测试：强制发掘安全漏洞的利器》

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-3-9 16:51 被1900编辑，原因：[#自动化挖掘](https://bbs.kanxue.com/forum-150-1-155.htm) [#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)
