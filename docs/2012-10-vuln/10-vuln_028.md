
# 栈溢出动态定位器 LSO1.0

> 原文：[`www.pediy.com/kssd/pediy10/75043.html`](https://www.pediy.com/kssd/pediy10/75043.html)

## 基本信息

标 题: 【原创】 栈溢出动态定位器 LSO1.0  
作 者: bambooooo  
时 间: 2008-10-21,10:40  
链 接: http://bbs.pediy.com/showthread.php?t=75043

## 关于栈溢出动态定位器 LSO1.0

程序发生溢出，并不总是能得到一个异常的提示框，如何判断程序内部堆栈是不是出现异常？

一个 `N` 层嵌套的函数中，最内层指令可能通过传入的参数导致最外层函数堆栈的溢出如何有效监视多重嵌套函数堆栈的溢出？

栈桢内的数据，不仅包含局部变量，也可能包含了函数指针，如何判定是否被覆盖？

多线程程序中，如果线程间关联较大，那么通过 `OD` 等工具手动完成栈溢出的定位效率可能会很低，如何同时监视多个线程的堆栈异常？

网上找不到一款称手的工具，于是决定自己写一个，附件下载，目前的版本选择了一个比较容易实现的方案:实时备份函数调用时压入堆栈的返回值，监视返回值的改变。

以下贴出简化的进程调试部分框架代码，希望能抛砖印玉，得到大牛们的指点，提出更完善的堆栈溢出定位方案。

附件程序中提供了两个溢出程序的示例，附源码用于测试，示例 1 为多线程程序,其中线程 2 会导致堆栈溢出，示例 2 为多层嵌套函数调用，嵌套的第 5 层函数代码会导致第 1 层函数堆栈溢出。

``` 
DWORD WINAPI DebugProcess(char *szFileName)  
{  
    GetStartupInfo(&Info);  
    if(!CreateProcess(NULL,szFileName,NULL,NULL,FALSE,DEBUG_PROCESS  
                |  DEBUG_ONLY_THIS_PROCESS,NULL,NULL,&Info,&ProInfo))  
    {  
        PostMsg(USER_CREATEPROCESS_ERROR);  
        goto Exit;  
    }  
    Initial();  
    do  
    {  
        WaitForDebugEvent(&DebugEvent,INFINITE);  
        switch(DebugEvent.dwDebugEventCode)  
        {  
        case CREATE_PROCESS_DEBUG_EVENT:  
             //设置指定的断点  
             SetBreakPointByTable(ProInfo.hProcess);  
             break;  
        case CREATE_THREAD_DEBUG_EVENT:  
             //记录增加的线程  
             AddThreadInfo(&DebugEvent,gTI);  
             //传递自定义消息以更新界面显示  
             PostMsg(USER_CREATE_THREAD);  
             break;  
        case EXIT_THREAD_DEBUG_EVENT:  
             //删除记忆的线程  
             DelThreadInfoByID(gTI,DebugEvent.dwThreadId);  
             //传递线程建立消息  
             PostMsg(USER_CREATE_THREAD);  
             break;  
        case LOAD_DLL_DEBUG_EVENT:  
             //记录加载的 DLL 文件名  
             GetDllFileName(&DebugEvent,  
                     gstrDllFileName,MAX_PATHNAME_LEN);  
             //加载 DLL 消息  
             PostMsg(USER_LOAD_DLL);  
             break;  
        case EXCEPTION_DEBUG_EVENT:  
             ////////////////////////断点异常处理//////////////////////////  
             if (DebugEvent.u.Exception.ExceptionRecord.ExceptionCode  
                               == EXCEPTION_BREAKPOINT)  
             {  
                 //异常地址  
                 pvExceptionAddress = DebugEvent.u.  
                          Exception.ExceptionRecord.ExceptionAddress;  
                 //异常地址过滤  
                if(IsAddrInRangeSeleted(pvExceptionAddress))  
                    break;  
                //恢复原指令        
                RestoreCmdFromImg(ProInfo.hProcess,pvExceptionAddress);  
                //设置指令回溯  
                SetCmdBackTrack(&DebugEvent);  
                //设置单步  
                SetCmdSingleStep(&DebugEvent);  
                //得到反汇编指令  
                dwCmdSize = GetDisasm(Cmd, MAXCMDSIZE, \\  
                        (DWORD)pvExceptionAddress, &Da, DISASM_CODE);  
                //如果是 CALL 指令  
                if (Da.cmdtype == C_CAL)  
                {  
                     //记录入栈的返回值  
                     LogRetValueInStack(&DebugEvent);  
                     //更新显示调用列表消息  
                     PostMsg(USER_UPDATE_CALLLIST);  
                }  
                //如果到达 RET 指令  
                if(Da.cmdtype == C_RET)  
                {  
                    //如返回值被改变  
                    if(RetValueChanged(&DebugEvent))  
                   {  
                       //记录堆栈溢出相关信息  
                       LogOverFlowInfo(&DebugEvent);  
                       //传递堆栈溢出消息  
                       PostMsg(USER_STACK_OVERFLOW);  
                   }  
                }  
            }  
           ///////////////////////////单步异常处理///////////////////////////////  
            if(DebugEvent.u.Exception.ExceptionRecord.ExceptionCode  
                                 == EXCEPTION_SINGLE_STEP)  
            {  
                //如果是从 int3 进入  
                if(IsInt3)  
                //恢复断点  
                RestoreBreakPoint(ProInfo.hProcess);  
                //校验记录过的堆栈返回值  
                CheckStackLog();  
            }  
           ///////////////////////////违规访问处理/////////////////////////////////  
            if (DebugEvent.u.Exception.ExceptionRecord.ExceptionCode  
                      == EXCEPTION_ACCESS_VIOLATION)  
            {  
                //记录违规及堆栈异常信息  
                LogException(ProInfo.hProcess,&DebugEvent);  
                //传递访问违规消息  
                PostMsg(USER_ACCESS_VIOLATION);  
                goto Exit;  
            }  
            break;  
         }  
         ContinueDebugEvent(DebugEvent.dwProcessId, \\  
                     DebugEvent.dwThreadId,dwContinueStatus);  
    }while(DebugEvent.dwDebugEventCode != EXIT_PROCESS_DEBUG_EVENT);  
Exit:  
  Uninitial();  
  return 1;  
}  
```

## 附件

上传的附件

![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | test.rar [附件请去论坛下载：[`bbs.pediy.com/showthread.php?t=75043`](http://bbs.pediy.com/showthread.php?t=75043%20) ]
