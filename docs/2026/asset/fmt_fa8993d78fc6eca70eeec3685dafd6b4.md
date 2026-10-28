
```cpp
if (fBreakOnDllLoad)
{
    if (_ShowSnaps)
    {
        // 在实际中断进调试器之前，输出模块名称和初始化例程的地址
        _DbgPrint("LDR: %wZ loaded.", &pModuleLoaderInfo->pModuleLoaderInfo);
        _DbgPrint("- About to call init routine at %lx\n", pfnInitRoutine)
    }

    // 中断进调试器
    _DbgBreakPoint(); // 它实际是一条INT 3指令，后面跟着RET指令
}
else if (_ShowSnaps && pfnInitRoutine)
{
    // 在调用初始化例程之前输出模块名称和初始化例程的地址
    _DbgPrint("LDR: %wZ loaded.", pModuleLoaderInfo->pModuleLoaderInfo);
    _DbgPrint("- Calling init routine at %lx\n", pfnInitRoutine);
}

if (pfnInitRoutine)
{
    // 设置标志来表明已将DLL_PROCESS_ATTACH通知发送给了DLL
    //
    // （难道这不应该是在实际调用初始化例程之后才设置吗？）
    //
    // X_LOADER_CALLED_PROCESS_ATTACH = 0x8

    pModuleLoaderInfo->Flags36 |= X_LOADER_CALLED_PROCESS_ATTACH;

    //
    // 如果此模块使用了线程局部存储（TLS），现在调用TLS初始化函数
    // *** 注意 ***
    // 这仅发生在一个进程首次调用此函数时（也就是在初始化隐含
    // 链接的DLL时)。动态加载的DLL不应该使用TLS变量，正如
    // SDK文档所说的那样
    //
    if (pModuleLoaderInfo->bHasTLS && bImplicitLoad)
    {
        _LdrpCallTlsInitializers(pModuleLoaderInfo->hModDLL, DLL_PROCESS_ATTACH);
    }

    hModDLL = pModuleLoaderInfo->hModDLL
    MOV     ESI, ESP // 将ESP寄存器的值保存到ESI中
    MOV     EDI, DWORD PTR [pfnInitRoutine] // 将模块的入口点
    //地址加载到EDI中

    //以下的汇编语言代码用C++代码表示就是：
    //
    // initRetValue =
    // pfnInitRoutine(hInstDLL,DLL_PROCESS_ATTACH,bImplicitLoad);
    //
    PUSH    DWORD PTR [bImplicitLoad]
    PUSH    DLL_PROCESS_ATTACH
    PUSH    DWORD PTR [hModDLL]
    CALL    EDI // 调用初始化例程。这是设置断点的最佳位置。
                // 单步跟踪这个调用就进入到了DLL的入口点中

    MOV     BYTE PTR [initRetValue], AL // 保存入口点函数的返回值
    MOV     DWORD PTR [_SaveSp], ESI // 保存入口点函数返回后的
    MOV     DWORD PTR [_CurSp], ESP // 堆栈指针的值
    MOV     ESP, ESI // 恢复调用入口点函数之前ESP中的值

    //
    // 校验调用前后堆栈指针（ESP）的值。如果它们不同，这
    // 表明DLL的初始化例程并没有正确地清理堆栈。例如，它的
    // 入口点函数可能定义的不正确。尽管这极少发生，但是如果
    // 它确实发生了，我们要通知用户并让他们决定是否继续执行
    //
    if (_CurSP != _SavSP)
    {
        hardErrorParam = pModuleLoaderInfo->FullDllPath;
        hardErrorRetCode =
            _NtRaiseHardError(
                STATUS_BAD_DLL_ENTRYPOINT | 0x10000000,
                1, // 参数个数
                1, // UnicodeStringParametersMask,
                &hardErrorParam,
                OptionYesNo, // 让用户决定
                &hardErrorResponse);

        if (_LdrpInLdrInit)
            _LdrpFatalHardErrorCount++;

        if ((hardErrorRetCode >= STATUS_SUCCESS)
            && (ResponseYes == hardErrorResponse))
        {
            return STATUS_DLL_INIT_FAILED;
        }
    }

    //
    // 如果DLL的入口点函数返回0（表示失败），通知用户
    //
    if (0 == initRetValue)
    {
        DWORD hardErrorParam2;
        DWORD hardErrorResponse2;

        hardErrorParam2 = pModuleLoaderInfo->FullDllPath;
        _NtRaiseHardError(
            STATUS_DLL_INIT_FAILED,
            1, // 参数个数
            1, // UnicodeStringParametersMask
            &hardErrorParam2,
            OptionOk, // 只能以“确定”作为响应
            &hardErrorResponse2);

        if (_LdrpInLdrInit)
            _LdrpFatalHardErrorCount++;

        return STATUS_DLL_INIT_FAILED;
    }
}

//

// 如果这个进程自身的EXE文件定义了TLS变量，现在调用TLS初始化例程。
// 要获取更详细的信息，参考前面调用_LdrpCallTlsInitializers时的注释
//
if (_LdrpImageHasTls && bImplicitLoad)
{
    _LdrpCallTlsInitializers(pCurrentPeb->ProcessImageBase, DLL_PROCESS_ATTACH);
}
}
__finally
{
    //
    // 在这个函数退出之前，确保它在前面分配的内存被释放
    //
    _RtlFreeHeap(GetProcessHeap(), 0, pInitNodeArray);
}

return STATUS_SUCCESS;
}
```

