
# System Mechanic Professiona6

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-460.htm`](https://www.pediy.com/kssd/pediy08/pediy8-460.htm)

## 元数据

- **【文章标题】**: System Mechanic Professiona6 的注册算法分析
- **【文章作者】**: hdhgzf
- **【软件名称】**: System Mechanic Professiona
- **【软件大小】**: 33M
- **【下载地址】**: 自己搜索下载
- **【加壳方式】**: aspack2.12
- **【保护方式】**: 注册码
- **【使用工具】**: od
- **【操作平台】**: windowsxp2
- **【软件介绍】**: 一个很好的系统优化工具，专业版捆绑了卡巴斯基防火墙和杀毒软件。
- **【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

用 `od` 载入主程序，`F9` 运行出现要求输入注册码的对话框，单击，进入输入框填写用户名，格式为一邮箱地址。注册码格式为：

`12345－p6123-1234567890`

返回到 `od`，查找文本字串，`The licensing information you entered is invalid.` 双击

```
008DA840  /$  55            push    ebp
008DA841  |.  8BEC          mov     ebp, esp
008DA843  |.  33C9          xor     ecx, ecx
008DA845  |.  51            push    ecx
008DA846  |.  51            push    ecx
008DA847  |.  51            push    ecx
008DA848  |.  51            push    ecx
008DA849  |.  51            push    ecx
008DA84A  |.  51            push    ecx
008DA84B  |.  51            push    ecx
008DA84C  |.  51            push    ecx
008DA84D  |.  53            push    ebx
008DA84E  |.  56            push    esi
008DA84F  |.  57            push    edi
008DA850  |.  8BF8          mov     edi, eax
008DA852  |.  33C0          xor     eax, eax
008DA854  |.  55            push    ebp
008DA855  |.  68 A5A98D00   push    008DA9A5
008DA85A  |.  64:FF30       push    dword ptr fs:[eax]
008DA85D  |.  64:8920       mov     fs:[eax], esp
008DA860  |.  8D45 F8       lea     eax, [ebp-8]
008DA863  |.  8B15 40A2AC00 mov     edx, [ACA240]                 ;  SysMech6.00ACC730
008DA869  |.  8B12          mov     edx, [edx]
008DA86B  |.  0FB652 04     movzx   edx, byte ptr [edx+4]
008DA86F  |.  8B0D C0A0AC00 mov     ecx, [ACA0C0]                 ;  SysMech6.00AB61A8
008DA875  |.  8B1491        mov     edx, [ecx+edx*4]
008DA878  |.  E8 4FB2B2FF   call    00405ACC
008DA87D  |.  8D45 EC       lea     eax, [ebp-14]
008DA880  |.  50            push    eax
008DA881  |.  8D55 E8       lea     edx, [ebp-18]
008DA884  |.  8B87 2C030000 mov     eax, [edi+32C]
008DA88A  |.  E8 FD74C0FF   call    004E1D8C
008DA88F  |.  8B45 E8       mov     eax, [ebp-18]
008DA892  |.  B9 02000000   mov     ecx, 2
008DA897  |.  BA 01000000   mov     edx, 1
008DA89C  |.  E8 DFB6B2FF   call    00405F80
008DA8A1  |.  8B45 EC       mov     eax, [ebp-14]
008DA8A4  |.  8D55 FC       lea     edx, [ebp-4]
008DA8A7  |.  E8 440BB3FF   call    0040B3F0
008DA8AC  |.  C645 F3 00    mov     byte ptr [ebp-D], 0
008DA8B0  |.  33DB          xor     ebx, ebx
008DA8B2  |.  8B35 0CA2AC00 mov     esi, [ACA20C]                 ;  SysMech6.00AB61D8
008DA8B8  |>  8D55 E4       /lea    edx, [ebp-1C]
008DA8BB  |.  8B06          |mov    eax, [esi]
008DA8BD  |.  E8 2E0BB3FF   |call   0040B3F0
008DA8C2  |.  8B45 E4       |mov    eax, [ebp-1C]
008DA8C5  |.  8B55 FC       |mov    edx, [ebp-4]
008DA8C8  |.  E8 9FB5B2FF   |call   00405E6C
008DA8CD  |.  75 05         |jnz    short 008DA8D4
008DA8CF  |.  885D F3       |mov    [ebp-D], bl
008DA8D2  |.  EB 09         |jmp    short 008DA8DD
008DA8D4  |>  43            |inc    ebx
008DA8D5  |.  83C6 04       |add    esi, 4
008DA8D8  |.  80FB 0C       |cmp    bl, 0C
008DA8DB  |.^ 75 DB         \jnz    short 008DA8B8
008DA8DD  |>  A1 40A2AC00   mov     eax, [ACA240]
008DA8E2  |.  8B00          mov     eax, [eax]
008DA8E4  |.  8A40 04       mov     al, [eax+4]
008DA8E7  |.  3A45 F3       cmp     al, [ebp-D]
008DA8EA  |.  74 06         je      short 008DA8F2
008DA8EC  |.  807D F3 00    cmp     byte ptr [ebp-D], 0
008DA8F0  |.  75 17         jnz     short 008DA909
008DA8F2  |>  57            push    edi
008DA8F3  |.  68 1CBB8D00   push    008DBB1C
008DA8F8  |.  BA BCA98D00   mov     edx, 008DA9BC                 ;  ASCII "The licensing information you entered is invalid.<BR><BR>In most cases this indicates that you have simply typed the information incorrectly. Please verify that you are entering this information exactly as it appears on your order receip"...
008DA8FD  |.  B8 1CAB8D00   mov     eax, 008DAB1C                 ;  ASCII "Invalid License"
008DA902  |.  E8 71F7FFFF   call    008DA078
008DA907  |.  EB 67         jmp     short 008DA970
```

一开始我在这里设断，可以拦下，但找不到所谓的关键 `call`, 于是就思考是不是这里就是一个产生错误的 `call` 呢？说干就干

查找对 `008DA840` 处调用，果然找到。在 `008DB0FB` 为了验证我在 `008D00C` 处设断。`F8` 走一遍，在 `008DB125` 发现关键 `call`

```
008DB00C  .  55            push    ebp
008DB00D  .  8BEC          mov     ebp, esp
008DB00F  .  33C9          xor     ecx, ecx
008DB011  .  51            push    ecx
008DB012  .  51            push    ecx
008DB013  .  51            push    ecx
008DB014  .  51            push    ecx
008DB015  .  51            push    ecx
008DB016  .  51            push    ecx
008DB017  .  51            push    ecx
008DB018  .  53            push    ebx
008DB019  .  8BD8          mov     ebx, eax
008DB01B  .  33C0          xor     eax, eax
008DB01D  .  55            push    ebp
008DB01E  .  68 DAB18D00   push    008DB1DA
008DB023  .  64:FF30       push    dword ptr fs:[eax]
008DB026  .  64:8920       mov     fs:[eax], esp
008DB029  .  33D2          xor     edx, edx
008DB02B  .  55            push    ebp
008DB02C  .  68 A3B18D00   push    008DB1A3
008DB031  .  64:FF32       push    dword ptr fs:[edx]
008DB034  .  64:8922       mov     fs:[edx], esp
008DB037  .  8D55 F4       lea     edx, [ebp-C]
008DB03A  .  8B83 28030000 mov     eax, [ebx+328]
008DB040  .  E8 476DC0FF   call    004E1D8C
008DB045  .  FF75 F4       push    dword ptr [ebp-C]
008DB048  .  68 F0B18D00   push    008DB1F0
008DB04D  .  8D55 F0       lea     edx, [ebp-10]
008DB050  .  8B83 2C030000 mov     eax, [ebx+32C]
008DB056  .  E8 316DC0FF   call    004E1D8C
008DB05B  .  FF75 F0       push    dword ptr [ebp-10]
008DB05E  .  68 F0B18D00   push    008DB1F0
008DB063  .  8D55 EC       lea     edx, [ebp-14]
008DB066  .  8B83 30030000 mov     eax, [ebx+330]
008DB06C  .  E8 1B6DC0FF   call    004E1D8C
008DB071  .  FF75 EC       push    dword ptr [ebp-14]
008DB074  .  8D45 F8       lea     eax, [ebp-8]
008DB077  .  BA 05000000   mov     edx, 5
008DB07C  .  E8 5FADB2FF   call    00405DE0
008DB081  .  8B45 F8       mov     eax, [ebp-8]
008DB084  .  8D55 FC       lea     edx, [ebp-4]
008DB087  .  E8 6403B3FF   call    0040B3F0
008DB08C  .  8D55 E8       lea     edx, [ebp-18]
008DB08F  .  8B83 24030000 mov     eax, [ebx+324]
008DB095  .  E8 5A09BCFF   call    0049B9F4
008DB09A  .  8B45 E8       mov     eax, [ebp-18]
008DB09D  .  8B55 FC       mov     edx, [ebp-4]
008DB0A0  .  E8 B71B0000   call    008DCC5C
008DB0A5  .  84C0          test    al, al
008DB0A7  .  74 16         je      short 008DB0BF
008DB0A9  .  E8 62260000   call    008DD710
008DB0AE  .  B0 01         mov     al, 1
008DB0B0  .  E8 071DD2FF   call    005FCDBC
008DB0B5  .  E8 7AA3B2FF   call    00405434
008DB0BA  .  E9 EB000000   jmp     008DB1AA
008DB0BF  >  E8 442B0000   call    008DDC08
008DB0C4  .  84C0          test    al, al
008DB0C6  .  74 11         je      short 008DB0D9
008DB0C8  .  B0 01         mov     al, 1
008DB0CA  .  E8 ED1CD2FF   call    005FCDBC
```



008DB0CF   .   E8 60A3B2FF   call    00405434  

008DB0D4   .   E9 D1000000   jmp     008DB1AA  

008DB0D9   >   8BC3          mov     eax, ebx  

008DB0DB   .   E8 9CF6FFFF   call    008DA77C  

008DB0E0   .   84C0          test    al, al  

008DB0E2   .   75 0A         jnz     short 008DB0EE  

008DB0E4   .   E8 4BA3B2FF   call    00405434  

008DB0E9   .   E9 BC000000   jmp     008DB1AA  

008DB0EE   >   E8 E12C0000   call    008DDDD4  

008DB0F3   .   84C0          test    al, al  

008DB0F5   .   74 13         je      short 008DB10A  

008DB0F7   .   B2 01         mov     dl, 1  

008DB0F9   .   8BC3          mov     eax, ebx  

008DB0FB   .   E8 40F7FFFF   call    008DA840  

008DB100   .   E8 2FA3B2FF   call    00405434  

008DB105   .   E9 A0000000   jmp     008DB1AA  

008DB10A   >   8D55 E4       lea     edx, [ebp-1C]  

008DB10D   .   8B83 24030000 mov     eax, [ebx+324]  

008DB113   .   E8 DC08BCFF   call    0049B9F4  

008DB118   .   8B55 E4       mov     edx, [ebp-1C]  

008DB11B   .   A1 40A2AC00   mov     eax, [ACA240]  

008DB120   .   8B00          mov     eax, [eax]  

008DB122   .   8B4D FC       mov     ecx, [ebp-4]  

008DB125   .   E8 D647D3FF   call    0060F900                           F7 进入  

008DB12A   .   84C0          test    al, al  

008DB12C   .   74 3A         je      short 008DB168  

008DB12E   .   8B55 FC       mov     edx, [ebp-4]  

008DB131   .   8BC3          mov     eax, ebx  

008DB133   .   E8 5CFCFFFF   call    008DAD94  

008DB138   .   68 F4B18D00   push    008DB1F4                         ; /Title = "System Mechanic Tray Notifyer"  

008DB13D   .   68 14B28D00   push    008DB214                         ; |Class = "TfrmSMTrayNotifyMain"  

008DB142   .   E8 51E8B2FF   call    00409998                         ; \\FindWindowA  

008DB147   .   85C0          test    eax, eax  

008DB149   .   74 26         je      short 008DB171  

008DB14B   .   6A 00         push    0                                  ; /lParam = 0  

008DB14D   .   6A 01         push    1                                  ; |wParam = 1  

008DB14F   .   6A 10         push    10                                 ; |Message = WM\_CLOSE  

008DB151   .   68 F4B18D00   push    008DB1F4                         ; |/Title = "System Mechanic Tray Notifyer"  

008DB156   .   68 14B28D00   push    008DB214                         ; ||Class = "TfrmSMTrayNotifyMain"  

008DB15B   .   E8 38E8B2FF   call    00409998                         ; |\\FindWindowA  

008DB160   .   50            push    eax                                ; |hWnd  

008DB161   .   E8 BAE8B2FF   call    00409D20                         ; \\SendMessageA  

008DB166   .   EB 09         jmp     short 008DB171  

008DB168   >   33D2          xor     edx, edx  

008DB16A   .   8BC3          mov     eax, ebx  

008DB16C   .   E8 CFF6FFFF   call    008DA840  

008DB171   >   33C0          xor     eax, eax  

008DB173   .   5A            pop     edx  

008DB174   .   59            pop     ecx  

008DB175   .   59            pop     ecx  

来到这里  

0060F900  /$   55           push    ebp  

0060F901  |.   8BEC         mov     ebp, esp  

0060F903  |.   6A 00        push    0  

0060F905  |.   6A 00        push    0  

0060F907  |.   6A 00        push    0  

0060F909  |.   53           push    ebx  

0060F90A  |.   56           push    esi  

0060F90B  |.   57           push    edi  

0060F90C  |.   8BF9         mov     edi, ecx  

0060F90E  |.   8BF2         mov     esi, edx  

0060F910  |.   8BD8         mov     ebx, eax  

0060F912  |.   33C0         xor     eax, eax  

0060F914  |.   55           push    ebp  

0060F915  |.   68 8EF96000  push    0060F98E  

0060F91A  |.   64:FF30      push    dword ptr fs:[eax]  

0060F91D  |.   64:8920      mov     fs:[eax], esp  

0060F920  |.   C645 FF 00   mov     byte ptr [ebp-1], 0  

0060F924  |.   807B 04 00   cmp     byte ptr [ebx+4], 0  

0060F928  |.   74 49        je      short 0060F973  

0060F92A  |.   85F6         test    esi, esi  

0060F92C  |.   74 45        je      short 0060F973  

0060F92E  |.   85FF         test    edi, edi  

0060F930  |.   74 41        je      short 0060F973  

0060F932  |.   8D45 F8      lea     eax, [ebp-8]  

0060F935  |.   50           push    eax                                ; /Arg1  

0060F936  |.   B1 01        mov     cl, 1                                ; |  

0060F938  |.   8A53 04      mov     dl, [ebx+4]                          ; |  

0060F93B  |.   8BC6         mov     eax, esi                             ; |  

0060F93D  |.   E8 E6EBFFFF  call    0060E528                         ; \\SysMech6.0060E528  

0060F942  |.   8B55 F8      mov     edx, [ebp-8]  

0060F945  |.   8BC7         mov     eax, edi  

0060F947  |.   E8 2065DFFF  call    00405E6C  

0060F94C  |.   74 20        je      short 0060F96E  

0060F94E  |.   8D45 F4      lea     eax, [ebp-C]  

0060F951  |.   50           push    eax                                ; /Arg1  

0060F952  |.   33C9         xor     ecx, ecx                            ; |  

0060F954  |.   8A53 04      mov     dl, [ebx+4]                          ; |  

0060F957  |.   8BC6         mov     eax, esi                             ; |  

0060F959  |.   E8 CAEBFFFF  call    0060E528                         ; \\SysMech6.0060E528  

0060F95E  |.   8B55 F4      mov     edx, [ebp-C]  

0060F961  |.   8BC7         mov     eax, edi  

0060F963  |.   E8 0465DFFF  call    00405E6C  

0060F968  |.   74 04        je      short 0060F96E  

0060F96A  |.   33C0         xor     eax, eax  

0060F96C  |.   EB 02        jmp     short 0060F970  

0060F96E  |>   B0 01        mov     al, 1  

0060F970  |>   8845 FF      mov     [ebp-1], al  

0060F973  |>   33C0         xor     eax, eax  

0060F975  |.   5A           pop     edx  

0060F976  |.   59           pop     ecx  

0060F977  |.   59           pop     ecx  

0060F978  |.   64:8910      mov     fs:[eax], edx  

0060F97B  |.   68 95F96000  push    0060F995  

0060F980  |>   8D45 F4      lea     eax, [ebp-C]  

0060F983  |.   BA 02000000  mov     edx, 2  

0060F988  |.   E8 CB60DFFF  call    00405A58  

0060F98D  \\.  C3           retn  

来到这里：  

0060E528  /$   55           push    ebp  
0060E529  |.   8BEC         mov     ebp, esp  
0060E52B  |.   83C4 F8      add     esp, -8  
0060E52E  |.   53           push    ebx  
0060E52F  |.   56           push    esi  
0060E530  |.   57           push    edi  
0060E531  |.   884D FB      mov     [ebp-5], cl  
0060E534  |.   8BDA         mov     ebx, edx  
0060E536  |.   8945 FC      mov     [ebp-4], eax  
0060E539  |.   8B7D 08      mov     edi, [ebp+8]  
0060E53C  |.   807D FB 00   cmp     byte ptr [ebp-5], 0  
0060E540  |.   74 07        je      short 0060E549  
0060E542  |.   BE 1A000000  mov     esi, 1A  
0060E547  |.   EB 02        jmp     short 0060E54B  
0060E549  |>   33F6         xor     esi, esi  
0060E54B  |>   33C0         xor     eax, eax  
0060E54D  |.   8AC3         mov     al, bl  
0060E54F  |.   83F8 0B      cmp     eax, 0B                              ;  Switch (cases 1..B)  
0060E552  |.   0F87 14010000 ja     0060E66C  
0060E558  |.   FF2485 5FE560>jmp    [eax*4+60E55F]                      ;  跳转到：`SysMech6.0060E646`  
0060E55F  |.   6CE66000     dd      `SysMech6.0060E66C`                  ;  分支表 被用于 `0060E558`  
0060E563  |.   8FE56000     dd      `SysMech6.0060E58F`  
0060E567  |.   A5E56000     dd      `SysMech6.0060E5A5`  

来到这里：  

0060E646  |>   \\56         push    esi                                ;  Case A of switch `0060E54F`  
0060E647  |.   57           push    edi  
0060E648  |.   8BCB         mov     ecx, ebx  
0060E64A  |.   BA 10000000  mov     edx, 10  
0060E64F  |.   8B45 FC      mov     eax, [ebp-4]  



0060E652 |. E8 89FCFFFF call 0060E2E0 跟进  

0060E657 |. EB 1A jmp short 0060E673  

0060E659 |> 56 push esi ; Case B of switch 0060E54F  

0060E65A |. 57 push edi  

0060E65B |. 8BCB mov ecx, ebx  

0060E65D |. BA 1C000000 mov edx, 1C  

0060E662 |. 8B45 FC mov eax, [ebp-4]  

0060E665 |. E8 76FCFFFF call 0060E2E0  

0060E66A |. EB 07 jmp short 0060E673  

0060E66C |> 8BC7 mov eax, edi ; Default case of switch 0060E54F  

0060E66E |. E8 C173DFFF call 00405A34  

0060E673 |> 5F pop edi  

0060E674 |. 5E pop esi  

0060E675 |. 5B pop ebx  

0060E676 |. 59 pop ecx  

0060E677 |. 59 pop ecx  

0060E678 |. 5D pop ebp  

0060E679 \. C2 0400 retn 4  

来到这里：  

```
0060E2E0 /$ 55 push ebp
0060E2E1 |. 8BEC mov ebp, esp
0060E2E3 |. 51 push ecx
0060E2E4 |. B9 06000000 mov ecx, 6
0060E2E9 |> 6A 00 /push 0
0060E2EB |. 6A 00 |push 0
0060E2ED |. 49 |dec ecx
0060E2EE |.^ 75 F9 \jnz short 0060E2E9
0060E2F0 |. 874D FC xchg [ebp-4], ecx
0060E2F3 |. 53 push ebx
0060E2F4 |. 56 push esi
0060E2F5 |. 57 push edi
0060E2F6 |. 884D FB mov [ebp-5], cl
0060E2F9 |. 8BFA mov edi, edx
0060E2FB |. 8945 FC mov [ebp-4], eax
0060E2FE |. 8B45 FC mov eax, [ebp-4]
0060E301 |. E8 0A7CDFFF call 00405F10
0060E306 |. 33C0 xor eax, eax
0060E308 |. 55 push ebp
0060E309 |. 68 08E56000 push 0060E508
0060E30E |. 64:FF30 push dword ptr fs:[eax]
0060E311 |. 64:8920 mov fs:[eax], esp
0060E314 |. BB 21000000 mov ebx, 21 ; 将 21 赋予 EBX，21 是字符!的 ASC 码值
0060E319 |. EB 16 jmp short 0060E331
0060E31B |> 8D45 EC /lea eax, [ebp-14]
0060E31E |. 8BD3 |mov edx, ebx
0060E320 |. E8 F778DFFF |call 00405C1C
0060E325 |. 8B55 EC |mov edx, [ebp-14]
0060E328 |. 8D45 FC |lea eax, [ebp-4]
0060E32B |. E8 F879DFFF |call 00405D28
0060E330 |. 43 |inc ebx ; 如果用户名长度不够十位，则从!开始补够。
0060E331 |> 8B45 FC mov eax, [ebp-4]
0060E334 |. E8 E779DFFF |call 00405D20 ; 计算用户名的长度
0060E339 |. 83F8 0A |cmp eax, 0A ; 与 A 比较，小于十位则跳转到 0060E31B
0060E33C |.^ 7C DD \jl short 0060E31B
0060E33E |. 8D55 E8 lea edx, [ebp-18]
0060E341 |. 8B45 FC mov eax, [ebp-4]
0060E344 |. E8 A7D0DFFF call 0040B3F0 ; 将用户名转换为大写
0060E349 |. 8B55 E8 mov edx, [ebp-18]
0060E34C |. 8D45 FC lea eax, [ebp-4]
0060E34F |. E8 7877DFFF call 00405ACC
0060E354 |. 8B45 FC mov eax, [ebp-4]
0060E357 |. E8 C479DFFF call 00405D20 ; 计算用户名的长度（如果用户名含有中文字符
0060E35C |. 8BF0 mov esi, eax ; 则一个中文字符按两位算）
0060E35E |. 85F6 test esi, esi ; ESI＝用户名的长度
0060E360 |. 7E 5E jle short 0060E3C0
0060E362 |. BB 01000000 mov ebx, 1
0060E367 |> 8B45 FC /mov eax, [ebp-4]
0060E36A |. 0FB64418 FF |movzx eax, byte ptr [eax+ebx-1] ; 按位取用户名 ASC 码值赋予 EAX
0060E36F |. 8B55 0C |mov edx, [ebp+C] ; [ebp+C]＝1A
0060E372 |. 83C2 46 |add edx, 46 ; 1A+46＝EDX＝60
0060E375 |. 3BC2 |cmp eax, edx ; EAx 与 EDX 比较
0060E377 |. 7E 22 |jle short 0060E39B ; 小于等于零则跳转到 0060E39B
0060E379 |. 8B45 FC |mov eax, [ebp-4]
0060E37C |. 0FB64418 FF |movzx eax, byte ptr [eax+ebx-1] ; 按位取用户名 ASC 码值赋予 EAX
0060E381 |. 8D143B |lea edx, [ebx+edi]
0060E384 |. 2BC2 |sub eax, edx ; EAX＝EAX－EDX
0060E386 |. 8D55 E4 |lea edx, [ebp-1C]
0060E389 |. E8 BEDBDFFF |call 0040BF4c ; 算法运算部分，将 EAx 的值转换为字符串。
0060E38E |. 8B55 E4 |mov edx, [ebp-1C]
0060E391 |. 8D45 F4 |lea eax, [ebp-C]
0060E394 |. E8 8F79DFFF |call 00405D28
0060E399 |. EB 20 |jmp short 0060E3BB
0060E39B |> 8B45 FC |mov eax, [ebp-4]
0060E39E |. 0FB64418 FF |movzx eax, byte ptr [eax+ebx-1] ; 按位取用户名 ASC 码值赋予 EAX
0060E3A3 |. 8D143B |lea edx, [ebx+edi] ; EBX＝1 （专业版时 EDI＝10，标准版时 EDI=26,移动版时 EDI＝1C） EDX＝EBX＋EDI
0060E3A6 |. 03C2 |add eax, edx ; EAX＝EAX＋EDX
0060E3A8 |. 8D55 E0 |lea edx, [ebp-20]
0060E3AB |. E8 9CDBDFFF |call 0040BF4C ; 算法运算部分，将 EAx 的值转换为字符串。
0060E3B0 |. 8B55 E0 |mov edx, [ebp-20]
0060E3B3 |. 8D45 F4 |lea eax, [ebp-C]
0060E3B6 |. E8 6D79DFFF |call 00405D28
0060E3BB |> 47 |inc edi ; EDI＋1
0060E3BC |. 43 |inc ebx ; EBX＋1
0060E3BD |. 4E dec esi
0060E3BE |.^ 75 A7 \jnz short 0060E367 ; 直到将用户名全部运算完毕。
0060E3C0 |> BB 31000000 mov ebx, 31
0060E3C5 |. EB 20 jmp short 0060E3E7 ; 跳转到 0060E3E7
0060E3C7 |> 8D45 DC /lea eax, [ebp-24]
0060E3CA |. 8BD3 |mov edx, ebx
0060E3CC |. E8 4B78DFFF |call 00405C1C
0060E3D1 |. 8B55 DC |mov edx, [ebp-24]
0060E3D4 |. 8D45 F4 |lea eax, [ebp-C]
0060E3D7 |. E8 4C79DFFF |call 00405D28
0060E3DC |. 43 inc ebx
0060E3DD |. 83FB 39 |cmp ebx, 39 ; 如果算法运算部分产生的字符串长度不够 20 位
0060E3E0 |. 75 05 |jnz short 0060E3E7 ; 则用 12345678 补够。
0060E3E2 |. BB 31000000 mov ebx, 31
0060E3E7 |> 8B45 F4 mov eax, [ebp-C] ; 算法运算部分产生的字符串赋予 EAX
0060E3EA |. E8 3179DFFF |call 00405D20 ; 计算其长度
0060E3EF |. 83F8 14 |cmp eax, 14 ; 要求最少 20 位，否者跳转到 0060E3C7
0060E3F2 |.^ 7C D3 \jl short 0060E3C7 ; ＝＝＝＝＝＝＝＝＝＝＝＝＝＝
0060E3F4 |. BB 01000000 mov ebx, 1
0060E3F9 |> 8D45 D8 /lea eax, [ebp-28]
0060E3FC |. 8B55 F4 |mov edx, [ebp-C]
0060E3FF |. 8A541A FF |mov dl, [edx+ebx-1]
0060E403 |. E8 1478DFFF |call 00405C1C ; 取 EAX 的前 5 位
0060E408 |. 8B55 D8 |mov edx, [ebp-28]
0060E40B |. 8D45 F0 |lea eax, [ebp-10]
0060E40E |. E8 1579DFFF |call 00405D28
0060E413 |. 43 inc ebx
0060E414 |. 83FB 06 |cmp ebx, 6
0060E417 |.^ 75 E0 \jnz short 0060E3F9 ; ＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
0060E419 |. 8D45 F0 lea eax, [ebp-10]
```



```
0060E41C  |.  BA 20E56000   mov     edx, 0060E520  

