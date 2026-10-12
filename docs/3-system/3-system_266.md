
# 乱解 API 函数 —— API 绝密档案系列之二

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-200.htm`](https://www.pediy.com/kssd/pediy08/pediy8-200.htm)

上一篇介绍了两个结构，今后你会发现，在系统程序中，对这两个结构的引用无所不在，理解这两个结构对正确理解系统程序有莫大的帮助。

这一篇加了副标题【通向高手之路】你可能要问，我是什么“手”，抱歉，我最多只能称“中手”，那么“中手”是什么意思？ 是高手之下，低手之上的那只手？ 错，“中手”－－乃中国之手－－乃国手也，哈哈。

那么我们为什么要研究这些底层的东西呢？这个问题提的好。一般来说有两个目的，第一是深刻的理解这些函数是如何工作的，在今后的开发中你也许可以借鉴这些底层的某些用法，来优化你的程序。其二就是弄一些自己的函数来取代这些函数从而达到某些特殊的目的。

对于如何用自己的代码来取代这些函数也有许多方法，最简单的就是将这些代码直接拷贝到你的程序中去，但这可能不是很好的办法，因为这些代码中，有许多函数是内部函数，不能直接调用,因而你不得不将这些所有的过程都搬到你的程序中。另一种方法就是在真正理解这些函数的基础上彻底改写。系统函数一般来讲都比较全面，各种情况都要考虑，所以比较繁琐，而你可能只是在某些特定的环境下使用这些函数，这就有可能简化这些函数，将你认为不需要的地方去掉，这样代码精炼，而且隐蔽性更好，如果更上一层楼，可以使用其他的方法达到相同的目的，这样就是最高境界了。

我在这里以 `GetModuleHandle` 函数为例，介绍了如何分析理解这些函数，Windows 从公开的到未公开的函数多如牛毛，不可能每一个的介绍，我没时间写，你也没有时间看，所以这里只是以这个函数为例，让你了解一下这些分析的过程，希望你能掌握这一思路和方法，自己去分析底层函数，这样你就有可能成为真正的高手。

## 首先我们来看看 GetModuleHandleW 的代码

(这里给出的是 `TimeDateStamp=0x42A0669B` 版)

```assembly
; Exported entry 320. GetModuleHandleW
; ======== S U B R O U T I N E ====================
; HMODULE __stdcall GetModuleHandleW(LPCWSTR lpModuleName)
public _GetModuleHandleW@4
_GetModuleHandleW@4 proc near
UnicodeModuleName= dword ptr -10h
ModuleName= byte ptr -8
MemoryPointer= dword ptr -4
lpModuleName= dword ptr 8
    push ebp
    mov ebp, esp
    sub esp, 10h
    cmp [ebp+lpModuleName], 0
    push esi
    jnz short IfNotSelfModuleHandle
    mov eax, large fs:18h               ; TEB.NT_TIB.Self
    mov eax, [eax+TEB.Peb]
    mov eax, [eax+PEB.ImageBaseAddress]
    jmp short GetModHandleExit
IfNotSelfModuleHandle:                  ; SourceString
    push [ebp+lpModuleName]
    lea eax, [ebp+UnicodeModuleName]
    push eax                              ; DestinationString
    call ds:__imp__RtlInitUnicodeString@8 ; 将 CWSTR 转成 Unicode
    cmp _gDoDllRedirection, 0           ; Dll 重定向标志
    jz short DontDllRedir
    ; _gDoDllRedirection = TRUE
    lea eax, [ebp+ModuleName]
    push eax                              ; returnName
    lea eax, [ebp+UnicodeModuleName]
    push eax                              ; lpModuleName
    call _ComputeRedirectedDllName@8    ; 所谓重定向就是到用户程序的目录下加载同名的 dll，
                                        ; _ComputeRedirectedDllName 函数将用户输入的 ModuleName
                                        ; 前面加上用户的目录(用户程序启动的目录)。
    test eax, eax
    jge short IfGetRedirDllName
    push eax                              ; Status
    call _BaseSetLastNTError@4          ; BaseSetLastNTError(x)
    xor eax, eax
    jmp short GetModHandleExit
