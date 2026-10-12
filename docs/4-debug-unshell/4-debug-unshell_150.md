
# ACProtect 脱壳——同益起名大师 V3.36

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-681.htm`](https://www.pediy.com/kssd/pediy07/pediy7-681.htm)

## 软件信息

下载页面：`http://www.skycn.com/soft/109.html`
软件大小：`6507 KB`
软件语言：简体中文
软件类别：国产软件 / 共享版 / 测字算命
应用平台：`Win9x/NT/2000/XP`
加入时间：`2005-02-04 12:01:23`
下载次数：`288768`
推荐等级：`*****`
开发商：`http://www.goodyour.com/`

**软件介绍**：是一个专业的起名测名软件，可以说是最优秀、最专业的，绝对 100% 精品。它有个人起名、公司行号命名、商标楼号命名、姓名八卦、吉号选择、姓名分析、名称分析、号码吉凶分析等及参考名字查询、成语查询、偏旁查字等多种活字典辞典功能。是姓名学爱好者及研究人员的得力工具，让您真正放心、方便、快捷地为您的公司商行或亲朋好友起个好名。

**【作者声明】**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教

**【调试环境】**：`WinXP`、`OllyDbg`、`PEiD`、`LordPE`

—————————————————————————————————

## 脱壳过程

同益起名大师加壳时使用了 `ACProtect` 专业版的几个特色功能，使得脱壳、修复时麻烦不少。

复制 `OllyDbg.exe` 重命名为 `cmd.exe` 或者 `eXpLorEr.exe`，这样就避开了 `ACProtect` 对父进程名检测。

设置 `OllyDbg` 忽略所有的异常选项。用 `IsDebug` 插件去掉 `OllyDbg` 的调试器标志。

—————————————————————————————————

### 一、壳中壳的 IAT

```
0067C000    60               pushad
//进入 OllyDbg 后暂停在这
0067C001    E8 01000000      call 0067C007
```

因为加壳时选择了 `ACProtect` 专业版功能之“使用 DRx 解码”，导致硬件断点不可随意使用。

没关系，我们可以用内存访问断点，或者在函数段末尾下普通断点。

`Ctrl+G`：`GetModuleHandleA` 设置内存访问断点

`Shift+F9` 运行，中断后取消断点。`Alt+F9` 返回

```
0068E9E2    0FB600           movzx eax,byte ptr ds:[eax]
//中断在这里
0068E9E5    83E8 33          sub eax,33
0068E9E8    3D 99000000      cmp eax,99
0068E9ED    74 10            je short 0068E9FF
0068E9EF    90               nop
0068E9F0    90               nop
0068E9F1    90               nop
0068E9F2    90               nop
0068E9F3    58               pop eax
0068E9F4    FF95 90E24100    call dword ptr ss:[ebp+41E290]
//[00699290]=7C80B529 (kernel32.GetModuleHandleA)
0068C51C    FF95 8CE24100    call dword ptr ss:[ebp+41E28C]
//[0069928C]=7C80AC28 (kernel32.GetProcAddress)
```

下面这段是处理壳所使用的输入表函数，因为 `EMbedded Protector` 要使用壳代码，所以我们要保留这些函数

`Ctrl+F` 在当前位置下搜索命令：`rep stos byte ptr es:[edi]`

找到在 `0068EC9A` 处，直接 `F4` 至 `0068EC9A`

```
0068EC8A    8DBD 5CFF4000    lea edi,dword ptr ss:[ebp+40FF5C]
0068EC90    8D8D CA004100    lea ecx,dword ptr ss:[ebp+4100CA]
0068EC96    2BCF             sub ecx,edi
0068EC98    33C0             xor eax,eax
0068EC9A    F3:AA            rep stos byte ptr es:[edi]
//清除 DLL、函数名            不让壳清除，NOP 掉 ★
0068EC9C    C3               retn
006916BC    E8 0CD3FFFF      call 0068E9CD
006916C1    E8 3EF9FFFF      call 00691004
//返回这里
```

记住 `call 0068E9CD` 是处理壳所使用函数的地方

—————————————————————————————————

### 二、搞定输入表

继续在 `GetModuleHandleA` 处设置内存访问断点

`Shift+F9` 运行，中断后取消断点。`Alt+F9` 返回

