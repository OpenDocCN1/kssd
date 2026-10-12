
# 用 Ollydbg 手脱 SafeDisc V2.43.000 加壳的 DLL

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-768.htm`](https://www.pediy.com/kssd/pediy07/pediy7-768.htm)

用 `Ollydbg` 手脱 `SafeDisc` `V2.43.000` 加壳的 `DLL`

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`Ollydbg` `V1.10`、`PEiD`、`LordPE`、`ImportREC`、`WinHex`

—————————————————————————————————

【脱壳过程】：

`SafeDisc` 是著名的光碟保护软件，现在也单独加壳 `PE` 文件了。

本教程演示的 `AdobeLM.dll` `V1.0.2.38` 是 `Adobe` `Premiere` `Pro` `V1.5` 的文件，`SafeDisc` `V2.43.000` 保护，不是最新版，况且一般壳保护 `DLL` 时强度都要降低，`SafeDisc` 对此 `DLL` 没有使用驱动和 `CC` 解码，因此相对来说难度不高。但是 `SafeDisc` 的输入表和 `SDK` 修复比较麻烦。

此文件以前 `jingulong` 搞定过，感谢 `heXer` 和 `shoooo` 的帮忙，`SDK` 由 `heXer` 修复。

由于没有主程序测试，可能还会有其他隐藏的暗桩。

此教程不适合新手练习。断断续续整理了很长时间，作为 2005 年的结笔吧，光阴似箭。

—————————————————————————————————

一、准备动作

设置 `Ollydbg` 忽略所有的异常选项。用 `IsDebug` 插件去掉 `OllyDBD` 的调试器标志。

```
1005A05E      55               push ebp
1005A05F      8BEC             mov ebp,esp
1005A061      60               pushad
1005A062      BB 5EA00510      mov ebx,1005A05E
1005A067      33C9             xor ecx,ecx
1005A069      8A0D 3DA00510    mov cl,byte ptr ds:[1005A03D]
1005A06F      85C9             test ecx,ecx
1005A071      74 0C            je short 1005A07F
1005A073      B8 D3A00510      mov eax,1005A0D3
1005A078      2BC3             sub eax,ebx
1005A07A      83E8 05          sub eax,5
1005A07D      EB 0E            jmp short 1005A08D
1005A07F      51               push ecx
1005A080      B9 19A10510      mov ecx,1005A119
1005A085      8BC1             mov eax,ecx
1005A087      2BC3             sub eax,ebx
1005A089      0341 01          add eax,dword ptr ds:[ecx+1]
1005A08C      59               pop ecx
1005A08D      C603 E9          mov byte ptr ds:[ebx],0E9
1005A090      8943 01          mov dword ptr ds:[ebx+1],eax
1005A093      51               push ecx
1005A094      68 09A00510      push 1005A009
1005A099      33C0             xor eax,eax
1005A09B      85C9             test ecx,ecx
1005A09D      74 05            je short 1005A0A4
1005A09F      8B45 08          mov eax,dword ptr ss:[ebp+8]
1005A0A2      EB 00            jmp short 1005A0A4
1005A0A4      50               push eax
1005A0A5      E8 76000000      call 1005A120
//壳处理
1005A0AA      83C4 08          add esp,8
1005A0AD      59               pop ecx
1005A0AE      83F8 00          cmp eax,0
1005A0B1      74 1C            je short 1005A0CF
1005A0B3      C603 C2          mov byte ptr ds:[ebx],0C2
1005A0B6      C643 01 0C       mov byte ptr ds:[ebx+1],0C
1005A0BA      85C9             test ecx,ecx
1005A0BC      74 09            je short 1005A0C7
1005A0BE      61               popad
1005A0BF      5D               pop ebp
1005A0C0      B8 00000000      mov eax,0
1005A0C5      EB 97            jmp short 1005A05E
1005A0C7      50               push eax
1005A0C8      A1 29A00510      mov eax,dword ptr ds:[1005A029]
1005A0CD      FFD0             call eax
1005A0CF      61               popad
1005A0D0      5D               pop ebp
1005A0D1      EB 46            jmp short 1005A119
1005A0D3      807C24 08 00     cmp byte ptr ss:[esp+8],0
1005A0D8      75 3F            jnz short 1005A119
1005A0DA      51               push ecx
1005A0DB      8B4C24 04        mov ecx,dword ptr ss:[esp+4]
1005A0DF      890D 13A10510    mov dword ptr ds:[1005A113],ecx
1005A0E5      B9 F1A00510      mov ecx,1005A0F1
1005A0EA      894C24 04        mov dword ptr ss:[esp+4],ecx
1005A0EE      59               pop ecx
1005A0EF      EB 28            jmp short 1005A119
1005A0F1      50               push eax
1005A0F2      B8 2DA00510      mov eax,1005A02D
1005A0F7      FF70 08          push dword ptr ds:[eax+8]
1005A0FA      8B40 0C          mov eax,dword ptr ds:[eax+C]
1005A0FD      FFD0             call eax
1005A0FF      B8 2DA00510      mov eax,1005A02D
1005A104      FF30             push dword ptr ds:[eax]
1005A106      8B40 04          mov eax,dword ptr ds:[eax+4]
1005A109      FFD0             call eax
1005A10B      58               pop eax
1005A10C      FF35 13A10510    push dword ptr ds:[1005A113]
1005A112      C3               retn
1005A119      E9 A6CEFCFF      jmp 10026FC4
//飞向光明之巅
```

—————————————————————————————————

二、Anti

`SafeDisc` `V2.43.000` 对 `OllyDBG` 的反调试还是很少的，下面没有分析对 `SoftICE` 的 `Anti`

`SafeDisc` 会在你的 `Temp` 目录下生成`~df394b.tmp`，这个其实是 `SecServ.dll`，里面 `Anti`

————————————————————————

1、`IsDebuggerPresent`

`BP` `IsDebuggerPresent`

`Shift+F9`，中断后取消断点，`Alt+F9`

```
00879ACA      FFD0             call eax
00879ACC      8BF0             mov esi,eax
//返回这里
00879ACE      66:85F6          test si,si
00879AD1      74 13            je short 00879AE6
//IsDebuggerPresent 检测
00879AD3      E8 A277FFFF      call 0087127A
00879AD8      66:8BF0          mov si,ax
00879ADB      66:F7DE          neg si
00879ADE      1BF6             sbb esi,esi
00879AE0      46               inc esi
00879AE1      66:85F6          test si,si
00879AE4      75 13            jnz short 00879AF9
00879AE6      8B4424 08        mov eax,dword ptr ss:[esp+8]
00879AEA      8B08             mov ecx,dword ptr ds:[eax]
00879AEC      81E1 EA894267   and ecx,674289EA
00879AF2      8908             mov dword ptr ds:[eax],ecx
00879AF4      66:8BC6          mov ax,si
00879AF7      5E               pop esi
00879AF8      C3               retn
```

因为我们已经使用 `IsDebug` 插件了，所以这里不必处理，只是借用此断点来继续下面的流程。

————————————————————————

2、`ZwQueryInformationProcess`

`BP` `GetCurrentProcess`

`Shift+F9`，中断后取消断点，`Alt+F9`

```
00879889     FF15 B4208C00    call dword ptr ds:[8C20B4] ; kernel32.GetCurrentProcess
//返回这里
0087988F     50               push eax
00879890     FFD7             call edi                     ; ntdll.ZwQueryInformationProcess
00879892     8B4424 0C        mov eax,dword ptr ss:[esp+C]
//检测 DebugPort，修改[esp+0C]处为 0，或者在下面跳转处改标志位
00879896     85C0             test eax,eax
00879898     75 02            jnz short 0087989C
//跳则 Over
```

————————————————————————

3、普通断点检测

脱壳时一般要有好的习惯，用过的断点要立刻清除。在此壳中不要留有普通断点。

如果上面我们没有清掉 `API` 断点，那么这里会检测到

```
00879520     53               push ebx
00879521     8B19             mov ebx,dword ptr ds:[ecx]
00879523     803C03 CC        cmp byte ptr ds:[ebx+eax],0CC
//自 kernel32.dll 各函数入口检测普通 INT3
00879527     75 01            jnz short 0087952A
00879529     46               inc esi
0087952A     83C1 04          add ecx,4
0087952D     4A               dec edx
0087952E     75 F1            jnz short 00879521
00879530     5B               pop ebx
00879531     33D2             xor edx,edx
00879533     3BD6             cmp edx,esi
00879535     5E               pop esi
00879536     1BC0             sbb eax,eax
00879538     F7D8             neg eax
0087953A     66:85C0          test ax,ax
//AX 要＝0
0087953D     75 12            jnz short 00879551
```

好了，`SafeDisc` `V2.43.000` 对 `OllyDBG` 的 `Anti` 手段就这么点

稍有威胁的是 `ZwQueryInformationProcess`，去除 `Anti` 不难

—————————————————————————————————

三、获取正确的函数地址

下面用脚本来演示这部分的处理。

重新加载 `AdobeLM.dll`，运行 `SafeDisc` `V2.43.000.osc` 脚本。脚本运行完毕后 `OllyDBG` 自动暂停在 `OEP`

```
10026FC4     55               push ebp
//OEP
10026FC5     8BEC             mov ebp,esp
10026FC7     53               push ebx
10026FC8     8B5D 08          mov ebx,dword ptr ss:[ebp+8]
10026FCB     56               push esi
10026FCC     8B75 0C          mov esi,dword ptr ss:[ebp+C]
10026FCF     57               push edi
10026FD0     8B7D 10          mov edi,dword ptr ss:[ebp+10]
10026FD3     85F6             test esi,esi
```



```
10026FD5     75 09                jnz short 10026FE0  

10026FD7     833D 80AE0410 00     cmp dword ptr ds:[1004AE80],0  

10026FDE     EB 26                jmp short 10027006  

10026FE0     83FE 01              cmp esi,1  

10026FE3     74 05                je short 10026FEA  

10026FE5     83FE 02              cmp esi,2  

10026FE8     75 22                jnz short 1002700C  

10026FEA     A1 08C60410          mov eax,dword ptr ds:[1004C608]  

10026FEF     85C0                 test eax,eax  

10026FF1     74 09                je short 10026FFC  

10026FF3     57                   push edi  

10026FF4     56                   push esi  

10026FF5     53                   push ebx  

10026FF6     FFD0                 call eax
```

随便从程序中找个 API 调用：  

```
10026EF8     FF15 E4610310        call dword ptr ds:[100361E4]; kernel32.GetSystemTime
```

数据窗口跟随 `100361E4`，输入表函数已经全部获得正确的系统地址了。  

```
10036000  BB D5 DC 77 8E 77 DA 77 E7 EB DA 77 F0 6B DA 77  
10036010  1B 76 DA 77 83 78 DA 77 B3 77 DA 77 00 00 00 00  
10036020  39 34 17 5D 00 00 00 00 3B 6A EF 77 A6 6C EF 77  
10036030  FC C6 EF 77 D5 5F EF 77 10 5E EF 77 82 9A EF 77  
10036040  C0 6D EF 77 0B 5D EF 77 F1 5F EF 77 A7 5B EF 77  
10036050  21 A8 EF 77 0C D1 EF 77 A0 59 EF 77 00 00 00 00  
10036060  D7 EF 80 7C 0E 18 80 7C 77 9B 80 7C C7 A0 80 7C  
10036070  7A A1 87 7C 8F 0C 81 7C AC 92 80 7C 37 97 80 7C  
10036080  57 B3 80 7C A1 97 83 7C 62 5F 82 7C AD 9C 80 7C  
10036090  4B 6F 82 7C 28 AC 80 7C 51 28 81 7C 31 03 93 7C  
100360A0  66 AA 80 7C 30 25 80 7C CB CA 81 7C 59 35 81 7C  
100360B0  24 1A 80 7C 19 90 83 7C 29 9F 80 7C 25 16 80 7C  
100360C0  42 24 80 7C 6E 9C 80 7C A9 26 82 7C 72 17 81 7C  
100360D0  50 F8 81 7C CB D8 81 7C 57 BB 80 7C 80 A4 80 7C  
100360E0  B9 8C 83 7C C0 9F 80 7C ED 70 83 7C 7E D4 80 7C  
100360F0  E3 12 81 7C 53 C1 81 7C 81 9A 80 7C 14 9B 80 7C  
10036100  29 29 81 7C 10 11 81 7C 6A 48 81 7C 78 2C 81 7C  
10036110  23 CC 81 7C 5F 48 81 7C 3F DC 81 7C EE 1E 80 7C  
10036120  69 10 81 7C A9 2C 81 7C CF C6 80 7C 8A 2B 86 7C  
10036130  86 03 81 7C 58 CD 80 7C A6 0D 81 7C 9F 0F 81 7C  
10036140  2A E8 81 7C 43 99 80 7C 50 97 80 7C 40 03 93 7C  
10036150  53 34 81 7C 0F 2B 81 7C ED 09 93 7C B3 9E 80 7C  
10036160  C4 2F 88 7C 29 B5 80 7C 11 03 81 7C E0 C6 80 7C  
10036170  4E 99 80 7C 4C 17 81 7C 54 2A 82 7C ED 10 92 7C  
10036180  05 10 92 7C A1 9F 80 7C 8A 18 93 7C 8D B7 80 7C  
10036190  6C 94 80 7C FE B9 80 7C FC B7 80 7C 2F 08 81 7C  
100361A0  3F EB 80 7C BD E4 81 7C 28 9C 80 7C 7C 2F 81 7C  
100361B0  1B EC 80 7C A7 24 80 7C 0D E0 80 7C 8D 2C 81 7C  
100361C0  AB 14 81 7C AD 97 80 7C 94 97 80 7C 7B 97 80 7C  
100361D0  40 7A 95 7C E1 EA 81 7C F5 9B 80 7C A9 CC 80 7C  
100361E0  AE 94 83 7C 6B 17 80 7C C1 C9 80 7C 2B 2E 83 7C  
100361F0  3D 04 93 7C D4 05 93 7C FD 79 93 7C A2 CA 81 7C  
10036200  4E A3 80 7C C4 CE 80 7C E6 2B 81 7C 93 D2 80 7C  
10036210  16 1E 80 7C 00 00 00 00 62 DB D1 77 AE E2 D1 77  
10036220  6D 86 D1 77 C5 D3 D1 77 56 B5 D1 77 97 86 D1 77  
10036230  DE D4 D1 77 5A DC D1 77 E1 88 D2 77 A8 67 D2 77  
10036240  C9 6C D2 77 50 8E D1 77 7C 94 D1 77 C5 B4 D1 77  
10036250  EF 01 D3 77 B1 B4 D1 77 A4 52 D2 77 AE 21 D2 77  
10036260  8E C7 D1 77 D3 DE D1 77 75 8F D1 77 7C B5 D1 77  
10036270  1A 8C D1 77 BB D7 D1 77 98 EC D3 77 B8 E7 D1 77  
10036280  3C FC D1 77 2E F8 D3 77 9D B4 D1 77 FA E8 D1 77  
10036290  A8 C6 D1 77 DC E5 D1 77 2F 3A D2 77 A9 F8 D3 77  
100362A0  2C 90 D1 77 64 C0 D1 77 4B E3 D1 77 2F 15 D3 77  
100362B0  32 E0 D1 77 D4 C4 D1 77 06 8C D1 77 DE A2 D1 77  
100362C0  BC C6 D1 77 0B 05 D5 77 00 00 00 00 00 00 00 00  
```

`OEP RVA=00026FC4   IAT RVA=00036000  IAT Size=2C8`  

用 `LordPE` 把 `AdobeLM.dll` 抓取出来，修复输入表得到 `dump\_.dll`  

----------------------------------  

`SafeDisc V2.43.000.osc` 辅助脚本如下  

```
//////////////////////////////////////////////////  

//  FileName    :  SafeDisc V2.43.000.osc  

//  Comment     :  SafeDisc V2.43.000 FixedImportingFunction  

//  Environment :  WinXP SP2,OllyDbg V1.10,OllyScript V0.92  

//  Author      :  fly  

