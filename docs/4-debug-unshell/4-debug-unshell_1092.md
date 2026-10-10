

# Themida 1.2.0.1 全过程分析

> 原文：[`www.pediy.com/kssd/pediy12/132522.html`](https://www.pediy.com/kssd/pediy12/132522.html)

【作者】  
  
by hkfans   --> 转载请注明作者，谢谢!  
  
【脱后感想】  
总共花了 3 个多月的业余时间来分析 themida.. 到现在我大概只剩半条命了。。。 真 TMD 的累! 脱壳不是人干的活啊。。眼睛快瞎了。精神也有点不大正常。。哎。。累! 再次膜拜写壳的人!  
  
【感谢】  
仅以此文感谢偶像 softworm，要不是他的天书【真的是天书，分析的太透彻了】，我现在估计还是苦苦挣扎在 VM 的汪洋大海之中..  
  
【思路】  
脱壳脱到最后发现其实就是做 4 件事，anti 解码 IAT OEP 到达 (大牛都参透了这个道理了 :) 最后一个算不算一件事情啊？^\_^)  
所以下面就按照这四个部分来分析 themida..  
  
【附件】  
本篇文章, NOTEPAD 样本 以及简单的 IAT 还原的脚本  
  
tmd1.2.0.1.rar  
  
【其他】  
代码太多了。我就不全部贴出来了。。只贴出这四部分的内容， 还有很多的准备工作的代码我就不贴出出来了。。  
  
准备工作包括:   
1. 一开始外壳分配内存来存储 VM 的 vmContext, oneByteOpcodeTable, twoByteOpcodeTable 以及所有 HANDLE 的代码.. themida 对 VirtualAlloc 的调用超级多 。。。  
  
2. 在 VM 用 LoadLibrary 加载 kernel32.dll  user32.dll advapi32.dll 在 VM 中获取 CreateFileA ReadFile CloseFile VirtualAlloc  
等 API 函数，然后利用 ReadFile 映射 kernel32.dll  user32.dll advapi32.dll, 这样以后基本上所有的 API 函数地址都是镜像地址，非真实的 API 函数地址..  
  
3. 创建用于解码外壳自身代码的线程，外壳经常是主线程和支线程交替解码，特别特别多的这样的代码。。真正有用的代码其实不多。大多数都是  
垃圾代码和这样解码外壳自身代码的代码。。  
  
【最后】  
可能有很多东西我漏掉分析。。有些地方自己还没有搞清楚。。。比如调用 ZwQuerySystemInformation 存储了一些信息之后，但是后面好像没有跟到利用这些信息的代码   
还有运行时间的检测，前面有看到保存时间，但后面又没有跟到检测时间的代码。。。<估计是 VM 跟丢了>  
  
下面进入正题:  
  
\=======================================================================================================================================================  
  
                【第一部分: anti】  
  
\[说明\]  
themida 1.2.0.1 的 anti-debug 的手段有: <好像还有很多我没有看到的样子。:)>  
  
OutputDebugStringA  ---> 针对 OD 的 BUG, 具体为什么我还没有细细研究..  
  
CreateFileA --> 检查是否使用 SoftIce  
  
FindWindowA ---> 查找是否有 Filemon RegMon 之类的软件  
  
【详细代码】  
  
1. 使用 OutputDebugStringA 对付存在 BUG 的 OD  
  
// 先获取 OutputDebugStringA  在镜像中的地址  
  

代码:

```
011D0555    8D85 7937AA09         lea     eax, dword ptr \[011D079C\]       
011D0688    8D85 711DAA09         lea     eax, dword ptr \[011CED94\]       
011D068E    0F8E 12000000         jle     011D06A6                        
011D0696    899D 451C9E09         mov     dword ptr \[0110EC68\], ebx       
011D06A6    FFD0                  call    eax                ; call  011CED94 调用函数获取 OutputDebugStringA 在镜像中的内存地址

```

  
// 然后在 VM 中调用 OutputDebugStringA    
  
代码:

```
0106   13 01    011D1A10         push      \[context.register\]
0107   07 02    011D19F4         mov       \[addr\_context.register\], context.esp
0108   23 01    011D1A1E         Sub       \[addr\_context.register\], 00000004
0109   30 01    011D19CA         pop       \[context.register\] (4 bytes)
0110   34 02    011D19E6         push      \[addr\_context.register\]
0111   2D 00    011D1A02         pop       \[addr\_context.esp\] (4 bytes)
0112   13 00    011D19D8         load      context.register, 07C61104
0113   36 00    011D1A2C         mov       \[addr\_context.register\], context.esp
0114   23 01    011D1A9C         Sub       \[addr\_context.register\], 00000004
0115   11 00    011D1A8E         push      \[addr\_context.register\]
0116   2D 00    011D1A56         pop       \[addr\_context.esp\] (4 bytes)
0117   20 00    011D1A72         push      011D1AB8
0118   39 01    011D1A3A         mov       \[addr\_context.register\], context.esp
0119   0A 02    011D1A80         pop       \[context.register\] (4 bytes)
0120   1A 01    011D1A64         mov       \[addr\_context.register\], context.ebp
0121   3B 00    011D1A48         Add       \[addr\_context.register\], 099E2B09
0122   11 02    011D1AAA         call      \[context.register\]          ; call 0098A14C (OutputDebugStringA)

```

// 堆栈  
0006FF94  011D1AB8  NOTEPAD\_.011D1AB8  
0006FF98  011D0C77  ASCII CR,LF,LF,LF,"------------------------------------------------",LF,CR,"---          Themida Professional            ---",LF,CR,"---      (c)2005 Oreans Technologies         ---",LF,CR,"------------------------------------------------",CR,LF...  
  
\================================================================================================================================  
  
2. 使用 FindWindowA 查找是否存在指定的软件  
  
011F6349  D0 DB F6 F3 2C 6F CE 95 46 69 6C 65 6D 6F 6E 43  雄鲶,oFilemonC  
011F6359  6C 61 73 73 00 46 69 6C 65 20 4D 6F 6E 69 74 6F  lass.File Monito  
011F6369  72 20 2D 20 53 79 73 69 6E 74 65 72 6E 61 6C 73  r - Sysinternals  
011F6379  3A 20 77 77 77 2E 73 79 73 69 6E 74 65 72 6E 61  : www.sysinterna  
011F6389  6C 73 2E 63 6F 6D 00 C8 61 86 47 74 9D 12 E3 00  ls.com.t??  
  
代码:

```
0037   00       011F70F4         push      00000000
0038   1A 01    011F713A         mov       \[addr\_context.register\], context.esp
0039   17 00    011F7102         Sub       \[addr\_context.register\], 00000004
0040   30 01    011F712C         pop       \[context.register\] (4 bytes)
0041   1F 01    011F7148         push      \[addr\_context.register\]
0042   0A 02    011F7110         pop       \[addr\_context.esp\] (4 bytes)      ; push 0

0043   13 00    011F711E         load      context.register, 07AA8B2C
0044   36 00    011F7156         mov       \[addr\_context.register\], context.ebp
0045   3B 00    011F7172         Add       \[addr\_context.register\], 09AC932E
0046   34 02    011F7164         push      \[addr\_context.register\]
0047   07 02    011F718E         mov       \[addr\_context.register\], addr\_context.eax
0048   14 02    011F7180         pop       \[context.register\] (4 bytes)      ; lea eax, dword ptr \[ebp+09AC932E\]

0049   36 00    011F719C         mov       \[addr\_context.register\], addr\_context.eax
0050   20 00    011F71D4         push      \[context.register\]
0051   39 01    011F71FE         mov       \[addr\_context.register\], context.esp
0052   08 00    011F71F0         Sub       \[addr\_context.register\], 00000004
0053   30 01    011F71C6         pop       \[context.register\] (4 bytes)
0054   32 01    011F71B8         push      \[addr\_context.register\]
0055   30 01    011F71E2         pop       \[addr\_context.esp\] (4 bytes)      ; push eax

0056   1C 02    011F71AA         load      context.register, 0A831D62
0057   36 00    011F720C         mov       \[addr\_context.register\], context.esp
0058   23 01    011F728A         Sub       \[addr\_context.register\], 00000004
0059   13 01    011F7244         push      \[addr\_context.register\]
0060   2D 00    011F7260         pop       \[addr\_context.esp\] (4 bytes)
0061   20 00    011F727C         push      011F7298
0062   39 01    011F7298         mov       \[addr\_context.register\], context.esp
0063   30 01    011F721A         pop       \[context.register\] (4 bytes)
0064   1A 01    011F7252         mov       \[addr\_context.register\], context.ebp    ; call 0A676E1 --> FindWindowA
0065   3B 00    011F7236         Add       \[addr\_context.register\], 099E0055  
0066   29 00    011F7228         call      \[context.register\]        ; call dword ptr \[ebp+099E0055\]

```

// 堆栈 ---> 返回窗口的句柄  
// 如果检测到的话就弹出错误  
A monitor program has been found running in your system.  
Please, unload it from memory and restart your program.  
  
0006FF70  011F6351  ASCII "Filemon Class"  -->  窗口的类名  
0006FF74  00000000  
  
代码:

