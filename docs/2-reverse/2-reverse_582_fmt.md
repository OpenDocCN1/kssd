
# 小软件逆向分析+程序设计

> 原文：[`www.pediy.com/kssd/pediy10/76786.html`](https://www.pediy.com/kssd/pediy10/76786.html)

## 文章信息

- 【文章标题】：`小软件爆破+分析+程序设计[精简版]`
- 【文章作者】：`eASYRAs`
- 【作者主页】：`http://blog.sina.com.cn/77muyulong`
- 【软件名称】：`AdslTime.exe`
- 【下载地址】：自己搜索下载
- 【加壳方式】：无
- 【保护方式】：机器号验证码、时间限制
- 【编写语言】：`Microsoft Visual C++ 6.0`
- 【软件介绍】：国产软件，可以管理 `ADSL` 上网时间。如果喜欢请购买~
- 【作者声明】：学习中的一点点小收获，望高手指点~ 仅作学习之用，勿于商业用途。

## 详细过程

前些日子家里的 `ADSL` 又超时了，于是觉得应该写个小东西给家里用，不能总是玩，适时要干些正事~

不过在这个过程中真是受尽了艰辛，深感软件工程之不易，遂撰写此文以记之。

文中涉及到一些极其幼稚的思路，望高手一笑而过~

起初就是打算写一个 `ADSL` 工具，自以为功夫不错，于是拿出工具动手，发现原来并不简单。刚开始的思路是读取任务栏 `ADSL` 联网对话框里的数据，之后把它显示出来并用文件记录就好了。甚至还想过 `hook` 联网与断网的 `code`，达到最省资源、最优的效果。然而，在实际操作中，种种方法被排斥在门外，无奈何自己过于菜虫…

### 方法 1：`HookCode`

排除原因：建立 `ADSL` 连接是一个在 `explorer` 的线程，过程本身是没有进程的。如果想要 `HOOK`，必须要调试 `explorer`。网上倒是有些资料，不过加载了一下后我就彻底放弃了，这不是常人能玩的~

### 方法 2：远程读取

既然 `HOOK` 不了，那做一个外挂总是可以的。所以我打算像制作修改器一样远程读取其它进程（`explorer`）的数据，之后显示在自己的程序里并操作。于是搜索了数据，发现是变址储存。然而 `explorer` 又有其特殊性（参见方案 1），所以还是无法通过调试的手段来得到变址地址，遂不成矣。

### 方法 3：`Timmer` 计时法

还想到了一个超级笨的方法，就是用一个 `Timmer` 来计时。这样只要有一个开始的信号就好了，`Timmer` 会不停的向文件记录信息。就算是非正常断网关机，都不会对其产生影响。虽然有些觉得浪费资源，不过此乃下下之策，总比没有强…=`\_ ,_=

于是花了些时间“拼凑”出一个超简陋的程序，然而到测试时才发现，忽略了最重要的因素就是那个开始的信号。当然的以为 `shell` 那个快捷方式就可以打开连接的，结果过确实不随人愿… 至此，方法 3 正式破产…

之后我苦苦追求，在网上 `google` 呀 `baidu` 呀，好久好久，一个软件出现在了我眼前：`AdslTime`，一个商业 `ADSL` 计时软件。我把它装上后试验了一下，比自己的简陋程序不知好了几千倍。不过却又时间限制，于是爆破之……

### 爆破方法

（还要说明一下，这个方法并不是最简单的，不过达到目的就好）

代码:

```
0040795C  . 90 nop
0040795D  . 90 nop
 
;:::::::::::::::::::::::::::::::
 
0040A350  . 90 nop
0040A351  . 90 nop
```

好了，之后只要启动后随便输入 `email` 和注册码就可以注册成功了。没有了限制就是舒服啊…呵呵

还不能这么安逸，凭什么咱写不出来的东西他就能做的这么好，一定要学到点东西才成……

### 逆向分析

拿出工具，开工……

`od` 载入，友好的 `VC++OEP`

代码:

```
00414AD4  AdslTim>/$ 55 push ebp ; (initial cpu selection)
00414AD5  |. 8BEC mov ebp, esp
00414AD7  |. 6A FF push -1
00414AD9  |. 68 B0014500 push 004501B0
00414ADE  |. 68 24AE4100 push 0041AE24 ; SE 处理程序安装
00414AE3  |. 64:A1 0000000>mov eax, dword ptr fs:\[0\]
00414AE9  |. 50 push eax
00414AEA  |. 64:8925 00000>mov dword ptr fs:\[0\], esp
00414AF1  |. 83EC 58 sub esp, 58
00414AF4  |. 53 push ebx
00414AF5  |. 56 push esi
00414AF6  |. 57 push edi
00414AF7  |. 8965 E8 mov dword ptr \[ebp-18\], esp
00414AFA  |. FF15 80934400 call dword ptr \[<&KERNEL32.GetVersion>; kernel32.GetVersion
 
```

先想想看软件的功能，毕竟咱自己也已经写过一个类似功能的“不成功”软件了，所以思路比较清晰。不过是什么原理，他肯定得怕一条不正常断网，例如关机等等。所以肯定会有一个周期性的记录操作。看看软件根目录，运行后出现 `adsl.bak`、`adsl.bs` 这两个文件，猜测肯定有一个是记录时间的（因为我就是用文件记录的）。用 `16` 进制编辑器看看：

`adsl.bak`

代码:

```
Offset 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
 
00000000 17 00 00 00 ....
 
```

木哈哈，太眼熟了，居然跟我一样，用一个 `DWORD` 来记录共用的秒数。这太清晰了，这里就是入手点…

时间过了一下下，编辑器提示我文件有更新是否刷新。这么快就更新了，看来是一秒钟一次了。`SetTimer` 就是突破口。

`OD` `bp` `SetTimer` 运行，第 2 次断下

代码:

```
77D18C2E  USER32.SetTimer /$ B8 1E120000 mov eax, 121E
77D18C33  |. BA 0003FE7F mov edx, 7FFE0300
77D18C38  |. FF12 call dword ptr \[edx\]
77D18C3A  \\. C2 1000 retn 10
```

`Alt+F9` 返回

代码:

```
00408F00  . 68 AC944000 push ; /杠 fd
00408F05  . 68 E8030000 push 3E8 ; |Timeout = 1000. ms
00408F0A  . 6A 02 push 2 ; |TimerID = 2
00408F0C  . FF76 1C push dword ptr \[esi+1C\] ; |hWnd
00408F0F  . FF15 60944400 call dword ptr \[<&USER32.SetTimer>\] ; \\SetTimer
```

参数第一个是句柄，第二个计时器 ID，第三个周期 ms，第四个 Tim 回调函数入口。在我这里加了标签，是 `0x4094AC`。

`Ctrl+G` `4094AC` 过去看看

代码:



```
004094AC /$ B8 DC664400 mov eax, 004466DC
004094B1 |. E8 82A40000 call 00413938
004094B6 |. 83EC 30 sub esp, 30
004094B9 |. A1 30C74500 mov eax, dword ptr [45C730]
004094BE |. 56 push esi
004094BF |. 57 push edi
004094C0 |. 8945 F0 mov dword ptr [ebp-10], eax
004094C3 |. 8365 FC 00 and dword ptr [ebp-4], 0
004094C7 |. 8945 E8 mov dword ptr [ebp-18], eax
004094CA |. 8945 EC mov dword ptr [ebp-14], eax
004094CD |. FF75 08 push dword ptr [ebp+8]
004094D0 |. C645 FC 02 mov byte ptr [ebp-4], 2
004094D4 |. E8 76EF0100 call 0042844F
004094D9 |. 8BF0 mov esi, eax
004094DB |. 8BCE mov ecx, esi
004094DD |. 8B06 mov eax, dword ptr [esi]
004094DF |. FF90 BC000000 call dword ptr [eax+BC]
004094E5 |. 6A 00 push 0 ; /Arg3 = 00000000
004094E7 |. 68 01E80000 push 0E801 ; |Arg2 = 0000E801
004094EC |. FF76 1C push dword ptr [esi+1C] ; |Arg1
004094EF |. E8 AF060200 call 00429BA3 ; \AdslTime.00429BA3
004094F4 |. FF05 C4F24500 inc dword ptr [45F2C4]
004094FA |. 8BF8 mov edi, eax
004094FC |. 85FF test edi, edi
004094FE |. 0F84 E6010000 je 004096EA
00409504 |. 8D45 EC lea eax, dword ptr [ebp-14]
00409507 |. 53 push ebx
00409508 |. 50 push eax
00409509 |. 8D45 E8 lea eax, dword ptr [ebp-18]
0040950C |. 50 push eax
0040950D |. 8D45 F0 lea eax, dword ptr [ebp-10]
00409510 |. 50 push eax
00409511 |. 8BCE mov ecx, esi
00409513 |. E8 E8170000 call <写文件+显示更新>
00409518 |. 57 push edi
00409519 |. 8D4D C4 lea ecx, dword ptr [ebp-3C]
0040951C |. E8 974B0200 call 0042E0B8
00409521 |. 8BCF mov ecx, edi
00409523 |. C645 FC 03 mov byte ptr [ebp-4], 3
00409527 |. E8 DFE0FFFF call 0040760B
0040952C |. 50 push eax
0040952D |. 8D4D C4 lea ecx, dword ptr [ebp-3C]
00409530 |. E8 C8440200 call 0042D9FD
00409535 |. 68 86000000 push 86
0040953A |. 8BCF mov ecx, edi
0040953C |. 8BD8 mov ebx, eax
0040953E |. E8 698C0200 call 004321AC
00409543 |. 8945 E0 mov dword ptr [ebp-20], eax
00409546 |. 8D45 EC lea eax, dword ptr [ebp-14]
00409549 |. 50 push eax
0040954A |. 8D45 F0 lea eax, dword ptr [ebp-10]
0040954D |. 50 push eax
0040954E |. 8D45 E4 lea eax, dword ptr [ebp-1C]
00409551 |. 50 push eax
00409552 |. E8 BAE20100 call 00427811
00409557 |. 8B00 mov eax, dword ptr [eax]
00409559 |. C645 FC 04 mov byte ptr [ebp-4], 4
0040955D |. 8B78 F8 mov edi, dword ptr [eax-8]
00409560 |. 8D45 EC lea eax, dword ptr [ebp-14]
00409563 |. 50 push eax
00409564 |. 8D45 F0 lea eax, dword ptr [ebp-10]
00409567 |. 50 push eax
00409568 |. 8D45 08 lea eax, dword ptr [ebp+8]
0040956B |. 50 push eax
0040956C |. E8 A0E20100 call 00427811
00409571 |. 8B00 mov eax, dword ptr [eax]
00409573 |. 8D4D D8 lea ecx, dword ptr [ebp-28]
00409576 |. 51 push ecx ; /pSize
00409577 |. 57 push edi ; |TextLen
00409578 |. 50 push eax ; |Text
00409579 |. FF75 CC push dword ptr [ebp-34] ; |hDC
0040957C |. FF15 78914400 call dword ptr [<&GDI32.GetTextExtent>; \GetTextExtentPoint32A
00409582 |. 8D4D 08 lea ecx, dword ptr [ebp+8]
00409585 |. E8 A4E00100 call 0042762E
0040958A |. 8D4D E4 lea ecx, dword ptr [ebp-1C]
0040958D |. C645 FC 03 mov byte ptr [ebp-4], 3
00409591 |. E8 98E00100 call 0042762E
00409596 |. 8D45 EC lea eax, dword ptr [ebp-14]
00409599 |. 50 push eax
0040959A |. 8D45 F0 lea eax, dword ptr [ebp-10]
0040959D |. 50 push eax
0040959E |. 8D45 08 lea eax, dword ptr [ebp+8]
004095A1 |. 50 push eax
004095A2 |. E8 6AE20100 call 00427811
004095A7 |. 50 push eax ; /Arg3
004095A8 |. 8BCE mov ecx, esi ; |
004095AA |. FF75 D8 push dword ptr [ebp-28] ; |Arg2
004095AD |. C645 FC 05 mov byte ptr [ebp-4], 5 ; |
004095B1 |. FF75 E0 push dword ptr [ebp-20] ; |Arg1
004095B4 |. E8 46130000 call 0040A8FF ; \AdslTime.0040A8FF
004095B9 |. 8D4D 08 lea ecx, dword ptr [ebp+8]
004095BC |. C645 FC 03 mov byte ptr [ebp-4], 3
004095C0 |. E8 69E00100 call 0042762E
004095C5 |. 53 push ebx
004095C6 |. 8D4D C4 lea ecx, dword ptr [ebp-3C]
004095C9 |. E8 2F440200 call 0042D9FD
004095CE |. 833D 20F34500 cmp dword ptr [45F320], 0
004095D5 |. 5B pop ebx
004095D6 |. 0F84 FB000000 je 004096D7
004095DC |. 8D45 F0 lea eax, dword ptr [ebp-10]
004095DF |. 8D8E 30090000 lea ecx, dword ptr [esi+930]
004095E5 |. 50 push eax
004095E6 |. E8 30E10100 call 0042771B
004095EB |. 8D45 F0 lea eax, dword ptr [ebp-10]
004095EE |. 68 0CBE4500 push 0045BE0C ; \n
004095F3 |. 50 push eax
004095F4 |. 8D45 E0 lea eax, dword ptr [ebp-20]
004095F7 |. 50 push eax
004095F8 |. E8 7AE20100 call 00427877
004095FD |. 8D4D EC lea ecx, dword ptr [ebp-14]
00409600 |. C645 FC 06 mov byte ptr [ebp-4], 6
00409604 |. 51 push ecx
00409605 |. 50 push eax
00409606 |. 8D45 08 lea eax, dword ptr [ebp+8]
00409609 |. 50 push eax
0040960A |. E8 02E20100 call 00427811
0040960F |. 50 push eax
00409610 |. 8D4D F0 lea ecx, dword ptr [ebp-10]
00409613 |. C645 FC 07 mov byte ptr [ebp-4], 7
00409617 |. E8 FFE00100 call 0042771B
0040961C |. 8D4D 08 lea ecx, dword ptr [ebp+8]
0040961F |. C645 FC 06 mov byte ptr [ebp-4], 6
00409623 |. E8 06E00100 call 0042762E
00409628 |. 8D4D E0 lea ecx, dword ptr [ebp-20]
0040962B |. C645 FC 03 mov byte ptr [ebp-4], 3
0040962F |. E8 FADF0100 call 0042762E
00409634 |. 8B86 94080000 mov eax, dword ptr [esi+894]
0040963A |. 8378 6C 01 cmp dword ptr [eax+6C], 1
0040963E |. 75 2B jnz short 0040966B
00409640 |. 8D45 F0 lea eax, dword ptr [ebp-10]
00409643 |. 50 push eax
00409644 |. 8D45 08 lea eax, dword ptr [ebp+8]
00409647 |. 68 04BE4500 push 0045BE04 ; 超时\n
0040964C |. 50 push eax
0040964D |. E8 99E20100 call 004278EB
00409652 |. 50 push eax
00409653 |. 8D4D F0 lea ecx, dword ptr [ebp-10]
00409656 |. C645 FC 08 mov byte ptr [ebp-4], 8
0040965A |. E8 BCE00100 call 0042771B
0040965F |. 8D4D 08 lea ecx, dword ptr [ebp+8]
00409662 |. C645 FC 03 mov byte ptr [ebp-4], 3
00409666 |. E8 C3DF0100 call 0042762E
0040966B |> 8D45 F0 lea eax, dword ptr [ebp-10]
0040966E |. 68 E0BD4500 push 0045BDE0 ; \n\n
00409673 |. 50 push eax
00409674 |. 8D45 E4 lea eax, dword ptr [ebp-1C]
00409677 |. 50 push eax
00409678 |. E8 FAE10100 call 00427877
0040967D |. 68 F8BD4500 push 0045BDF8 ; 本次:
00409682 |. 50 push eax
00409683 |. 8D45 E0 lea eax, dword ptr [ebp-20]
00409686 |. C645 FC 09 mov byte ptr [ebp-4], 9
0040968A |. 50 push eax
0040968B |. E8 E7E10100 call 00427877
00409690 |. 8D4D E8 lea ecx, dword ptr [ebp-18]
00409693 |. C645 FC 0A mov byte ptr [ebp-4], 0A
00409697 |. 51 push ecx
00409698 |. 50 push eax
00409699 |. 8D45 08 lea eax, dword ptr [ebp+8]
0040969C |. 50 push eax
0040969D |. E8 6FE10100 call 00427811
004096A2 |. FF30 push dword ptr [eax]
004096A4 |. 8D8E 98080000 lea ecx, dword ptr [esi+898]
004096AA |. C645 FC 0B mov byte ptr [ebp-4], 0B
004096AE |. E8 85800000 call 00411738
004096B3 |. 8D4D 08 lea ecx, dword ptr [ebp+8]
004096B6 |. C645 FC 0A mov byte ptr [ebp-4], 0A
004096BA |. E8 6FDF0100 call 0042762E
004096BF |. 8D4D E0 lea ecx, dword ptr [ebp-20]
004096C2 |. C645 FC 09 mov byte ptr [ebp-4], 9
004096C6 |. E8 63DF0100 call 0042762E
004096CB |. 8D4D E4 lea ecx, dword ptr [ebp-1C]
004096CE |. C645 FC 03 mov byte ptr [ebp-4], 3
004096D2 |. E8 57DF0100 call 0042762E
004096D7 |> 8BCE mov ecx, esi
004096D9 |. E8 32140000 call <得到网络状态>
004096DE |. 8D4D C4 lea ecx, dword ptr [ebp-3C]
004096E1 |. C645 FC 02 mov byte ptr [ebp-4], 2
004096E5 |. E8 404A0200 call 0042E12A
004096EA |> 8D4D EC lea ecx, dword ptr [ebp-14]
004096ED |. C645 FC 01 mov byte ptr [ebp-4], 1
004096F1 |. E8 38DF0100 call 0042762E
004096F6 |. 8065 FC 00 and byte ptr [ebp-4], 0
004096FA |. 8D4D E8 lea ecx, dword ptr [ebp-18]
004096FD |. E8 2CDF0100 call 0042762E
00409702 |. 834D FC FF or dword ptr [ebp-4], FFFFFFFF
00409706 |. 8D4D F0 lea ecx, dword ptr [ebp-10]
00409709 |. E8 20DF0100 call 0042762E
0040970E |. 8B4D F4 mov ecx, dword ptr [ebp-C]
00409711 |. 5F pop edi
00409712 |. 5E pop esi
00409713 |. 64:890D 00000 mov dword ptr fs:[0], ecx
0040971A |. C9 leave
0040971B \\. C2 1000 ret 10
```



### 代码分析

超长的回调函数看来做了不少事情，估计核心代码也就在这里了。比如它到底是如何实现的得到联网时间？我想这个作者虽然很强大，但是应该也不会是特别麻烦的方法吧。还是看看他导入了哪些函数吧。

代码：

```
\->Import Table
1.略
2.略

3. ImageImportDescriptor:
OriginalFirstThunk: 0x00057DA0
TimeDateStamp: 0x00000000 (GMT: Thu Jan 01 00:00:00 1970)
ForwarderChain: 0x00000000
Name: 0x00058634 ("RASAPI32.dll")
FirstThunk: 0x000493E8

Ordinal/Hint API name
\------------ ---------------------------------------
0x000F "RasEditPhonebookEntryA"
0x002E "RasGetErrorStringA"
0x0022 "RasGetConnectStatusA"
0x0039 "RasHangUpA"
0x002A "RasGetEntryDialParamsA"
0x000C "RasDialA"
0x0048 "RasSetEntryDialParamsA"
0x0013 "RasEnumConnectionsA"
0x0018 "RasEnumEntriesA"
0x002C "RasGetEntryPropertiesA"
0x0033 "RasGetProjectionInfoA"
4.略
5.略
6.略
...
..
.
```

哇，好多不认识的啊。不过没关系，还好我有 `google`。经过 G 大师的指点，很快 `RASAPI32.dll` 的 API 引起了我的注意。

#### 3. ImageImportDescriptor

`OriginalFirstThunk`: `0x00057DA0`

`TimeDateStamp`: `0x00000000` (GMT: Thu Jan 01 00:00:00 1970)

`ForwarderChain`: `0x00000000`

`Name`: `0x00058634` ("`RASAPI32.dll`")

`FirstThunk`: `0x000493E8`

`Ordinal/Hint` API name

\------------ ---------------------------------------

`0x000F` "`RasEditPhonebookEntryA`"

`0x002E` "`RasGetErrorStringA`"

`0x0022` "`RasGetConnectStatusA`"

`0x0039` "`RasHangUpA`"

`0x002A` "`RasGetEntryDialParamsA`"

`0x000C` "`RasDialA`"

`0x0048` "`RasSetEntryDialParamsA`"

`0x0013` "`RasEnumConnectionsA`"

`0x0018` "`RasEnumEntriesA`"

`0x002C` "`RasGetEntryPropertiesA`"

`0x0033` "`RasGetProjectionInfoA`"

像

`0x000C` "`RasDialA`" 是建立拨号连接用的

`0x0048` "`RasSetEntryDialParamsA`" 是设置联网属性用的

`0x0013` "`RasEnumConnectionsA`" 是得到当前活动链接用的

等等等等。

搜索了一下，通常来讲获得联网时间都会有两个函数连用，那就是 `RasEnumConnectionsA` 和 `RasGetConnectionStatistics`。

前者获得当前活动的链接句柄，后者通过这个句柄得到当前连接的属性。不过好像导入库里没有第二个 API，总之跟 `RasEnumConnectionsA` 再说。`BP` `RasEnumConnectionsA`

断下后返回来到 `0x0040E2EE`。

代码：

```
0040E293  /$  55      push    ebp
0040E294  |.  8BEC    mov     ebp, esp
0040E296  |.  83EC 10   sub     esp, 10
0040E299  |.  53      push    ebx
0040E29A  |.  56      push    esi
0040E29B  |.  57      push    edi
0040E29C  |.  68 F8C34500 push    0045C3F8  ; /rasgetconnectionstatistics
0040E2A1  |.  FF71 64   push    dword ptr [ecx+64]  ; |hModule
0040E2A4  |.  FF15 90934400 call    dword ptr [<&KERNEL32.GetProcAdd>; \GetProcAddress
0040E2AA  |.  85C0    test    eax, eax
0040E2AC  |.  8945 F4   mov     dword ptr [ebp-C], eax
0040E2AF  |.  75 13     jnz     short 0040E2C4
0040E2B1  |.  50      push    eax   ; /Arg3
0040E2B2  |.  50      push    eax   ; |Arg2
0040E2B3  |.  68 C8C34500 push    0045C3C8  ; |getprocaddress() rasgetconnectionstatistics
0040E2B8  |.  E8 8B370200 call    00431A48  ; \AdslTime.00431A48
0040E2BD  |.  6A 01     push    1
0040E2BF  |.  E9 13010000 jmp     0040E3D7
0040E2C4  |>  8B3D 5C934400 mov     edi, dword ptr [<&KERNEL32.GetPr>; kernel32.GetProcessHeap
0040E2CA  |.  BE 9C010000 mov     esi, 19C
0040E2CF  |.  56      push    esi   ; /HeapSize => 19C (412.)
0040E2D0  |.  6A 08     push    8     ; |Flags = HEAP_ZERO_MEMORY
0040E2D2  |.  8975 F8   mov     dword ptr [ebp-8], esi  ; |
0040E2D5  |.  FFD7    call    edi   ; |[GetProcessHeap
0040E2D7  |.  50      push    eax   ; |hHeap
0040E2D8  |.  FF15 54934400 call    dword ptr [<&KERNEL32.HeapAlloc>>; \HeapAlloc
0040E2DE  |.  8BD8    mov     ebx, eax
0040E2E0  |.  8D45 F0   lea     eax, dword ptr [ebp-10]
0040E2E3  |.  50      push    eax
0040E2E4  |.  8D45 F8   lea     eax, dword ptr [ebp-8]
0040E2E7  |.  50      push    eax
0040E2E8  |.  53      push    ebx
0040E2E9  |.  895D FC   mov     dword ptr [ebp-4], ebx
0040E2EC  |.  8933    mov     dword ptr [ebx], esi
0040E2EE  |.  E8 89420000 call    ;断下返回在这里
0040E2F3  |.  85C0    test    eax, eax
0040E2F5  |.  74 17     je      short 0040E30E
0040E2F7  |.  85DB    test    ebx, ebx
0040E2F9  |.  74 0C     je      short 0040E307
0040E2FB  |.  53      push    ebx
0040E2FC  |.  6A 00     push    0
0040E2FE  |.  FFD7    call    edi
0040E300  |.  50      push    eax   ; |hHeap
0040E301  |.  FF15 58934400 call    dword ptr [<&KERNEL32.HeapFree>]; \HeapFree
0040E307  |>  6A 04     push    4
0040E309  |.  E9 C9000000 jmp     0040E3D7
0040E30E  |>  6A 3C     push    3C
0040E310  |.  6A 08     push    8
0040E312  |.  FFD7    call    edi
0040E314  |.  50      push    eax   ; |hHeap
0040E315  |.  FF15 54934400 call    dword ptr [<&KERNEL32.HeapAlloc>>; \HeapAlloc
0040E31B  |.  8BF0    mov     esi, eax
0040E31D  |.  C706 3C000000 mov     dword ptr [esi], 3C
0040E323  |.  8B5B 04   mov     ebx, dword ptr [ebx+4]
0040E326  |.  85DB    test    ebx, ebx
0040E328  |.  0F84 A7000000 je      0040E3D5
0040E32E  |.  56      push    esi
0040E32F  |.  53      push    ebx
0040E330  |.  FF55 F4   call    dword ptr [ebp-C]
0040E333  |.  85C0    test    eax, eax
0040E335  |.  0F85 96000000 jnz     0040E3D1
0040E33B  |.  8B4D 08   mov     ecx, dword ptr [ebp+8]
0040E33E  |.  8B46 08   mov     eax, dword ptr [esi+8]
0040E341  |.  33D2    xor     edx, edx
0040E343  |.  BB 80EE3600 mov     ebx, 36EE80
0040E348  |.  8901    mov     dword ptr [ecx], eax
0040E34A  |.  8B4D 0C   mov     ecx, dword ptr [ebp+C]
0040E34D  |.  8B46 04   mov     eax, dword ptr [esi+4]
0040E350  |.  6A 18     push    18
0040E352  |.  8901    mov     dword ptr [ecx], eax
0040E354  |.  8B4D 10   mov     ecx, dword ptr [ebp+10]
0040E357  |.  8B46 34   mov     eax, dword ptr [esi+34]
0040E35A  |.  8901    mov     dword ptr [ecx], eax
0040E35C  |.  A1 30C74500 mov     eax, dword ptr [45C730]
0040E361  |.  8B4E 38   mov     ecx, dword ptr [esi+38]
0040E364  |.  8945 08   mov     dword ptr [ebp+8], eax
0040E367  |.  8BC1    mov     eax, ecx
0040E369  |.  F7F3    div     ebx
0040E36B  |.  5B      pop     ebx
0040E36C  |.  6A 3C     push    3C
0040E36E  |.  99      cdq
0040E36F  |.  F7FB    idiv    ebx
0040E371  |.  8BC1    mov     eax, ecx
0040E373  |.  B9 60EA0000 mov     ecx, 0EA60
0040E378  |.  8BDA    mov     ebx, edx
0040E37A  |.  33D2    xor     edx, edx
0040E37C  |.  F7F1    div     ecx
0040E37E  |.  6BDB 3C   imul    ebx, ebx, 3C
0040E381  |.  59      pop     ecx
0040E382  |.  6A 3C     push    3C
0040E384  |.  99      cdq
0040E385  |.  F7F9    idiv    ecx
0040E387  |.  8B46 38   mov     eax, dword ptr [esi+38]
0040E38A  |.  B9 E8030000 mov     ecx, 3E8
0040E38F  |.  03DA    add     ebx, edx
0040E391  |.  33D2    xor     edx, edx
0040E393  |.  F7F1    div     ecx
0040E395  |.  6BDB 3C   imul    ebx, ebx, 3C
0040E398  |.  59      pop     ecx
0040E399  |.  99      cdq
0040E39A  |.  F7F9    idiv    ecx
0040E39C  |.  8B45 14   mov     eax, dword ptr [ebp+14]
0040E39F  |.  8D4D 08   lea     ecx, dword ptr [ebp+8]
0040E3A2  |.  03DA    add     ebx, edx
0040E3A4  |.  8918    mov     dword ptr [eax], ebx
0040E3A6  |.  E8 83920100 call    0042762E
0040E3AB  |.  837D FC 00 cmp     dword ptr [ebp-4], 0
0040E3AF  |.  8B1D 58934400 mov     ebx, dword ptr [<&KERNEL32.HeapF>; ntdll.RtlFreeHeap
0040E3B5  |.  74 0A     je      short 0040E3C1
0040E3B7  |.  FF75 FC   push    dword ptr [ebp-4]
0040E3BA  |.  6A 00     push    0
0040E3BC  |.  FFD7    call    edi
0040E3BE  |.  50      push    eax
0040E3BF  |.  FFD3    call    ebx
0040E3C1  |>  85F6    test    esi, esi
0040E3C3  |.  74 08     je      short 0040E3CD
0040E3C5  |.  56      push    esi
0040E3C6  |.  6A 00     push    0
0040E3C8  |.  FFD7    call    edi
0040E3CA  |.  50      push    eax
0040E3CB  |.  FFD3    call    ebx
0040E3CD  |>  33C0    xor     eax, eax
0040E3CF  |.  EB 07     jmp     short 0040E3D8
0040E3D1  |>  6A 02     push    2
0040E3D3  |.  EB 02     jmp     short 0040E3D7
0040E3D5  |>  6A 03     push    3
0040E3D7  |>  58      pop     eax
0040E3D8  |>  5F      pop     edi
0040E3D9  |.  5E      pop     esi
0040E3DA  |.  5B      pop     ebx
0040E3DB  |.  C9      leave
0040E3DC  \.  C2 1000   ret     10
```

向上看看，哇，有一个关键的东东。

代码：

```
0040E29C  |.  68 F8C34500 push    0045C3F8  ; /rasgetconnectionstatistics
0040E2A1  |.  FF71 64   push    dword ptr [ecx+64]  ; |hModule
0040E2A4  |.  FF15 90934400 call    dword ptr [<&KERNEL32.GetProcAdd>; \GetProcAddress
```

这不是显示加载的一种方法么？加载的还正是黄金搭档 `rasgetconnectionstatistics`。

得到函数地址后把地址存入了 `ebp-c` 这个局部变量，看来下面是会用到的。

代码：

```
0040E32E  |.  56      push    esi
0040E32F  |.  53      push    ebx
0040E330  |.  FF55 F4   call    dword ptr [ebp-C]
```

刚往下走了几行，就已经 `call` 到了。很明显，这就得到了联网信息，哈哈，可谓大功告成!!

返回后发现是一个整合信息的函数，把得到的数据格式化分开。看看调用，只有一个 `call` 到了这个函数，那就是

`004468D0`= `004468D0`

本地调用来自 `timer` 回调函数+`22D`

`AdslTime.得到网络状态`

这下就真相大白了，程序是通过 `SetTimer` 来使函数不断的获得信息、写入文件、更新显示的。有之前的跟踪可知，周期是 1 秒。

很好，这下都清晰了，就像是自己的程序一样了~了然于胸……

知道了它的工作原理不觉手痒，总不能让自己写的程序真就费了呀。于是又对程序改造一翻，期间几遇波折困难，然最终皆克之。

拼凑出了我的计时程序 `NetlinkTime`。基本思路跟目标程序一致，不过我把函数的加载改为了隐式，想也许效率会高一丁丁~

先贴代码，代码写的很丑，高手见笑，只是为了模拟这个功能罢了。

代码：



```assembly
.386
.model flat,stdcall
option casemap:none
include windows.inc
include user32.inc
include kernel32.inc
include rasapi32.inc
include shell32.inc
includelib user32.lib
includelib kernel32.lib
includelib shell32.lib
includelib rasapi32.lib

IdPlay        equ  2000
WM_SHELLNOTIFY equ  WM_USER+5
IDI_TRAY      equ  0
IDM_RESTORE   equ  1000
IDM_EXIT      equ  1010

WinMain PROTO :DWORD,:DWORD,:DWORD,:DWORD
TimProc PROTO :HWND,:UINT,:UINT,:DWORD

.data
RAS_STATS   struc
dwSize             dd ?
dwBytesXmited      dd ?
dwBytesRcved       dd ?
dwFramesXmited     dd ?
dwFramesRcved      dd ?
dwCrcErr           dd ?
dwTimeoutErr       dd ?
dwAlignmentErr     dd ?
dwHardwareOverrunErr dd ?
dwFramingErr       dd ?
dwBufferOverrunErr dd ?
dwCompressionRatioIn dd ?
dwCompressionRatioOut dd ?
dwBps              dd ?
dwConnectDuration  dd ?    ;时间/毫秒
RAS_STATS   ends

rastats RAS_STATS <>
long           dd 19ch
linkN          dd 0
rasconn        RASCONN <>
hStatic        dd 0
miao           dd 0
fen            dd 0
shi            dd 0
pfile          dd 0
Ttime          dd 0
Taddtime       dd 0
Ttimexian      dd 0
Fname          db "c:\Time.DAT",0
Iszero         db 0,0,0,0,1,0
tamp           db "本月累计上网约%d 小时%d 分钟%d 秒",0
XX             dd  0
play           db 0 dup (32)
ClassName      db "WinClass",0
AppName        db "NetLinkTime_Ver1.0 COPY(C)himcrack",0
RestoreString  db "(&R)显示程序",0
ExitString     db "(&X)退出程序",0
ButtonCNm      db "Static",0
buffer         db 0

.data?
mr             dd ?
hInstance      dd ?
note           NOTIFYICONDATA <>
hPopupMenu     dd ?

.code
start:
invoke GetModuleHandle, NULL
mov    hInstance,eax
invoke WinMain, hInstance,NULL,NULL, SW_SHOWDEFAULT

.if linkN!=0
invoke CreateFile,ADDR Fname,GENERIC_READ or GENERIC_WRITE ,\
             FILE_SHARE_READ or FILE_SHARE_WRITE,\
             NULL,OPEN_ALWAYS,FILE_ATTRIBUTE_NORMAL,\
             NULL
mov pfile,eax
lea ebx,Ttime
invoke WriteFile,eax,ebx,4,ADDR mr,0
mov eax,pfile
invoke CloseHandle,eax
.endif
invoke ExitProcess,eax

WinMain proc hInst:HINSTANCE,hPrevInst:HINSTANCE,CmdLine:LPSTR,CmdShow:DWORD
    LOCAL wc:WNDCLASSEX
    LOCAL msg:MSG
    LOCAL hwnd:HWND
    LOCAL LCtime:SYSTEMTIME
    mov   wc.cbSize,SIZEOF WNDCLASSEX
    mov   wc.style, CS_HREDRAW or CS_VREDRAW or CS_DBLCLKS
    mov   wc.lpfnWndProc, OFFSET WndProc
    mov   wc.cbClsExtra,NULL
    mov   wc.cbWndExtra,NULL
    push  hInst
    pop   wc.hInstance
    mov   wc.hbrBackground,COLOR_APPWORKSPACE
    mov   wc.lpszMenuName,NULL
    mov   wc.lpszClassName,OFFSET ClassName
    ;****************************************************************
    invoke GetModuleHandle,NULL
    invoke LoadIcon,eax,1
    ;****************************************************************
    mov   wc.hIcon,eax
    mov   wc.hIconSm,eax
    invoke LoadCursor,NULL,IDC_ARROW
    mov   wc.hCursor,eax
    invoke RegisterClassEx, addr wc
    invoke CreateWindowEx,WS_EX_CLIENTEDGE,ADDR ClassName,ADDR Appname,\
    WS_OVERLAPPED+WS_CAPTION+WS_SYSMENU+WS_MINIMIZEBOX+WS_VISIBLE,CW_USEDEFAULT,\
    CW_USEDEFAULT,400,200,NULL,NULL,\
    hInst,NULL
    mov   hwnd,eax
    ;****************************************************************
    invoke SendMessage,hwnd,WM_SIZE,SIZE_MINIMIZED,NULL
    lea    eax,LCtime
    invoke GetLocalTime,eax
    .if LCtime.wDay==1
    invoke CreateFile,ADDR Fname,GENERIC_READ or GENERIC_WRITE ,\
             FILE_SHARE_READ or FILE_SHARE_WRITE,\
             NULL,OPEN_ALWAYS,FILE_ATTRIBUTE_NORMAL,\
             NULL
    mov pfile,eax
    invoke SetFilePointer,eax,4,0,FILE_BEGIN
    mov eax,pfile
    invoke ReadFile,eax,addr buffer,1,addr mr,NULL
    .if buffer==0
    mov eax,pfile
    invoke SetFilePointer,eax,0,0,FILE_BEGIN
    mov eax,pfile
    lea ebx,Iszero
    invoke WriteFile,eax,ebx,5,ADDR mr,0
    .endif
    mov eax,pfile
    invoke CloseHandle,eax
    .elseif LCtime.wDay==2
    invoke CreateFile,ADDR Fname,GENERIC_READ or GENERIC_WRITE ,\
             FILE_SHARE_READ or FILE_SHARE_WRITE,\
             NULL,OPEN_ALWAYS,FILE_ATTRIBUTE_NORMAL,\
             NULL
    mov pfile,eax
    invoke SetFilePointer,eax,4,0,FILE_BEGIN
    mov eax,pfile
    lea ebx,Iszero
    invoke WriteFile,eax,ebx,1,ADDR mr,0
    mov eax,pfile
    invoke CloseHandle,eax
    .endif
    invoke CreateFile,ADDR Fname,GENERIC_READ or GENERIC_WRITE ,\
             FILE_SHARE_READ or FILE_SHARE_WRITE,\
             NULL,OPEN_ALWAYS,FILE_ATTRIBUTE_NORMAL,\
             NULL
    mov pfile,eax
    invoke ReadFile,eax,addr Ttime,4,addr mr,NULL
    mov eax,pfile
    invoke CloseHandle,eax
    invoke SetTimer,hwnd,1,1000,OFFSET TimProc
    ;****************************************************************
    .while TRUE
    invoke GetMessage, ADDR msg,NULL,0,0
    .BREAK .IF (!eax)
    invoke TranslateMessage, ADDR msg
    invoke DispatchMessage, ADDR msg
    .endw
    mov eax,msg.wParam
    ret
WinMain endp

WndProc proc hWnd:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM
    LOCAL pt:POINT
    .if uMsg==WM_CREATE
    Invoke CreateWindowEx,0,Addr ButtonCNm,addr play,WS_VISIBLE or WS_CHILD or WS_BORDER or BS_PUSHBUTTON,50,20,250,20,hWnd,IdPlay,hInstance,0
    mov hStatic,eax
    invoke CreatePopupMenu
    mov hPopupMenu,eax
    invoke AppendMenu,hPopupMenu,MF_STRING,IDM_RESTORE,addr RestoreString
    invoke AppendMenu,hPopupMenu,MF_STRING,IDM_EXIT,addr ExitString
    .elseif uMsg==WM_DESTROY
    invoke DestroyMenu,hPopupMenu
    invoke PostQuitMessage,NULL
    .elseif uMsg==WM_SIZE
    .if wParam==SIZE_MINIMIZED
    mov note.cbSize,sizeof NOTIFYICONDATA
    push hWnd
    pop note.hwnd
    mov note.uID,IDI_TRAY
    mov note.uFlags,NIF_ICON+NIF_MESSAGE+NIF_TIP
    mov note.uCallbackMessage,WM_SHELLNOTIFY
    ;****************************************************************
    invoke GetModuleHandle,NULL
    invoke LoadIcon,eax,1
    ;****************************************************************
    mov note.hIcon,eax
    invoke lstrcpy,addr note.szTip,addr AppName
    invoke ShowWindow,hWnd,SW_HIDE
    invoke Shell_NotifyIcon,NIM_ADD,addr note
    .endif
    .elseif uMsg==WM_COMMAND
    .if lParam==0
    invoke Shell_NotifyIcon,NIM_DELETE,addr note
    mov eax,wParam
    .if ax==IDM_RESTORE
    invoke ShowWindow,hWnd,SW_RESTORE
    .else
    invoke DestroyWindow,hWnd
    invoke KillTimer,hWnd,1
    .endif
    .endif
    .elseif uMsg==WM_SHELLNOTIFY
    .if wParam==IDI_TRAY
    .if lParam==WM_RBUTTONDOWN
    invoke GetCursorPos,addr pt
    invoke SetForegroundWindow,hWnd
    invoke TrackPopupMenu,hPopupMenu,TPM_RIGHTALIGN,pt.x,pt.y,NULL,hWnd,NULL
    invoke PostMessage,hWnd,WM_NULL,0,0
    .elseif lParam==WM_LBUTTONDBLCLK
    invoke SendMessage,hWnd,WM_COMMAND,IDM_RESTORE,0
    .endif
    .endif
    .else
    invoke DefWindowProc,hWnd,uMsg,wParam,lParam
    ret
    .endif
    xor eax,eax
    ret
WndProc endp

TimProc proc hWnd:HWND,nMsg:UINT,nTimerid:UINT,dwTime:DWORD
;****************************************************************
;以下为写文件的 TIMER 回调函数
;****************************************************************
mov eax,sizeof rasconn
inc eax
mov rasconn.dwSize,eax
invoke RasEnumConnections,addr rasconn,addr long,addr linkN
.if linkN==0
mov eax,Ttimexian
mov Ttime,eax
jmp exit
.endif
mov eax,sizeof rastats
mov rastats.dwSize,eax
invoke RasGetConnectionStatistics,rasconn.hrasconn,addr rastats
mov eax,rastats.dwConnectDuration
mov ecx,1000
cdq
div ecx
mov Taddtime,eax ;本次当前联网秒数
;****************************************************************
mov eax,Ttime
add eax,Taddtime
mov Ttimexian,eax
invoke CreateFile,ADDR Fname,GENERIC_READ or GENERIC_WRITE ,\
             FILE_SHARE_READ or FILE_SHARE_WRITE,\
             NULL,OPEN_ALWAYS,FILE_ATTRIBUTE_NORMAL,\
             NULL
mov pfile,eax
lea ebx,Ttimexian
invoke WriteFile,eax,ebx,4,ADDR mr,0
mov eax,pfile
invoke CloseHandle,eax
mov eax,Ttimexian
mov ecx,60
cdq
div ecx
mov miao,edx
cdq
div ecx
mov fen,edx
mov shi,eax
invoke wsprintf,addr play,addr tamp,shi,fen,miao
invoke SetWindowText,hStatic,addr play

exit: ret
TimProc endp
end start
```



## 算法设计思考

算法设计上存在问题，并不能与商业作品比。不过基本功能已经做出来了，想完善还会远么？呵呵……

## 前言

过多的东西我也不写了，这篇文章的代码部分很长，所以篇幅有些大，就不再赘言了。

感谢您耐心看到这里。为可以忍受菜菜的笔记而感谢…
![扮鬼脸](img/f43c1e514327696dccbb0da997496b2a.jpg)

---

## 经验总结

编程时遇到很大难题，`API` 调用出错几率很大，细心 `coding` 耐心 `debuging` 才是最大的收获。

所有用到的 `API` 的 `ASM` 调用形式在此帖汇集，希望可以帮助到和我一样菜的小菜虫~

`google`、`baidu` 要善用…

`THAT is ALl!!!!!`

`THX`

---

## 发布时间

`2008 年 11 月 10 日 4:40:15`

## 上传的附件

| | NetLinkTime.rar |
| :--- | :--- |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | |
