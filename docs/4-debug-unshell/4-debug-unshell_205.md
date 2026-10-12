
# ARM Debug-Blocker+CopyMem-II+Enable Memory-Patching Protections

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-559.htm`](https://www.pediy.com/kssd/pediy08/pediy8-559.htm)

【破解日期】 2006 年 4 月 17 日  
【破解作者】 冷血书生`[OCN]` `[DFCG]`  
【作者邮箱】 `meiyou`  
【作者主页】 `meiyou`  
【使用工具】 `FlyOD`、`LordPE`、`ImportREC`  
【破解平台】 `Win9x`/`NT`/`2000`/`XP`  
【软件名称】 `FotoBatch` (魔法转换) 5.0.3  
【下载地址】 `http://www.onlinedown.net/soft/4139.htm`  
【软件简介】 `FotoBatch`(原名：魔法转换)是一个功能强大的图像批量转换、批量处理软件。使用它您可以制定图像处理脚本，然后根据脚本对图像进行批量转换。`FotoBatch` 提供超过 30 个图像处理脚本命令，比如调整图像大小、调整饱和度、增加半透明水印等。同时它还能创建可自动播放的 `EXE` 文件并可与您的家人朋友分享。它还支持 `WINDOWS` 右键菜单进行转换操作，并可以自定义 `WINDOWS` 右键菜单。它支持目前所有流行的图像格式，输出格式为：`bmp`、`jpg`、`gif`、`png`、`tif`、`pcx`、`wmf`、`emf`、`tga`、`ico`、`wbmp`，还包括 `txt`、`rtf`、`htm` 等字符画格式。  
【软件大小】 3846KB  
【加壳方式】 `Armadillo` 1.xx - 2.xx -> `Silicon` `Realms` `Toolworks` `[Overlay]`  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

--------------------------------------------------------------------------------

## 【破解内容】

本来是想找 `C++` 的 `RVA` 是怎么找的,无意中发现 `baby2008` 的脱壳求助,于是就 `DOWN` 下来看看,再于是,就有

了这篇菜菜的脱文,没啥含量,高手就跳过

### 一,寻找 `OEP`+解码 `Dump`

