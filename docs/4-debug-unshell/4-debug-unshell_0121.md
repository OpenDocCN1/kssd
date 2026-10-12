
# AntiCrack Protector 1.0x

> 原文：[`www.pediy.com/kssd/pediy06/pediy6909.htm`](https://www.pediy.com/kssd/pediy06/pediy6909.htm)

**【脱文标题】** `AntiCrack Protector 1.0x` 之 `Divx Avi Asf Wmv Wma Rm Rmvb 3.23` 脱壳破解

**【脱文作者】** `weiyi75[Dfcg]`

**【作者邮箱】** `weiyi75@sohu.com`

**【作者主页】** `Dfcg 官方大本营`

**【使用工具】** `Peid`, `ZeroAdd`, `UnkillOd`, `ImportREC1.42`

**【脱壳平台】** `Win2000/XP`

**【软件名称】** `Divx Avi Asf Wmv Wma Rm Rmvb 修复器 2.23`

**【下载地址】** `http://www1.skycn.com/soft/11574.html`

**【软件简介】** 只需轻松的一次点击就可以修复不能拖动的或者不能播放的 `divx avi asf wmv wma rm rmvb` 文件。`Divx Avi Asf Wmv Wma Rm Rmvb 修复器`可以修复你通过 `http,ftp,mms,rtsp` 方式由于某些原因没有下载完全的 `divx avi asf wmv wma rm rmvb` 文件。修复后的文件可以流畅的播放，自由的拖动。`Divx Avi Asf Wmv Wma Rm Rmvb 修复器`也可以修复在播放过程中不能拖动的 `divx avi asf wmv wma rm rmvb` 文件。修复后的文件可以随意的拖动。`Divx Avi Asf Wmv Wma Rm Rmvb 修复器`还有另一个功能，它可以强行修复部分损坏的 `divx avi asf wmv wma rm rmvb` 文件。修复后的文件可以跳过坏的数据块，继续播放。如果一些播放器，例如 `Mediaplayer,realplayer` 提示 `dvix avi asf wmv wma rm rmvb` 文件不能播放或文件损坏，都可以尝试用 `Divx Avi Asf Wmv Wma Rm Rmvb 修复器`来修复。用 `Divx Avi Asf Wmv Wma Rm Rmvb 修复器`修复后的文件可以让一些多媒体编辑软件例如 `VirtualDub,RealProducer Plus` 进行进一步的操作，例如合并，分割，格式转换等。`Divx Avi Asf Wmv Wma Rm Rmvb 修复器`修得的成功率达到了 `80%-90%`。

**【软件大小】** `1.29M`

**【加壳方式】** `AntiCrack Protector 1.0x -> RISCO Software Inc`

**【保护方式】** `10` 天试用期限制，`NAG` 注册提示，`RsaKey` 功能保护。

**【脱壳声明】** 我是一只小菜鸟，偶得一点心得，愿与大家分享：

---

## 前言

首先我假设你具备了 `Acprotect` 几个版本的脱壳经验，`OD` 操作等级 `5` 级以上，拥有上面所有调试工具和使用方法，可以开始了，否则 `Exit`。

俗话说一个好汉三个帮，脱壳手也是一样。

首先你要参考这些文章

*   `模拟跟踪+修复方法之 ACProtect 脱壳——完美卸载 XP V9.12`
*   `梦幻 Ollydbg 之 ACPr 修复篇 Divx Avi Asf Wmv Wma Rm Rmvb V3.23`
*   `UltraProtect 1.x 代码段的还原`

## 一. Replace Code

首先要增加一个区段存放 `Replace Code`，大小估计 `2500` 足够，大些文件太肥。

`Zeroadd` 增加区段名 `Dfcg`，大小 `10000`。

`UnkillOd` 载入程序，不忽略内存异常，其余全部忽略。

```
006E2000  v>  60            pushad //入口
006E2001     F8            clc
006E2002     43            inc ebx
006E2003     87D7          xchg edi,edx
006E2005     F8            clc
006E2006     E8 01000000    call videofix.006E200C
006E200B     E8 83042406    call 06922493
006E2010     C3            retn
006E2011     0F83 05000000  jnb videofix.006E201C
006E2017     BA A1FB2289    mov edx,8922FBA1
006E201C     50            push eax
006E201D     E8 01000000    call videofix.006E2023
```

命令行下

`BP GlobalAlloc+5`

`F9` 运行

中断 `5` 次，注意堆栈友好提示。

```
77E7911F     68 4092E777      push KERNEL32.77E79240 //第 5 次中断，清除断点。
77E79124     68 FD13E877      push KERNEL32.77E813FD
77E79129     64:A1 00000000   mov eax,dword ptr fs:[0]
77E7912F     50               push eax
77E79130     64:8925 00000000 mov dword ptr fs:[0],esp
77E79137     51               push ecx
77E79138     51               push ecx
77E79139     83EC 14          sub esp,14
77E7913C     53               push ebx
77E7913D     56               push esi
77E7913E     57               push edi
77E7913F     8965 E8          mov dword ptr ss:[ebp-18],esp
```

堆栈友好提示

```
0012FF24    FFFFFFFF
0012FF28    002E1000
0012FF2C    006E5BB1 返回到 videofix.006E5BB1
0012FF30    00000040
```

```
006E5BB1     8BF8            mov edi,eax //EAX=001367E0 申请的低位区段
```

狸猫换太子，`Alt+M` 查看 `Dfcg` 区段是 `703000`

于是修改信息框处 `EAX` 为 `703000`

```
006E5BB3     81C7 A00F0000    add edi,0FA0
006E5BB9     50               push eax
006E5BBA     B9 70170000      mov ecx,1770
006E5BBF     8DB5 05204000    lea esi,dword ptr ss:[ebp+402005]
006E5BC5     F3:A4            rep movs byte ptr es:[edi],byte pt>
006E5BC7     5A               pop edx
006E5BC8     8BF2             mov esi,edx
006E5BCA     81C6 A00F0000    add esi,0FA0
006E5BD0     8BFE             mov edi,esi
006E5BD2     B9 70170000      mov ecx,1770
006E5BD7     AC               lods byte ptr ds:[esi]
....................................................................
```

`F9` 运行遇到 `Int1` 中断

`Alt+M` 打开内存镜像

内存镜像，项目 `12`

```
 地址=00401000 //对准这里下 F2 断点，Shift+F9 飞向光明之巅
 大小=000DB000 (897024.)
 Owner=videofix 00400000
 区段=CODE
 包含=code
 类型=Imag 01001002
 访问=R
 初始访问=RWE
```

```
004DBA0A     53            push ebx //OD 插件不选择重建输入表脱壳，复制 DBA0A 供 IR 修复用。
004DBA0B     B8 4CB54D00    mov eax,videofix.004DB54C
004DBA10     E8 23B6F2FF    call videofix.00407038
004DBA15     8B1D A0DE4D00  mov ebx,dword ptr ds:[4DDEA0]     ; videofix.004DFC38
004DBA1B     8B03           mov eax,dword ptr ds:[ebx]
004DBA1D     E8 C2B0F8FF    call videofix.00466AE4
004DBA22     8B0D 20E04D00  mov ecx,dword ptr ds:[4DE020]     ; videofix.00500884
004DBA28     8B03           mov eax,dword ptr ds:[ebx]
004DBA2A     8B15 B0614D00  mov edx,dword ptr ds:[4D61B0]     ; videofix.004D61FC
004DBA30     E8 C7B0F8FF    call videofix.00466AFC
004DBA35     8B0D B4E04D00  mov ecx,dword ptr ds:[4DE0B4]     ; videofix.004DFEC0
004DBA3B     8B03           mov eax,dword ptr ds:[ebx]
004DBA3D     8B15 28FE4C00  mov edx,dword ptr ds:[4CFE28]     ; videofix.004CFE74
004DBA43     E8 B4B0F8FF    call videofix.00466AFC
...................................................................................
```

关闭 `OD`，单独运行程序，运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `DBA0A`，点 `IT AutoSearch`，点“`Get Import`”，用“追踪层次 `3`”修复之，`FixDump`，脱壳就完成了。

## 二. 功能修复。

这个版本没有入口校验，运行脱壳程序，提示还剩 `1` 天，其实是`-1` 天，点修复按钮提示过期。

### 目标 1，去除 NAG

看 `NAG` 虽然漂亮，但还是 `MessageBox` 消息窗口，难怪 `Acptect` 单独照顾 `MessageBox Api`。

`OD` 载入程序，命令行

`bp MessageBoxA`

`F9` 运行

```
77E23D68 u>  55            push ebp //中断后清除断点
77E23D69     8BEC          mov ebp,esp
77E23D6B     51            push ecx
77E23D6C     833D B884E477 00  cmp dword ptr ds:[77E484B8],0
77E23D73     74 29         je short user32.77E23D9E
77E23D75     64:A1 18000000 mov eax,dword ptr fs:[18]
77E23D7B     8B40 24       mov eax,dword ptr ds:[eax+24]
77E23D7E     8945 FC       mov dword ptr ss:[ebp-4],eax
77E23D81     B8 00000000    mov eax,0
77E23D86     B9 8088E477    mov ecx,user32.77E48880
77E23D8B     8B55 FC       mov edx,dword ptr ss:[ebp-4]
```

堆栈友好提示

```
0012FB68    004DA890  /CALL 到 MessageBoxA 来自 Dump\_.004DA88B //Alt+F9 返回看看
0012FB6C    FFFFFFFF  |hOwner = FFFFFFFF
0012FB70    0012FBB2  |Text = ""
0012FB74    00000000  |Title = NULL
0012FB78    00000000  \Style = MB_OK|MB_APPLMODAL
0012FB7C    0012FBE4  指针到下一个 SEH 记录
0012FB80    004DA9BC  SE 句柄
0012FB84    0012FBD4
0012FB88    00460018  Dump\_.00460018
```

```
004DA88B     E8 68D2F2FF      call <jmp.&user32.MessageBoxA>
004DA890     6A 01            push 1 //返回到这里
004DA892     6A 00            push 0
004DA894     8D45 FF          lea eax,dword ptr ss:[ebp-1]
004DA897     50               push eax
004DA898     6A FF            push -1
004DA89A     E8 59D2F2FF      call <jmp.&user32.MessageBoxA>
004DA89F     8D45 C8          lea eax,dword ptr ss:[ebp-38]
004DA8A2     8D55 DE          lea edx,dword ptr ss:[ebp-22]
004DA8A5     B9 21000000      mov ecx,21
004DA8AA     E8 75A4F2FF      call Dump\_.00404D24
004DA8AF     8B45 C8          mov eax,dword ptr ss:[ebp-38]
004DA8B2     8D55 CC          lea edx,dword ptr ss:[ebp-34]
004DA8B5     E8 E2E8F2FF      call Dump\_.0040919C
004DA8BA     837D CC 00       cmp dword ptr ss:[ebp-34],0 //这里的 dword ptr ss:[ebp-34]如果你有 Rsakey 就保存名字，如果强行写入数据后面的校验还是无法通过。
004DA8BE     74 7C            je short Dump\_.004DA93C //如果这里不跳，你就无 NaG 并伪注册了。
```

哪有那么简单，这里的代码是前面动态生成的，这段代码是核心，我们必须保存出来，往下还有一段自毁代码。

```
004DA8C0     33D2             xor edx,edx
004DA8C2     8B86 24030000    mov eax,dword ptr ds:[esi+324]
004DA8C8     E8 8BB4F6FF      call Dump\_.00445D58
004DA8CD     33D2             xor edx,edx
```



```
004DA8CF     8B86 28030000        mov eax,dword ptr ds:[esi+328]  

004DA8D5     E8 7EB4F6FF         call Dump_.00445D58  

004DA8DA     B2 01               mov dl,1  

004DA8DC     A1 34CB4600        mov eax,dword ptr ds:[46CB34]  

004DA8E1     E8 4E23F9FF        call Dump_.0046CC34  

004DA8E6     8BD8                mov ebx,eax  

004DA8E8     BA 01000080        mov edx,80000001  

004DA8ED     8BC3                mov eax,ebx  

004DA8EF     E8 E023F9FF        call Dump_.0046CCD4  

004DA8F4     33C9                xor ecx,ecx  

004DA8F6     BA D4A94D00        mov edx,Dump_.004DA9D4              ; ASCII "\Software\FixVideo\VideoFixer\"  

004DA8FB     8BC3                mov eax,ebx  

004DA8FD     E8 3624F9FF        call Dump_.0046CD38  

004DA902     84C0                test al,al  

004DA904     74 79               je short Dump_.004DA97F  

004DA906     8D4D C0             lea ecx,dword ptr ss:[ebp-40]  

004DA909     BA 1CAA4D00        mov edx,Dump_.004DAA1C              ; ASCII "UserName"  

.........................................................................................  

自毁代码  

004DA97A     E8 E9C2F8FF        call Dump_.00466C68  

004DA97F     60                  pushad          //开始  

004DA980     E8 00000000        call Dump_.004DA985  

004DA985     5F                  pop edi  

004DA986     81EF BB330000      sub edi,33BB  

004DA98C     B9 B9330000        mov ecx,33B9  

004DA991     0F31                rdtsc  

004DA993     8907                mov dword ptr ds:[edi],eax  

004DA995     83C7 04             add edi,4  

004DA998     83E9 04             sub ecx,4  

004DA99B     83F9 04             cmp ecx,4  

004DA99E   ^ 73 F1               jnb short Dump_.004DA991  

004DA9A0     61                  popad           //结束  

004DA9A1     33C0                xor eax,eax  

对付很容易，修改  

004DA97F     60                  pushad          

为  

004DA97F    /EB 20               jmp short Dump_.004DA9A1  

004DA981    |90                  nop  

004DA982    |90                  nop  

004DA983    |90                  nop  

004DA984    |90                  nop  

我们再确认刚才保护的代码范围  

..................................................................................  

004DA712     90                  nop   //开始，必须全力保证这段不被重新解码  

004DA713     90                  nop  

004DA714     90                  nop  

004DA715     90                  nop  

004DA716     90                  nop  

004DA717     90                  nop  

004DA718     90                  nop  

004DA719     90                  nop  

004DA71A     90                  nop  

004DA71B     90                  nop  

004DA71C     90                  nop  

004DA71D     90                  nop  

004DA71E     90                  nop  

004DA71F     90                  nop  

004DA720     90                  nop  

004DA721     90                  nop  

004DA722     90                  nop  

004DA723     90                  nop  

004DA724     90                  nop  

004DA725     90                  nop  

004DA726     90                  nop  

004DA727     90                  nop  

004DA728     90                  nop  

004DA729     90                  nop  

004DA72A     90                  nop  

004DA72B     90                  nop  

004DA72C     90                  nop  

004DA72D     90                  nop  

004DA72E     90                  nop  

004DA72F     90                  nop  

004DA730     90                  nop  

004DA731     90                  nop  

004DA732     90                  nop  

004DA733     90                  nop  

004DA734     90                  nop  

004DA735     90                  nop  

004DA736     90                  nop  

004DA737     90                  nop  

004DA738     90                  nop  

004DA739     90                  nop  

004DA73A     90                  nop  

004DA73B     90                  nop  

004DA73C     90                  nop  

004DA73D     90                  nop  

004DA73E     90                  nop  

004DA73F     90                  nop  

004DA740     90                  nop  

004DA741     90                  nop  

004DA742     90                  nop  

004DA743     90                  nop  

004DA744     90                  nop  

004DA745     90                  nop  

004DA746     90                  nop  

004DA747     90                  nop  

004DA748     90                  nop  

004DA749     90                  nop  

004DA74A     90                  nop  

004DA74B     90                  nop  

004DA74C     90                  nop  

004DA74D     90                  nop  

004DA74E     90                  nop  

004DA74F     90                  nop  

004DA750     90                  nop  

004DA751     90                  nop  

004DA752     90                  nop  

004DA753     90                  nop  

004DA754     90                  nop  

004DA755     90                  nop  

004DA756     90                  nop  

004DA757     90                  nop  

004DA758     90                  nop  

004DA759     90                  nop  

004DA75A     90                  nop  

004DA75B     90                  nop  

004DA75C     90                  nop  

004DA75D     90                  nop  

004DA75E     90                  nop  

004DA75F     90                  nop  

004DA760     90                  nop  

004DA761     90                  nop  

004DA762     90                  nop  

004DA763     90                  nop  

004DA764     90                  nop  

004DA765     90                  nop  

004DA766     90                  nop  

004DA767     90                  nop  

004DA768     90                  nop  

004DA769     90                  nop  

004DA76A     90                  nop  

004DA76B     90                  nop  

004DA76C     90                  nop  

004DA76D     90                  nop  

004DA76E     90                  nop  

004DA76F     90                  nop  

004DA770     90                  nop  

004DA771     90                  nop  

004DA772     90                  nop  

004DA773     90                  nop  

004DA774     90                  nop  

004DA775     90                  nop  

004DA776     90                  nop  

004DA777     90                  nop  

004DA778     90                  nop  

004DA779     90                  nop  

004DA77A     90                  nop  

004DA77B     90                  nop  

004DA77C     90                  nop  

004DA77D     90                  nop  

004DA77E     90                  nop  

004DA77F     90                  nop  

004DA780     90                  nop  

004DA781     90                  nop  

004DA782     90                  nop  

004DA783     90                  nop  

004DA784     90                  nop  

004DA785     90                  nop  

004DA786     90                  nop  

004DA787     90                  nop  

004DA788     90                  nop  

004DA789     90                  nop  

004DA78A     90                  nop  

004DA78B     90                  nop  

004DA78C     90                  nop  

004DA78D     90                  nop  

004DA78E     90                  nop  

004DA78F     90                  nop  

004DA790     90                  nop  

004DA791     90                  nop  

004DA792     90                  nop  

004DA793     90                  nop  

004DA794     90                  nop  

004DA795     90                  nop  

004DA796     90                  nop  

004DA797     90                  nop  

004DA798     90                  nop  

004DA799     90                  nop  

004DA79A     90                  nop  

004DA79B     90                  nop  

004DA79C     90                  nop  

004DA79D     90                  nop  

004DA79E     90                  nop  

004DA79F     90                  nop
```



```
004DA7A0     90                  nop
004DA7A1     90                  nop
004DA7A2     90                  nop
004DA7A3     90                  nop
004DA7A4     90                  nop
004DA7A5     90                  nop
004DA7A6     90                  nop
004DA7A7     90                  nop
004DA7A8     90                  nop
004DA7A9     90                  nop
004DA7AA     90                  nop
004DA7AB     90                  nop
004DA7AC     90                  nop
004DA7AD     90                  nop
004DA7AE     90                  nop
004DA7AF     90                  nop
004DA7B0     90                  nop
004DA7B1     90                  nop
004DA7B2     90                  nop
004DA7B3     90                  nop
004DA7B4     90                  nop
004DA7B5     90                  nop
004DA7B6     90                  nop
004DA7B7     90                  nop
004DA7B8     90                  nop
004DA7B9     90                  nop
004DA7BA     90                  nop
004DA7BB     90                  nop
004DA7BC     90                  nop
004DA7BD     90                  nop
004DA7BE     90                  nop
004DA7BF     90                  nop
004DA7C0     90                  nop
004DA7C1     90                  nop
004DA7C2     90                  nop
004DA7C3     90                  nop
004DA7C4     90                  nop
004DA7C5     90                  nop
004DA7C6     90                  nop
004DA7C7     90                  nop
004DA7C8     90                  nop
004DA7C9     90                  nop
004DA7CA     90                  nop
004DA7CB     90                  nop
004DA7CC     90                  nop
004DA7CD     61                  popad
004DA7CE     B2 01               mov dl,1
004DA7D0     A1 34CB4600         mov eax,dword ptr ds:[46CB34]
004DA7D5     E8 5A24F9FF         call Dump_.0046CC34
004DA7DA     8BD8                mov ebx,eax
004DA7DC     BA 01000080         mov edx,80000001
004DA7E1     8BC3                mov eax,ebx
004DA7E3     E8 EC24F9FF         call Dump_.0046CCD4
004DA7E8     33C9                xor ecx,ecx
004DA7EA     BA D4A94D00         mov edx,Dump_.004DA9D4         ; ASCII "\\Software\\FixVideo\\VideoFixer\\"
004DA7EF     8BC3                mov eax,ebx
004DA7F1     E8 4225F9FF         call Dump_.0046CD38
004DA7F6     84C0                test al,al
004DA7F8     74 1D               je short Dump_.004DA817
004DA7FA     8D4D D8             lea ecx,dword ptr ss:[ebp-28]
004DA7FD     BA FCA94D00         mov edx,Dump_.004DA9FC         ; ASCII "Times"
004DA802     8BC3                mov eax,ebx
004DA804     E8 F726F9FF         call Dump_.0046CF00
004DA809     8B55 D8             mov edx,dword ptr ss:[ebp-28]
004DA80C     8D86 A0030000       lea eax,dword ptr ds:[esi+3A0]
004DA812     E8 F9A2F2FF         call Dump_.00404B10
004DA817     33C9                xor ecx,ecx
004DA819     BA D4A94D00         mov edx,Dump_.004DA9D4         ; ASCII "\\Software\\FixVideo\\VideoFixer\\"
004DA81E     8BC3                mov eax,ebx
004DA820     E8 1325F9FF         call Dump_.0046CD38
004DA825     84C0                test al,al
004DA827     74 24               je short Dump_.004DA84D
004DA829     8D4D D4             lea ecx,dword ptr ss:[ebp-2C]
004DA82C     BA 0CAA4D00         mov edx,Dump_.004DAA0C         ; ASCII "Date"
004DA831     8BC3                mov eax,ebx
004DA833     E8 C826F9FF         call Dump_.0046CF00
004DA838     8B55 D4             mov edx,dword ptr ss:[ebp-2C]
004DA83B     8D86 A8030000       lea eax,dword ptr ds:[esi+3A8]
004DA841     E8 CAA2F2FF         call Dump_.00404B10
004DA846     8BC3                mov eax,ebx
004DA848     E8 5724F9FF         call Dump_.0046CCA4
004DA84D     8BC3                mov eax,ebx
004DA84F     E8 5894F2FF         call Dump_.00403CAC
004DA854     56                  push esi
004DA855     8D55 D0             lea edx,dword ptr ss:[ebp-30]
004DA858     33C0                xor eax,eax
004DA85A     E8 DD82F2FF         call Dump_.00402B3C
004DA85F     8B55 D0             mov edx,dword ptr ss:[ebp-30]
004DA862     8D8E A4030000       lea ecx,dword ptr ds:[esi+3A4]
004DA868     A1 88085000         mov eax,dword ptr ds:[500888]
004DA86D     E8 524CFFFF         call Dump_.004CF4C4
004DA872     8D45 DE             lea eax,dword ptr ss:[ebp-22]
004DA875     33C9                xor ecx,ecx
004DA877     BA 21000000         mov edx,21
004DA87C     E8 6B8AF2FF         call Dump_.004032EC
004DA881     6A 00               push 0
004DA883     6A 00               push 0
004DA885     8D45 DE             lea eax,dword ptr ss:[ebp-22]
004DA888     50                  push eax
004DA889     6A FF               push -1
004DA88B     E8 68D2F2FF         call <jmp.&user32.MessageBoxA>
004DA890     6A 01               push 1
004DA892     6A 00               push 0
004DA894     8D45 FF             lea eax,dword ptr ss:[ebp-1]
004DA897     50                  push eax
004DA898     6A FF               push -1
004DA89A     E8 59D2F2FF         call <jmp.&user32.MessageBoxA>
004DA89F     8D45 C8             lea eax,dword ptr ss:[ebp-38]
004DA8A2     8D55 DE             lea edx,dword ptr ss:[ebp-22]
004DA8A5     B9 21000000         mov ecx,21
004DA8AA     E8 75A4F2FF         call Dump_.00404D24
004DA8AF     8B45 C8             mov eax,dword ptr ss:[ebp-38]
004DA8B2     8D55 CC             lea edx,dword ptr ss:[ebp-34]
004DA8B5     E8 E2E8F2FF         call Dump_.0040919C
004DA8BA     837D CC 00          cmp dword ptr ss:[ebp-34],0
004DA8BE     74 7C               je short Dump_.004DA93C  //这里是爆破点
```

修改为

```
004DA8BE     90                  nop
004DA8BF     90                  nop
004DA8C0     33D2                xor edx,edx
004DA8C2     8B86 24030000       mov eax,dword ptr ds:[esi+324]
004DA8C8     E8 8BB4F6FF         call Dump_.00445D58
004DA8CD     33D2                xor edx,edx
004DA8CF     8B86 28030000       mov eax,dword ptr ds:[esi+328]
004DA8D5     E8 7EB4F6FF         call Dump_.00445D58
004DA8DA     B2 01               mov dl,1
004DA8DC     A1 34CB4600         mov eax,dword ptr ds:[46CB34]
004DA8E1     E8 4E23F9FF         call Dump_.0046CC34
004DA8E6     8BD8                mov ebx,eax
004DA8E8     BA 01000080         mov edx,80000001
004DA8ED     8BC3                mov eax,ebx
004DA8EF     E8 E023F9FF         call Dump_.0046CCD4
004DA8F4     33C9                xor ecx,ecx
004DA8F6     BA D4A94D00         mov edx,Dump_.004DA9D4         ; ASCII "\\Software\\FixVideo\\VideoFixer\\"
004DA8FB     8BC3                mov eax,ebx
004DA8FD     E8 3624F9FF         call Dump_.0046CD38
004DA902     84C0                test al,al
004DA904     74 79               je short Dump_.004DA97F
004DA906     8D4D C0             lea ecx,dword ptr ss:[ebp-40]
004DA909     BA 1CAA4D00         mov edx,Dump_.004DAA1C         ; ASCII "UserName"
004DA90E     8BC3                mov eax,ebx
004DA910     E8 EB25F9FF         call Dump_.0046CF00
004DA915     8B4D C0             mov ecx,dword ptr ss:[ebp-40]
004DA918     8D45 C4             lea eax,dword ptr ss:[ebp-3C]
004DA91B     BA 30AA4D00         mov edx,Dump_.004DAA30         ; ASCII "
"
004DA920     E8 9BA4F2FF         call Dump_.00404DC0
004DA925     8B55 C4             mov edx,dword ptr ss:[ebp-3C]
004DA928     8B86 90030000       mov eax,dword ptr ds:[esi+390]
004DA92E     E8 35B5F6FF         call Dump_.00445E68
004DA933     8BC3                mov eax,ebx
```



## 解码与保护代码段

004DA935 E8 6A23F9FF call Dump\_.0046CCA4
004DA93A EB 43 jmp short Dump\_.004DA97F
004DA93C 8D55 BC lea edx,dword ptr ss:\[ebp\-44\]
004DA93F 33C0 xor eax,eax
004DA941 8A45 FF mov al,byte ptr ss:\[ebp\-1\]
004DA944 E8 6FEAF2FF call Dump\_.004093B8
004DA949 8B55 BC mov edx,dword ptr ss:\[ebp\-44\]
004DA94C A1 C4DB4D00 mov eax,dword ptr ds:\[4DDBC4\]
004DA951 8B00 mov eax,dword ptr ds:\[eax\]
004DA953 8B80 1C030000 mov eax,dword ptr ds:\[eax+31C\]
004DA959 E8 0AB5F6FF call Dump\_.00445E68
004DA95E A1 C4DB4D00 mov eax,dword ptr ds:\[4DDBC4\]
004DA963 8B00 mov eax,dword ptr ds:\[eax\]
004DA965 8B10 mov edx,dword ptr ds:\[eax\]
004DA967 FF92 E8000000 call dword ptr ds:\[edx+E8\]
004DA96D 807D FF 00 cmp byte ptr ss:\[ebp\-1\],0
004DA971 75 0C jnz short Dump\_.004DA97F
004DA973 A1 A0DE4D00 mov eax,dword ptr ds:\[4DDEA0\]
004DA978 8B00 mov eax,dword ptr ds:\[eax\]
004DA97A E8 74C2F8FF call Dump\_.00466BF3
004DA97F EB 20 jmp short Dump\_.004DA9A1
004DA981 90 nop
004DA982 90 nop
004DA983 90 nop
004DA984 90 nop
004DA985 5F pop edi
004DA986 81EF BB330000 sub edi,33BB
004DA98C B9 B9330000 mov ecx,33B9
004DA991 0F31 rdtsc
004DA993 8907 mov dword ptr ds:\[edi\],eax
004DA995 83C7 04 add edi,4
004DA998 83E9 04 sub ecx,4
004DA99B 83F9 04 cmp ecx,4
004DA99E ^ 73 F1 jnb short Dump\_.004DA991
004DA9A0 61 popad
004DA9A1 33C0 xor eax,eax

...............................................................................

关于这段代码来历就太复杂了。

举个例子

A-B-C-D-E-F-G

`A` 是解码起点，`G` 是我们要保护的代码。

我当时尝试找每个解码段，都失败而归。

正常程序的代码段代码是不写入的，而 `Acprotect` 是边走边解下一段代码，逐级连络。

困惑好久，总算不负众望。

我们直接在 `A` 开始解码就跳到 `G` 不就可以了，管它 `B`-`F` 搞什么飞机。

首先复制

004DA712 90 nop

到

004DA9A1 33C0 xor eax,eax

之间的全部代码到文件中。

然后就是保护它不被解码覆盖。

重启 `OD`。

我们目前处于：

004DBA0A D> $ 53 push ebx
004DBA0B . B8 4CB54D00 mov eax,Dump\_.004DB54C
004DBA10 . E8 23B6F2FF call Dump\_.00407038
004DBA15 . 8B1D A0DE4D00 mov ebx,dword ptr ds:\[4DDEA0\] ; Dump\_.004DFC38
004DBA1B . 8B03 mov eax,dword ptr ds:\[ebx\]
004DBA1D . E8 C2B0F8FF call Dump\_.00466AE4
004DBA22 . 8B0D 20E04D00 mov ecx,dword ptr ds:\[4DE020\] ; Dump\_.00500884
004DBA28 . 8B03 mov eax,dword ptr ds:\[ebx\]
004DBA2A . 8B15 B0614D00 mov edx,dword ptr ds:\[4D61B0\] ; Dump\_.004D61FC
004DBA30 . E8 C7B0F8FF call Dump\_.00466AFC
004DBA35 . 8B0D B4E04D00 mov ecx,dword ptr ds:\[4DE0B4\] ; Dump\_.004DFEC0

代码段，当程序要修改代码段代码必然有内存写入。

于是对准：

```
内存镜像，项目 12
地址=00401000 //这里下内存写入断点
大小=000DB000 (897024.)
Owner=Dump_ 00400000
区段=CODE
包含=code
类型=Imag 01001002
访问=R
初始访问=RWE
```

`F9` 立即中断。

004D75C9 8945 1E mov dword ptr ss:\[ebp+1E\],eax //这里中断，清除内存断点

### 外科手术

```
004D75C9 /E9 44310000 jmp Dump\_.004DA712 //直接到解码位置。
004D75CC EB 01 jmp short Dump\_.004D75CF
004D75CE ^ 7D E9 jge short Dump\_.004D75B9
004D75D0 05 000000BE add eax,BE000000
004D75D5 14 92 adc al,92
004D75D7 93 xchg eax,ebx
004D75D8 298B FDBD6D01 sub dword ptr ds:\[ebx+16DBDFD\],ecx
```

...............................................................................

一切尽在掌握，`Nag` 已经 `Over`。

## 第二处功能

点修复，提示过期。

还是

`bp MessageBox`

```
77E23D68 u> 55 push ebp //立即中断
77E23D69 8BEC mov ebp,esp
77E23D6B 51 push ecx
77E23D6C 833D B884E477 00 cmp dword ptr ds:\[77E484B8\],0
77E23D73 74 29 je short user32.77E23D9E
77E23D75 64:A1 18000000 mov eax,dword ptr fs:\[18\]
77E23D7B 8B40 24 mov eax,dword ptr ds:\[eax+24\]
77E23D7E 8945 FC mov dword ptr ss:\[ebp\-4\],eax
77E23D81 B8 00000000 mov eax,0
77E23D86 B9 8088E477 mov ecx,user32.77E48880
77E23D8B 8B55 FC mov edx,dword ptr ss:\[ebp\-4\]
77E23D8E F0:0FB111 lock cmpxchg dword ptr ds:\[ecx\],ed>
77E23D92 85C0 test eax,eax
```

### 堆栈友好提示

```
0012FC6C 004DACB9 /CALL 到 MessageBoxA 来自 Dump3.004DACB4
0012FC70 FFFFFFFF |hOwner = FFFFFFFF
0012FC74 0012FCA6 |Text = ""
0012FC78 00000000 |Title = NULL
0012FC7C 00000000 \\Style = MB_OK|MB_APPLMODAL
0012FC80 0012FE54 指针到下一个 SEH 记录
0012FC84 004DAE9F SE 句柄
0012FC88 0012FCC8
0012FC8C 004AA548 Dump3.004AA548
```

`Alt+F9` 返回分析一段事件代码

```
004DACA5 E8 4286F2FF call Dump3.004032EC
004DACAA 6A 00 push 0
004DACAC 6A 00 push 0
004DACAE 8D45 DE lea eax,dword ptr ss:\[ebp\-22\]
004DACB1 50 push eax
004DACB2 6A FF push -1
004DACB4 E8 3FCEF2FF call <jmp.&user32.MessageBoxA>
004DACB9 6A 01 push 1
004DACBB 6A 00 push 0
004DACBD 8D45 FF lea eax,dword ptr ss:\[ebp\-1\]
004DACC0 50 push eax
004DACC1 6A FF push -1
004DACC3 E8 30CEF2FF call <jmp.&user32.MessageBoxA>
004DACC8 8D45 D4 lea eax,dword ptr ss:\[ebp\-2C\]
004DACCB 8D55 DE lea edx,dword ptr ss:\[ebp\-22\]
004DACCE B9 21000000 mov ecx,21
004DACD3 E8 4CA0F2FF call Dump3.00404D24
004DACD8 8B45 D4 mov eax,dword ptr ss:\[ebp\-2C\]
004DACDB 8D55 D8 lea edx,dword ptr ss:\[ebp\-28\]
004DACDE E8 B9E4F2FF call Dump3.0040919C
004DACE3 837D D8 00 cmp dword ptr ss:\[ebp\-28\],0
004DACE7 75 27 jnz short Dump3.004DAD10
004DACE9 8D45 CC lea eax,dword ptr ss:\[ebp\-34\]
004DACEC 8D55 DE lea edx,dword ptr ss:\[ebp\-22\]
004DACEF B9 21000000 mov ecx,21
004DACF4 E8 2BA0F2FF call Dump3.00404D24
004DACF9 8B45 CC mov eax,dword ptr ss:\[ebp\-34\]
004DACFC 8D55 D0 lea edx,dword ptr ss:\[ebp\-30\]
004DACFF E8 98E4F2FF call Dump3.0040919C
004DAD04 837D D0 00 cmp dword ptr ss:\[ebp\-30\],0
004DAD08 75 7C jnz short Dump3.004DAD86
004DAD0A 807D FF 00 cmp byte ptr ss:\[ebp\-1\],0
004DAD0E 76 76 jbe short Dump3.004DAD86 //这里，跳走 Over，放 Tnt
004DAD0E 90 nop
004DAD0F 90 nop
004DAD10 B2 01 mov dl,1
004DAD12 8B83 98030000 mov eax,dword ptr ds:\[ebx+398\]
004DAD18 E8 3BB0F6FF call Dump3.00445D58
004DAD1D 33D2 xor edx,edx
004DAD1F 8B83 98030000 mov eax,dword ptr ds:\[ebx+398\]
004DAD25 E8 52D8F9FF call Dump3.0047857C
004DAD2A 8B83 34030000 mov eax,dword ptr ds:\[ebx+334\]
004DAD30 8B80 2C020000 mov eax,dword ptr ds:\[eax+22C\]
004DAD36 E8 E90DFAFF call Dump3.0047BB24
004DAD3B 85C0 test eax,eax
004DAD3D 75 18 jnz short Dump3.004DAD57
```



004DAD3F      8B83 38030000       mov eax,dword ptr ds:`[ebx+338]`  

004DAD45      8B80 20020000       mov eax,dword ptr ds:`[eax+220]`  

004DAD4B      BA B4AE4D00        mov edx,Dump3.004DAEB4  

004DAD50      8B08              mov ecx,dword ptr ds:`[eax]`  

004DAD52      FF51 38          call dword ptr ds:`[ecx+38]`  

004DAD55      EB 4D            jmp short Dump3.004DADA4  

004DAD57      8BC3              mov eax,ebx  

004DAD59      E8 46BEFFFF       call Dump3.004D6BA4  

004DAD5E      B1 01             mov cl,1  

004DAD60      B2 01             mov dl,1  

004DAD62      A1 BC594D00       mov eax,dword ptr ds:`[4D59BC]`  

004DAD67      E8 C05FF4FF       call Dump3.00420D2C  

004DAD6C      8983 9C030000     mov dword ptr ds:`[ebx+39C]`,eax  

004DAD72      C683 AC030000 00  mov byte ptr ds:`[ebx+3AC]`,0  

004DAD79      8B83 9C030000     mov eax,dword ptr ds:`[ebx+39C]`  

004DAD7F      E8 E862F4FF       call Dump3.0042106C  

004DAD84      EB 1E            jmp short Dump3.004DADA4  

004DAD86      8B83 8C030000     mov eax,dword ptr ds:`[ebx+38C]`  

004DAD8C      83C0 68          add eax,68  

004DAD8F      BA CCAE4D00       mov edx,Dump3.004DAECC  

004DAD94      E8 779DF2FF       call Dump3.00404B10  

004DAD99      8B83 8C030000     mov eax,dword ptr ds:`[ebx+38C]`  

004DAD9F      8B10              mov edx,dword ptr ds:`[eax]`  

004DADA1      FF52 30          call dword ptr ds:`[edx+30]`  

004DADA4      8B83 A8030000     mov eax,dword ptr ds:`[ebx+3A8]`  

004DADAA      BA F8AE4D00       mov edx,Dump3.004DAEF8  

004DADAF      E8 04A1F2FF       call Dump3.00404EB8  

004DADB4      75 0A            jnz short Dump3.004DADC0  

004DADB6      E8 69BBFFFF       call Dump3.004D6924  

004DADBB      E9 86000000       jmp Dump3.004DAE46  

004DADC0      E8 B30AFBFF       call Dump3.0048B878  

004DADC5      83C4 F8          add esp,-8  

004DADC8      DD1C24           fstp qword ptr ss:`[esp]`  

004DADCB      9B                wait  

004DADCC      E8 DF0AFBFF       call Dump3.0048B8B0  

004DADD1      8BF0              mov esi,eax  

004DADD3      0FB7C6           movzx eax,si  

004DADD6      B9 0D000000       mov ecx,0D  

004DADDB      33D2              xor edx,edx  

004DADDD      F7F1              div ecx  

004DADDF      85D2              test edx,edx  

004DADE1      75 63            jnz short Dump3.004DAE46  

004DADE3      8B83 A0030000     mov eax,dword ptr ds:`[ebx+3A0]`  

004DADE9      8B93 A4030000     mov edx,dword ptr ds:`[ebx+3A4]`  

004DADEF      E8 C4A0F2FF       call Dump3.00404EB8  

004DADF4      74 50            je short Dump3.004DAE46  

004DADF6      B2 01             mov dl,1  

004DADF8      A1 34CB4600       mov eax,dword ptr ds:`[46CB34]`  

004DADFD      E8 321EF9FF       call Dump3.0046CC34  

004DAE02      8BD8              mov ebx,eax  

004DAE04      BA 01000080       mov edx,80000001  

004DAE09      8BC3              mov eax,ebx  

004DAE0B      E8 C41EF9FF       call Dump3.0046CCD4  

004DAE10      33C9              xor ecx,ecx  

004DAE12      BA 04AF4D00       mov edx,Dump3.004DAF04            ; ASCII "\Software\FixVideo\VideoFixer\"  

004DAE17      8BC3              mov eax,ebx  

004DAE19      E8 1A1FF9FF       call Dump3.0046CD38  

004DAE1E      84C0              test al,al  

004DAE20      74 18            je short Dump3.004DAE3A  

004DAE22      B9 F8AE4D00       mov ecx,Dump3.004DAEF8  

004DAE27      BA 2CAF4D00       mov edx,Dump3.004DAF2C            ; ASCII "Date"  

004DAE2C      8BC3              mov eax,ebx  

004DAE2E      E8 A120F9FF       call Dump3.0046CED4  

004DAE33      8BC3              mov eax,ebx  

004DAE35      E8 6A1EF9FF       call Dump3.0046CCA4  

004DAE3A      8BC3              mov eax,ebx  

004DAE3C      E8 6B8EF2FF       call Dump3.00403CAC  

004DAE41      E8 DEBAFFFF       call Dump3.004D6924  

004DAE46      60                pushad  //又是自毁代码  

004DAE47      60                pushad  

004DAE48      E8 00000000       call Dump3.004DAE4D  

004DAE4D      5E                pop esi  

004DAE4E      83EE 06          sub esi,6  

004DAE51      B9 AD010000       mov ecx,1AD  

004DAE56      29CE              sub esi,ecx  

004DAE58      BA 3081C79D       mov edx,9DC78130  

004DAE5D      C1E9 02          shr ecx,2  

004DAE60      83E9 02          sub ecx,2  

004DAE63      83F9 00          cmp ecx,0  

004DAE66      7C 1A            jl short Dump3.004DAE82  

004DAE68      8B048E           mov eax,dword ptr ds:`[esi+ecx*4]`  

004DAE6B      8B5C8E 04        mov ebx,dword ptr ds:`[esi+ecx*4+4]`  

004DAE6F      2BC3              sub eax,ebx  

004DAE71      C1C8 17          ror eax,17  

004DAE74      33C2              xor eax,edx  

004DAE76      81C2 4B354324     add edx,2443354B  

004DAE7C      89048E           mov dword ptr ds:`[esi+ecx*4]`,eax  

004DAE7F      49                dec ecx  

004DAE80      EB E1            jmp short Dump3.004DAE63  

004DAE82      61                popad  

004DAE83      61                popad //结束自毁代码  

004DAE84      33C0              xor eax,eax  

004DAE86      5A                pop edx  

004DAE87      59                pop ecx  

...........................................................  

还是要保存有用动态代码，摧毁自毁代码，先让它自毁，确认有用代码起点和终点。  

004DAC8E      E8 01000000       call Dump3.004DAC94  //我们这里用花指令插件清理一下  

004DAC93      78 83            js short Dump3.004DAC18  

004DAC95      C4044D F8FC09EA  les eax,fword ptr ds:`[ecx*2+EA09FC]` > //这里开始往下有红色代码  

004DAC9C      43                inc ebx  

004DAC9D      A6                cmps byte ptr ds:`[esi]`,byte ptr es:  

004DAC9E      86FF              xchg bh,bh  

004DACA0      D7                xlat byte ptr ds:`[ebx+al]`  

004DACA1      60                pushad  

004DACA2      C7                ???                                ; 未知命令  

004DACA3      E7 9D            out 9D,eax  

004DACA5      0AF6              or dh,dh  

............................................................  

004DAC8E      90                nop  

004DAC8F      90                nop  

004DAC90      90                nop  

004DAC91      90                nop  

004DAC92      90                nop  

004DAC93      90                nop  

004DAC94      90                nop  

004DAC95      90                nop  

004DAC96      90                nop  

004DAC97      4D                dec ebp  

好啦，可以取认位置  

........................................................................  

004DAC97      4D                dec ebp  //开始  

004DAC98      F8                clc  

004DAC99      FC                cld  

004DAC9A      61                popad  

004DAC9B      8D45 DE          lea eax,dword ptr ss:`[ebp-22]`  

004DAC9E      33C9              xor ecx,ecx  

004DACA0      BA 21000000       mov edx,21  

004DACA5      E8 4286F2FF       call Dump3.004032EC  

004DACAA      6A 00             push 0  

004DACAC      6A 00             push 0  

004DACAE      8D45 DE          lea eax,dword ptr ss:`[ebp-22]`  

004DACB1      50                push eax  

004DACB2      6A FF             push -1  

004DACB4      E8 3FCEF2FF       call `<jmp.&user32.MessageBoxA>`  

004DACB9      6A 01             push 1  

004DACBB      6A 00             push 0  

004DACBD      8D45 FF          lea eax,dword ptr ss:`[ebp-1]`  

004DACC0      50                push eax  

004DACC1      6A FF             push -1  

004DACC3      E8 30CEF2FF       call `<jmp.&user32.MessageBoxA>`  

004DACC8      8D45 D4          lea eax,dword ptr ss:`[ebp-2C]`  

004DACCB      8D55 DE          lea edx,dword ptr ss:`[ebp-22]`  

004DACCE      B9 21000000       mov ecx,21  

004DACD3      E8 4CA0F2FF       call Dump3.00404D24  

004DACD8      8B45 D4          mov eax,dword ptr ss:`[ebp-2C]`  

004DACDB      8D55 D8          lea edx,dword ptr ss:`[ebp-28]`  



```
004DACDE     E8 B9E4F2FF         call Dump3.0040919C

004DACE3     837D D8 00          cmp dword ptr ss:[ebp-28],0

004DACE7     75 27              jnz short Dump3.004DAD10

004DACE9     8D45 CC           lea eax,dword ptr ss:[ebp-34]

004DACEC     8D55 DE           lea edx,dword ptr ss:[ebp-22]

004DACEF     B9 21000000       mov ecx,21

004DACF4     E8 2BA0F2FF       call Dump3.00404D24

004DACF9     8B45 CC           mov eax,dword ptr ss:[ebp-34]

004DACFC     8D55 D0           lea edx,dword ptr ss:[ebp-30]

004DACFF     E8 98E4F2FF      call Dump3.0040919C

004DAD04     837D D0 00       cmp dword ptr ss:[ebp-30],0

004DAD08     75 7C            jnz short Dump3.004DAD86

004DAD0A     807D FF 00       cmp byte ptr ss:[ebp-1],0

004DAD0E     90               nop

004DAD0F     90               nop

004DAD10     B2 01            mov dl,1

004DAD12     8B83 98030000    mov eax,dword ptr ds:[ebx+398]

004DAD18     E8 3BB0F6FF      call Dump3.00445D58

004DAD1D     33D2             xor edx,edx

004DAD1F     8B83 98030000    mov eax,dword ptr ds:[ebx+398]

004DAD25     E8 52D8F9FF      call Dump3.0047857C

004DAD2A     8B83 34030000    mov eax,dword ptr ds:[ebx+334]

004DAD30     8B80 2C020000    mov eax,dword ptr ds:[eax+22C]

004DAD36     E8 E90DFAFF      call Dump3.0047BB24

004DAD3B     85C0             test eax,eax

004DAD3D     75 18            jnz short Dump3.004DAD57

004DAD3F     8B83 38030000    mov eax,dword ptr ds:[ebx+338]

004DAD45     8B80 20020000    mov eax,dword ptr ds:[eax+220]

004DAD4B     BA B4AE4D00      mov edx,Dump3.004DAEB4

004DAD50     8B08             mov ecx,dword ptr ds:[eax]

004DAD52     FF51 38          call dword ptr ds:[ecx+38]

004DAD55     EB 4D            jmp short Dump3.004DADA4

004DAD57     8BC3             mov eax,ebx

004DAD59     E8 46BEFFFF      call Dump3.004D6BA4

004DAD5E     B1 01            mov cl,1

004DAD60     B2 01            mov dl,1

004DAD62     A1 BC594D00      mov eax,dword ptr ds:[4D59BC]

004DAD67     E8 C05FF4FF      call Dump3.00420D2C

004DAD6C     8983 9C030000    mov dword ptr ds:[ebx+39C],eax

004DAD72     C683 AC030000 00 mov byte ptr ds:[ebx+3AC],0

004DAD79     8B83 9C030000    mov eax,dword ptr ds:[ebx+39C]

004DAD7F     E8 E862F4FF      call Dump3.0042106C

004DAD84     EB 1E            jmp short Dump3.004DADA4

004DAD86     8B83 8C030000    mov eax,dword ptr ds:[ebx+38C]

004DAD8C     83C0 68          add eax,68

004DAD8F     BA CCAE4D00      mov edx,Dump3.004DAECC

004DAD94     E8 779DF2FF      call Dump3.00404B10

004DAD99     8B83 8C030000    mov eax,dword ptr ds:[ebx+38C]

004DAD9F     8B10             mov edx,dword ptr ds:[eax]

004DADA1     FF52 30          call dword ptr ds:[edx+30]

004DADA4     8B83 A8030000    mov eax,dword ptr ds:[ebx+3A8]

004DADAA     BA F8AE4D00      mov edx,Dump3.004DAEF8

004DADAF     E8 04A1F2FF      call Dump3.00404EB8

004DADB4     75 0A            jnz short Dump3.004DADC0

004DADB6     E8 69BBFFFF      call Dump3.004D6924

004DADBB     E9 86000000      jmp Dump3.004DAE46

004DADC0     E8 B30AFBFF      call Dump3.0048B878

004DADC5     83C4 F8          add esp,-8

004DADC8     DD1C24           fstp qword ptr ss:[esp]

004DADCB     9B               wait

004DADCC     E8 DF0AFBFF      call Dump3.0048B8B0

004DADD1     8BF0             mov esi,eax

004DADD3     0FB7C6           movzx eax,si

004DADD6     B9 0D000000      mov ecx,0D

004DADDB     33D2             xor edx,edx

004DADDD     F7F1             div ecx

004DADDF     85D2             test edx,edx

004DADE1     75 63            jnz short Dump3.004DAE46

004DADE3     8B83 A0030000    mov eax,dword ptr ds:[ebx+3A0]

004DADE9     8B93 A4030000    mov edx,dword ptr ds:[ebx+3A4]

004DADEF     E8 C4A0F2FF      call Dump3.00404EB8

004DADF4     74 50            je short Dump3.004DAE46

004DADF6     B2 01            mov dl,1

004DADF8     A1 34CB4600      mov eax,dword ptr ds:[46CB34]

004DADFD     E8 321EF9FF      call Dump3.0046CC34

004DAE02     8BD8             mov ebx,eax

004DAE04     BA 01000080      mov edx,80000001

004DAE09     8BC3             mov eax,ebx

004DAE0B     E8 C41EF9FF      call Dump3.0046CCD4

004DAE10     33C9             xor ecx,ecx

004DAE12     BA 04AF4D00      mov edx,Dump3.004DAF04      ; ASCII "\\\Software\\\\FixVideo\\\\VideoFixer\\\\"

004DAE17     8BC3             mov eax,ebx

004DAE19     E8 1A1FF9FF      call Dump3.0046CD38

004DAE1E     84C0             test al,al

004DAE20     74 18            je short Dump3.004DAE3A

004DAE22     B9 F8AE4D00      mov ecx,Dump3.004DAEF8

004DAE27     BA 2CAF4D00      mov edx,Dump3.004DAF2C      ; ASCII "Date"

004DAE2C     8BC3             mov eax,ebx

004DAE2E     E8 A120F9FF      call Dump3.0046CED4

004DAE33     8BC3             mov eax,ebx

004DAE35     E8 6A1EF9FF      call Dump3.0046CCA4

004DAE3A     8BC3             mov eax,ebx

004DAE3C     E8 6B8EF2FF      call Dump3.00403CAC

004DAE41     E8 DEBAFFFF      call Dump3.004D6924

004DAE46     60               pushad  //摧毁自杀代码

004DAE46   /EB 3C            jmp short Dump3.004DAE84

004DAE47     60               pushad

004DAE48     E8 00000000      call Dump3.004DAE4D

004DAE4D     5E               pop esi

004DAE4E     83EE 06          sub esi,6

004DAE51     B9 AD010000      mov ecx,1AD

004DAE56     29CE             sub esi,ecx

004DAE58     BA 3081C79D      mov edx,9DC78130

004DAE5D     C1E9 02          shr ecx,2

004DAE60     83E9 02          sub ecx,2

004DAE63     83F9 00          cmp ecx,0

004DAE66     7C 1A            jl short Dump3.004DAE82

004DAE68     8B048E           mov eax,dword ptr ds:[esi+ecx*4]

004DAE6B     8B5C8E 04        mov ebx,dword ptr ds:[esi+ecx*4+4]

004DAE6F     2BC3             sub eax,ebx

004DAE71     C1C8 17          ror eax,17

004DAE74     33C2             xor eax,edx

004DAE76     81C2 4B354324    add edx,2443354B

004DAE7C     89048E           mov dword ptr ds:[esi+ecx*4],eax

004DAE7F     49               dec ecx

004DAE80   ^ EB E1           jmp short Dump3.004DAE63

004DAE82     61               popad

004DAE83     61               popad

004DAE84     33C0             xor eax,eax //结束。
```

然后还是要保护

```
004DAC97     4D               dec ebp
```

到

```
004DAE84     33C0             xor eax,eax
```

之间的代码不被覆盖。

重启 OD，同样在点修复按钮前下 `Code` 段内存写入断点

```
004DAB41     8945 26          mov dword ptr ss:[ebp+26],eax     ; Dump3.004DAC9A
004DAB44     78 03            js short Dump3.004DAB49
004DAB46     79 01            jns short Dump3.004DAB49
```

往上一点使用花指令插件

```
004DAADE     55               push ebp
004DAADF     68 9FAE4D00      push Dump3.004DAE9F
004DAAE4     64:FF30         push dword ptr fs:[eax]
004DAAE7     64:8920         mov dword ptr fs:[eax],esp
004DAAEA     60               pushad
004DAAEB     FC               cld
004DAAEC     1BCE             sbb ecx,esi
004DAAEE     C1E9 EA         shr ecx,0EA
004DAAF1     72 01            jb short Dump3.004DAAF4
004DAAF3     43               inc ebx
004DAAF4     50               push eax
004DAAF5     E8 01000000      call Dump3.004DAAFB
004DAAFA     EB 58            jmp short Dump3.004DAB54
```



# 汇编指令序列整理

## 入口点 `004DAAFC`

```
004DAAFC     58                      pop eax
```

## 执行序列 `004DAACA` 至 `004DAAD7`

```
004DAACA     53                      push ebx
004DAACB     56                      push esi
004DAACC     33C9                    xor ecx,ecx
004DAACE     894D D8                 mov dword ptr ss:[ebp-28],ecx
004DAAD1     894D D0                 mov dword ptr ss:[ebp-30],ecx
004DAAD4     894D CC                 mov dword ptr ss:[ebp-34],ecx
004DAAD7     894D D4                 mov dword ptr ss:[ebp-2C],ecx
```

## 执行序列 `004DAADA` 至 `004DAAEC`

```
004DAADA     8BD8                    mov ebx,eax
004DAADC     33C0                    xor eax,eax
004DAADE     55                      push ebp
004DAADF     68 9FAE4D00             push Dump3.004DAE9F
004DAAE4     64:FF30                 push dword ptr fs:[eax]
004DAAE7     64:8920                 mov dword ptr fs:[eax],esp
004DAAEA     60                      pushad
004DAAEB     FC                      cld
004DAAEC     1BCE                    sbb ecx,esi
```

## 执行序列 `004DAAEE` 至 `004DAAF3`

```
004DAAEE     C1E9 EA                 shr ecx,0EA
004DAAF1     72 01                   jb short Dump3.004DAAF4
004DAAF3     43                      inc ebx
```

## 执行序列 `004DAAF4` 至 `004DAAFC`

```
004DAAF4     90                      nop
004DAAF5     90                      nop
004DAAF6     90                      nop
004DAAF7     90                      nop
004DAAF8     90                      nop
004DAAF9     90                      nop
004DAAFA     90                      nop
004DAAFB     90                      nop
004DAAFC     90                      nop
```

## 执行序列 `004DAAFD` 至 `004DAB0C`

```
004DAAFD     0F88 02000000          js Dump3.004DAB05
004DAB03     0BDD                    or ebx,ebp
004DAB05     90                      nop
004DAB06     90                      nop
004DAB07     90                      nop
004DAB08     90                      nop
004DAB09     90                      nop
004DAB0A     90                      nop
004DAB0B     90                      nop
004DAB0C     90                      nop
```

## 执行序列 `004DAB0D` 至 `004DAB1F`

```
004DAB0D     90                      nop
004DAB0E     F9                      stc
004DAB0F     90                      nop
004DAB10     90                      nop
004DAB11     90                      nop
004DAB12     90                      nop
004DAB13     90                      nop
004DAB14     F9                      stc
004DAB15     90                      nop
004DAB16     90                      nop
004DAB17     90                      nop
004DAB18     90                      nop
004DAB19     90                      nop
004DAB1A     90                      nop
004DAB1B     90                      nop
004DAB1C     90                      nop
004DAB1D     90                      nop
004DAB1E     90                      nop
004DAB1F     90                      nop
```

## 执行序列 `004DAB20` 至 `004DAB2D`

```
004DAB20     66:BD 2B85              mov bp,852B
004DAB24     90                      nop
004DAB25     90                      nop
004DAB26     90                      nop
004DAB27     90                      nop
004DAB28     90                      nop
004DAB29     90                      nop
004DAB2A     90                      nop
004DAB2B     90                      nop
004DAB2C     90                      nop
004DAB2D     74 02                   je short Dump3.004DAB31
```

## 执行序列 `004DAB2F` 至 `004DAB36`

```
004DAB2F     85E9                    test ecx,ebp
004DAB31     E8 00000000             call Dump3.004DAB36
004DAB36     5D                      pop ebp
```

## 执行序列 `004DAB37` 至 `004DAB44`

```
004DAB37     8BC5                    mov eax,ebp
004DAB39     3B45 26                 cmp eax,dword ptr ss:[ebp+26]
004DAB3C     7C 06                   jl short Dump3.004DAB44
004DAB3E     0345 26                 add eax,dword ptr ss:[ebp+26]
004DAB41     8945 26                 mov dword ptr ss:[ebp+26],eax ; Dump3.004DAC9A
004DAB44     90                      nop
```

## 执行序列 `004DAB45` 至 `004DAB49`

```
004DAB45     90                      nop
004DAB46     90                      nop
004DAB47     90                      nop
004DAB48     90                      nop
004DAB49     85CB                    test ebx,ecx
```

## 执行序列 `004DAB4B` 至 `004DAB53`

```
004DAB4B     0F89 02000000          jns Dump3.004DAB53
004DAB51     85CD                    test ebp,ecx
004DAB53     0F89 02000000          jns Dump3.004DAB5B
```

## 执行序列 `004DAB59` 至 `004DAB5B`

```
004DAB59     D3E5                    shl ebp,cl
004DAB5B     B8 64010000             mov eax,164
```

## 执行序列 `004DAB60` 至 `004DAB6D`

```
004DAB60     90                      nop
004DAB61     90                      nop
004DAB62     90                      nop
004DAB63     90                      nop
004DAB64     90                      nop
004DAB65     90                      nop
004DAB66     90                      nop
004DAB67     90                      nop
004DAB68     90                      nop
004DAB69     90                      nop
004DAB6A     90                      nop
004DAB6B     90                      nop
004DAB6C     90                      nop
004DAB6D     E9 04000000             jmp Dump3.004DAB76
```

## 执行序列 `004DAB72` 至 `004DAB76`

```
004DAB72     66:8BCF                 mov cx,di
004DAB75     F9                      stc
004DAB76     BE F35C587D             mov esi,7D585CF3
```

## 执行序列 `004DAB7B` 至 `004DAB8A`

```
004DAB7B     90                      nop
004DAB7C     90                      nop
004DAB7D     90                      nop
004DAB7E     90                      nop
004DAB7F     90                      nop
004DAB80     90                      nop
004DAB81     90                      nop
004DAB82     90                      nop
004DAB83     90                      nop
004DAB84     90                      nop
004DAB85     90                      nop
004DAB86     90                      nop
004DAB87     90                      nop
004DAB88     90                      nop
004DAB89     90                      nop
004DAB8A     75 02                   jnz short Dump3.004DAB8E
```

## 执行序列 `004DAB8C` 至 `004DAB8E`

```
004DAB8C     87DD                    xchg ebp,ebx
004DAB8E     C1C5 50                 rol ebp,50
```

## 执行序列 `004DAB91` 至 `004DAB9D`

```
004DAB91     68 6A000000             push 6A
004DAB96     0F88 01000000          js Dump3.004DAB9D
004DAB9C     4B                      dec ebx
004DAB9D     5F                      pop edi
```

## 执行序列 `004DAB9E` 至 `004DABBA`

```
004DAB9E     90                      nop
004DAB9F     90                      nop
004DABA0     90                      nop
004DABA1     F9                      stc
004DABA2     87EB                    xchg ebx,ebp
004DABA4     66:BB 2F05              mov bx,52F
004DABA8     8B10                    mov edx,dword ptr ds:[eax] // 修改这里，因为下面的循环到这里。
004DABA8     /E9 EA000000            jmp Dump3.004DAC97 // 直接到保护地点。
004DABAD     |90                     nop
004DABAE     |90                     nop
```

## 执行序列 `004DABAA` 至 `004DABBE`

```
004DABAA     90                      nop
004DABAB     90                      nop
004DABAC     90                      nop
004DABAD     90                      nop
004DABAE     90                      nop
004DABAF     90                      nop
004DABB0     90                      nop
004DABB1     90                      nop
004DABB2     90                      nop
004DABB3     90                      nop
004DABB4     90                      nop
004DABB5     E9 04000000             jmp Dump3.004DABBE
004DABBA     66:D3D1                 rcl cx,cl
004DABBD     F8                      clc
004DABBE     0F85 01000000          jnz Dump3.004DABC5
```

## 执行序列 `004DABC4` 至 `004DABCD`

```
004DABC4     F9                      stc
004DABC5     81E1 6E082B94           and ecx,942B086E
004DABCB     33D6                    xor edx,esi
004DABCD     90                      nop
```

## 执行序列 `004DABCE` 至 `004DABE4`

```
004DABCE     90                      nop
004DABCF     90                      nop
004DABD0     90                      nop
004DABD1     90                      nop
004DABD2     90                      nop
004DABD3     90                      nop
004DABD4     90                      nop
004DABD5     90                      nop
004DABD6     90                      nop
004DABD7     90                      nop
004DABD8     66:D3F9                 sar cx,cl
004DABDB     0F86 03000000          jbe Dump3.004DABE4
004DABE1     66:23E9                 and bp,cx
004DABE4     E9 0A000000             jmp Dump3.004DABF3
```

## 执行序列 `004DABE9` 至 `004DABF3`

```
004DABE9     81F1 0D64113C           xor ecx,3C11640D
004DABEF     66:B9 F8AA              mov cx,0AAF8
004DABF3     C1C2 17                 rol edx,17
```

## 执行序列 `004DABF6` 至 `004DABFA`

```
004DABF6     90                      nop
004DABF7     90                      nop
004DABF8     90                      nop
004DABF9     F9                      stc
004DABFA     41                      inc ecx
```



```
004DABFB     E9 0B000000         jmp Dump3.004DAC0B  

004DAC00     81C1 1C874B1E       add ecx,1E4B871C  

004DAC06     BD CA3E7B37         mov ebp,377B3ECA  

004DAC0B     83E8 FC             sub eax,-4  

004DAC0E     0310                add edx,dword ptr ds:[eax]  

004DAC10     83E8 04             sub eax,4  

004DAC13     90                  nop  

004DAC14     90                  nop  

004DAC15     90                  nop  

004DAC16     90                  nop  

004DAC17     90                  nop  

004DAC18     90                  nop  

004DAC19     90                  nop  

004DAC1A     90                  nop  

004DAC1B     90                  nop  

004DAC1C     90                  nop  

004DAC1D     85EB                test ebx,ebp  

004DAC1F     8910                mov dword ptr ds:[eax],edx  

004DAC21     90                  nop  

004DAC22     90                  nop  

004DAC23     90                  nop  

004DAC24     90                  nop  

004DAC25     90                  nop  

004DAC26     BB 2A616C18         mov ebx,186C612A  

004DAC2B     7A 02               jpe short Dump3.004DAC2F  

004DAC2D     85E9                test ecx,ebp  

004DAC2F     E9 0D000000         jmp Dump3.004DAC41  

004DAC34     81D1 CC6E5327       adc ecx,27536ECC  

004DAC3A     0F81 01000000       jno Dump3.004DAC41  

004DAC40     49                  dec ecx  

004DAC41     81EE 4B354324       sub esi,2443354B  

004DAC47     90                  nop  

004DAC48     90                  nop  

004DAC49     90                  nop  

004DAC4A     90                  nop  

004DAC4B     90                  nop  

004DAC4C     90                  nop  

004DAC4D     90                  nop  

004DAC4E     90                  nop  

004DAC4F     90                  nop  

004DAC50     90                  nop  

004DAC51     90                  nop  

004DAC52     81C9 D961400D       or ecx,0D4061D9  

004DAC58     D3E9                shr ecx,cl  

004DAC5A     81D9 A7C469DF       sbb ecx,DF69C4A7  

004DAC60     81C0 04000000       add eax,4  

004DAC66     90                  nop  

004DAC67     90                  nop  

004DAC68     90                  nop  

004DAC69     90                  nop  

004DAC6A     90                  nop  

004DAC6B     90                  nop  

004DAC6C     90                  nop  

004DAC6D     90                  nop  

004DAC6E     90                  nop  

004DAC6F     90                  nop  

004DAC70     90                  nop  

004DAC71     E9 08000000         jmp Dump3.004DAC7E  

004DAC76     8BE8                mov ebp,eax  

004DAC78     81D5 1650FE3E       adc ebp,3EFE5016  

004DAC7E     B9 2A940BBB         mov ecx,BB0B942A  

004DAC83     0BDA                or ebx,edx  

004DAC85     83EF 01             sub edi,1  

004DAC88    ^ 0F85 1AFFFFFF       jnz Dump3.004DABA8 //循环

.............................................................................
```

保存所有修改，主功能完成，剩下就是 `rsakey` 保护了，不知能否搞定。

## 【破解总结】
`Acprotect` 的壳采用了 `Stolen Code`，动态解码，还有 `SDK` 内外结合，需要耐心攻下它的。破解不难，只要能保持有用代码，摧毁自杀代码，胜利就属于你。

## 【版权声明】
本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