```
0069082F    8B95 1FFC4000    mov edx,dword ptr ss:[ebp+40FC1F]
00690835    8BB5 E0FC4000    mov esi,dword ptr ss:[ebp+40FCE0]
0069083B    03F2             add esi,edx
0069083D    8B46 0C          mov eax,dword ptr ds:[esi+C]
00690840    0BC0             or eax,eax
00690842    0F84 25020000    je 00690A6D
00690848    8366 0C 00       and dword ptr ds:[esi+C],0
//这里清空 ImageImportDescriptor 的 Name！     NOP 掉 ①  ★
```

当我们中断后返回 `0069086B` 时，这里已经运行过一次了，清除了第一个 Name 指针。

可以根据当时的寄存器情况来恢复这个指针。在 `0069086B` 时 `ESI＝00581000`，`[esi+C]`=`[0058100C]`=`00 00`，而 `EBX`＝`00581904` ASCII `"KERNEL32.dll"` 是第一个处理的 DLL 名，所以可以确定`[0058100C]`=`00181904`，修改之，否则 `DLLName` 会有错误 `★`

```
0069084C    03C2             add eax,edx
0069084E    8BD8             mov ebx,eax
00690850    56               push esi
00690851    57               push edi
00690852    50               push eax
00690853    8BF3             mov esi,ebx
00690855    8BFB             mov edi,ebx
00690857    AC               lods byte ptr ds:[esi]
00690858    C0C0 03          rol al,3
//解码出 DLL 名 ★
0069085B    AA               stos byte ptr es:[edi]
0069085C    803F 00          cmp byte ptr ds:[edi],0
0069085F    75 F6            jnz short 00690857
00690861    58               pop eax
00690862    5F               pop edi
00690863    5E               pop esi
00690864    50               push eax
00690865    FF95 90E24100    call dword ptr ss:[ebp+41E290]
0069086B    0BC0             or eax,eax
//返回这里  此时 ESI＝00581000-00400000＝00181000   输入表的 RVA  ★
0069086D    75 43            jnz short 006908B2
0069086F    90               nop
00690870    90               nop
00690871    90               nop
00690872    90               nop
00690873    53               push ebx
00690874    FF95 94E24100    call dword ptr ss:[ebp+41E294]
0069087A    0BC0             or eax,eax
0069087C    75 34            jnz short 006908B2
0069087E    90               nop
0069087F    90               nop
00690880    90               nop
00690881    90               nop
00690882    8B95 1FFC4000    mov edx,dword ptr ss:[ebp+40FC1F]
00690888    0195 1D1F4000    add dword ptr ss:[ebp+401F1D],edx
0069088E    0195 211F4000    add dword ptr ss:[ebp+401F21],edx
00690894    6A 00            push 0
00690896    FFB5 1D1F4000    push dword ptr ss:[ebp+401F1D]
0069089C    FFB5 211F4000    push dword ptr ss:[ebp+401F21]
006908A2    6A 00            push 0
006908A4    FF95 9CE24100    call dword ptr ss:[ebp+41E29C]
006908AA    6A 00            push 0
006908AC    FF95 98E24100    call dword ptr ss:[ebp+41E298]
006908B2    60               pushad
006908B3    2BC0             sub eax,eax
006908B5    8803             mov byte ptr ds:[ebx],al
//用完之后清空 DLL 名   NOP 掉 ②！ ★
006908B7    43               inc ebx
006908B8    3803             cmp byte ptr ds:[ebx],al
006908BA    75 F9            jnz short 006908B5
006908BC    61               popad
006908BD    8985 17FC4000    mov dword ptr ss:[ebp+40FC17],eax
//保存 DLL 基址
006908C3    C785 1BFC4000 0>mov dword ptr ss:[ebp+40FC1B],0
006908CD    8B95 1FFC4000    mov edx,dword ptr ss:[ebp+40FC1F]
006908D3    8B06             mov eax,dword ptr ds:[esi]
006908D5    0BC0             or eax,eax
006908D7    75 07            jnz short 006908E0
006908D9    90               nop
006908DA    90               nop
006908DB    90               nop
006908DC    90               nop
006908DD    8B46 10          mov eax,dword ptr ds:[esi+10]
006908E0    03C2             add eax,edx
006908E2    0385 1BFC4000    add eax,dword ptr ss:[ebp+40FC1B]
006908E8    8B18             mov ebx,dword ptr ds:[eax]
006908EA    8B7E 10          mov edi,dword ptr ds:[esi+10]
006908ED    03FA             add edi,edx
006908EF    03BD 1BFC4000    add edi,dword ptr ss:[ebp+40FC1B]
006908F5    85DB             test ebx,ebx
006908F7    0F84 62010000    je 00690A5F
006908FD    F7C3 00000080    test ebx,80000000
00690903    75 1D            jnz short 00690922
00690905    90               nop
00690906    90               nop
00690907    90               nop
00690908    90               nop
00690909    03DA             add ebx,edx
0069090B    83C3 02          add ebx,2
0069090E    56               push esi
0069090F    57               push edi
00690910    50               push eax
00690911    8BF3             mov esi,ebx
00690913    8BFB             mov edi,ebx
00690915    AC               lods byte ptr ds:[esi]
00690916    C0C0 03          rol al,3
//解码出函数名 ★
00690919    AA               stos byte ptr es:[edi]
0069091A    803F 00          cmp byte ptr ds:[edi],0
0069091D    75 F6            jnz short 00690915
0069091F    58               pop eax
00690920    5F               pop edi
00690921    5E               pop esi
00690922    3B9D 1FFC4000    cmp ebx,dword ptr ss:[ebp+40FC1F]
00690928    7C 11            jl short 0069093B
0069092A    90               nop
0069092B    90               nop
0069092C    90               nop
0069092D    90               nop
0069092E    83BD 02244000 0>cmp dword ptr ss:[ebp+402402],0
00690935    75 0A            jnz short 00690941
00690937    90               nop
00690938    90               nop
00690939    90               nop
0069093A    90               nop
```



