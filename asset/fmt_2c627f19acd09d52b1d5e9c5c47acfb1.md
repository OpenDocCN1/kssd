
# Exescope v6.50 注册算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-109.htm`](https://www.pediy.com/kssd/pediy07/pediy7-109.htm)

**【目标】**: `Exescope` `v6.50`  
**【工具】**: `Peid`, `DeDe`, `Ollydbg` `v1.10`  
**【保护类型】**: 无壳，未使用密码学算法  
**【难度】**: 易  
**【作者】**: 隐者无疆`[BCG]`  
**【目的】**: 兴趣  

**【详细过程】**:

`Exescope` 这款软件相信大家一定都不陌生吧，这一次研究的是它的 `6.50` 版。

1. 用 `Peid` 侦测，显示“Borland Delphi 6.0 - 7.0”。未加壳，难度降了一级。
2. 既然是 `Delphi` 编写的程序，反汇编工具自然应该优先选择 `DeDe`。

`RegBtnClick` 事件对应的部分汇编代码如下

```asm
004C2A5B   64FF30           push    dword ptr fs:[eax]
004C2A5E   648920           mov     fs:[eax], esp
004C2A61   A15CFC4C00       mov     eax, dword ptr [$004CFC5C]
004C2A66   803800           cmp     byte ptr [eax], $00
004C2A69   740F            jz      004C2A7A
004C2A6B   C7834C02000002000000 mov dword ptr [ebx+$024C], $00000002
004C2A75   E9FF000000       jmp     004C2B79
004C2A7A   8D55FC          lea     edx, [ebp-$04]
* Reference to control NameEdit : TEdit
|
004C2A7D   8B83F8020000    mov     eax, [ebx+$02F8]
* Reference to: controls.TControl.GetText(TControl):System.String;
|
004C2A83   E8F4F8FAFF      call    0047237C                       ;获取注册名
004C2A88   8B55FC          mov     edx, [ebp-$04]
004C2A8B   A1E8FE4C00      mov     eax, dword ptr [$004CFEE8]
|
004C2A90   E84F1FF4FF      call    004049E4
004C2A95   8D55F8          lea     edx, [ebp-$08]
* Reference to control IDEdit : TEdit
|
004C2A98   8B83FC020000    mov     eax, [ebx+$02FC]
* Reference to: controls.TControl.GetText(TControl):System.String;
|
004C2A9E   E8D9F8FAFF      call    0047237C                       ;  获取注册码
004C2AA3   8B55F8          mov     edx, [ebp-$08]
004C2AA6   A14CFE4C00      mov     eax, dword ptr [$004CFE4C]
|
004C2AAB   E8341FF4FF      call    004049E4
004C2AB0   8B154CFE4C00    mov     edx, [$004CFE4C]
004C2AB6   8B12          mov     edx, [edx]
004C2AB8   A154FC4C00      mov     eax, dword ptr [$004CFC54]
004C2ABD   8B00          mov     eax, [eax]
* Reference to : TFMain.CheckCode()
|
004C2ABF   E8B8940000      call    004CBF7C                        ;  关键 Call,跟进
004C2AC4   84C0            test    al, al                          
004C2AC6   0F848D000000    jz      004C2B59                        ;  关键跳转,不能跳
    .........           .................
004C2B59   6A00            push    $00
004C2B5B   8D55EC          lea     edx, [ebp-$14]
* Possible String Reference to: 'Invalid ID or Name'
|                         |
004C2B5E   B8F02B4C00      mov     eax, $004C2BF0
* Reference to : TFReg._PROC_004C38D0()
|
004C2B63   E8680D0000      call    004C38D0
```

3. `Ollydbg` 跟进关键 `Call`.

   试验码: `A191067890`  
   注册名: `warshon`

   `call 004CBF7C` 对应的反汇编代码如下：

