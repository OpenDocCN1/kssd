
# EncryptPE V2.2005.3.14 试炼 Service 保护方式

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-661.htm`](https://www.pediy.com/kssd/pediy07/pediy7-661.htm)

EncryptPE V2.2005.3.14 试炼 Service 保护方式脱壳

- 下载页面： `http://www.encryptpe.com/`
- 软件大小： 2.11M
- 软件语言： 简体中文
- 软件类别： 国产软件 / 共享版 / 加密工具
- 应用平台： `Win9X`/`ME`/`NT`/`2000`/`XP`/`2003`

**软件简介**： EncryptPE 是老王的强壳。EncryptPE 能加密保护常规 PE 文件（`EXE`、`DLL`、`OCX` 等一般程序或 NT 服务程序），防静态分析修改，反动态跟踪调试，有效地保护软件，防止盗版。除常规的对抗调试器（`SoftIce`、`TRW`、`OllyDbg` 等）、监视器、`DUMP` 工具方法外，EncryptPE 采用的加密保护的手段还有：随机加密算法、`CRC` 校验、变形、代码替换、进程注入、`APIHOOK`、多线程、调试运行、全程监控等。

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教

【调试环境】：`WinXP`、`flyODBG`、`PEiD`、`LordPE`、`ImportREC`

---

## 一、EncryptPE 的版本和加壳方式

EncryptPE 是老王的强壳，想必大家都看到过，外挂、木马之类用 EncryptPE 加壳的较多。

简单整理了一下以前的脱壳记录，希望 EncryptPE 越来越猛，说点拜年话，免得老王来扁我，呵呵，不过大多数外挂都是采用“非 Service”加壳方式，所以老王也不必担心。

其实 EncryptPE V2.2004.8.10 和 EncryptPE V2.2005.3.14 就脱壳而言是一样的。

```
[EncryptPE V2.2004.8.10-V2.2005.3.14 -> WFS]

signature = 60 9C 64 FF 35 00 00 00 00 E8 7A

ep_only = true
```

最简单、准确地察看 EncryptPE 版本号是用 `WinHex` 等 16 进制工具打开被加壳程序，在 PE 头下面就可以看到。

```
00000220   45 50 45 3A 20 45 6E 63  72 79 70 74 50 45 20 56   EPE: EncryptPE V
00000230   32 2E 32 30 30 35 2E 33  2E 31 34 2C 20 43 6F 70   2.2005.3.14, Cop
00000240   79 72 69 67 68 74 20 28  43 29 20 57 46 53 00 00   yright (C) WFS..
```

EncryptPE 有 2 种加壳方式：Service 和非 Service。看帮助里的说明：“选择您要加密的 PE 文件（`EXE`、`DLL`、`OCX` 等），如果是 NT 服务程序，请选中 Service 项（一般程序也可以在选中 Service 项后加密，其保护强度比不选中 Service 时稍低）。”

最简单的判断加壳方式的方法是：运行加壳程序后在 `LordPE` 里可以看见真实进程名的是“Service”加壳方式，只看见 `[system]` 字样的是“非 Service”加壳方式，如 EncryptPE 主程序。

“Service”加壳方式强度低了不少，柿子拣软的捏，呵呵，当然从这个先下手了。

其实两种加壳方式的输入表处理、代码变形的处理方法都是一样的，只不过“非 Service”加壳方式要注入 `explorer.exe` 解码、开启新的进程。

---

## 二、获得加壳前文件的 PE 头、OEP 等信息

运行 EncryptPE 以“Service”方式加壳 `Win98` 的记事本，下面开始演示脱壳。

设置 `OllyDbg` 忽略所有异常选项。用 `IsDebug` 插件去掉 `OllyDbg` 的调试器标志。

```
0040D000    60               pushad
//进入 OllyDbg 后暂停在这
0040D001    9C               pushfd
0040D002    64:FF35 0000000> push dword ptr fs:[0]
0040D009    E8 7A010000      call 0040D188
```

`BP IsDebuggerPresent` 中断后看堆栈：

```
0104FF40   711AF6B2  /CALL 到 IsDebuggerPresent 来自 V2200531.711AF6B0
0104FF44   0104FF78  指针到下一个 SEH 记录
```

看到 EncryptPE 开始检测 `IsDebuggerPresent`，取消这个断点。

`Alt+M` 打开内存察看窗口，找到 EncryptPE 所使用的支持库 `V2200531.epe`

