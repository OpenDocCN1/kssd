
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



```
00472DF2   .  FF15 1C124000 call    dword ptr [<&MSVBVM60.__vbaVarFo>;  MSVBVM60.__vbaVarForNext  

00472DF8   .  BE 01000000   mov     esi, 1  

00472DFD   .^ E9 1BFFFFFF   jmp     00472D1D  

00472E02   >  8B55 AC       mov     edx, dword ptr [ebp-54]          ;  用户名出现了  

00472E05   .  66:8B02       mov     ax, word ptr [edx]               ;  取用户名第一位  

00472E08   .  50           push    eax                            ;  转成 10 进制字符串  

00472E09   .  FF15 08104000 call    dword ptr [<&MSVBVM60.__vbaStrI2>;  MSVBVM60.__vbaStrI2  

00472E0F   .  8B3D F4114000 mov     edi, dword ptr [<&MSVBVM60.__vba>;  MSVBVM60.__vbaStrMove  

00472E15   .  8BD0          mov     edx, eax  

00472E17   .  8D4D B8       lea     ecx, dword ptr [ebp-48]  

00472E1A   .  FFD7         call    edi                            ;  <&MSVBVM60.__vbaStrMove>  

00472E1C   .  8D8D 50FFFFFF lea     ecx, dword ptr [ebp-B0]  

00472E22   .  8D95 40FFFFFF lea     edx, dword ptr [ebp-C0]  

00472E28   .  51           push    ecx  

00472E29   .  8D85 30FFFFFF lea     eax, dword ptr [ebp-D0]  

00472E2F   .  52           push    edx  

00472E30   .  8D8D E8FEFFFF lea     ecx, dword ptr [ebp-118]  

00472E36   .  50           push    eax  

00472E37   .  8D95 F8FEFFFF lea     edx, dword ptr [ebp-108]  

00472E3D   .  51           push    ecx  

00472E3E   .  8D45 CC       lea     eax, dword ptr [ebp-34]  

00472E41   .  BE 01000000   mov     esi, 1  

00472E46   .  52           push    edx  

00472E47   .  50           push    eax  

00472E48   .  89B5 58FFFFFF mov     dword ptr [ebp-A8], esi  

00472E4E   .  899D 50FFFFFF mov     dword ptr [ebp-B0], ebx  

00472E54   .  C785 48FFFFFF>mov     dword ptr [ebp-B8], 5  

00472E5E   .  899D 40FFFFFF mov     dword ptr [ebp-C0], ebx  

00472E64   .  89B5 38FFFFFF mov     dword ptr [ebp-C8], esi  

00472E6A   .  899D 30FFFFFF mov     dword ptr [ebp-D0], ebx  

00472E70   .  FF15 80104000 call    dword ptr [<&MSVBVM60.__vbaVarFo>;  MSVBVM60.__vbaVarForInit  

00472E76   >  85C0          test    eax, eax                       ;  循环开始  

00472E78   .  74 7C        je      short 00472EF6  

00472E7A   .  8D4D CC       lea     ecx, dword ptr [ebp-34]  

00472E7D   .  51           push    ecx  

00472E7E   .  FF15 BC114000 call    dword ptr [<&MSVBVM60.__vbaI4Var>;  MSVBVM60.__vbaI4Var  

00472E84   .  8BF0         mov     esi, eax                       ;  计数器比较  

00472E86   .  83FE 07       cmp     esi, 7  

00472E89   .  72 06        jb      short 00472E91  

00472E8B   .  FF15 CC104000 call    dword ptr [<&MSVBVM60.__vbaGener>;  MSVBVM60.__vbaGenerateBoundsError  

00472E91   >  8B55 B8       mov     edx, dword ptr [ebp-48]        ;  上一次循环的结果  

00472E94   .  52           push    edx  

00472E95   .  FF15 80114000 call    dword ptr [<&MSVBVM60.__vbaR8Str>;  MSVBVM60.__vbaR8Str  

00472E9B   .  8B45 AC       mov     eax, dword ptr [ebp-54]  

00472E9E   .  83EC 08       sub     esp, 8  

00472EA1   .  0FBF0C70      movsx   ecx, word ptr [eax+esi*2]      ;  取下一个用户名字符  

00472EA5   .  898D B0FEFFFF mov     dword ptr [ebp-150], ecx  

00472EAB   .  DB85 B0FEFFFF fild    dword ptr [ebp-150]            ;  转成实数  

00472EB1   .  DD9D A8FEFFFF fstp    qword ptr [ebp-158]  

00472EB7   .  DC8D A8FEFFFF fmul    qword ptr [ebp-158]            ;  用户名字符 ascii 码值 10 进制累相乘  

00472EBD   .  DFE0          fstsw   ax                             ;  保存状态字的值到 AX  

00472EBF   .  A8 0D         test    al, 0D                         ;  是否是 0D  

00472EC1   .  0F85 86030000 jnz     0047324D  

00472EC7   .  DD1C24        fstp    qword ptr [esp]                ;  这里就是相乘的结果  

00472ECA   .  FF15 04114000 call    dword ptr [<&MSVBVM60.__vbaStrR8>;  MSVBVM60.__vbaStrR8  

00472ED0   .  8BD0          mov     edx, eax                       ;  转 10 进制字符串  

00472ED2   .  8D4D B8       lea     ecx, dword ptr [ebp-48]  

00472ED5   .  FFD7         call    edi  

00472ED7   .  8D95 E8FEFFFF lea     edx, dword ptr [ebp-118]  

00472EDD   .  8D85 F8FEFFFF lea     eax, dword ptr [ebp-108]  

00472EE3   .  52           push    edx  

00472EE4   .  8D4D CC       lea     ecx, dword ptr [ebp-34]  

00472EE7   .  50           push    eax  

00472EE8   .  51           push    ecx  

00472EE9   .  FF15 1C124000 call    dword ptr [<&MSVBVM60.__vbaVarFo>;  MSVBVM60.__vbaVarForNext  

00472EEF   .  BE 01000000   mov     esi, 1                         ;  计数器  

00472EF4   .^ EB 80        jmp     short 00472E76                 ;  循环回去继续  

00472EF6   >  8B55 B8       mov     edx, dword ptr [ebp-48]        ;  这里出现的就是上面循环用户名后得到的字符串“”  

00472EF9   .  52           push    edx  

00472EFA   .  FF15 80114000 call    dword ptr [<&MSVBVM60.__vbaR8Str>;  MSVBVM60.__vbaR8Str  

00472F00   .  8B45 08       mov     eax, dword ptr [ebp+8]  

00472F03   .  83EC 08       sub     esp, 8  

00472F06   .  DC00          fadd    qword ptr [eax]                ;  实数加。用户名的结果+机器码  

00472F08   .  DFE0          fstsw   ax  

00472F0A   .  A8 0D         test    al, 0D  

00472F0C   .  0F85 3B030000 jnz     0047324D  

00472F12   .  DD1C24        fstp    qword ptr [esp]                ;  这里是相加的结果  

00472F15   .  FF15 04114000 call    dword ptr [<&MSVBVM60.__vbaStrR8>;  MSVBVM60.__vbaStrR8  

00472F1B   .  8BD0          mov     edx, eax                       ;  转字符串  "1281198006758"  

00472F1D   .  8D4D B8       lea     ecx, dword ptr [ebp-48]  

00472F20   .  FFD7         call    edi  

00472F22   .  8D4D B8       lea     ecx, dword ptr [ebp-48]  

00472F25   .  8D55 80       lea     edx, dword ptr [ebp-80]  

00472F28   .  898D 58FFFFFF mov     dword ptr [ebp-A8], ecx  

00472F2E   .  52           push    edx  

00472F2F   .  8D85 50FFFFFF lea     eax, dword ptr [ebp-B0]  

00472F35   .  56           push    esi  

00472F36   .  8D8D 70FFFFFF lea     ecx, dword ptr [ebp-90]  

00472F3C   .  50           push    eax  

00472F3D   .  51           push    ecx  

00472F3E   .  8975 88       mov     dword ptr [ebp-78], esi  

00472F41   .  895D 80       mov     dword ptr [ebp-80], ebx  

00472F44   .  C785 50FFFFFF>mov     dword ptr [ebp-B0], 4008  

00472F4E   .  FF15 B8104000 call    dword ptr [<&MSVBVM60.#632>]     ;  MSVBVM60.rtcMidCharVar  

00472F54   .  8D95 70FFFFFF lea     edx, dword ptr [ebp-90]  

00472F5A   .  52           push    edx  

00472F5B   .  FF15 20104000 call    dword ptr [<&MSVBVM60.__vbaStrVa>;  MSVBVM60.__vbaStrVarMove  

00472F61   .  8BD0          mov     edx, eax  

00472F63   .  8D4D 98       lea     ecx, dword ptr [ebp-68]  

00472F66   .  FFD7         call    edi  

00472F68   .  8D85 70FFFFFF lea     eax, dword ptr [ebp-90]  

00472F6E   .  8D4D 80       lea     ecx, dword ptr [ebp-80]  

00472F71   .  50           push    eax  

00472F72   .  51           push    ecx  

00472F73   .  53           push    ebx  

00472F74   .  FF15 2C104000 call    dword ptr [<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVarList  

00472F7A   .  8B55 B8       mov     edx, dword ptr [ebp-48]  

00472F7D   .  83C4 0C       add     esp, 0C  

00472F80   .  89B5 58FFFFFF mov     dword ptr [ebp-A8], esi  

00472F86   .  899D 50FFFFFF mov     dword ptr [ebp-B0], ebx  

00472F8C   .  52           push    edx  

00472F8D   .  FF15 1C104000 call    dword ptr [<&MSVBVM60.__vbaLenBs>;  MSVBVM60.__vbaLenBstr  

00472F93   .  8985 48FFFFFF mov     dword ptr [ebp-B8], eax  

00472F99   .  8D85 50FFFFFF lea     eax, dword ptr [ebp-B0]  

00472F9F   .  8D8D 40FFFFFF lea     ecx, dword ptr [ebp-C0]  

00472FA5   .  50           push    eax  

00472FA6   .  8D95 30FFFFFF lea     edx, dword ptr [ebp-D0]  

00472FAC   .  51           push    ecx  

00472FAD   .  8D85 C8FEFFFF lea     eax, dword ptr [ebp-138]  

00472FB3   .  52           push    edx  
```