```
0067   02       011F72A6         mov       \[addr\_context.register\], addr\_context.eax
0068   11 00    011F72B4         push      \[context.register\]
0069   07 02    011F72DE         mov       \[addr\_context.register\], addr\_context.eax
0070   13 01    011F72C2         push      \[context.register\]
0071   3B 01    011F72D0         Test      \[esp+4\], \[esp\] (4 bytes)      ; test eax, eax
0072   3A 00    011F72EC         jnz       011F7500  


0007   3C 02    011F72FA         mov       \[addr\_context.register\], context.ebp
0008   3B 00    011F7316         Add       \[addr\_context.register\], 09AC933B
0009   32 01    011F7340         push      \[addr\_context.register\]
0010   36 00    011F7324         mov       \[addr\_context.register\], addr\_context.eax
0011   30 01    011F7308         pop       \[context.register\] (4 bytes)      ; lea eax, dword ptr \[ebp+09AC933B\]

0012   1C 02    011F7332         load      context.register, 0AAE8C69
0013   1A 01    011F734E         mov       \[addr\_context.register\], addr\_context.eax
0014   11 00    011F7386         push      \[context.register\]
0015   1A 01    011F735C         mov       \[addr\_context.register\], context.esp
0016   08 00    011F7378         Sub       \[addr\_context.register\], 00000004
0017   2D 00    011F736A         pop       \[context.register\] (4 bytes)
0018   20 00    011F7394         push      \[addr\_context.register\]
0019   30 01    011F73A2         pop       \[addr\_context.esp\] (4 bytes)      ; push eax

0020   34 02    011F73B0         push      00000000
0021   07 02    011F73BE         mov       \[addr\_context.register\], context.esp
0022   23 01    011F73DA         Sub       \[addr\_context.register\], 00000004
0023   0A 02    011F73E8         pop       \[context.register\] (4 bytes)
0024   13 01    011F73CC         push      \[addr\_context.register\]
0025   14 02    011F73F6         pop       \[addr\_context.esp\] (4 bytes)      ; push 0

0026   07 02    011F7404         mov       \[addr\_context.register\], context.esp
0027   23 01    011F742E         Sub       \[addr\_context.register\], 00000004
0028   34 02    011F7412         push      \[addr\_context.register\]
0029   2D 00    011F744A         pop       \[addr\_context.esp\] (4 bytes)
0030   11 00    011F7466         push      011F7490
0031   07 02    011F7482         mov       \[addr\_context.register\], context.esp
0032   30 01    011F7458         pop       \[context.register\] (4 bytes)
0033   07 02    011F7474         mov       \[addr\_context.register\], context.ebp
0034   3B 00    011F7420         Add       \[addr\_context.register\], 099E0055
0035   25 00    011F743C         call      \[context.register\]        ; call FindWindowA

```

  
// 堆栈  
0006FF70  00000000  
0006FF74  011F635E  ASCII "File Monitor - Sysinternals: www.sysinternals.com"  
  
\---------------------------------------------------------------------  
  
011F860B  20 AB A6 C3 1C FF BE 65 52 65 67 6D 6F 6E 43 6C   ?RegmonCl  
011F861B  61 73 73 00 52 65 67 69 73 74 72 79 20 4D 6F 6E  ass.Registry Mon  
011F862B  69 74 6F 72 20 2D 20 53 79 73 69 6E 74 65 72 6E  itor - Sysintern  
011F863B  61 6C 73 3A 20 77 77 77 2E 73 79 73 69 6E 74 65  als: www.sysinte  
011F864B  72 6E 61 6C 73 2E 63 6F 6D 00 F8 D1 36 37 A4 0D  rnals.com.67?  
  
代码:

```
0112   00       011F939D         push      00000000
0113   36 00    011F93C7         mov       \[addr\_context.register\], context.esp
0114   23 01    011F93AB         Sub       \[addr\_context.register\], 00000004
0115   30 01    011F93D5         pop       \[context.register\] (4 bytes)
0116   20 00    011F93B9         push      \[addr\_context.register\]
0117   14 02    011F93E3         pop       \[addr\_context.esp\] (4 bytes)
0118   39 01    011F93F1         mov       \[addr\_context.register\], context.ebp
0119   35 01    011F941B         Add       \[addr\_context.register\], 09ACB5F0
0120   11 00    011F940D         push      \[addr\_context.register\]
0121   1A 01    011F9437         mov       \[addr\_context.register\], addr\_context.eax
0122   0A 02    011F9429         pop       \[context.register\] (4 bytes)
0123   13 00    011F93FF         load      context.register, 006D7A4E
0124   39 01    011F9445         mov       \[addr\_context.register\], addr\_context.eax
0125   32 01    011F9453         push      \[context.register\]
0126   39 01    011F946F         mov       \[addr\_context.register\], context.esp
0127   23 01    011F9461         Sub       \[addr\_context.register\], 00000004
0128   14 02    011F9499         pop       \[context.register\] (4 bytes)
0129   13 01    011F948B         push      \[addr\_context.register\]
0130   2D 00    011F947D         pop       \[addr\_context.esp\] (4 bytes)
0131   13 00    011F94A7         load      context.register, 0BA7DD87
0132   07 02    011F94B5         mov       \[addr\_context.register\], context.esp
0133   17 00    011F9533         Sub       \[addr\_context.register\], 00000004
0134   1F 01    011F94C3         push      \[addr\_context.register\]
0135   14 02    011F94ED         pop       \[addr\_context.esp\] (4 bytes)
0136   11 00    011F94DF         push      011F9541
0137   1A 01    011F94D1         mov       \[addr\_context.register\], context.esp
0138   0A 02    011F9517         pop       \[context.register\] (4 bytes)
0139   1A 01    011F9509         mov       \[addr\_context.register\], context.ebp
0140   3B 00    011F9525         Add       \[addr\_context.register\], 099E0055    
0141   29 00    011F94FB         call      \[context.register\]        ; call FindWindowA

```

// 堆栈  
0006FF70  011F8613  ASCII "RegmonClass"  
0006FF74  00000000  
  
  
代码:

```
0142   02       011F9541         mov       \[addr\_context.register\], addr\_context.eax
0143   34 02    011F954F         push      \[context.register\]
0144   39 01    011F9579         mov       \[addr\_context.register\], addr\_context.eax
0145   1F 01    011F956B         push      \[context.register\]
0146   3B 01    011F955D         Test      \[esp+4\], \[esp\] (4 bytes)
0147   38 00    011F9587         jnz       011F97B7


0007   3C 02    011F9595         mov       \[addr\_context.register\], context.ebp
0008   35 01    011F95BF         Add       \[addr\_context.register\], 09ACB5FC
0009   34 02    011F95DB         push      \[addr\_context.register\]
0010   36 00    011F95A3         mov       \[addr\_context.register\], addr\_context.eax
0011   0A 02    011F95CD         pop       \[context.register\] (4 bytes)
0012   1C 02    011F95B1         load      context.register, 09463DEA
0013   36 00    011F95E9         mov       \[addr\_context.register\], addr\_context.eax
0014   32 01    011F9613         push      \[context.register\]
0015   39 01    011F963D         mov       \[addr\_context.register\], context.esp
0016   23 01    011F9605         Sub       \[addr\_context.register\], 00000004
0017   2D 00    011F9621         pop       \[context.register\] (4 bytes)
0018   32 01    011F95F7         push      \[addr\_context.register\]
0019   2D 00    011F962F         pop       \[addr\_context.esp\] (4 bytes)
0020   13 00    011F964B         load      context.register, 0CA35115
0021   34 02    011F9659         push      00000000
0022   07 02    011F9675         mov       \[addr\_context.register\], context.esp
0023   23 01    011F9691         Sub       \[addr\_context.register\], 00000004
0024   30 01    011F969F         pop       \[context.register\] (4 bytes)
0025   32 01    011F9683         push      \[addr\_context.register\]
0026   0A 02    011F9667         pop       \[addr\_context.esp\] (4 bytes)
0027   1A 01    011F96AD         mov       \[addr\_context.register\], context.esp
0028   08 00    011F96C9         Sub       \[addr\_context.register\], 00000004
0029   20 00    011F96E5         push      \[addr\_context.register\]
0030   2D 00    011F9701         pop       \[addr\_context.esp\] (4 bytes)
0031   11 00    011F96BB         push      011F9739
0032   07 02    011F970F         mov       \[addr\_context.register\], context.esp
0033   30 01    011F9739         pop       \[context.register\] (4 bytes)
0034   36 00    011F96D7         mov       \[addr\_context.register\], context.ebp
0035   35 01    011F971D         Add       \[addr\_context.register\], 099E0055
0036   3A 00    011F972B         call      \[context.register\]        ; call FindWindowA

```

// 堆栈  
0006FF70  00000000  
0006FF74  011F861F  ASCII "Registry Monitor - Sysinternals: www.sysinternals.com"  
  
\================================================================================================================================  
  
3. 使用 CreateFileA 函数检测 SoftICE 调试软件  
  
//////////////////////////////////////////////////////////////////////////  
// 下面调用 CreateFileA 来检测 SoftICE 调试软件  
//  
011B367C  5C 5C 2E 5C 53 49 43 45 00 5C 5C 2E 5C 53 49 57  \\\\.\\SICE.\\\\.\\SIW  
011B368C  56 49 44 00 5C 5C 2E 5C 4E 54 49 43 45           VID.\\\\.\\NTICE  
  
// 堆栈  
0006FF80   011B38B3  /CALL 到 CreateFileA 来自 NOTEPAD\_.011B38B1  
0006FF84   011B367C  |FileName = "\\\\.\\SICE"  
0006FF88   C0000000  |Access = GENERIC\_READ|GENERIC\_WRITE  
0006FF8C   00000003  |ShareMode = FILE\_SHARE\_READ|FILE\_SHARE\_WRITE  
0006FF90   00000000  |pSecurity = NULL  
0006FF94   00000003  |Mode = OPEN\_EXISTING  
0006FF98   00000080  |Attributes = NORMAL  
0006FF9C   00000000  \\hTemplateFile = NULL  
  
  
代码:

```
011B38A8    891C24                mov     dword ptr \[esp\], ebx            ;
011B38AB    899D 4D029E09         mov     dword ptr \[0110D270\], ebx       
011B38B1    FFD0                  call    eax                             ; eax = 7C801A28 CreateFileA (这里返回 0xFFFFFFFF 表示不存在)


// 这里的乱序模式很常见...
011B38B3    6A 00                 push    0                               ; 
011B38B5    56                    push    esi                             ; 
011B38BE    BE BB381B01           mov     esi, 011B38BB                   
011B38BF    897424 04             mov     dword ptr \[esp+4\], esi          
011B38C3    814424 04 18000000    add     dword ptr \[esp+4\], 18           
011B38CB    46                    inc     esi                             
011B38CC    56                    push    esi                             ; 
011B38CD    C3                    retn                                    
011B38BC    5E                    pop     esi                             
011B38BD    C3                    retn                                    


//
011B38D9    8BD0                  mov     edx, eax                        
011B38DA    40                    inc     eax                             ; eax = -1 
011B38DB    0F85 18030000         jnz     011B3BF9                        ; 如果这里不等于 0 就表示检测到 SOFTICE 软件

// 这里的乱序模式很常见...
011B390D    6A 00                 push    0                               ; \[VM\_Context.PCode\_data\]  ---> PCode 数据地址
011B390F    57                    push    edi                             ; \[VM\_Context.PCode\_data\]  ---> PCode 数据地址
011B3918    BF 15391B01           mov     edi, 011B3915                   
011B3919    897C24 04             mov     dword ptr \[esp+4\], edi          
011B391D    814424 04 16000000    add     dword ptr \[esp+4\], 16           
011B3925    47                    inc     edi                             
011B3926    57                    push    edi                             ; \[VM\_Context.PCode\_data\]  ---> PCode 数据地址
011B3927    C3                    retn                                    
011B3916    5F                    pop     edi                             
011B3917    C3                    retn                                    


011B3A3B    66:81D2 701A          adc     dx, 1A70                        
011B3A40    0F014C24 FE           sidt    fword ptr \[esp-2\]               
011B3A45    5E                    pop     esi                             
011B3A46    89BD 55059E09         mov     dword ptr \[0110D578\], edi       
011B3A4C    5E                    pop     esi                             
011B3A4D    FFD0                  call    eax               ; eax = 7C801A28 CreateFileA (这里返回 0xFFFFFFFF 表示不存在)                    

// 堆栈
0006FF80   011B3A4F  /CALL 到 CreateFileA 来自 NOTEPAD\_.011B3A4D
0006FF84   011B3685  |FileName = "\\\\.\\SIWVID"
0006FF88   C0000000  |Access = GENERIC\_READ|GENERIC\_WRITE
0006FF8C   00000003  |ShareMode = FILE\_SHARE\_READ|FILE\_SHARE\_WRITE
0006FF90   00000000  |pSecurity = NULL
0006FF94   00000003  |Mode = OPEN\_EXISTING
0006FF98   00000080  |Attributes = NORMAL
0006FF9C   00000000  \\hTemplateFile = NULL

                                 
011B3A6B    8985 A1209E09         mov     dword ptr \[0110F0C4\], eax       
011B3A71    40                    inc     eax                             
011B3A72    0F85 81010000         jnz     011B3BF9                        ; 如果这里不等于 0 就表示检测到 SIWVID



011B3BAA    23B5 4D1B9E09         and     esi, dword ptr \[0110EB70\]       
011B3BB0    FFD0                  call    eax                            ; eax = 7C801A28 CreateFileA (这里返回 0xFFFFFFFF 表示不存在)     

// 堆栈
0006FF80   011B3BB2  /CALL 到 CreateFileA 来自 NOTEPAD\_.011B3BB0
0006FF84   011B3690  |FileName = "\\\\.\\NTICE"
0006FF88   C0000000  |Access = GENERIC\_READ|GENERIC\_WRITE
0006FF8C   00000003  |ShareMode = FILE\_SHARE\_READ|FILE\_SHARE\_WRITE
0006FF90   00000000  |pSecurity = NULL
0006FF94   00000003  |Mode = OPEN\_EXISTING
0006FF98   00000080  |Attributes = NORMAL
0006FF9C   00000000  \\hTemplateFile = NULL

011B3BB2    0995 7D0D9E09         or      dword ptr \[0110DDA0\], edx       
011B3BB8    40                    inc     eax                          ; 如果这里不等于 0 就表示检测到 NTICES
011B3BB9    0F85 3A000000         jnz     011B3BF9                     

```

  
\================================================================================================================================  
  
          【第二部分：解码】  
  
1. 先用 VirtualProtect 修改内存属性..  
  
01203A7D    8B85 C5239E09         mov     eax, dword ptr \[0110F3E8\]         
01203A83    8D8D 536AAD09         lea     ecx, dword ptr \[01203A76\]         
01203A89    8941 01               mov     dword ptr \[ecx+1\], eax            
01203A8C    8BC3                  mov     eax, ebx                          
01203A8E    C685 4D229E09 56      mov     byte ptr \[0110F270\], 56           
01203A95    68 DF30C5A9           push    A9C530DF                          
01203A9A    FFB5 55209E09         push    dword ptr \[0110F078\]              
01203AA0    8D85 711DAA09         lea     eax, dword ptr \[011CED94\]         
01203AA6    FFD0                  call    eax                             ; 获取 API 函数 VirtualProtect  eax = 00930ED4  
  
01203AA8    8D8D 99209E09         lea     ecx, dword ptr \[0110F0BC\]       ; eax = 00930ED4  
01203AAE    51                    push    ecx                               
01203AAF    6A 40                 push    40                                
01203AB1    FFB5 1D109E09         push    dword ptr \[0110E040\]              
01203AB7    FFB5 1D259E09         push    dword ptr \[0110F540\]              
01203ABD    FFD0                  call    eax                             ; 调用 VirtualProtect 修改内存访问属性  
  
// 堆栈  
0006FF28   01001000  |Address = NOTEPAD\_.01001000       ; 终于要开始解码被保护程序的代码了... shit  
0006FF2C   000034F8  |Size = 34F8 (13560.)  
0006FF30   00000040  |NewProtect = PAGE\_EXECUTE\_READWRITE    ; 修改的范围 010044F8 --> 01001000  
0006FF34   0110F0BC  \\pOldProtect = NOTEPAD\_.0110F0BC  
  
  
  
  
2. 解压.rsrc 资源段数据  
  
//  
// 这里调用 UPX 解压缩代码的函数 01114A45, 注意这里解压的是 .rsrc 资源段的数据的一部分  0100B000 --> 0100B568 范围的数据  
//  
//                    
代码:

```
01204AB9    8BD8                  mov     ebx, eax                        
01204ABB    8B95 E1229E09         mov     edx, dword ptr \[0110F304\]       
01204AC1    0395 81269E09         add     edx, dword ptr \[0110F6A4\]       ; edx = 0100B568
01204AC7    8BC2                  mov     eax, edx                        ; eax = edx

01204AC9    8B8D 49139E09         mov     ecx, dword ptr \[0110E36C\]       ; ecx = 0000400D
01204ACF    8330 25               xor     dword ptr \[eax\], 25             ; eax = 0100B568
01204AD2    40                    inc     eax                             
01204AD3    49                    dec     ecx                             
01204AD4    0F85 F5FFFFFF         jnz     01204ACF                        


01204ADA    53                    push    ebx                         ; ebx = 014D0000  (UPX 解压的目的地址)
01204ADB    52                    push    edx                           ; edx = 0100B568  (压缩后呆解压的数据)
01204ADC    8D85 227A9E09         lea     eax, dword ptr \[01114A45\]       
01204AE2    FFD0                  call    eax                            ; UPX 解压缩函数 


//
01204AE4    8BBD E1229E09         mov     edi, dword ptr \[0110F304\]       
01204AEA    03BD 81269E09         add     edi, dword ptr \[0110F6A4\]       ; edi = 0100B568

01204AF0    8BF3                  mov     esi, ebx                        
01204AF2    8BC8                  mov     ecx, eax                        
01204AF4    F3:A4                 rep     movs byte ptr es:\[edi\], byte ptr \[esi\]  ; esi = 014D0000  (复制解压缩后的数据回去)




01204AF6    C685 4D229E09 56      mov     byte ptr \[0110F270\], 56         
01204AFD    68 396D1FD4           push    D41F6D39                        
01204B02    FFB5 55209E09         push    dword ptr \[0110F078\]            
01204B08    8D85 711DAA09         lea     eax, dword ptr \[011CED94\]       
01204B0E    FFD0                  call    eax                             ; 查找 API 函数 VirtualFree

01204B10    68 00800000           push    8000                            
01204B15    6A 00                 push    0                               
01204B17    52                    push    edx                             
01204B18    FFD0                  call    eax                             ; 调用 VirtualFree 释放内存 --> 这里释放失败，因为那个内存不是用 VirtualAlloc 分配的内存, 搞什么鬼?

// 堆栈
0006FF28   0100B568  |Address = NOTEPAD\_.0100B568
0006FF2C   00000000  |Size = 0
0006FF30   00008000  \\FreeType = MEM\_RELEASE


01204B1A    8BBD 81269E09         mov     edi, dword ptr \[0110F6A4\]       ; edi = 01000000 模块地址
01204B20    037F 3C               add     edi, dword ptr \[edi+3C\]         ; PE 文件头

01204B23    8BBF 88000000         mov     edi, dword ptr \[edi+88\]         
01204B29    03BD 81269E09         add     edi, dword ptr \[0110F6A4\]       ; edi = 0100B000 (.rsrc 段的 RVA) 

01204B2F    8BB5 19049E09         mov     esi, dword ptr \[0110D43C\]       ; esi = 01203D5A
01204B35    B9 68050000           mov     ecx, 568                        
01204B3A    F3:A4                 rep     movs byte ptr es:\[edi\], byte ptr \[esi\]
01204B3C    B8 414B0000           mov     eax, 4B41                       
01204B41    8BC0                  mov     eax, eax                        
01204B43    83BD 15009E09 00      cmp     dword ptr \[0110D038\], 0         
01204B4A    75 09                 jnz     short 01204B55                  


3. 解码被保护的代码
//

// 获取 API 函数 VirtualAlloc 
//
01204BDF    8B9D 81269E09         mov     ebx, dword ptr \[0110F6A4\]       
01204BE5    C685 4D229E09 56      mov     byte ptr \[0110F270\], 56         
01204BEC    68 52B8A89C           push    9CA8B852                        
01204BF1    FFB5 55209E09         push    dword ptr \[0110F078\]            
01204BF7    8D85 711DAA09         lea     eax, dword ptr \[011CED94\]       
01204BFD    FFD0                  call    eax                             

01204BFF    6A 04                 push    4                               
01204C01    68 00100000           push    1000                            
01204C06    FFB5 891D9E09         push    dword ptr \[0110EDAC\]            
01204C0C    6A 00                 push    0                               
01204C0E    FFD0                  call    eax                             ; 先调用 VirtualAlloc 分配内存, 存放待解压缩的代码，后面会释放

0006FF24   00000000  |Address = NULL
0006FF28   0000A000  |Size = A000 (40960.)
0006FF2C   00001000  |AllocationType = MEM\_COMMIT
0006FF30   00000004  \\Protect = PAGE\_READWRITE

01204C10    85C0                  test    eax, eax                        ; eax = 014E0000
01204C12    0F85 0D000000         jnz     01204C25                        

01204C25    8BC8                  mov     ecx, eax                        
01204C27    8BC3                  mov     eax, ebx                        
01204C29    0340 3C               add     eax, dword ptr \[eax+3C\]        ; eax = 010000E0 PE 文件头    
01204C2C    05 F8000000           add     eax, 0F8                        
01204C31    8B50 0C               mov     edx, dword ptr \[eax+C\]          
01204C34    03D3                  add     edx, ebx                        ; edx = 01001000 .code 段
01204C36    83BD A5089E09 00      cmp     dword ptr \[0110D8C8\], 0         
01204C3D    0F84 0E000000         je      01204C51                        

//
// UPX 解压缩到申请的内存中
// 
01204C51    51                    push    ecx                             ; 前面申请的内存,   
01204C52    52                    push    edx                             ; 01001000 (这个是压缩后的被保护的代码)
01204C53    8D85 227A9E09         lea     eax, dword ptr \[01114A45\]       
01204C59    FFD0                  call    eax                             ; 调用 UPX 解压函数


// 堆栈
0006FF2C   01001000  NOTEPAD\_.01001000  --> 待解压的数据(源数据)
0006FF30   014E0000      --> 解压后存放的内存(目的地址)


//
// 拷贝回去被保护程序的区段，然后释放申请的内存...
//
01204C5B    8BFA                  mov     edi, edx                        
01204C5D    8BF1                  mov     esi, ecx                        
01204C5F    8BD1                  mov     edx, ecx                        
01204C61    8BC8                  mov     ecx, eax                        
01204C63    F3:A4                 rep     movs byte ptr es:\[edi\], byte ptr \[esi\]  ; esi = 014E0000  edi = 01001000 (赋值回代码)
01204C65    C685 4D229E09 56      mov     byte ptr \[0110F270\], 56         
01204C6C    68 396D1FD4           push    D41F6D39                        
01204C71    FFB5 55209E09         push    dword ptr \[0110F078\]            
01204C77    8D85 711DAA09         lea     eax, dword ptr \[011CED94\]       
01204C7D    FFD0                  call    eax                                 ; 查找 API 函数 VirtualFree, 释放内存，和前面一样

```

\================================================================================================================================  
            【第三部分：IAT 加载】  
  
IAT 的加载被分割成前后两部分:  
  
第一部分是在 VM 中加载被保护程序需要的动态库，然后将加载后得到的导入模块的基地址保存在一块申请的内存中  
  
第二部分才是获取导入函数的地址，然后加密 Kernel32.dll user32.dll advapi32.dll 三个模块的导入函数，加密一些特殊的函数如 ExitProcess, 最后修复  
被保护程序中调用这些 API 函数的指令... 如 FF15 FF25 类型的..  
  
  
1. 获取一些需要特殊处理的 API 函数的地址..  
  
代码:

```
0120BF5C    C785 AD0D9E09 00000000mov     dword ptr \[0110DDD0\], 0         
0120BF66    C685 4D229E09 45      mov     byte ptr \[0110F270\], 45         
0120BF6D    68 6969728E           push    8E726969                        
0120BF72    FFB5 812E9E09         push    dword ptr \[0110FEA4\]            
0120BF78    8D85 52369E09         lea     eax, dword ptr \[01110675\]       
0120BF7E    FFD0                  call    eax                           ; 查找 API 函数 ExitProcess = 7C81CB12   (这个是真实的模块地址，非镜像地址)   

0120BF80    8985 912E9E09         mov     dword ptr \[0110FEB4\], eax         ; eax = 7C81CB12    --> ExitProcess
0120BF86    C685 4D229E09 43      mov     byte ptr \[0110F270\], 43         
0120BF8D    68 5E6B679C           push    9C676B5E                        
0120BF92    FFB5 812E9E09         push    dword ptr \[0110FEA4\]            
0120BF98    8D85 52369E09         lea     eax, dword ptr \[01110675\]       
0120BF9E    FFD0                  call    eax                               ; 查找 API 函数 CreateThread

0120BFA0    8985 EDEDAD09         mov     dword ptr \[0120BE10\], eax         ; eax = 7C8106D7 --> CreateThread
0120BFA6    C685 4D229E09 54      mov     byte ptr \[0110F270\], 54         
0120BFAD    68 296862EE           push    EE626829                        
0120BFB2    FFB5 812E9E09         push    dword ptr \[0110FEA4\]            
0120BFB8    8D85 52369E09         lea     eax, dword ptr \[01110675\]       
0120BFBE    FFD0                  call    eax                             

0120BFC0    8985 F1EDAD09         mov     dword ptr \[0120BE14\], eax         ; eax = 7C81CB3B  --> TerminateThread
0120BFC6    C685 4D229E09 45      mov     byte ptr \[0110F270\], 45         
0120BFCD    68 C23860DA           push    DA6038C2                        
0120BFD2    FFB5 812E9E09         push    dword ptr \[0110FEA4\]            
0120BFD8    8D85 52369E09         lea     eax, dword ptr \[01110675\]       
0120BFDE    FFD0                  call    eax                             

0120BFE0    8985 F5EDAD09         mov     dword ptr \[0120BE18\], eax         ; eax = 7C80C0F8 --> ExitThread
0120BFE6    83BD 391A9E09 00      cmp     dword ptr \[0110EA5C\], 0         
0120BFED    0F85 45000000         jnz     0120C038                        


0120C038    C685 4D229E09 52      mov     byte ptr \[0110F270\], 52         
0120C03F    68 0DE44ABC           push    BC4AE40D                        
0120C044    FFB5 E51F9E09         push    dword ptr \[0110F008\]            
0120C04A    8D85 52369E09         lea     eax, dword ptr \[01110675\]       
0120C050    FFD0                  call    eax                             

0120C052    8985 F9EDAD09         mov     dword ptr \[0120BE1C\], eax         ; eax = ntdll.RtlEnterCriticalSection
0120C058    C685 4D229E09 52      mov     byte ptr \[0110F270\], 52         
0120C05F    68 484A4CA6           push    A64C4A48                        
0120C064    FFB5 E51F9E09         push    dword ptr \[0110F008\]            
0120C06A    8D85 52369E09         lea     eax, dword ptr \[01110675\]       
0120C070    FFD0                  call    eax                             

0120C072    8985 FDEDAD09         mov     dword ptr \[0120BE20\], eax         ; eax = 7C9210E0 ntdll.RtlLeaveCriticalSection
0120C078    C685 4D229E09 52      mov     byte ptr \[0110F270\], 52         
0120C07F    68 EAB442FD           push    FD42B4EA                        
0120C084    FFB5 812E9E09         push    dword ptr \[0110FEA4\]            
0120C08A    8D85 52369E09         lea     eax, dword ptr \[01110675\]       
0120C090    FFD0                  call    eax                             


0120C092    8985 E5199E09         mov     dword ptr \[0110EA08\], eax         ; eax = 7C801812 ReadFile
0120C098    C685 4D229E09 77      mov     byte ptr \[0110F270\], 77         
0120C09F    68 A3A897F3           push    F397A8A3                        
0120C0A4    FFB5 19119E09         push    dword ptr \[0110E13C\]            
0120C0AA    8D85 52369E09         lea     eax, dword ptr \[01110675\]       
0120C0B0    FFD0                  call    eax                             

0120C0B2    8985 89139E09         mov     dword ptr \[0110E3AC\], eax         ; eax = 77D1A8AD wsprintfA
0120C0B8    C685 4D229E09 52      mov     byte ptr \[0110F270\], 52         
0120C0BF    68 1731DD8A           push    8ADD3117                        
0120C0C4    FFB5 812E9E09         push    dword ptr \[0110FEA4\]            
0120C0CA    8D85 52369E09         lea     eax, dword ptr \[01110675\]       
0120C0D0    FFD0                  call    eax                             

0120C0D2    8985 BD089E09         mov     dword ptr \[0110D8E0\], eax         ; eax = 7C812AA9 RaiseException
0120C0D8    C685 4D229E09 56      mov     byte ptr \[0110F270\], 56         
0120C0DF    68 396D1FD4           push    D41F6D39                        
0120C0E4    FFB5 55209E09         push    dword ptr \[0110F078\]            
0120C0EA    8D85 711DAA09         lea     eax, dword ptr \[011CED94\]       
0120C0F0    FFD0                  call    eax                               ; 这里是查找镜像 API 函数 VirtualFree 地址

0120C0F2    8985 B91B9E09         mov     dword ptr \[0110EBDC\], eax         ; eax = 00938F84 VirtualFree
0120C0F8    C685 4D229E09 47      mov     byte ptr \[0110F270\], 47         
0120C0FF    68 7A10DC81           push    81DC107A                        
0120C104    FFB5 812E9E09         push    dword ptr \[0110FEA4\]            
0120C10A    8D85 52369E09         lea     eax, dword ptr \[01110675\]       
0120C110    FFD0                  call    eax                             

0120C112    8985 11059E09         mov     dword ptr \[0110D534\], eax         ; eax =7C80B56F GetModuleFileNameA
0120C118    C685 4D229E09 47      mov     byte ptr \[0110F270\], 47         
0120C11F    68 E2B50F8A           push    8A0FB5E2                        
0120C124    FFB5 812E9E09         push    dword ptr \[0110FEA4\]            
0120C12A    8D85 52369E09         lea     eax, dword ptr \[01110675\]       
0120C130    FFD0                  call    eax                             


0120C132    8985 1DEEAD09         mov     dword ptr \[0120BE40\], eax         ; EAX = 7C80AE40  GetProcAddress
0120C138    83BD 3D1D9E09 00      cmp     dword ptr \[0110ED60\], 0         
0120C13F    0F84 40040000         je      0120C585                        

```