IfGetRedirDllName:
    lea eax, [ebp+ModuleName]
    push eax                              ; lpModuleName
    call _GetModuleHandleForUnicodeString@4 ; GetModuleHandleForUnicodeString(x)
    mov esi, eax
    mov eax, large fs:18h               ; TEB.NT_TIB.Self
    push [ebp+MemoryPointer]            ; MemoryPointer
    mov eax, [eax+TEB.Peb]
    push 0                               ; Flags
    push [eax+PEB.ProcessHeap]          ; HeapHandle
    call ds:__imp__RtlFreeHeap@12      ; __declspec(dllimport) RtlFreeHeap(x,x,x)
    test esi, esi
    jz short DontDllRedir
    mov eax, esi
    jmp short GetModHandleExit
    ; _gDoDllRedirection = FALSE
DontDllRedir:
    lea eax, [ebp+UnicodeModuleName]
    push eax                              ; lpModuleName
    call _GetModuleHandleForUnicodeString@4 ; GetModuleHandleForUnicodeString(x)
GetModHandleExit:
    pop esi
    leave
    retn 4
_GetModuleHandleW@4 endp
```

### ModuleHandle 到底是什么？

其实，在第一篇中已经有了答案，`ModuleHandle` 是运行程序加载的基地址，即：

`ModuleHandle == ImageBaseAddress`

这个 `ImageBaseAddress` 和 PE 头那个 `ImageBaseAddress` 有点不同，PE 头中的 `ImageBaseAddress` 是由编译器生成的，这个地址可以由用户设定，例如在汇编程序的连接中，就可以通过如下的方法来设定 `ImageBaseAddress`:

`LINK.EXE /base:0x500000 /SUBSYSTEM:WINDOWS /OUT:Prog.exe,Prog.obj,Prog.res`

这里我们指定 `ImageBase` 为 `0x500000`，这个参数如果不设置，缺省值是 `0x400000` (对 exe 文件,连接驱动程序这个参数不起作用）操作系统在加载用户程序时，将以这个参数为首选，如果该地址没有加载其他程序，用户程序将加载到这个地址，这时 PE 头中的 `ImageBaseAddress` 就和 `ModuleHandle` 相同，Win32 运行环境是每个进程独享 4GB(去掉操作系统所占实际为 2GB 空间），所以一般情况下，对于用户程序而言，`ImageBaseAddress` 总是和 `ModuleHandle` 相同，对于 Dll 和 Sys 而言，自己编写的模块，加载地址和 PE 头文件的地址多数是不同的，而老盖的系统模块则基本相同(加载到固定的地址)。

## 代码分析

现在我们将分析上面的代码：

`GetModuleHandleW` 根据 `lpModuleName` 分两种方式来处理

### 1、lpModuleName = Null

这个我们在第一篇中已经讨论过，即简单的从 `TEB` -- `PEB` -- `ImageBaseAddress` 来获得当前进程的加载地址。具体代码如下：

```assembly
    mov eax, large fs:18h               ; TEB.NT_TIB.Self
    mov eax, [eax+TEB.Peb]
    mov eax, [eax+PEB.ImageBaseAddress]
```

### 2、lpModuleName != Null

当 `lpModuleName` 不为空时，又分为两种情况，其实在稍早一些的版本中，只有一种情况，新的版本才多出这一种。

```assembly
    cmp _gDoDllRedirection, 0           ; Dll 重定向标志
    jz short DontDllRedir