```
00472FB4   .  8D8D D8FEFFFF  lea     ecx, dword ptr [ebp-128]  
00472FBA   .  50            push    eax  
00472FBB   .  8D55 BC       lea     edx, dword ptr [ebp-44]  
00472FBE   .  51            push    ecx  
00472FBF   .  52            push    edx  
00472FC0   .  C785 40FFFFFF>mov     dword ptr [ebp-C0], 3  
00472FCA   .  899D 38FFFFFF mov     dword ptr [ebp-C8], ebx  
00472FD0   .  899D 30FFFFFF mov     dword ptr [ebp-D0], ebx  
00472FD6   .  FF15 80104000 call    dword ptr [<&MSVBVM60.__vbaVarFo>;  MSVBVM60.__vbaVarForInit  
00472FDC   .  8B35 D4104000 mov     esi, dword ptr [<&MSVBVM60.__vba>;  MSVBVM60.__vbaStrCmp  
00472FE2   >  85C0          test    eax, eax                    ;  循环开始  
00472FE4   .  0F84 90010000 je      0047317A  
00472FEA   .  8D4D 80       lea     ecx, dword ptr [ebp-80]  
00472FED   .  8D55 BC       lea     edx, dword ptr [ebp-44]  
00472FF0   .  8D45 B8       lea     eax, dword ptr [ebp-48]  
00472FF3   .  51            push    ecx  
00472FF4   .  52            push    edx  
00472FF5   .  C745 88 01000>mov     dword ptr [ebp-78], 1  
00472FFC   .  895D 80       mov     dword ptr [ebp-80], ebx  
00472FFF   .  8985 58FFFFFF mov     dword ptr [ebp-A8], eax  
00473005   .  C785 50FFFFFF>mov     dword ptr [ebp-B0], 4008  
0047300F   .  FF15 BC114000 call    dword ptr [<&MSVBVM60.__vbaI4Var>;  MSVBVM60.__vbaI4Var  
00473015   .  50            push    eax  
00473016   .  8D85 50FFFFFF lea     eax, dword ptr [ebp-B0]  
0047301C   .  8D8D 70FFFFFF lea     ecx, dword ptr [ebp-90]  
00473022   .  50            push    eax  
00473023   .  51            push    ecx  
00473024   .  FF15 B8104000 call    dword ptr [<&MSVBVM60.#632>]     ;  MSVBVM60.rtcMidCharVar  
0047302A   .  8D95 70FFFFFF lea     edx, dword ptr [ebp-90]  
00473030   .  52            push    edx  
00473031   .  FF15 20104000 call    dword ptr [<&MSVBVM60.__vbaStrVa>;  MSVBVM60.__vbaStrVarMove  
00473037   .  8BD0          mov     edx, eax  
00473039   .  8D4D 98       lea     ecx, dword ptr [ebp-68]  
0047303C   .  FFD6          call    edi  
0047303E   .  8D85 70FFFFFF lea     eax, dword ptr [ebp-90]  
00473044   .  8D4D 80       lea     ecx, dword ptr [ebp-80]  
00473047   .  50            push    eax  
00473048   .  51            push    ecx  
00473049   .  53            push    ebx  
0047304A   .  FF15 2C104000 call    dword ptr [<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVarList  
00473050   .  8B55 98       mov     edx, dword ptr [ebp-68]  
00473053   .  83C4 0C       add     esp, 0C  
00473056   .  52            push    edx  
00473057   .  68 BC394200   push    004239BC                    ;  1  
0047305C   .  FFD6          call    esi  
0047305E   .  8BD8          mov     ebx, eax  
00473060   .  8B45 98       mov     eax, dword ptr [ebp-68]  
00473063   .  F7DB          neg     ebx  
00473065   .  1BDB          sbb     ebx, ebx  
00473067   .  50            push    eax  
00473068   .  68 B4394200   push    004239B4                    ;  0  
0047306D   .  F7DB          neg     ebx  
0047306F   .  FFD6          call    esi  
00473071   .  F7D8          neg     eax  
00473073   .  8B4D 98       mov     ecx, dword ptr [ebp-68]  
00473076   .  1BC0          sbb     eax, eax  
00473078   .  51            push    ecx  
00473079   .  F7D8          neg     eax  
0047307B   .  23D8          and     ebx, eax  
0047307D   .  68 C4394200   push    004239C4                    ;  2  
00473082   .  F7DB          neg     ebx  
00473084   .  1BDB          sbb     ebx, ebx  
00473086   .  F7DB          neg     ebx  
00473088   .  FFD6          call    esi  
0047308A   .  F7D8          neg     eax  
0047308C   .  8B55 98       mov     edx, dword ptr [ebp-68]  
0047308F   .  1BC0          sbb     eax, eax  
00473091   .  52            push    edx  
00473092   .  F7D8          neg     eax  
00473094   .  23D8          and     ebx, eax  
00473096   .  68 CC394200   push    004239CC                    ;  3  
0047309B   .  F7DB          neg     ebx  
0047309D   .  1BDB          sbb     ebx, ebx  
0047309F   .  F7DB          neg     ebx  
004730A1   .  FFD6          call    esi  
004730A3   .  F7D8          neg     eax  
004730A5   .  1BC0          sbb     eax, eax  
004730A7   .  F7D8          neg     eax  
004730A9   .  23D8          and     ebx, eax  
004730AB   .  8B45 98       mov     eax, dword ptr [ebp-68]  
004730AE   .  F7DB          neg     ebx  
004730B0   .  1BDB          sbb     ebx, ebx  
004730B2   .  50            push    eax  
004730B3   .  68 D4394200   push    004239D4                    ;  4  
004730B8   .  F7DB          neg     ebx  
004730BA   .  FFD6          call    esi  
004730BC   .  F7D8          neg     eax  
004730BE   .  1BC0          sbb     eax, eax  
004730C0   .  F7D8          neg     eax  
004730C2   .  23D8          and     ebx, eax  
004730C4   .  F7DB          neg     ebx  
004730C6   .  1BDB          sbb     ebx, ebx  
004730C8   .  F7DB          neg     ebx  
004730CA   .  8B4D 98       mov     ecx, dword ptr [ebp-68]  
004730CD   .  51            push    ecx  
004730CE   .  68 DC394200   push    004239DC                    ;  5  
004730D3   .  FFD6          call    esi  
004730D5   .  F7D8          neg     eax  
004730D7   .  8B55 98       mov     edx, dword ptr [ebp-68]  
004730DA   .  1BC0          sbb     eax, eax  
004730DC   .  52            push    edx  
004730DD   .  F7D8          neg     eax  
004730DF   .  23D8          and     ebx, eax  
004730E1   .  68 E4394200   push    004239E4                    ;  6  
004730E6   .  F7DB          neg     ebx  
004730E8   .  1BDB          sbb     ebx, ebx  
004730EA   .  F7DB          neg     ebx  
004730EC   .  FFD6          call    esi  
004730EE   .  F7D8          neg     eax  
004730F0   .  1BC0          sbb     eax, eax  
004730F2   .  F7D8          neg     eax  
004730F4   .  23D8          and     ebx, eax  
004730F6   .  8B45 98       mov     eax, dword ptr [ebp-68]  
004730F9   .  F7DB          neg     ebx  
004730FB   .  1BDB          sbb     ebx, ebx  
004730FD   .  50            push    eax  
004730FE   .  68 EC394200   push    004239EC                    ;  7  
00473103   .  F7DB          neg     ebx  
00473105   .  FFD6          call    esi  
00473107   .  F7D8          neg     eax  
00473109   .  8B4D 98       mov     ecx, dword ptr [ebp-68]  
0047310C   .  1BC0          sbb     eax, eax  
0047310E   .  51            push    ecx  
0047310F   .  F7D8          neg     eax  
00473111   .  23D8          and     ebx, eax  
00473113   .  68 F4394200   push    004239F4                    ;  8  
00473118   .  F7DB          neg     ebx  
0047311A   .  1BDB          sbb     ebx, ebx  
0047311C   .  F7DB          neg     ebx  
0047311E   .  FFD6          call    esi  
00473120   .  F7D8          neg     eax  
00473122   .  8B55 98       mov     edx, dword ptr [ebp-68]  
00473125   .  1BC0          sbb     eax, eax  
00473127   .  52            push    edx  
00473128   .  F7D8          neg     eax  
0047312A   .  23D8          and     ebx, eax  
0047312C   .  68 FC394200   push    004239FC                    ;  9  
00473131   .  F7DB          neg     ebx  
00473133   .  1BDB          sbb     ebx, ebx  
00473135   .  F7DB          neg     ebx  
00473137   .  FFD6          call    esi  
00473139   .  F7D8          neg     eax  
0047313B   .  1BC0          sbb     eax, eax  
0047313D   .  F7D8          neg     eax  
0047313F   .  85D8          test    eax, ebx  
00473141   .  75 15         jnz     short 00473158  
00473143   .  8B45 98       mov     eax, dword ptr [ebp-68]  
00473146   .  8B4D 94       mov     ecx, dword ptr [ebp-6C]  
00473149   .  50            push    eax  
```



