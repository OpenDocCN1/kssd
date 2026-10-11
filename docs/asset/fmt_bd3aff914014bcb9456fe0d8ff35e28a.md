
```cpp
return STATUS_DLL_INIT_FAILED;
    }
}
```

```cpp
//
// 如果这个进程自身的 EXE 文件定义了 TLS 变量，现在调用 TLS 初始化例程。
// 要获取更详细的信息，参考前面调用 _LdrpCallTlsInitializers 时的注释
//

if ( _LdrpImageHasTls && bImplicitLoad )
{
    _LdrpCallTlsInitializers(   pCurrentPeb->ProcessImageBase,
                                DLL_PROCESS_ATTACH );
}
}

__finally
{
    //
    // 在这个函数退出之前，确保它在前面分配的内存被释放
    //

    _RtlFreeHeap( GetProcessHeap(), 0, pInitNodeArray );
}

return STATUS_SUCCESS;
```

### 图 6 `CALC.EXE` 的 `ShowSnaps` 输出信息

```text
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
LDR: LdrGetProcedureAddress by NAME - RtlDeleteCriticalSection
//其余部分省略....
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
LDR: LdrGetProcedureAddress by NAME - RtlAllocateHeap
//其余部分省略....

注意加载器开始查找并校验 COMCTL32 导入并绑定的 DLL

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
LDR: COMCTL32.dll has correct binding to ADVAPI32.dll
//其余部分省略....
LDR: Refcount   COMCTL32.dll (1)
LDR: Refcount   GDI32.dll (3)
LDR: Refcount   KERNEL32.dll (6)
LDR: Refcount   USER32.dll (4)
LDR: Refcount   ADVAPI32.dll (5)
LDR: Refcount   KERNEL32.dll (7)
LDR: Refcount   GDI32.dll (4)
LDR: Refcount   USER32.dll (5)

List of implicit link DLLs to be init'ed.

LDR: Real INIT LIST
    C:\WINNT\system32\KERNEL32.dll init routine 77f01000
    C:\WINNT\system32\RPCRT4.dll init routine 77e1b6d5
    C:\WINNT\system32\ADVAPI32.dll init routine 77dc1000
    C:\WINNT\system32\USER32.dll init routine 77e78037
    C:\WINNT\system32\COMCTL32.dll init routine 71031a18
    C:\WINNT\system32\SHELL32.dll init routine 77c41094

开始实际调用隐含链接的 DLL 的初始化例程

LDR: KERNEL32.dll loaded. - Calling init routine at 77f01000
LDR: RPCRT4.dll loaded. - Calling init routine at 77e1b6d5
LDR: ADVAPI32.dll loaded. - Calling init routine at 77dc1000
LDR: USER32.dll loaded. - Calling init routine at 77e78037

USER32 开始做与 AppInit_DLLs 有关的工作，因此静态初始化被暂时中断
这个例子中，“globaldll.dll”是在 USER32 的初始化代码中由 LoadLibrary 加载的

LDR: LdrLoadDll, loading c:\temp\globaldll.dll from C:\WINNT\system32;.;
LDR: Loading (DYNAMIC) c:\temp\globaldll.dll
Loaded 'C:\TEMP\GlobalDLL.dll', no matching symbolic information found.
LDR: KERNEL32.dll used by globaldll.dll
//其余部分省略....

LDR: Real INIT LIST
    c:\temp\globaldll.dll init routine 10001310
LDR: globaldll.dll loaded. - Calling init routine at 10001310

现在接着初始化隐含链接的 DLL

LDR: COMCTL32.dll loaded. - Calling init routine at 71031a18
LDR: LdrGetDllHandle, searching for USER32.dll from
LDR: LdrGetProcedureAddress by NAME - GetSystemMetrics
LDR: LdrGetProcedureAddress by NAME - MonitorFromWindow
LDR: SHELL32.dll loaded. - Calling init routine at 77c41094
//其余部分省略....
```

### 图 7 `TLSInit.cpp`

```cpp
void _LdrpCallTlsInitializers( HMODULE hModule, DWORD fdwReason )
{
    PIMAGE_TLS_DIRECTORY pTlsDir;
    DWORD size

    // 从 IMAGE_OPTIONAL_HEADER.DataDirectory 中查找 TLS 目录
    pTlsDir = _RtlImageDirectoryEntryToData(hModule,
                                             1,
                                             IMAGE_DIRECTORY_ENTRY_TLS,
                                             &size );

    __try   // 用 try/catch 块保护所有代码
    {
        if ( pTlsDir->AddressOfCallbacks )
        {
            if ( _ShowSnaps )   // 输出诊断信息
            {
                _DbgPrint(  "LDR: Tls Callbacks Found. "
                            "Imagebase %lx Tls %lx CallBacks %lx\n",
                            hModule, TlsDir, pTlsDir->AddressOfCallbacks );
            }

            // 获取指向包含 TLS 回调函数地址的数组的起始位置的指针
            PVOID * pCallbacks = pTlsDir->AddressOfCallbacks;

            while ( *pCallbacks )   // 遍历数组中的每一个元素
            {
                PIMAGE_TLS_CALLBACK pTlsCallback = *pCallbacks;
                pCallbacks++;

                if ( _ShowSnaps )   // 输出更多诊断信息
                {
                    _DbgPrint(  "LDR: Calling Tls Callback "
                                "Imagebase %lx Function %lx\n",
                                hModule, pTlsCallback );
                }

                // 实际调用回调函数
                pTlsCallback( hModule, fdwReason, 0 );
            }
        }
    }
    __except( EXCEPTION_EXECUTE_HANDLER )
    {
    }
}
```

[Microsoft System Journal 1999 年 9 月 Under The Hood 专栏](http://www.microsoft.com/msj/0999/hood/hood0999.aspx)

译者：SmartTech  电子信箱：zhzhtst@163.com

`4dd466d0e1e0cc123afbe504da2462ca`
