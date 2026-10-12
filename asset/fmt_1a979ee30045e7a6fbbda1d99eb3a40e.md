
# 音乐报时系统 4.0.2

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-461.htm`](https://www.pediy.com/kssd/pediy08/pediy8-461.htm)

【破文标题】音乐报时系统 4.0.2 注册算法

【破文作者】XXNB

【作者邮箱】

【作者主页】`http://free.ys168.com/?binbinbin7456`

【破解工具】`OD`

【破解平台】`xpsp2`

【软件名称】音乐报时系统 4.0.2

【软件大小】`2512kb`

【原版下载】`http://www.newhua.com/soft/53310.htm`

【保护方式】机器码

【软件简介】音乐报时系统 V4.0 版是一款针对学校作息时间控制而定制的学校电铃替代软件。音乐报时系统第一版至第三版经历了四年时间供 6 所学校免费使用，经测试非常稳定。V4.0 版新改进的任务策略更方便完成周一升旗和考试时间的控制。软件提供了详细的软、硬件设置说明。

【破解声明】向大侠们学习！！！只为学习！请尊重作者的劳动成功！

------------------------------------------------------------------------

【破解过程】

1、用 `VB` 专用的任何一个断点都能轻松断下。经过分析，下面这段代码是关键。

```
00472C20   $  55             push    ebp
00472C21   .  8BEC           mov     ebp, esp
00472C23   .  83EC 0C        sub     esp, 0C
00472C26   .  68 B6204000    push    <jmp.&MSVBVM60.__vbaExceptHandle>;  SE 处理程序安装
00472C2B   .  64:A1 0000000>mov     eax, dword ptr fs:[0]
00472C31   .  50             push    eax
00472C32   .  64:8925 00000>mov     dword ptr fs:[0], esp
00472C39   .  81EC 44010000  sub     esp, 144
00472C3F   .  53             push    ebx
00472C40   .  56             push    esi
00472C41   .  57             push    edi
00472C42   .  8965 F4       mov     dword ptr [ebp-C], esp
00472C45   .  C745 F8 101C4>mov     dword ptr [ebp-8], 00401C10
00472C4C   .  BB 02000000   mov     ebx, 2
00472C51   .  8D45 A0       lea     eax, dword ptr [ebp-60]
00472C54   .  53             push    ebx
00472C55   .  33FF           xor     edi, edi
00472C57   .  68 043A4200   push    00423A04
00472C5C   .  50             push    eax
00472C5D   .  897D DC       mov     dword ptr [ebp-24], edi
00472C60   .  897D CC       mov     dword ptr [ebp-34], edi
00472C63   .  897D BC       mov     dword ptr [ebp-44], edi
00472C66   .  897D B8       mov     dword ptr [ebp-48], edi
00472C69   .  897D 98       mov     dword ptr [ebp-68], edi
00472C6C   .  897D 94       mov     dword ptr [ebp-6C], edi
00472C6F   .  897D 90       mov     dword ptr [ebp-70], edi
00472C72   .  897D 80       mov     dword ptr [ebp-80], edi
00472C75   .  89BD 70FFFFFF mov     dword ptr [ebp-90], edi
00472C7B   .  89BD 60FFFFFF mov     dword ptr [ebp-A0], edi
00472C81   .  89BD 50FFFFFF mov     dword ptr [ebp-B0], edi
00472C87   .  89BD 40FFFFFF mov     dword ptr [ebp-C0], edi
00472C8D   .  89BD 30FFFFFF mov     dword ptr [ebp-D0], edi
00472C93   .  89BD 2CFFFFFF mov     dword ptr [ebp-D4], edi
00472C99   .  89BD 18FFFFFF mov     dword ptr [ebp-E8], edi
00472C9F   .  89BD 08FFFFFF mov     dword ptr [ebp-F8], edi
00472CA5   .  89BD F8FEFFFF mov     dword ptr [ebp-108], edi
00472CAB   .  89BD E8FEFFFF mov     dword ptr [ebp-118], edi
00472CB1   .  89BD D8FEFFFF mov     dword ptr [ebp-128], edi
00472CB7   .  89BD C8FEFFFF mov     dword ptr [ebp-138], edi
00472CBD   .  FF15 D8104000 call    dword ptr [<&MSVBVM60.__vbaAryCo>;  MSVBVM60.__vbaAryConstruct2
00472CC3   .  8D8D 50FFFFFF lea     ecx, dword ptr [ebp-B0]
00472CC9   .  8D95 40FFFFFF lea     edx, dword ptr [ebp-C0]
00472CCF   .  51             push    ecx
00472CD0   .  8D85 30FFFFFF lea     eax, dword ptr [ebp-D0]
00472CD6   .  52             push    edx
00472CD7   .  8D8D 08FFFFFF lea     ecx, dword ptr [ebp-F8]
00472CDD   .  50             push    eax
00472CDE   .  8D95 18FFFFFF lea     edx, dword ptr [ebp-E8]
00472CE4   .  51             push    ecx
00472CE5   .  8D45 DC       lea     eax, dword ptr [ebp-24]
00472CE8   .  BE 01000000   mov     esi, 1
00472CED   .  52             push    edx
00472CEE   .  50             push    eax
00472CEF   .  89B5 58FFFFFF mov     dword ptr [ebp-A8], esi
00472CF5   .  899D 50FFFFFF mov     dword ptr [ebp-B0], ebx
00472CFB   .  C785 48FFFFFF>mov     dword ptr [ebp-B8], 6
00472D05   .  899D 40FFFFFF mov     dword ptr [ebp-C0], ebx
00472D0B   .  89B5 38FFFFFF mov     dword ptr [ebp-C8], esi
00472D11   .  899D 30FFFFFF mov     dword ptr [ebp-D0], ebx
00472D17   .  FF15 80104000 call    dword ptr [<&MSVBVM60.__vbaVarFo>;  MSVBVM60.__vbaVarForInit
00472D1D   >  3BC7           cmp     eax, edi
00472D1F   .  0F84 DD000000 je      00472E02
00472D25   .  8B4D 0C       mov     ecx, dword ptr [ebp+C]
00472D28   .  8D55 80       lea     edx, dword ptr [ebp-80]
00472D2B   .  8D45 DC       lea     eax, dword ptr [ebp-24]
00472D2E   .  8975 88       mov     dword ptr [ebp-78], esi
00472D31   .  8B35 BC114000 mov     esi, dword ptr [<&MSVBVM60.__vba>;  MSVBVM60.__vbaI4Var
00472D37   .  52             push    edx
00472D38   .  50             push    eax
00472D39   .  895D 80       mov     dword ptr [ebp-80], ebx
00472D3C   .  898D 58FFFFFF mov     dword ptr [ebp-A8], ecx
00472D42   .  C785 50FFFFFF>mov     dword ptr [ebp-B0], 4008
00472D4C   .  FFD6          call    esi                              ;  <&MSVBVM60.__vbaI4Var>
00472D4E   .  8D8D 50FFFFFF lea     ecx, dword ptr [ebp-B0]
00472D54   .  50             push    eax
00472D55   .  8D95 70FFFFFF lea     edx, dword ptr [ebp-90]
00472D5B   .  51             push    ecx
00472D5C   .  52             push    edx
00472D5D   .  FF15 B8104000 call    dword ptr [<&MSVBVM60.#632>]     ;  MSVBVM60.rtcMidCharVar
00472D63   .  8D45 DC       lea     eax, dword ptr [ebp-24]
00472D66   .  8D8D 30FFFFFF lea     ecx, dword ptr [ebp-D0]
00472D6C   .  50             push    eax
00472D6D   .  8D95 60FFFFFF lea     edx, dword ptr [ebp-A0]
00472D73   .  51             push    ecx
00472D74   .  52             push    edx
00472D75   .  C785 38FFFFFF>mov     dword ptr [ebp-C8], 1
00472D7F   .  899D 30FFFFFF mov     dword ptr [ebp-D0], ebx
00472D85   .  FF15 04104000 call    dword ptr [<&MSVBVM60.__vbaVarSu>;  MSVBVM60.__vbaVarSub
00472D8B   .  50             push    eax
00472D8C   .  FFD6          call    esi
00472D8E   .  8BF0          mov     esi, eax
00472D90   .  83FE 07       cmp     esi, 7
00472D93   .  72 06         jb      short 00472D9B
00472D95   .  FF15 CC104000 call    dword ptr [<&MSVBVM60.__vbaGener>;  MSVBVM60.__vbaGenerateBoundsError
00472D9B   >  8D85 70FFFFFF lea     eax, dword ptr [ebp-90]
00472DA1   .  8D4D 90       lea     ecx, dword ptr [ebp-70]
00472DA4   .  50             push    eax
00472DA5   .  51             push    ecx
00472DA6   .  FF15 54114000 call    dword ptr [<&MSVBVM60.__vbaStrVa>;  MSVBVM60.__vbaStrVarVal
00472DAC   .  50             push    eax
00472DAD   .  FF15 3C104000 call    dword ptr [<&MSVBVM60.#516>]     ;  MSVBVM60.rtcAnsiValueBstr
00472DB3   .  8BC8          mov     ecx, eax
00472DB5   .  FF15 48104000 call    dword ptr [<&MSVBVM60.__vbaI2Abs>;  MSVBVM60.__vbaI2Abs
00472DBB   .  8B55 AC       mov     edx, dword ptr [ebp-54]
00472DBE   .  8D4D 90       lea     ecx, dword ptr [ebp-70]
00472DC1   .  66:890472    mov     word ptr [edx+esi*2], ax
00472DC5   .  FF15 28124000 call    dword ptr [<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr
00472DCB   .  8D85 70FFFFFF lea     eax, dword ptr [ebp-90]
00472DD1   .  8D4D 80       lea     ecx, dword ptr [ebp-80]
00472DD4   .  50             push    eax
00472DD5   .  51             push    ecx
00472DD6   .  53             push    ebx
00472DD7   .  FF15 2C104000 call    dword ptr [<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVarList
00472DDD   .  83C4 0C       add     esp, 0C
00472DE0   .  8D95 08FFFFFF lea     edx, dword ptr [ebp-F8]
00472DE6   .  8D85 18FFFFFF lea     eax, dword ptr [ebp-E8]
00472DEC   .  8D4D DC       lea     ecx, dword ptr [ebp-24]
00472DEF   .  52             push    edx
00472DF0   .  50             push    eax
00472DF1   .  51             push    ecx
```