0060E421  |.  E8 0279DFFF   call    00405D28  

0060E426  |.  8D45 F0       lea     eax, [ebp-10]  

0060E429  |.  33D2          xor     edx, edx  

0060E42B  |.  8A55 FB       mov     dl, [ebp-5]  

0060E42E  |.  8B1495 D861AB>mov     edx, [edx*4+AB61D8]  

0060E435  |.  E8 EE78DFFF   call    00405D28  

0060E43A  |.  BB 03000000   mov     ebx, 3  

0060E43F  |>   8B45 F4       /mov     eax, [ebp-C]  

0060E442  |.  E8 D978DFFF   |call    00405D20  

0060E447  |.  8945 D0       |mov     [ebp-30], eax  

0060E44A  |.  DB45 D0       |fild    dword ptr [ebp-30]                 将字串的长度转换为十进制  

0060E44D  |.  D835 24E56000 |fdiv    dword ptr [60E524]                 将字串的长度除以 2  

0060E453  |.  E8 9C50DFFF   |call    004034F4  

0060E458  |.  83C0 03       |add     eax, 3  

0060E45B  |.  83D2 00       |adc     edx, 0  

0060E45E  |.  52            |push    edx  

0060E45F  |.  50            |push    eax  

0060E460  |.  8BC3          |mov     eax, ebx  

