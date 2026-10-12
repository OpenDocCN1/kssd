
# 转图精灵算法分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-160.htm`](https://www.pediy.com/kssd/pediy09/pediy09-160.htm)

> 【破文标题】转图精灵算法分析  
> 【破文作者】XXNB  
> 【作者邮箱】289704427@163.com  
> 【作者主页】binbinbin7456.ys168.com  
> 【破解声明】学习交流用。请支持正版  
> \------------------------------------------------------------------------  
> 【破解过程】

因为我做了个视频的教程（黑鹰 06-07 发布了），所以顺便发了这个破文，留作纪念。

一个附加在浏览器里面的小插件，`VB` 写的 `dll`。重启验证的例子。比较简单

1、首先载入浏览器，我是用 `TheWorld` 上网的，然后查看他调用的 `dll`。结果就发现了这个转图精灵的 `dll`。`unicode` 查找，找到了些字符串，作者没有加密。运行起来随便输入：`3132132166`，点击注册。下断点 `bp rtcMsgBox`。返回到得到下面：

```
11009790    55                 push    ebp
11009791    8BEC               mov     ebp, esp
11009793    83EC 14            sub     esp, 14
11009796    68 06150011        push    <jmp.&MSVBVM60.__vbaExceptHandle>
1100979B    64:A1 00000000     mov     eax, fs:[0]
110097A1    50                 push    eax
110097A2    64:8925 0000000>   mov     fs:[0], esp
110097A9    81EC B0000000      sub     esp, 0B0
110097AF    53                 push    ebx
110097B0    56                 push    esi
110097B1    57                 push    edi
110097B2    8965 EC            mov     [ebp-14], esp
110097B5    C745 F0 5814001>   mov     dword ptr [ebp-10], 11001458
110097BC    8B75 08            mov     esi, [ebp+8]
110097BF    8BC6               mov     eax, esi
110097C1    83E0 01            and     eax, 1
110097C4    8945 F4            mov     [ebp-C], eax
110097C7    83E6 FE            and     esi, FFFFFFFE
110097CA    8975 08            mov     [ebp+8], esi
110097CD    33DB               xor     ebx, ebx
110097CF    895D F8            mov     [ebp-8], ebx
110097D2    8B0E               mov     ecx, [esi]
110097D4    56                 push    esi
110097D5    FF51 04            call    [ecx+4]
110097D8    895D E0            mov     [ebp-20], ebx
110097DB    895D DC            mov     [ebp-24], ebx
110097DE    895D CC            mov     [ebp-34], ebx
110097E1    895D BC            mov     [ebp-44], ebx
110097E4    895D AC            mov     [ebp-54], ebx
110097E7    895D 9C            mov     [ebp-64], ebx
110097EA    895D 8C            mov     [ebp-74], ebx
110097ED    899D 7CFFFFFF      mov     [ebp-84], ebx
110097F3    68 0C2E0011        push    11002E0C
110097F8    FF15 1C100011      call    [<&MSVBVM60.__vbaAptOffset>]     ; MSVBVM60.__vbaAptOffset
110097FE    8985 3CFFFFFF      mov     [ebp-C4], eax
11009804    6A 01             push    1
11009806    FF15 74100011      call    [<&MSVBVM60.__vbaOnError>]       ; MSVBVM60.__vbaOnError
1100980C    8B16               mov     edx, [esi]
1100980E    56                 push    esi
1100980F    FF92 0C030000      call    [edx+30C]
11009815    50                 push    eax
11009816    8D45 DC            lea     eax, [ebp-24]
11009819    50                 push    eax
1100981A    FF15 70100011      call    [<&MSVBVM60.__vbaObjSet>]        ; MSVBVM60.__vbaObjSet
11009820    8BF8               mov     edi, eax
11009822    8B0F               mov     ecx, [edi]
11009824    8D55 E0            lea     edx, [ebp-20]
11009827    52                 push    edx
11009828    57                 push    edi
11009829    FF91 A0000000      call    [ecx+A0]
1100982F    DBE2               fclex
11009831    3BC3               cmp     eax, ebx
11009833    7D 12              jge     short 11009847
11009835    68 A0000000        push    0A0
1100983A    68 543B0011        push    11003B54
1100983F    57                 push    edi
11009840    50                 push    eax
11009841    FF15 54100011      call    [<&MSVBVM60.__vbaHresultCheckObj>; MSVBVM60.__vbaHresultCheckObj
11009847    8B45 E0            mov     eax, [ebp-20]                  ; 读出我们输入的假码
1100984A    50                 push    eax
1100984B    68 84340011        push    11003484
11009850    FF15 A8100011      call    [<&MSVBVM60.__vbaStrCmp>]        ; MSVBVM60.__vbaStrCmp
11009856    8BF8               mov     edi, eax
11009858    F7DF               neg     edi
1100985A    1BFF               sbb     edi, edi
1100985C    47                 inc     edi
1100985D    F7DF               neg     edi
1100985F    8D4D E0            lea     ecx, [ebp-20]
11009862    FF15 A8110011      call    [<&MSVBVM60.__vbaFreeStr>]       ; MSVBVM60.__vbaFreeStr
11009868    8D4D DC            lea     ecx, [ebp-24]
1100986B    FF15 A4110011      call    [<&MSVBVM60.__vbaFreeObj>]       ; MSVBVM60.__vbaFreeObj
11009871    66:3BFB            cmp     di, bx
11009874    0F84 83000000      je      110098FD                         ; 这些是检测，一定要跳的
1100987A    B9 04000280        mov     ecx, 80020004
1100987F    894D A4            mov     [ebp-5C], ecx
11009882    B8 0A000000        mov     eax, 0A
11009887    8945 9C            mov     [ebp-64], eax
1100988A    894D B4            mov     [ebp-4C], ecx
1100988D    8945 AC            mov     [ebp-54], eax
11009890    C745 84 8030001>   mov     dword ptr [ebp-7C], 11003080    ; ASCII "l`V}譽 p1"
11009897    BE 08000000        mov     esi, 8
1100989C    89B5 7CFFFFFF      mov     [ebp-84], esi
110098A2    8D95 7CFFFFFF      lea     edx, [ebp-84]
110098A8    8D4D BC            lea     ecx, [ebp-44]
110098AB    8B3D 58110011      mov     edi, [<&MSVBVM60.__vbaVarDup>]   ; MSVBVM60.__vbaVarDup
110098B1    FFD7               call    edi
110098B3    C745 94 743E001>   mov     dword ptr [ebp-6C], 11003E74
110098BA    8975 8C            mov     [ebp-74], esi
110098BD    8D55 8C            lea     edx, [ebp-74]
110098C0    8D4D CC            lea     ecx, [ebp-34]
110098C3    FFD7               call    edi
110098C5    8D4D 9C            lea     ecx, [ebp-64]
110098C8    51                 push    ecx
110098C9    8D55 AC            lea     edx, [ebp-54]
110098CC    52                 push    edx
110098CD    8D45 BC            lea     eax, [ebp-44]
110098D0    50                 push    eax
110098D1    6A 40             push    40
110098D3    8D4D CC            lea     ecx, [ebp-34]
110098D6    51                 push    ecx
110098D7    FF15 78100011      call    [<&MSVBVM60.#595>]              ; MSVBVM60.rtcMsgBox
110098DD    8D55 9C            lea     edx, [ebp-64]
110098E0    52                 push    edx
110098E1    8D45 AC            lea     eax, [ebp-54]
110098E4    50                 push    eax
110098E5    8D4D BC            lea     ecx, [ebp-44]
110098E8    51                 push    ecx
110098E9    8D55 CC            lea     edx, [ebp-34]
110098EC    52                 push    edx
110098ED    6A 04             push    4
110098EF    FF15 28100011      call    [<&MSVBVM60.__vbaFreeVarList>]   ; MSVBVM60.__vbaFreeVarList
110098F5    83C4 14            add     esp, 14
110098F8    E9 3A020000        jmp     11009B37
110098FD    8B06               mov     eax, [esi]
110098FF    56                 push    esi
11009900    FF90 0C030000      call    [eax+30C]
11009906    50                 push    eax
11009907    8D4D DC            lea     ecx, [ebp-24]
1100990A    51                 push    ecx
1100990B    FF15 70100011      call    [<&MSVBVM60.__vbaObjSet>]        ; MSVBVM60.__vbaObjSet
11009911    8BF8               mov     edi, eax
11009913    8B17               mov     edx, [edi]
11009915    8D45 E0            lea     eax, [ebp-20]
11009918    50                 push    eax
11009919    57                 push    edi
1100991A    FF92 A0000000      call    [edx+A0]
11009920    DBE2               fclex
11009922    3BC3               cmp     eax, ebx
11009924    7D 12              jge     short 11009938
11009926    68 A0000000        push    0A0
```



1. **反汇编代码片段 1**

```asm
1100992B    68 543B0011     push    11003B54
11009930    57              push    edi
11009931    50              push    eax
11009932    FF15 54100011   call    [<&MSVBVM60.__vbaHresultCheckObj>]; MSVBVM60.__vbaHresultCheckObj
11009938    8B4D E0         mov     ecx, [ebp-20]                     ; 又读出
1100993B    51              push    ecx
1100993C    FF15 24100011   call    [<&MSVBVM60.__vbaLenBstr>]        ; MSVBVM60.__vbaLenBstr
11009942    33D2            xor     edx, edx
11009944    83F8 0F         cmp     eax, 0F                           ; 这里比较
11009947    0F95C2          setne   dl                                ; 要求 15 位数的假码。
1100994A    F7DA            neg     edx
1100994C    8BFA            mov     edi, edx
1100994E    8D4D E0         lea     ecx, [ebp-20]
11009951    FF15 A8110011   call    [<&MSVBVM60.__vbaFreeStr>]        ; MSVBVM60.__vbaFreeStr
11009957    8D4D DC         lea     ecx, [ebp-24]
1100995A    FF15 A4110011   call    [<&MSVBVM60.__vbaFreeObj>]        ; MSVBVM60.__vbaFreeObj
11009960    66:3BFB         cmp     di, bx
11009963    0F84 9D000000   je      11009A06                          ; 要跳的
11009969    B9 04000280     mov     ecx, 80020004
1100996E    894D A4         mov     [ebp-5C], ecx
11009971    B8 0A000000     mov     eax, 0A
11009976    8945 9C         mov     [ebp-64], eax
11009979    894D B4         mov     [ebp-4C], ecx
1100997C    8945 AC         mov     [ebp-54], eax
1100997F    C745 94 8030001>mov     dword ptr [ebp-6C], 11003080      ; ASCII "l`V}譽 p1"
11009986    BE 08000000     mov     esi, 8
1100998B    8975 8C         mov     [ebp-74], esi
1100998E    8D55 8C         lea     edx, [ebp-74]
11009991    8D4D BC         lea     ecx, [ebp-44]
11009994    FF15 58110011   call    [<&MSVBVM60.__vbaVarDup>]         ; MSVBVM60.__vbaVarDup
1100999A    68 983E0011     push    11003E98
1100999F    68 FC340011     push    110034FC                          ; \n\n
110099A4    8B1D 48100011   mov     ebx, [<&MSVBVM60.__vbaStrCat>]    ; MSVBVM60.__vbaStrCat
110099AA    FFD3            call    ebx
110099AC    8BD0            mov     edx, eax
110099AE    8D4D E0         lea     ecx, [ebp-20]
110099B1    FF15 7C110011   call    [<&MSVBVM60.__vbaStrMove>]        ; MSVBVM60.__vbaStrMove
110099B7    50              push    eax
110099B8    68 A43B0011     push    11003BA4
110099BD    FFD3            call    ebx
110099BF    8945 D4         mov     [ebp-2C], eax
110099C2    8975 CC         mov     [ebp-34], esi
110099C5    8D45 9C         lea     eax, [ebp-64]
110099C8    50              push    eax
110099C9    8D4D AC         lea     ecx, [ebp-54]
110099CC    51              push    ecx
110099CD    8D55 BC         lea     edx, [ebp-44]
110099D0    52              push    edx
110099D1    6A 40           push    40
110099D3    8D45 CC         lea     eax, [ebp-34]
110099D6    50              push    eax
110099D7    FF15 78100011   call    [<&MSVBVM60.#595>]                ; MSVBVM60.rtcMsgBox
110099DD    8D4D E0         lea     ecx, [ebp-20]                     ; 堆栈返回到这里~~上面是出错信息框
110099E0    FF15 A8110011   call    [<&MSVBVM60.__vbaFreeStr>]        ; MSVBVM60.__vbaFreeStr
110099E6    8D4D 9C         lea     ecx, [ebp-64]
110099E9    51              push    ecx
110099EA    8D55 AC         lea     edx, [ebp-54]
110099ED    52              push    edx
110099EE    8D45 BC         lea     eax, [ebp-44]
110099F1    50              push    eax
110099F2    8D4D CC         lea     ecx, [ebp-34]
110099F5    51              push    ecx
110099F6    6A 04           push    4
110099F8    FF15 28100011   call    [<&MSVBVM60.__vbaFreeVarList>]    ; MSVBVM60.__vbaFreeVarList
110099FE    83C4 14         add     esp, 14
11009A01    E9 31010000     jmp     11009B37
11009A06    8B16            mov     edx, [esi]
11009A08    56              push    esi
11009A09    FF92 08070000   call    [edx+708]                         ; 这里是保存注册信息 call
```

2. **假码输入 15 位数后重新按确定。进入到下面的保存注册信息的 call：**

```asm
1100A120    55              push    ebp
1100A121    8BEC            mov     ebp, esp
1100A123    83EC 08         sub     esp, 8
1100A126    68 06150011     push    <jmp.&MSVBVM60.__vbaExceptHandle>
1100A12B    64:A1 00000000  mov     eax, fs:[0]
1100A131    50              push    eax
1100A132    64:8925 0000000>mov     fs:[0], esp
1100A139    83EC 20         sub     esp, 20
1100A13C    53              push    ebx
1100A13D    56              push    esi
1100A13E    57              push    edi
1100A13F    8965 F8         mov     [ebp-8], esp
1100A142    C745 FC B014001>mov     dword ptr [ebp-4], 110014B0
1100A149    8D45 EC         lea     eax, [ebp-14]
1100A14C    33FF            xor     edi, edi
1100A14E    50              push    eax
1100A14F    68 3C300011     push    1100303C                          ; .tst4\openwithlist\delphi.exe\
1100A154    68 E4300011     push    110030E4                          ; {3a0c97b5-3d1c-4ce8-bna9-00bpb3q522l7}
1100A159    897D EC         mov     [ebp-14], edi                   ; 注册表查找上面这串就可以看到信息
1100A15C    897D E8         mov     [ebp-18], edi
1100A15F    897D E4         mov     [ebp-1C], edi
1100A162    897D E0         mov     [ebp-20], edi
1100A165    897D DC         mov     [ebp-24], edi
1100A168    FF15 48100011   call    [<&MSVBVM60.__vbaStrCat>]         ; MSVBVM60.__vbaStrCat
1100A16E    8BD0            mov     edx, eax                        ; 连接函数
1100A170    8D4D E8         lea     ecx, [ebp-18]
1100A173    FF15 7C110011   call    [<&MSVBVM60.__vbaStrMove>]        ; MSVBVM60.__vbaStrMove
1100A179    8B35 5C110011   mov     esi, [<&MSVBVM60.__vbaStrToAnsi>>; MSVBVM60.__vbaStrToAnsi
1100A17F    8D4D E4         lea     ecx, [ebp-1C]
1100A182    50              push    eax
1100A183    51              push    ecx
1100A184    FFD6            call    esi
1100A186    50              push    eax
1100A187    68 00000080     push    80000000
1100A18C    E8 8B95FFFF     call    1100371C
1100A191    8B1D 50100011   mov     ebx, [<&MSVBVM60.__vbaSetSystemE>; MSVBVM60.__vbaSetSystemError
1100A197    FFD3            call    ebx
1100A199    8D55 E4         lea     edx, [ebp-1C]
1100A19C    8D45 E8         lea     eax, [ebp-18]
1100A19F    52              push    edx
1100A1A0    50              push    eax
1100A1A1    6A 02           push    2
1100A1A3    FF15 28110011   call    [<&MSVBVM60.__vbaFreeStrList>]    ; MSVBVM60.__vbaFreeStrList
1100A1A9    83C4 0C         add     esp, 0C
1100A1AC    8D4D E4         lea     ecx, [ebp-1C]
1100A1AF    6A 0F           push    0F
1100A1B1    68 643F0011     push    11003F64                          ; delphi.mdp.6.7
1100A1B6    51              push    ecx
1100A1B7    FFD6            call    esi
1100A1B9    50              push    eax
1100A1BA    6A 01           push    1
1100A1BC    57              push    edi
1100A1BD    8D55 E8         lea     edx, [ebp-18]
1100A1C0    68 84340011     push    11003484
1100A1C5    52              push    edx
1100A1C6    FFD6            call    esi
1100A1C8    50              push    eax
1100A1C9    8B45 EC         mov     eax, [ebp-14]
1100A1CC    50              push    eax
1100A1CD    E8 9295FFFF     call    11003764
1100A1D2    FFD3            call    ebx
```



```
1100A1D4    8D4D E4         lea     ecx, [ebp-1C]  
1100A1D7    8D55 E8         lea     edx, [ebp-18]  
1100A1DA    51              push    ecx  
1100A1DB    52              push    edx  
1100A1DC    6A 02           push    2  
1100A1DE    FF15 28110011   call    [<&MSVBVM60.__vbaFreeStrList>]   ; MSVBVM60.__vbaFreeStrList  
1100A1E4    8B45 08         mov     eax, [ebp+8]  
1100A1E7    83C4 0C         add     esp, 0C  
1100A1EA    8B08            mov     ecx, [eax]  
1100A1EC    50              push    eax  
1100A1ED    FF91 0C030000   call    [ecx+30C]  
1100A1F3    8D55 DC         lea     edx, [ebp-24]  
1100A1F6    50              push    eax  
1100A1F7    52              push    edx  
1100A1F8    FF15 70100011   call    [<&MSVBVM60.__vbaObjSet>]        ; MSVBVM60.__vbaObjSet  
1100A1FE    8BF8            mov     edi, eax  
1100A200    8D4D E8         lea     ecx, [ebp-18]  
1100A203    51              push    ecx  
1100A204    57              push    edi  
1100A205    8B07            mov     eax, [edi]  
1100A207    FF90 A0000000   call    [eax+A0]  
1100A20D    85C0            test    eax, eax  
1100A20F    DBE2            fclex  
1100A211    7D 12           jge     short 1100A225  
1100A213    68 A0000000     push    0A0  
1100A218    68 543B0011     push    11003B54  
1100A21D    57              push    edi  
1100A21E    50              push    eax  
1100A21F    FF15 54100011   call    [<&MSVBVM60.__vbaHresultCheckObj>; MSVBVM60.__vbaHresultCheckObj  
1100A225    8B55 E8         mov     edx, [ebp-18]  
1100A228    6A 10           push    10  
1100A22A    8D45 E0         lea     eax, [ebp-20]  
1100A22D    52              push    edx  
1100A22E    50              push    eax  
1100A22F    FFD6            call    esi  
1100A231    50              push    eax  
1100A232    6A 01           push    1  
1100A234    6A 00           push    0  
1100A236    8D4D E4         lea     ecx, [ebp-1C]  
1100A239    68 A8380011     push    110038A8                     ; sysinternal  
1100A23E    51              push    ecx                           ; 保存在这个项下面  
1100A23F    FFD6            call    esi                           ; 下面是 closereg  
```

3、unicode 查找字符串`"sysinternal"`，下断，重启。来到下面这里

```
11007450    55              push    ebp                       ; 这个软件是重启验证的  
11007451    8BEC            mov     ebp, esp  
11007453    83EC 0C         sub     esp, 0C  
11007456    68 06150011     push    <jmp.&MSVBVM60.__vbaExceptHandle>  
1100745B    64:A1 00000000  mov     eax, fs:[0]  
11007461    50              push    eax  
11007462    64:8925 0000000>mov     fs:[0], esp  
11007469    83EC 68         sub     esp, 68  
1100746C    53              push    ebx  
1100746D    56              push    esi  
1100746E    57              push    edi  
1100746F    8965 F4         mov     [ebp-C], esp  
11007472    C745 F8 F012001>mov     dword ptr [ebp-8], 110012F0  
11007479    B9 07000000     mov     ecx, 7  
1100747E    33C0            xor     eax, eax  
11007480    8D7D C8         lea     edi, [ebp-38]  
11007483    33F6            xor     esi, esi  
11007485    F3:AB           rep     stos dword ptr es:[edi]  
11007487    8D4D E8         lea     ecx, [ebp-18]  
1100748A    8975 E8         mov     [ebp-18], esi  
1100748D    66:AB           stos    word ptr es:[edi]  
1100748F    8B45 0C         mov     eax, [ebp+C]  
11007492    51              push    ecx  
11007493    68 3C300011     push    1100303C                     ; .tst4\\openwithlist\\delphi.exe\\  
11007498    68 E4300011     push    110030E4                     ; {3a0c97b5-3d1c-4ce8-bna9-00bpb3q522l7}  
1100749D    8975 C4         mov     [ebp-3C], esi                 ; 查找上面那串，在注册表  
110074A0    8975 C0         mov     [ebp-40], esi  
110074A3    8975 BC         mov     [ebp-44], esi  
110074A6    8975 B8         mov     [ebp-48], esi  
110074A9    8975 B4         mov     [ebp-4C], esi  
110074AC    8975 A4         mov     [ebp-5C], esi  
110074AF    8975 94         mov     [ebp-6C], esi  
110074B2    8975 90         mov     [ebp-70], esi  
110074B5    8975 8C         mov     [ebp-74], esi  
110074B8    8930            mov     [eax], esi  
110074BA    FF15 48100011   call    [<&MSVBVM60.__vbaStrCat>]        ; MSVBVM60.__vbaStrCat  
110074C0    8B3D 7C110011   mov     edi, [<&MSVBVM60.__vbaStrMove>]  ; MSVBVM60.__vbaStrMove  
110074C6    8BD0            mov     edx, eax  
110074C8    8D4D C0         lea     ecx, [ebp-40]  
110074CB    FFD7            call    edi  
110074CD    8B1D 5C110011   mov     ebx, [<&MSVBVM60.__vbaStrToAnsi>>; MSVBVM60.__vbaStrToAnsi  
110074D3    8D55 BC         lea     edx, [ebp-44]  
110074D6    50              push    eax  
110074D7    52              push    edx  
110074D8    FFD3            call    ebx  
110074DA    50              push    eax  
110074DB    68 00000080     push    80000000  
110074E0    E8 37C2FFFF     call    1100371C  
110074E5    FF15 50100011   call    [<&MSVBVM60.__vbaSetSystemError>>; MSVBVM60.__vbaSetSystemError  
110074EB    8D45 BC         lea     eax, [ebp-44]  
110074EE    8D4D C0         lea     ecx, [ebp-40]  
110074F1    50              push    eax  
110074F2    51              push    ecx  
110074F3    6A 02           push    2  
110074F5    FF15 28110011   call    [<&MSVBVM60.__vbaFreeStrList>]   ; MSVBVM60.__vbaFreeStrList  
110074FB    83C4 0C         add     esp, 0C  
110074FE    8D55 8C         lea     edx, [ebp-74]  
11007501    8D45 C8         lea     eax, [ebp-38]  
11007504    C745 8C 1000000>mov     dword ptr [ebp-74], 10  
1100750B    52              push    edx  
1100750C    50              push    eax  
1100750D    6A 0F           push    0F  
1100750F    C745 90 0100000>mov     dword ptr [ebp-70], 1  
11007516    FF15 90100011   call    [<&MSVBVM60.__vbaStrFixstr>]     ; MSVBVM60.__vbaStrFixstr  
1100751C    8BD0            mov     edx, eax  
1100751E    8D4D BC         lea     ecx, [ebp-44]  
11007521    FFD7            call    edi  
11007523    8D4D B8         lea     ecx, [ebp-48]  
11007526    50              push    eax  
11007527    51              push    ecx  
11007528    FFD3            call    ebx  
1100752A    8D55 90         lea     edx, [ebp-70]  
1100752D    50              push    eax  
1100752E    52              push    edx  
1100752F    56              push    esi  
11007530    8D45 C0         lea     eax, [ebp-40]                   ；这里断下  
11007533    68 A8380011     push    110038A8                     ; sysinternal  
11007538    50              push    eax                           ; 注册表中的项  
11007539    FFD3            call    ebx  
1100753B    8B4D E8         mov     ecx, [ebp-18]  
1100753E    50              push    eax  
1100753F    51              push    ecx  
11007540    E8 6BC2FFFF     call    110037B0  
11007545    8B1D 50100011   mov     ebx, [<&MSVBVM60.__vbaSetSystemE>; MSVBVM60.__vbaSetSystemError  
1100754B    FFD3            call    ebx  
1100754D    8B55 B8         mov     edx, [ebp-48]  
11007550    8D45 B4         lea     eax, [ebp-4C]                   ; 看到没有，读出我们输入的假码了  
11007553    52              push    edx  
11007554    50              push    eax  
```

4、读完注册表后，返回到下面

```
11006DE0    FF50 38         call    [eax+38]                       ; 这个就是从注册表中取出假码的 call  
11006DE3    8B4D E4         mov     ecx, [ebp-1C]                   ；  retn 回到这里  
```



## 汇编代码分析

```asm
11006DE6    8B17                 mov     edx, [edi]
11006DE8    8D45 E0              lea     eax, [ebp-20]
11006DEB    50                   push    eax
11006DEC    51                   push    ecx                                ; 下面这个是固定字符串“7h91j”好像没啥用
11006DED    68 C0350011          push    110035C0                           ; 7h91j
11006DF2    57                   push    edi
11006DF3    FF52 3C              call    [edx+3C]                           ; 这里。进去《《《《《《《《《计算注册码的 call
11006DF6    E8 05240000          call    11009200                           ; 这里不能再按 F8.会直接运行的。这个 call 取出机器码
11006DFB    8BD0                 mov     edx, eax                           ; 我们直接在这里下断
11006DFD    8D4D DC              lea     ecx, [ebp-24]
11006E00    FF15 7C110011        call    [<MSVBVM60.__vbaStrMove>]          ; MSVBVM60.__vbaStrMove
11006E06    8B55 E0              mov     edx, [ebp-20]                      ; 这里就是刚才算法算出来的东西了
11006E09    50                   push    eax
11006E0A    52                   push    edx                                ; 到这里才发现原来是要算出的东西和机器码相等
11006E0B    FF15 A8100011        call    [<MSVBVM60.__vbaStrCmp>]           ; MSVBVM60.__vbaStrCmp
11006E11    8BF0                 mov     esi, eax                           ;
11006E13    8D45 E0              lea     eax, [ebp-20]                      ;
11006E16    F7DE                 neg     esi
11006E18    8D4D DC              lea     ecx, [ebp-24]
11006E1B    50                   push    eax
11006E1C    1BF6                 sbb     esi, esi
11006E1E    8D55 E4              lea     edx, [ebp-1C]
11006E21    51                   push    ecx
11006E22    46                   inc     esi
11006E23    52                   push    edx
11006E24    6A 03                push    3
11006E26    F7DE                 neg     esi
11006E28    FF15 28110011        call    [<MSVBVM60.__vbaFreeStrList>]      ; MSVBVM60.__vbaFreeStrList
11006E2E    83C4 10              add     esp, 10
11006E31    66:3BF3              cmp     si, bx
11006E34    74 21                je      short 11006E57                     ; 这个如果跳的话就成功
11006E36    8B85 48FFFFFF        mov     eax, [ebp-B8]                      ; 爆破就在这里啦
```

## 假码变换逻辑

变换假码的地方是：变换后的假码 `=` 机器码的话，我们输入的假码就是正确的。

```asm
1100773C    8945 E8              mov     [ebp-18], eax
1100773F    66:3B45 D8           cmp     ax, [ebp-28]                       ; 循环
11007743    0F8F 0E010000        jg      11007857
11007749    0FBFD0               movsx   edx, ax                            ; 逐位检测
1100774C    8B45 D4              mov     eax, [ebp-2C]
1100774F    8D4D B8              lea     ecx, [ebp-48]
11007752    51                   push    ecx
11007753    52                   push    edx
11007754    50                   push    eax
11007755    C745 C0 0100000      mov     dword ptr [ebp-40], 1
1100775C    C745 B8 0200000      mov     dword ptr [ebp-48], 2
11007763    FF15 98100011        call    [<MSVBVM60.#631>]                 ; MSVBVM60.rtcMidCharBstr
11007769    8BD0                 mov     edx, eax
1100776B    8D4D C8              lea     ecx, [ebp-38]
1100776E    FFD7                 call    edi
11007770    50                   push    eax
11007771    FF15 34100011        call    [<MSVBVM60.#516>]                 ; MSVBVM60.rtcAnsiValueBstr
11007777    8D4D C8              lea     ecx, [ebp-38]                      ; 看寄存器
1100777A    8BF0                 mov     esi, eax
1100777C    FF15 A8110011        call    [<MSVBVM60.__vbaFreeStr>]         ; MSVBVM60.__vbaFreeStr
11007782    8D4D B8              lea     ecx, [ebp-48]
11007785    FFD3                 call    ebx
11007787    66:83FE 20           cmp     si, 20                             ; 和空格比较
1100778B    0F8C B3000000        jl      11007844
11007791    66:83FE 7E           cmp     si, 7E                             ; 和“~”比较
11007795    0F8F A9000000        jg      11007844
1100779B    8D4D B8              lea     ecx, [ebp-48]
1100779E    66:83EE 20           sub     si, 20                             ; 减去 20H
110077A2    51                   push    ecx
110077A3    C745 C0 0400028      mov     dword ptr [ebp-40], 80020004
110077AA    0F80 15010000        jo      110078C5
110077B0    C745 B8 0A00000      mov     dword ptr [ebp-48], 0A
110077B7    FF15 68100011        call    [<MSVBVM60.#593>]                 ; MSVBVM60.rtcRandomNext
110077BD    D95D A4              fstp    dword ptr [ebp-5C]                ; 随机生成一个数
110077C0    D945 A4              fld     dword ptr [ebp-5C]                ; 其实是固定的一个表。
110077C3    D80D 00130011        fmul    dword ptr [11001300]
110077C9    DFE0                 fstsw   ax
110077CB    A8 0D                test    al, 0D
110077CD    0F85 ED000000        jnz     110078C0
110077D3    FF15 88110011        call    [<MSVBVM60.__vbaR8IntI4>]        ; MSVBVM60.__vbaR8IntI4
110077D9    8D4D B8              lea     ecx, [ebp-48]
110077DC    8945 DC              mov     [ebp-24], eax
110077DF    FFD3                 call    ebx
110077E1    8B55 DC              mov     edx, [ebp-24]
110077E4    B9 5F000000          mov     ecx, 5F
110077E9    0FBFC6               movsx   eax, si
110077EC    2BC2                 sub     eax, edx                           ; 相减
110077EE    0F80 D1000000        jo      110078C5
110077F4    99                   cdq
110077F5    F7F9                 idiv    ecx
110077F7    8BCA                 mov     ecx, edx                           ; 整除，余数到 ecx
110077F9    FF15 B4100011        call    [<MSVBVM60.__vbaI2I4>]            ; MSVBVM60.__vbaI2I4
110077FF    66:85C0              test    ax, ax                             ; 如果相减的结果大于 0 的话就不加 5F。
11007802    7D 0A                jge     short 1100780E
11007804    66:05 5F00           add     ax, 5F                             ; 又加上 5f
11007808    0F80 B7000000        jo      110078C5
1100780E    8B55 D0              mov     edx, [ebp-30]
11007811    66:05 2000           add     ax, 20                             ; 加上 20H
11007815    0F80 AA000000        jo      110078C5
1100781B    0FBFC0               movsx   eax, ax
1100781E    52                   push    edx
1100781F    50                   push    eax
11007820    FF15 04110011        call    [<MSVBVM60.#537>]                 ; MSVBVM60.rtcBstrFromAnsi
11007826    8BD0                 mov     edx, eax
11007828    8D4D C8              lea     ecx, [ebp-38]
1100782B    FFD7                 call    edi
1100782D    50                   push    eax
1100782E    FF15 48100011        call    [<MSVBVM60.__vbaStrCat>]           ; MSVBVM60.__vbaStrCat
11007834    8BD0                 mov     edx, eax
11007836    8D4D D0              lea     ecx, [ebp-30]
11007839    FFD7                 call    edi
1100783B    8D4D C8              lea     ecx, [ebp-38]
1100783E    FF15 A8110011        call    [<MSVBVM60.__vbaFreeStr>]         ; MSVBVM60.__vbaFreeStr
11007844    B8 01000000          mov     eax, 1
11007849    66:0345 E8           add     ax, [ebp-18]
1100784D    70 76                jo      short 110078C5
1100784F    8945 E8              mov     [ebp-18], eax
11007852    E9 E8FEFFFF          jmp     1100773F                           ; 往回跳
11007857    8B55 D0              mov     edx, [ebp-30]
1100785A    8D4D E0              lea     ecx, [ebp-20]                      ; 这里出现的是由假码运算出来的字符串。就一个循环
```

## 破解总结

多谢耐心看完。

由输入的假码运算后，如果等于机器码就注册成功。

下面是那个循环的易语言实现方式：

```text
.版本 2

编辑框 2.内容 ＝ “”
sz ＝ { 20, 29, 19, 92, 53, 54, 78, 16, 89, 60, 26, 8, 60, 44, 26 }
.计次循环首 (取文本长度 (编辑框 1.内容), i)
    a ＝ 取代码 (编辑框 1.内容, i)
    a ＝ a － 32
    a ＝ a － sz [i]
    b ＝ a ％ 95
    c ＝ 位与 (b, 65535)
    .如果 (b ＞ 0)
        c ＝ c ＋ 32
    .否则
        c ＝ c ＋ 95 ＋ 32
    .如果结束
    编辑框 2.加入文本 (字符 (c))
.计次循环尾 ()
```

下面是易语言注册机算法：



版本 2

程序集 窗口程序集 1

程序集变量 `sz`, 整数型, , "0"

子程序 `__ 启动窗口_ 创建完毕`

```
sz ＝ { 20, 29, 19, 92, 53, 54, 78, 16, 89, 60, 26, 8, 60, 44, 26 }
```

子程序 `_ 按钮 2_ 被单击`

局部变量 `i`, 整数型

```
.计次循环首 (15, i)
    运算 (i, 取文本中间 (编辑框 1.内容, i, 1))
.计次循环尾 ()
```

子程序 `运算`

参数 `数组固定值`, 整数型

参数 `机器码字符`, 文本型

局部变量 `k`, 整数型

局部变量 `a`, 整数型

局部变量 `b`, 整数型

局部变量 `c`, 整数型

```
.计次循环首 (94, k)
    a ＝ k ＋ 31
    a ＝ a － 32
    a ＝ a － sz [数组固定值]
    b ＝ a ％ 95
    c ＝ 位与 (b, 65535)  ' 这个是取低位
    .如果 (b ＞ 0)
        c ＝ c ＋ 32
    .否则
        c ＝ c ＋ 95 ＋ 32
    .如果结束
    .如果真 (字符 (c) ＝ 机器码字符)
        输出调试文本 (到文本 (字符 (k ＋ 31)))
        编辑框 2.加入文本 (到文本 (字符 (k ＋ 31)))
    .如果真结束
    a ＝ 0
.计次循环尾 ()
```

`4dd466d0e1e0cc123afbe504da2462ca`
