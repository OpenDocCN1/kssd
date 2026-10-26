
# [原创]文件夹加密精灵算法分析

- 【文章标题】: 文件夹加密精灵算法分析
- 【文章作者】: 8713007
- 【软件名称】: 文件夹加密精灵
- 【软件大小】: 448k
- 【下载地址】: 自己搜索下载
- 【保护方式】: 序列号（机器码）+重起
- 【编写语言】: Vc++
- 【使用工具】: OD,W32dsm8.93+
- 【操作平台】: WinXp
- 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 【详细过程】

启动程序，点击注册，输入序列号：12345678901234567890，程序提示重起。复制机器码后搜索注册表发现程序在注册表名为`000`处保存机器码。利用OD载入，下段`bp RegQueryValueExA`，F9

```asm
  77DA2410 >  55              push    ebp/////////////////段在这里，ctrl+F9返回
  77DA2411    8BEC            mov     ebp, esp
  77DA2413    83EC 2C         sub     esp, 2C
  77DA2416    57              push    edi
  77DA2417    33FF            xor     edi, edi
  77DA2419    397D 10         cmp     [ebp+10], edi
  77DA241C    897D F8         mov     [ebp-8], edi
  77DA241F    897D F4         mov     [ebp-C], edi
  77DA2422    0F85 37F60000   jnz     77DB1A5F
  77DA2428    397D 18         cmp     [ebp+18], edi
  77DA242B    0F85 C9000000   jnz     77DA24FA
  77DA2431    53              push    ebx
  77DA2432    8D45 F4         lea     eax, [ebp-C]
  77DA2435    50              push    eax
  77DA2436    FF75 08         push    dword ptr [ebp+8]
  77DA2439    E8 92F2FFFF     call    77DA16D0
  77DA243E    8BD8            mov     ebx, eax
  77DA2440    3BDF            cmp     ebx, edi
  77DA2442    0F84 EDF50000   je      77DB1A35
  77DA2448    56              push    esi
  77DA2449    897D E0         mov     [ebp-20], edi
  77DA244C    64:A1 18000000  mov     eax, fs:[18]
  77DA2452    FF75 0C         push    dword ptr [ebp+C]
  77DA2455    8DB0 F80B0000   lea     esi, [eax+BF8]
  77DA245B    8D45 D4         lea     eax, [ebp-2C]
  77DA245E    50              push    eax
  77DA245F    FF15 7413DA77   call    [<&ntdll.RtlInitAnsiString>]     ; ntdll.RtlInitAnsiString
  77DA2465    57              push    edi

  /////////////////////////////////////////////////

  0040518F  |.  8945 E4       mov     [ebp-1C], eax/////////////////////////返回在这里，F8单步跟踪
  00405192  |.  837D E4 00    cmp     dword ptr [ebp-1C], 0
  00405196  |.  74 04         je      short 0040519C
  00405198  |.  33C0          xor     eax, eax
  0040519A  |.  EB 53         jmp     short 004051EF
  0040519C  |>  C745 E8 00000>mov     dword ptr [ebp-18], 0
  004051A3  |.  EB 09         jmp     short 004051AE
  004051A5  |>  8B55 E8       /mov     edx, [ebp-18]///////ebp-18入edx
  004051A8  |.  83C2 01       |add     edx, 1/////////////////edx+1
  004051AB  |.  8955 E8       |mov     [ebp-18], edx
  004051AE  |>  837D E8 14     cmp     dword ptr [ebp-18], 14/////比较是否大于20
  004051B2  |.  7D 17         |jge     short 004051CB
  004051B4  |.  8B45 E0       |mov     eax, [ebp-20]
  004051B7  |.  0345 E8       |add     eax, [ebp-18]
  004051BA  |.  8B4D E8       |mov     ecx, [ebp-18]
  004051BD  |.  8A90 C4000000 |mov     dl, [eax+C4]
  004051C3  |.  8891 54E84500 |mov     [ecx+45E854], dl/////循环取注册码前20位入ecx+45E854
  004051C9  |.^ EB DA         \jmp     short 004051A5
  004051CB  |>  C605 68E84500>mov     byte ptr [45E868], 0
  004051D2  |.  68 54E84500   push    0045E854                         ; /Arg1 = 0045E854
  004051D7  |.  8B4D E0       mov     ecx, [ebp-20]                    ; |
  004051DA  |.  E8 D1080000   call    00405AB0                         ; \FolderPr.00405AB0//////// 关键call,F7跟进
  004051DF  |.  8945 F0       mov     [ebp-10], eax
  004051E2  |.  8B45 EC       mov     eax, [ebp-14]
  004051E5  |.  50            push    eax                              ; /hKey
  004051E6  |.  FF15 00F04400 call    [<&ADVAPI32.RegCloseKey>]        ; \RegCloseKey
  004051EC  |.  8B45 F0       mov     eax, [ebp-10]
  004051EF  |>  8BE5          mov     esp, ebp
  004051F1  |.  5D            pop     ebp
  004051F2  \.  C3            retn
```