```
0069093B    81E3 FFFFF0F   and ebx,0FFFFFFF  
00690941    53              push ebx  
00690942    FFB5 17FC4000   push dword ptr ss:[ebp+40FC17]  
00690948    FF95 8CE24100   call dword ptr ss:[ebp+41E28C]  
0069094E    3B9D 1FFC4000   cmp ebx,dword ptr ss:[ebp+40FC1F]  
00690954    7C 0F           jl short 00690965  
00690956    90              nop  
00690957    90              nop  
00690958    90              nop  
00690959    90              nop  
0069095A    60              pushad  
0069095B    2BC0            sub eax,eax  
0069095D    8803            mov byte ptr ds:[ebx],al  

//用完之后清空函数名   NOP 掉 ③！ ★

0069095F    43              inc ebx  
00690960    3803            cmp byte ptr ds:[ebx],al  
00690962    75 F9           jnz short 0069095D  
00690964    61              popad  
00690965    0BC0            or eax,eax  
00690967    0F84 15FFFFFF   je 00690882  
0069096D    3B85 9CE24100   cmp eax,dword ptr ss:[ebp+41E29C]  

//是否是 MessageBoxA ？EMbedded Protector 专用 APT 接口

00690973    74 20           je short 00690995  
00690975    90              nop  
00690976    90              nop  
00690977    90              nop  
00690978    90              nop  
00690979    3B85 9D014100   cmp eax,dword ptr ss:[ebp+41019D]  

//是否是 RegisterHotKey ？

0069097F    74 09           je short 0069098A  
00690981    90              nop  
00690982    90              nop  
00690983    90              nop  
00690984    90              nop  
00690985    EB 14           jmp short 0069099B  
00690987    90              nop  
00690988    90              nop  
00690989    90              nop  
0069098A    8D85 0A024100   lea eax,dword ptr ss:[ebp+41020A]  
00690990    EB 09           jmp short 0069099B  
00690992    90              nop  
00690993    90              nop  
00690994    90              nop  
00690995    8D85 24024100   lea eax,dword ptr ss:[ebp+410224]  
0069099B    56              push esi  
0069099C    FFB5 17FC4000   push dword ptr ss:[ebp+40FC17]  
006909A2    5E              pop esi  
006909A3    39B5 FA234000   cmp dword ptr ss:[ebp+4023FA],esi  

//比较是否是 Kernel32.DLL 基址

006909A9    74 15           je short 006909C0  
006909AB    90              nop  
006909AC    90              nop  
006909AD    90              nop  
006909AE    90              nop  
006909AF    39B5 FE234000   cmp dword ptr ss:[ebp+4023FE],esi  

//比较是否是 User32.DLL 基址

006909B5    74 09           je short 006909C0  
006909B7    90              nop  
006909B8    90              nop  
006909B9    90              nop  
006909BA    90              nop  
006909BB    EB 63           jmp short 00690A20  
006909BD    90              nop  
006909BE    90              nop  
006909BF    90              nop  
006909C0    80BD D2594100 0>cmp byte ptr ss:[ebp+4159D2],0  
006909C7    74 57           je short 00690A20  

//Magic Jump！ 如果用 ImportREC 修复输入表，则可以修改这里为：jmp 00690A20

006909C9    90              nop  
006909CA    90              nop  
006909CB    90              nop  
006909CC    90              nop  
006909CD    EB 07           jmp short 006909D6  

//下面就是加密了

006909CF    90              nop  
006909D0    90              nop  
006909D1    90              nop  
006909D2    0100            add dword ptr ds:[eax],eax  
006909D4    0000            add byte ptr ds:[eax],al  
006909D6    8BB5 E4FC4000   mov esi,dword ptr ss:[ebp+40FCE4]  
006909DC    83C6 0D         add esi,0D  
006909DF    81EE EA1B4000   sub esi,401BEA  
006909E5    2BF5            sub esi,ebp  
006909E7    83FE 00         cmp esi,0  
006909EA    7F 34           jg short 00690A20  
006909EC    90              nop  
006909ED    90              nop  
006909EE    90              nop  
006909EF    90              nop  
006909F0    8BB5 E4FC4000   mov esi,dword ptr ss:[ebp+40FCE4]  
006909F6    53              push ebx  
006909F7    50              push eax  
006909F8    E8 A3B2FFFF     call 0068BCA0  
006909FD    8BD8            mov ebx,eax  
006909FF    58              pop eax  
00690A00    33C3            xor eax,ebx  

//函数加密只是简单异或

00690A02    C606 68         mov byte ptr ds:[esi],68  
00690A05    8946 01         mov dword ptr ds:[esi+1],eax  
00690A08    C746 05 8134240>mov dword ptr ds:[esi+5],243481  
00690A0F    895E 08         mov dword ptr ds:[esi+8],ebx  
00690A12    C646 0C C3      mov byte ptr ds:[esi+C],0C3  
00690A16    5B              pop ebx  
00690A17    8BC6            mov eax,esi  
00690A19    8385 E4FC4000 0>add dword ptr ss:[ebp+40FCE4],0D  
00690A20    5E              pop esi  
00690A21    60              pushad  
00690A22    8BD0            mov edx,eax  
00690A24    2BBD 1FFC4000   sub edi,dword ptr ss:[ebp+40FC1F]  
00690A2A    8BC7            mov eax,edi  
00690A2C    B9 01010000     mov ecx,101  
00690A31    8DBD D3F04000   lea edi,dword ptr ss:[ebp+40F0D3]  
00690A37    F2:AF           repne scas dword ptr es:[edi]  
00690A39    0BC9            or ecx,ecx  
00690A3B    74 13           je short 00690A50  
00690A3D    90              nop  
00690A3E    90              nop  
00690A3F    90              nop  
00690A40    90              nop  
00690A41    81E9 01010000   sub ecx,101  
00690A47    F7D1            not ecx  
00690A49    89948D D3EC4000 mov dword ptr ss:[ebp+ecx*4+40ECD3],edx  

//注意[ebp+ecx*4+40ECD3]从 00689CD3 开始到 0068A0D3 结束  ★  Size=400

00690A49 处就是处理“API 定向到一个地址”的地方！其实这里和 0058117C 至 0058159C 之间的函数是一样的（去除里面的 00000000）。修复代码见最后一节。

00690A50    61              popad  
00690A51    8907            mov dword ptr ds:[edi],eax  

//API 函数的系统地址（或者加密地址）填充到 IAT 中    NOP 掉！④ ★

00690A53    8385 1BFC4000 0>add dword ptr ss:[ebp+40FC1B],4  
00690A5A    E9 6EFEFFFF     jmp 006908CD  
00690A5F    83C6 14         add esi,14  
00690A62    8B95 1FFC4000   mov edx,dword ptr ss:[ebp+40FC1F]  
00690A68    E9 D0FDFFFF     jmp 0069083D  

//循环处理

00690A6D    8DBD D3F04000   lea edi,dword ptr ss:[ebp+40F0D3]  

//修改上面 4 处后直接 F4 到这里   输入表处理完毕了

00690A73    33C0            xor eax,eax  
00690A75    B9 00010000     mov ecx,100  
00690A7A    F3:AB           rep stos dword ptr es:[edi]  
00690A7C    60              pushad  
00690A7D    E8 00000000     call 00690A82
```

