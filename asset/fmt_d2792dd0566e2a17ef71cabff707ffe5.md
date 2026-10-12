
# 交通法规考试系统 2005

> 原文：[`www.pediy.com/kssd/pediy06/pediy6631.htm`](https://www.pediy.com/kssd/pediy06/pediy6631.htm)

交通法规考试系统 2005 破解简析

工具：`od` , `VB` 反编译器

声明：高手略过…… 破 VB 一个字烦.

交通法规考试系统 2005 分北京版本,与全国版,破了任何一个,都开始用任何一个~对作者冲击很大,简单的分析一下;有错误的地方,感谢指出.

软件介绍:是浮游工作室的主打产品，对用户通过驾驶执照的理论考试有巨大的帮助，根据两年来大量用户的反馈，使用本软件进行备考，可以明显缩短备考时间，消除机考的紧张感，并让您轻松通过理论考试。

注册事件入口:

```
0045E250    > \55                push ebp
0045E251    .   8BEC             mov ebp,esp
0045E253    .   83EC 14          sub esp,14
0045E256    .   68 762A4000      push <jmp.&MSVBVM60.__vbaExceptHandler>  ;  SE handler installation
0045E25B    .   64:A1 00000000   mov eax,dword ptr fs:[0]
0045E261    .   50               push eax
0045E262    .   64:8925 00000000 mov dword ptr fs:[0],esp
0045E269    .   81EC E8000000    sub esp,0E8
```

……找到关键 CALL:

```
0045E4AD    .   8D4D C0          lea ecx,dword ptr ss:[ebp-40]
0045E4B0    .   51               push ecx               ；指向假注册吗
0045E4B1    .   8D55 C4          lea edx,dword ptr ss:[ebp-3C]
0045E4B4    .   52               push edx               ；指向用户名
0045E4B5    .   E8 160AFDFF      call TranExam.0042EED
```

……函数入口：

```
0042EED0    $   55               push ebp
0042EED1    .   8BEC             mov ebp,esp
0042EED3    .   83EC 08          sub esp,8
```

……

q        i     w      e         i         x        u       e    长度为：8
71       69    77      65       69       78        75      65     ；16 进制
113      105   119     101      105      120       117     101    ；10 进制
10.6301 10.247 10.9087 10.04988 10.24695 10.95445  10.81665 10.049876 ；平方根

第一个循环，8 次：

```
0042EF6B    .   BE 01000000      mov esi,1                           ;  esi 计数器=1
0042EF70    .   8975 E8          mov dword ptr ss:[ebp-18],esi       ;  [ebp-18] =esi=1
0042EF73    >   3BF0             cmp esi,eax                         ;  eax=8，esi=1
0042EF75    .   0F8F C5020000    jg TranExam.0042F240                ；当 esi 大与 eax，跳出循环
0042EF7B    .   8B45 08          mov eax,dword ptr ss:[ebp+8]        ;  (*[ebp-8])=qiweixue(0x1747e4)
0042EF7E    .   8D8D 78FFFFFF    lea ecx,dword ptr ss:[ebp-88]       ;  [ebp-88]=0
0042EF84    .   8985 40FFFFFF    mov dword ptr ss:[ebp-C0],eax       ;  (*[ebp-c0])=qiweixue(0x1747e4)
0042EF8A    .   51               push ecx
0042EF8B    .   8D95 38FFFFFF    lea edx,dword ptr ss:[ebp-C8]       ;  [ebp-c8]=0
0042EF91    .   56               push esi                            ;  esi=1 记数为 1
0042EF92    .   8D85 68FFFFFF    lea eax,dword ptr ss:[ebp-98]       ;  [ebp-98]=0
0042EF98    .   52               push edx
0042EF99    .   50               push eax
0042EF9A    .   C745 80 01000000 mov dword ptr ss:[ebp-80],1        ;  [ebp-80]=1
0042EFA1    .   C785 78FFFFFF 020 mov dword ptr ss:[ebp-88],2        ;  [ebp-88]=2
0042EFAB    .   C785 38FFFFFF 084 mov dword ptr ss:[ebp-C8],4008
0042EFB5    .   FF15 BC104000    call dword ptr ds:[<&MSVBVM60.#632>] ;  MSVBVM60.rtcMidCharVar
0042EFBB    .   8D8D 68FFFFFF    lea ecx,dword ptr ss:[ebp-98]
0042EFC1    .   8D55 A8          lea edx,dword ptr ss:[ebp-58]
0042EFC4    .   51               push ecx
0042EFC5    .   52               push edx
0042EFC6    .   FFD3             call ebx
0042EFC8    .   50               push eax
0042EFC9    .   FF15 40104000    call dword ptr ds:[<&MSVBVM60.#516>] ;  MSVBVM60.rtcAnsiValueBstr
0042EFCF    .   8BC8             mov ecx,eax                         ;  返回 q 的 ASC=71
0042EFD1    .   FFD7             call edi                             ;  取绝对值
0042EFD3    .   0FBFC0           movsx eax,ax
0042EFD6    .   8985 C8FEFFFF    mov dword ptr ss:[ebp-138],eax      ;  [ebp-138]=71
0042EFDC    .   DB85 C8FEFFFF    fild dword ptr ss:[ebp-138]         ;  取实数 q 的 113
0042EFE2    .   DD9D C0FEFFFF    fstp qword ptr ss:[ebp-140]         ;  [ebp-140]为 0
0042EFE8    .   8B8D C4FEFFFF    mov ecx,dword ptr ss:[ebp-13C]      ;  [ebp-13c]=405c4000
0042EFEE    .   8B95 C0FEFFFF    mov edx,dword ptr ss:[ebp-140]
0042EFF4    .   51               push ecx
0042EFF5    .   52               push edx
0042EFF6    .   FF15 D4114000    call dword ptr ds:[<&MSVBVM60.#614>] ;  MSVBVM60.rtcSqr
0042EFFC    .   8B45 08          mov eax,dword ptr ss:[ebp+8]        ;  qiweixue
0042EFFF    .   8D4D 98          lea ecx,dword ptr ss:[ebp-68]       ;  [ebp-68]=0
0042F002    .   DD9D 18FFFFFF    fstp qword ptr ss:[ebp-E8]         ;  [ebp-e8]=78d2d036
0042F008    .   8985 60FFFFFF    mov dword ptr ss:[ebp-A0],eax      ;  (*[ebp-a0])=qiweixue
0042F00E    .   51               push ecx
0042F00F    .   8D95 58FFFFFF    lea edx,dword ptr ss:[ebp-A8]       ;  [ebp-a8]= 0
0042F015    .   56               push esi
0042F016    .   8D45 88          lea eax,dword ptr ss:[ebp-78]       ;  [ebp-78]=0
0042F019    .   52               push edx
0042F01A    .   50               push eax
0042F01B    .   C745 A0 01000000 mov dword ptr ss:[ebp-60],1
0042F022    .   C745 98 02000000 mov dword ptr ss:[ebp-68],2
0042F029    .   C785 58FFFFFF 084 mov dword ptr ss:[ebp-A8],4008
0042F033    .   FF15 BC104000    call dword ptr ds:[<&MSVBVM60.#632>] ;  MSVBVM60.rtcMidCharVar
0042F039    .   8D4D 88          lea ecx,dword ptr ss:[ebp-78]
0042F03C    .   8D55 AC          lea edx,dword ptr ss:[ebp-54]
0042F03F    .   51               push ecx
0042F040    .   52               push edx
0042F041    .   FFD3             call ebx
0042F043    .   50               push eax
0042F044    .   FF15 40104000    call dword ptr ds:[<&MSVBVM60.#516>] ;  MSVBVM60.rtcAnsiValueBstr
0042F04A    .   8BC8             mov ecx,eax
0042F04C    .   FFD7             call edi
0042F04E    .   0FBFC0           movsx eax,ax
0042F051    .   0FAFC6           imul eax,esi                        ;  用户名*计数数值
0042F054    .   0F80 54060000    jo TranExam.0042F6AE                ;  溢出就 error
0042F05A    .   0FAFC6           imul eax,esi                        ；eax=eax*esi
0042F05D    .   8B4D E4          mov ecx,dword ptr ss:[ebp-1C]
0042F060    .   0F80 48060000    jo TranExam.0042F6AE                ；溢出就跳死
0042F066    .   03C1             add eax,ecx                         ；eax=eax+ecx
0042F068    .   0F80 40060000    jo TranExam.0042F6AE                ；溢出就跳死
0042F06E    .   8985 BCFEFFFF    mov dword ptr ss:[ebp-144],eax
0042F074    .   DB85 BCFEFFFF    fild dword ptr ss:[ebp-144]         ；浮点计算
0042F07A    .   DD9D B4FEFFFF    fstp qword ptr ss:[ebp-14C]
0042F080    .   DB45 E8          fild dword ptr ss:[ebp-18]
0042F083    .   DD9D ACFEFFFF    fstp qword ptr ss:[ebp-154]
0042F089    .   DD85 ACFEFFFF    fld qword ptr ss:[ebp-154]
0042F08F    .   DC8D 18FFFFFF    fmul qword ptr ss:[ebp-E8]
0042F095    .   DC05 50124000    fadd qword ptr ds:[401250]
0042F09B    .   DC8D B4FEFFFF    fmul qword ptr ss:[ebp-14C]
0042F0A1    .   DFE0             fstsw ax
0042F0A3    .   A8 0D            test al,0D                           ；al 与 od 比较
0042F0A5    .   0F85 FE050000    jnz TranExam.0042F6A9               ;  跳就 overflow 的
0042F0AB    .   FF15 E8114000    call dword ptr ds:[<&MSVBVM60.__vbaFpI4>>;  MSVBVM60.__vbaFpI4
0042F0B1    .   99               cdq                                 ;  eax=522
0042F0B2    .   B9 10270000      mov ecx,2710                        ;  ecx=2710
0042F0B7    .   F7F9             idiv ecx                            ；除以 2710
```