//  WebSite     :  http://www.unpack.cn  

//  Date        :  2005-11-23 22:00  

//////////////////////////////////////////////////  

#log  

dbh  

var EP  

var Temp  

var IsDebuggerPresent  

var GetCurrentProcess  

var ZwQueryInformationProcess  

var CreateEventA  

var MagicJmp  

var FixedOver  

//IsDebuggerPresent————————————————  

mov EP,eip  

log EP  

gpa "IsDebuggerPresent", "KERNEL32.dll"  

mov IsDebuggerPresent,$RESULT  

eob IsDebuggerPresent  

bp IsDebuggerPresent  

esto  

GoOn0:  

esto  

IsDebuggerPresent:  

log eip  

cmp eip,IsDebuggerPresent  

jne GoOn0  

bc IsDebuggerPresent  

//ZwQueryInformationProcess————————————  

/\*  

00879889     FF15 B4208C00       call dword ptr ds:[8C20B4] ; kernel32.GetCurrentProcess  

0087988F     50                  push eax  

00879890     FFD7                call edi                 ; ntdll.ZwQueryInformationProcess  

00879892     8B4424 0C           mov eax,dword ptr ss:[esp+C]  

00879896     85C0                test eax,eax  

00879898     75 02               jnz short 0087989C  

\*/  

gpa "GetCurrentProcess", "KERNEL32.dll"  

mov GetCurrentProcess,$RESULT  

eob GetCurrentProcess  

bp GetCurrentProcess  

esto  

GoOn1:  

esto  

GetCurrentProcess:  

cmp eip,GetCurrentProcess  

jne GoOn1  

bc GetCurrentProcess  

rtu  

find eip, #8B44240C85C0#  

cmp $RESULT, 0  

je NoFind  

mov ZwQueryInformationProcess,$RESULT  

log ZwQueryInformationProcess  

eob ZwQueryInformationProcess  

bp ZwQueryInformationProcess  

esto  

ZwQueryInformationProcess:  

bc ZwQueryInformationProcess  

mov Temp,esp  

add Temp,0C  

mov [Temp],0000  

//CreateEventA——————————————————  

gpa "CreateEventA", "KERNEL32.dll"  

mov CreateEventA,$RESULT  

eob CreateEventA  

bphws CreateEventA, "x"  

esto  

GoOn2:  

esto  

CreateEventA:  

log eip  

cmp eip,CreateEventA  

jne GoOn2  

bphwc CreateEventA  

rtu  

//EP———————————————————————  

add EP,1  

mov Temp, [EP]  

add Temp,4  

add EP,Temp  

add EP,6  

log EP  

mov Temp, [EP]  

and Temp,0FF  

log Temp  

add EP,1  

add EP,Temp  

log EP  

//jmp Second  

//FixedImportingFunction—————————————  

/\*  

008BF088     8B45 F4            mov eax,dword ptr ss:[ebp-C]  

008BF08B     40                 inc eax  

008BF08C     8945 F4            mov dword ptr ss:[ebp-C],eax  

008BF08F     8B45 F4            mov eax,dword ptr ss:[ebp-C]  

008BF092     3B45 14            cmp eax,dword ptr ss:[ebp+14]  

008BF095     73 55              jnb short 008BF0EC  

008BF097     8B45 F4            mov eax,dword ptr ss:[ebp-C]  

008BF09A     C1E8 03            shr eax,3  

008BF09D     8B4D F8            mov ecx,dword ptr ss:[ebp-8]  

008BF0A0     8B15 DCEC8D00      mov edx,dword ptr ds:[8DECDC]  

008BF0A6     8B0C8A             mov ecx,dword ptr ds:[edx+ecx*4]  

008BF0A9     0FB60401           movzx eax,byte ptr ds:[ecx+eax]  

008BF0AD     8B4D F4            mov ecx,dword ptr ss:[ebp-C]  

008BF0B0     83E1 07            and ecx,7  

008BF0B3     6A 01              push 1  

008BF0B5     5A                 pop edx  

008BF0B6     D3E2              shl edx,cl  

008BF0B8     23C2              and eax,edx  

008BF0BA     85C0              test eax,eax  

008BF0BC     75 2C             jnz short 008BF0EA  

008BF0BE     8B45 F8            mov eax,dword ptr ss:[ebp-8]  

008BF0C1     69C0 8D000000      imul eax,eax,8D  

008BF0C7     8B0D E0EC8D00      mov ecx,dword ptr ds:[8DECE0]  

008BF0CD     8B4401 4C          mov eax,dword ptr ds:[ecx+eax+4C]  

008BF0D1     8B4D F4            mov ecx,dword ptr ss:[ebp-C]  

008BF0D4     FF3488            push dword ptr ds:[eax+ecx*4]  

008BF0D7     FF75 F8           push dword ptr ss:[ebp-8]  

008BF0DA     E8 DB000000       call 008BF1BA  

008BF0DF     59                 pop ecx  

008BF0E0     59                 pop ecx  

008BF0E1     8B4D F4            mov ecx,dword ptr ss:[ebp-C]
```



### 汇编代码片段

```
008BF0E4      8B55 18             mov edx,dword ptr ss:[ebp+18]  

008BF0E7      89048A             mov dword ptr ds:[edx+ecx*4],eax  

008BF0EA      EB 9C              jmp short 008BF088  

008BF0EC      EB 07              jmp short 008BF0F5  

*/  

eob FixedImportingFunction  

find eip, #D3E223C285C0752C8B45F8#  

cmp $RESULT, 0  

je NoFind  

add $RESULT,4  

mov MagicJmp,$RESULT  

bphws MagicJmp, "x"  

find MagicJmp, #EB9CEB07#  

cmp $RESULT, 0  

je NoFind  

add $RESULT,2  

mov FixedOver,$RESULT  

bphws FixedOver, "x"  

bphws EP, "x"  

esto  

GoOn3:  

esto  

FixedImportingFunction:  

cmp eip,MagicJmp  

je MagicJmp  

cmp eip,FixedOver  

je MagicJmp  

cmp eip,EP  

je EP  

MagicJmp:  

bphwc MagicJmp  

asm MagicJmp, "xor eax,eax"  

esto  

FixedOver:  

asm MagicJmp, "test eax,eax"  

bphws MagicJmp, "x"  

jmp GoOn3  

Second:  

bphws EP, "x"  

eob EP  

esto  

EP:  

log EP  

bphwc MagicJmp  

bphwc FixedOver  

bphwc EP  

sti   

//GameOver———————————————————— 

log eip  

cmt eip, "This is the OEP! Found By: fly"  

MSG "Just : OEP !  Dump and Fix IAT/Reloction.  Good Luck  "  

ret  

NoFind:  

MSG "Error! Maybe It's not SafeDisc V2.43.000 !  "  

ret  

————————————————————————————————— 
```

### 四、修复函数调用地址

虽然已经获得了正确的函数系统地址，但是 `SafeDisc` 的输入表呼叫地址乱处理了，麻烦就在这里了。

把 `SafeDisc V2.43.000.osc` 中“`//jmp Second`”的“`//`”去掉，新开个 `OllyDBG`，重新加载 `AdobeLM.dll`，运行脚本。现在脚本没有处理输入表函数，直接停在 `OEP` 处。

把我们第三步获取的 `10036000`-`100362C8` 函数地址复制到 `10046000` 处，以备下面比较、修复。

`Alt+M` 察看 `AdobeLM.dll` 内存

```
10000000      00001000 (4096.)       AdobeLM  10000000                PE header  

10001000      00035000 (217088.)     AdobeLM  10000000      .text      code  

10036000      00008000 (32768.)      AdobeLM  10000000      .rdata  

1003E000      0000F000 (61440.)      AdobeLM  10000000      .data  

1004D000      00005000 (20480.)      AdobeLM  10000000      .rsrc  

10052000      00005000 (20480.)      AdobeLM  10000000      .reloc  

10057000      00003000 (12288.)      AdobeLM  10000000      stxt774  

1005A000      00004000 (16384.)      AdobeLM  10000000      stxt371    SFX,imports
```

我们把修复代码放在第 3 个区段吧，设置这几个区段为完整权限。

