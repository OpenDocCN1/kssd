
Figure 13展现了我写的`UnhandledExceptionFilter`的伪代码. 这个API的开始有点奇怪(至少我这么认为). 假设错误是一个`EXCEPTION_ACCESS_VIOLATION`, 那么代码会调用`_BasepCheckForReadOnlyResource`. 虽然我还没提供这个函数的伪代码, 但是我可以总结它一下. 如果异常发生是由于一个EXE或DLL的resource section(`.rsrc`)被写入了, 那么`*BasepCurrentTopLevelFilter`会修改错误页面的本来的正常的只读属性, 从而允许写入发生. 如果这个特定的场景发生的话, `UnhandledExceptionFilter` 会返回`EXCEPTION_CONTINUE_EXECUTION`, 并且执行会在错误的指令处重新执行.

# Figure 13 UnHandledExceptionFilter Pseudocode

```
UnhandledExceptionFilter( STRUCT _EXCEPTION_POINTERS *pExceptionPtrs )
{
PEXCEPTION_RECORD pExcptRec;
DWORD currentESP;
DWORD retValue;
DWORD DEBUGPORT;
DWORD   dwTemp2;
DWORD   dwUseJustInTimeDebugger;
CHAR    szDbgCmdFmt[256];
/ / Template string retrieved
from
AeDebug key
CHAR    szDbgCmdLine[256];
/ / Actual debugger string after filling
in
STARTUPINFO startupinfo;
PROCESS_INFORMATION pi;
HARDERR_STRUCT  harderr;
/ / ???
BOOL fAeDebugAuto;
TIB * pTib;
/ / Thread information block
pExcptRec = pExceptionPtrs->ExceptionRecord;
if (   (pExcptRec->ExceptionCode == EXCEPTION_ACCESS_VIOLATION)
&& (pExcptRec->ExceptionInformation[0]) )
{
retValue =  _BasepCheckForReadOnlyResource(pExcptRec->ExceptionInformation[1]);
if ( EXCEPTION_CONTINUE_EXECUTION == retValue )
return EXCEPTION_CONTINUE_EXECUTION;
}

/ / See if this process is being run under a debugger...
retValue = NtQueryInformationProcess(GetCurrentProcess(), ProcessDebugPort,
&debugPort, sizeof(debugPort),
0 );
if ( (retValue >= 0) && debugPort ) / / Let debugger have it
return EXCEPTION_CONTINUE_SEARCH;

/ / Did the user call SetUnhandledExceptionFilter?  If so, call their
/ / installed proc now.
if ( _BasepCurrentTopLevelFilter )
{
retValue = _BasepCurrentTopLevelFilter( pExceptionPtrs );
if ( EXCEPTION_EXECUTE_HANDLER == retValue )
return EXCEPTION_EXECUTE_HANDLER;
if ( EXCEPTION_CONTINUE_EXECUTION == retValue )
return EXCEPTION_CONTINUE_EXECUTION;
/ / Only EXCEPTION_CONTINUE_SEARCH goes on from here
}

/ / Has SetErrorMode(SEM_NOGPFAULTERRORBOX) been called?
if ( 0 == (GetErrorMode() & SEM_NOGPFAULTERRORBOX) )
{
harderr.elem0 = pExcptRec->ExceptionCode;
harderr.elem1 = pExcptRec->ExceptionAddress;
if ( EXCEPTION_IN_PAGE_ERROR == pExcptRec->ExceptionCode )
harderr.elem2 = pExcptRec->ExceptionInformation[2];
else
harderr.elem2 = pExcptRec->ExceptionInformation[0];
dwTemp2 = 1;
fAeDebugAuto = FALSE;
harderr.elem3 = pExcptRec->ExceptionInformation[1];
pTib = FS:[18h];
DWORD someVal = pTib->pProcess->0xC;
if ( pTib->threadID != someVal )
{
__try
{
char szDbgCmdFmt[256]
retValue = _GetProfileStringA(
"AeDebug",
"Debugger",
0,
szDbgCmdFmt, sizeof(szDbgCmdFmt)-1
);
if ( retValue )
dwTemp2 = 2;
char szAuto[8]
retValue = GetProfileStringA(
"AeDebug",
"Auto",
"0",
szAuto, sizeof(szAuto)-1
);
if ( retValue )
if ( 0 == strcmp( szAuto, "1" ) )
if ( 2 == dwTemp2 )
fAeDebugAuto = TRUE;
}
__except( EXCEPTION_EXECUTE_HANDLER )
{
ESP = currentESP;
dwTemp2 = 1
fAeDebugAuto = FALSE;
}
}
if ( FALSE == fAeDebugAuto )
{
retValue =  NtRaiseHardError(
STATUS_UNHANDLED_EXCEPTION |
0x10000000,
4,
0, &harderr,
_BasepAlreadyHadHardError ?
1 : dwTemp2,
&dwUseJustInTimeDebugger );
}
else
{
dwUseJustInTimeDebugger = 3;
retValue = 0;
}
if (    retValue >= 0
&&  ( dwUseJustInTimeDebugger == 3 )
&&  ( !_BasepAlreadyHadHardError )
&&  ( !_BaseRunningInServerProcess ) )
{
_BasepAlreadyHadHardError = 1;
SECURITY_ATTRIBUTES secAttr = { sizeof(secAttr),
0, TRUE };
HANDLE hEvent = CreateEventA( &secAttr, TRUE,
0,
0 );
memset( &startupinfo,
0, sizeof(startupinfo) );
sprintf(szDbgCmdLine, szDbgCmdFmt, GetCurrentProcessId(), hEvent);
startupinfo.cb = sizeof(startupinfo);
startupinfo.lpDesktop = "Winsta0\Default"
CsrIdentifyAlertableThread();
/ / ???
retValue = CreateProcessA(
0, / / lpApplicationName
szDbgCmdLine, / / Command line
0, 0, / / process, thread security attrs
1, / / bInheritHandles
0, 0, / / creation flags, environment
0, / / current directory.
&statupinfo, / / STARTUPINFO
&pi ); / / PROCESS_INFORMATION
if ( retValue && hEvent )
{
NtWaitForSingleObject( hEvent,
1,
0 );
return EXCEPTION_CONTINUE_SEARCH;
}
}
if ( _BasepAlreadyHadHardError )
NtTerminateProcess(GetCurrentProcess(), pExcptRec->ExceptionCode);
}
return EXCEPTION_EXECUTE_HANDLER;
}
LPTOP_LEVEL_EXCEPTION_FILTER
SetUnhandledExceptionFilter(
LPTOP_LEVEL_EXCEPTION_FILTER lpTopLevelExceptionFilter );
{
/ / _BasepCurrentTopLevelFilter is a KERNEL32.DLL global var
LPTOP_LEVEL_EXCEPTION_FILTER previous= _BasepCurrentTopLevelFilter;
/ / Set the new value
_BasepCurrentTopLevelFilter = lpTopLevelExceptionFilter;
return previous; / / return the old value
}
```