| 地址 | 大小 | 物主 | 区段 | 包含 |
| :--- | :--- | :--- | :--- | :--- |
| `71120000` | `00001000` | `V2200531` | | PE header |
| `71121000` | `000AC000` | `V2200531` | `EPE0` | |
| `711CD000` | `0006A000` | `V2200531` | `EPE1` | code |
| `71237000` | `00001000` | `V2200531` | `.rsrc` | data,imports,export |

`V2200531.epe` 就是 EncryptPE 壳的核心了。

`Ctrl+G`：`711CD000` 这是 `V2200531.epe` 第 3 区段的开始地址。

```
711CD000    894D F8          mov dword ptr ss:[ebp-8],ecx
//V2200531.epe 第 3 区段的开始地址
711CD003    8955 FC          mov dword ptr ss:[ebp-4],edx
711CD006    8BF0             mov esi,eax
```

`Ctrl+S` 在当前位置下搜索命令序列：

```
lea ecx,dword ptr ss:[ebp-28]
mov eax,dword ptr ss:[ebp-18]
mov edx,dword ptr ds:[eax]
mov eax,dword ptr ss:[ebp-28]
```

找到在 `711E4514` 处，我们在其下 `cmp dword ptr ss:[ebp-28],0` 的 `711E4524` 处设置硬件执行断点，`Shift+F9` 中断下来

```
711E4514    8D4D D8          lea ecx,dword ptr ss:[ebp-28]
//找到这里
711E4517    8B45 E8          mov eax,dword ptr ss:[ebp-18]
711E451A    8B10             mov edx,dword ptr ds:[eax]
711E451C    8B45 D8          mov eax,dword ptr ss:[ebp-28]
711E451F    E8 0CA3FCFF      call 711AE830
711E4524    837D D8 00       cmp dword ptr ss:[ebp-28],0
//硬件执行断点              [ebp-28]=[0013FF5C]=00E4D6DC  ★
711E4528    0F84 AE0C0000    je 711E51DC
711E452E    837D D4 00       cmp dword ptr ss:[ebp-2C],0
711E4532    0F8E A40C0000    jle 711E51DC
711E4538    8B45 D8          mov eax,dword ptr ss:[ebp-28]
711E453B    8B40 3C          mov eax,dword ptr ds:[eax+3C]
711E453E    0345 D8          add eax,dword ptr ss:[ebp-28]
711E4541    8B40 28          mov eax,dword ptr ds:[eax+28]
711E4544    0345 DC          add eax,dword ptr ss:[ebp-24]
//EAX=000010CC+00400000=004010CC    ★   OEP 值
711E4547    83F0 FF          xor eax,FFFFFFFF
//异或 FFFFFFFF 后作为下面解码的参数
711E454A    8945 F0          mov dword ptr ss:[ebp-10],eax
```

在 `711E4524` 处中断后在数据窗口察看 `[0013FF5C]=00E4D6DC` 处数据：

```
00E4D6DC   4D 5A 90 00 03 00 00 00 04 00 00 00 FF FF 00 00  MZ?..............
00E4D6EC   B8 00 00 00 00 00 00 00 40 00 00 00 00 00 00 00  ?......@.......
00E4D6FC   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
00E4D70C   00 00 00 00 00 00 00 00 00 00 00 00 80 00 00 00  ............€...
00E4D71C   0E 1F BA 0E 00 B4 09 CD 21 B8 01 4C CD 21 54 68  ?.???L?Th
00E4D72C   69 73 20 70 72 6F 67 72 61 6D 20 63 61 6E 6E 6F  is program canno
00E4D73C   74 20 62 65 20 72 75 6E 20 69 6E 20 44 4F 53 20  t be run in DOS
00E4D74C   6D 6F 64 65 2E 0D 0D 0A 24 00 00 00 00 00 00 00  mode....$.......
00E4D75C   50 45 00 00 4C 01 05 00 65 91 46 35 00 00 00 00  PE..L.e 慒 5....
```

很明显，这是 PE 头数据，是哪个 PE 文件的？呵呵，加壳前的文件！并且 PE 头、资源段、重定位表段等都是原来的，当然，代码段某些数据是壳所加密的。现在用 `LordPE` 部分脱壳出这部分数据，`Address=00E4D6DC`、`Size=0000D000`，存为 `dumped0.exe`

`Size` 可以根据最后一个区段信息计算得出。

```
00E4D8EC   00 00 00 00 40 00 00 40 2E 72 65 6C 6F 63 00 00  ....@..@.reloc..
00E4D8FC   9C 0A 00 00 00 C0 00 00 00 10 00 00 00 C0 00 00  ?...?.....?.
```

`Size=0000C000(ROffset)+00001000(RSize)=0000D000`

---

## 三、Magic Jmp 搞定输入表

下面来避开输入表加密。`Ctrl+F` 在整个段块搜索命令：`mov eax,0F`