`Ctrl+G`：`1003E000`，在 `1003E000` 处 `Ctrl+\*` 新建 `EIP`，写入 `Patch` 代码：

```
1003E000      60                   pushad  

1003E001      BE 00100010        mov esi,10001000  

//代码段开始地址  

1003E006      BF 005F0310        mov edi,10035F00  

//代码段结束地址  

1003E00B      3BF7               cmp esi,edi  

1003E00D      7C 05              jl short 1003E014  

1003E00F      E9 91000000        jmp 1003E0A5  

//修复结束跳转  

1003E014      8B06               mov eax,dword ptr ds:[esi]  

1003E016      3D 00600310        cmp eax,10036000  

//输入表开始地址  

1003E01B      7D 03              jge short 1003E020  

1003E01D      46                 inc esi  

1003E01E      EB EB              jmp short 1003E00B  

1003E020      3D C8620310        cmp eax,100362C8  

//输入表结束地址  

1003E025      7F F6              jg short 1003E01D  

1003E027      8B18               mov ebx,dword ptr ds:[eax]  

1003E029      85DB               test ebx,ebx  

1003E02B      74 F0              je short 1003E01D  

1003E02D      81FB 00000010      cmp ebx,10000000  

//判断是否是壳不加密的 API  

1003E033      7F E8              jg short 1003E01D  

1003E035      8D4E FE            lea ecx,dword ptr ds:[esi-2]  

//取函数调用的地址  

1003E038      66:8B19            mov bx,word ptr ds:[ecx]  

1003E03B      66:81FB FF15       cmp bx,15FF  

//比较是否是 call  

1003E040      75 DB              jnz short 1003E01D  

//循环扫描符合 call dword ptr ds:[10036XXX]条件的  

1003E042      8B1D F0E00310      mov ebx,dword ptr ds:[1003E0F0]  

//[1003E0F0]处预先写入 1003E100  ★  

1003E048      8933               mov dword ptr ds:[ebx],esi  

//保存搜索进度  

1003E04A      83C3 04            add ebx,4  

1003E04D      891D F0E00310      mov dword ptr ds:[1003E0F0],ebx  

//保存  

1003E053      8935 F4E00310      mov dword ptr ds:[1003E0F4],esi  

1003E059      FFE1               jmp ecx  

//跳到函数调用的地址处执行  

1003E05A      90                 nop  

1003E05B      90                 nop  

1003E05C      90                 nop  

1003E05D      90                 nop  

1003E05E      90                 nop  

1003E05F      90                 nop  

1003E060      90                 nop  

1003E061      90                 nop  

1003E062      90                 nop  

1003E063      90                 nop  

1003E064      8B1D F0E00310      mov ebx,dword ptr ds:[1003E0F0]  

//SafeDisc 解密后强制跳到这里  

1003E06A      8B0424             mov eax,dword ptr ss:[esp]  

//[ESP]是解密后的函数系统地址  

1003E06D      8903               mov dword ptr ds:[ebx],eax  

//保存函数系统地址  

1003E06F      B9 C8020000        mov ecx,2C8  

1003E074      BF 00600410        mov edi,10046000  

//把我们第三步获取的 10036000-100362C8 函数地址复制到 10046000 处   ★  

1003E079      F2:AF              repne scas dword ptr es:[edi]  

//搜寻相同的函数地址  

1003E07B      75 28              jnz short 1003E0A5  

//没找到？哦，应该都可以找到的  

1003E07D      90                 nop  

1003E07E      90                 nop  

1003E07F      90                 nop  

1003E080      90                 nop  

1003E081      81EF 04000100      sub edi,10004  

//10046000－10036000＝100000    再减 4 就是找到的存放函数地址的地址了   ★  

1003E087      8B35 F4E00310      mov esi,dword ptr ds:[1003E0F4]  

//函数调用地址  

1003E08D      893E               mov dword ptr ds:[esi],edi  

//修复吧  

1003E08F      83C6 04            add esi,4  

1003E092      E9 6FFFFFFF        jmp 1003E006  

//继续循环  

1003E097      8B35 F4E00310      mov esi,dword ptr ds:[1003E0F4]  

1003E09D      83C6 04            add esi,4  

1003E0A0      E9 61FFFFFF        jmp 1003E006  

//继续循环  

1003E0A5      61                 popad  

//Game Over  修复完毕  

1003E0A6      EB FE              jmp short 1003E0A6
```

二进制代码复制如下：

```
60 BE 00 10 00 10 BF 00 5F 03 10 3B F7 7C 05 E9 91 00 00 00 8B 06 3D 00 60 03 10 7D 03 46 EB EB  

3D C8 62 03 10 7F F6 8B 18 85 DB 74 F0 81 FB 00 00 00 10 7F E8 8D 4E FE 66 8B 19 66 81 FB FF 15  

75 DB 8B 1D F0 E0 03 10 89 33 83 C3 04 89 1D F0 E0 03 10 89 35 F4 E0 03 10 FF E1 90 90 90 90 90  

90 90 90 90 8B 1D F0 E0 03 10 8B 04 24 89 03 B9 C8 02 00 00 BF 00 60 04 10 F2 AF 75 28 90 90 90  

90 81 EF 04 00 01 00 8B 35 F4 E0 03 10 89 3E 83 C6 04 E9 6F FF FF FF 8B 35 F4 E0 03 10 83 C6 04  

E9 61 FF FF FF 61 EB FE  
```

### `SafeDisc` 解密 `CALL` 里面的修改

```
10001403      FF15 C4620310      call dword ptr ds:[100362C4]  

00AED180      68 1713EABF        push BFEA1317  

00AED185      9C                 pushfd  

00AED186      60                 pushad  

00AED187      54                 push esp  

00AED188      68 C0D1AE00        push 0AED1C0  

00AED18D      E8 0322DDFF        call 008BF395  

008BF395      55                 push ebp  

008BF396      8BEC               mov ebp,esp  

008BF398      83EC 40            sub esp,40  

008BF39B      53                 push ebx  

008BF39C      56                 push esi  

008BF39D      57                 push edi  

008BF39E      F0:FF05 742F8D00  lock inc dword ptr ds:[8D2F74]  

008BF3A5      74 0E              je short 008BF3B5  

008BF3A7      6A FF              push -1  

008BF3A9      FF35 48ED8D00      push dword ptr ds:[8DED48]  

008BF3AF      FF15 84208C00      call dword ptr ds:[8C2084]  

008BF3B5      EB 0A              jmp short 008BF3C1  

008BF3D5      8B45 08            mov eax,dword ptr ss:[ebp+8]  

008BF3D8      8B00               mov eax,dword ptr ds:[eax]  

008BF3DA      8945 E0            mov dword ptr ss:[ebp-20],eax  

008BF3DD      8B45 08            mov eax,dword ptr ss:[ebp+8]  

008BF3E0      8B40 04            mov eax,dword ptr ds:[eax+4]  

008BF3E3      8945 E4            mov dword ptr ss:[ebp-1C],eax  

008BF3E6      837D E0 FF         cmp dword ptr ss:[ebp-20],-1  

008BF3EA      0F85 A5000000      jnz 008BF495  

008BF3F0      8365 E0 00         and dword ptr ss:[ebp-20],0  

008BF3F4      EB 07              jmp short 008BF3FD  

008BF3F6      8B45 E0            mov eax,dword ptr ss:[ebp-20]  

008BF3F9      40                 inc eax  
```



