
# PowerPoint to Flash 1.6.7 脱壳+破解

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-623.htm`](https://www.pediy.com/kssd/pediy07/pediy7-623.htm)

【破文标题】：PowerPoint to Flash 1.6.7 脱壳+破解  

【破文作者】：KuNgBiM[DFCG]  

【作者邮箱】：`gb_1227@163.com`  

【软件名称】：PowerPoint to Flash 1.6.7  

【软件大小】：734 KB  

【软件语言】：英文  

【下载地址】：[`www.dreamingsoft.com/download/pfsetup.exe`](http://www.dreamingsoft.com/download/pfsetup.exe)  

【软件简介】：PowerPoint to Flash 是第一个能够将 PowerPoint 的 `*.ppt` 档转换为 Flash 的 `*.swf` 文件的软件，它支持大量转档，可一次将多个演示文稿档案转换成目前最受欢迎的 Flash 格式，方便发布于网页，因为 Flash 只要浏览器就可以开启了，而 `*.ppt` 文件需要安装 PowerPoint 或检视程序。  

【保护方式】：注册码 + 30 次试用限制 + 启动 NAG 提示框  

【加密保护】：Armadillo 1.xx - 2.xx -> Silicon Realms Toolworks  

【编译语言】：Microsoft Visual C++ 7.0 [Debug]  

【调试环境】：WinXP、PEiD、Ollydbg、LordPE、ImportREC  

【破解日期】：2005-08-18  

【破解目的】：推广使用 Ollydbg 手动脱壳  

【作者声明】：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  

—————————————————————————————————  

### 脱壳过程  

#### 一、准备工作  

**侦壳**：用 PEiD 查壳，Armadillo 1.xx - 2.xx -> Silicon Realms Toolworks 加壳，打开 LordPE 看见该程序有 2 个进程，证明该软件具有双进程特征，下面我们就必须先分离父进程，要程序把自己当成子进程运行。  

**试运行**：启动时出现了 NAG 提示框，很明显，这个使用次数限制是 Armadillo 壳所添加的。  

—————————————————————————————————  

#### 二、脱壳  

1. 分离父进程，使程序把自己当成子进程运行  

设置 Ollydbg 忽略所有其它异常选项。老规矩：用 IsDebug 1.4 插件去掉 Ollydbg 的调试器标志。  

Ollydbg 载入主程序：  

```
00483914  >  55                    push ebp                          // OD 载入后停在这里
00483915    8BEC                   mov ebp,esp
00483917    6A FF                  push -1
00483919    68 D0D54800           push pptFlash.0048D5D0
0048391E    68 34344800           push pptFlash.00483434
00483923    64:A1 00000000        mov eax,dword ptr fs:[0]
00483929    50                    push eax
0048392A    64:8925 00000000      mov dword ptr fs:[0],esp
00483931    83EC 58               sub esp,58
00483934    53                    push ebx
00483935    56                    push esi
00483936    57                    push edi
00483937    8965 E8               mov dword ptr ss:[ebp-18],esp
0048393A    FF15 68A14800        call dword ptr ds:[<&KERNEL32.GetVersion>]        ; kernel32.GetVersion
00483940    33D2                 xor edx,edx
```

命令行下断：`BP OpenMutexA` 然后 `F9` 运行：

```
77E62391  >  55                    push ebp                          // 注意观察堆栈
77E62392    8BEC                   mov ebp,esp
77E62394    51                    push ecx
77E62395    51                    push ecx
77E62396    837D 10 00           cmp dword ptr ss:[ebp+10],0
77E6239A    56                    push esi
77E6239B    0F84 C2E30100         je kernel32.77E80763
77E623A1    64:A1 18000000        mov eax,dword ptr fs:[18]
77E623A7    FF75 10              push dword ptr ss:[ebp+10]
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

注意看 `BP OpenMutexA` 时的堆栈：