```asm
0047314A   .  51             push    ecx  
0047314B   .  FF15 50104000  call    dword ptr [<&MSVBVM60.__vbaStrCa>;  MSVBVM60.__vbaStrCat  
00473151   .  8BD0          mov     edx, eax  
00473153   .  8D4D 94       lea     ecx, dword ptr [ebp-6C]  
00473156   .  FFD7          call    edi  
00473158   >  8D95 C8FEFFFF lea     edx, dword ptr [ebp-138]  
0047315E   .  8D85 D8FEFFFF lea     eax, dword ptr [ebp-128]  
00473164   .  52            push    edx  
00473165   .  8D4D BC       lea     ecx, dword ptr [ebp-44]  
00473168   .  50            push    eax  
00473169   .  51            push    ecx  
0047316A   .  FF15 1C124000  call    dword ptr [<&MSVBVM60.__vbaVarFo>;  MSVBVM60.__vbaVarForNext  
00473170   .  BB 02000000   mov     ebx, 2  
00473175   .^ E9 68FEFFFF   jmp     00472FE2                 ;  循环回去继续，这个循环是倒转字符串  
0047317A   >  8B55 94       mov     edx, dword ptr [ebp-6C]        ;  经过上面的循环居然少了个 1。就是去掉最前面的一个字符  
0047317D   .  8D4D B8       lea     ecx, dword ptr [ebp-48]        ;  上面这个应该就是真码了  
00473180   .  FF15 98114000  call    dword ptr [<&MSVBVM60.__vbaStrCo>;  MSVBVM60.__vbaStrCopy  
00473186   .  9B            wait  
00473187   .  68 37324700   push    00473237  
0047318C   .  EB 36         jmp     short 004731C4  
```