```
代码:

* * *

一,寻找 OEP+解码 Dump

00736363 >  55            push ebp                                ; OD 载入停在这里,下断 BP 
WaitForDebugEvent
00736364    8BEC          mov ebp,esp
00736366    6A FF         push -1
00736368    68 200B7600   push fotobatc.00760B20
0073636D    68 A0607300   push fotobatc.007360A0
00736372    64:A1 00000000  mov eax,dword ptr fs:[0]

////////////////////////////////////////////////////////////////////////////////////////////

7C849128 >  8BFF          mov edi,edi                             ; 中断在这里,取消断点
7C84912A    55            push ebp
7C84912B    8BEC          mov ebp,esp
7C84912D    83EC 68       sub esp,68
7C849130    56            push esi
////////////////////////////////////////////////////////////////////////////////////////////
堆栈:
0012BCB8   007264DF  /CALL 到 WaitForDebugEvent 来自 fotobatc.007264D9
0012BCBC   0012CD90  |pDebugEvent = 0012CD90      ---------> 在数据窗口跟随,接着下断：BP WriteProcessMemory
0012BCC0   000003E8  \Timeout = 1000. ms


////////////////////////////////////////////////////////////////////////////////////////////
0012BCB8   007264DF  /CALL 到 WaitForDebugEvent 来自 fotobatc.007264D9  -------> 在反汇编中跟随,Ctrl+F 在当前位置下搜索命令：
or eax,0FFFFFFF8,在其上找到 00726A63    83BD CCF5FFFF 0>cmp dword ptr
 ss:[ebp-A34],0 处设置断点
////////////////////////////////////////////////////////////////////////////////////////////
00726A63    83BD CCF5FFFF 0>cmp dword ptr ss:[ebp-A34],0 中断后取消断点,并把[ebp-A34]=[0012CD7C]=000002A8 清 0
00726A6A    0F8C A8020000   jl fotobatc.00726D18
00726A70    8B8D CCF5FFFF   mov ecx,dword ptr ss:[ebp-A34]
00726A76    3B0D 481F7600   cmp ecx,dword ptr ds:[761F48]   ★注意[761F48]   
00726A7C    0F8D 96020000   jge fotobatc.00726D18
00726A82    8B95 40F6FFFF   mov edx,dword ptr ss:[ebp-9C0]
00726A88    81E2 FF000000   and edx,0FF
00726A8E    85D2            test edx,edx
00726A90    0F84 AD000000   je fotobatc.00726B43
00726A96    6A 00           push 0
00726A98    8BB5 CCF5FFFF   mov esi,dword ptr ss:[ebp-A34]
00726A9E    C1E6 04         shl esi,4
00726AA1    8B85 CCF5FFFF   mov eax,dword ptr ss:[ebp-A34]
00726AA7    25 07000080     and eax,80000007
00726AAC    79 05           jns short fotobatc.00726AB3
00726AAE    48              dec eax
00726AAF    83C8 F8         or eax,FFFFFFF8   ★找到这里
00726AB2    40              inc eax
00726AB3    33C9            xor ecx,ecx
00726AB5    8A88 1CF97500   mov cl,byte ptr ds:[eax+75F91C]
00726ABB    8B95 CCF5FFFF   mov edx,dword ptr ss:[ebp-A34]
00726AC1    81E2 07000080   and edx,80000007
00726AC7    79 05           jns short fotobatc.00726ACE
00726AC9    4A              dec edx
00726ACA    83CA F8         or edx,FFFFFFF8
00726ACD    42              inc edx
00726ACE    33C0            xor eax,eax
00726AD0    8A82 1DF97500   mov al,byte ptr ds:[edx+75F91D]
00726AD6    8B3C8D 70B37500 mov edi,dword ptr ds:[ecx*4+75B370]
00726ADD    333C85 70B37500 xor edi,dword ptr ds:[eax*4+75B370]
00726AE4    8B8D CCF5FFFF   mov ecx,dword ptr ss:[ebp-A34]
00726AEA    81E1 07000080   and ecx,80000007
00726AF0    79 05           jns short fotobatc.00726AF7
00726AF2    49              dec ecx
00726AF3    83C9 F8         or ecx,FFFFFFF8
00726AF6    41              inc ecx
00726AF7    33D2            xor edx,edx
00726AF9    8A91 1EF97500   mov dl,byte ptr ds:[ecx+75F91E]
00726AFF    333C95 70B37500 xor edi,dword ptr ds:[edx*4+75B370]
00726B06    8B85 CCF5FFFF   mov eax,dword ptr ss:[ebp-A34]
00726B0C    99              cdq
00726B0D    B9 1C000000     mov ecx,1C
00726B12    F7F9            idiv ecx
00726B14    8BCA            mov ecx,edx
00726B16    D3EF            shr edi,cl
00726B18    83E7 0F         and edi,0F
00726B1B    03F7            add esi,edi
00726B1D    8B15 2C1F7600   mov edx,dword ptr ds:[761F2C]
00726B23    8D04B2          lea eax,dword ptr ds:[edx+esi*4]
00726B26    50              push eax
00726B27    8B8D CCF5FFFF   mov ecx,dword ptr ss:[ebp-A34]
00726B2D    51              push ecx
00726B2E    E8 2F210000     call fotobatc.00728C62
00726B33    83C4 0C         add esp,0C
00726B36    25 FF000000     and eax,0FF    ///这里开始修改
00726B3B    85C0            test eax,eax
00726B3D    0F84 D5010000   je fotobatc.00726D18
////////////////////////////////////////////////////////////////////////////////////////////
修改为: 
00726B36    FF85 CCF5FFFF   inc dword ptr ss:[ebp-0A34]
00726B3C    C705 4C1F7600 0>mov dword ptr ds:[761F4C],1
00726B46  ^ E9 18FFFFFF     jmp fotobatc.00726A63
////////////////////////////////////////////////////////////////////////////////////////////
修改完后,在 00726A7C 处跳转到 00726D18 下断点,接着就 Shift+F9,中断在上面 BP WriteProcessMemory 断点处,看数据窗口:
0012CD90  01 00 00 00 64 0B 00 00 FC 0A 00 00 01 00 00 80  ...d..?....€
0012CDA0  00 00 00 00 00 00 00 00 80 9E 6A 00 02 00 00 00  ........€瀓....
0012CDB0  00 00 00 00 80 9E 6A 00 80 9E 6A 00 01 00 00 00  ....€瀓.€瀓....
0012CDC0  00 00 00 00 00 00 00 00 13 00 00 00 88 FD 6D 81  ...........堼 m
看到 OEP=006A9E80
////////////////////////////////////////////////////////////////////////////////////////////
取消断点，继续 Shift+F9,中断在 00726D18 处,取消断点,运行 LordPE，完全 Dump 出子进程,修改脱壳后的 dumped 的 OEP 为 2A9E80

* * *

```

