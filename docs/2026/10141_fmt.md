
# 使用 Win32调试API 注意内存泄漏

最近学习 Win32调试API，写了点程序，又学到了一些东西。

比如用`A.EXE`做 Debuger，`B.EXE`做 Debuggee。

一般的教程都象下面这样写`A.EXE`:

```c
PROCESS_INFORMATION  pi;
STARTUPINFO          si;
DEBUG_EVENT          de;
DWORD                dwContinueStatus;
DWORD                dwExceptionNum=0;           // 异常次数

GetStartupInfo(&si);

CreateProcess("B.EXE", 0, 0, 0, 0, DEBUG_PROCESS, 0, 0, &si, &pi));

while( WaitForDebugEvent(&de, INFINITE) )
{
     dwContinueStatus = DBG_EXCEPTION_NOT_HANDLED;
     if (de.dwDebugEventCode == EXIT_PROCESS_DEBUG_EVENT)
     {
           break;                          // debugee 结束
     }
     else if (de.dwDebugEventCode == EXCEPTION_DEBUG_EVENT)
     {
           dwExceptionNum++;               // 第一次异常需要注意
           if (dwExceptionNum == 1) dwContinueStatus = DBG_CONTINUE;
     }
     ...
     ContinueDebugEvent(de.dwProcessId, de.dwThreadId, dwContinueStatus);
}

CloseHandle(pi.hThread);
CloseHandle(pi.hProcess);
```

上面的程序好象没问题。但我在调试中发现，`B.EXE`结束后，只要`A.EXE`不关闭，`B.EXE`就不能删除改名等，

这说明`A.EXE`中打开了`B.EXE`的文件句柄，但一直没有关闭。经过反复研究，终于找到了问题所在，改写程序如下：

```c
while( WaitForDebugEvent(&de, INFINITE) )
{
     dwContinueStatus = DBG_EXCEPTION_NOT_HANDLED;
     if (de.dwDebugEventCode == EXIT_PROCESS_DEBUG_EVENT)
     {
          break;                          // debugee 结束
     }
     else if (de.dwDebugEventCode == CREATE_PROCESS_DEBUG_EVENT)
    {
          // Handle to the process's image file
          CloseHandle(de.u.CreateProcessInfo.hFile);
          // Handle to the process
          CloseHandle(de.u.CreateProcessInfo.hProcess);
          // Handle to the initial thread of the process identified by the hProcess member
          CloseHandle(de.u.CreateProcessInfo.hThread);
     }
     else if (de.dwDebugEventCode == CREATE_THREAD_DEBUG_EVENT)
     {
          // Handle to the thread whose creation caused the debugging event
          CloseHandle(de.u.CreateThread.hThread);
      }
      else if (de.dwDebugEventCode == LOAD_DLL_DEBUG_EVENT)
     {
           // Handle to the loaded DLL
           CloseHandle(de.u.LoadDll.hFile);
      }
      else if (de.dwDebugEventCode == EXCEPTION_DEBUG_EVENT)
     {
           dwExceptionNum++;         // 第一次异常需要注意
           if (dwExceptionNum == 1) dwContinueStatus = DBG_CONTINUE;
      }
      ...
      ContinueDebugEvent(de.dwProcessId, de.dwThreadId, dwContinueStatus);
}

CloseHandle(pi.hThread);
CloseHandle(pi.hProcess);
```

原因就在于`CREATE_PROCESS_DEBUG_EVENT`、`CREATE_THREAD_DEBUG_EVENT`、`LOAD_DLL_DEBUG_EVENT`时会打开一些句柄，如果不使用这些句柄，最好马上关闭。

当然`A.EXE`关闭时，这些句柄都会自动关闭，但有些程序，debuger 创建在`Explorer`，就会造成内存泄露。

具体细节请参考 MSDN。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