上面的运算返回到下面这个 `call`

```asm
00472519   .  50            push    eax  
0047251A   .  51            push    ecx  
0047251B   .  E8 00070000   call    00472C20                 ;  这个是算法 call  
00472520   .  8BD0          mov     edx, eax                 ;  这里是真码了。内存注册机  
00472522   .  8D4D B0       lea     ecx, dword ptr [ebp-50]  
00472525   .  FFD7          call    edi  
00472527   .  50            push    eax  
00472528   .  FF15 D4104000  call    dword ptr [<&MSVBVM60.__vbaStrCm>;  MSVBVM60.__vbaStrCmp  
0047252E   .  8BF8          mov     edi, eax                 ;  真假码比较函数  
00472530   .  8D55 B0       lea     edx, dword ptr [ebp-50]  
00472533   .  F7DF          neg     edi  
00472535   .  8D45 BC       lea     eax, dword ptr [ebp-44]  
00472538   .  52            push    edx  
00472539   .  1BFF          sbb     edi, edi  
0047253B   .  8D4D B4       lea     ecx, dword ptr [ebp-4C]  
0047253E   .  50            push    eax  
0047253F   .  47            inc     edi  
00472540   .  51            push    ecx  
00472541   .  6A 03        push    3  
00472543   .  F7DF          neg     edi  
00472545   .  FF15 9C114000  call    dword ptr [<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStrList  
0047254B   .  8D55 A0       lea     edx, dword ptr [ebp-60]  
0047254E   .  8D45 A4       lea     eax, dword ptr [ebp-5C]  
00472551   .  52            push    edx  
00472552   .  8D4D A8       lea     ecx, dword ptr [ebp-58]  
00472555   .  50            push    eax  
00472556   .  8D55 AC       lea     edx, dword ptr [ebp-54]  
00472559   .  51            push    ecx  
0047255A   .  52            push    edx  
0047255B   .  6A 04        push    4  
0047255D   .  FF15 38104000  call    dword ptr [<&MSVBVM60.__vbaFreeO>;  MSVBVM60.__vbaFreeObjList  
00472563   .  83C4 24       add     esp, 24  
00472566   .  66:3BFB       cmp     di, bx  
00472569      0F84 D5000000  je     00472644                 ;  关键跳转  
0047256F      8D85 1CFFFFFF  lea     eax, dword ptr [ebp-E4]  
00472575      8D4D C0       lea     ecx, dword ptr [ebp-40]  
00472578      50            push    eax  
00472579   .  51            push    ecx  
```