0060E462  |.  99            |cdq    

0060E463  |.  290424        |sub     [esp], eax  

0060E466  |.  195424 04     |sbb     [esp+4], edx  

0060E46A  |.  58            |pop     eax                              ＝＝＝＝＝＝＝＝＝＝＝＝＝  

0060E46B  |.  5A            |pop     edx  

0060E46C  |.  8B55 F4       |mov     edx, [ebp-C]  

0060E46F  |.  8A5402 FF     |mov     dl, [edx+eax-1]                   从字串的中间向后取三位  

0060E473  |.  8D45 D4       |lea     eax, [ebp-2C]  

0060E476  |.  E8 A177DFFF   |call    00405C1C  

0060E47B  |.  8B55 D4       |mov     edx, [ebp-2C]  

0060E47E  |.  8D45 F0       |lea     eax, [ebp-10]  

0060E481  |.  E8 A278DFFF   |call    00405D28  

0060E486  |.  4B            |dec     ebx  

0060E487  |.  85DB         |test    ebx, ebx  

0060E489  |.^ 75 B4        \jnz     short 0060E43F                  ＝＝＝＝＝＝＝＝＝＝＝＝＝＝  

0060E48B  |.  8D45 F0       lea     eax, [ebp-10]  

0060E48E  |.  BA 20E56000   mov     edx, 0060E520  