```
0012F5B8   0047E85D  /CALL 到 OpenMutexA 来自 pptFlash.0047E857
0012F5BC   001F0001  |Access = 1F0001
0012F5C0   00000000  |Inheritable = FALSE
0012F5C4   0012FBF8  \MutexName = "7A4::DA3935EA83"                ★注意这个地址：0012FBF8
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

`Ctrl+G`：401000 键入以下代码：

```
00401000    60                    pushad
00401001    9C                    pushfd
00401002    68 F8FB1200          push 12FBF8                       ★ 堆栈里看到的值
00401007    33C0                 xor eax,eax
00401009    50                    push eax
0040100A    50                    push eax
0040100B    E8 B5A6A577          call kernel32.CreateMutexA
00401010    9D                    popfd
00401011    61                    popad
00401012  - E9 7A13A677         jmp kernel32.OpenMutexA
```

在 401000 处新建起源，`F9` 运行，再次中断在 OpenMutexA 处，此时 `Ctrl+G`：401000 撤销刚才的修改的代码，使代码还原。OK，父进程分离完毕！把这段代码以二进制保存起来，下次用时直接粘贴，修改 00401002 处 push 的值就 OK 了。

二进制代码：`60 9C 68 F8 FB 12 00 33 C0 50 50 E8 B5 A6 A5 77 9D 61 E9 7A 13 A6 77`

—————————————————————————————————  

2. 寻找 Magic Jump，避开 IAT 加密  

取消以前所有的断点，下断：`BP GetModuleHandleA` 然后 `F9` 运行：

```
77E5AD86  >  837C24 04 00        cmp dword ptr ss:[esp+4],0         // 断在这，注意看堆栈
77E5AD8B    0F84 37010000        je kernel32.77E5AEC8
77E5AD91    FF7424 04           push dword ptr ss:[esp+4]
77E5AD95    E8 F8050000         call kernel32.77E5B392
77E5AD9A    85C0                 test eax,eax
77E5AD9C    74 08               je short kernel32.77E5ADA6
77E5AD9E    FF70 04            push dword ptr ds:[eax+4]
77E5ADA1    E8 27060000        call kernel32.GetModuleHandleW
77E5ADA6    C2 0400            retn 4
```

在这里中断 N 次，然后 `Alt+F9` 返回程序。其实很好判断返回程序的时机。

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

注意看 `BP GetModuleHandleA` 时的堆栈变化：

```
0012ED04   77C059FC  /CALL 到 GetModuleHandleA 来自 msvcrt.77C059F6
0012ED08   77BE31AC  \pModule = "kernel32.dll"

0012EDC8   772A8663  /CALL 到 GetModuleHandleA 来自 772A865D
0012EDCC   772AF8FC  \pModule = "KERNEL32.DLL"

0012ECF0   770FB124  /CALL 到 GetModuleHandleA 来自 OLEAUT32.770FB11E
0012ECF4   771722E4  \pModule = "KERNEL32.DLL"

0012ECEC   770FB124  /CALL 到 GetModuleHandleA 来自 OLEAUT32.770FB11E
0012ECF0   771722E4  \pModule = "KERNEL32.DLL"

0012F570   0047DDD9  /CALL 到 GetModuleHandleA 来自 pptFlash.0047DDD3
0012F574   00000000  \pModule = NULL

0012E00C   003D60DB  /CALL 到 GetModuleHandleA 来自 003D60D5
0012E010   003EB808  \pModule = "kernel32.dll"

0012E00C   003D60DB  /CALL 到 GetModuleHandleA 来自 003D60D5
0012E010   003EB7FC  \pModule = "user32.dll"

0012E048   003E0375  /CALL 到 GetModuleHandleA 来自 003E036F
0012E04C   00C52008  \pModule = "SHLWAPI.dll"

0012E02C   003D653E  /CALL 到 GetModuleHandleA 来自 003D6538        ★注意！在这里 Alt+F9 返回程序
0012E030   00000000  \pModule = NULL
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

当堆栈如上变化后，就可以 `Alt+F9` 返回程序代码了。

