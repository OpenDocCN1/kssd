
```
  0044639B   .  BB 659A4501   mov     ebx, 1459A65  

  004463A0   >  8BCB          mov     ecx, ebx                     ;  ecx=038E908C XOR 12214336 = 11AFD3BA  

  004463A2   .  81F1 36432112 xor     ecx, 12214336                ;  ecx 入机器码的 16 制数  

  004463A8   .  8BC1          mov     eax, ecx  

  004463AA   .  99            cdq  

  004463AB   .  33C2          xor     eax, edx  

  004463AD   .  2BC2          sub     eax, edx  

  004463AF   .  8BD8          mov     ebx, eax  

  004463B1   .  8BCB          mov     ecx, ebx  

  004463B3   .  81C9 61341221 or      ecx, 21123461                ;  11AFD3BA OR 21123461 =31BFF7FB  

  004463B9   .  8BC1          mov     eax, ecx  

  004463BB   .  99            cdq  

  004463BC   .  33C2          xor     eax, edx  

  004463BE   .  2BC2          sub     eax, edx  

  004463C0   .  8BD8          mov     ebx, eax                     ;  11AFD3BA  

  004463C2   .  895D B8       mov     dword ptr [ebp-48], ebx  

  004463C5   .  DB45 B8       fild    dword ptr [ebp-48]           ;  浮点加载整数  

  004463C8   .  DD5D C4       fstp    qword ptr [ebp-3C]           ;  fstp 弹栈，将 st0 弹出,ST7 存放  

  004463CB   .  DD45 C4       fld     qword ptr [ebp-3C]           ;  浮点加载  

  004463CE   .  D80D 14654400 fmul    dword ptr [446514]          ;  浮点乘法  

  004463D4   .  DD5D C4       fstp    qword ptr [ebp-3C]  

  004463D7   .  EB 09         jmp     short 004463E2  

  004463D9   >  DD45 C4       fld     qword ptr [ebp-3C]  

  004463DC   .  DC65 BC       fsub    qword ptr [ebp-44]  

  004463DF   .  DD5D C4       fstp    qword ptr [ebp-3C]           ;  相减  

  004463E2   >  DD45 C4       fld     qword ptr [ebp-3C]           ;  29213255505  

  004463E5   .  DC5D BC       fcomp   qword ptr [ebp-44]  

  004463E8   .  DFE0          fstsw   ax                           ;  保存状态字的值到 AX  

  004463EA   .  9E            sahfs          ;  恢复标志  

  004463EB   .^ 77 EC        ja      short 004463D9           ;  大于或等于则继续相减，直接差小于 2147483647  

  004463ED   .  DD45 C4       fld     qword ptr [ebp-3C]  

  004463F0   .  E8 39D22100   call    <jmp.&CC3250MT.__ftol>  

  004463F5   .  8BD8          mov     ebx, eax                     ;  eax=4D3EE75E(十进制：1295968094)  

  004463F7   .  8BCB          mov     ecx, ebx  

  004463F9     81F1 10567081 xor     ecx, 81705610                ;  ecx=4D3EE75E xor  81705610  

  004463FF   .  8BC1          mov     eax, ecx  

  00446401   .  99            cdq  

  00446402   .  33C2          xor     eax, edx  

  00446404   .  2BC2          sub     eax, edx  

  00446406   .  8BD8          mov     ebx, eax  

  00446408   .  895D B8       mov     dword ptr [ebp-48], ebx      ;  33B14EB2 存入 EBP-48,即存于 0012EB74  

  0044640B   .  DB45 B8       fild    dword ptr [ebp-48]           ;  浮点加载  

  0044640E   .  DD5D C4       fstp    qword ptr [ebp-3C]           ;  以浮点数弹出  

  00446411   .  DB2D 18654400 fld     tbyte ptr [446518]           ;  加载实数 ds:[00446518]=0.0454545454545454528  

  00446417   .  DC4D C4       fmul    qword ptr [ebp-3C]           ;   

  0044641A   .  DD5D C4       fstp    qword ptr [ebp-3C]  

  0044641D   .  EB 09         jmp     short 00446428  

  0044641F   >  DD45 C4       fld     qword ptr [ebp-3C]  

  00446422   .  DC65 BC       fsub    qword ptr [ebp-44]  

  00446425   .  DD5D C4       fstp    qword ptr [ebp-3C]  

  00446428   >  DD45 C4       fld     qword ptr [ebp-3C]  

  0044642B   .  DC5D BC       fcomp   qword ptr [ebp-44]  

  0044642E   .  DFE0          fstsw   ax  

  00446430   .  9E            sahfs  

  00446431   .^ 77 EC        ja      short 0044641F           ;  大于或等于则继续相减，直接差小于 2147483647        

  00446433   .  DD45 C4       fld     qword ptr [ebp-3C]  

  00446436   .  E8 F3D12100   call    <jmp.&CC3250MT.__ftol>  

  0044643B   .  8BD8          mov     ebx, eax  

  0044643D   .  8BCB          mov     ecx, ebx  

  0044643F   .  81C9 22221111 or      ecx, 11112222  

  00446445   .  8BC1          mov     eax, ecx  

  00446447   .  99            cdq  

  00446448   .  33C2          xor     eax, edx                 ;   

  0044644A   .  2BC2          sub     eax, edx  

  0044644C   .  8BD8          mov     ebx, eax                 ;  ebx=1359A3B3,堆栈 ss:[0012EB74]=33B14EB2  

  0044644E   .  895D B8       mov     dword ptr [ebp-48], ebx      ;  堆栈 ss:[0012EB74]=1359A3B3 (十进制 324641715.)  

  00446451   .  DB45 B8       fild    dword ptr [ebp-48]  

  00446454   .  DD5D C4       fstp    qword ptr [ebp-3C]  

  00446457   .  DD45 C4       fld     qword ptr [ebp-3C]  

  0044645A   .  D80D 24654400 fmul    dword ptr [446524]          ;  st=324641715.00000000000，ds:[00446524]=30.00000，作乘法  

  00446460   .  DD5D C4       fstp    qword ptr [ebp-3C]  

  00446463   .  EB 09         jmp     short 0044646E  

  00446465   >  DD45 C4       fld     qword ptr [ebp-3C]  

  00446468   .  DC65 BC       fsub    qword ptr [ebp-44]  

  0044646B   .  DD5D C4       fstp    qword ptr [ebp-3C]  

  0044646E   >  DD45 C4       fld     qword ptr [ebp-3C]  

  00446471   .  DC5D BC       fcomp   qword ptr [ebp-44]  

  00446474   .  DFE0          fstsw   ax  

  00446476   .  9E            sahfs  

  00446477   .^ 77 EC        ja      short 00446465  

  00446479   .  DD45 C4       fld     qword ptr [ebp-3C]           ;  注意这个结果，里面有解锁真码  

  0044647C   .  E8 ADD12100   call    <jmp.&CC3250MT.__ftol>  

  00446481   .  8BD8          mov     ebx, eax                     ;  含真码的  

  00446483   .  B9 E8030000   mov     ecx, 3E8                     ;  3EB = 十进制 1000  

  00446488   .  8BC3          mov     eax, ebx  

  0044648A   .  99            cdq                            ;  CDQ 双字扩展. (把 EAX 中的字的符号扩展到 EDX 中去)  

  0044648B   .  F7F9          idiv    ecx                      ;  整数 3E8，即十进制除以 1000，相当于左移 3 位，eax 处为真码  

  0044648D   .  66:C745 DC 2C>mov     word ptr [ebp-24], 2C       ;  2C 存入 0012EB98  

  00446493   .  8BD8          mov     ebx, eax                     ;  此处 EBX 存放真正的注册码  
```