2.  申请两块中间过渡的内存，用来存储 API 复制代码和经过乱序变形后的 API 函数代码  
  
代码:

```
0120C585    6A 04                 push    4                               
0120C587    68 00100000           push    1000                            
0120C58C    68 00100000           push    1000                            
0120C591    6A 00                 push    0                               
0120C593    FF95 41169E09         call    dword ptr \[0110E664\]              ; 申请内存 VirtualAlloc


0120C599    8985 B5259E09         mov     dword ptr \[0110F5D8\], eax         ; eax = 014E0000
0120C59F    8D85 E491AD09         lea     eax, dword ptr \[01206207\]       
0120C5A5    FFD0                  call    eax                             


0120C5A7    6A 04                 push    4                               
0120C5A9    68 00100000           push    1000                            
0120C5AE    68 00200000           push    2000                            
0120C5B3    6A 00                 push    0                               
0120C5B5    FF95 41169E09         call    dword ptr \[0110E664\]            ; 调用 VirtualAlloc 分配内存 (用于保存临时的 API 函数的复制代码)

0120C5BB    8985 89119E09         mov     dword ptr \[0110E1AC\], eax       
0120C5C1    8985 511A9E09         mov     dword ptr \[0110EA74\], eax       
0120C5C7    6A 40                 push    40                              
0120C5C9    68 00100000           push    1000                            
0120C5CE    68 00000100           push    10000                           
0120C5D3    6A 00                 push    0                               
0120C5D5    FF95 41169E09         call    dword ptr \[0110E664\]           ; 调用 VirtualAlloc 分配内存  (用于保存临时的 API 函数变形代码)

0120C5DB    8985 7D229E09         mov     dword ptr \[0110F2A0\], eax       

```

3. IAT 数据的结构大概如下 <不准确>  
//  
// 数据 -- 所有的 IAT 数据,结构如下  
//  
  
struct stTmdIAT  
{  
  DWORD apiNameHashVal;    //  导入 API 函数的名称的 HASH 值(未解密)  
  DWORD iatAddress;    //  该导入 API 函数的 IAT 地址 (未解密)  ===> 如果后面紧跟 AAAAAAAA 表示 FF25 jmp \[iat\]类型的。。  
  
  DWORD callApiInstructions\[n\];  //  被保护程序中调用该 API 函数的指令地址数组, 以 FFFFFFFF 结束  
  
};  
  
  
01204F0A  DE EF 5F 05 67 DE B5 5D AE 0C 00 C0 FF FF FF FF  揎\_g 薜\]?.?  
01204F1A  DD DD DD DD 80 98 AF 3A 67 DE B5 7D 00 06 00 C0  葺葺:g 薜}..  
01204F2A  FF FF FF FF DD DD DD DD 27 E6 3E FE 67 DE B5 9D  葺葺'?薜  
01204F3A  DC 0B 00 C0 FF FF FF FF DD DD DD DD EE A9 90 37  ?.?葺葺瞟?  
01204F4A  67 DE B5 BD 1B 06 00 40 FF FF FF FF DD DD DD DD  g 薜?.@葺葺  
01204F5A  FE 6E EE 41 67 DE B5 DD 61 05 00 C0 FF FF FF FF  g 薜.?  
01204F6A  DD DD DD DD 08 47 94 FA 67 DE B5 FD FF FF FF FF  葺葺 Gg 薜?  
01204F7A  DD DD DD DD 64 89 63 97 68 DE B5 1D F4 05 00 C0  葺葺 d 薜?.  
01204F8A  FF FF FF FF DD DD DD DD 24 F8 5B 5F 68 DE B5 3D  葺葺$\_h 薜=  
01204F9A  00 04 00 60 1A 05 00 00 FF FF FF FF DD DD DD DD  ..\`..葺葺  
01204FAA  4E F9 7C 55 68 DE B5 5D 98 05 00 00 FF FF FF FF  NUh 薜\]?..  
01204FBA  DD DD DD DD EE EE EE EE DD DD DD DD A9 D4 FC D2  葺葺铑铑葺葺┰  
01204FCA  5C DE B5 DD FF FF FF FF DD DD DD DD 55 EC EE 66  \\薜?葺葺 U 祛 f  
  
  
4. 循环处理每个导入库  
  
代码:

```
0120C5E1    8BB5 E51C9E09         mov     esi, dword ptr \[0110ED08\]       ; ESI = 01204F0A 
0120C5E7    8B9D 11249E09         mov     ebx, dword ptr \[0110F434\]       ; ebx = 00FE0000  (之前申请的用于保存被保护程序导出库的库基地址)

0120C5ED    89B5 612B9E09         mov     dword ptr \[0110FB84\], esi       
0120C5F3    899D 2D239E09         mov     dword ptr \[0110F350\], ebx  
     
0120C5F9    8B9D 11249E09         mov     ebx, dword ptr \[0110F434\]       ; ebx = 00FE0000
0120C5FF    8B0B                  mov     ecx, dword ptr \[ebx\]            ; ecx = 76320000 (comdlg32 模块基地址)
0120C601    83F9 00               cmp     ecx, 0                          
0120C604    0F84 8D090000         je      0120CF97                        ; 循环处理每个导入模块 (等于 0 表示处理完所有的导入库)

// 处理一个导入模块, 先将该模块的所有导出函数名称进行 HASH，保存 HASH 数值到申请的内存中

0120C60A    50                    push    eax                             ; eax = 01500000
0120C60B    51                    push    ecx                             ; 入栈模块基地址(comdlg32.76320000 )
0120C60C    60                    pushad                                  
0120C60D    33C0                  xor     eax, eax                        
0120C60F    8985 D1089E09         mov     dword ptr \[0110D8F4\], eax       

0120C615    BE 3C000000           mov     esi, 3C                         ; esi = 3C
0120C61A    037424 20             add     esi, dword ptr \[esp+20\]         ; esi = 7632003C

0120C61E    66:AD                 lods    word ptr \[esi\]                  
0120C620    034424 20             add     eax, dword ptr \[esp+20\]         ; "PE"文件头

0120C624    8B70 78               mov     esi, dword ptr \[eax+78\]         
0120C627    037424 20             add     esi, dword ptr \[esp+20\]         ; comdlg32 导出表的 RVA esi = 76322050

0120C62B    8B7E 18               mov     edi, dword ptr \[esi+18\]         ; edi = 1A 导出表的名称个数
0120C62E    89BD 8D029E09         mov     dword ptr \[0110D2B0\], edi       
0120C634    85FF                  test    edi, edi                        
0120C636    0F85 0A000000         jnz     0120C646                        ; 循环该库导出表的每个函数
\-------------------

//
// 申请内存，用于存放每个导出函数名的 HASH 值
//
0120C646    51                    push    ecx                             
0120C647    8BD7                  mov     edx, edi                        
0120C649    6BD2 04               imul    edx, edx, 4                     
0120C64C    8995 75009E09         mov     dword ptr \[0110D098\], edx       ; edx = 1A\*4 = 68

0120C652    6A 04                 push    4                               
0120C654    68 00100000           push    1000                            
0120C659    52                    push    edx                             
0120C65A    6A 00                 push    0                               
0120C65C    FF95 41169E09         call    dword ptr \[0110E664\]            ; VirtualAlloc 申请内存

// 堆栈
0006FEF8   00000000  |Address = NULL
0006FEFC   00000068  |Size = 68 (104.)
0006FF00   00001000  |AllocationType = MEM\_COMMIT
0006FF04   00000004  \\Protect = PAGE\_READWRITE


//----------------------------------------------
// 下面先获取导出函数名称字符串的字符个数，然后对每个导出函数进行查表 HASH 算法, 保存 HASH 的 DWORD 值到前面申请的内存中 。。
// 循环处理开始