找到在 `711E3748` 处，这里就是处理输入表加密的地方了。

在 `711E3756` 处设置硬件执行断点，`Shift+F9` 中断后修改 `Magic Jmp`

```
711E3728    55               push ebp
711E3729    8BEC             mov ebp,esp
711E372B    83C4 F8          add esp,-8
711E372E    53               push ebx
711E372F    56               push esi
711E3730    57               push edi
711E3731    8BF9             mov edi,ecx
711E3733    8955 FC          mov dword ptr ss:[ebp-4],edx
711E3736    8BF0             mov esi,eax
711E3738    8B45 FC          mov eax,dword ptr ss:[ebp-4]
711E373B    8945 F8          mov dword ptr ss:[ebp-8],eax
711E373E    807D 08 00       cmp byte ptr ss:[ebp+8],0
711E3742    0F84 85000000    je 711E37CD
711E3748    B8 0F000000      mov eax,0F
//找到这里
711E374D    E8 1AF0F3FF      call 7112276C
711E3752    8BD8             mov ebx,eax
711E3754    85DB             test ebx,ebx
711E3756    74 75            je short 711E37CD
//修改为：JMP 711E37CD   ★  Magic Jmp
711E3758    8B86 68030000    mov eax,dword ptr ds:[esi+368]
711E375E    E8 3D24F4FF      call 71125BA0
711E3763    40               inc eax
711E3764    50               push eax
711E3765    8D86 68030000    lea eax,dword ptr ds:[esi+368]
711E376B    B9 01000000      mov ecx,1
711E3770    8B15 ACF61D71    mov edx,dword ptr ds:[711DF6AC]
711E3776    E8 E125F4FF      call 71125D5C
711E377B    83C4 04          add esp,4
711E377E    8B86 68030000    mov eax,dword ptr ds:[esi+368]
711E3784    E8 1724F4FF      call 71125BA0
711E3789    8B96 68030000    mov edx,dword ptr ds:[esi+368]
711E378F    895C82 FC        mov dword ptr ds:[edx+eax*4-4],ebx
711E3793    C603 E8          mov byte ptr ds:[ebx],0E8
711E3796    8BD3             mov edx,ebx
711E3798    8BC2             mov eax,edx
711E379A    40               inc eax
711E379B    B9 68C91A71      mov ecx,711AC968
711E37A0    2BC8             sub ecx,eax
711E37A2    83E9 04          sub ecx,4
711E37A5    8908             mov dword ptr ds:[eax],ecx
711E37A7    C643 05 FF       mov byte ptr ds:[ebx+5],0FF
711E37AB    C643 06 25       mov byte ptr ds:[ebx+6],25
711E37AF    8BC2             mov eax,edx
711E37B1    83C0 07          add eax,7
711E37B4    8BCA             mov ecx,edx
711E37B6    83C1 0B          add ecx,0B
711E37B9    8908             mov dword ptr ds:[eax],ecx
711E37BB    8BC3             mov eax,ebx
711E37BD    83C0 0B          add eax,0B
```



```
711E37C0    8BCA            mov ecx,edx  
711E37C2    83C1 05         add ecx,5  
711E37C5    334D FC         xor ecx,dword ptr ss:[ebp-4]  
711E37C8    8908            mov dword ptr ds:[eax],ecx  
711E37CA    8955 F8         mov dword ptr ss:[ebp-8],edx  
711E37CD    85FF            test edi,edi  
711E37CF    74 29           je short 711E37FA  
711E37D1    33D2            xor edx,edx  
711E37D3    55              push ebp  
711E37D4    68 F0371E71     push 711E37F0  
711E37D9    64:FF32         push dword ptr fs:[edx]  
711E37DC    64:8922         mov dword ptr fs:[edx],esp  
711E37DF    8BC7            mov eax,edi  
711E37E1    8B55 F8         mov edx,dword ptr ss:[ebp-8]  
711E37E4    8910            mov dword ptr ds:[eax],edx ; SHELL32.ShellExecuteA  

//正确的函数写入，这里可以看见 IAT 地址  
711E37E6    33C0            xor eax,eax  
711E37E8    5A              pop edx  
711E37E9    59              pop ecx  
711E37EA    59              pop ecx  
711E37EB    64:8910         mov dword ptr fs:[eax],edx  
711E37EE    EB 0A           jmp short 711E37FA  
711E37F0    E9 7706F4FF     jmp 71123E6C  
711E37F5    E8 DA09F4FF     call 711241D4  
711E37FA    8B45 F8         mov eax,dword ptr ss:[ebp-8]  
711E37FD    5F              pop edi  
711E37FE    5E              pop esi  
711E37FF    5B              pop ebx  
711E3800    59              pop ecx  
711E3801    59              pop ecx  
711E3802    5D              pop ebp  
711E3803    C2 0400         retn 4  

//返回 711E4E06 处  
```