```
008BF3FA      8945 E0             mov dword ptr ss:[ebp-20],eax  

008BF3FD      A1 E0EC8D00         mov eax,dword ptr ds:[8DECE0]  

008BF402      8B4D E0             mov ecx,dword ptr ss:[ebp-20]  

008BF405      3B48 0F             cmp ecx,dword ptr ds:[eax+F]  

008BF408      0F83 87000000      jnb 008BF495  

008BF40E      FF75 E0             push dword ptr ss:[ebp-20]  

008BF411      E8 F7040000        call 008BF90D  

008BF416      59                 pop ecx  

008BF417      0FB7C0             movzx eax,ax  

008BF41A      85C0               test eax,eax  

008BF41C      74 72              je short 008BF490  

008BF41E      8365 E4 00          and dword ptr ss:[ebp-1C],0  

008BF422      EB 07              jmp short 008BF42B  

008BF424      8B45 E4             mov eax,dword ptr ss:[ebp-1C]  

008BF427      40                 inc eax  

008BF428      8945 E4             mov dword ptr ss:[ebp-1C],eax  

008BF42B      8B45 E0             mov eax,dword ptr ss:[ebp-20]  

008BF42E      69C0 8D000000       imul eax,eax,8D  

008BF434      8B0D E0EC8D00      mov ecx,dword ptr ds:[8DECE0]  

008BF43A      8B55 E4             mov edx,dword ptr ss:[ebp-1C]  

008BF43D      3B5401 58          cmp edx,dword ptr ds:[ecx+eax+58]  

008BF441      73 43              jnb short 008BF486  

008BF443      8B45 E0             mov eax,dword ptr ss:[ebp-20]  

008BF446      69C0 8D000000       imul eax,eax,8D  

008BF44C      8B4D E4             mov ecx,dword ptr ss:[ebp-1C]  

008BF44F      69C9 C3040000       imul ecx,ecx,4C3  

008BF455      8B15 E0EC8D00      mov edx,dword ptr ds:[8DECE0]  

008BF45B      8B8402 C3000000    mov eax,dword ptr ds:[edx+eax+C3]  

008BF462      8B55 08             mov edx,dword ptr ss:[ebp+8]  

008BF465      8B52 08             mov edx,dword ptr ds:[edx+8]  

008BF468      3B9408 AA040000    cmp edx,dword ptr ds:[eax+ecx+4AA]  

008BF46F      75 13              jnz short 008BF484  

008BF471      8B45 08             mov eax,dword ptr ss:[ebp+8]  

008BF474      8B4D E4             mov ecx,dword ptr ss:[ebp-1C]  

008BF477      8948 04             mov dword ptr ds:[eax+4],ecx  

008BF47A      8B45 08             mov eax,dword ptr ss:[ebp+8]  

008BF47D      8B4D E0             mov ecx,dword ptr ss:[ebp-20]  

008BF480      8908                mov dword ptr ds:[eax],ecx  

008BF482      EB 02              jmp short 008BF486  

008BF484      EB 9E              jmp short 008BF424  

008BF486      8B45 08             mov eax,dword ptr ss:[ebp+8]  

008BF489      8338 FF             cmp dword ptr ds:[eax],-1  

008BF48C      74 02              je short 008BF490  

008BF48E      EB 05              jmp short 008BF495  

008BF490      E9 61FFFFFF        jmp 008BF3F6  

008BF495      8B45 E0             mov eax,dword ptr ss:[ebp-20]  

008BF498      69C0 8D000000       imul eax,eax,8D  

008BF49E      8B0D E0EC8D00      mov ecx,dword ptr ds:[8DECE0]  

008BF4A4      8B8401 C3000000    mov eax,dword ptr ds:[ecx+eax+C3]  

008BF4AB      8945 DC             mov dword ptr ss:[ebp-24],eax  

008BF4AE      8B45 C8             mov eax,dword ptr ss:[ebp-38]  

008BF4B1      8945 FC             mov dword ptr ss:[ebp-4],eax  

008BF4B4      FF75 C8             push dword ptr ss:[ebp-38]  

008BF4B7      FF75 E4             push dword ptr ss:[ebp-1C]  

008BF4BA      FF75 DC             push dword ptr ss:[ebp-24]  

008BF4BD      E8 E1F1FFFF        call 008BE6A3  

008BF4C2      83C4 0C             add esp,0C  

008BF4C5      8945 F4             mov dword ptr ss:[ebp-C],eax  

008BF4C8      837D F4 00          cmp dword ptr ss:[ebp-C],0  

008BF4CC      74 39              je short 008BF507  

//Patch①、jmp 008BF507   ★    强制每次都解密  

008BF507      8B45 E4             mov eax,dword ptr ss:[ebp-1C]  

008BF50A      8945 D8             mov dword ptr ss:[ebp-28],eax  

008BF50D      8D45 CC             lea eax,dword ptr ss:[ebp-34]  

008BF510      50                 push eax  

008BF511      8D45 D0             lea eax,dword ptr ss:[ebp-30]  

008BF514      50                 push eax  

008BF515      8D45 F8             lea eax,dword ptr ss:[ebp-8]  

008BF518      50                 push eax  

008BF519      FF75 C8             push dword ptr ss:[ebp-38]  

008BF51C      E8 4F040000        call 008BF970  

008BF521      83C4 10             add esp,10  

008BF524      0FB7C0             movzx eax,ax  

008BF527      83F8 01             cmp eax,1  

008BF52A      0F85 75010000      jnz 008BF6A5  

008BF530      8B45 C8             mov eax,dword ptr ss:[ebp-38]  

008BF533      2B45 CC             sub eax,dword ptr ss:[ebp-34]  

008BF536      50                 push eax  

008BF537      E8 7C0EFCFF        call 008803B8  

008BF53C      50                 push eax  

008BF53D      E8 7B0EFCFF        call 008803BD  

008BF542      59                 pop ecx  

008BF543      59                 pop ecx  

008BF544      0FB7C0             movzx eax,ax  

008BF547      85C0               test eax,eax  

008BF549      0F84 A5000000      je 008BF5F4  

008BF54F      8B45 C8             mov eax,dword ptr ss:[ebp-38]  

//这里是 SafeDisc 的暗桩，某些符合上面扫描条件的地址含有 SafeDisc 暗桩  

//Patch②代码：   ★  

008BF54F      F0:FF0D 742F8D00   lock dec dword ptr ds:[8D2F74]  

008BF556      78 0C              js short 008BF564  

008BF558      FF35 48ED8D00      push dword ptr ds:[8DED48]  

008BF55E      FF15 4C208C00      call dword ptr ds:[8C204C]  

008BF564      8B65 0C             mov esp,dword ptr ss:[ebp+C]  

008BF567      61                 popad  

008BF568      9D                 popfd  

008BF569      58                 pop eax  

008BF56A      83C4 04             add esp,4  

008BF56D      E9 25EB770F        jmp 1003E097  

//平衡堆栈后跳回去继续循环  

OllyDBG 中二进制代码复制如下：  

F0 FF 0D 74 2F 8D 00 78 0C FF 35 48 ED 8D 00 FF 15 4C 20 8C 00 8B 65 0C 61 9D 58 83 C4 04 E9 25  

EB 77 0F  

008BF5F4      8B45 F8             mov eax,dword ptr ss:[ebp-8]  

008BF5F7      0345 CC             add eax,dword ptr ss:[ebp-34]  

008BF5FA      8B4D C8             mov ecx,dword ptr ss:[ebp-38]  

008BF5FD      2BC8                sub ecx,eax  

008BF5FF      894D F0             mov dword ptr ss:[ebp-10],ecx  

008BF602      FF75 F0             push dword ptr ss:[ebp-10]  

008BF605      E8 14070000        call 008BFD1E  

008BF60A      59                 pop ecx  

008BF60B      0FB7C0             movzx eax,ax  

008BF60E      83F8 01             cmp eax,1  

008BF611      0F85 8E000000      jnz 008BF6A5  

008BF617      8B45 E4             mov eax,dword ptr ss:[ebp-1C]  

008BF61A      69C0 C3040000       imul eax,eax,4C3  

008BF620      8B4D FC             mov ecx,dword ptr ss:[ebp-4]  

008BF623      8B55 DC             mov edx,dword ptr ss:[ebp-24]  

008BF626      8B49 02             mov ecx,dword ptr ds:[ecx+2]  

008BF629      3B8C02 AA040000    cmp ecx,dword ptr ds:[edx+eax+4AA]  

008BF630      75 73              jnz short 008BF6A5  

008BF632      8B45 FC             mov eax,dword ptr ss:[ebp-4]  

008BF635      0FB600              movzx eax,byte ptr ds:[eax]  

008BF638      3D FF000000        cmp eax,0FF  

008BF63D      75 66              jnz short 008BF6A5  

008BF63F      8B45 FC             mov eax,dword ptr ss:[ebp-4]  

008BF642      0FB640 01           movzx eax,byte ptr ds:[eax+1]  

008BF646      83F8 15             cmp eax,15  

008BF649      75 5A              jnz short 008BF6A5  

008BF64B      8B45 E4             mov eax,dword ptr ss:[ebp-1C]  

008BF64E      8945 D8             mov dword ptr ss:[ebp-28],eax  

008BF651      A1 E0EC8D00         mov eax,dword ptr ds:[8DECE0]  

008BF656      8B40 26             mov eax,dword ptr ds:[eax+26]  

008BF659      0345 F0             add eax,dword ptr ss:[ebp-10]  

008BF65C      50                 push eax  

008BF65D      FF75 D8             push dword ptr ss:[ebp-28]  

008BF660      8B45 E0             mov eax,dword ptr ss:[ebp-20]  

008BF663      69C0 8D000000       imul eax,eax,8D  

008BF669      8B0D E0EC8D00      mov ecx,dword ptr ds:[8DECE0]  
```