--------------------------------------------------------------------------------

--------------------------------------------------------------------------------



## 二、获取 RVA

```
006A9E80 >  55             push ebp
006A9E81    8BEC           mov ebp,esp
006A9E83    B9 13000000    mov ecx,13
006A9E88    6A 00          push 0
006A9E8A    6A 00          push 0
006A9E8C    49             dec ecx
006A9E8D  ^ 75 F9          jnz short dumped.006A9E88
006A9E8F    53             push ebx
006A9E90    56             push esi
006A9E91    57             push edi
006A9E92    B8 90986A00    mov eax,dumped.006A9890
006A9E97    E8 B0CFD5FF    call dumped.00406E4C               ; 这里跟进

//////////////////////////////////////////////////////////////////////////////////////////

00406E4C    53             push ebx
00406E4D    8BD8           mov ebx,eax
00406E4F    33C0           xor eax,eax
00406E51    A3 ACB06A00    mov dword ptr ds:[6AB0AC],eax
00406E56    6A 00          push 0
00406E58    E8 2BFFFFFF    call dumped.00406D88               ; 再跟进
//////////////////////////////////////////////////////////////////////////////////////////

00406D88  - FF25 78736E00  jmp dword ptr ds:[6E7378]         ; Ctrl+G--> 6E7378
00406D8E    8BC0           mov eax,eax
00406D90  - FF25 74736E00  jmp dword ptr ds:[6E7374]         ; kernel32.LocalAlloc
00406D96    8BC0           mov eax,eax
00406D98  - FF25 70736E00  jmp dword ptr ds:[6E7370]         ; kernel32.TlsGetValue
00406D9E    8BC0           mov eax,eax
00406DA0  - FF25 6C736E00  jmp dword ptr ds:[6E736C]         ; kernel32.TlsSetValue
//////////////////////////////////////////////////////////////////////////////////////////

006E7278  00 00 00 00 00 00 00 00  ........
006E7280  65 AE 96 7C FC F2 95 7C  e 畺|晐
006E7288  37 F3 95 7C E0 84 82 7C  7 髸|鄤倈
006E7290  FC 8C 82 7C C9 BE 82 7C  鼘倈删倈
006E7298  09 BC 82 7C 92 BB 82 7C  .紓|捇倈
006E72A0  75 FE 05 01 6D BC 82 7C  u?m 紓|
006E72A8  4F B4 82 7C 3B B4 82 7C  O 磦|;磦|
006E72B0  A7 8E 81 7C 10 DC 82 7C  亅軅|
006E72B8  64 6F 81 7C 7C BC 82 7C  do 亅|紓|
006E72C0  08 EF 82 7C 02 77 81 7C  飩|w 亅
006E72C8  25 C4 05 01 5F E1 82 7C  %?\_ 醾|
006E72D0  F6 1F 80 7C CD B3 05 01  ?€|统
006E72D8  9D F3 05 01 78 5F 82 7C  濗 x\_ 倈
006E72E0  D9 6D 82 7C C3 F4 95 7C  賛倈敏晐
006E72E8  81 FE 05 01 71 C8 05 01  侢 q?
006E72F0  77 EE 05 01 2D AC 82 7C  w?-瑐|
006E72F8  22 CE 05 01 26 CF 05 01  "?&?
006E7300  80 CF 05 01 5D DB 05 01  €?\]?
006E7308  1C 95 85 7C 31 DD 05 01  晠|1?
006E7310  A9 F8 82 7C 19 41 94 7C  倈 A 攟
006E7318  20 D9 05 01 AE 5D 81 7C   ?甝亅
006E7320  6F 07 82 7C E8 DD 05 01  o 倈栎
006E7328  24 DF 05 01 96 D4 05 01  $?栐
006E7330  A5 DE 05 01 1E BF 05 01  マ&?
006E7338  C5 DF E2 77 EC 82 E1 77  胚鈝靷醱
006E7340  26 F4 05 01 C7 A7 E1 77  &?千醱
006E7348  91 BF 05 01 8C 76 F4 77  懣寁魒
006E7350  1B 72 F4 77 70 DA F4 77  r 魒 p 隰 w
006E7358  E0 BE 05 01 10 3C 5F 77  嗑<\_w
006E7360  81 0F 60 77 D9 45 5F 77  ?\`w 貳\_w
006E7368  B8 BF 05 01 85 8C 82 7C  缚厡倈
006E7370  F0 BD 82 7C 92 BB 82 7C  鸾倈捇倈
006E7378  9D F3 05 01 E0 BE 05 01  濗嗑
```