修改完 `Magic Jmp` 后取消 `711E3756` 处硬件断点，继续看

```
711E4E01    E8 22E9FFFF     call 711E3728  

//处理输入表加密  
711E4E06    EB 36           jmp short 711E4E3E  
711E4E08    8BD0            mov edx,eax  
711E4E0A    81E2 FFFF0000   and edx,0FFFF  
711E4E10    8B45 FC         mov eax,dword ptr ss:[ebp-4]  
711E4E13    E8 3874FCFF     call 711AC250  
711E4E18    8945 BC         mov dword ptr ss:[ebp-44],eax  
711E4E1B    8A45 EF         mov al,byte ptr ss:[ebp-11]  
711E4E1E    50              push eax  
711E4E1F    A1 94DD1F71     mov eax,dword ptr ds:[711FDD94]  
711E4E24    8B00            mov eax,dword ptr ds:[eax]  
711E4E26    8B55 E4         mov edx,dword ptr ss:[ebp-1C]  
711E4E29    8B0490          mov eax,dword ptr ds:[eax+edx*4]  
711E4E2C    8BCF            mov ecx,edi  
711E4E2E    8B55 BC         mov edx,dword ptr ss:[ebp-44]  
711E4E31    E8 F2E8FFFF     call 711E3728  
711E4E36    3BFB            cmp edi,ebx  
711E4E38    74 04           je short 711E4E3E  
711E4E3A    33C0            xor eax,eax  
711E4E3C    8903            mov dword ptr ds:[ebx],eax  
711E4E3E    83C3 04         add ebx,4  
711E4E41    83C7 04         add edi,4  
711E4E44    8B03            mov eax,dword ptr ds:[ebx]  
711E4E46    85C0            test eax,eax  
711E4E48    0F87 93FEFFFF  ja 711E4CE1  
711E4E4E    A1 9CDB1F71     mov eax,dword ptr ds:[711FDB9C]  

//[711FDB9C]=71200685  
711E4E53    8038 00         cmp byte ptr ds:[eax],0  

//[71200685]处保存的是检验标志位  
711E4E56    75 13           jnz short 711E4E6B  

//NOP 掉  ★  
711E4E58    83C6 14         add esi,14  
711E4E5B    837E 0C 00      cmp dword ptr ds:[esi+C],0  
711E4E5F    76 0A           jbe short 711E4E6B  
711E4E61    837E 10 00      cmp dword ptr ds:[esi+10],0  
711E4E65    0F87 6AFBFFFF  ja 711E49D5  

//循环处理输入表  
711E4E6B    33C0            xor eax,eax  
```

//这里设置硬件执行断点  ★

Shift+F9，当我们中断在 `711E4E6B` 处时输入表就处理完毕了。运行 `ImportRec`，填入 `RVA=000062E0`、`Size=240`，获取输入表，函数都是有效的。修改 `OEP RVA=000010CC`，可以保存树文件了。

## 四、部分检验

现在还原上面所作的修改吧，壳还有自检验的

```
711E4E6B    33C0            xor eax,eax  
711E4E6D    E8 9A050000     call 711E540C  

//检验 ①   有兴趣可以跟踪进入看看 EncryptPE 是如何检验的  
711E4E72    84C0            test al,al  
711E4E74    75 0B           jnz short 711E4E8  

//如果检验有问题的话，则手动修改标志位 Z=1，使这里不跳转 ★  
711E4E76    33C0            xor eax,eax  
711E4E78    E8 2FBBFCFF     call 711B09AC  

//检验 ②  
711E4E7D    84C0            test al,al  
711E4E7F    74 08           je short 711E4E89  

//如果检验有问题的话，则手动修改标志位 Z=1，使这里跳转   ★  
711E4E81    A1 9CDB1F71     mov eax,dword ptr ds:[711FDB9C]  
711E4E86    C600 01         mov byte ptr ds:[eax],1  

//这里如果置 1 的话，则解码会不完全！因此要跳过去！★  
```

## 五、解码修复

如果此时 `Ctrl+G：4010CC`，或者 `Dump` 修复的话，则代码还没有还原完全

```
004010CC    55              push ebp  
004010CD    8BEC            mov ebp,esp  
004010CF    83EC 44         sub esp,44  
004010D2    56              push esi  
004010D3    90              nop  
004010D4    E8 D38CBFFF     call FFFF9DAC  

//代码还没有还原  
```

