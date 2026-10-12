
# Skip Manager 1.79

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-465.htm`](https://www.pediy.com/kssd/pediy08/pediy8-465.htm)

## 破文信息

- 【破文标题】Skip Manager 1.79 注册算法
- 【破文作者】XXNB
- 【作者邮箱】支持 PYG
- 【作者主页】`binbinbin7456.ys168.com`
- 【破解工具】`OD`
- 【破解平台】`XPsp2`
- 【软件名称】Skip Manager 1.79
- 【软件大小】`17834KB`
- 【原版下载】`http://www.newhua.com/soft/53764.htm`
- 【保护方式】码
- 【软件简介】一款能让你在系统中设置你所要「忽略」项目的软件。经由设置为忽略的项目後，便会自动忽略此项目的存取动作，加快系统的运行速度
- 【破解声明】菜鸟向高手学习！只为学习！

## 破解过程

### 1. 定位字符串

老罗的字符串查找 `Unicode` “registered” 可以轻松定位。

```assembly
004FB250   > \55            push    ebp
004FB251   .  8BEC          mov     ebp, esp
004FB253   .  83EC 0C       sub     esp, 0C
004FB256   .  68 F62E4000   push    <jmp.&MSVBVM60.__vbaExceptHandle>;  SE 处理程序安装
004FB25B   .  64:A1 0000000>mov     eax, dword ptr fs:[0]
004FB261   .  50            push    eax
004FB262   .  64:8925 00000>mov     dword ptr fs:[0], esp
004FB269   .  81EC B8000000 sub     esp, 0B8
004FB26F   .  53            push    ebx
004FB270   .  56            push    esi
004FB271   .  57            push    edi
004FB272   .  8965 F4       mov     dword ptr [ebp-C], esp
004FB275   .  C745 F8 88284>mov     dword ptr [ebp-8], 00402888
004FB27C   .  8B75 08       mov     esi, dword ptr [ebp+8]
004FB27F   .  8BC6          mov     eax, esi
004FB281   .  83E0 01       and     eax, 1
004FB284   .  8945 FC       mov     dword ptr [ebp-4], eax
004FB287   .  83E6 FE       and     esi, FFFFFFFE
004FB28A   .  8B0E          mov     ecx, dword ptr [esi]
004FB28C   .  56            push    esi
004FB28D   .  8975 08       mov     dword ptr [ebp+8], esi
004FB290   .  FF51 04       call    dword ptr [ecx+4]
004FB293   .  8B16          mov     edx, dword ptr [esi]
004FB295   .  33FF          xor     edi, edi
004FB297   .  56            push    esi
004FB298   .  897D E8       mov     dword ptr [ebp-18], edi
004FB29B   .  897D E4       mov     dword ptr [ebp-1C], edi
004FB29E   .  897D E0       mov     dword ptr [ebp-20], edi
004FB2A1   .  897D DC       mov     dword ptr [ebp-24], edi
004FB2A4   .  897D CC       mov     dword ptr [ebp-34], edi
004FB2A7   .  897D BC       mov     dword ptr [ebp-44], edi
004FB2AA   .  897D AC       mov     dword ptr [ebp-54], edi
004FB2AD   .  897D 9C       mov     dword ptr [ebp-64], edi
004FB2B0   .  897D 8C       mov     dword ptr [ebp-74], edi
004FB2B3   .  89BD 7CFFFFFF mov     dword ptr [ebp-84], edi
004FB2B9   .  FF92 08030000 call    dword ptr [edx+308]
004FB2BF   .  8B1D B0104000 mov     ebx, dword ptr [<&MSVBVM60.__vba>;  MSVBVM60.__vbaObjSet
004FB2C5   .  50            push    eax
004FB2C6   .  8D45 E0       lea     eax, dword ptr [ebp-20]
004FB2C9   .  50            push    eax
004FB2CA   .  FFD3          call    ebx                 ;  <&MSVBVM60.__vbaObjSet>
004FB2CC   .  8B08          mov     ecx, dword ptr [eax]
004FB2CE   .  8D55 E8       lea     edx, dword ptr [ebp-18]
004FB2D1   .  52            push    edx
004FB2D2   .  50            push    eax
004FB2D3   .  8985 58FFFFFF mov     dword ptr [ebp-A8], eax
004FB2D9   .  FF91 A0000000 call    dword ptr [ecx+A0]
004FB2DF   .  DBE2          fclex
004FB2E1   .  3BC7          cmp     eax, edi
004FB2E3   .  7D 18         jge     short 004FB2FD
004FB2E5   .  8B8D 58FFFFFF mov     ecx, dword ptr [ebp-A8]
004FB2EB   .  68 A0000000   push    0A0                 ; /Arg4 = 000000A0
004FB2F0   .  68 48554200   push    00425548            ; |Arg3 = 00425548
004FB2F5   .  51            push    ecx                 ; |Arg2
004FB2F6   .  50            push    eax                 ; |Arg1
004FB2F7   .  FF15 78104000 call    dword ptr [<&MSVBVM60.__vbaHresu>; \_\_vbaHresultCheckObj
004FB2FD   >  8B55 E8       mov     edx, dword ptr [ebp-18]       ;  照例，假码出现了
004FB300   .  8D4D E4       lea     ecx, dword ptr [ebp-1C]
004FB303   .  897D E8       mov     dword ptr [ebp-18], edi
004FB306   .  FF15 68124000 call    dword ptr [<&MSVBVM60.__vbaStrMo>;  MSVBVM60.__vbaStrMove
004FB30C   .  8D55 E4       lea     edx, dword ptr [ebp-1C]       ;  移动到 eax
004FB30F   .  52            push    edx
004FB310   .  8D45 CC       lea     eax, dword ptr [ebp-34]
004FB313   .  50            push    eax
004FB314   .  E8 4780F9FF   call    00493360            ;  这个放眼都猜到是算法 call。跟进看看《《《《《－－－－
004FB319   .  8D4D CC       lea     ecx, dword ptr [ebp-34]
004FB31C   .  51            push    ecx
004FB31D   .  FF15 E0104000 call    dword ptr [<&MSVBVM60.__vbaBoolV>;  MSVBVM60.__vbaBoolVarNull
004FB323   .  8D4D E4       lea     ecx, dword ptr [ebp-1C]
004FB326   .  66:8985 50FFF>mov     word ptr [ebp-B0], ax
004FB32D   .  FF15 98124000 call    dword ptr [<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr
004FB333   .  8D4D E0       lea     ecx, dword ptr [ebp-20]
004FB336   .  FF15 94124000 call    dword ptr [<&MSVBVM60.__vbaFreeO>;  MSVBVM60.__vbaFreeObj
004FB33C   .  8D4D CC       lea     ecx, dword ptr [ebp-34]
004FB33F   .  FF15 24104000 call    dword ptr [<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVar
004FB345   .  66:39BD 50FFF>cmp     word ptr [ebp-B0], di
004FB34C   .  B9 04000280   mov     ecx, 80020004
004FB351   .  B8 0A000000   mov     eax, 0A
004FB356   .  894D A4       mov     dword ptr [ebp-5C], ecx
004FB359   .  8945 9C       mov     dword ptr [ebp-64], eax
004FB35C   .  894D B4       mov     dword ptr [ebp-4C], ecx
004FB35F   .  8945 AC       mov     dword ptr [ebp-54], eax
004FB362   .  894D C4       mov     dword ptr [ebp-3C], ecx
004FB365   .  8945 BC       mov     dword ptr [ebp-44], eax
004FB368   .  0F84 C1010000 je      004FB52F            ;  关键跳转
004FB36E   .  8D55 8C       lea     edx, dword ptr [ebp-74]
004FB371   .  8D4D CC       lea     ecx, dword ptr [ebp-34]       ;  成功信息
004FB374   .  C745 94 98F84>mov     dword ptr [ebp-6C], 0042F898  ;  thank you.  your product is now registered.
004FB37B   .  C745 8C 08000>mov     dword ptr [ebp-74], 8
004FB382   .  FF15 34124000 call    dword ptr [<&MSVBVM60.__vbaVarDu>;  MSVBVM60.__vbaVarDup
004FB388   .  8D55 9C       lea     edx, dword ptr [ebp-64]
004FB38B   .  52            push    edx
004FB38C   .  8D45 AC       lea     eax, dword ptr [ebp-54]
```