```

即根据 `_gDoDllRedirection` 是否为零来分枝，`_gDoDllRedirection` 是一布尔变量，下面我们先看看这个 `_gDoDllRedirection` 是怎么来的。

对于用户程序，Win 操作系统在加载用户程序时会根据输入表加载相应的模块，如果使用了 `Kernel32.dll` 中的函数，则 `Kernel32.dll` 将被加载（如果 `Kernel32` 已经加载，则在用户空间会产生一个映射），在 `Kernel32` 加载时初始化了这个 `_gDoDllRedirection` 布尔变量。我们摘录部分代码分析，下面是 `Kernel32.dll` 初始化部分：

```assembly
; NTSTATUS __stdcall BaseDllInitialize(HINSTANCE hInstance,DWORD dwReason,LPVOID lpReserved)
_BaseDllInitialize@12 proc near
pObjectDirectory= dword ptr -210h
PEB   = dword ptr -10h
pConnectionInfo= dword ptr -0Ch
@ConnectionInfoSize= dword ptr -8
@ServerToServerCall= dword ptr -1
hInstance= dword ptr  8
dwReason= dword ptr  0Ch
lpReserved= dword ptr  10h
    push ebp
    mov ebp, esp
    sub esp, 210h
... ...
... ...
loc_77E67C66:
    push 104h                       ; nSize
    push offset _BaseDefaultPathBuffer  ; lpFilename
    push esi                        ; hModule esi=Null 获取当前用户带路径的程序名
    call _GetModuleFileNameW@12         ; GetModuleFileNameW(x,x,x)
    movzx eax, ax                   ; 返回文件名的长度
    mov esi, offset a_local         ; ".Local"
    lea edi, _BaseDefaultPathBuffer[eax*2] ;将指针移到文件名字符串的尾部
    movsd
    movsd
    movsd
    movsw
    mov esi, offset _BaseDefaultPathBuffer
    push esi                         ; FileName
    call ds:__imp__RtlDoesFileExists_U@4 ; __declspec(dllimport) RtlDoesFileExists_U(x)
    mov edi, src
    mov _gDoDllRedirection, al
... ...
... ...
```

首先是调用 `GetModuleFileNameW` 因为参数 `hModule=Null` 返回用户程序名。然后再对用户程序名进行简单的处理，在尾巴添加 “.Local”，实际就是将所谓的 `DOSPathName` 转变为 `NtPathName`，处理过的文件名如下

```text
C:\MyDir\MyProc.exe.Local (Nt 格式的文件名，注意，是 WSTR 字符串)
```

然后调用 `RtlDoesFileExists_U` 函数，代码如下：

```text
...
```



`RtlDoesFileExists_U` 直接调用 `RtlDoesFileExists_UEx`，代码如下：

```
代码:

* * *

77F895D0 ; Exported entry 430. RtlDoesFileExists_U
77F895D0 ; ======== S U B R O U T I N E ====================
77F895D0 ; BOOLEAN __stdcall RtlDoesFileExists_U(PWSTR FileName)
77F895D0       public _RtlDoesFileExists_U@4
77F895D0 _RtlDoesFileExists_U@4 proc near
77F895D0 FileName= dword ptr  4
77F895D0       push TRUE                               ; AllowShare
77F895D2       push [esp+4+FileName]                   ; FileName
77F895D6       call _RtlDoesFileExists_UEx
77F895DB       retn 4
77F895DB _RtlDoesFileExists_U@4 endp

* * *
```

下面是 `RtlDoesFileExists_UEx` 的相关部分:

```
代码:

* * *