0060E493  |.  E8 9078DFFF   call    00405D28  

0060E498  |.  8B45 F4       mov     eax, [ebp-C]  

0060E49B  |.  E8 8078DFFF   call    00405D20  

0060E4A0  |.  8BD8          mov     ebx, eax  

0060E4A2  |.  8B45 F4       mov     eax, [ebp-C]  

0060E4A5  |.  E8 7678DFFF   call    00405D20  

0060E4AA  |.  8BF0          mov     esi, eax  

0060E4AC  |.  83EE 09       sub     esi, 9                          ＝＝＝＝＝＝＝＝＝＝＝＝＝＝  

0060E4AF  |.  2BF3          sub     esi, ebx                        

0060E4B1  |.  7F 1F         jg      short 0060E4D2  

0060E4B3  |.  4E            dec     esi  

0060E4B4  |>   8D45 CC       /lea     eax, [ebp-34]  

0060E4B7  |.  8B55 F4       |mov     edx, [ebp-C]  

0060E4BA  |.  8A541A FF     |mov     dl, [edx+ebx-1]                 从字串的末尾向前取 10 位  

0060E4BE  |.  E8 5977DFFF   |call    00405C1C  

0060E4C3  |.  8B55 CC       |mov     edx, [ebp-34]  

0060E4C6  |.  8D45 F0       |lea     eax, [ebp-10]  

