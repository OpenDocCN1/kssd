
# ArmadilloCopyMem-ll+Debug-Block

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-590.htm`](https://www.pediy.com/kssd/pediy07/pediy7-590.htm)

下载地址：`http://www.2wwwmx.com/down/WWvip2.1.exe`

脱壳声明：只为交流！学习提高！

这是一个非标准壳~~至于是 `3.X` 还是 `4.X` 的我不好判断~~

`6` 次断点法！

```
bp WaitForDebugEvent
bp WriteProcessMemory
he WaitForDebugEvent
bp DebugActiveProcess
BP OpenMutexA
BP GetModuleHandleA+5
```

`OD` 载入

```
0058FD93  >/$  55             push ebp
0058FD94  |.  8BEC           mov ebp,esp
0058FD96  |.  6A FF          push -1
```

纪录 `55 8B`

下断 `bp WaitForDebugEvent`

```
0012BCBC   0057FF8F  /CALL 到 WaitForDebugEvent 来自 2wmxvip2.0057FF89
0012BCC0   0012CD90  |pDebugEvent = 0012CD90  //转存中跟随
0012BCC4   000003E8  \Timeout = 1000. ms
```

取消断点，下断 `bp WriteProcessMemory`

```
0012CDA8  004ECC04  2wmxvip2.004ECC04
0012CDAC  00000002
0012CDB0  00000000
0012CDB4  004ECC04  2wmxvip2.004ECC04  //004ECC04 就是 OEP 的位置了
0012CDB8  004ECC04  2wmxvip2.004ECC04
```

取消断点，重新载入~下 `he WaitForDebugEvent`

```
7C85A268 >  8BFF          mov edi,edi  //中断在这里~
7C85A26A   55            push ebp
7C85A26B   8BEC          mov ebp,esp
7C85A26D   83EC 68       sub esp,68
```

取消断点，返回！

```
0057FF8F  .  85C0          test eax,eax  //返回到这里~
0057FF91  .  0F84 AC260000 je 2wmxvip2.00582643
0057FF97  .  8B85 FCFDFFFF mov eax,dword ptr ss:[ebp-204]
0057FF9D  .  25 FF000000   and eax,0FF
0057FFA2  .  85C0          test eax,eax
0057FFA4  .  74 13         je short 2wmxvip2.0057FFB9
0057FFA6  .  8B0D 1CB35B00 mov ecx,dword ptr ds:[5BB31C]
0057FFAC  .  8379 20 00    cmp dword ptr ds:[ecx+20],0
```

搜索---全部常数---`FFFFFFF8`

参考位于 `2wmxvip2:.text` 到常数`-8`，项目 `1888`

`地址=0058055F`           //双击来到代码处
`反汇编=or eax,FFFFFFF8`

。。。。。。`8` 个位置

```
00580513  > \83BD CCF5FFFF>cmp dword ptr ss:[ebp-A34],0  //下硬件执行，把[ebp-A34]清 0！并记
0058051A  .  0F8C A8020000 jl 2wmxvip2.005807C8
00580520  .  8B8D CCF5FFFF mov ecx,dword ptr ss:[ebp-A34]
00580526  .  3B0D 20B35B00 cmp ecx,dword ptr ds:[5BB320]  //记住这个
0058052C  .  0F8D 96020000 jge 2wmxvip2.005807C8
00580532  .  8B95 40F6FFFF mov edx,dword ptr ss:[ebp-9C0]
00580538  .  81E2 FF000000 and edx,0FF
0058053E  .  85D2          test edx,edx
00580540  .  0F84 AD000000 je 2wmxvip2.005805F3
00580546  .  6A 00         push 0
00580548  .  8BB5 CCF5FFFF mov esi,dword ptr ss:[ebp-A34]
0058054E  .  C1E6 04      shl esi,4
00580551  .  8B85 CCF5FFFF mov eax,dword ptr ss:[ebp-A34]
00580557  .  25 07000080  and eax,80000007
0058055C  .  79 05         jns short 2wmxvip2.00580563
0058055E  .  48            dec eax
0058055F  .  83C8 F8      or eax,FFFFFFF8  //来到这里
00580562  .  40            inc eax
00580563  >  33C9         xor ecx,ecx
```

`Stack ss:[0012CD7C]=000000EB`
`Jumps from 00580356, 0058050C`

`Stack ss:[0012CD7C]=00000000`  //清为 0
`Jumps from 00580356, 0058050C`

```
005805D6  .  50            push eax
005805D7  .  8B8D CCF5FFFF mov ecx,dword ptr ss:[ebp-A34]
005805DD  .  51            push ecx
005805DE  .  E8 B0200000   call 2wmxvip2.00582693
005805E3  .  83C4 0C      add esp,0C
005805E6  .  25 FF000000  and eax,0FF       //Patch！！
005805EB  .  85C0          test eax,eax
005805ED  .  0F84 D5010000 je 2wmxvip2.005807C8
005805F3  >  837D D8 00    cmp dword ptr ss:[ebp-28],0
```

`Patch!!`

```
005805E6     FF05 7CCD1200 inc dword ptr ds:[12CD7C]
005805EC     C705 24B35B00>mov dword ptr ds:[5BB324],1
005805F6    ^ E9 18FFFFFF  jmp 00580513
```