```
003D6522    55                    push ebp
003D6523    8BEC                   mov ebp,esp
003D6525    53                    push ebx
003D6526    56                    push esi
003D6527    57                    push edi
003D6528    33FF                   xor edi,edi
003D652A    33DB                   xor ebx,ebx
003D652C    66:F745 0E FFFF       test word ptr ss:[ebp+E],0FFFF
003D6532    75 03                jnz short 003D6537
003D6534    8B5D 0C             mov ebx,dword ptr ss:[ebp+C]
003D6537    57                    push edi
003D6538    FF15 CC903E00        call dword ptr ds:[3E90CC]                       ; kernel32.GetModuleHandleA
003D653E    8B4D 08             mov ecx,dword ptr ss:[ebp+8]         // 返回到这里
003D6541    3BC8                 cmp ecx,eax
003D6543    75 07                jnz short 003D654C
003D6545    B8 A8B33E00          mov eax,3EB3A8
003D654A    EB 2F                jmp short 003D657B
003D654C    393D D8B73E00        cmp dword ptr ds:[3EB7D8],edi
003D6552    B8 D8B73E00          mov eax,3EB7D8
003D6557    74 0C                je short 003D6565                  ★ Magic Jump ★  // 改为 ：jmp 003D6565
003D6559    3B48 08             cmp ecx,dword ptr ds:[eax+8]
003D655C    74 1A                je short 003D6578
003D655E    83C0 0C              add eax,0C
003D6561    3938                 cmp dword ptr ds:[eax],edi
003D6563  ^ 75 F4               jnz short 003D6559
003D6565    FF75 0C             push dword ptr ss:[ebp+C]
003D6568    FF75 08             push dword ptr ss:[ebp+8]
003D656B    FF15 F8903E00        call dword ptr ds:[3E90F8]                       ; kernel32.GetProcAddress
003D6571    5F                    pop edi
003D6572    5E                    pop esi
003D6573    5B                    pop ebx
003D6574    5D                    pop ebp
003D6575    C2 0800              retn 8
```

以前脱 Armadillo 3.60 以上的壳时总是在 Magic Jump 处下 硬件执行 断点，每次断下后改标志 Z=1,使其 `JMP`。如果直接改为 `JMP`，程序会异常出错！主要是程序自校验问题，但对付这种 Armadillo 旧版壳时，就可以直接修改的！

—————————————————————————————————  

3. 第 2 区段内存断点大法，直达 OEP  

`Alt+M` 查看内存，在 401000 区段上下“内存访问断点”，`F9` 运行，直接中断在 OEP 处：



`00438570    6A 60                   push 60                                   // 在这儿用 LordPE 纠正 ImageSize 后完全 Dump 这个进程`

`00438572    68 98534600             push pptFlash.00465398`

`00438577    E8 30230000             call pptFlash.0043A8AC`

`0043857C    BF 94000000             mov edi,94`

`00438581    8BC7                    mov eax,edi`

`00438583    E8 A8FAFFFF             call pptFlash.00438030`

`00438588    8965 E8                 mov dword ptr ss:[ebp-18],esp`

`0043858B    8BF4                    mov esi,esp`

`0043858D    893E                    mov dword ptr ds:[esi],edi`

`0043858F    56                     push esi`

`00438590    FF15 B4C14500          call dword ptr ds:[45C1B4]               ; kernel32.GetVersionExA`

`00438596    8B4E 10                mov ecx,dword ptr ds:[esi+10]`

`00438599    890D 14794700          mov dword ptr ds:[477914],ecx`

`0043859F    8B46 04                mov eax,dword ptr ds:[esi+4]`

`004385A2    A3 20794700            mov dword ptr ds:[477920],eax`

`004385A7    8B56 08                mov edx,dword ptr ds:[esi+8]`

`004385AA    8915 24794700          mov dword ptr ds:[477924],edx`

`004385B0    8B76 0C                mov esi,dword ptr ds:[esi+C]`

`004385B3    81E6 FF7F0000          and esi,7FFF`

`004385B9    8935 18794700          mov dword ptr ds:[477918],esi`

`004385BF    83F9 02                cmp ecx,2`

`004385C2    74 0C                  je short pptFlash.004385D0`

运行 `ImportREC 1.6`，选择这个进程。把 `OEP` 改为 `00038570`，点 `IT AutoSearch`，函数全部有效。`FixDump`，正常运行！

优化：用 `LordPE` 删除区段“.text1”、“.data1”、“.pdata”然后“重建 PE”！程序由脱壳后的 2.18MB 变为 1.59MB！嘿嘿~~！！

### 三、破解

程序采用了 `Armadillo` 的注册模块，脱壳后自然就不必注册了，30 次的试用限制也不存在了。

但是，在某些无用户名的机器上出现注册非法，下面我就来搞定它！使它变成“永久注册版”，嘿嘿~~~

再次打开 `Ollydbg`，载入我们脱壳优化后的文件“`dumped_1.exe`”，右键使用 `Ultra String Reference` 插件的 `Find ASCII` 功能项，查找我们需要的相关信息：