下一个`UnhandledExceptionFilter`的任务是确定进程是否运行在一个Win32 Debugger之下. 如果是的话, 进程的`DEBUG_PROCESS` 或`BUG_ONLY_THIS_PROCESS` 标志在进程创建时就被标好. `UnhandledExceptionFilter` 使用`NtQueryInformationProcess` 函数(该函数我在这个月的Under the Hood专栏里描述过)来判断进程是否正在被debug. 如果是的话, 那么这个API就返回`EXCEPTION_CONTINUE_SEARCH`, 这就会告诉系统的其他部分来唤醒debugger进程, 并且告诉debugger进程debuggee里发生了一个异常.

接下来, `UnhandledExceptionFilter`'s 会调用user-installed的为unhandled exception filter, 如果有的话. 正常情况下, user-installed的callback函数不存在的, 但是用户可以通过`SetUnhandledExceptionFilter` 这个API来安装一个. 我也为这个API提供了伪代码. 这个API简单地使用"用户的新callback函数的地址"来替换一个global variable, 并且返回老callback的值.

有了前面基础问题的解决, `UnhandledExceptionFilter` 能够开始做它的工作了: 使用永久时尚的应用程序错误对话框来通知你你的可耻的编程大错. :) 有两种方法可以避免这个对话框.

第一, 如果进程调用了`SetErrorMode` 并且指定了`SEM_NOGPFAULTERRORBOX` 标志.

第二, `AeDebug`注册表的`Auto`键设为1. 在这种情况下, `UnhandledExceptionFilter` 会跳过应用程序错误对话框, 并自动开启在`AeDebug` 注册表的`Debugger`键指定的debugger. 如果你熟悉"just in time debugging", 这就是操作系统支持它的地方了. 晚些时候在继续这个部分.

在多数情况下, 这两种避免对话框的情况都不为true, 并且`UnhandledExceptionFilter` 会调用`NTDLL.DLL`中的`NtRaiseHardError`方法. 这个方法会带来Application Error 对话框. 这个对话框等你点击OK来结束进程, 或者点击Cancel来debug它. (或许只是我这样想吧, 点击cancel来开启debugger看起来有点像是退步)

如果你点击Application Error对话框的OK按钮, `UnhandledExceptionFilter` 会返回`EXCEPTION_EXECUTE_HANDLER`. 调用`UnhandledExceptionFilter` 的代码通常会通过结束它自己来响应(正如你在`BaseProcessStart` 的代码中看到的). 这带来了一个有趣的要点. 大多数人假设系统会使用一个unhandled exception来结束进程. 其实更正确的方法是, 系统配置好了一切, 以便于unhandled exception能够引发进程终结它自己.