下面，我用我机器上实际运算过程，来分析下如何由机器码，算出解锁码的。

我的电脑机器是`59674764`，由注册程序提供。

首先，`038E908C XOR 12214336 = 11AFD3BA`

`11AFD3BA`再与`21123461`（固定值）相 OR，得到`31BFF7FB`（十进制：`834664443`），存于`0012EB74`

其次，`834664443 * ds:[00446514]=35.00000 = 29213255505`

计算出的值，与`2147483647.000000`比较大小，大于或等于则继续计算，过程如下：

`29213255505 - 2147483647 =27065771858`

`27065771858 - 2147483647 = 24918288211`

`24918288211 - 2147483647 = 22770804564`

`22770804564 －2147483647 = 20623320917`

……

`3443451741 －2147483647 =  1295968094`

结果`1295968094`小于`2147483647`，则程序不再跳转，执行一步运算。

`1295968094`（十六进制`4D3EE75E`）`XOR 81705610`（固定值）= `CC4EB14E`

再由`CC4EB14E XOR FFFFFFFF = 33B14EB1`

`33B14EB1 - FFFFFFFF = 33B14EB2`（十进制：`867258034`）

将结果`33B14EB2`存于`[EBP-48]=0012EB74`处，

取实数`ds:[00446518]=0.0454545454545454528`

将`867258034.0000000`（十六进制）`* 0.0454545454545454528 = 39420819.727272729600`(放入`st`)，

将`st=39420819.727272724480`与堆栈`ss:[0012EB78]=2147483647.000000`比较

没有实现，继续执行下一行代码

然后，`02598393`（十进制`39420819`）`OR 11112222 = 1359A3B3`

将`1359A3B3`（十进制`324641715`）放入`st=324641715.00000000000`，

与`ds:[00446524]=30.00000`作乘法，结果`st=9739251449.9999989760`

`st=9739251449.9999989760`与堆栈`ss:[0012EB78]=2147483647.000000`比较大小

大于或等于则作相减运算，

`9739251449.9999989760 －2147483647.000000 ＝7591767802.999998976`

`7591767802.999998976 － 2147483647.000000 ＝5444284155.999998976`

`5444284155.999998976 － 2147483647.000000 ＝3296800508.999998976`

`3296800508.999998976 － 2147483647.000000 ＝ 1149316861.999998976`

`44812EFE`（十进制`1149316862`）左移 3 位，得真正的解锁码为`1149316`（十进制）。