运行 `LordPE` 完全 Dump 出这个进程，修正 `dumped.exe` 的 `Import Table RVA=00181000`

### 三、OEP Stolen Code

`Alt+M` 打开内存查看窗口，在 `401000` 第二区段上设置内存访问断点，`Shift+F9` 运行

```
004067F4    53              push ebx  

//中断在这里，取消断点

004067F5    8BD8            mov ebx,eax  
004067F7    33C0            xor eax,eax  
004067F9    A3 9CB05700     mov dword ptr ds:[57B09C],eax  
004067FE    6A 00           push 0  
00406800    E8 25542800     call 0068BC2A  
00406805    A3 68065800     mov dword ptr ds:[580668],eax  
0040680A    A1 68065800     mov eax,dword ptr ds:[580668]  
0040680F    A3 A8B05700     mov dword ptr ds:[57B0A8],eax  
00406814    33C0            xor eax,eax  
00406816    A3 ACB05700     mov dword ptr ds:[57B0AC],eax  
0040681B    33C0            xor eax,eax  
0040681D    A3 B0B05700     mov dword ptr ds:[57B0B0],eax  
00406822    E8 C1FFFFFF     call 004067E8  
00406827    BA A4B05700     mov edx,57B0A4  
0040682C    8BC3            mov eax,ebx  
0040682E    E8 9DD7FFFF     call 00403FD0  
00406833    5B              pop ebx  
00406834    C3              retn
```