下面开始变形代码的修复。通过修改 `EncryptPE` 的运行流程，使其自动还原变形代码。

`Ctrl+F` 在当前位置下搜索命令：`sub ax,0CCE9`，找到在 `711E4FD2` 处，`F4` 过去

```
711E4FCA    8BD8            mov ebx,eax  
711E4FCC    83EB 02         sub ebx,2  
711E4FCF    66:8B03         mov ax,word ptr ds:[ebx]  

//下面是根据 MOV 和调用方式进行恢复代码 ★  
711E4FD2    66:2D E9CC      sub ax,0CCE9  

//找到这里  
711E4FD6    74 0E           je short 711E4FE6  
711E4FD8    66:2D A71B      sub ax,1BA7  
711E4FDC    74 47           je short 711E5025  
711E4FDE    66:2D 0001      sub ax,100  
711E4FE2    74 41           je short 711E5025  
711E4FE4    EB 7D           jmp short 711E5063  
```

### 1、调用表和跳转表解密

```
711E4FE6    E8 297EFCFF     call 711ACE14  

//统一放 call 711ACE14 里面 Patch！★  
711E4FEB    66:C700 FF25    mov word ptr ds:[eax],25FF  

//修改 1 ：NOP ★  
711E4FF0    8B55 FC         mov edx,dword ptr ss:[ebp-4]  

//修改 2 ：NOP ★  
711E4FF3    8950 02         mov dword ptr ds:[eax+2],edx  

//修改 3 ：NOP ★  
711E4FF6    8BD0            mov edx,eax  
711E4FF8    8B5D E8         mov ebx,dword ptr ss:[ebp-18]  
711E4FFB    83C3 06         add ebx,6  
711E4FFE    8B45 E0         mov eax,dword ptr ss:[ebp-20]  
711E5001    03C0            add eax,eax  
711E5003    03D8            add ebx,eax  
711E5005    8B45 E8         mov eax,dword ptr ss:[ebp-18]  
711E5008    8B00            mov eax,dword ptr ds:[eax]  
711E500A    0FB70B          movzx ecx,word ptr ds:[ebx]  
711E500D    81E9 00300000   sub ecx,3000  
711E5013    03C1            add eax,ecx  
711E5015    0345 DC         add eax,dword ptr ss:[ebp-24]  
711E5018    48              dec eax  
711E5019    2BD0            sub edx,eax  
711E501B    83EA 04         sub edx,4  
711E501E    8910            mov dword ptr ds:[eax],edx  

//修改 4 ：写入加密地址  NOP 掉！   ★  
711E5020    E9 68010000     jmp 711E518D  
711E5025    E8 EA7DFCFF     call 711ACE14  

//统一放 call 711ACE14 里面 Patch！★  
711E502A    66:C700 FF25    mov word ptr ds:[eax],25FF  

//修改 5 ：NOP ★  
711E502F    8B55 FC         mov edx,dword ptr ss:[ebp-4]  

//修改 6 ：NOP ★  
711E5032    8950 02         mov dword ptr ds:[eax+2],edx  

//修改 7 ：NOP ★  
711E5035    8BD0            mov edx,eax  
711E5037    8B5D E8         mov ebx,dword ptr ss:[ebp-18]  
711E503A    83C3 06         add ebx,6  
711E503D    8B45 E0         mov eax,dword ptr ss:[ebp-20]  
711E5040    03C0            add eax,eax  
711E5042    03D8            add ebx,eax  
711E5044    8B45 E8         mov eax,dword ptr ss:[ebp-18]  
711E5047    8B00            mov eax,dword ptr ds:[eax]  
711E5049    0FB70B          movzx ecx,word ptr ds:[ebx]  
711E504C    81E9 00300000   sub ecx,3000  
711E5052    03C1            add eax,ecx  
711E5054    0345 DC         add eax,dword ptr ss:[ebp-24]  
711E5057    2BD0            sub edx,eax  
711E5059    83EA 04         sub edx,4  
711E505C    8910            mov dword ptr ds:[eax],edx  

//修改 8 ：写入加密地址  NOP 掉！   ★  
711E505E    E9 2A010000     jmp 711E518D  
```

### 2、MOV 类型解密

```
711E5063    8B15 94DD1F71   mov edx,dword ptr ds:[711FDD94]  
711E5069    8B12            mov edx,dword ptr ds:[edx]  
711E506B    8B4D E4         mov ecx,dword ptr ss:[ebp-1C]  
711E506E    8B148A          mov edx,dword ptr ds:[edx+ecx*4]  
711E5071    80BA 5E030000 0>cmp byte ptr ds:[edx+35E],3  
```