在 `006E7278` 处已无数据，则 `RAV = 006E7278 - 40000 = 2E7278`，关闭 OD。

## 三、修复 DUMP 后的程序

重新载入原程序，下断：`BP DebugActiveProcess`，中断后看堆栈：

```
0012BCBC   0072633A  /CALL 到 DebugActiveProcess 来自 fotobatc.00726334
0012BCC0   00000D38  \ProcessId = D38  --------> 新开 OD,附加此进程
//////////////////////////////////////////////////////////////////////////////////////////
```

`F9`，再 `F12`，会暂停在 EP 处：

```
00736363 >- EB FE          JMP SHORT fotobatc.<ModuleEntryPoint>   ///恢复原来 EP 处的代码：55 8B
00736365     EC            IN AL,DX
00736366     6A FF         PUSH -1
00736368     68 200B7600   PUSH fotobatc.00760B20
//////////////////////////////////////////////////////////////////////////////////////////
```

切换单进程。`BP OpenMutexA`，`F9` 运行，注意看堆栈：

```
0012D7AC   00721F41  /CALL 到 OpenMutexA 来自 fotobatc.00721F3B
0012D7B0   001F0001  |Access = 1F0001
0012D7B4   00000000  |Inheritable = FALSE
0012D7B8   0012DDEC  \MutexName = "D38::DA525E6931"    ★注意 0012DDEC
//////////////////////////////////////////////////////////////////////////////////////////
```

`Ctrl+G`：`401000` 键入以下分离代码：

```
00401000    60             PUSHAD
00401001    9C             PUSHFD
00401002    68 ECDD1200    PUSH 12DDEC                     ; ASCII "D38::DA525E6931"
00401007    33C0           XOR EAX,EAX
00401009    50             PUSH EAX
0040100A    50             PUSH EAX
0040100B    E8 105A417C    CALL kernel32.CreateMutexA
00401010    9D             POPFD
00401011    61             POPAD
00401012  - E9 1062417C    JMP kernel32.OpenMutexA
```