到 `005807C8` 下硬件执行！运行！去掉断点！然后用 `LordPE` 来 `DUMP`！

再用 `LordPE` 修改 `OEP` 为 `ECC04`！

`手动寻找 IAT！！`

另外开 `OD`，载入 `dump.exe`！

```
004ECC04 d>  55            push ebp //入口
004ECC05   8BEC           mov ebp,esp
004ECC07   83C4 F0        add esp,-10
004ECC0A   53             push ebx
004ECC0B   B8 84C84E00    mov eax,dumped.004EC884
004ECC10   E8 5B95F1FF    call dumped.00406170  //F7 进
004ECC15   8B1D 8CF84E00  mov ebx,dword ptr ds:[4EF88C]     ; dumped.004F0BEC
004ECC1B   8B03          mov eax,dword ptr ds:[ebx]
00406170   53             push ebx //到这里~
00406171   8BD8          mov ebx,eax
00406173   33C0          xor eax,eax
00406175   A3 A0D04E00    mov dword ptr ds:[4ED0A0],eax
0040617A   6A 00          push 0
0040617C   E8 2BFFFFFF    call dumped.004060AC  //F7 进
00406181   A3 64064F00    mov dword ptr ds:[4F0664],eax
00406186   A1 64064F00    mov eax,dword ptr ds:[4F0664]
004060A5   A3 30004F00    mov dword ptr ds:[4F0030],eax
004060AA   C3             retn
004060AB   90             nop
004060AC  - FF25 50025400 jmp dword ptr ds:[540250]  //到这里
004060B2   8BC0          mov eax,eax
004060B4  - FF25 4C025400 jmp dword ptr ds:[54024C]       ; kernel32.LocalAlloc
004060BA   8BC0          mov eax,eax
004060BC  - FF25 48025400 jmp dword ptr ds:[540248]       ; kernel32.TlsGetValue
004060C2   8BC0          mov eax,eax
004060C4  - FF25 44025400 jmp dword ptr ds:[540244]       ; kernel32.TlsSetValue
004060CA   8BC0          mov eax,eax
004060CC   50             push eax
```

鼠标转到转存器窗口，`Ctrl+G` `540250`

```
00540170  00 00 00 00 00 00 00 00  ....
00540178  00 00 00 00 8A 18 93 7C  ..?粓=========>54017C,上面就全是 00 00 了~
00540180  ED 10 92 7C 05 10 92 7C  ?粒?粒
00540188  A1 9F 80 7C 14 9B 80 7C  龡粀鬔粀
00540190  81 9A 80 7C 5D 99 80 7C  骁粀饝粀
00540198  BD 99 80 7C AC 92 80 7C  馽粀銬粀
005401A0  17 A4 80 7C 2D A1 C9 00  ?粀?é
005401A8  37 97 80 7C 94 97 80 7C  霷粀鞔粀
005401B0  7B 97 80 7C 59 B8 80 7C  靻粀?粀
005401B8  C7 A0 80 7C AD 9C 80 7C  ?粀鲭粀
005401C0  E0 C6 80 7C 11 03 81 7C  ?粀?粁
005401C8  6A 67 C9 00 05 A4 80 7C  杪é?粀
005401D0  EE 1E 80 7C 9D 57 C9 00  ?粀垝é
005401D8  50 96 C9 00 57 B3 80 7C  限é?粀
005401E0  7E D4 80 7C 39 A1 C9 00  ?粀?é
005401E8  42 6B C9 00 28 91 C9 00  歂é鄨é
005401F0  D7 EF 80 7C 72 70 C9 00  ?粀灲é
005401F8  A5 7D C9 00 8A 2B 86 7C  綥é?粆
00540200  40 7A 95 7C E1 EA 81 7C  穀粕?粁
00540208  A9 2C 81 7C A1 62 C9 00  ?粁抡é
00540210  46 FA D3 77 98 EC D3 77  ?矓?矓
00540218  D9 96 C9 00 40 EC D3 77  雙é?矓
00540220  CF 62 C9 00 83 78 DA 77  拏é砡矚
00540228  1B 76 DA 77 F0 6B DA 77  瘛矚毰矚
00540230  19 63 C9 00 50 48 0F 77  挙é?眏
00540238  9D C9 11 77 59 4B 0F 77  ?眑?眏
00540240  61 63 C9 00 F5 9B 80 7C  捡é鯵粀
00540248  50 97 80 7C BD 99 80 7C  靐粀馽粀
00540250  50 96 C9 00 A1 62 C9 00  限é抡é  =====>到这里，向上翻看！
00540258  83 78 DA 77 1B 76 DA 77  砡矚瘛矚
00540260  F0 6B DA 77 58 62 C9 00  毰矚托é
```

我们就取 `54017C` 作为起始地址，那么 `RAV=54017C-40000=14017C`

重新载入主程序窗口！

下断 `bp DebugActiveProcess`，`F9`

堆栈

```
0012BCC0   0057FDEA  /CALL 到 DebugActiveProcess 来自 2wmxvip2.0057FDE4
0012BCC4   00000184  \ProcessId = 6EC  //子程序的 ID，这个是会变化的~我们现在看到的就不一样了
```

~

```
0012BCC8   0012FF2C
```

`OK`，我们就用刚才载入 `dump.exe` 的那个 `OD`，附加 `6EC` 子程序！

然后按 `Alt+F9` 返回！