0120C670    56                    push    esi                             
0120C671    AD                    lods    dword ptr \[esi\]                 
0120C672    034424 24             add     eax, dword ptr \[esp+24\]        ; eax = 7632218D "ChooseColorA"---> 每个输出函数名称字符串 
0120C676    97                    xchg    eax, edi                        
0120C677    8BDF                  mov     ebx, edi                        
0120C679    57                    push    edi                             
0120C67A    32C0                  xor     al, al                          
0120C67C    AE                    scas    byte ptr es:\[edi\]               ; 计算函数名称字符串的个数
0120C67D    0F85 F9FFFFFF         jnz     0120C67C                        
0120C683    5E                    pop     esi                             

0120C684    2BFB                  sub     edi, ebx                        ; edi = 0D (14 个字符)
0120C686    52                    push    edx                             
0120C687    8BD7                  mov     edx, edi                        ; edx = 0D
0120C689    8BBD B5259E09         mov     edi, dword ptr \[0110F5D8\]       ; edi = 014E0000 (前面申请的内存,存放解码用的数据表)
0120C68F    83C9 FF               or      ecx, FFFFFFFF                   ; ecx = FFFFFFFF


0120C692    33C0                  xor     eax, eax                        ; eax = 0
0120C694    8A06                  mov     al, byte ptr \[esi\]              ; 字符'C'
0120C696    32C1                  xor     al, cl                          ; xor al, FF
0120C698    46                    inc     esi                             
0120C699    8B0487                mov     eax, dword ptr \[edi+eax\*4\]      ; 查表 014E0000 --> 得到解码 KEY
0120C69C    C1E9 08               shr     ecx, 8                          
0120C69F    33C8                  xor     ecx, eax                        
0120C6A1    4A                    dec     edx                             ; 字符个数减去 1
0120C6A2    0F85 EAFFFFFF         jnz     0120C692                        ; 继续下一个字符

0120C6A8    8BC1                  mov     eax, ecx                        
0120C6AA    F7D0                  not     eax                             ; eax = 9E59A0D5
0120C6AC    5A                    pop     edx                             
0120C6AD    8902                  mov     dword ptr \[edx\], eax            ; edx = 01510000 (前面申请的用于保存一个导出表中所有导出函数名称字符串的 HASH 数值)

0120C6AF    83C2 04               add     edx, 4                          
0120C6B2    52                    push    edx                             
0120C6B3    FF85 D1089E09         inc     dword ptr \[0110D8F4\]            ; 已 HASH 的函数个数
0120C6B9    8B95 D1089E09         mov     edx, dword ptr \[0110D8F4\]       
0120C6BF    3995 8D029E09         cmp     dword ptr \[0110D2B0\], edx       ;  \[0110D2B0\] = 1A (总的导出函数名称个数 = 1A)
0120C6C5    0F84 0A000000         je      0120C6D5                        

0120C6CB    5A                    pop     edx                             
0120C6CC    5E                    pop     esi                             
0120C6CD    83C6 04               add     esi, 4   
                       
// 循环处理结束 --> HASH 完该导出库的所有 API 函数名称字符串.. 执行到这里


//-----------------------------------------------------
// 这里开始获取并计算要查找的导入函数的 HASH 值
//

0120C778    B9 D9454449           mov     ecx, 494445D9                     ; 硬编码的解码 KEY1
0120C77D    BA 75518541           mov     edx, 41855175                     ; 硬编码的解码 KEY2

0120C782    AD                    lods    dword ptr \[esi\]                   ; esi = 01204F0A
0120CE4E    C746 FC 0000000>      mov     dword ptr \[esi-4\], 0      ; 读取然后清 0  
0120C783    89B5 612B9E09         mov     dword ptr \[0110FB84\], esi         ; eax = 055FEFDE (原始数据，计算后得到导入函数名称 HASH 值)
      ; 
0120C790    3D EEEEEEEE           cmp     eax, EEEEEEEE                   
0120C795    0F85 20000000         jnz     0120C7BB                          ; 是否等于 EEEEEEEE                   

{-------------------
0120C79B    813E DDDDDDDD         cmp     dword ptr \[esi\], DDDDDDDD       
0120C7A1    0F85 14000000         jnz     0120C7BB     

0120C7A7    C706 00000000         mov     dword ptr \[esi\], 0                ; 如果是 EEEEEEEE 后面跟着 DDDDDDDD 的话
0120C7AD    83C6 04               add     esi, 4                            ; 表示
0120C7B0    89B5 612B9E09         mov     dword ptr \[0110FB84\], esi       
0120CF62    89B5 612B9E09         mov     dword ptr \[0110FB84\], esi       
0120CF68    52                    push    edx                             
0120CF69    68 00800000           push    8000                            
0120CF6E    6A 00                 push    0                               
0120CF70    FFB5 BD259E09         push    dword ptr \[0110F5E0\]            
0120CF76    FF95 B91B9E09         call    dword ptr \[0110EBDC\]            
 }


//
// 解码导入函数的 HASH 值
//
0120C7BB    8BD8                  mov     ebx, eax                          ; ebx = eax = 055FEFDE 
0120C7BD    3385 AD0D9E09         xor     eax, dword ptr \[0110DDD0\]       
0120C7C3    C1C8 03               ror     eax, 3                          
0120C7C6    2BC2                  sub     eax, edx                        
0120C7C8    C1C0 10               rol     eax, 10                         
0120C7CB    33C1                  xor     eax, ecx                        
0120C7CD    899D AD0D9E09         mov     dword ptr \[0110DDD0\], ebx       
0120C7D3    3D 00000100           cmp     eax, 10000                      
0120C7D8    0F83 45000000         jnb     0120C823                          ; eax =  E5C23AFF (最终的导入函数名称的 HASH 值)

\-----------------------
// 下面循环查找 HASH 值相等的库导出函数

0120C823    51                    push    ecx                               ; 保存解码 KEY1
0120C824    52                    push    edx                               ; 保存解码 KEY2

0120C825    33C9                  xor     ecx, ecx                        
0120C827    8B95 BD259E09         mov     edx, dword ptr \[0110F5E0\]       
0120C82D    3B02                  cmp     eax, dword ptr \[edx\]              ; eax = E5C23AFF  edx = 01510000
0120C82F    0F84 38000000         je      0120C86D                          ; 注意: 这里的 EAX 就是真正要找的被保护程序的导入函数名的 HASH 值

0120C835    83C2 04               add     edx, 4                          
0120C838    41                    inc     ecx                             
0120C839    3B8D 8D029E09         cmp     ecx, dword ptr \[0110D2B0\]         ; \[0110D2B0\]  = 1A (comdlg32 模块导出函数个数)
0120C83F    0F85 E8FFFFFF         jnz     0120C82D                        

// 跳到这里，表示找到需要的导入函数, 保存名称表的索引值
            ; 这里是导入函数 PageSetupDlgW
0120C86D    898D D1089E09         mov     dword ptr \[0110D8F4\], ecx         ; ecx = 0F ( 表示导出库的第 15 个导出函数是被保护程序需要的),序数是从 1 开始的，所以这里是

0120C873    5A                    pop     edx                               ; 弹出解码 KEY2
0120C874    59                    pop     ecx                               ; 弹出解码 KEY1
\--------------------------


0120C875    56                    push    esi                             
0120C876    8B9D 11249E09         mov     ebx, dword ptr \[0110F434\]       
0120C87C    8B0B                  mov     ecx, dword ptr \[ebx\]              ;  ebx = 00FE0000  (之前申请的用于保存被保护程序导入库的库基地址)

0120C87E    8B85 D1089E09         mov     eax, dword ptr \[0110D8F4\]         ; eax = 0F (前面比较得到的导入函数索引值)
0120C884    D1E0                  shl     eax, 1                            
0120C886    0385 21009E09         add     eax, dword ptr \[0110D044\]         ; eax = 7632216A

0120C88C    33F6                  xor     esi, esi      
0120C88E    96                  xchg    eax, esi        ; 
                  
//
// 0120C89C    最关键的代码地址，拦截这里可以看到导入函数地址 -->  eax = 76344906 comdlg32.PageSetupDlgW
//
0120C891    C1E0 02               shl     eax, 2                          
0120C894    0385 112A9E09         add     eax, dword ptr \[0110FA34\]         ; eax = 763220B8
0120C89A    96                    xchg    eax, esi                        
0120C89B    AD                    lods    dword ptr \[esi\]                 
0120C89C    03C1                  add     eax, ecx                 ; 最关键的代码 eax = 76344906 comdlg32.PageSetupDlgW
             
0120C89E    5E                    pop     esi                             
0120C89F    83BD 952A9E09 01      cmp     dword ptr \[0110FAB8\], 1         
0120C8A6    0F84 39000000         je      0120C8E5                        

//
// 比较当前模块，如果是 Kernel32.dll User32.dll Advapi32.dll 3 个导入库的话，就进行特殊的处理
//
0120C8AC    3B8D 812E9E09         cmp     ecx, dword ptr \[0110FEA4\]         ; \[0110FEA4\] = Kernel32.7C80000 
0120C8B2    0F84 2D000000         je      0120C8E5                        

0120C8B8    3B8D 19119E09         cmp     ecx, dword ptr \[0110E13C\]         ; \[0110E13C\] = User32.77D10000
0120C8BE    0F84 21000000         je      0120C8E5           

0120C8C4    3B8D BD2A9E09         cmp     ecx, dword ptr \[0110FAE0\]         ; \[0110FAE0\] = ADVAPI32.77DA0000
0120C8CA    0F84 15000000         je      0120C8E5                        
             

