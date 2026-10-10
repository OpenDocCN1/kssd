

# 从源代码看.net 下 exe 的加载过程

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-333.htm`](https://www.pediy.com/kssd/pediy08/pediy8-333.htm)

这里的源代码自然不是指.net Framework 的源码，不过微软公开了一个代号为 rotor 的 open source cli 的源码，你可以把它看为轻量级的.net framework。最关键的是，它俩的运行机理大致相同。今天，我们就从 rotor 的源码中看看做为程序调试最基本的 exe 文件的动态加载。同样，先给出参考文献，免得有人说我抄袭。《inside the rotor cli》，另一本是《shared source cli》，只不过网上搞不到。当然，还要从 MSDN 的网站下载 sscli2.0 压缩包。  
    和 win32 下一样，系统会提供一个 loader 将 exe 读入，sscli 中提供了另一个 loader 的例子：clix.exe。我们暂且把它看为系统默认的 loader，来看源码（clix.cpp），注意红色的代码  
  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> DWORD Launch(WCHAR\* pFileName, WCHAR\* pCmdLine)
> {
>     WCHAR exeFileName\[MAX\_PATH + 1\];
>     DWORD dwAttrs;
>     DWORD dwError;
>     DWORD nExitCode;
> 
> ...
> //这里进行一系列文件的属性检查
> ...
>  
>     if (dwError != ERROR\_SUCCESS) {
>         // We can't find the file, or there's some other problem. Exit with an error.
>         fwprintf(stderr, L"%s: ", pFileName);
>         DisplayMessageFromSystem(dwError);
>         return 1;   // error
>     }
>     nExitCode = \_CorExeMain2(NULL, 0, pFileName, NULL, pCmdLine);
> 
>     // \_CorExeMain2 never returns with success
>     \_ASSERTE(nExitCode != 0);
> 
>     DisplayMessageFromSystem(::GetLastError());
> 
>     return nExitCode;
> }
> 
> * * *
> 
> 
> ```

  
  
    这里我们看到了著名的 CorExeMain，还记得用 PE 编辑文件打开.netPE 文件，只引入了一个函数吗？mscoree.dll!\_CorExeMain。奇怪，怎么不是\_CorExeMain2 呢？这只是 rotor 和商业版的 framework 的一点区别而已。你可以用 IDApro 逆一下 mscoree.dll，就可以看到\_CorExeMain()只不过是一个中转，代码如下  
  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> .text:79011B47                 push    offset a\_corexemain ; "\_CorExeMain"
> .text:79011B4C                 push    \[ebp+hModule\]   ; hModule
> .text:79011B4F                 call    ds:\_\_imp\_\_GetProcAddress@8 ; GetProcAddress(x,x)
> .text:79011B55                 test    eax, eax
> .text:79011B57                 jz      loc\_79019B46
> .text:79011B5D                 call    eax
> 
> * * *
> 
> 
> ```

  
  
进入后马上就调用了 mscorwks.dll 的\_CorExeMain。而这个函数和 rotor 中刚才提到的\_CorExeMain2 提供的功能差不多，就开始 exe 载入的初始化了。这些都可以从反汇编代码与源代码比较看出来。继续回到 sscli 中，来看\_CorExeMain2()的代码（ceemain.cpp）  
  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> \_\_int32 STDMETHODCALLTYPE \_CorExeMain2( // Executable exit code.
>     PBYTE   pUnmappedPE,                // -> memory mapped code
>     DWORD   cUnmappedPE,                // Size of memory mapped code
>     \_\_in LPWSTR  pImageNameIn,          // -> Executable Name
>     \_\_in LPWSTR  pLoadersFileName,      // -> Loaders Name
>     \_\_in LPWSTR  pCmdLine)              // -> Command Line
> {
> 
>     // This entry point is used by clix
>     BOOL bRetVal = 0;
> 
>     //BEGIN\_ENTRYPOINT\_VOIDRET;
> 
>     // Before we initialize the EE, make sure we've snooped for all EE-specific
>     // command line arguments that might guide our startup.
>     HRESULT result = CorCommandLine::SetArgvW(pCmdLine);
> 
>     if (!CacheCommandLine(pCmdLine, CorCommandLine::GetArgvW(NULL))) {
>         LOG((LF\_STARTUP, LL\_INFO10, "Program exiting - CacheCommandLine failed\\n"));
>         bRetVal = -1;
>         goto exit;
>     }
> 
>     if (SUCCEEDED(result))
>         result = CoInitializeEE(COINITEE\_DEFAULT | COINITEE\_MAIN);
> 
>     if (FAILED(result)) {
>         VMDumpCOMErrors(result);
>         SetLatchedExitCode (-1);
>         goto exit;
>     }
> 
>     // This is here to get the ZAPMONITOR working correctly
>     INSTALL\_UNWIND\_AND\_CONTINUE\_HANDLER;
> 
> 
>     // Load the executable
>     bRetVal = ExecuteEXE(pImageNameIn);
> 
> ...
> ...
>     
> 
> * * *
> 
> 
> ```

  
      
    大多数代码都可以略过，关键的就两个，一个是初始化 ee（execute engine），初始化成功后就调用 ExecuteEXE，参数是文件名。这里可以清楚地看到\_CorExeMain()的传入参数是什么。ExecuteEXE()的代码不多，也是个跳板：  
  
  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> BOOL STDMETHODCALLTYPE ExecuteEXE(HMODULE hMod)