711E5078    0F86 0F010000   `jbe 711E518D`

711E507E    66:8B13         `mov dx, word ptr ds:[ebx]`

711E5081    66:81E2 FF00    `and dx, 0FF`

711E5086    66:81FA 8900    `cmp dx, 89`

711E508B    74 13           `je short 711E50A0`

711E508D    66:8B13         `mov dx, word ptr ds:[ebx]`

711E5090    66:81E2 FF00    `and dx, 0FF`

711E5095    66:81FA 8B00    `cmp dx, 8B`

711E509A    0F85 ED000000   `jnz 711E518D`

711E50A0    66:8B03         `mov ax, word ptr ds:[ebx]`

711E50A3    66:25 00FF      `and ax, 0FF00`

711E50A7    66:3D 0005      `cmp ax, 500`

711E50AB    74 4C           `je short 711E50F9`

711E50AD    66:3D 0015      `cmp ax, 1500`

711E50B1    74 46           `je short 711E50F9`

711E50B3    66:3D 0025      `cmp ax, 2500`

711E50B7    74 40           `je short 711E50F9`

711E50B9    66:3D 0035      `cmp ax, 3500`

711E50BD    74 3A           `je short 711E50F9`

711E50BF    66:3D 000D      `cmp ax, 0D00`

711E50C3    74 34           `je short 711E50F9`

711E50C5    66:3D 001D      `cmp ax, 1D00`

711E50C9    74 2E           `je short 711E50F9`

711E50CB    66:3D 002D      `cmp ax, 2D00`

711E50CF    74 28           `je short 711E50F9`

711E50D1    66:3D 003D      `cmp ax, 3D00`

711E50D5    74 22           `je short 711E50F9`

711E50D7    66:8B03         `mov ax, word ptr ds:[ebx]`

711E50DA    66:25 00FF      `and ax, 0FF00`

711E50DE    66:3D 0080      `cmp ax, 8000`

711E50E2    0F82 A5000000   `jb 711E518D`

711E50E8    66:8B03         `mov ax, word ptr ds:[ebx]`

711E50EB    66:25 00FF      `and ax, 0FF00`

711E50EF    66:3D 00C0      `cmp ax, 0C000`

711E50F3    0F83 94000000   `jnb 711E518D`

711E50F9    66:8B03         `mov ax, word ptr ds:[ebx]`

711E50FC    66:25 FF00      `and ax, 0FF`

711E5100    66:3D 8900      `cmp ax, 89`

711E5104    74 0D           `je short 711E5113`

711E5106    66:8B03         `mov ax, word ptr ds:[ebx]`

711E5109    66:25 FF00      `and ax, 0FF`

711E510D    66:3D 8B00      `cmp ax, 8B`

711E5111    75 7A           `jnz short 711E518D`

711E5113    E8 FC7CFCFF     `call 711ACE14`

// 需要为下面的类型判断一下 ★ 统一放 `call 711ACE14` 里面 Patch！★

711E5118    66:8B13         `mov dx, word ptr ds:[ebx]`

711E511B    66:81E2 FF00    `and dx, 0FF`

711E5120    66:81FA 8900    `cmp dx, 89`

711E5125    75 12           `jnz short 711E5139`

711E5127    66:8B13         `mov dx, word ptr ds:[ebx]`

711E512A    66:81E2 00FF    `and dx, 0FF00`

711E512F    66:81C2 8B00    `add dx, 8B`

711E5134    66:8910         `mov word ptr ds:[eax], dx`

// 修改 9：`mov word ptr ds:[ebx], dx` ★

// 呵呵，替换 89->8B

711E5137    EB 10           `jmp short 711E5149`

711E5139    66:8B13         `mov dx, word ptr ds:[ebx]`

711E513C    66:81E2 00FF    `and dx, 0FF00`

711E5141    66:81C2 8900    `add dx, 89`

711E5146    66:8910         `mov word ptr ds:[eax], dx`

// 修改 10：`mov word ptr ds:[ebx], dx` ★

// 呵呵，替换 8B->89

711E5149    8B55 FC         `mov edx, dword ptr ss:[ebp-4]`

711E514C    8950 02         `mov dword ptr ds:[eax+2], edx`

711E514F    66:C740 06 90E9 `mov word ptr ds:[eax+6], 0E990`

// 修改 11：NOP ! ★

711E5155    8BD3            `mov edx, ebx`

711E5157    2BD0            `sub edx, eax`

711E5159    83EA 06         `sub edx, 6`

711E515C    8950 08         `mov dword ptr ds:[eax+8], edx`