## 反汇编代码段

```
008BF66F     FF7401 58          push dword ptr ds:[ecx+eax+58]  

008BF673     E8 AA020000        call 008BF922  

008BF678     83C4 0C            add esp,0C  

008BF67B     8945 D8            mov dword ptr ss:[ebp-28],eax  

008BF67E     8B45 D8            mov eax,dword ptr ss:[ebp-28]  

008BF681     C1E8 03            shr eax,3  

008BF684     8B4D E0            mov ecx,dword ptr ss:[ebp-20]  

008BF687     8B15 DCEC8D00      mov edx,dword ptr ds:[8DECDC]  

008BF68D     8B0C8A             mov ecx,dword ptr ds:[edx+ecx*4]  

008BF690     0FB60401           movzx eax,byte ptr ds:[ecx+eax]  

008BF694     8B4D D8            mov ecx,dword ptr ss:[ebp-28]  

008BF697     83E1 07            and ecx,7  

008BF69A     6A 01              push 1  

008BF69C     5A                 pop edx  

008BF69D     D3E2               shl edx,cl  

008BF69F     23C2               and eax,edx  

008BF6A1     85C0               test eax,eax  

008BF6A3     74 AC              je short 008BF651  

008BF6A5     8B45 E0            mov eax,dword ptr ss:[ebp-20]  

008BF6A8     69C0 8D000000      imul eax,eax,8D  

008BF6AE     8B0D E0EC8D00      mov ecx,dword ptr ds:[8DECE0]  

008BF6B4     8B4401 4C          mov eax,dword ptr ds:[ecx+eax+4C]  

008BF6B8     8B4D D8            mov ecx,dword ptr ss:[ebp-28]  

008BF6BB     8B0488             mov eax,dword ptr ds:[eax+ecx*4]  

008BF6BE     8945 D8            mov dword ptr ss:[ebp-28],eax  

008BF6C1     8B45 D8            mov eax,dword ptr ss:[ebp-28]  

008BF6C4     69C0 C3040000      imul eax,eax,4C3  

008BF6CA     8B4D DC            mov ecx,dword ptr ss:[ebp-24]  

008BF6CD     8B8401 72040000    mov eax,dword ptr ds:[ecx+eax+472]  

008BF6D4     8945 F4            mov dword ptr ss:[ebp-C],eax  

008BF6D7     837D F4 00         cmp dword ptr ss:[ebp-C],0  

008BF6DB     75 26              jnz short 008BF703  

008BF6DD     FF75 D8            push dword ptr ss:[ebp-28]  

008BF6E0     FF75 E0            push dword ptr ss:[ebp-20]  

008BF6E3     E8 D2FAFFFF        call 008BF1BA  

008BF6E8     59                 pop ecx  

008BF6E9     59                 pop ecx  

008BF6EA     8945 F4            mov dword ptr ss:[ebp-C],eax  

008BF6ED     8B45 D8            mov eax,dword ptr ss:[ebp-28]  

008BF6F0     69C0 C3040000      imul eax,eax,4C3  

008BF6F6     8B4D DC            mov ecx,dword ptr ss:[ebp-24]  

008BF6F9     8B55 F4            mov edx,dword ptr ss:[ebp-C]  

008BF6FC     899401 72040000    mov dword ptr ds:[ecx+eax+472],edx  

008BF703     FF75 F4            push dword ptr ss:[ebp-C]  

008BF706     FF75 C8            push dword ptr ss:[ebp-38]  

008BF709     FF75 E4            push dword ptr ss:[ebp-1C]  

008BF70C     FF75 DC            push dword ptr ss:[ebp-24]  

008BF70F     E8 F0EFFFFF        call 008BE704  

008BF714     83C4 10            add esp,10  

008BF717     8B45 0C            mov eax,dword ptr ss:[ebp+C]  

008BF71A     83C0 24            add eax,24  

008BF71D     8945 C0            mov dword ptr ss:[ebp-40],eax  

008BF720     8B45 C0            mov eax,dword ptr ss:[ebp-40]  

008BF723     8B4D F4            mov ecx,dword ptr ss:[ebp-C]  

008BF726     8908               mov dword ptr ds:[eax],ecx  

008BF728     8B45 C0            mov eax,dword ptr ss:[ebp-40]  

008BF72B     83C0 04            add eax,4  

008BF72E     50                 push eax  

008BF72F     E8 1F61FDFF        call 00895853  

008BF734     59                 pop ecx  

008BF735     F0:FF0D 742F8D00   lock dec dword ptr ds:[8D2F74]  

008BF73C     78 0C              js short 008BF74A  

008BF73E     FF35 48ED8D00      push dword ptr ds:[8DED48]  

008BF744     FF15 4C208C00      call dword ptr ds:[8C204C]  

008BF74A     8B65 0C            mov esp,dword ptr ss:[ebp+C]  

008BF74D     61                 popad  

008BF74E     9D                 popfd  

008BF74F     C3                 retn  
```

//Patch③：`jmp 1003E064` 解密完毕后跳回去控制处理，`[ESP]`是解密后的函数系统地址

—————————————————————————————————

## 五、类 SDK 输入表函数调用地址

上面修复完毕后不要关闭 `OllyDBG`，还有一些类似 SDK 的函数调用需要修复。

此 SDK 同样使用上面的解码 `CALL`，但是 `Patch②`不需要修改，`Patch①`依旧。

修改上面 `Patch③`的 `008BF74F` 处为 `jmp 1003E0DD`，控制流程。

```
1002DE3F     33C0               xor eax,eax  

1002DE41     6A 00              push 0  

1002DE43     394424 08          cmp dword ptr ss:[esp+8],eax  

1002DE47     68 00100000        push 1000  

1002DE4C     0F94C0             sete al  

1002DE4F     50                 push eax  

1002DE50     E9 D3990200        jmp AdobeLM.10057828  
```

//类似 SDK 的函数调用需要修复

```
10057828     53                 push ebx  

10057829     E8 98FCFFFF        call AdobeLM.100574C6  

100574C6     870424             xchg dword ptr ss:[esp],eax  

100574C9     9C                 pushfd  

100574CA     05 DF100000        add eax,10DF  

100574CF     8B18               mov ebx,dword ptr ds:[eax]  

100574D1     6BDB 2E            imul ebx,ebx,2E  

100574D4     0358 04            add ebx,dword ptr ds:[eax+4]  

100574D7     9D                 popfd  

100574D8     58                 pop eax  

100574D9     871C24             xchg dword ptr ss:[esp],ebx  

100574DC     C3                 retn  
```

//入壳处理

```
00AFCBC9     68 56DE0210        push 1002DE56  

00AFCBCE     68 0A13EABF        push BFEA130A  

00AFCBD3     9C                 pushfd  

00AFCBD4     60                 pushad  

00AFCBD5     54                 push esp  

00AFCBD6     68 09CCAF00        push 0AFCC09  

00AFCBDB     E8 B527DCFF        call ~df394b.008BF395  

00AFCBE0     83C4 08            add esp,8  

00AFCBE3     6A 00              push 0  

00AFCBE5     58                 pop eax  

00AFCBE6     61                 popad  

00AFCBE7     9D                 popfd  

00AFCBE8     C3                 retn  
```

