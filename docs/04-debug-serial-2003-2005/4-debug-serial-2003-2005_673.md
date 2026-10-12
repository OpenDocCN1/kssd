
# 卸载精灵 V4.1

> 原文：[`www.pediy.com/kssd/pediy06/pediy6731.htm`](https://www.pediy.com/kssd/pediy06/pediy6731.htm)

**【破解作者】** `kyc[dfcg][czg]`

**【作者邮箱】** `muyang008@163.com`

**【使用工具】** `old1.10c`

**【破解平台】** `win2003`

**【软件名称】** `卸载精灵 V4.1`

**【下载地址】** `http://www.skycn.com/soft/4502.html`

**【软件简介】** 您的电脑里面有很多软件、游戏无法卸载？已经厌倦了 `WINDOWS` 控制面板那个功能很弱的“添加/删除程序”了吗？

卸载精灵的智能卸载功能可以干净彻底地卸载软件，包括那些无法正常卸载的程序它都可以帮您卸载。`2.0` 版新增加了清除垃圾文件的功能。

`2.3` 版的智能卸载更强大，可以卸载桌面快捷方式及菜单。`3.0` 以上版本增加了 `IE` 浏览器修复功能。

这个软件特别适合网吧做系统维护，清除系统中的垃圾软件，优化系统的运行速度。它操作方便，界面友好，安全可靠。

**【软件大小】** `3210 KB`

**【加壳方式】** 无壳

**【破解声明】** 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

## 【破解内容】

这两天破了几个小软件下载了不少共享软件，我的电脑几乎装满了于是下载了 `卸载精灵 V4.1`。

觉得好用就开始破解了。

### 根据序列号错误

```
00405E93    /.  55            push ebp
00405E94    |.  8BEC          mov ebp,esp
00405E96    |.  83EC 14       sub esp,14
00405E99    |.  894D F4       mov dword ptr ss:[ebp-C],ecx
00405E9C    |.  68 4C5A4100   push Cleaner.00415A4C
00405EA1    |.  8B4D F4       mov ecx,dword ptr ss:[ebp-C]
00405EA4    |.  81C1 A0000000 add ecx,0A0
00405EAA    |.  E8 79570000   call <jmp.&MFC42.#3874_CWnd::GetWindowTextA>         ;  取用户名
00405EAF    |.  68 485A4100   push Cleaner.00415A48
00405EB4    |.  8B4D F4       mov ecx,dword ptr ss:[ebp-C]
00405EB7    |.  83C1 60       add ecx,60
00405EBA    |.  E8 69570000   call <jmp.&MFC42.#3874_CWnd::GetWindowTextA>         ;  取注册码
00405EBF    |.  51            push ecx
00405EC0    |.  8BCC          mov ecx,esp
00405EC2    |.  8965 F8       mov dword ptr ss:[ebp-8],esp
00405EC5    |.  68 485A4100   push Cleaner.00415A48
00405ECA    |.  E8 ED530000   call <jmp.&MFC42.#535_CString::CString>
00405ECF    |.  8945 F0       mov dword ptr ss:[ebp-10],eax                       ; |
00405ED2    |.  E8 66BBFFFF   call Cleaner.00401A3D                               ; \f7  关键算法
00405ED7    |.  83C4 04       add esp,4
00405EDA    |.  8845 EC       mov byte ptr ss:[ebp-14],al
00405EDD    |.  8B45 EC       mov eax,dword ptr ss:[ebp-14]
00405EE0    |.  25 FF000000   and eax,0FF
00405EE5    |.  85C0          test eax,eax
00405EE7    |.  0F84 DD000000 je Cleaner.00405FCA                                 ;  跳就失败
================================================================================call Cleaner.00401A3D
00401A3D    /$  55            push ebp
00401A3E    |.  8BEC          mov ebp,esp
00401A40    |.  6A FF         push -1
00401A42    |.  68 27C04000   push Cleaner.0040C027                                ;  SE handler installation
00401A47    |.  64:A1 0000000>mov eax,dword ptr fs:[0]
00401A4D    |.  50            push eax
00401A4E    |.  64:8925 00000>mov dword ptr fs:[0],esp
00401A55    |.  83EC 1C       sub esp,1C
00401A58    |.  56            push esi
00401A59    |.  C745 FC 00000>mov dword ptr ss:[ebp-4],0
00401A60    |.  8D4D 08       lea ecx,dword ptr ss:[ebp+8]
00401A63    |.  E8 48030000   call Cleaner.00401DB0
00401A68    |.  83F8 08       cmp eax,8                                             ;  注册码必须是 8 位
00401A6B    |.  74 1B         je short Cleaner.00401A88
00401A6D    |.  C645 F0 00    mov byte ptr ss:[ebp-10],0
00401A71    |.  C745 FC FFFFF>mov dword ptr ss:[ebp-4],-1
00401A78    |.  8D4D 08       lea ecx,dword ptr ss:[ebp+8]
00401A7B    |.  E8 10970000   call <jmp.&MFC42.#800_CString::~CString>
00401A80    |.  8A45 F0       mov al,byte ptr ss:[ebp-10]
00401A83    |.  E9 3C010000   jmp Cleaner.00401BC4
00401A88    |>  6A 00         push 0                                                ; /Arg1 = 00000000
00401A8A    |.  8D4D 08       lea ecx,dword ptr ss:[ebp+8]                          ; |
00401A8D    |.  E8 6E030000   call Cleaner.00401E00                                 ; \Cleaner.00401E00
00401A92    |.  0FBEF0        movsx esi,al                                          ;  ESI=取注册码的第 1 位
00401A95    |.  6A 02         push 2                                                ; /Arg1 = 00000002
00401A97    |.  8D4D 08       lea ecx,dword ptr ss:[ebp+8]                          ; |
00401A9A    |.  E8 61030000   call Cleaner.00401E00                                 ; \Cleaner.00401E00
00401A9F    |.  0FBEC0        movsx eax,al                                          ;  EAX=取注册码的第 3 位
00401AA2    |.  03F0          add esi,eax                                           ;  ESI=EAX+ESI
00401AA4    |.  83FE 6D       cmp esi,6D                                            ;  ESI==6D 吗 不等就失败
00401AA7    |.  74 1B         je short Cleaner.00401AC4
00401AA9    |.  C645 EC 00    mov byte ptr ss:[ebp-14],0
00401AAD    |.  C745 FC FFFFF>mov dword ptr ss:[ebp-4],-1
00401AB4    |.  8D4D 08       lea ecx,dword ptr ss:[ebp+8]
00401AB7    |.  E8 D4960000   call <jmp.&MFC42.#800_CString::~CString>
00401ABC    |.  8A45 EC       mov al,byte ptr ss:[ebp-14]
00401ABF    |.  E9 00010000   jmp Cleaner.00401BC4
00401AC4    |>  6A 01         push 1                                                ; /Arg1 = 00000001
00401AC6    |.  8D4D 08       lea ecx,dword ptr ss:[ebp+8]                          ; |
00401AC9    |.  E8 32030000   call Cleaner.00401E00                                 ; \Cleaner.00401E00
00401ACE    |.  0FBEF0        movsx esi,al                                          ;  ESI=取注册码的第 2 位
00401AD1    |.  6A 03         push 3                                                ; /Arg1 = 00000003
00401AD3    |.  8D4D 08       lea ecx,dword ptr ss:[ebp+8]                          ; |
00401AD6    |.  E8 25030000   call Cleaner.00401E00                                 ; \Cleaner.00401E00
00401ADB    |.  0FBEC8        movsx ecx,al                                          ;  ECX=取注册码的第 4 位
00401ADE    |.  03F1          add esi,ecx                                           ;  ESI=ECX+ESI
00401AE0    |.  83FE 6A       cmp esi,6A                                            ;  ESI==6A 吗 不等就失败
00401AE3    |.  74 1B         je short Cleaner.00401B00
00401AE5    |.  C645 E8 00    mov byte ptr ss:[ebp-18],0
00401AE9    |.  C745 FC FFFFF>mov dword ptr ss:[ebp-4],-1
00401AF0    |.  8D4D 08       lea ecx,dword ptr ss:[ebp+8]
00401AF3    |.  E8 98960000   call <jmp.&MFC42.#800_CString::~CString>
00401AF8    |.  8A45 E8       mov al,byte ptr ss:[ebp-18]
00401AFB    |.  E9 C4000000   jmp Cleaner.00401BC4
00401B00    |>  6A 04         push 4                                                ; /Arg1 = 00000004
00401B02    |.  8D4D 08       lea ecx,dword ptr ss:[ebp+8]                          ; |
00401B05    |.  E8 F6020000   call Cleaner.00401E00                                 ; \Cleaner.00401E00
00401B0A    |.  0FBEF0        movsx esi,al                                          ;  ESI=取注册码的第 5 位
00401B0D    |.  6A 05         push 5                                                ; /Arg1 = 00000005
00401B0F    |.  8D4D 08       lea ecx,dword ptr ss:[ebp+8]                          ; |
00401B12    |.  E8 E9020000   call Cleaner.00401E00                                 ; \Cleaner.00401E00
00401B17    |.  0FBED0        movsx edx,al                                          ;  EDX=取注册码的第 6 位
00401B1A    |.  03F2          add esi,edx                                           ;  ESI=EDX+ESI
00401B1C    |.  83FE 6B       cmp esi,6B                                            ;  ESI==6B 吗 不等就失败
00401B1F    |.  74 1B         je short Cleaner.00401B3C
00401B21    |.  C645 E4 00    mov byte ptr ss:[ebp-1C],0
00401B25    |.  C745 FC FFFFF>mov dword ptr ss:[ebp-4],-1
00401B2C    |.  8D4D 08       lea ecx,dword ptr ss:[ebp+8]
00401B2F    |.  E8 5C960000   call <jmp.&MFC42.#800_CString::~CString>
```



## 汇编代码段

```assembly
00401B34    |.   8A45 E4         mov al,byte ptr ss:[ebp-1C]
00401B37    |.   E9 88000000    jmp Cleaner.00401BC4
00401B3C    |>   6A 06          push 6                                ; /Arg1 = 00000006
00401B3E    |.   8D4D 08        lea ecx,dword ptr ss:[ebp+8]          ; |
00401B41    |.   E8 BA020000    call Cleaner.00401E00                 ; \Cleaner.00401E00
00401B46    |.   0FBEF0         movsx esi,al                          ;  ESI=取注册码的第 7 位
00401B49    |.   6A 07          push 7                                ; /Arg1 = 00000007
00401B4B    |.   8D4D 08        lea ecx,dword ptr ss:[ebp+8]          ; |ESI=取注册码的第 7 位
00401B4E    |.   E8 AD020000    call Cleaner.00401E00                 ; \Cleaner.00401E00
00401B53    |.   0FBEC0         movsx eax,al                          ;  EAX=取注册码的第 8 位
00401B56    |.   03F0           add esi,eax                           ;  ESI=EAX+ESI
00401B58    |.   83FE 6C        cmp esi,6C                            ;  ESI==6C 吗 不等就失败
00401B5B    |.   74 18          je short Cleaner.00401B75
00401B5D    |.   C645 E0 00     mov byte ptr ss:[ebp-20],0
00401B61    |.   C745 FC FFFF>  mov dword ptr ss:[ebp-4],-1
00401B68    |.   8D4D 08        lea ecx,dword ptr ss:[ebp+8]
00401B6B    |.   E8 20960000    call <jmp.&MFC42.#800_CString::~CString>
00401B70    |.   8A45 E0        mov al,byte ptr ss:[ebp-20]
00401B73    |.   EB 4F          jmp short Cleaner.00401BC4
00401B75    |>   6A 00          push 0                                ; /Arg1 = 00000000
00401B77    |.   8D4D 08        lea ecx,dword ptr ss:[ebp+8]          ; |
00401B7A    |.   E8 81020000    call Cleaner.00401E00                 ; \Cleaner.00401E00
00401B7F    |.   0FBEF0         movsx esi,al                          ;  ESI=取注册码的第 1 位
00401B82    |.   6A 07          push 7                                ; /Arg1 = 00000007
00401B84    |.   8D4D 08        lea ecx,dword ptr ss:[ebp+8]          ; |
00401B87    |.   E8 74020000    call Cleaner.00401E00                 ; \Cleaner.00401E00
00401B8C    |.   0FBEC8         movsx ecx,al                          ;  ECX=取注册码的第 8 位
00401B8F    |.   03F1           add esi,ecx                           ;  ESI=ECX+ESI
00401B91    |.   83FE 69        cmp esi,69                            ;  ESI==69 吗 不等就失败
00401B94    |.   74 18          je short Cleaner.00401BAE
00401B96    |.   C645 DC 00     mov byte ptr ss:[ebp-24],0
00401B9A    |.   C745 FC FFFF>  mov dword ptr ss:[ebp-4],-1
00401BA1    |.   8D4D 08        lea ecx,dword ptr ss:[ebp+8]
00401BA4    |.   E8 E7950000    call <jmp.&MFC42.#800_CString::~CString>
00401BA9    |.   8A45 DC        mov al,byte ptr ss:[ebp-24]
00401BAC    |.   EB 16          jmp short Cleaner.00401BC4
00401BAE    |>   C645 D8 01     mov byte ptr ss:[ebp-28],1
00401BB2    |.   C745 FC FFFF>  mov dword ptr ss:[ebp-4],-1
00401BB9    |.   8D4D 08        lea ecx,dword ptr ss:[ebp+8]
00401BBC    |.   E8 CF950000    call <jmp.&MFC42.#800_CString::~CString>
00401BC1    |.   8A45 D8        mov al,byte ptr ss:[ebp-28]
00401BC4    |>   8B4D F4        mov ecx,dword ptr ss:[ebp-C]
00401BC7    |.   64:890D 00000> mov dword ptr fs:[0],ecx
00401BCE    |.   5E             pop esi
00401BCF    |.   8BE5           mov esp,ebp
00401BD1    |.   5D             pop ebp
00401BD2    \.   C3             retn
```

========================================================================

```assembly
00405EED    |.   C605 505A4100> mov byte ptr ds:[415A50],1
00405EF4    |.   8D4D FC        lea ecx,dword ptr ss:[ebp-4]
00405EF7    |.   51             push ecx                            ; /pHandle
00405EF8    |.   68 3F000F00    push 0F003F                         ; |Access = KEY_ALL_ACCESS
00405EFD    |.   6A 00          push 0                              ; |Reserved = 0
00405EFF    |.   68 4C544100    push Cleaner.0041544C               ; |Subkey = "Software\Legendsoft\Uninstaller"
00405F04    |.   68 02000080    push 80000002                       ; |hKey = HKEY_LOCAL_MACHINE
00405F09    |.   FF15 00D04000 call dword ptr ds:[<&ADVAPI32.RegOpenKeyExA>] ; \RegOpenKeyExA
00405F0F    |.   85C0           test eax,eax
00405F11    |.   75 72          jnz short Cleaner.00405F85
00405F13    |.   B9 4C5A4100    mov ecx,Cleaner.00415A4C
00405F18    |.   E8 93BEFFFF    call Cleaner.00401DB0
00405F1D    |.   50             push eax
00405F1E    |.   B9 4C5A4100    mov ecx,Cleaner.00415A4C
00405F23    |.   E8 88BEFFFF    call Cleaner.00401DB0
00405F28    |.   50             push eax
00405F29    |.   B9 4C5A4100    mov ecx,Cleaner.00415A4C
00405F2E    |.   E8 EF560000    call <jmp.&MFC42.#2915_CString::GetBuffer> ;  写入注册表
00405F33    |.   50             push eax                            ; |Buffer
00405F34    |.   6A 01          push 1                              ; |ValueType = REG_SZ
00405F36    |.   6A 00          push 0                              ; |Reserved = 0
00405F38    |.   68 6C544100    push Cleaner.0041546C               ; |ValueName = "RegName"
00405F3D    |.   8B55 FC        mov edx,dword ptr ss:[ebp-4]        ; |
00405F40    |.   52             push edx                            ; |hKey
00405F41    |.   FF15 18D04000 call dword ptr ds:[<&ADVAPI32.RegSetValueExA>] ; \RegSetValueExA
00405F47    |.   B9 485A4100    mov ecx,Cleaner.00415A48
00405F4C    |.   E8 5FBEFFFF    call Cleaner.00401DB0
00405F51    |.   50             push eax
00405F52    |.   B9 485A4100    mov ecx,Cleaner.00415A48
00405F57    |.   E8 54BEFFFF    call Cleaner.00401DB0
00405F5C    |.   50             push eax
00405F5D    |.   B9 485A4100    mov ecx,Cleaner.00415A48
00405F62    |.   E8 BB560000    call <jmp.&MFC42.#2915_CString::GetBuffer>
00405F67    |.   50             push eax                            ; |Buffer
00405F68    |.   6A 01          push 1                              ; |ValueType = REG_SZ
00405F6A    |.   6A 00          push 0                              ; |Reserved = 0
00405F6C    |.   68 74544100    push Cleaner.00415474               ; |ValueName = "RegCode"
00405F71    |.   8B45 FC        mov eax,dword ptr ss:[ebp-4]        ; |
00405F74    |.   50             push eax                            ; |hKey
00405F75    |.   FF15 18D04000 call dword ptr ds:[<&ADVAPI32.RegSetValueExA>] ; \RegSetValueExA
00405F7B    |.   8B4D FC        mov ecx,dword ptr ss:[ebp-4]
00405F7E    |.   51             push ecx                            ; /hKey
00405F7F    |.   FF15 1CD04000 call dword ptr ds:[<&ADVAPI32.RegCloseKey>] ; \RegCloseKey
```

===================================================================================================

## 总结

注册码和用户名无关，计算一个可用的注册码

* `35+38=6D`：`5`，`8`。第 1 位 `5`，第 3 位 `5`
* `33+37=6A`：`3`，`7`。第 2 位 `3`，第 4 位 `7`
* `35+36=6B`：`5`，`6`。第 5 位 `5`，第 6 位 `6`
* `38+34=6C`：`8`，`4`。第 7 位 `8`，第 8 位 `4`
* `35+34=69`：`5`，`4`。第 1 位 `5`，第 8 位 `4`

所以得到：

`53875684`

注册后注册码在注册表的 `HKLM\SOFTWARE\Legendsoft\Uninstaller\RegCode`

删除后提示未注册.

`4dd466d0e1e0cc123afbe504da2462ca`