77F89524 ; BOOLEAN __stdcall _RtlDoesFileExists_UEx(PWSTR FileName,BOOLEAN AllowShare)
77F89524 _RtlDoesFileExists_UEx proc near
77F89524
77F89524 FileInformation= _FILE_BASIC_INFORMATION ptr -54h
77F89524 var_30= dword ptr -30h
77F89524 ObjectAttributes= OBJECT_ATTRIBUTES ptr -2Ch
77F89524 DirectoryInfo= dword ptr -14h
77F89524 CurDir= dword ptr -10h
77F89524 Status= dword ptr -0Ch
77F89524 NtPathName= UNICODE_STRING ptr -8
77F89524 FileName= dword ptr  8
77F89524 AllowShare= byte ptr  0Ch
77F89524
77F89524       push ebp
77F89525       mov ebp, esp
77F89527       sub esp, 54h
... ...
... ...
77F89564       mov eax, [ebp+Status]
77F89567       mov [ebp+ObjectAttributes.Length], 18h      ; sizeof(OBJECT_ATTRIBUTES)
77F8956E       mov [ebp+ObjectAttributes.RootDirectory], eax
77F89571       lea eax, [ebp+NtPathName]
77F89574       mov [ebp+ObjectAttributes.ObjectName], eax
77F89577       lea eax, [ebp+FileInformation]
77F8957A       push eax                            ; FileInformation
77F8957B       lea eax, [ebp+ObjectAttributes]
77F8957E       push eax                            ; ObjectAttributes
77F8957F       mov [ebp+ObjectAttributes.Attributes], OBJ_CASE_INSENSITIVE
77F89586       mov [ebp+ObjectAttributes.SecurityDescriptor], ebx
77F89589       mov [ebp+ObjectAttributes.SecurityQualityOfService], ebx
77F8958C       call _ZwQueryAttributesFile@8               ; ZwQueryAttributesFile(x,x)
77F89591       mov esi, eax                        ;ZwQueryAttributesFile 返回结果为
... ...
... ...
77F895AA       cmp esi, STATUS_SHARING_VIOLATION         
77F895B0       jz short loc_77F895BE
77F895B2       cmp esi, STATUS_ACCESS_DENIED
77F895B8       jz short loc_77F895BE
77F895BA       xor al, al
77F895BC       jmp short loc_77F895C8
... ...

* * *
```

`RtlDoesFileExists_UEx` 调用 Ring0 层函数 `ZwQueryAttributesFile`，`ntdll.dll` 包装了这个底层函数，具体如下：

```
代码:

* * *

77F8880C ; NTSTATUS __stdcall ZwQueryAttributesFile(POBJECT_ATTRIBUTES ObjectAttributes,PFILE_BASIC_INFORMATION FileInformation)
77F8880C        public _ZwQueryAttributesFile@8
77F8880C _ZwQueryAttributesFile@8 proc near
77F8880C
77F8880C ObjectAttributes= dword ptr  4
77F8880C FileInformation= dword ptr  8
77F8880C
77F8880C        mov   eax, 7Ah                       ; NtQueryAttributesFile
77F88811        lea   edx, [esp+ObjectAttributes]
77F88815        int   2Eh                          ; DOS 2+ internal - EXECUTE COMMAND
77F88815                                   ; DS:SI -> counted CR-terminated command string
77F88817        retn  8
77F88817 _ZwQueryAttributesFile@8 endp

* * *
```

当 `ZwQueryAttributesFile` 返回结果为 `STATUS_SHARING_VIOLATION` 或 `STATUS_ACCESS_DENIED` 时 `_gDoDllRedirection = 1` (`TRUE`)，其余情况为 `0`。所以用户程序返回结果多数为 `0` (`FALSE`)。

老盖为了防止用户直接调 `int 2Eh` (`KiSystemService`)，每次 `Updata` 功能号都可能发生改变，另外在 `ntoskrnl.exe` 的输出表中，列出了许多函数，供驱动程序使用，但有些 Ring3 使用的函数，例如这个 `ZwQueryAttributesFile` 就没有列出在输出表中，属于 `ntoskrnl.exe` 的内部函数。在 `IDA` 中可以通过 `Names` 窗口，很容易的找到这个函数。如果想看 `ZwQueryAttributesFile` 的代码，还需要将前面的 `Zw` 换成 `Nt`，即 `NtQueryAttributesFile`。凡是标有 `Zw` 字头的函数，多数是通过 `int 2Eh` (`KiSystemService`) 而进入 `Nt` 相应名称的函数，当然也有部分的 `Nt` 起头的函数通过 `int 2Eh` (`KiSystemService`) 来转向，大多数都是因为函数本身实现不在 `ntoskrnl.exe` 内部，而在其他模块。功能号为 `Byte`，多数在内部实现，功能号为 `Word` 则多数在外部模块实现，所以跟踪一个函数，可能会跳两三个模块才能找到实现的代码。

### `2.1 _gDoDllRedirection = TRUE`

```
代码:

* * *

77E80C58       lea eax, [ebp+ModuleName]
77E80C5B       push eax                    ; returnName
77E80C5C       lea eax, [ebp+UnicodeModuleName]
77E80C5F       push eax                    ; lpModuleName
77E80C60       call _ComputeRedirectedDllName@8    ; 所谓重定向就是到用户程序的目录下加载同名的 dll，
77E80C60                       ; _ComputeRedirectedDllName 函数将用户输入的 ModuleName
77E80C60                       ; 前面加上用户的目录(用户程序启动的目录)。
77E80C65       test eax, eax
77E80C67       jge short IfGetRedirDllName
77E80C69       push eax                    ; Status
77E80C6A       call _BaseSetLastNTError@4           ; BaseSetLastNTError(x)
77E80C6F       xor eax, eax
77E80C71       jmp short GetModHandleExit
77E80C73 IfGetRedirDllName:
77E80C73       lea eax, [ebp+ModuleName]
77E80C76       push eax                    ; lpModuleName
77E80C77       call _GetModuleHandleForUnicodeString@4 ; GetModuleHandleForUnicodeString(x)
77E80C7C       mov esi, eax
77E80C7E       mov eax, large fs:18h                  ; TEB.NT_TIB.Self
77E80C84       push [ebp+MemoryPointer]            ; MemoryPointer
77E80C87       mov eax, [eax+TEB.Peb]
77E80C8A       push 0                           ; Flags
77E80C8C       push [eax+PEB.ProcessHeap]           ; HeapHandle
77E80C8F       call ds:__imp__RtlFreeHeap@12                 ; __declspec(dllimport) RtlFreeHeap(x,x,x)
77E80C95       test esi, esi
77E80C97       jz short DontDllRedir
77E80C99       mov eax, esi
77E80C9B       jmp short GetModHandleExit

* * *
```

`_gDoDllRedirection = TRUE` 首先调用 `ComputeRedirectedDllName` 函数，该函数通过调用 `GetModuleFileNameW` 获得用户程序的目录，其后是字符串处理，将用户的 `lpModuleName` 加在用户的目录之后。这里就不列出这段代码了。

然后调用 `GetModuleHandleForUnicodeString`，因为在 `ComputeRedirectedDllName` 函数中分配了堆栈，所以后面调用 `RtlFreeHeap` 释放堆栈。`GetModuleHandleForUnicodeString` 将在下面一起说明。

如果发生错误，将调用 `BaseSetLastNTError` ，代码如下：

```
代码:

* * *

77E68239 _BaseSetLastNTError@4 proc near
77E68239 Status= dword ptr  8
77E68239       push esi
77E6823A       push [esp+Status]                   ; Status
77E6823E       call ds:__imp__RtlNtStatusToDosError@4 ; __declspec(dllimport) RtlNtStatusToDosError(x)
77E68244       mov esi, eax
77E68246       push esi                    ; Status
77E68247       call _SetLastError@4                 ; SetLastError(x)
77E6824C       mov eax, esi
77E6824E       pop esi
77E6824F       retn 4
77E6824F _BaseSetLastNTError@4 endp

77E68252 ; Exported entry 675. SetLastError
77E68252 ; ======== S U B R O U T I N E ====================
77E68252 ; int __stdcall SetLastError(NTSTATUS Status)
77E68252       public _SetLastError@4
77E68252 _SetLastError@4 proc near
77E68252 Status= dword ptr  8
77E68252
77E68252       push ebp
77E68253       mov ebp, esp
77E68255       mov eax, large fs:18h
77E6825B       mov ecx, [ebp+Status]
77E6825E       mov [eax+TEB.LastErrorValue], ecx
77E68261       pop ebp
77E68262       retn 4
77E68262 _SetLastError@4 endp

* * *
```

内部函数 `BaseSetLastNTError` 调用 `RtlNtStatusToDosError` 将 Nt 的错误代码转换成 Dos 错误代码，然后调用 `SetLastError`，上面代码显示，`SetLastError` 就是将错误代码存入 `TEB.LastErrorValue`。

### `2.2 _gDoDllRedirection = FALSE`

```
代码:

* * *

        ;_gDoDllRedirection = FALSE
77E80C9D DontDllRedir:
77E80C9D       lea eax, [ebp+UnicodeModuleName]
77E80CA0       push eax                    ; lpModuleName
77E80CA1       call _GetModuleHandleForUnicodeString@4 ; GetModuleHandleForUnicodeString(x)

* * *
```



不管走那条路，最后都是调用内部函数 `GetModuleHandleForUnicodeString`，代码如下：

```
代码:

* * *

77E80B49 ; BOOLEAN __stdcall GetModuleHandleForUnicodeString(LPCTSTR lpModuleName)
77E80B49 _GetModuleHandleForUnicodeString@4 proc near
77E80B49
77E80B49 TEB   = dword ptr -30h
77E80B49 var_2C= dword ptr -2Ch
77E80B49 pHModule= dword ptr -28h
77E80B49 NtStatus1= dword ptr -20h
77E80B49 MemoryPointer= dword ptr -1Ch
77E80B49 storeESP= dword ptr -18h
77E80B49 var_14= dword ptr -14h
77E80B49 var_10= dword ptr -10h
77E80B49 RetStatus= dword ptr -4
77E80B49 lpModuleName= dword ptr  8
77E80B49
77E80B49       push ebp
77E80B4A       mov ebp, esp
77E80B4C       push 0FFFFFFFFh
77E80B4E       push offset dword_77E62550
77E80B53       push offset __except_handler3
77E80B58       mov eax, large fs:0
77E80B5E       push eax
77E80B5F       mov large fs:0, esp
77E80B66       push ecx
77E80B67       push ecx
77E80B68       sub esp, 1Ch
77E80B6B       push ebx
77E80B6C       push esi
77E80B6D       push edi
77E80B6E       mov [ebp+storeESP], esp
77E80B71       lea eax, [ebp+pHModule]
77E80B74       push eax                            ; pHModule
77E80B75       push [ebp+lpModuleName]            ; ModuleFileName
77E80B78       push 0                             ; Unused
77E80B7A       push 1                             ; DllPath
77E80B7C       call _LdrGetDllHandle@16           ; LdrGetDllHandle(x,x,x,x)
77E80B81       test eax, eax
77E80B83       jge loc_77E80C13
77E80B89       call _GetEnvironmentStringsW@0      ; GetEnvironmentStringsW()
77E80B8E       mov esi, eax
77E80B90       test esi, esi
77E80B92       jz short loc_77E80BAB
77E80B94       push esi                            ; Environment
77E80B95       push FALSE                         ; BOOLEAN
77E80B97       call _BaseComputeProcessDllPath@8   ; BaseComputeProcessDllPath(x,x)
77E80B9C       mov ebx, eax
77E80B9E       mov [ebp+MemoryPointer], ebx
77E80BA1       push esi
77E80BA2       call _FreeEnvironmentStringsA@4     ; FreeEnvironmentStringsA(x)
77E80BA7       test ebx, ebx
77E80BA9       jnz short loc_77E80BB2
77E80BAB loc_77E80BAB:
77E80BAB       mov esi, STATUS_NO_MEMORY
77E80BB0       jmp short loc_77E80C05
77E80BB2 loc_77E80BB2:
77E80BB2       and [ebp+RetStatus], 0
77E80BB6       lea eax, [ebp+pHModule]
77E80BB9       push eax                            ; pHModule
77E80BBA       push [ebp+lpModuleName]            ; ModuleFileName
77E80BBD       push 0                             ; Unused
77E80BBF       push ebx                            ; DllPath
77E80BC0       call _LdrGetDllHandle@16           ; LdrGetDllHandle(x,x,x,x)
77E80BC5       mov esi, eax
77E80BC7       mov [ebp+NtStatus1], esi           ; NTSTATUS
77E80BCA       mov eax, large fs:18h
77E80BD0       mov [ebp+TEB], eax
77E80BD3       push ebx                            ; MemoryPointer
77E80BD4       jmp short loc_77E80BF3
77E80BD6 loc_77E80BD6:
77E80BD6       mov eax, [ebp+var_14]
77E80BD9       mov eax, [eax]
77E80BDB       mov eax, [eax]
77E80BDD       mov [ebp+var_2C], eax
77E80BE0       push TRUE
77E80BE2       pop eax
77E80BE3       retn
77E80BE4
77E80BE4 loc_77E80BE4:
77E80BE4       mov esp, [ebp+storeESP]
77E80BE7       mov esi, [ebp+var_2C]
77E80BEA       mov eax, large fs:18h
77E80BF0       push [ebp+MemoryPointer]           ; MemoryPointer
77E80BF3 loc_77E80BF3:
77E80BF3       push 0                             ; Flags
77E80BF5       mov eax, [eax+TEB.Peb]
77E80BF8       push [eax+PEB.ProcessHeap]         ; HeapHandle
77E80BFB       call ds:__imp__RtlFreeHeap@12      ; __declspec(dllimport) RtlFreeHeap(x,x,x)
77E80C01       or [ebp+RetStatus], 0FFFFFFFFh
77E80C05 loc_77E80C05:
77E80C05       test esi, esi
77E80C07       jge short loc_77E80C13
77E80C09       push esi                            ; Status
77E80C0A       call _BaseSetLastNTError@4         ; BaseSetLastNTError(x)
77E80C0F       xor eax, eax
77E80C11       jmp short loc_77E80C16
77E80C13 loc_77E80C13:
77E80C13       mov eax, [ebp+pHModule]
77E80C16 loc_77E80C16:
77E80C16       mov ecx, [ebp+var_10]
77E80C19       mov large fs:0, ecx
77E80C20       pop edi
77E80C21       pop esi
77E80C22       pop ebx
77E80C23       leave
77E80C24       retn 4
77E80C24 _GetModuleHandleForUnicodeString@4 endp