算法太简单了，就是先得到输入的 6 位数的用户名。逐个取它的 `ascii` 码值的 10 进制累乘。累乘的结果以 10 进制字符串形式表示，去掉第一位后，整个字符串倒过来就是真注册码了。

## 版本 2

```e
.程序集 窗口程序集 1

.子程序 _ 按钮 1_ 被单击

.局部变量 name, 文本型
.局部变量 len, 整数型
.局部变量 i, 整数型
.局部变量 eax, 整数型
.局部变量 sum, 长整数型
.局部变量 code, 文本型
.局部变量 code1, 文本型
.局部变量 jiqima, 文本型

name ＝ 编辑框 1.内容
jiqima ＝ 编辑框 3.内容
len ＝ 取文本长度 (name)
sum ＝ 1

.如果 (len ＝ 6)
    .计次循环首 (len, i)
        eax ＝ 取代码 (name, i)
        sum ＝ sum × eax
    .计次循环尾 ()
.否则
    信息框 (“只要六位数的字母”, 0, )
.如果结束

code1 ＝ 到文本 (sum ＋ 到数值 (jiqima))
code1 ＝ 取文本右边 (code1, 取文本长度 (code1) － 1)
code ＝ 倒转字符串 (code1)
编辑框 2.内容 ＝ code

.子程序 倒转字符串, 文本型
.参数 string, 文本型
.局部变量 k, 整数型
.局部变量 RChar, 文本型
.局部变量 LChar, 文本型
.局部变量 WChar, 文本型

.判断循环首 (k ＜ 取文本长度 (string))
    LChar ＝ 取文本左边 (string, 取文本长度 (string) － k)
    RChar ＝ 取文本右边 (LChar, 1)
    k ＝ k ＋ 1
    WChar ＝ WChar ＋ RChar
.判断循环尾 ()

返回 (WChar)
```

`4dd466d0e1e0cc123afbe504da2462ca`