`00AE0000` 区段里面包含了需要处理的函数调用地址，可以依此为突破点。

### 写 Patch 代码

在 `1003E0A8` 处新建 `EIP`。

```
1003E0A8     60                 pushad  

1003E0A9     BE 0000AE00        mov esi,00AE0000  

1003E0AE     BF 00600410        mov edi,10046000  

1003E0B3     46                 inc esi  

1003E0B4     81FE 0050B100      cmp esi,00B15000  

//00B15000 是 00AE0000 区段的结束地址  

1003E0BA     7D 53              jge short 1003E10F  

//扫描完毕后跳转  

1003E0BC     803E 68            cmp byte ptr ds:[esi],68  

1003E0BF     75 F2              jnz short 1003E0B3  

1003E0C1     66:817E 04 1068    cmp word ptr ds:[esi+4],6810  

1003E0C7     75 EA              jnz short 1003E0B3  

1003E0C9     817E 0A 9C605468   cmp dword ptr ds:[esi+A],6854609C  

1003E0D0     75 E1              jnz short 1003E0B3  

1003E0D2     817E 01 00000010   cmp dword ptr ds:[esi+1],10000000  

1003E0D9     74 30              je short 1003E10B  

//搜索符合条件的地址  

1003E0DB     FFE6               jmp esi  

//跳过去执行  

1003E0DD     3E:8B0424          mov eax,dword ptr ds:[esp]  

//SafeDisc 解密后强制跳到这里  ★  

//[ESP]是解密后的函数系统地址  

1003E0E1     8B5E 01            mov ebx,dword ptr ds:[esi+1]  

1003E0E4     66:C743 FA FF15    mov word ptr ds:[ebx-6],15FF  

1003E0EA     33C9               xor ecx,ecx  

1003E0EC     3B0439             cmp eax,dword ptr ds:[ecx+edi]  

//自 10046000 搜寻相同的函数地址  

1003E0EF     74 0F              je short 1003E100  

1003E0F1     83C1 04            add ecx,4  

1003E0F4     81F9 D0020000      cmp ecx,2D0  

1003E0FA     72 F0              jb short 1003E0EC  

1003E0FC     EB FE              jmp short 1003E0FC  

//留一个出错的处理点，不过没用到  

1003E0FE     90                 nop  

1003E0FF     90                 nop  

1003E100     8D0C39             lea ecx,dword ptr ds:[ecx+edi]  

1003E103     81E9 00000100      sub ecx,10000  

//10046000－10036000＝100000    找到的存放函数地址的地址  

1003E109     894B FC            mov dword ptr ds:[ebx-4],ecx  

//修复函数调用地址  

1003E10C     EB A5              jmp short 1003E0B3  

//循环  

1003E10E     90                 nop  

1003E10F     61                 popad  

//修复完成  

1003E110     EB FE              jmp short 1003E110  
```



### 二进制代码

```
60 BE 00 00 AE 00 BF 00 60 04 10 46 81 FE 00 50 B1 00 7D 53 80 3E 68 75 F2 66 81 7E 04 10 68 75
EA 81 7E 0A 9C 60 54 68 75 E1 81 7E 01 00 00 00 10 74 30 FF E6 3E 8B 04 24 8B 5E 01 66 C7 43 FA
FF 15 33 C9 3B 04 39 74 0F 83 C1 04 81 F9 D0 02 00 00 72 F0 EB FE 90 90 8D 0C 39 81 E9 00 00 01
00 89 4B FC EB A5 90 61 EB FE
```

注意：由于目标程序是 `DLL` 所以需要考虑重定位表，而这部分地址并没有包含在加壳后 `DLL` 的重定位表中，因此可以在上面的修复代码中加点代码保存每次修复时的地址，便于最后修复重定位表。

---

## 六、类 `CC` 的 `SDK`

### 1. `DLL` 虽然无法使用 `CC`，但是却有 `SDK` 来控制流程

```
10002463     E8 611D0000        call 100041C9
```

// 有很多 `call 100041C9`

```
100041C9     51                 push ecx
100041CA     50                 push eax
100041CB     E8 13F3FFFF        call 100034E3
100034E3     B8 7BEFFFFF        mov eax,-1085
100034E8     59                 pop ecx
100034E9     8D0408             lea eax,dword ptr ds:[eax+ecx]
100034EC     8B00               mov eax,dword ptr ds:[eax]
100034EE     FFE0               jmp eax ; ~df394b.0088127D
```

// 进入 `~df394b.tmp`

### 2. 需要说明的是，文件中有不少假的 `SDK`，`SafeDisc` 真狡猾。

可以手动察看，搜索所有的 `call 100041C9` 命令，把假的 `SDK` 去掉再扫描修复。

如何识别，看你的火眼金睛了，呵呵，举例来说：

```
1000109B     E8 29310000        call AdobeLM.100041C9
```

滚动一下鼠标，会发现这里有了变化：

```
1000109A     6A E8              push -18
1000109C     2931               sub dword ptr ds:[ecx],esi
1000109E     0000               add byte ptr ds:[eax],al
100010A0     8B4C24 04          mov ecx,dword ptr ss:[esp+4]
100010A4     85C9               test ecx,ecx
100010A6     74 06              je short AdobeLM.100010AE
```

还有一种有点难判断：

```
1002AF0A     C3                 retn
1002AF0B     E8 B992FDFF        call AdobeLM.100041C9
1002AF10     55                 push ebp
1002AF11     8BEC               mov ebp,esp
```

看到这个 `CALL` 在 `retn` 附近，`Ctrl+A` 后没有其他地方调用这里，可以判定是烟雾弹了。

把以下地址暂时修改为 `call 100041CC`

```
1000109B   call      100041C9
10001FAB   call      100041C9
10002F0B   call      100041C9
1000380B   call      100041C9
10003B8B   call      100041C9
1000476B   call      100041C9
10004FBB   call      100041C9
1000572B   call      100041C9
100066AB   call      100041C9
1000EB5B   call      100041C9
1000FE3B   call      100041C9
1000FEEB   call      100041C9
10011B9B   call      100041C9
10011DDB   call      100041C9
1001379B   call      100041C9
10013A5B   call      100041C9
10013BDB   call      100041C9
10013C3B   call      100041C9
10013D0B   call      100041C9
1001409B   call      100041C9
1001439B   call      100041C9
1001441B   call      100041C9
1002908B   call      100041C9
1002913B   call      100041C9
1002AF0B   call      100041C9
1002D53B   call      100041C9
```

等扫描完毕后再全部恢复回来，免得误修复。

### 3. 写 `Patch` 代码，在 `1003E112` 处新建 `EIP`

```
1003E112      60                 pushad
1003E113      BE 00100010        mov esi,10001000
1003E118      46                 inc esi
1003E119      81FE 905A0310      cmp esi,10035A90
1003E11F      77 18              ja short 1003E139
```
// 扫描完毕后跳转

```
1003E121      803E E8            cmp byte ptr ds:[esi],0E8
1003E124      75 F2              jnz short 1003E118
1003E126      8B46 01            mov eax,dword ptr ds:[esi+1]
1003E129      03C6               add eax,esi
1003E12B      83C0 05            add eax,5
1003E12E      3D C9410010        cmp eax,100041C9
1003E133      75 E3              jnz short 1003E118
```
// 循环扫描所有 `call 100041C9` 的地方

```
1003E135      FFD6               call esi
```
// 调用

```
1003E137      EB DF              jmp short 1003E118
```
// 循环

```
1003E139      61                 popad
```
// 解码完毕后中断在这里

```
1003E13A      EB FE              jmp short 1003E13A
```

二进制代码复制如下：

```
60 BE 00 10 00 10 46 81 FE 90 5A 03 10 77 18 80 3E E8 75 F2 8B 46 01 03 C6 83 C0 05 3D C9 41 00
10 75 E3 FF D6 EB DF 61 EB FE
```

### 4. `SafeDisc` 解密 `CALL` 里面的修改，和上面的解码地方不同了