0060E4C9  |.  E8 5A78DFFF   |call    00405D28  

0060E4CE  |.  4B            |dec     ebx  

0060E4CF  |.  46            |inc     esi  

0060E4D0  |.^ 75 E2        \jnz     short 0060E4B4                  ＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝  

0060E4D2  |>   8B45 08       mov     eax, [ebp+8]  

0060E4D5  |.  8B55 F0       mov     edx, [ebp-10]                   前 5 位－（专业版为 P6，标准版为 S6，移动版为 M6）＋中间三位－后 10 位  

0060E4D8  |.  E8 AB75DFFF   call    00405A88  

0060E4DD  |.  33C0          xor     eax, eax  

0060E4DF  |.  5A            pop     edx  

0060E4E0  |.  59            pop     ecx  

0060E4E1  |.  59            pop     ecx  

0060E4E2  |.  64:8910       mov     fs:[eax], edx  

0060E4E5  |.  68 0FE56000   push    0060E50F  

0060E4EA  |>   8D45 CC       lea     eax, [ebp-34]  

0060E4ED  |.  E8 4275DFFF   call    00405A34  

0060E4F2  |.  8D45 D4       lea     eax, [ebp-2C]  

0060E4F5  |.  BA 09000000   mov     edx, 9  

0060E4FA  |.  E8 5975DFFF   call    00405A58  