```asm
  /////////////////////////////////F7跟进后来到这里

  00405AB0  /$  55            push    ebp
  00405AB1  |.  8BEC          mov     ebp, esp
  00405AB3  |.  83EC 24       sub     esp, 24
  00405AB6  |.  894D DC       mov     [ebp-24], ecx
  00405AB9  |.  C745 FC 01000>mov     dword ptr [ebp-4], 1
  00405AC0  |.  C745 E0 00000>mov     dword ptr [ebp-20], 0
  00405AC7  |.  EB 09         jmp     short 00405AD2
  00405AC9  |>  8B45 E0       /mov     eax, [ebp-20]
  00405ACC  |.  83C0 01       |add     eax, 1
  00405ACF  |.  8945 E0       |mov     [ebp-20], eax
  00405AD2  |>  837D E0 14     cmp     dword ptr [ebp-20], 14////////////////比较是否取够20位
  00405AD6  |.  7D 17         |jge     short 00405AEF///取够以后跳向00405AEF
  00405AD8  |.  8B4D E0       |mov     ecx, [ebp-20]
  00405ADB  |.  51            |push    ecx                             ; /Arg1
  00405ADC  |.  B9 6CE84500   |mov     ecx, 0045E86C                   ; |
  00405AE1  |.  E8 3ADEFFFF   |call    00403920 /////////////取得机器码
  00405AE6  |.  8B55 E0       |mov     edx, [ebp-20]
  00405AE9  |.  884415 E4     |mov     [ebp+edx-1C], al////////机器码入ebp+edx-1c
  00405AED  |.^ EB DA         \jmp     short 00405AC9
  00405AEF  |>  C705 1CE54500>mov     dword ptr [45E51C], 0
  00405AF9  |.  EB 0D         jmp     short 00405B08
  00405AFB  |>  A1 1CE54500   /mov     eax, [45E51C]
  00405B00  |.  83C0 01       |add     eax, 1
  00405B03  |.  A3 1CE54500   |mov     [45E51C], eax
  00405B08  |>  833D 1CE54500> cmp     dword ptr [45E51C], 14
  00405B0F  |.  0F83 86000000 |jnb     00405B9B
  00405B15  |.  8B0D 1CE54500 |mov     ecx, [45E51C]
  00405B1B  |.  8A540D E4     |mov     dl, [ebp+ecx-1C]/////依次取机器码的Ascii每一位入dl,
  00405B1F  |.  8855 F8       |mov     [ebp-8], dl///////////dl入ebp-8
  00405B22  |.  0FBE45 F8     |movsx   eax, byte ptr [ebp-8]/////ebp-8入eax
  00405B26  |.  83F8 30       |cmp     eax, 30///////////////////Ascii小于30跳走
  00405B29  |.  7C 1E         |jl      short 00405B49
  00405B2B  |.  0FBE4D F8     |movsx   ecx, byte ptr [ebp-8]
  00405B2F  |.  83F9 39       |cmp     ecx, 39/////////////////Ascii大于39跳走
  00405B32  |.  7F 15         |jg      short 00405B49
  00405B34  |.  0FBE55 F8     |movsx   edx, byte ptr [ebp-8]//////机器码入edx
  00405B38  |.  B8 69000000   |mov     eax, 69//////////////////69入eax
  00405B3D  |.  2BC2          |sub     eax, edx////////////////////edx-eax结果入eax
  00405B3F  |.  8B0D 1CE54500 |mov     ecx, [45E51C]
  00405B45  |.  88440D E4     |mov     [ebp+ecx-1C], al//////将计算的结果取代原来的机器码
  00405B49  |>  0FBE55 F8     |movsx   edx, byte ptr [ebp-8]////////机器码的Ascii大于39来到这里
  00405B4D  |.  83FA 41       |cmp     edx, 41////////////比较机器码的Ascii大于41跳走
  00405B50  |.  7C 1D         |jl      short 00405B6F
  00405B52  |.  0FBE45 F8     |movsx   eax, byte ptr [ebp-8]///机器码入eax
  00405B56  |.  83F8 5A       |cmp     eax, 5A/////////////大于5A跳走
  00405B59  |.  7F 14         |jg      short 00405B6F
  00405B5B  |.  0FBE4D F8     |movsx   ecx, byte ptr [ebp-8]//////机器码入ecx
  00405B5F  |.  BA 9B000000   |mov     edx, 9B////9B入edx
  00405B64  |.  2BD1          |sub     edx, ecx///////edx-ecx结果入edx
  00405B66  |.  A1 1CE54500   |mov     eax, [45E51C]
  00405B6B  |.  885405 E4     |mov     [ebp+eax-1C], dl//////将计算的结果取代原来的机器码
  00405B6F  |>  0FBE4D F8     |movsx   ecx, byte ptr [ebp-8]//////////机器码的Ascii<30 或Ascii>39来到这里
  00405B73  |.  83F9 61       |cmp     ecx, 61////////////////小于61则重新取值
  00405B76  |.  7C 1E         |jl      short 00405B96
  00405B78  |.  0FBE55 F8     |movsx   edx, byte ptr [ebp-8]
  00405B7C  |.  83FA 7A       |cmp     edx, 7A/////////////////大于7A则重新取值
  00405B7F  |.  7F 15         |jg      short 00405B96
  00405B81  |.  0FBE45 F8     |movsx   eax, byte ptr [ebp-8]
  00405B85  |.  B9 DB000000   |mov     ecx, 0DB //////DB入ecx
  00405B8A  |.  2BC8          |sub     ecx, eax//////
  00405B8C  |.  8B15 1CE54500 |mov     edx, [45E51C]
  00405B92  |.  884C15 E4     |mov     [ebp+edx-1C], cl
  00405B96  |>^ E9 60FFFFFF   \jmp     00405AFB///////////////////////循环
```

计算完成后的数组我们设为`X`，`X`共21位

/////////////////////////////////计算完成后来到这里