很明显，这是 `Delphi` 入口后的第一个 `CALL` 代码，`OEP` 处的代码已经被 `Stolen` 了

此时寄存器和堆栈情况如下：

```
EAX 0057A00C GoodName.0057A00C  
ECX 0013FFB0  
EDX 7C92EB94 ntdll.KiFastSystemCallRet  
EBX 7FFDF000  
ESP 0013FFA4 ASCII "jvi"  
EBP 0013FFC0  
ESI 00000020  
EDI 00791000  
EIP 004067F4 GoodName.004067F4  

0013FFA4    0069766A  返回到 GoodName.0069766A 来自 GoodName.004067F4  
0013FFA8    00000020  
0013FFAC    7FFDF000  
0013FFB0    7C92EB94  ntdll.KiFastSystemCallRet  
0013FFB4    00000020  
0013FFB8    00000020  
0013FFBC    00791000
```



`0013FFC0` `0013FFF0`

————————————————————————

`Ctrl+G`：`0069766A` 处看看：

```
0069761B    8B05 11CC6700   mov eax,dword ptr ds:[67CC11]; GoodName.0067CDC5
00697621    8F05 85CD6700   pop dword ptr ds:[67CD85]
00697627    8B0D 85CD6700   mov ecx,dword ptr ds:[67CD85]
0069762D    8930            mov dword ptr ds:[eax],esi
0069762F    8F05 D9CC6700   pop dword ptr ds:[67CCD9]    ; GoodName.0057A00C
00697635    56              push esi
00697636    BE D9CC6700     mov esi,67CCD9
0069763B    8B06            mov eax,dword ptr ds:[esi]
0069763D    5E              pop esi
0069763E    8F05 75CD6700   pop dword ptr ds:[67CD75]
00697644    890D E9CE6700   mov dword ptr ds:[67CEE9],ecx
0069764A    FF35 E9CE6700   push dword ptr ds:[67CEE9]
00697650    68 75CD6700     push 67CD75
00697655    59              pop ecx
00697656    8B31            mov esi,dword ptr ds:[ecx]
00697658    8F05 B9CC6700   pop dword ptr ds:[67CCB9]
0069765E    8B0D B9CC6700   mov ecx,dword ptr ds:[67CCB9]
00697664    FF15 C5CD6700   call dword ptr ds:[67CDC5]   ; GoodName.004067F4
0069766A    90              nop
```