返回

图6 CALC.EXE的ShowSnaps输出信息

## 以##开头的是我的注释

```
Loaded 'C:\WINNT\system32\CALC.EXE', no matching symbolic information found.

Loaded symbols for 'C:\WINNT\system32\ntdll.dll'

LDR: PID: 0x3a started - '"C:\WINNT\system32\CALC.EXE"'

LDR: NEW PROCESS

     Image Path: C:\WINNT\system32\CALC.EXE (CALC.EXE)

     Current Directory: C:\WINNT\system32

     Search Path: C:\WINNT\system32;.;C:\WINNT\System32;C:\WINNT\system;...

LDR: SHELL32.dll used by CALC.EXE

Loaded 'C:\WINNT\system32\SHELL32.DLL', no matching symbolic information found.

LDR: ntdll.dll used by SHELL32.dll

LDR: Snapping imports for SHELL32.dll from ntdll.dll

LDR: KERNEL32.dll used by SHELL32.dll

Loaded symbols for 'C:\WINNT\system32\KERNEL32.DLL'

LDR: ntdll.dll used by KERNEL32.dll

LDR: Snapping imports for KERNEL32.dll from ntdll.dll

LDR: Snapping imports for SHELL32.dll from KERNEL32.dll

LDR: LdrLoadDll, loading NTDLL.dll from

LDR: LdrGetProcedureAddress by NAME - RtlEnterCriticalSection

LDR: LdrLoadDll, loading NTDLL.dll from

LDR: LdrGetProcedureAddress by NAME - RtlDeleteCriticalSection//其余部分省略....

LDR: GDI32.dll used by SHELL32.dll

Loaded symbols for 'C:\WINNT\system32\GDI32.DLL'

LDR: ntdll.dll used by GDI32.dll

LDR: Snapping imports for GDI32.dll from ntdll.dll

LDR: KERNEL32.dll used by GDI32.dll

LDR: Snapping imports for GDI32.dll from KERNEL32.dll

LDR: USER32.dll used by GDI32.dll

Loaded symbols for 'C:\WINNT\system32\USER32.DLL'

LDR: ntdll.dll used by USER32.dll

LDR: Snapping imports for USER32.dll from ntdll.dll

LDR: KERNEL32.dll used by USER32.dll

LDR: Snapping imports for USER32.dll from KERNEL32.dll

LDR: LdrLoadDll, loading NTDLL.dll from

LDR: LdrGetProcedureAddress by NAME - RtlSizeHeap

LDR: LdrLoadDll, loading NTDLL.dll from

LDR: LdrGetProcedureAddress by NAME - RtlReAllocateHeap

LDR: LdrLoadDll, loading NTDLL.dll from

LDR: LdrGetProcedureAddress by NAME - RtlFreeHeap

LDR: LdrLoadDll, loading NTDLL.dll from

LDR: LdrGetProcedureAddress by NAME - RtlAllocateHeap//其余部分省略....

## 注意加载器开始查找并校验COMCTL32导入并绑定的DLL

Loaded 'C:\WINNT\system32\COMCTL32.DLL', no matching symbolic information found.

LDR: COMCTL32.dll bound to ntdll.dll

LDR: COMCTL32.dll has correct binding to ntdll.dll

LDR: COMCTL32.dll bound to GDI32.dll

LDR: COMCTL32.dll has correct binding to GDI32.dll

LDR: COMCTL32.dll bound to KERNEL32.dll

LDR: COMCTL32.dll has correct binding to KERNEL32.dll

LDR: COMCTL32.dll bound to ntdll.dll via forwarder(s) from KERNEL32.dll

LDR: COMCTL32.dll has correct binding to ntdll.dll

LDR: COMCTL32.dll bound to USER32.dll

LDR: COMCTL32.dll has correct binding to USER32.dll

LDR: COMCTL32.dll bound to ADVAPI32.dll

LDR: COMCTL32.dll has correct binding to ADVAPI32.dll//其余部分省略....

LDR: Refcount   COMCTL32.dll (1)

LDR: Refcount   GDI32.dll (3)
```