0060E4FF  |.  8D45 FC       lea     eax, [ebp-4]  

0060E502  |.  E8 2D75DFFF   call    00405A34  

0060E507  \\.  C3           retn  

0060E508   .^ E9 7F6DDFFF   jmp     0040528C  

0060E50D   .^ EB DB        jmp     short 0060E4EA  

0060E50F    . 5F           pop     edi  

0060E510    . 5E           pop     esi  

0060E511    . 5B           pop     ebx  

0060E512    . 8BE5         mov     esp, ebp  

0060E514    . 5D           pop     ebp  

0060E515    . C2 0800      retn    8  
```

### 算法运算部分：

```
0040BF4C  /$  56           push    esi  

0040BF4D  |.  89E6        mov     esi, esp  

0040BF4F  |.  83EC 10     sub     esp, 10  

0040BF52  |.  31C9        xor     ecx, ecx  

0040BF54  |.  52           push    edx  

0040BF55  |.  31D2        xor     edx, edx  

0040BF57  |.  E8 A4FFFFFF call    0040BF00                     跟进  

0040BF5C  |.  89F2        mov     edx, esi  

0040BF5E  |.  58           pop     eax  

0040BF5F  |.  E8 C09BFFFF call    00405B24  

0040BF64  |.  83C4 10     add     esp, 10  