711E515F    8BD0            `mov edx, eax`

711E5161    66:C703 90E9    `mov word ptr ds:[ebx], 0E990`

// 修改 12：NOP ! ★

711E5166    8B5D E8         `mov ebx, dword ptr ss:[ebp-18]`

711E5169    83C3 06         `add ebx, 6`

711E516C    8B45 E0         `mov eax, dword ptr ss:[ebp-20]`

711E516F    03C0            `add eax, eax`

711E5171    03D8            `add ebx, eax`

711E5173    8B45 E8         `mov eax, dword ptr ss:[ebp-18]`

711E5176    8B00            `mov eax, dword ptr ds:[eax]`

711E5178    0FB70B          `movzx ecx, word ptr ds:[ebx]`

711E517B    81E9 00300000   `sub ecx, 3000`

711E5181    03C1            `add eax, ecx`

711E5183    0345 DC         `add eax, dword ptr ss:[ebp-24]`

711E5186    2BD0            `sub edx, eax`

711E5188    83EA 04         `sub edx, 4`

711E518B    8910            `mov dword ptr ds:[eax], edx`

// 修改 13：NOP ! ★

711E518D    8B5D E8         `mov ebx, dword ptr ss:[ebp-18]`

711E5190    83C3 06         `add ebx, 6`

711E5193    8B45 E0         `mov eax, dword ptr ss:[ebp-20]`

711E5196    03C0            `add eax, eax`

711E5198    03D8            `add ebx, eax`

711E519A    66:C703 0000    `mov word ptr ds:[ebx], 0`

711E519F    FF4D E0         `dec dword ptr ss:[ebp-20]`

711E51A2    837D E0 00      `cmp dword ptr ss:[ebp-20], 0`

711E51A6    0F8F B2FDFFFF   `jg 711E4F5E`

// 循环

711E51AC    8B45 E8         `mov eax, dword ptr ss:[ebp-18]`

711E51AF    33D2            `xor edx, edx`

711E51B1    8910            `mov dword ptr ds:[eax], edx`

711E51B3    A1 9CDB1F71     `mov eax, dword ptr ds:[711FDB9C]`

711E51B8    8038 00         `cmp byte ptr ds:[eax], 0`

// 比较检验标志位！！

711E51BB    75 0A           `jnz short 711E51C7`

// 修改 14：NOP ! ★ 检验失败则这里跳转，则解码不完全！

711E51BD    837D F8 00      `cmp dword ptr ss:[ebp-8], 0`

711E51C1    0F8F 20FDFFFF   `jg 711E4EE7`

// 循环

### 3、统一放 `call 711ACE14` 里面 Patch！

711ACE14    53              `push ebx`

711ACE15    B8 0C000000     `mov eax, 0C`

711ACE1A    E8 4D59F7FF     `call 7112276C`

711ACE1F    8BD8            `mov ebx, eax`

711ACE21    85DB            `test ebx, ebx`

711ACE23    74 38           `je short 711ACE5D`

711ACE25    A1 68062071     `mov eax, dword ptr ds:[71200668]`

711ACE2A    E8 718DF7FF     `call 71125BA0`

711ACE2F    40              `inc eax`

711ACE30    50              `push eax`

711ACE31    B8 68062071     `mov eax, 71200668`

711ACE36    B9 01000000     `mov ecx, 1`

711ACE3B    8B15 10B51A71   `mov edx, dword ptr ds:[711AB510]`

711ACE41    E8 168FF7FF     `call 71125D5C`

711ACE46    83C4 04         `add esp, 4`

711ACE49    A1 68062071     `mov eax, dword ptr ds:[71200668]`

711ACE4E    E8 4D8DF7FF     `call 71125BA0`

711ACE53    8B15 68062071   `mov edx, dword ptr ds:[71200668]`

711ACE59    895C82 FC       `mov dword ptr ds:[edx+eax*4-4], ebx`

// Patch ：`jmp 71236E4D` ★

711ACE5D    8BC3            `mov eax, ebx`

711ACE5F    5B              `pop ebx`

711ACE60    C3              `retn`

为了防止异常，先在 `OllyDbg` 里面设置 `V220531.epe` 第 2 个区段为完整权限

在 `71236E4D` 处写入以下 patch 代码：

71236E4D    895C82 FC       `mov dword ptr ds:[edx+eax*4-4], ebx`

// 继续运行 `711ACE59` 处代码

71236E51    8BC3            `mov eax, ebx`

71236E53    5B              `pop ebx`

// 下面开始判断、修改

71236E54    66:813B 90E8    `cmp word ptr ds:[ebx], 0E890`

// 比较代码类型

