
```
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
77F8957A       push eax                                    ; FileInformation
77F8957B       lea eax, [ebp+ObjectAttributes]
77F8957E       push eax                                    ; ObjectAttributes
77F8957F       mov [ebp+ObjectAttributes.Attributes], OBJ_CASE_INSENSITIVE
77F89586       mov [ebp+ObjectAttributes.SecurityDescriptor], ebx
77F89589       mov [ebp+ObjectAttributes.SecurityQualityOfService], ebx
77F8958C       call _ZwQueryAttributesFile@8               ; ZwQueryAttributesFile(x,x)
77F89591       mov esi, eax                                ;ZwQueryAttributesFile 返回结果为
... ...
... ...
77F895AA       cmp esi, STATUS_SHARING_VIOLATION
77F895B0       jz short loc_77F895BE
77F895B2       cmp esi, STATUS_ACCESS_DENIED
77F895B8       jz short loc_77F895BE
77F895BA       xor al, al
77F895BC       jmp short loc_77F895C8
... ...
```

`RtlDoesFileExists_UEx` 调用 Ring0 层函数 `ZwQueryAttributesFile`，`ntdll.dll` 包装了这个底层函数，具体如下：

```
77F8880C ; NTSTATUS __stdcall ZwQueryAttributesFile(POBJECT_ATTRIBUTES ObjectAttributes,PFILE_BASIC_INFORMATION FileInformation)
77F8880C           public _ZwQueryAttributesFile@8
77F8880C _ZwQueryAttributesFile@8 proc near
77F8880C ObjectAttributes= dword ptr  4
77F8880C FileInformation= dword ptr  8
77F8880C           mov   eax, 7Ah                          ; NtQueryAttributesFile
77F88811           lea   edx, [esp+ObjectAttributes]
77F88815           int   2Eh                               ; DOS 2+ internal - EXECUTE COMMAND
77F88815                                                   ; DS:SI -> counted CR-terminated command string
77F88817           retn  8
77F88817 _ZwQueryAttributesFile@8 endp
```

当 `ZwQueryAttributesFile` 返回结果为 `STATUS_SHARING_VIOLATION` 或 `STATUS_ACCESS_DENIED` 时 `_gDoDllRedirection = 1 (TRUE)`，其余情况为 `0`。所以用户程序返回结果多数为 `0 (FALSE)`。

老盖为了防止用户直接调 `int 2Eh` (`KiSystemService`)，每次 Updata 功能号都可能发生改变，另外在 `ntoskrnl.exe` 的输出表中，列出了许多函数，供驱动程序使用，但有些 Ring3 使用的函数，例如这个 `ZwQueryAttributesFile` 就没有列出在输出表中，属于 `ntoskrnl.exe` 的内部函数。在 IDA 中可以通过 Names 窗口，很容易的找到这个函数。如果想看 `ZwQueryAttributesFile` 的代码，还需要将前面的 `Zw` 换成 `Nt`，即 `NtQueryAttributesFile`。凡是标有 `Zw` 字头的函数，多数是通过 `int 2Eh` (`KiSystemService`) 而进入 `Nt` 相应名称的函数，当然也有部分的 `Nt` 起头的函数通过 `int 2Eh` (`KiSystemService`) 来转向，大多数都是因为函数本身实现不在 `ntoskrnl.exe` 内部，而在其他模块。功能号为 `Byte`，多数在内部实现，功能号为 `Word` 则多数在外部模块实现，所以跟踪一个函数，可能会跳两三个模块才能找到实现的代码。

## 2.1 `_gDoDllRedirection = TRUE`

```
77E80C58       lea eax, [ebp+ModuleName]
77E80C5B       push eax                            ; returnName
77E80C5C       lea eax, [ebp+UnicodeModuleName]
77E80C5F       push eax                            ; lpModuleName
77E80C60       call _ComputeRedirectedDllName@8    ; 所谓重定向就是到用户程序的目录下加载同名的dll，
77E80C60                                           ; _ComputeRedirectedDllName函数将用户输入的ModuleName
77E80C60                                           ; 前面加上用户的目录(用户程序启动的目录)。
77E80C65       test eax, eax
77E80C67       jge short IfGetRedirDllName
77E80C69       push eax                            ; Status
77E80C6A       call _BaseSetLastNTError@4          ; BaseSetLastNTError(x)
77E80C6F       xor eax, eax
77E80C71       jmp short GetModHandleExit
77E80C73 IfGetRedirDllName:
77E80C73       lea eax, [ebp+ModuleName]
77E80C76       push eax                            ; lpModuleName
77E80C77       call _GetModuleHandleForUnicodeString@4 ; GetModuleHandleForUnicodeString(x)
77E80C7C       mov esi, eax
77E80C7E       mov eax, large fs:18h               ; TEB.NT_TIB.Self
77E80C84       push [ebp+MemoryPointer]            ; MemoryPointer
77E80C87       mov eax, [eax+TEB.Peb]
77E80C8A       push 0                              ; Flags
77E80C8C       push [eax+PEB.ProcessHeap]          ; HeapHandle
77E80C8F       call ds:__imp__RtlFreeHeap@12       ; __declspec(dllimport) RtlFreeHeap(x,x,x)
77E80C95       test esi, esi
77E80C97       jz short DontDllRedir
77E80C99       mov eax, esi
77E80C9B       jmp short GetModHandleExit
```

`_gDoDllRedirection = TRUE` 首先调用 `ComputeRedirectedDllName` 函数，该函数通过调用 `GetModuleFileNameW` 获得用户程序的目录，其后是字符串处理，将用户的 `lpModuleName` 加在用户的目录之后。这里就不列出这段代码了。

然后调用 `GetModuleHandleForUnicodeString`，因为在 `ComputeRedirectedDllName` 函数中分配了堆栈，所以后面调用 `RtlFreeHeap` 释放堆栈。`GetModuleHandleForUnicodeString` 将在下面一起说明。

如果发生错误，将调用 `BaseSetLastNTError`，代码如下：

```
77E68239 _BaseSetLastNTError@4 proc near
77E68239 Status= dword ptr  8
77E68239       push esi
77E6823A       push [esp+Status]                   ; Status
77E6823E       call ds:__imp__RtlNtStatusToDosError@4 ; __declspec(dllimport) RtlNtStatusToDosError(x)
77E68244       mov esi, eax
77E68246       push esi                            ; Status
77E68247       call _SetLastError@4                ; SetLastError(x)
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
```

内部函数 `BaseSetLastNTError` 调用 `RtlNtStatusToDosError` 将 Nt 的错误代码转换成 Dos 错误代码，然后调用 `SetLastError`，上面代码显示，`SetLastError` 就是将错误代码存入 `TEB.LastErrorValue`。

## 2.2 `_gDoDllRedirection = FALSE`

```
         ;_gDoDllRedirection = FALSE
77E80C9D DontDllRedir:
77E80C9D       lea eax, [ebp+UnicodeModuleName]
77E80CA0       push eax                            ; lpModuleName
77E80CA1       call _GetModuleHandleForUnicodeString@4 ; GetModuleHandleForUnicodeString(x)
```

不管走那条路，最后都是调用内部函数 `GetModuleHandleForUnicodeString`，代码如下：