// 其他的模块就走到这里 (不是 Kernel32.dll User32.dll Advapi32.dll 3 个导入库的话)

0120C8D0    8D9D 79FFAD09         lea     ebx, dword ptr \[0120CF9C\]       
0120C8D6    FFD3                  call    ebx                             


 
//
// 下面是获取 API 函数的 IAT 地址... (保存在哪里) 取值是接下来的 4 个字节, 经过简单的计算再加上被保护程序的基地址 01000000
//
// 最关键的代码 0120CE69    8908                  mov     dword ptr \[eax\], ecx  (拦截这里可以看到 API 函数地址和该 API 函数对应的 IAT 地址)
//

0120C8D8    8BF8                  mov     edi, eax                        
0120C8DA    8985 6D109E09         mov     dword ptr \[0110E090\], eax         ; eax = 76344906 comdlg32.PageSetupDlgW
0120CE47    8BB5 612B9E09         mov     esi, dword ptr \[0110FB84\]       
0120CE4D    AD                    lods    dword ptr \[esi\]                   ; eax = 5DB5DE67
0120CE4E    C746 FC 0000000>      mov     dword ptr \[esi-4\], 0      ; 读取然后清 0

0120CE55    C1C0 05               rol     eax, 5                          
0120CE58    05 D9454449           add     eax, 494445D9                   
0120CE5D    0385 81269E09         add     eax, dword ptr \[0110F6A4\]         ; eax = 010012C4 (API 函数的 IAT 的地址)
0120CE63    8B8D 6D109E09         mov     ecx, dword ptr \[0110E090\]       ; ecx = 76344906 comdlg32.PageSetupDlgW
  
0120CE69    8908                  mov     dword ptr \[eax\], ecx            ; 最关键的代码
    
//------------------------------------------------------------------------------------------------
// 这里循环处理被保护程序调用该 API 函数的指令，之前都是 NOP 填充掉的。。现在修改为 CALL 调用到 API 函数
//

// 循环开始
//
0120CE6B    AD                    lods    dword ptr \[esi\]                 
0120CE6C    C746 FC 00000000      mov     dword ptr \[esi-4\], 0            
      
0120CE73    89B5 612B9E09         mov     dword ptr \[0110FB84\], esi       
0120CE79    83F8 FF               cmp     eax, -1                         
0120CE7C    0F85 20000000         jnz     0120CEA2                          ; 比较是否等于 -1 ( 循环被保护程序所有的对该 API 调用的指令地址)


0120CEA2    C1C0 03               rol     eax, 3                          
0120CEA5    0385 81269E09         add     eax, dword ptr \[0110F6A4\]         ; eax = 01006576

{
// 被保护的该地址是 调用该 IAT 指向的 API 函数... 
01006576  |.  FF15 C4120001 call    dword ptr \[<&comdlg32.PageSetupDlgW>\]    ;  comdlg32.PageSetupDlgW
}

0120CEAB    83BD 71219E09 01      cmp     dword ptr \[0110F194\], 1           ; 这个比较是干什么用的?
0120CEB2    0F84 9D000000         je      0120CF55                        

0120CEB8    813E AAAAAAAA         cmp     dword ptr \[esi\], AAAAAAAA         ; esi = 结构 stTmdIAT 指针, 如果不是 FFFFFFF,看是否为 AAAAAAAA       
0120CEBE    0F85 12000000         jnz     0120CED6                        

// 如果是 AAAAAAAA --> 表示用 jmp 修复调用 API 函数的指令
0120CEC4    83C6 04               add     esi, 4                          
0120CEC7    C746 FC 00000000      mov     dword ptr \[0120503A\], 0         
0120CECE    97                    xchg    eax, edi                        
0120CECF    B0 E9                 mov     al, 0E9                    ; AL = E9         
               
//------------ 这种类型的修复 FF25 类型的
0100737A   $- FF25 B8120001 jmp     dword ptr \[<&WINSPOOL.ClosePrinter>\]        ;  WINSPOOL.ClosePrinter
//--------------


// 如果不是是 AAAAAAAA ---> 表示用 CALL 修复调用 API 函数的指令  --> FF15 类型的

0120CED6    97                    xchg    eax, edi                        
0120CED7    B0 E8                 mov     al, 0E8                           ; eax 最后一个字节变 E8 --CALL
0120CED9    50                    push    eax                             
0120CEDA    83BD 952A9E09 01      cmp     dword ptr \[0110FAB8\], 1         
0120CEE1    0F84 3E000000         je      0120CF25                        


0120CEE7    B8 00010000           mov     eax, 100                        
0120CEEC    83BD 05EEAD09 00      cmp     dword ptr \[0120BE28\], 0         
0120CEF3    0F84 08000000         je      0120CF01                        

0120CEF9    8D9D 6743AD09         lea     ebx, dword ptr \[0120138A\]       
0120CEFF    FFD3                  call    ebx                               ; 这里调用？ 返回 8B


0120CF01    803F 90               cmp     byte ptr \[edi\], 90                ; edi = 01006576 (TMD 已经将 API 函数的调用的指令用 NOP 填充了)

{
// 

}

0120CF04    0F84 08000000         je      0120CF12                          ; 

0120CF12    83F8 50               cmp     eax, 50                         ; eax = 60 (这里的分支干什么的?)
0120CF15    0F82 0A000000         jb      0120CF25                        

0120CF1B    B0 90                 mov     al, 90                          
0120CF1D    AA                    stos    byte ptr es:\[edi\]                 ; 第一个字节还是 0x90
0120CF1E    58                    pop     eax                               ; eax 最后一个字节变 E8 --CALL