```asm
004CBF7C  /$  >push ebp
004CBF7D  |.  >mov ebp,esp
004CBF7F  |.  >push ecx
004CBF80  |.  >push ebx
004CBF81  |.  >mov dword ptr ss:[ebp-4],edx
004CBF84  |.  >mov eax,dword ptr ss:[ebp-4]
004CBF87  |.  >call eXeScope.00404E40
004CBF8C  |.  >xor eax,eax
004CBF8E  |.  >push ebp
004CBF8F  |.  >push eXeScope.004CC01B
004CBF94  |.  >push dword ptr fs:[eax]
004CBF97  |.  >mov dword ptr fs:[eax],esp
004CBF9A  |.  >xor ebx,ebx
004CBF9C  |.  >mov eax,dword ptr ss:[ebp-4]
004CBF9F  |.  >call eXeScope.00404C50                   ;  LStrLen
004CBFA4  |.  >cmp eax,0A                               ;  注册码必须为十位
004CBFA7  |.  >jnz short <eXeScope.Wrong!!!>
004CBFA9  |.  >mov edx,dword ptr ss:[ebp-4]
004CBFAC  |.  >mov eax,eXeScope.004CC030                 ;  ASCII "A1910"
004CBFB1  |.  >call eXeScope.00404F94                   ;  跟进
004CBFB6  |.  >dec eax
004CBFB7  |.  >je short eXeScope.004CBFC9
004CBFB9  |.  >mov edx,dword ptr ss:[ebp-4]
004CBFBC  |.  >mov eax,eXeScope.004CC040                 ;  ASCII "A1423"
004CBFC1  |.  >call eXeScope.00404F94                   ;  同上
004CBFC6  |.  >dec eax
004CBFC7  |.  >jnz short <eXeScope.Wrong!!!>
004CBFC9  |>  >mov eax,2
004CBFCE  |>  >/mov edx,dword ptr ss:[ebp-4]
004CBFD1  |.  >|mov dl,byte ptr ds:[edx+eax-1]
004CBFD5  |.  >|cmp dl,30                                \\
004CBFD8  |.  >|jb short <eXeScope.Wrong!!!>              |
004CBFDA  |.  >|cmp dl,39                                 |
004CBFDD  |.  >|ja short <eXeScope.Wrong!!!>              /   注册码第二位以后各位必须为数字
004CBFDF  |.  >|inc eax
004CBFE0  |.  >|cmp eax,0B
004CBFE3  |.^>\\jnz short eXeScope.004CBFCE
004CBFE5  |.  >mov eax,dword ptr ss:[ebp-4]               ;  eax=reg_code
004CBFE8  |.  >movzx eax,byte ptr ds:[eax+8]              ;  eax=注册码第九位的 Ascii 码
004CBFEC  |.  >mov edx,dword ptr ss:[ebp-4]               ;  edx=reg_code
004CBFEF  |.  >movzx edx,byte ptr ds:[edx+9]              ;  edx=注册码第十位的 Ascii 码
004CBFF3  |.  >add eax,edx
004CBFF5  |.  >mov ecx,0A
004CBFFA  |.  >xor edx,edx
004CBFFC  |.  >div ecx                                     ;  ecx=0xA
004CBFFE  |.  >cmp edx,4                                    ;  余数必须为 4
004CC001  |.  >jnz short <eXeScope.Wrong!!!>
004CC003  |.  >mov bl,1
004CC005  |>  >xor eax,eax
004CC007  |.  >pop edx
004CC008  |.  >pop ecx
004CC009  |.  >pop ecx
004CC00A  |.  >mov dword ptr fs:[eax],edx
004CC00D  |.  >push eXeScope.004CC022
004CC012  |>  >lea eax,dword ptr ss:[ebp-4]
004CC015  |.  >call eXeScope.00404990
004CC01A  \.  >retn
004CC01B   .^>jmp eXeScope.00404374
004CC020   .^>jmp short eXeScope.004CC012
004CC022   . >mov eax,ebx
004CC024   . >pop ebx
004CC025   . >pop ecx
004CC026   . >pop ebp
004CC027   . >retn
```

`call eXeScope.00404F94` 对应的反汇编代码如下:

```asm
00404F94  /$  >test eax,eax                                ;  "A1910" 或者 "A1423"
00404F96  |.  >je short eXeScope.00404FD8
00404F98  |.  >test edx,edx                                ;  reg_code
00404F9A  |.  >je short eXeScope.00404FCD
00404F9C  |.  >push ebx
00404F9D  |.  >push esi
00404F9E  |.  >push edi
00404F9F  |.  >mov esi,eax                                  ;  "A1910" 或者 "A1423"
00404FA1  |.  >mov edi,edx                                   ;  reg_code
00404FA3  |.  >mov ecx,dword ptr ds:[edi-4]                 ;  0xA
00404FA6  |.  >push edi
00404FA7  |.  >mov edx,dword ptr ds:[esi-4]                 ;  0x5
00404FAA  |.  >dec edx
00404FAB  |.  >js short <eXeScope.Wrong!!!>
00404FAD  |.  >mov al,byte ptr ds:[esi]
00404FAF  |.  >inc esi
00404FB0  |.  >sub ecx,edx                                   ;  ecx=6
00404FB2  |.  >jle short <eXeScope.Wrong!!!>
00404FB4  |>  >/repne scas byte ptr es:[edi]
00404FB6  |.  >|jnz short <eXeScope.Wrong!!!>
00404FB8  |.  >|mov ebx,ecx
00404FBA  |.  >|push esi
00404FBB  |.  >|push edi
00404FBC  |.  >|mov ecx,edx
00404FBE  |.  >|repe cmps byte ptr es:[edi],byte ptr ds:[esi]
00404FC0  |.  >|pop edi
00404FC1  |.  >|pop esi
00404FC2  |.  >|je short eXeScope.00404FD0
00404FC4  |.  >|mov ecx,ebx
00404FC6  |.^ >\\jmp short eXeScope.00404FB4                 ;  注册码的首五位必须为 "A1910" 或者 "A1423"
00404FC8  |>  >pop edx
00404FC9  |.  >xor eax,eax
00404FCB  |.  >jmp short eXeScope.00404FD5
00404FCD  |>  >xor eax,eax
00404FCF  |.  >retn
00404FD0  |>  >pop edx
00404FD1  |.  >mov eax,edi
00404FD3  |.  >sub eax,edx
00404FD5  |>  >pop edi
00404FD6  |.  >pop esi
00404FD7  |.  >pop ebx
00404FD8  \\>  >retn
```

4. 注册算法分析如上述注释所示。