```
0088127D      58                 pop eax
0088127E      59                 pop ecx
0088127F      68 00004000        push 400000
00881284      9C                 pushfd
00881285      60                 pushad
00881286      54                 push esp
00881287      E8 D2FFFFFF        call 0088125E
0088128C      5C                 pop esp
0088128D      61                 popad
0088128E      9D                 popfd
0088128F      C3                 retn
```

// `Patch④` 修改为：

```
0088128F      BC C0E10600        mov esp,6E1C0
```
// 控制流程，返回 `Patch` 代码的地方

```
00881294      C3                 retn
```

注意，这里的 `mov esp,XXXXXXXX` 具体是何要看此时的堆栈。

如这次中断在 `0088128F` 处堆栈为：

```
0006E1BC    1000F0FE  AdobeLM.1000F0FE
0006E1C0    1003E137  返回到 AdobeLM.1003E137
```

```
00881192      55                 push ebp
00881193      8BEC               mov ebp,esp
00881195      81EC D0020000      sub esp,2D0
0088119B      53                 push ebx
0088119C      8BD9               mov ebx,ecx
0088119E      56                 push esi
0088119F      57                 push edi
008811A0      8D43 20            lea eax,dword ptr ds:[ebx+20]
008811A3      50                 push eax
008811A4      8945 FC            mov dword ptr ss:[ebp-4],eax
008811A7      FF15 70208C00      call dword ptr ds:[8C2070]
008811AD      8D85 30FDFFFF      lea eax,dword ptr ss:[ebp-2D0]
008811B3      8BCB               mov ecx,ebx
008811B5      50                 push eax
008811B6      FF75 08            push dword ptr ss:[ebp+8]
008811B9      E8 E5FEFFFF        call 008810A3
008811BE      8B85 E8FDFFFF      mov eax,dword ptr ss:[ebp-218]
008811C4      B9 60ED8D00        mov ecx,8DED60
008811C9      8BF8               mov edi,eax
008811CB      2B43 04            sub eax,dword ptr ds:[ebx+4]
008811CE      50                 push eax
008811CF      E8 E6FE0300        call 008C10BA
008811D4      50                 push eax
008811D5      E8 F1010000        call 008813CB
008811DA      8BC8               mov ecx,eax
008811DC      E8 FC010000        call 008813DD
```
// 判断是否是 `SDK`

```
008811E1      8BF0               mov esi,eax
008811E3      85F6               test esi,esi
008811E5      74 3F              je short 00881226
008811E7      66:837B 08 01      cmp word ptr ds:[ebx+8],1
008811EC      75 3D              jnz short 0088122B
008811EE      8D85 30FDFFFF      lea eax,dword ptr ss:[ebp-2D0]
008811F4      8BCE               mov ecx,esi
008811F6      50                 push eax
008811F7      E8 E7550100        call 008967E3
008811FC      8BCB               mov ecx,ebx
008811FE      E8 8AFEFFFF        call 0088108D
00881203      83F8 04            cmp eax,4
```
// 记数

```
00881206      72 14              jb short 0088121C
```
// `Patch⑤`: `NOP` 强制解码

```
00881208      8BCE               mov ecx,esi
0088120A      E8 BA540100        call 008966C9
0088120F      83F8 04            cmp eax,4
```
// `Patch⑥`: `cmp eax,6` 解码字节数

```
00881212      72 08              jb short 0088121C
```
// `Patch⑦`: `ja short 0088121C` 超过 6 位则不解码

```
00881214      57                 push edi
00881215      8BCE               mov ecx,esi
00881217      E8 FE540100        call 0089671A
```
// 解码

```
0088121C      56                 push esi
0088121D      8BCB               mov ecx,ebx
0088121F      E8 33FEFFFF        call 00881057
00881224      EB 05              jmp short 0088122B
0088140E      394424 14          cmp dword ptr ss:[esp+14],eax
00881412      74 0B              je short 0088141F
00881414      45                 inc ebp
00881415      81FD 80000000      cmp ebp,80
0088141B      72 CF              jb short 008813EC
0088141D      EB 0C              jmp short 0088142B
0088141F      8D1C76             lea ebx,dword ptr ds:[esi+esi*2]
00881422      C1E3 04            shl ebx,4
00881425      81C3 E0EE8D00      add ebx,8DEEE0
0088142B      5F                 pop edi
```



```
0088142C     5E             pop esi
0088142D     8BC3           mov eax,ebx
0088142F     5D             pop ebp
00881430     5B             pop ebx
00881431     C2 0400        retn 4
00881434     A1 DCED8D00    mov eax,dword ptr ds:[8DEDDC]
00881439     C3             retn
```

//Patch⑧修改为：

```
0088141D     EB 15          jmp short 00881434
0088141F     8D1C76         lea ebx,dword ptr ds:[esi+esi*2]
00881422     C1E3 04        shl ebx,4
00881425     81C3 E0EE8D00  add ebx,8DEEE0
0088142B     5F             pop edi
0088142C     5E             pop esi
0088142D     8BC3           mov eax,ebx
0088142F     5D             pop ebp
00881430     5B             pop ebx
00881431     C2 0400        retn 4
00881434     33DB           xor ebx,ebx
```

//若不是 SDK，则 ebx 清 0 使其跳转

```
00881436     EB F3          jmp short 0088142B
```

————————————————————————

好了，当我们中断在 `1003E139` 处时所有 SDK 都修复完毕了。

现在把代码段 `10001000`－`10036000` 数据复制下来，用 `WinHex` 复制数据写入到 `dump\_.dll` 相应处

—————————————————————————————————

### 七、PE 优化+修复重定位表

把 `dump\_.dll` 复制一份，另存为 `UnPacKed.dll`

用 `LordPE` 把 `UnPacKed.dll` 后面 2 个壳区段删除，用 `WinHex` 删除 `0X00057000` 至末尾的数据

可以用 `ImportREC` 把输入表放在 `RVA`=`0003D380` 处，当然，也可以放在其他可用的空白处

修正各区段的 `RSize` 和 `VSize` 为实际值

由于删除了壳区段，重定位表部分需要调整

把下面壳区段的重定位表数据清 0

```
000560A0                                             00 A0 05 00
000560B0   20 00 00 00 63 30 6B 30  74 30 81 30 95 30 C9 30
000560C0   E1 30 E6 30 F3 30 00 31  0E 31 00 00 00 C0 05 00
000560D0   10 00 00 00 8A 33 92 33  73 3F 00 00 00 00 00 00
000560E0   00 00 00 00 60 3B 68 3B  70 3B 78 3B 80 3B 88 3B
000560F0   90 3B 98 3B A0 3B A8 3B  B0 3B B8 3B C0 3B C8 3B
00056100   D0 3B 6C 3C 70 3C 00 00  00 00 00 00 00 00 00 00
```

最重要的一点是，第五步的“类 SDK 输入表函数调用地址”要加到重定位表里面，如：

```
1002DE50     FF15 00610310      call dword ptr ds:[10036100] ; kernel32.HeapCreate
```

这部分操作比较麻烦，可以手动添加后修正相应 `Size`。

也可以在完成上面六步修复后复制 `AdobeLM.dll`，在当前 `OllyDBG` 里面直接写代码 `Load` 复制的 `AdobeLM.dll`，同样的方法修复 SDK 后再用 `Relox` 修复最终的重定位表。

#### Game Over

—————————————————————————————————

```
          ,     \_/  
        /| \_.-~/      \\\     ,        青春都一晌
       ( /~   /         \\~-.\_ |\
       \`\\\\  \_/           \   ~\ )          忍把浮名 
   _-~~~-.)  )\_\_/;;,.     \\\  //'
 /'\_,\   --~   \ ~~~-  ,;;\\\_\_\_  (.-~~~-.        换了破解轻狂
\`~ \_( ,\_..--\ (     ,;'' /    ~--   /.\_\`\\
  /~~//'   /' \`~\\         ) /--..\_, )\_  \`~
  "  \`~"  "       \""      /~'\`\\    \`\\\\~~\  
                     "     "   "~'  ""
```

UnPacked By :  heXer、fly

2005-12-24 00:00

`4dd466d0e1e0cc123afbe504da2462ca`