0120CF1F    AA                    stos    byte ptr es:\[edi\]                 ; EAX = 763449E8 (最后一个字节是 E8 CALL)
0120CF49    8B85 6D109E09         mov     eax, dword ptr \[0110E090\]         ; EAX = 76344906 (PageSetupDlgW
0120CF4F    2BC7                  sub     eax, edi                          ; 
0120CF51    83E8 04               sub     eax, 4                            ; 这里计算 CALL 到 API 函数的偏移地址... 
0120CF54    AB                    stos    dword ptr es:\[edi\]                ; CALL PageSetupDlgW


0120CF55    AD                    lods    dword ptr \[esi\]                   ; esi = 结构 stTmdIAT 指针 
0120CF56    C746 FC 00000000      mov     dword ptr \[01204F12\], 0         
0120CE73    89B5 612B9E09         mov     dword ptr \[0110FB84\], esi       
0120CE79    83F8 FF               cmp     eax, -1                           ; 循环处理每条调用该 API 函数的指令
0120CE7C    0F85 20000000         jnz     0120CEA2                          ; 如果是 FFFFFFFF 表示结束  < 跳到前面继续处理下一条调用该 API 函数的指令>

// 循环结束
//
//-----------------------------------------------------------------------
0120CE82    813E DDDDDDDD         cmp     dword ptr \[esi\], DDDDDDDD         ; 是否为 DDDDDDDD (什么意思?结束?)    
0120CE88    0F85 14000000         jnz     0120CEA2                        

0120CE8E    C706 00000000         mov     dword ptr \[esi\], 0              
0120CE94    83C6 04               add     esi, 4                          
0120CE97    89B5 612B9E09         mov     dword ptr \[0110FB84\], esi       
0120C6DA    C785 39169E09 00000000mov     dword ptr \[0110E65C\], 0         
0120C6E4    C785 2D259E09 00000000mov     dword ptr \[0110F550\], 0         
0120C6EE    83BD 05EEAD09 00      cmp     dword ptr \[0120BE28\], 0         
0120C6F5    0F84 08000000         je      0120C703                        

0120C6FB    8D9D 483AAD09         lea     ebx, dword ptr \[01200A6B\]         ; 调用函数 01200A6B ( 清 0)
0120C701    FFD3                  call    ebx                             

0120C703    FF85 BD029E09         inc     dword ptr \[0110D2E0\]              ; 这个什么计数器?
0120C709    83BD BD029E09 64      cmp     dword ptr \[0110D2E0\], 64        
0120C710    0F82 62000000         jb      0120C778                        

// 跳到前面继续循环

0120C778    B9 D9454449           mov     ecx, 494445D9                      ; 硬编码的解码 KEY1
0120C77D    BA 75518541           mov     edx, 41855175                   
0120C782    AD                    lods    dword ptr \[esi\]                 
0120C783    89B5 612B9E09         mov     dword ptr \[0110FB84\], esi       
0120C789    C746 FC 00000000      mov     dword ptr \[01204F1A\], 0         
0120C790    3D EEEEEEEE           cmp     eax, EEEEEEEE                     ;  是否为 EEEEEEEE
0120C795    0F85 20000000         jnz     0120C7BB                        

```

\================================================================================================================================  
  
          【第四部分：OEP 到达】  
  
说明：OEP 到达前有一段变态的 变形+VM 的代码。。太恐怖了。。暂时没有去看。。。 有些地方分析到我也是糊里糊涂的。呵呵  
  
  
1. 进入 OEP 前的一些工作  
  
//  
// 下面修改第一个区段为可读可执行的代码 (修改区段标志)  
//      
01213149    8BBD 81269E09         mov     edi, dword ptr \[0110F6A4\]         ; edi = 01000000  
0121314F    0F82 05000000         jb      0121315A                          
  
0121315A    037F 3C               add     edi, dword ptr \[edi+3C\]           ; edi = 010000E0 PE 文件头         
01213163    81C7 F8000000         add     edi, 0F8                          ; edi = 010001D8 (第一个区段结构信息)  
  
  
\==> 原本区段是 C0000040 ,即不是可执行的..  
  
01000224    400000C0    DD C0000040          ;  Characteristics = INITIALIZED\_DATA|READ|WRITE  
  
\==> 这里修改为 60000020 , 代码可取可执行  
010001FC    20000060    DD 60000020          ;  Characteristics = CODE|EXECUTE|READ  
  
  
  
01213189    C747 24 20000060      mov     dword ptr \[edi+24\], 60000020      ; 修改区段的标志信息 C000040 --> 60000020                   
012131A5    58                    pop     eax                               ; eax = 00930ED4  
  
012131A6    81D2 7F7F736B         adc     edx, 6B737F7F                     
012131AC    8D8D 3D129E09         lea     ecx, dword ptr \[0110E260\]         
012131C9    51                    push    ecx                               
012131F0    FC                    cld                                       
012131F1    0FBFF2                movsx   esi, dx                           
012131F4    FFB5 99209E09         push    dword ptr \[0110F0BC\]              
0121322D    68 D60468A0           push    A06804D6                          
01213232    66:8BF1               mov     si, cx                            
01213235    810424 2A0B985F       add     dword ptr \[esp\], 5F980B2A         
01213250    0F85 02000000         jnz     01213258                          
01213258    FFB5 81269E09         push    dword ptr \[0110F6A4\]              
0121325E    89B5 DD139E09         mov     dword ptr \[0110E400\], esi         
01213264    898D AD109E09         mov     dword ptr \[0110E0D0\], ecx         
0121326A    FFD0                  call    eax                               ; 调用 VirtualProtect 修改 PE 文件段  
  
  
// 堆栈  
0006FF24   01000000  |Address = NOTEPAD\_.01000000  
0006FF28   00001000  |Size = 1000 (4096.)  
0006FF2C   00000002  |NewProtect = PAGE\_READONLY  
0006FF30   0110E260  \\pOldProtect = NOTEPAD\_.0110E260  
  
  
2. 经过一大堆的乱序变形之后就糊里糊涂来到这里了。。最后一个 Retn 就进入了 VM-OEP  
  
012136E3    8B85 09109E09         mov     eax, dword ptr \[0110E02C\]         
012136F4    C600 00               mov     byte ptr \[eax\], 0                 
  
  
012139F5    8DB5 9065AE09         lea     esi, dword ptr \[012135B3\]         
01213B40    F3:AA                 rep     stos byte ptr es:\[edi\]            
  
01213CBD    C3                    retn                                  ; 从这里返回到 VM   (这里进入的 VM 是被保护程序 OEP 处的 VM)...     
  
  
  
3. VM-OEP 伪指令代码  
0120AEB4    68 4E30CD46     push    46CD304E  
0120AEB9  ^ E9 39CAFFFF     jmp     012078F7  
0120AEBE    68 C1A26510     push    1065A2C1    ==> call 01007568  
0120AEC3  ^ E9 2FCAFFFF     jmp     012078F7  
0120AEC8    68 32EE2A31     push    312AEE32    ==> call edi  
0120AECD  ^ E9 25CAFFFF     jmp     012078F7  
  
//  
// 下面是被 VM 掉的 OEP， 对比记事本的入口代码，完全一样。。。:)  
//  
代码:

```
0634   09 02    0120AA46         push      00000070
0635   3C 02    0120AA8C         mov       \[addr\_context.register\], context.esp
0636   16 01    0120AA70         Sub       \[addr\_context.register\], 00000004
0637   3A 02    0120AA9A         pop       \[context.register\] (4 bytes)
0638   09 02    0120AA54         push      \[addr\_context.register\]
0639   3A 02    0120AA62         pop       \[addr\_context.esp\] (4 bytes)      ; push 70

0640   27 02    0120AA7E         load      context.register, 060AD197
0641   09 02    0120AAA8         push      01001898
0642   3C 02    0120AAD2         mov       \[addr\_context.register\], context.esp
0643   16 01    0120AAB6         Sub       \[addr\_context.register\], 00000004
0644   3A 02    0120AAE0         pop       \[context.register\] (4 bytes)
0645   09 02    0120AAC4         push      \[addr\_context.register\]
0646   3A 02    0120AAEE         pop       \[addr\_context.esp\] (4 bytes)      ; push 01001898

0647   3C 02    0120AAFC         mov       \[addr\_context.register\], context.esp
0648   16 01    0120AB6C         Sub       \[addr\_context.register\], 00000004
0649   09 02    0120AB18         push      \[addr\_context.register\]
0650   3A 02    0120AB50         pop       \[addr\_context.esp\] (4 bytes)
0651   09 02    0120AB42         push      0120AB7A
0652   3C 02    0120AB0A         mov       \[addr\_context.register\], context.esp
0653   3A 02    0120AB26         pop       \[context.register\] (4 bytes)
0654   3C 02    0120AB34         mov       \[addr\_context.register\], 01007568
0655   02 02    0120AB5E         call      context.register        ; call 01007568



0656   02       0120AB7A         mov       \[addr\_context.register\], addr\_context.ebx
0657   09 02    0120ABDC         push      \[context.register\]
0658   3C 02    0120ABB2         mov       \[addr\_context.register\], addr\_context.ebx
0659   09 02    0120ABCE         push      \[context.register\]
0660   21 00    0120ABA4         Xor       \[esp+4\], \[esp\] (4 bytes)
0661   3C 02    0120ABC0         mov       \[addr\_context.register\], addr\_context.ebx
0662   3A 02    0120AB96         pop       \[context.register\] (4 bytes)        ; xor ebx, ebx

0663   27 02    0120AB88         load      context.register, 0C006D96
0664   3C 02    0120ABEA         mov       \[addr\_context.register\], addr\_context.ebx
0665   09 02    0120ABF8         push      \[context.register\]
0666   3C 02    0120AC06         mov       \[addr\_context.register\], context.esp
0667   16 01    0120AC30         Sub       \[addr\_context.register\], 00000004
0668   3A 02    0120AC4C         pop       \[context.register\] (4 bytes)
0669   09 02    0120AC3E         push      \[addr\_context.register\]
0670   3A 02    0120AC14         pop       \[addr\_context.esp\] (4 bytes)        ; push ebx


0671   27 02    0120AC22         load      context.register, 07BA6731
0672   3C 02    0120AC5A         mov       \[addr\_context.register\], 010010CC
0673   09 02    0120AC84         push      \[context.register\]
0674   3C 02    0120AC68         mov       \[addr\_context.register\], addr\_context.edi
0675   3A 02    0120AC76         pop       \[context.register\] (4 bytes)        ; mov edi, dword ptr \[010010CC\]

0676   3C 02    0120AC92         mov       \[addr\_context.register\], context.esp
0677   16 01    0120ACCA         Sub       \[addr\_context.register\], 00000004
0678   09 02    0120ACE6         push      \[addr\_context.register\]
0679   3A 02    0120ACBC         pop       \[addr\_context.esp\] (4 bytes)
0680   09 02    0120ACA0         push      0120AD10
0681   3C 02    0120AD02         mov       \[addr\_context.register\], context.esp
0682   3A 02    0120ACD8         pop       \[context.register\] (4 bytes)
0683   3C 02    0120ACAE         mov       \[addr\_context.register\], addr\_context.edi

0684   02 02    0120ACF4         call      \[context.register\]          ; call edi


0685   02       0120AD10         mov       \[addr\_context.register\], context.eax
0686   09 02    0120AD48         push      \[context.register\]
0687   09 02    0120AD2C         push      00005A4D
0688   3E 01    0120AD1E         Cmp       \[esp+4\], \[esp\] (2 bytes)        ; cmp word ptr \[eax\], 5A4D
0689   27 02    0120AD3A         load      context.register, 0E6ADC79

0690   3C 02    0120AD56         mov       \[addr\_context.register\], 010073DA
0691   02 02    0120AD64         jnz       context.register          ; jnz 010073DA

0692   3C 02    0120AD72         mov       \[addr\_context.register\], context.eax
0693   04 00    0120AD9C         Add       \[addr\_context.register\], 0000003C
0694   09 02    0120AD80         push      \[context.register\]
0695   3C 02    0120AD8E         mov       \[addr\_context.register\], addr\_context.ecx
0696   3A 02    0120ADB8         pop       \[context.register\] (4 bytes)        ; mov ecx, dword ptr \[eax+3C\]

0697   27 02    0120ADAA         load      context.register, 0B1B1124
0698   09 02    0120ADC6         push      010073BE
0699   3C 02    0120AE0C         mov       \[addr\_context.register\], context.esp
0700   16 01    0120ADFE         Sub       \[addr\_context.register\], 00000004
0701   3A 02    0120ADE2         pop       \[context.register\] (4 bytes)
0702   09 02    0120AE1A         push      \[addr\_context.register\]
0703   3A 02    0120ADF0         pop       \[addr\_context.esp\] (4 bytes)        ; push 010073BE

0704   27 02    0120ADD4         load      context.register, 018AABBB
0705   3C 02    0120AE28         mov       \[addr\_context.register\], context.esp
0706   09 02    0120AE60         push      \[context.register\]
0707   3C 02    0120AE7C         mov       \[addr\_context.register\], context.esp
0708   04 00    0120AE52         Add       \[addr\_context.register\], 00000004
0709   09 02    0120AE6E         push      \[addr\_context.register\]
0710   3A 02    0120AE36         pop       \[addr\_context.esp\] (4 bytes)
0711   3A 02    0120AE44         pop       \[addr\_context.register\] (4 bytes)      ; pop Reg

0712   02 02    0120AE8A         jmp       context.register          ; jmp Reg --> jmp 010073BE (跳去 VM 下的正常代码)

```

【感谢您看到这里，哈哈】