### 2. 算法跟进

```assembly
00493360   $  55            push    ebp
00493361   .  8BEC          mov     ebp, esp
00493363   .  83EC 0C       sub     esp, 0C
00493366   .  68 F62E4000   push    <jmp.&MSVBVM60.__vbaExceptHandle>;  SE 处理程序安装
0049336B   .  64:A1 0000000>mov     eax, dword ptr fs:[0]
00493371   .  50            push    eax
00493372   .  64:8925 00000>mov     dword ptr fs:[0], esp
00493379   .  81EC 00040000 sub     esp, 400
0049337F   .  53            push    ebx
00493380   .  56            push    esi
00493381   .  57            push    edi
00493382   .  8965 F4       mov     dword ptr [ebp-C], esp
00493385   .  C745 F8 10144>mov     dword ptr [ebp-8], 00401410
0049338C   .  33F6          xor     esi, esi
0049338E   .  8D95 0CFDFFFF lea     edx, dword ptr [ebp-2F4]
00493394   .  8D8D 7CFFFFFF lea     ecx, dword ptr [ebp-84]
0049339A   .  8975 DC       mov     dword ptr [ebp-24], esi
0049339D   .  8975 CC       mov     dword ptr [ebp-34], esi
004933A0   .  8975 BC       mov     dword ptr [ebp-44], esi
004933A3   .  8975 B8       mov     dword ptr [ebp-48], esi
```