71236E59    75 0D           `jnz short 71236E68`

71236E5B    66:C703 FF15    `mov word ptr ds:[ebx], 15FF`

// 是 `0E890` 就放入 `15FF`

71236E60    8B55 FC         `mov edx, dword ptr ss:[ebp-4]`

71236E63    8953 02         `mov dword ptr ds:[ebx+2], edx`

71236E66    EB 15           `jmp short 71236E7D`

71236E68    803B 89         `cmp byte ptr ds:[ebx], 89`

// `89` ？

71236E6B    74 10           `je short 71236E7D`

// 是则不处理

71236E6D    803B 8B         `cmp byte ptr ds:[ebx], 8B`

// `8B` ？

71236E70    74 0B           `je short 71236E7D`

// 是则不处理

71236E72    66:C703 FF25    `mov word ptr ds:[ebx], 25FF`

// 不是以上的就放入 `25FF`

71236E77    8B55 FC         `mov edx, dword ptr ss:[ebp-4]`

71236E7A    8953 02         `mov dword ptr ds:[ebx+2], edx`

// 放入地址

71236E7D    C3              `retn`

// OK，返回

`OllyDbg` 二进制代码如下：

```
89 5C 82 FC 8B C3 5B 66 81 3B 90 E8 75 0D 66 C7 03 FF 15 8B 55 FC 89 53 02 EB 15 80 3B 89 74 10

80 3B 8B 74 0B 66 C7 03 FF 25 8B 55 FC 89 53 02 C3
```

### 六、结局：完成脱壳

修改完以上所有部分后，在循环外面的 `711E51C7` 处设置硬件执行断点。

711E51C7    33C0            `xor eax, eax`

// 这里下硬件断点，`Shift+F9` 中断后所有解码完毕 ★

711E51C9    8945 C8         `mov dword ptr ss:[ebp-38], eax`

711E51CC    33C0            `xor eax, eax`

711E51CE    8945 CC         `mov dword ptr ss:[ebp-34], eax`

711E51D1    8B55 D4         `mov edx, dword ptr ss:[ebp-2C]`



```
711E51D4    8B45 D8         mov eax,dword ptr ss:[ebp-28]  

711E51D7    E8 B0D5F3FF     call 7112278C  

711E51DC    A1 9CDB1F71     mov eax,dword ptr ds:[711FDB9C]  

711E51E1    8038 00        cmp byte ptr ds:[eax],0  

711E51E4    74 05         je short 711E51EB  

711E51E6    E8 3D75F4FF     call 7112C728  

711E51EB    A1 CCDC1F71     mov eax,dword ptr ds:[711FDCCC]  

711E51F0    8038 00        cmp byte ptr ds:[eax],0  

711E51F3    74 17         je short 711E520C  

711E51F5    E8 01000000     call 711E51FB  

711E51FA    E9 5883C012     jmp 83DED557  

711E51FF    35 FFFFFFFF     xor eax,FFFFFFFF  

711E5204    60           pushad  

711E5205    E8 7E73FCFF     call 711AC588  

711E520A    FF15 E8010000   call dword ptr ds:[1E8]  

711E5210    00E9         add cl,ch  

711E5212    58           pop eax  

711E5213    83C0 0C        add eax,0C  

711E5216    60           pushad  

711E5217    E8 9072FCFF     call 711AC4AC  
```

记事本没有 `SDK`，不需要处理 `SDK` 代码部分。`OK`，可以用 `LordPE` 完全 `Dump` 出这个进程了。用第三步获得到的输入表直接修复就行了，若想做的好点，可以把代码段替换进第二步得到的 `dumped0.eXe` 再修复，这样就不必清除壳数据了。

一点题外话：当初看 `EncryptPE` 断断续续用了好几天，脱壳需要毅力和耐心，万事皆如此吧。

—————————————————————————————————

```
          ,     \_/  
        /| \_.-~/          \\\ ,       青春都一晌  
       ( /~   /             \~-.\ |\\  
       \`\\  \_/             \   ~\\ )         忍把浮名  
  _-~~~-.)  )_\_/;;,.        \\\\  //'  
 /'\_,\   --~   \ ~~~-  ,;;\\_\_\_(  (.-~~~-.        换了脱壳轻狂  
 \`~ _( ,\_..-\ (     ,;'' /    ~--  /.\_\`\\  
  /~~//'   / \`~\      ) /--..\_, )_  \`~  
  "  \`~"  "      `"      /~'\`\\   \`\\~~\\  
                 "     "   "~'  ""  
         UnPacKed By :  fly  
          2005-05-15 20:00  
```

`4dd466d0e1e0cc123afbe504da2462ca`