0040BF67  |.  5E           pop     esi  

0040BF68  \\.  C3          retn  
```

### 来到这里：

```
0040BF00  /$  08C9        or      cl, cl  

0040BF02  |.  75 17       jnz     short 0040BF1B  

0040BF04  |.  09C0        or      eax, eax  

0040BF06  |.  79 0E       jns     short 0040BF16              跳转  

0040BF08  |.  F7D8        neg     eax  

0040BF0A  |.  E8 07000000 call    0040BF16  

0040BF0F  |.  B0 2D       mov     al, 2D  

0040BF11  |.  41           inc     ecx  

0040BF12  |.  4E           dec     esi  

0040BF13  |.  8806        mov     [esi], al  

0040BF15  |.  C3           retn  

0040BF16  |$  B9 0A000000 mov     ecx, 0A                 赋 ECX 值 0A  

0040BF1B  |>   52           push    edx  

0040BF1C  |.  56           push    esi  

0040BF1D  |>   31D2         /xor      edx, edx  

0040BF1F  |.  F7F1         |div     ecx  

0040BF21  |.  4E           |dec     esi  

0040BF22  |.  80C2 30      |add     dl, 30  

0040BF25  |.  80FA 3A      |cmp     dl, 3A  

0040BF28  |.  72 03        |jb      short 0040BF2D  