```
00401A7B   push 复件_dum.0045C848                    Unregistered Copy
00401AD3   push 复件_dum.0045C828                    This copy is licensed to:\n\n%s
00401BF0   push 复件_dum.0045C85C                    open
00401FE8   push 复件_dum.0045C8B4                    ArmAccess.DLL
00402004   push 复件_dum.0045C89C                    System Error! 0X0E01
0040200B   push 复件_dum.0045C890                    CheckCode
0040201E   push 复件_dum.0045C878                    System Error! 0X0E02
00402068   push 复件_dum.0045C868                    Invalid Key!
00402155   push 复件_dum.0045C8E0                    USERNAME       ★读取用户名，双击这里★
0040220A   push 复件_dum.0045C8D8                    DEFAULT
0040224C   push 复件_dum.0045C8D0                    EXPIRED
0040229A   push 复件_dum.0045C8C4                    USESLEFT
004024A8   push 复件_dum.0045C8EC                    %d uses left
........
```

程序代码：

```
0040214F    68 FF000000          push 0FF                            // 读取用户名 ★从这里修改★
00402154    50                   push eax
00402155    68 E0C84500          push 复件_dum.0045C8E0             // 双击来到这里
0040215A    FFD5                 call ebp
0040215C    85C0                 test eax,eax
0040215E    8B16                 mov edx,dword ptr ds:[esi]          // 这就是保存注册名的位置
00402160    75 78                jnz short 复件_dum.004021DA
00402162    85D2                 test edx,edx
00402164    74 19                je short 复件_dum.0040217F
00402166    8BC2                 mov eax,edx
00402168    8D78 01              lea edi,dword ptr ds:[eax+1]
0040216B    EB 03                jmp short 复件_dum.00402170
0040216D    8D49 00              lea ecx,dword ptr ds:[ecx]
00402170    8A08                 mov cl,byte ptr ds:[eax]
00402172    40                   inc eax
00402173    84C9                 test cl,cl
00402175   ^ 75 F9                jnz short 复件_dum.00402170
00402177    2BC7                 sub eax,edi
00402179    0F88 9B020000        js 复件_dum.0040241A
0040217F    3B42 F8              cmp eax,dword ptr ds:[edx-8]
00402182    0F8F 92020000        jg 复件_dum.0040241A
00402188    8942 F4              mov dword ptr ds:[edx-C],eax
0040218B    8B0E                 mov ecx,dword ptr ds:[esi]
0040218D    C60408 00            mov byte ptr ds:[eax+ecx],0
00402191    8D4C24 24            lea ecx,dword ptr ss:[esp+24]
00402195    E8 46040000          call 复件_dum.004025E0
0040219A    6A 10                push 10
0040219C    68 661B0000          push 1B66
004021A1    8D4C24 2C            lea ecx,dword ptr ss:[esp+2C]
004021A5    C74424 24 00000000  mov dword ptr ss:[esp+24],0        // 比较注册名是否为 0
004021AD    E8 4E040000          call 复件_dum.00402600             // 为 0 则死！★这里可以 nop 掉，但没注册名看起不太爽！★
004021B2    8D4C24 24            lea ecx,dword ptr ss:[esp+24]
004021B6    C74424 1C FFFFFFFF   mov dword ptr ss:[esp+1C],-1
004021BE    E8 2D040000          call 复件_dum.004025F0
004021C3    5F                   pop edi
004021C4    5E                   pop esi
004021C5    5D                   pop ebp
004021C6    33C0                 xor eax,eax
004021C8    5B                   pop ebx
004021C9    8B4C24 04            mov ecx,dword ptr ss:[esp+4]
004021CD    64:890D 00000000     mov dword ptr fs:[0],ecx
004021D4    83C4 10              add esp,10
004021D7    C2 0400              retn 4
```

`0040214F` 处修改代码：

```
0040214F    C700 204B754E        mov dword ptr ds:[eax],4E754B20
00402155    C740 04 6742694D     mov dword ptr ds:[eax+4],4D694267
```

修改后保存即可！永久注册版诞生咯~~~~

版权所有（C）2005 `KuNgBiM[DFCG]`
Copyright (C) 2005 `KuNgBiM[DFCG]`

Cracked By `KuNgBiM[DFCG]`

2005-08-18

14:29:18 PM

`4dd466d0e1e0cc123afbe504da2462ca`