> {
>     STATIC\_CONTRACT\_GC\_TRIGGERS;
> 
>     \_ASSERTE(hMod);
>     if (!hMod)
>         return FALSE;
> 
>     ETWTraceStartup::TraceEvent(ETW\_TYPE\_STARTUP\_EXEC\_EXE);
>     TIMELINE\_START(STARTUP, ("ExecuteExe"));
> 
>     EX\_TRY\_NOCATCH
>     {
>         // Executables are part of the system domain
>         SystemDomain::ExecuteMainMethod(hMod);
>     }
>     EX\_END\_NOCATCH;
> 
>     ETWTraceStartup::TraceEvent(ETW\_TYPE\_STARTUP\_EXEC\_EXE+1);
>     TIMELINE\_END(STARTUP, ("ExecuteExe"));
> 
>     return TRUE;
> }       
> 
> * * *
> 
> 
> ```

   
  
    同样，关键的代码只有一行，SystemDomain::ExecuteMainMethod(hMod)。其中，字面上看 ExecuteMainMethod 是将传入的文件作为了一个 module，在.net 中，如果要以包含关系算的话，assembly > module > class > method。也就是说每一个 assembly 可能包含多个 module，且至少有一个 module 有且只有一个 MainMethod，就是入口方法。  
      
    下面转到 SystemDomain::ExecuteMainMethod()的代码中（assembly.cpp）  
      

>  
> 
> ```
> 代码:
> 
> * * *
> 
>     
> INT32 Assembly::ExecuteMainMethod(PTRARRAYREF \*stringArgs)
> {
>     CONTRACTL
>     {
>         INSTANCE\_CHECK;
>         THROWS;
>         GC\_TRIGGERS;
>         MODE\_ANY;
>         ENTRY\_POINT;
>         INJECT\_FAULT(COMPlusThrowOM());
>     }
>     CONTRACTL\_END;
> 
>     HRESULT hr = S\_OK;
>     INT32   iRetVal = 0;
> 
>     BEGIN\_ENTRYPOINT\_THROWS;
> 
>     Thread \*pThread = GetThread();
>     MethodDesc \*pMeth;
>     {
>         // This thread looks like it wandered in -- but actually we rely on it to keep the process alive.
>         pThread->SetBackground(FALSE);
>     
>         GCX\_COOP();
> 
>         pMeth = GetEntryPoint();
>         if (pMeth) {
>             RunMainPre();
>             hr = ClassLoader::RunMain(pMeth, 1, &iRetVal, stringArgs);
>         }
>     }
> 
>     //RunMainPost is supposed to be called on the main thread of an EXE,
>     //after that thread has finished doing useful work.  It contains logic
>     //to decide when the process should get torn down.  So, don't call it from
>     // AppDomain.ExecuteAssembly()
>     if (pMeth) {
>         if (stringArgs == NULL)
>             RunMainPost();
>     }
>     else {
>         StackSString displayName;
>         GetDisplayName(displayName);
>         COMPlusThrowHR(COR\_E\_MISSINGMETHOD, IDS\_EE\_FAILED\_TO\_FIND\_MAIN, displayName);
>     }
> 
>     if (FAILED(hr))
>         ThrowHR(hr);
>     END\_ENTRYPOINT\_THROWS;
>     return iRetVal;
> }   
> 
> * * *
> 
> 
> ```

   
  
    关键的步骤还是两个，准备好线程环境，然后运行 Main 方法。下面来到 clsload.cpp 中看 ClassLoader::RunMain，这也是这次我们的最后一站。  
      

>  
> 
> ```
> 代码:
> 
> * * *
> 
> HRESULT ClassLoader::RunMain(MethodDesc \*pFD ,
>                              short numSkipArgs,
>                              INT32 \*piRetVal,
>                              PTRARRAYREF \*stringArgs /\*=NULL\*/)
> {
>     STATIC\_CONTRACT\_THROWS;
>     \_ASSERTE(piRetVal);
> 
>     DWORD       cCommandArgs = 0;  // count of args on command line
>     DWORD       arg = 0;
>     LPWSTR      \*wzArgs = NULL; // command line args
>     HRESULT     hr = S\_OK;
> 
>     \*piRetVal = -1;
> 
>     // The exit code for the process is communicated in one of two ways.  If the
>     // entrypoint returns an 'int' we take that.  Otherwise we take a latched
>     // process exit code.  This can be modified by the app via setting
>     // Environment's ExitCode property.
>     if (stringArgs == NULL)
>         SetLatchedExitCode(0);
> 
>     if (!pFD) {
>         \_ASSERTE(!"Must have a function to call!");
>         return E\_FAIL;
>     }
> 
>     CorEntryPointType EntryType = EntryManagedMain;
>     ValidateMainMethod(pFD, &EntryType);
> 
>     if ((EntryType == EntryManagedMain) &&
>         (stringArgs == NULL)) {
>         // If you look at the DIFF on this code then you will see a major change which is that we
>         // no longer accept all the different types of data arguments to main.  We now only accept
>         // an array of strings.
> 
>         wzArgs = CorCommandLine::GetArgvW(&cCommandArgs);
>         // In the WindowsCE case where the app has additional args the count will come back zero.
>         if (cCommandArgs > 0) {
>             if (!wzArgs)
>                 return E\_INVALIDARG;
>         }
>     }
> 
> 
>     ETWTraceStartup::TraceEvent(ETW\_TYPE\_STARTUP\_MAIN);
>     TIMELINE\_START(STARTUP, ("RunMain"));
> 
>     EX\_TRY\_NOCATCH
>     {
>         MethodDescCallSite  threadStart(pFD);
>         
>         PTRARRAYREF StrArgArray = NULL;
>         GCPROTECT\_BEGIN(StrArgArray);
> 
>         // Build the parameter array and invoke the method.
>         if (EntryType == EntryManagedMain) {
>             if (stringArgs == NULL) {
>                 // Allocate a COM Array object with enough slots for cCommandArgs - 1
>                 StrArgArray = (PTRARRAYREF) AllocateObjectArray((cCommandArgs - numSkipArgs), g\_pStringClass);
> 
>                 // Create Stringrefs for each of the args
>                 for( arg = numSkipArgs; arg < cCommandArgs; arg++) {
>                     STRINGREF sref = COMString::NewString(wzArgs\[arg\]);
>                     StrArgArray->SetAt(arg-numSkipArgs, (OBJECTREF) sref);
>                 }
>             }
>             else
>                 StrArgArray = \*stringArgs;
>         }
> 
> #ifdef STRESS\_THREAD
>         OBJECTHANDLE argHandle = (StrArgArray != NULL) ? CreateGlobalStrongHandle (StrArgArray) : NULL;
>         Stress\_Thread\_Param Param = {pFD, argHandle, numSkipArgs, EntryType, 0};
>         Stress\_Thread\_Start (&Param);
> #endif
> 
>         ARG\_SLOT stackVar = ObjToArgSlot(StrArgArray);
> 
>         if (pFD->IsVoid()) 
>         {
>             // Set the return value to 0 instead of returning random junk
>             \*piRetVal = 0;
>             threadStart.Call(&stackVar);
>         }
>         else 
>         {
>             \*piRetVal = (INT32)threadStart.Call\_RetArgSlot(&stackVar);
>             if (stringArgs == NULL) 
>             {
>                 SetLatchedExitCode(\*piRetVal);
>             }
>         }
> 
>         GCPROTECT\_END();
> 
>         fflush(stdout);
>         fflush(stderr);
>     }
>     EX\_END\_NOCATCH
> 
>     ETWTraceStartup::TraceEvent(ETW\_TYPE\_STARTUP\_MAIN+1);
>     TIMELINE\_END(STARTUP, ("RunMain"));
> 
>     return hr;
> }
> 
> * * *
> 
> 
> ```

  
      
    这些代码主要是进行方法最终运行前的一些准备，然后运行。分两种，有返回值的和 void()的。下面的运行情况就是深入到 framework 的核心中了，改天看了再写吧。代码中运用了许多 COM 下的定义，也可见.net 和 COM 关系的密切。就像.net 下的 Debugger 和 Profiler 甚至直接调用了 COM 接口来编译。只是我对 COM 了解不深，无法就此问题深入。  
    btw:在看雪发了几篇.net 文章，主要是看雪类似的文章较少，研究的人也不多。要是有兴趣共同学习.net 的内核，欢迎和我交流。 

 4dd466d0e1e0cc123afbe504da2462ca