0040BF2A  |.  80C2 07      |add     dl, 7  

0040BF2D  |>   8816        |mov     [esi], dl  

0040BF2F  |.  09C0         |or      eax, eax  

0040BF31  |.^ 75 EA        \jnz     short 0040BF1D  

0040BF33  |.  59           pop     ecx  

0040BF34  |.  5A           pop     edx  

0040BF35  |.  29F1        sub     ecx, esi  

0040BF37  |.  29CA        sub     edx, ecx  

0040BF39  |.  76 10       jbe     short 0040BF4B  

0040BF3B  |.  01D1        add     ecx, edx  

0040BF3D  |.  B0 30       mov     al, 30  

0040BF3F  |.  29D6        sub     esi, edx  

0040BF41  |.  EB 03       jmp     short 0040BF46  

0040BF43  |>   880432       /mov     [edx+esi], al  

0040BF46  |>   4A           dec     edx  

0040BF47  |.^ 75 FA        \jnz     short 0040BF43  

0040BF49  |.  8806        mov     [esi], al  

0040BF4B  \\.  C3          retn  
```

### 专业版注册机的源代码：

```vb
Private Sub Text1_Change()  

Dim name As String, id As String, s As String  

Dim ln As Integer, asci As Integer  

name = StrConv(Text1.Text, 1)  

ln = Len(name)  

If ln < 10 Then  
a = 32  
For m = ln To 9  
a = a + 1  
name = name + Chr(a)  
Next  
End If  
B = 16  
c = 1  
ln = Len(name)  
For n = 1 To ln  
asci = Asc(Mid(name, n, 1))  
If asci <= 96 Then  
asci = asci + B + c  
s = Trim(s) + Trim(Str(asci))  
Else  
asci = asci - B - c  
s = Trim(s) + Trim(Str(asci))  
End If  
B = B + 1  
c = c + 1  
Next  
ln = Len(s)  
s1 = Mid(s, Int(ln / 2), 3)  
s2 = Right(s, 10)  
n = Len(s2)  
Do  
s22 = Trim(s22) + Trim(Mid(s2, n, 1))  
n = n - 1  
Loop While n > 0  
Text2.Text = Left(s, 5) + "-" + "P6" + s1 + "-" + s22  
End Sub
```

另外，哪位大虾指点一下，如果用户名为中文的话，一个字符在 `VB` 中如何按两位进行运算，并且将其转换为十进制。在这里先谢谢了。

--------------------------------------------------------------------------------

**【版权声明】**: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!



2006 年 11 月 27 日 13:17:00

`4dd466d0e1e0cc123afbe504da2462ca`