* * *
```

`GetModuleHandleForUnicodeString` 头部涉及一个 C 的异常处理过程，网上有许多这方面的文章，这里就不加解释了，（无非就是抄一遍），这是一篇英文的资料，写的非常详细，`http://www.codeproject.com/cpp/exceptionhandler.asp`，网上好像有中文的翻译版，你可以自己 `google` 一把。

`GetModuleHandleForUnicodeString` 实际就是调用 `_LdrGetDllHandle` 函数，该函数在 `ntdll.dll` 中，但最后通过调 `NtOpenFile` 等一系列的底层函数，完成，其过程非常复杂，代码超过 1000 行，这里就不列出了。

如果调用失败，调用 `GetEnvironmentStringsW` 获得环境串，遍列环境串中的所有目录，重复调用 `_LdrGetDllHandle`，直到成功或失败。

### 后话

这一篇写的死气沉沉，何也，心情不好，也没有时间，在匆忙中草率完成，所以无法再笑解了，不哭解已经不错了。我的 House 终于找到租客，原本是好事，可惜每天忙着整理东西，然后想蚂蚁那样，一件一件的搬到 Basement 去，孤家寡人.每天楼上楼下没完没了的跑，都没有时间碰计算机，在过几天，连这个都没有了，估计要若干年后才能回来，我的电脑太重，没有办法搬到中国去，几年后也成为垃圾，只好送给邻居，一个鬼子好朋友。家里还有那么多的红木家具，都不知怎么搬下去，这里不同中国，周围没有一个中国人，最后也只好去找鬼子帮忙。还有 7000 多平方英尺的花园要种草，先卖 Top soil。然后铺上，然后平整，然后施肥，然后播 Canada No1.的草种，这可是真正的苦力。这一切都要在交房前完成，只好和计算机拜拜了，如果还能抽点时间，准备写如何打造自己的 `GetModuleHandle` 和 `GetProcAddress`。

回到中国，买了计算机后，就又能泡制垃圾了。

4dd466d0e1e0cc123afbe504da2462ca