————————————————————————

结合 Delphi 程序的一般入口特征代码分析，可以分析出 `OEP` 处的前面几行代码：

```
  push ebp
  mov ebp,esp
  add esp,-10
  push ebx
  push esi
  mov eax,0057A00C
  call 004067F4
```

`Ctrl+F9` 返回 `0069766A` 后继续在 `401000` 第二区段上设置内存访问断点，`Shift+F9` 运行

```
0057A4BD    E8 72C5E8FF     call 00406A34 

//中断这里

0057A4C2    8BD8            mov ebx,eax
0057A4C4    85DB            test ebx,ebx
0057A4C6    74 17           je short 0057A4DF
```

可以看出下面已经是主干代码了，`OEP` 处的 `Stolen Code` 到此结束

寄存器和堆栈情况如下：

`EAX 00000000`

`ECX 0013FF94`

`EDX 0013FFB4`

`EBX 7FFDA000`

`ESP 0013FF9C`

`EBP 0013FFC0`

`ESI 00580C0C GoodName.00580C0C`

`EDI 00000040`

`EIP 0057A4BD GoodName.0057A4BD`

`0013FF9C   00000000`

`0013FFA0   FFFFFFFF`

`0013FFA4   0057A530  ASCII "GoodName"`

分析出这几行代码：

```
  mov esi,dword ptr ds:[00XXXXXX]; 00580C0C
  push 0057A530
  push -1
  push 0
```

如何确定`[00XXXXXX]`？可以在 `CODE` 和 `DATA` 段内搜索 `0C0C5800`，一般在 `CODE` 末尾或者 `DATA` 段开始

找到：`0057F018 0C 0C 58 00`

`OEP` 处 `Stolen Code` 修复如下：

```
0057A49C    55              push ebp
0057A49D    8BEC            mov ebp,esp
0057A49F    83C4 F0         add esp,-10
0057A4A2    53              push ebx
0057A4A3    56              push esi
0057A4A4    B8 0CA05700     mov eax,57A00C
0057A4A9    E8 46C3E8FF     call 004067F4 
0057A4AE    8B35 18F05700   mov esi,dword ptr ds:[57F018]
0057A4B4    68 30A55700     push 57A530 ; ASCII "GoodName"
0057A4B9    6A FF           push -1
0057A4BB    6A 00           push 0
0057A4BD    E8 72C5E8FF     call 00406A34
```

用 `LordPE` 修正 `dumped.exe` 的 `OEP RVA=0017A49C`

—————————————————————————————————

### 四、修复壳中初始化数据之 1

现在脱壳文件是无法运行的。设置 `OllyDbg` 不忽略内存访问异常选项，载入 `dumped.exe`

```
0067D612    8BBD 23FC4000   mov edi,dword ptr ss:[ebp+40FC23]
0067D618    B8 0A000000     mov eax,0A
0067D61D    F7E1            mul ecx
0067D61F    03F8            add edi,eax
0067D621    8A9D 06244000   mov bl,byte ptr ss:[ebp+402406]
0067D627    B9 0A000000     mov ecx,0A
0067D62C    AC              lods byte ptr ds:[esi]
0067D62D    32C3            xor al,bl
0067D62F    AA              stos byte ptr es:[edi]
0067D630    E2 FA           loopd short 0067D62C 

//访问违反： 写入到 [00143C10]
```

在数据窗口 `Ctrl+G`：`ebp+40FC23`

```
0068AC23   E8 36 14 00  
```

这个地方的数据在壳中初始化过了，现在要找到哪里进行初始化的。

载入原版 `GoodName.exe`，在 `0068AC23` 处设置内存写入断点，`Shift+F9`

