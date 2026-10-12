
# System Cleaner v4.91

> 原文：[`www.pediy.com/kssd/pediy06/pediy6680.htm`](https://www.pediy.com/kssd/pediy06/pediy6680.htm)

以壳解壳--ASProtect 1.23 RC4 [System Cleaner v4.91]  

【破解作者】 hmimys  

【作者邮箱】 hmimys@163.com  

【软件名称】 System Cleaner v4.91  

【保护方式】 ASProtect 1.23 RC4 - 1.3.08.24  

【软件简介】 提供了叁种简单有效的减肥方式。执行清除的速度很快，如果你不想清理某文件也可以设定 System Cleaner 2000 把文件先放到一个目录暂存，或是执行 ZIP 把文件压缩起来。有定时的功能，可以让你订定大扫除的时间，或是定时清理你的硬盘。  

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  

【调试环境】：WinXP+SP1、hmimyOD、PEiD、LordPE、AsprDbgr、ImportREC 1.6  

===========================================================================================================

## 脱壳过程

### 一、Pre-Dip\\Dump 以及区域脱壳

老规矩：用 `IsDebug 1.4` 插件去掉 `Ollydbg` 的调试器标志。

设置 `Ollydbg` 忽略除了“内存访问异常”之外的所有其它异常选项。

代码:

-------------------------------------------------------------------------------------------------------------

```
00401000 >  68 01B05F00     PUSH    5FB001 \\OD 载入后来到这里
00401005    E8 01000000     CALL    0040100B                  ; SystemCl.0040100B
0040100A    C3              RETN
0040100B    C3              RETN
0040100C    0F39            ???                               ; 未知命令
```

-------------------------------------------------------------------------------------------------------------

`Shift+F9` 通过异常,当堆栈第二次出现硬盘指纹

代码:

-------------------------------------------------------------------------------------------------------------

```
017F46E5    3100            XOR     [EAX], EAX
017F46E7    EB 01           JMP     SHORT 017F46EA
017F46E9    68 648F0500     PUSH    58F64
017F46EE    0000            ADD     [EAX], AL
017F46F0    00EB            ADD     BL, CH
017F46F2    02E8            ADD     CH, AL
017F46F4    0158 A1         ADD     [EAX-5F], EBX
017F46F7    B4 64           MOV     AH, 64
017F46F9    7F 01           JG      SHORT 017F46FC
017F46FB    8038 00         CMP     BYTE PTR [EAX], 0
017F46FE    0F84 63010000   JE      017F4867
017F4704    33C0            XOR     EAX, EAX
017F4706    55              PUSH    EBP
017F4707    68 65477F01     PUSH    17F4765
017F470C    64:FF30         PUSH    DWORD PTR FS:[EAX]
```

-------------------------------------------------------------------------------------------------------------

代码:

-------------------------------------------------------------------------------------------------------------

```
0012FF3C   0012FF44  指针到下一个 SEH 记录
0012FF40   017F469C  SE 句柄
0012FF44   0012FFE0  指针到下一个 SEH 记录
0012FF48   017F4C89  SE 句柄
0012FF4C   0012FF90
0012FF50   017E0000
0012FF54   017C0000
0012FF58   017F4178
0012FF5C   01811118  ASCII "wrpj/ABwEfQ=" 硬盘指纹
0012FF60   00000001
```

-------------------------------------------------------------------------------------------------------------

我们 `Alt+M` 打开内存镜像

在这里下内存访问断点,`Shift+F9` 运行

代码:

-------------------------------------------------------------------------------------------------------------

```
00563EB0    55              PUSH    EBP \\断在这里,清除内存断点
00563EB1    8BEC            MOV     EBP, ESP
00563EB3    A1 2CD75600     MOV     EAX, [56D72C] \\F4 来到这行,[56D72C]=00566578 先记下这个值
00563EB8    8B55 08         MOV     EDX, [EBP+8]
00563EBB    8910            MOV     [EAX], EDX
00563EBD    5D              POP     EBP
00563EBE    C2 0400         RETN    4
00563EC1    8D40 00         LEA     EAX, [EAX]
00563EC4    55              PUSH    EBP
00563EC5    8BEC            MOV     EBP, ESP
00563EC7    A1 6CD25600     MOV     EAX, [56D26C]
00563ECC    8B55 08         MOV     EDX, [EBP+8]
00563ECF    8910            MOV     [EAX], EDX
00563ED1    A1 44D85600     MOV     EAX, [56D844]
00563ED6    8B55 0C         MOV     EDX, [EBP+C]
00563ED9    8910            MOV     [EAX], EDX
00563EDB    5D              POP     EBP
00563EDC    C2 0800         RETN    8
```

-------------------------------------------------------------------------------------------------------------

取消内存断点，继续 `Shift+F9` 运行。来到 ASProtect 最后 1 次典型异常处。

代码:

-------------------------------------------------------------------------------------------------------------

```
017F3A2C    3100            XOR     [EAX], EAX \\也就是这里了,代码很有特征的!
017F3A2E    64:8F05 0000000>POP     DWORD PTR FS:[0]
017F3A35    58              POP     EAX
017F3A36    833D B07E7F01 0>CMP     DWORD PTR [17F7EB0], 0
017F3A3D    74 14           JE      SHORT 017F3A53
017F3A3F    6A 0C           PUSH    0C
017F3A41    B9 B07E7F01     MOV     ECX, 17F7EB0
017F3A46    8D45 F8         LEA     EAX, [EBP-8]
017F3A49    BA 04000000     MOV     EDX, 4
017F3A4E    E8 EDD0FFFF     CALL    017F0B40
017F3A53    FF75 FC         PUSH    DWORD PTR [EBP-4]
017F3A56    FF75 F8         PUSH    DWORD PTR [EBP-8]
017F3A59    8B45 F4         MOV     EAX, [EBP-C]
017F3A5C    8338 00         CMP     DWORD PTR [EAX], 0
017F3A5F    74 02           JE      SHORT 017F3A63
017F3A61    FF30            PUSH    DWORD PTR [EAX]
017F3A63    FF75 F0         PUSH    DWORD PTR [EBP-10]
017F3A66    FF75 EC         PUSH    DWORD PTR [EBP-14]
017F3A69    C3              RETN \\这里 F2 设断
```

-------------------------------------------------------------------------------------------------------------

`Shift+F9` 断在下断处,此时我们看堆栈!

代码:

-------------------------------------------------------------------------------------------------------------

```
0012FF5C   01809510
0012FF60   00400000  ASCII "MZP"
0012FF64   3F17F5E4
0012FF68   0012FFA4 \\注意此行 0012FF68 这个值!
```

-------------------------------------------------------------------------------------------------------------

在命令栏里输入 `HR 0012FF68`,`F9` 运行

代码:

-------------------------------------------------------------------------------------------------------------

```
0180C0EC   /EB 44           JMP     SHORT 0180C132 \\断在这里
0180C0EE   |EB 01          JMP     SHORT 0180C0F1
0180C0F0   |9A 51579CFC BF0>CALL    FAR 00BF:FC9C5751
0180C0F7   |0000            ADD     [EAX], AL
0180C0F9   |00B9 00000000   ADD     [ECX], BH
0180C0FF   |F3:AA           REP     STOS BYTE PTR ES:[EDI]
0180C101   |9D              POPFD
0180C102   |5F              POP     EDI
0180C103   |59              POP     ECX
0180C104   |C3              RETN
```

-------------------------------------------------------------------------------------------------------------

删除硬件断点,`F7` 一下!

代码:

-------------------------------------------------------------------------------------------------------------

```
0180C132    03C3            ADD     EAX, EBX \\来到这里          ; SystemCl.00400000
0180C134    BB CC050000     MOV     EBX, 5CC \\记注这里,等会修复时用的着
0180C139    0BDB            OR      EBX, EBX
0180C13B    75 07           JNZ     SHORT 0180C144
0180C13D    894424 1C       MOV     [ESP+1C], EAX
0180C141    61              POPAD
0180C142    50              PUSH    EAX
0180C143    C3              RETN
0180C144    E8 00000000     CALL    0180C149 \\记住这个 CALL 前的地址,修复时用的着
0180C149    5D              POP     EBP
0180C14A    81ED 49E14B00   SUB     EBP, 4BE149
0180C150    8D85 EEE04B00   LEA     EAX, [EBP+4BE0EE]
0180C156    8D8D 90E14B00   LEA     ECX, [EBP+4BE190]
0180C15C    03CB            ADD     ECX, EBX
0180C15E    8941 01         MOV     [ECX+1], EAX
0180C161    8D85 32E14B00   LEA     EAX, [EBP+4BE132]
0180C167    8D8D F6E04B00   LEA     ECX, [EBP+4BE0F6]
0180C16D    8901            MOV     [ECX], EAX
0180C16F    B8 5E140000     MOV     EAX, 145E
0180C174    8D8D FBE04B00   LEA     ECX, [EBP+4BE0FB]
0180C17A    8901            MOV     [ECX], EAX
0180C17C    8D8D 90E14B00   LEA     ECX, [EBP+4BE190]
0180C182    8D85 90F34B00   LEA     EAX, [EBP+4BF390]
```



### 逆向与脱壳操作

`0180C188    51               PUSH    ECX`  
`0180C189    50               PUSH    EAX`  
`0180C18A    E8 76FFFFFF      CALL    0180C105`  

\---------------------------------------------------------------

还记得`00566578`吗,不要告诉我不记得了,下命令`DD 00566578`  

`00566578  017E3861` ---> 这里指向注册名 ,改成`00566808`  

`0056657C  0000001E` ---> 这里指向使用天数,改成`FFFFFFFF`  

`00566580  0000001E` ---> 这里指向剩余天数,改成`FFFFFFFF`  

找个空的空间,我选了`00566808`,写入`Cracked by hmimys`  

`F7`后程序将跳到这里，到了这里程序代码已经解开，可以用`LordPE`纠正一下文件大小后完整`Dump`下程序。  

接着我们再进行区域脱壳：地址=`0180C000`，大小＝`00008000`，也就是脱出上面的部分壳处理代码段。  

至此我们已经将`ASProtect 1.23RC4`壳脱出来。  

现在我们来“组装”一下`dumped.exe`。先用`LordPE`打开`dumped.exe`，然后从磁盘载入刚才区域脱壳的`Region0180C000-01814000.dmp`区段，修改其`Voffset=0140C000`（`0180C000`－`00400000`＝`0140C000`），只保留`LordPE`的“验证 PE”选项，最后重建`PE`。  

#### 三、用 AsprDbgr 搞定输入表

现在我们接着来修复`IAT`，用`AsprDbgr`搞定输入表比较简单。启动`AsprDbgr`，加载未脱壳的程序，一路按确定键，直到目标程序启动。  

代码:  

\---------------------------------------------------------------

```
AsprDbgr v1.0beta (:P) Made by me... Manko.
  
  iEP=401000 (C:\\Program Files\\System Cleaner 2001\\SystemCleaner.exe)
    IAT Start: 57221C
          End: 572BA0
       Length: 984
      IATentry 572244 = 17F1CCC resolved as GetVersion
      IATentry 57227C = 17F17E4 resolved as GetProcAddress
      IATentry 572280 = 17F1CA4 resolved as GetModuleHandleA
      IATentry 572294 = 17F1D18 resolved as GetCommandLineA
      IATentry 57232C = 17F1CA4 resolved as GetModuleHandleA
      IATentry 5723F4 = 17F1D08 resolved as LockResource
      IATentry 572444 = 17F1CCC resolved as GetVersion
      IATentry 572474 = 17F17E4 resolved as GetProcAddress
      IATentry 572480 = 17F1CA4 resolved as GetModuleHandleA
      IATentry 5724C8 = 17F1D00 resolved as GetCurrentProcessId
      IATentry 5724CC = 17F1CF8 resolved as GetCurrentProcess
      IATentry 5724D8 = 17F1D30 resolved as FreeResource
    26 invalid entries erased.
  
  Dip-Table at adress: 17F7AB4
    0 563EB0 0 0 563EC4 0 0 563EE0 5642D8 56430C 0 0 0 0
  
  Last SEH passed. (17F3A2E) Searching for signatures. Singlestepping to OEP!
    
    Call + OEP-jump-setup at: 180C144 ( Code: E8000000 5D81ED )
    
    Mutated, stolen bytes at: 180C190 ( Code: 26EB01F2 F2EB01F3 )
    
    Erase of stolen bytes at: 180C0F3 ( Code: 9CFCBF32 C18001B9 )
      
      Repz ... found. Skipping erase of stolen bytes. ;)
  
  possible (temp)OEP: 407278 (Reached from preOEP: 180C104)
  
  Sugested tempOEP at: 564A5F
```

\---------------------------------------------------------------

启动`RecImport`，`OEP`：`0140C000`，`RAV`：`0017221C`，大小：`984`，搜索`IAT`，找到的`IAT`全部有效，修复`Dump`文件。`IAT`修复完毕。  

#### 四、以壳解壳：Stolen Code 简便解决方案

最后我们只用`OD`加载脱壳修复后的程序,现在的`OEP`是`0180C000`，修改其入口代码为：  

代码:  

\---------------------------------------------------------------

```
0180C000    BB CC050000     MOV     EBX, 5CC   
0180C005    E9 3A010000     JMP     0180C144   
```

\---------------------------------------------------------------

这样就利用了原壳的代码来处理`Stolen Code`了。保存之后就可以运行了。  

`Thanks Fly.Forgot.loveboom.jwh51.lordor.temerata.David.all of my friends and you!`  

`4dd466d0e1e0cc123afbe504da2462ca`
