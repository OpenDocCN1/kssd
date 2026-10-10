

# Quick unpack 查壳方式

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-324.htm`](https://www.pediy.com/kssd/pediy08/pediy8-324.htm)

![](img/9e41176b4900ac914e77ca870db1e3dd.jpg) 被老大一吹我就激动了，发个 QU 的查壳方式吧。  
  
  
                                    Quick unpack 查壳方式  
  
by nbw  
4 \[NE365\]  
  
    很多查壳软件，原理都类似，前阵子 backer 同学说了一下 peid 的查壳方式，很不错。我这里单表 quick unpack 查壳方式。用的是 Quick Unpack v1.0。查壳分为 2 种，一种是硬编码查找 OEP 地方的特征，类似弄个反汇编引擎检查每句话是啥指令，还有一种是自己组织了一个壳的特征库，将 OEP 代码跟每个库进行比较，有个相似度比较。下面说以下。  
  
1.直接硬编码比较  
  
//打开目标文件  
012B35B7          6A 03           push    3  
012B35B9          68 000000C0     push    C0000000  
012B35BE          53              push    ebx  
012B35BF          E8 08FFFFFF     call    <jmp.&kernel32.CreateFileA>  
  
//判断 MZ 标记  
012B35EF          50              push    eax  
012B35F0          E8 E7FEFFFF     call    <jmp.&kernel32.ReadFile>  
012B35F5          66:813D 74562B0>cmp     word ptr \[12B5674\], 5A4D  
012B35FE          74 15           je      short 012B3615  
  
//判断 PE 标记  
012B365B          68 F8000000     push    0F8  
012B3660          68 80562B01     push    012B5680                                  ; ASCII "PE"  
012B3665          A1 6C562B01     mov     eax, \[12B566C\]  
012B366A          50              push    eax  
012B366B          E8 6CFEFFFF     call    <jmp.&kernel32.ReadFile>  
012B3670          813D 80562B01 5>cmp     dword ptr \[12B5680\], 4550  
012B367A          74 15           je      short 012B3691  
  
//读入每个节区属性  
012B36A6          6A 00           push    0  
012B36A8          68 7C562B01     push    012B567C  
012B36AD          6A 28           push    28  
012B36AF          A1 78562B01     mov     eax, \[12B5678\]  
012B36B4          8D0480          lea     eax, \[eax+eax\*4\]  
012B36B7          8D04C5 50572B01 lea     eax, \[eax\*8+12B5750\]  
012B36BE          50              push    eax  
012B36BF          A1 6C562B01     mov     eax, \[12B566C\]  
012B36C4          50              push    eax  
012B36C5          E8 12FEFFFF     call    <jmp.&kernel32.ReadFile>  
012B36CA          FF05 78562B01   inc     dword ptr \[12B5678\]  
012B36D0          4B              dec     ebx  
012B36D1        ^ 75 D3           jnz     short 012B36A6  
  
//读入 OEP 内容，长度 0x64  
012B36ED          6A 00           push    0  
012B36EF          68 7C562B01     push    012B567C  
012B36F4          6A 64           push    64  
012B36F6          56              push    esi  
012B36F7          A1 6C562B01     mov     eax, \[12B566C\]  
012B36FC          50              push    eax  
012B36FD          E8 DAFDFFFF     call    <jmp.&kernel32.ReadFile>  
  
下面开始判断壳的类型。方法就是根据入口处的特征码。比如我这里读入的 OEP 内容是：  
  
00401092 > $  B8 BC584000   mov     eax, 004058BC  
00401097   .  50            push    eax  
00401098   .  64:FF35 00000>push    dword ptr fs:\[0\]  
0040109F   .  64:8925 00000>mov     fs:\[0\], esp  
004010A6   .  33C0          xor     eax, eax  
004010A8   .  8908          mov     \[eax\], ecx  
004010AA   .  50            push    eax  
004010AB   .  45            inc     ebp  
004010AC   .  43            inc     ebx  
004010AD   .  6F            outs    dx, dword ptr es:\[edi\]  
004010AE   .  6D            ins     dword ptr es:\[edi\], dx  
004010AF   .  70 61         jo      short 00401112  
  
//判断每句 OEP 部分的一些特征  
012B3820          803E B8         cmp     byte ptr \[esi\], 0B8  
012B3823          75 41           jnz     short 012B3866  
012B3825          807E 05 50      cmp     byte ptr \[esi+5\], 50  
012B3829          75 3B           jnz     short 012B3866  
012B382B          807E 06 64      cmp     byte ptr \[esi+6\], 64  
012B382F          75 35           jnz     short 012B3866  
012B3831          807E 07 FF      cmp     byte ptr \[esi+7\], 0FF  
012B3835          75 2F           jnz     short 012B3866  
012B3837          807E 08 35      cmp     byte ptr \[esi+8\], 35  
012B383B          75 29           jnz     short 012B3866  
012B383D          807E 09 00      cmp     byte ptr \[esi+9\], 0  
012B3841          75 23           jnz     short 012B3866  
012B3843          807E 0A 00      cmp     byte ptr \[esi+A\], 0  
012B3847          75 1D           jnz     short 012B3866  
012B3849          807E 0B 00      cmp     byte ptr \[esi+B\], 0  
012B384D          75 17           jnz     short 012B3866  
012B384F          807E 0C 00      cmp     byte ptr \[esi+C\], 0  
012B3853          75 11           jnz     short 012B3866  
012B3855          807E 0D 64      cmp     byte ptr \[esi+D\], 64  
012B3859          75 0B           jnz     short 012B3866  
012B385B          807E 0E 89      cmp     byte ptr \[esi+E\], 89  
012B385F          75 05           jnz     short 012B3866  
  
//以上特征都符合，认为是"PECompact 2.xx"  
012B3861          BF E4382B01     mov     edi, 012B38E4                             ; ASCII "PECompact 2.xx"  
  
  
2.相似度比较  
  
下面由 PESniffer 进行比较，这种方式比较时候把所有壳的特征组成一个链表，不再象上面那样每个壳写入一段比较代码，类似于杀毒软件特征库，便于管理、节省空间。  
  
比较时候采用模糊比较，如果一样的数据达到 90%，则认为属于该壳，虽然可能误报，但可以查出来一些壳的升级版本之类，应该说比较不错。  
  
  
//调用 PESniffer 的比较模块，传入的参数为目标文件路径： "F:\\PECompat.exe"  
00409CD5        |.  8D4C24 0C     lea     ecx, \[esp+C\]  
00409CD9        |.  51            push    ecx  
00409CDA        |.  6A 00         push    0  
00409CDC        |.  6A 00         push    0  
00409CDE        |.  50            push    eax  
00409CDF        |.  E8 00470500   call    <jmp.&PESniffer.AnalyzeFile>  
  
进入：  
  
//打开文件："F:\\PECompat.exe"  
10001C99          56              push    esi  
10001C9A          68 A7000008     push    80000A7  
10001C9F          6A 03           push    3  
10001CA1          56              push    esi  
10001CA2          6A 01           push    1  
10001CA4          68 00000080     push    80000000  
10001CA9          FF7424 20       push    dword ptr \[esp+20\]  
10001CAD          FF15 08100010   call    \[<&KERNEL32.CreateFileA>\]                   ; KERNEL32.CreateFileA  
  
//建立该文件的文件映象  
10001CBD          56              push    esi  
10001CBE          50              push    eax  
10001CBF          FF15 50100010   call    \[<&KERNEL32.GetFileSize>\]                   ; KERNEL32.GetFileSize  
10001CC5          3BC6            cmp     eax, esi  
10001CC7          A3 AC210010     mov     \[100021AC\], eax  
10001CCC          76 3D           jbe     short 10001D0B  
10001CCE          56              push    esi  
10001CCF          50              push    eax  
10001CD0          56              push    esi  
10001CD1          6A 02           push    2  
10001CD3          56              push    esi  
10001CD4          FF35 80100010   push    dword ptr \[10001080\]  
10001CDA          FF15 34100010   call    \[<&KERNEL32.CreateFileMappingA>\]            ; KERNEL32.CreateFileMappingA  
10001CE0          83F8 FF         cmp     eax, -1  
10001CE3          A3 84100010     mov     \[10001084\], eax  
10001CE8          74 21           je      short 10001D0B  
10001CEA          56              push    esi  
10001CEB          56              push    esi  
10001CEC          56              push    esi  
10001CED          6A 04           push    4  
10001CEF          50              push    eax  
10001CF0          FF15 30100010   call    \[<&KERNEL32.MapViewOfFile>\]                 ; KERNEL32.MapViewOfFile  
  
//判断 MZ 标记  
100012DE          66:813F 4D5A    cmp     word ptr \[edi\], 5A4D  
100012E3          74 08           je      short 100012ED  
  
//判断 PE 标记  
10001307          8138 50450000   cmp     dword ptr \[eax\], 4550  
1000130D        ^ 75 D6           jnz     short 100012E5  
  
下面判断壳。所有壳的特征可以看作被放在一个链表中，将链表每个节点中的特征与目标文件 OEP 的内容进行比较，如果符合，则认为是该壳。  
  
这种方法很象杀毒软件扫描病毒。  
  
//扫描循环  
10001398         /837D 0C 00      /cmp     dword ptr \[ebp+C\], 0  
1000139C         |0F85 84000000   |jnz     10001426  
                ................  
100013B3         |50              |push    eax  
100013B4         |FF75 10         |push    dword ptr \[ebp+10\]  
100013B7         |8B45 F8         |mov     eax, \[ebp-8\]  
100013BA         |03C8            |add     ecx, eax  
100013BC         |8D86 04010000   |lea     eax, \[esi+104\]  
100013C2         |FFB6 14050000   |push    dword ptr \[esi+514\]      //这个是匹配过程中略过的数据。如果匹配时候碰到该内容，则不予处理  
100013C8         |FF75 F4         |push    dword ptr \[ebp-C\]  
100013CB         |51              |push    ecx                      //读入的 OEP 内容  
100013CC         |FFB6 18050000   |push    dword ptr \[esi+518\]      //需要匹配的数据长度  
100013D2         |50              |push    eax                      //目标匹配数据  
100013D3         |E8 3B020000     |call    10001613  
100013D8         |83C4 1C         |add     esp, 1C  
100013DB         |85C0            |test    eax, eax  
100013DD         |0F84 D7000000   |je      100014BA  
                ................  
1000140C         |FF35 94210010   |push    dword ptr \[10002194\]  
10001412         |8B0D 9C210010   |mov     ecx, \[1000219C\]  
10001418         |FF75 FC         |push    dword ptr \[ebp-4\]  
1000141B         |E8 E00B0000     |call    10002000               //获得壳的名称  
10001420         |FF45 FC         |inc     dword ptr \[ebp-4\]  
10001423         |8B75 14         |mov     esi, \[ebp+14\]  
10001426         |837D 0C 01      |cmp     dword ptr \[ebp+C\], 1  
1000142A         |0F85 8D000000   |jnz     100014BD  
                ................  
100014A9         |8B0D 9C210010   |mov     ecx, \[1000219C\]  
100014AF         |FF75 FC         |push    dword ptr \[ebp-4\]  
100014B2         |E8 490B0000     |call    10002000  
100014B7         |FF45 FC         |inc     dword ptr \[ebp-4\]  
100014BA         |8B75 14         |mov     esi, \[ebp+14\]  
100014BD         |FF45 08         |inc     dword ptr \[ebp+8\]  
100014C0         |A1 A0210010     |mov     eax, \[100021A0\]  
100014C5         |8B4D 08         |mov     ecx, \[ebp+8\]  
100014C8         |3B48 04         |cmp     ecx, \[eax+4\]  
100014CB        ^\\0F8C C7FEFFFF   \\jl      10001398               //循环回去，进行下一个壳的特征比较  
  
上面的循环，关键函数是：  
  
call    10001613  
  
进入，看一下：  
  
//将 OEP 处的数据与壳特征数据进行逐字节比较，看有多少相同字节，记录到 edi 内  
//edx 纪录的是特征中的有效数据长度  
1000163B          8A08            mov     cl, \[eax\]  
1000163D          0FB6F1          movzx   esi, cl  
10001640          3B75 18         cmp     esi, \[ebp+18\]  
10001643          74 0C           je      short 10001651  
10001645          43              inc     ebx  
10001646          3B75 18         cmp     esi, \[ebp+18\]  
10001649          74 06           je      short 10001651  
1000164B          3A0C02          cmp     cl, \[edx+eax\]  
1000164E          75 01           jnz     short 10001651  
10001650          47              inc     edi  
10001651          40              inc     eax  
10001652          FF4D 0C         dec     dword ptr \[ebp+C\]  
10001655        ^ 75 E4           jnz     short 1000163B  
  
//判断是否有相同内容的数据  
10001657          33F6            xor     esi, esi  
10001659          3BFE            cmp     edi, esi  
1000165B          74 29           je      short 10001686  
  
//这里计算一样数据占的比例，用的浮点指令。相当于 (edi/ebx) \* 100  
1000165D          897D F8         mov     \[ebp-8\], edi  
10001660          8975 FC         mov     \[ebp-4\], esi  
10001663          DF6D F8         fild    qword ptr \[ebp-8\]  
10001666          895D F8         mov     \[ebp-8\], ebx  
10001669          8975 FC         mov     \[ebp-4\], esi  
1000166C          DC0D C8100010   fmul    qword ptr \[100010C8\]  
10001672          DF6D F8         fild    qword ptr \[ebp-8\]  
10001675          DEF9            fdivp   st(1), st  
10001677          E8 0A0B0000     call    <jmp.&MSVCRT.\_ftol>  
  
//判断是不是 100%一样呢？  
10001681        |.  83F8 64       cmp     eax, 64  
10001684        |.  74 12         je      short 10001698  
  
//判断是不是 90%以上一样呢？  
10001693        |.  83F8 5A       cmp     eax, 5A  
10001696        |.^ 72 EE         jb      short 10001686  
  
  
看了上面的肯定觉得很乱，我这里总结一下，具体意思如下：  
  
比如现在有某个壳的特征：  
  
002143A4  60 E8 00 00 00 00 5D B9 01 01 01 01 80 31 15 41  \`?...\]?€1A  
                                  -----------  
002143B4  81 F9                                            侚.  
  
反汇编为：  
  
002143A4          60              pushad  
002143A5          E8 00000000     call    002143AA  
002143AA          5D              pop     ebp  
002143AB          B9 01010101     mov     ecx, 1010101      // 这里的 1010101 是不计入最终计算的  
002143B0          8031 15         xor     byte ptr \[ecx\], 15  
002143B3          41              inc     ecx  
002143B4          81F9 00000000   cmp     ecx, 0  
  
特征数据长度为 0x12，其中的 01 是允许变得，也就是不算为特征。  
  
现在考虑识别时候。假如现在某个文件 OEP 前 0x12 个字节为：  
  
002143A4  60 E8 02 00 00 00 5D B9 01 01 01 01 80 31 15 51  \`?...\]?€1Q  
002143B4  81 F9                                            侚.  
  
将这段内容和上面的特征比较以下，有下面结果：  
  
1.总长度 0x12，有 4 个 0x01，所以有效长度为 0x12 - 0x04 = 0x0E  
2.除去 4 个 0x01，以及不一样的字节，总共有 0x0C 个字节内容一样  
3. 0x0C / 0x0E \* 0x64 = 0x55 ，说白了就是计算以下百分比，这里是 85%  
4.0x55 < 0x5A ，so，这个文件不属于这个特征码  
  
这里的 0x5A 就是 90%，如果有 90%以上的内容一样，就判定为符合该特征了。  

 4dd466d0e1e0cc123afbe504da2462ca