```
0067CB7F    8937            mov dword ptr ds:[edi],esi

//中断第 1 次

0068E938    FF95 D8004100   call dword ptr ss:[ebp+4100D8]

//申请内存

0068E93E    8985 23FC4000   mov dword ptr ss:[ebp+40FC23],eax

//中断第 2 次，申请到的内存地址写入[ebp+40FC23]  ★

0068E944    59              pop ecx
0068E945    58              pop eax
0068E946    0385 1FFC4000   add eax,dword ptr ss:[ebp+40FC1F]
0068E94C    8BF0            mov esi,eax
0068E94E    50              push eax
0068E94F    8BBD 23FC4000   mov edi,dword ptr ss:[ebp+40FC23]
0068E955    F3:A4           rep movs byte ptr es:[edi],byte ptr ds:[esi]
0068E957    58              pop eax
0068E958    50              push eax
0068E959    FFB5 23FC4000   push dword ptr ss:[ebp+40FC23]
0068E95F    E8 CBA50000     call 00698F2F
0068E964    FFB5 23FC4000   push dword ptr ss:[ebp+40FC23]
0068E96A    FF95 DC004100   call dword ptr ss:[ebp+4100DC]
0068E970    5E              pop esi
0068E971    83C6 08         add esi,8
0068E974    EB B1           jmp short 0068E927
```

`Ctrl+F9` 返回 `006916CB` 处

```
006916C6    E8 9FD0FFFF     call 0068E76A

//这里面申请内存

006916CB    E8 42FBFFFF     call 00691212

//返回这里

0068E76A    C3              retn

//壳运行过会把这里修改为 C3，因此我们要重新跟踪得到此处的代码：60  修改之
```

在入口修复代码里要添加：`call 0068E76A`

—————————————————————————————————

### 五、修复 EMbedded Protector

用 `LordPE` 修正 `dumped.exe` 的 `OEP RVA=0027C000`，键入以下代码：

```
0067C000    60              pushad
0067C001    E8 64270100     call 0068E76A

//修复壳中初始化数据之 1

0067C006    61              popad
0067C007    E9 90E4EFFF     jmp 0057A49C
```

`BP MessageBoxA Shift+F9`，中断下来

```
0013FD2C   00570121   CALL 到 MessageBoxA 来自 dumped.0057011C

00570112    61              popad
00570113    60              pushad
00570114    6A 05           push 5
00570116    6A 00           push 0
00570118    6A 00           push 0
0057011A    6A FF          push -1
0057011C    E8 8372E9FF     call 004073A4 ; <jmp.&user32.MessageBoxA>

//EMbedded Protector

00570121    61              popad
00570122    90              nop
00570123    60              pushad
00570124    40              inc eax
00570125    87D0            xchg eax,edx

004073A4    FF25 2C165800   jmp dword ptr ds:[58162C]; user32.MessageBoxA

//跟踪原版，得出这里的值＝0068B224  ★
```

—————————————————————————————————

### 六、修复壳中初始化数据之 2

```
0068BB64    8BB48D 251F4000 mov esi,dword ptr ss:[ebp+ecx*4+401F25]
0068BB6B    03B5 1FFC4000   add esi,dword ptr ss:[ebp+40FC1F]
0068BB71    8B948D B5204000 mov edx,dword ptr ss:[ebp+ecx*4+4020B5]
0068BB78    8BBC8D 45224000 mov edi,dword ptr ss:[ebp+ecx*4+402245]
0068BB7F    87CA            xchg edx,ecx
0068BB81    F3:A4           rep movs byte ptr es:[edi],byte ptr ds:[esi]

//访问违反： 写入到 [0019C420]
```

在数据窗口 `Ctrl+G`：`ebp+eDx*4+402245`

```
0067D2A9  20 C4 19 00
```

调试原版 `GoodName.exe`，在 `0067D2A9` 处设置内存写入断点，`Shift+F9`

```
00691426    60              pushad
00691427    E8 A6B0FFFF     call 0068C4D2
0069142C    33D2            xor edx,edx
0069142E    8BB495 251F4000 mov esi,dword ptr ss:[ebp+edx*4+401F25]
00691435    0BF6            or esi,esi
00691437    74 31           je short 0069146A
00691439    90              nop
0069143A    90              nop
0069143B    90              nop
0069143C    90              nop
0069143D    03B5 1FFC4000   add esi,dword ptr ss:[ebp+40FC1F]
00691443    8B8C95 B5204000 mov ecx,dword ptr ss:[ebp+edx*4+4020B5]
0069144A    60              pushad
0069144B    52              push edx
0069144C    51              push ecx
0069144D    6A 40           push 40
0069144F    FF95 D8004100   call near dword ptr ss:[ebp+4100D8]

//申请内存

00691455    5A              pop edx
00691456    898495 45224000 mov dword ptr ss:[ebp+edx*4+402245],eax

//中断第 2 次，申请到的内存地址写入[ebp+edx*4+402245]  ★

0069145D    61              popad
0069145E    8BBC95 45224000 mov edi,dword ptr ss:[ebp+edx*4+402245]
00691465    F3:A4           rep movs byte ptr es:[edi],byte ptr ds:[esi]
00691467    42              inc edx
00691468    EB C4           jmp short 0069142E
0069146A    61              popad
0069146B    C3              retn
```