二进制代码：
`60 9C 68 EC DD 13 00 33 C0 50 50 E8 2F DB 40 7C 9D 61 E9 04 DC 40 7C`

在 `401000` 处新建 EIP，`F9` 运行，再次中断在 `OpenMutexA` 处。返回 `401000` 处，“撤销选择”，清掉写入的代码。再取消 `OpenMutexA` 断点。

```
//////////////////////////////////////////////////////////////////////////////////////////
```

下断：`HE GetModuleHandleA`，`F9` 运行。

```
0012CFA8   71B110E9  /CALL 到 GetModuleHandleA 来自 WSOCK32.71B110E3
0012CFAC   71B12F8C  \pModule = "kernel32.dll"

0012D750   00721083  /CALL 到 GetModuleHandleA 来自 fotobatc.0072107D
0012D754   00000000  \pModule = NULL
0012D758   7C82B3C8  返回到 kernel32.7C82B3C8 来自 ntdll.ZwOpenMutant

00126884   01073637  /CALL 到 GetModuleHandleA 来自 01073631
00126888   01087474  \pModule = "kernel32.dll"
0012688C   01088744  ASCII "VirtualAlloc"

00126884   01073654  /CALL 到 GetModuleHandleA 来自 0107364E
00126888   01087474  \pModule = "kernel32.dll"
0012688C   01088738  ASCII "VirtualFree"

001265F4   0105ABB9  /CALL 到 GetModuleHandleA 来自 0105ABB3
001265F8   00126738  \pModule = "kernel32.dll"      -------------> 返回的好时机

//////////////////////////////////////////////////////////////////////////////////////////
```

```
0105ABB9     8B0D E4C90801   MOV ECX,DWORD PTR [108C9E4]                   ; 返回到这里
0105ABBF     89040E          MOV DWORD PTR [ESI+ECX],EAX
0105ABC2     A1 E4C90801     MOV EAX,DWORD PTR [108C9E4]
0105ABC7     391C06          CMP DWORD PTR [ESI+EAX],EBX
0105ABCA     75 16           JNZ SHORT 0105ABE2
0105ABCC     8D85 B4FEFFFF   LEA EAX,DWORD PTR [EBP-14C]
0105ABD2     50             PUSH EAX
0105ABD3     FF15 E0200801  CALL DWORD PTR [10820E0]                    ; kernel32.LoadLibraryA
0105ABD9     8B0D E4C90801   MOV ECX,DWORD PTR [108C9E4]
0105ABDF     89040E          MOV DWORD PTR [ESI+ECX],EAX
0105ABE2     A1 E4C90801     MOV EAX,DWORD PTR [108C9E4]
0105ABE7     391C06          CMP DWORD PTR [ESI+EAX],EBX
0105ABEA     E9 33010000     JMP 0105AD22                                ; Magic Jump,修改为 jmp 0105AD22
0105ABEF     90             NOP
0105ABF0     33C9           XOR ECX,ECX
```

清除所有的断点，`F9` 运行。

```
//////////////////////////////////////////////////////////////////////////////////////////
```

打开 ImportREC 选择当前进程
`OEP=2A9E80`
`RAV=2E7278`
`SIZE=1000`
直接点击获取输入表，CUT 掉无效的指针修复 dumped 即可以了，程序正常运行。

## 参考文献

1. Fly 的《Armadillo V4.X CopyMem-II 脱壳——魔法转换(Magic Converter) V4.0 正式版》，2005-04-28 发表于看雪学院
2. layper 的《Armadillo 1.xx - 2.xx 脱壳》，2005-07-21 发表于看雪学院

## 后记

如果想去掉运行后的那个框，可以在这里 `005A4ADF` 下个断点，可以获得一串注册码，这只是一个评估版本，即使获得注册码也多大用处。



【破解总结】

没啥好总结的，希望对和我一样菜的朋友有点帮助！

【版权声明】

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