在入口修复代码里要添加：`call 00691426`

—————————————————————————————————

### 七、最终修复代码

```
0067C000    60              pushad
0067C001    36:A1 D0115800  mov eax,dword ptr ss:[5811D0]
```



```asm
//GetModuleHandleA
0067C007    36:A3 90926900  mov dword ptr ss:[699290],eax
0067C00D    A3 10B16800     mov dword ptr ds:[68B110],eax
0067C012    36:A1 CC115800  mov eax,dword ptr ss:[5811CC]

//GetProcAddress
0067C018    36:A3 8C926900  mov dword ptr ss:[69928C],eax
0067C01E    36:A3 0CB16800  mov dword ptr ss:[68B10C],eax
0067C024    E8 A4290100     call 0068E9CD

//处理“壳中之壳”所使用的函数
0067C029    E8 3C270100     call 0068E76A

//修复壳中初始化数据之 1
0067C02E    E8 F3530100     call 00691426

//修复壳中初始化数据之 2
0067C033    BF D39C6800     mov edi,689CD3
0067C038    BE 7C115800     mov esi,58117C
0067C03D    8B06            mov eax,dword ptr ds:[esi]
0067C03F    85C0            test eax,eax
0067C041    74 05           je short 0067C048
0067C043    8907            mov dword ptr ds:[edi],eax
0067C045    83C7 04         add edi,4
0067C048    83C6 04         add esi,4
0067C04B    81FF D3A06800   cmp edi,68A0D3
0067C051    75 EA           jnz short 0067C03D

//上面这个循环是把 0058117C－0058159C 之间的函数（去除 00000000）复制到 00689CD3－0068A0D3
//修复“API 定向到一个地址”所用到的函数
0067C053    36:A1 D4125800  mov eax,dword ptr ss:[5812D4]

//LoadLibraryA   跨系统平台测试时发现这几个地方还需要修复
0067C059    36:A3 14B16800  mov dword ptr ss:[68B114],eax
0067C05F    36:A3 94926900  mov dword ptr ss:[699294],eax
0067C065    36:A1 EC115800  mov eax,dword ptr ss:[5811EC]

//ExitProcess
0067C06B    36:A3 18B16800  mov dword ptr ss:[68B118],eax
0067C071    36:A3 98926900  mov dword ptr ss:[699298],eax
0067C077    36:A1 2C165800  mov eax,dword ptr ss:[58162C]

//MessageBoxA
0067C07D    36:A3 9C926900  mov dword ptr ss:[69929C],eax
0067C083    36:C705 2C16580>mov dword ptr ss:[58162C],68B224

//EMbedded Protector
0067C08E    36:C705 639D680>mov dword ptr ss:[689D63],68B224

//EMbedded Protector
0067C099    61              popad
0067C09A    E9 FDE3EFFF     jmp 0057A49C

//返回 OEP 继续执行，OK！
```

—————————————————————————————————

```
             ,       \_/ 
           /| \_.-~/         \\ \ ,        青春都一晌
          ( /~   /            \\~.\ |\\ 
          `\\   \_/            \   ~\ )             忍把浮名
   _-~~~-.  )\_//;;,.          \\   // /'
  /'\_ \ --~   \ ~~~- ,;;\_\_(  (.-~~~-.        换了破解轻狂
  \`~ _( ,_..\ (      ,;'' /    ~--  /.\_\ \
   /~~//' / \`~\       ) /--..\, )_ \`~
   "  \`"  "     \`"    /~'\ \    \`\\~~\ 
                   "   "   "~'  ""
             UnPacKed By :  fly
                2004-04-24 零点
```

`4dd466d0e1e0cc123afbe504da2462ca`
