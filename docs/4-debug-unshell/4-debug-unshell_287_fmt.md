
# SWF Decompiler MX 2005 (arm4) 脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-543.htm`](https://www.pediy.com/kssd/pediy07/pediy7-543.htm)

Sothink SWF Decompiler MX 2005a Build 50318 (Arm 4) 脱壳

注：网上很早就有破解版，但有加壳压缩了还有 1.9M 多，原因你也看一下，脱壳后更利害 20 多 M，不能跨平台，懒得用

我也来一下，什么，自己脱壳！我自己优化后（不是最优）2M 多点加壳才 695K，再脱壳也不是这个 20 多 M 的天文数字

我不会提供程序，想练手可以自己到网上找一下这个版本吧

**【脱壳保护】**：双进程调试保护 `CopyMEM2`，IAT 乱序，Stolen Code 远程跳转，CC，壳注册保护

**【脱壳难度】**：熟悉了解了就不是难的事。做成脚本运行，就是好玩！

**【作者声明】**：写此文只是感兴趣，同时我也很想借此机会锻炼一下自己的语文（主要原因），经常是人家听不懂，说话又结巴的。失误之处敬请诸位大侠赐教

**【调试环境】**：`WinXP`、`OllyDbg1.10_Diy`（我喜欢玩自己 DIY 的）`_Anti`、`PEiD`、`LordPE`、`ImportRec`

## 一、探望 OEP

`PEid0.93` 查得是 `Armadillo 3.78` -> `Silicon Realms Toolworks`

**【OD 设置】**：忽略所有异常，同时忽略异常范围勾上：

`C000001D` (`ILLEGAL ISTRUCTION`)

`C0000005` (`ACCESS ViO`)

`C000001E` (`INVALID LOCK SEQUENCE`)

OD 载入停在入口处，隐藏好调试标记

```
代码:

* * *

005C5000 >pushad                //停在入口
005C5001  call 005C5006         ; SWFDecom.005C5006
005C5006  pop ebp
005C5007  push eax
005C5008  push ecx
005C5009  bswap edx
005C500B  not edx
005C500D  pushfd
005C500E  not edx
005C5010  bswap edx
005C5012  jmp short 005C5023    ; SWFDecom.005C5023

* * *

```

在 `"OpenMutexA"`, `"kernel32.dll"` 上先进行单转双的操作，有很多的教程都有，脚本又有，不多说了

转换成功后，取消所有断点

命令行设置断点：

`he OutputDebugStringA`

`he SetProcessWorkingSetSize`

点 `F9` 运行，中断在 `OutputDebugStringA` 上，说明是 `arm4` 以上的版本加的壳

这个反调试手段，没两下，就会人识穿，如果你没自己玩一下 `DIY`，按 `Fly` 等说的办

取消这个断点，点 `F9` 运行，

中断在 `SetProcessWorkingSetSize` 上，取消这个断点。

`Alt+F9` 返回，找一下“`armVersion`”

加壳版本显示变成这样，要运行它前面一大堆的代码才能出来的，没有以前的直接 `Push` 版本号了

```
代码:

* * *

00D6E613  add esp,18
00D6E616  push eax                          //这个应该是版本号
00D6E617  push 0D96008                      ; ASCII "   <armVersion xsi:type=""xsd:string"">%s</armVersion>",LF
00D6E61C  lea eax,dword ptr ss:[ebp-2200]
00D6E622  push 0
00D6E624  push eax
00D6E625  call esi

* * *

```

`Ctrl+S` 在当前位置下搜索命令序列：

`sub ecx, edx`

`call ecx`

有两处如下：

```
代码:

* * *

00D8C7E5  sub ecx,edx
00D8C7E7  call ecx          //这里估计也是 OEP，可能是校验或解码出错跑这吧
00D8C7E9  jmp short 00D8C806
00D8C7EB  cmp edi,1
00D8C7EE  jnz short 00D8C808
00D8C7F0  push dword ptr ds:[esi+4]
00D8C7F3  push dword ptr ds:[esi+8]
00D8C7F6  push 0
00D8C7F8  push edx
00D8C7F9  mov edx,dword ptr ds:[eax+74]
00D8C7FC  xor edx,dword ptr ds:[eax+60]
00D8C7FF  xor edx,dword ptr ds:[eax+2C]
00D8C802  sub ecx,edx
00D8C804  call ecx  //**** 这里是前往 OEP 4B2BCD ****//

* * *

```

## 二、Magic Jump，避开 IAT 加密

重来，方法是用 `fly` 的定位方式，到达 `Magic Jump` 的地方，方便避开了反跟踪，

```
代码:

* * *

00D74588  call dword ptr ds:[D92104]         ; kernel32.GetModuleHandleA
00D7458E  mov ecx,dword ptr ds:[D9F67C]      //返回这里
00D74594  mov dword ptr ds:[esi+ecx],eax
00D74597  mov eax,dword ptr ds:[D9F67C]
00D7459C  cmp dword ptr ds:[esi+eax],ebx
00D7459F  jnz short 00D745B7
00D745A1  lea eax,dword ptr ss:[ebp-14C]
00D745A7  push eax
00D745A8  call dword ptr ds:[D9210C]         ; kernel32.LoadLibraryA
00D745AE  mov ecx,dword ptr ds:[D9F67C]
00D745B4  mov dword ptr ds:[esi+ecx],eax
00D745B7  mov eax,dword ptr ds:[D9F67C]
00D745BC  cmp dword ptr ds:[esi+eax],ebx
00D745BF  je 00D746F4                       //传闻很久的 Magic Jump，改跳就是啦
00D745C5  xor ecx,ecx
00D745C7  mov eax,dword ptr ds:[edi]
00D745C9  cmp dword ptr ds:[eax],ebx
00D745CB  je short 00D745D3

* * *

```

`IAT` 处理可以运行我本人推荐的脚本进行修复，不懂不想用你也可以自已写代码修复，网上也留传很多修复方案，很简单的事！

多说几句，如果你用我的 `IAT` 修复脚本，那么在完成第 4 步，你可以先将整个申请内存块复制下来备用，

在 `OEP` 处（或重来在你认为适当的时机，申请同样大小的内存块，将备份块弄回去，注意要参数要对应，一般我不推荐这样做 ，但若你申请到的块地址是一样的就不用改参数）运行第 5 个进行修复。

我脚本修复的最终特点是：`IAT` 没有重复的 `API`，按模块排序，方便更改跨平台 `API`（一般指 `NT.dll` 的 `API`）；另外，我所说的“未必通用”是指前往 `Magic JMP` 的脚本 1（脚本 1 是用来修改 `Magic Jmp` 和到达 `Iat` 正式处理的地方），其它的可以套用，关键你看懂了脚本参数没有。

## 三、Stolen Code 远程跳转

```
代码:

* * *

004010A0  jmp 03530000             //这里就是 Stolen Code 远程跳转的块首地址
004010A5  push edi
004010A6  pop edi
004010A7  mov eax,dword ptr fs:[0]
004010AD  jmp 03530023
004010B2  push ecx
004010B3  xchg ebx,edx
004010B5  bswap edi
004010B7  bswap edi
004010B9  bswap edi
004010BB  bswap edi
004010BD  xchg ebx,edx
004010BF  pop ecx
004010C0  call 00503183                   ; SWFDecom.00503183

* * *

```

脱壳后的程序需要有这个 `Stolen Code` 远程跳转块，才能正常运行，没有的话...嘿嘿

解决的办法，可以简单处理，也可以变态还原处理，一般我们建议简单处理。

**简单处理**：在脱壳后的程序增开一个节，将该节的 `RVA` 设置为当时动态获得的 `Stolen Code` 的 `RVA`

**变态还原**：指将在 `Stolen Code` 块隐藏的程序原本的代码，逐一还原到代码段。

我们来到一下这个处理的地方：

现在的新版，已经不再使用 `VirtualAlloc` 这个函数申请空间，而是采用 `msvcrt.operator new`

当然这个名称是 `OD` 打印给我们的，实际上是这样的形式：`"??2@YAPAXI@Z"`, `"msvcrt.dll"`

经过修改 `Magic Jmp` 后 中断这个函数，注意观察函数申请大小的变化，变成 `142000` 的要提神了。



```
代码:

00D880EC  call 00D91726                          ; jmp to msvcrt.operator new
00D880F1  pop ecx                                 //返回这里
00D880F2  mov dword ptr ss:[ebp+FFFFAFD0],eax
00D880F8  mov eax,dword ptr ss:[ebp+FFFFAFD0]
00D880FE  mov dword ptr ss:[ebp-2A34],eax
00D88104  mov eax,dword ptr ss:[ebp-2A34]
00D8810A  mov dword ptr ss:[ebp-2A2C],eax
00D88110  mov eax,dword ptr ss:[ebp-2A38]
00D88116  add eax,10000                           ; UNICODE "=::=::\\"
……
00D88185  push 0D98A20                          ; ASCII "Location ES1"
00D8818A  mov eax,dword ptr ss:[ebp+8]
00D8818D  push dword ptr ds:[eax+4]
00D88190  call 00D917BA                       ; jmp to msvcrt.strcpy
00D88195  pop ecx
00D88196  pop ecx
00D88197  xor eax,eax
00D88199  jmp 00D8AA42
00D8819E  cmp dword ptr ss:[ebp-2A44],0
00D881A5  jnz short 00D881A9
00D881A7  jmp short 00D881C0
00D881A9  mov eax,dword ptr ss:[ebp-2A2C]
00D881AF  add eax,dword ptr ss:[ebp-2A44]
00D881B5  mov dword ptr ss:[ebp-2A2C],eax
00D881BB  jmp 00D8812C
00D881C0  mov eax,dword ptr ss:[ebp-2A2C]     ; // 401000 块的首次解码内容的首地址
00D881C6  sub eax,dword ptr ss:[ebp-2A34]     ; //应该是 401000 解码后的 最后地址
00D881CC  mov dword ptr ss:[ebp-2A30],eax     ; // 保存 41000 段解码后的 大小
00D881D2  mov eax,dword ptr ss:[ebp-2A30]
00D881D8  cmp eax,dword ptr ss:[ebp-2A38]     ; // 比较是否与小于内存映像块的大小
00D881DE  jnb short 00D881FD
00D881E0  mov eax,dword ptr ss:[ebp-2A38]     ; // 默认是小于
00D881E6  sub eax,dword ptr ss:[ebp-2A30]     ; // 求它们之间的差：size
00D881EC  push eax                            ; // 将其差值空间置 0
00D881ED  push 0
00D881EF  push dword ptr ss:[ebp-2A2C]
00D881F5  call 00D91732                       ; jmp to msvcrt.memset
00D881FA  add esp,0C
00D881FD  cmp dword ptr ss:[ebp-2800],0     ; // 应该是一个表来的，地址为：1007360
00D88204  je 00D8837D                       ; // 为 0 表示不处理或结束吧
00D8820A  mov eax,dword ptr ss:[ebp-27F4]   ; // 程序基址
00D88210  add eax,dword ptr ss:[ebp-2A3C]   ; // 基址 + 本块的 RVA
00D88216  mov ecx,dword ptr ss:[ebp-2800]
00D8821C  cmp dword ptr ds:[ecx],eax        ; // 比较是否少于该块的基地
00D8821E  jb 00D8837D                       ; // 小于也不处理
00D88224  mov eax,dword ptr ss:[ebp-27F4]   ; // 基址
00D8822A  add eax,dword ptr ss:[ebp-2A3C]   ; // 本块的 RVA
00D88230  add eax,dword ptr ss:[ebp-2A38]   ; // + size = 计算出下一个块的地址
00D88236  mov ecx,dword ptr ss:[ebp-2800]   ; // 表
00D8823C  cmp dword ptr ds:[ecx],eax        ; // 是否超越边界
00D8823E  jnb 00D8837D
00D88244  mov ecx,0D9B448
00D88249  call 00D8B9DE
00D8824E  mov byte ptr ss:[ebp-2A48],al
00D88254  movzx eax,byte ptr ss:[ebp-2A48]
00D8825B  shl eax,18
00D8825E  movzx ecx,byte ptr ss:[ebp-2A48]
00D88265  shl ecx,10
00D88268  or eax,ecx
00D8826A  movzx ecx,byte ptr ss:[ebp-2A48]
00D88271  shl ecx,8
00D88274  or eax,ecx
00D88276  movzx ecx,byte ptr ss:[ebp-2A48]
00D8827D  or eax,ecx
00D8827F  mov dword ptr ss:[ebp-2A4C],eax
00D88285  mov eax,dword ptr ss:[ebp-2800]
00D8828B  mov dword ptr ss:[ebp-2A50],eax
00D88291  mov eax,dword ptr ss:[ebp-2A50]
00D88297  cmp dword ptr ds:[eax],0           ; // 表指针，是否完成
00D8829A  je 00D8837D
00D882A0  mov eax,dword ptr ss:[ebp-2A50]
00D882A6  mov eax,dword ptr ds:[eax]         ; //取出补丁地址
00D882A8  mov dword ptr ss:[ebp-2A58],eax
00D882AE  mov eax,dword ptr ss:[ebp-2A50]
00D882B4  add eax,4                           ; // 表指针 +4
00D882B7  mov dword ptr ss:[ebp-2A50],eax
00D882BD  mov eax,dword ptr ss:[ebp-2A50]
00D882C3  mov eax,dword ptr ds:[eax]         ; // 取跳转偏移量
00D882C5  mov dword ptr ss:[ebp-2A54],eax
00D882CB  mov eax,dword ptr ss:[ebp-2A50]
00D882D1  add eax,4
00D882D4  mov dword ptr ss:[ebp-2A50],eax
00D882DA  mov eax,dword ptr ss:[ebp-27F4]
00D882E0  add eax,dword ptr ss:[ebp-2A3C]
00D882E6  cmp dword ptr ss:[ebp-2A58],eax
00D882EC  jbe short 00D8836A
00D882EE  mov eax,dword ptr ss:[ebp-27F4]
00D882F4  add eax,dword ptr ss:[ebp-2A3C]
00D882FA  add eax,dword ptr ss:[ebp-2A38]
00D88300  cmp dword ptr ss:[ebp-2A58],eax   ; // 比较代码段的补丁地址是否大于代码段地址
00D88306  jnb short 00D8836A
00D88308  mov eax,dword ptr ss:[ebp-27F4]
00D8830E  add eax,dword ptr ss:[ebp-2A3C]
00D88314  mov ecx,dword ptr ss:[ebp-2A58]
00D8831A  sub ecx,eax                       ; // 取本块补丁地址偏移量, 如:4010A1 - 401000 = A1
00D8831C  mov dword ptr ss:[ebp-2A5C],ecx
00D88322  mov al,byte ptr ds:[D9B86C]
00D88327  mov byte ptr ss:[ebp+FFFFAE38],al
00D8832D  movzx eax,byte ptr ss:[ebp+FFFFAE>
00D88334  test eax,eax
00D88336  je short 00D88354
00D88338  mov eax,dword ptr ss:[ebp-2A54]
00D8833E  xor eax,dword ptr ss:[ebp-2A4C]
00D88344  mov ecx,dword ptr ss:[ebp-2A34]
00D8834A  add ecx,dword ptr ss:[ebp-2A5C]
00D88350  mov dword ptr ds:[ecx],eax
00D88352  jmp short 00D88368
00D88354  mov eax,dword ptr ss:[ebp-2A34]   ; // 取代码段首地址
00D8835A  add eax,dword ptr ss:[ebp-2A5C]   ; // + 补丁地址偏移量 = 计算出修复的地址
00D88360  mov ecx,dword ptr ss:[ebp-2A54]   ; // 取跳转偏移量，补丁的机会 First:
00D88366  mov dword ptr ds:[eax],ecx        ; // 写入跳转偏移量
00D88368  jmp short 00D88378
00D8836A  mov eax,dword ptr ss:[ebp-2A58]
00D88370  mov ecx,dword ptr ss:[ebp-2A54]   ; // 取跳转偏移量， 也是机会 Second:
00D88376  mov dword ptr ds:[eax],ecx        ; // 写入偷取代码块的跳转偏移量
00D88378  jmp 00D88291
```

//表：`1007360` 的组成很有规律的，对应动态申请的 `Stolen Code`，自己看一下就明白了。

//上面这段是处理 `Stolen Code` `JMP` 偏移量的计算处理，因为是块动态申请的，所以壳要做这部分工作

//看到了吧，跟一下，就能体会到的，这时还没写入 `IAT`，过了这个步骤，就快要解码处理 `IAT` 的代码！

//现在你可以做的事，不管它，直接到 `IAT` 处理的部分；来个小小的处理，使 `DUMP` 文件一脱下来就有 `Stolen Code` 部分(这个要在入口处做个复制粘贴的工作，我是用脚本完成的)

找个空间地方写类似下面的代码，主要是将 `JMP` 的偏移地址改到你认为合适的程序地址，我选择了壳的运行代码块 `585000`

`First: 4010000`

壳申请首地址（`1B80000`）--动态的

-跳转偏移值（`ecx`）

自定义 `Stolen Code` 首地址（`585000`）

```
D88360  Call (First) //改到你补丁的地方
D88365  nop
D88366  mov dword ptr ds:[eax],ecx
```

`First:`

```
  mov ecx,dword ptr ss:[ebp-2A54] //失去的指令
  push eax
  mov eax,1B80000
  sub eax,585000
  sub ecx,eax
  pop eax
  retn
  nop
  nop
```

`Second: 585000`

```
D88370  call (Second)  //改到你补丁的地方
D88375  nop
```

`Second:`

```
  mov ecx,dword ptr ss:[ebp-2A54]
  push eax
  mov edx,1B80000
  add eax,4
  add ecx,eax
  sub eax,edx
  add eax,585000
  sub ecx,eax
  pop eax
  retn
  nop
  nop
```

//在 `00D88378` 的下一个指令下断（即出口），运行中断后，撤消刚才的代码，免得校验出问题！

//接下来是 `IAT` 的处理部分，我不多说了，`fxyang` 等已经介绍过很多了！



```
代码:

* * *

00D8A454     call dword ptr ds:[D92170]                 ; kernel32.VirtualProtect
00D8A45A     mov al,byte ptr ds:[D9B86C]                //经典的更改节属性为可写的跟踪必中点
00D8A45F     mov byte ptr ss:[ebp+FFFFAD30],al
00D8A465     movzx eax,byte ptr ss:[ebp+FFFFAD30]
00D8A46C     test eax,eax
00D8A46E     je short 00D8A48D
00D8A470     push 0
00D8A472     push dword ptr ss:[ebp+FFFFB0C8]
00D8A478     mov eax,dword ptr ss:[ebp-27F4]
00D8A47E     add eax,dword ptr ss:[ebp+FFFFB0B8]
00D8A484     push eax
00D8A485     call 00D8B4A7
00D8A48A     add esp,0C
00D8A48D     mov eax,dword ptr ss:[ebp-2804]
00D8A493     mov esi,dword ptr ss:[ebp-27F4]
00D8A499     sub esi,dword ptr ds:[eax+34]
00D8A49C     call 00D8CFFD
00D8A4A1     neg eax
00D8A4A3     sbb eax,eax
00D8A4A5     and eax,1000
00D8A4AA     add esi,eax
00D8A4AC     mov dword ptr ss:[ebp+FFFFB0C0],esi
00D8A4B2     and dword ptr ss:[ebp+FFFFB0C4],0
00D8A4B9     jmp short 00D8A4C8
00D8A4BB     mov eax,dword ptr ss:[ebp+FFFFB0C4]
00D8A4C1     inc eax
00D8A4C2     mov dword ptr ss:[ebp+FFFFB0C4],eax
00D8A4C8     mov eax,dword ptr ss:[ebp+FFFFB0C4]
00D8A4CE     mov ecx,dword ptr ss:[ebp-26DC]
00D8A4D4     cmp dword ptr ds:[ecx+eax*4],0
00D8A4D8     je 00D8A56E
00D8A4DE     mov eax,dword ptr ss:[ebp+FFFFB0C4]
00D8A4E4     mov ecx,dword ptr ss:[ebp-26DC]
00D8A4EA     mov edx,dword ptr ss:[ebp-27F4]
00D8A4F0     add edx,dword ptr ds:[ecx+eax*4]
00D8A4F3     mov dword ptr ss:[ebp+FFFFB0B4],edx
00D8A4F9     mov eax,dword ptr ss:[ebp+FFFFB0B4]
00D8A4FF     mov eax,dword ptr ds:[eax]
00D8A501     mov dword ptr ss:[ebp+FFFFB0B0],eax
00D8A507     cmp dword ptr ss:[ebp+FFFFB0B0],90909090
00D8A511     je short 00D8A569
00D8A513     mov eax,dword ptr ss:[ebp+FFFFB0B0]
00D8A519     sub eax,dword ptr ss:[ebp+FFFFB0C0]
00D8A51F     mov dword ptr ss:[ebp+FFFFB0B0],eax
00D8A525     push dword ptr ss:[ebp+FFFFB0B0]
00D8A52B     mov eax,dword ptr ss:[ebp+FFFFB0C4]
00D8A531     xor edx,edx
00D8A533     push 10
00D8A535     pop ecx
00D8A536     div ecx
00D8A538     call dword ptr ds:[edx*4+D927EC]
00D8A53F     pop ecx
00D8A540     mov dword ptr ss:[ebp+FFFFB0B0],eax
00D8A546     mov eax,dword ptr ss:[ebp+FFFFB0B0]
00D8A54C     mov ecx,dword ptr ss:[ebp-281C]            ; // Start IAT Position
00D8A552     lea eax,dword ptr ds:[ecx+eax*4]
00D8A555     mov dword ptr ss:[ebp+FFFFB0B0],eax
00D8A55B     mov eax,dword ptr ss:[ebp+FFFFB0B4]        ; // Modify Address
00D8A561     mov ecx,dword ptr ss:[ebp+FFFFB0B0]        ; // Api Address
00D8A567     mov dword ptr ds:[eax],ecx                 ; // Writing Maybe Crypt Api Address
00D8A569     jmp 00D8A4BB

* * *
```

// 我们再来找 Anti Dump，主要是壳会修改 PE 头的 OffSet 位置（因为他已经映射好了），使得你通过获取程序进程无法分析或 Dump 下来的脱壳文件天生就是错误的 `PEheader`

// 对付的办法也挺简单，也是 `fly` 常说的选择来自磁盘文件的 `PEheader`。想知道壳哪里动手，也很简单，在`[基地 + 3C]`这个 `Dword` 上下访问断点，中断后就知道了，不贴代码了！

// `IAT` 处理完，就可按上面前往 `OEP` 的方式抵达 `OEP`，这时可以将 `Stolen Code` 的块复制到你预定程序的位置 如：`585000`

// 将程序的所有节权限全部改为完整（点内存窗口就可以右键菜单改），我是用脚本 `movedata.txt` 完成的，`OD` 的复制粘贴也是很简单的事！

// 如果你是用我的脚本，现在你看到的一些 `Call []` 是 壳处理的结果，再运行我第 5 个 脚本，嘿嘿，怎么样，不到 1 秒钟就很好看了。

// 现在可以拿出 `LordPE` 和 `ImportRec` 分别 Dump 出脱壳文件和获取 Iat 信息，`ImportRec` 只要入口填对，就能获得整齐简洁的 `IAT`，可以先保存成树文件下来。

到现在为止，你如果想获得跨平台，我这里 XP 没有装 `flash` 控件，所以要到 98 上运行，就要改 `nt.dll` 上的 `API` 为与之对应的 `Kernel32.dll` 的 `API`，`fly` 的 `arm4.0` 加壳的精华 last 版有介绍，就不多说了，改刚才的 Iat 树文件吧，搞定后 Load 之， fix Iat!

一运行，嘿嘿，我们就要遇上 CC 了，好在这个程序没有加上很多的 CC，所以手工修复，也是个不错的选择，多数都很好猜是硬插的绝对跳转来的，不猜也行！

### 四、CC 修复

下面是 CC 手动修复（只有修复到能运行，不是参考 snoop 汉化版的过程，其它的自己来吧，不会很多，基本上修复的地方与破解是关联的）

在这个阶段大部分是手工活，也是最花时间的 `Armadillo` 的 Nanomites 保护，其特征代码我就不全贴了，没意义，前人的文章多的是。

我们仍然需要取得 4 个表：`INT3` 地址表，跳转类型表，跳转距离表，非跳距离表

网上也没有流传 能够一次性完整修复 所有 CC 的方案

当然你喜欢查表，逆向运算表的，自己玩，我可没这本事！

虽然我看过看雪精华 6 前面兄弟的修复文章，又 Nop 代码，又移动代码，又加自己代码，一轮又一轮...看得挺烦的，不如按自己理解的来（跳转类型方案还没完全确定下来，不过确定的现在够用），这里我只是用自己写代码 Log 运行程序记录的 4 个表，即属于一种被动修复方式，适用于 CC 修复量较小。

`OD` 设置一样，重来加载程序，`He GetThreadContext`，中断两次就取消断点来到：

```
代码:

* * *

005A144F  call dword ptr ds:[5D50DC]  ; kernel32.GetThreadContext
005A1455  push eax                    //来到这里
005A1456  not eax
005A1458  bswap eax
005A145A  pop eax
……//向下翻找到这里
005A1614   mov eax,dword ptr ss:[ebp-119C]
005A161A   mov ecx,dword ptr ds:[eax*4+5DBE90]
……
005A170C  add esp,4
005A170F  mov dword ptr ss:[ebp-1480],eax
005A1715  mov eax,dword ptr ss:[ebp-1480]
005A171B  and eax,1
005A171E  test eax,eax                 //计算跳与不跳的关键点，在这里补一下代码
005A1720  je 005A17D4                  ; SWFDecom.005A17D4
005A1726  pushad
005A1727  xor eax,eax

* * *
```

我用 C-pen 的方式确定跳转类型，其实大家的方式是基本一致的，只不过是谁的好理解。

本例中 `5A1614` -- `5A171E` 是可以借助确定跳转类型的过程

申请一个内存块，用来运行和记录表，我申请了 `10000h`，地址是：`E60000`

贴上这片代码：

///////////////////////////////////

`E60000`:

```
00 00 00 00 00 00 00 00 01 00 00 00 04 00 00 00 40 00 00 00 80 00 00 00 00 08 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 90 90 90 90 8B 85 64 EE FF FF 8B 0C 85 90 BE 5D 00 8B 95 90
EB FF FF 33 C0 8A 04 11 89 85 74 EB FF FF 8B 85 74 EB FF FF 99 83 E2 0F 03 C2 C1 F8 04 89 85 7C
EB FF FF 8B 8D 74 EB FF FF 81 E1 0F 00 00 80 79 05 49 83 C9 F0 41 89 8D 78 EB FF FF 8B 95 7C EB
FF FF 3B 95 78 EB FF FF 75 1B 8B 85 78 EB FF FF 83 C0 01 25 0F 00 00 80 79 05 48 83 C8 F0 40 89
85 78 EB FF FF 8B 8D 74 EB FF FF 8B 95 7C EB FF FF 8B 04 8D E8 AF 5D 00 33 04 95 CC 52 5D 00 8B
8D 78 EB FF FF 33 04 8D CC 52 5D 00 89 85 84 EB FF FF E8 00 00 00 00 5A 81 EA D7 00 00 00 8B 02
8B 54 82 04 52 8B 85 74 EB FF FF 0F BE 88 20 98 5D 00 FF 14 8D 28 99 5D 00 83 C4 04 89 85 88 EB
FF FF 8B 95 44 EC FF FF 52 8B 85 88 EB FF FF 50 FF 95 84 EB FF FF 83 C4 08 50 8B 8D 74 EB FF FF
0F BE 91 20 98 5D 00 FF 14 95 68 99 5D 00 83 C4 04 89 85 80 EB FF FF 8B 85 80 EB FF FF 83 E0 01
90 90 90 90 E8 00 00 00 00 59 81 E9 49 01 00 00 8B 59 04 D1 E3 03 D8 89 59 04 8B 11 42 89 11 83
FA 06 0F 85 C8 FE FF FF 8B C1 05 00 80 00 00 8B 50 FC 89 1C 90 42 89 50 FC 33 C0 89 01 89 41 04
C3 90 90 90 60 8B 8D 64 EE FF FF 8B 0C 8D B0 BD 5D 00 8B 85 90 EB FF FF 33 D2 BE 10 00 00 00 F7
F6 8B 85 90 EB FF FF 8B 0C 81 33 8C 95 8C EE FF FF 8B 95 50 EC FF FF E8 00 00 00 00 58 2D BC 01
00 00 8B D8 81 C3 00 A0 00 00 8B 7B FC 41 89 0C BB 47 89 7B FC 05 00 E0 00 00 8B 78 FC 4A 89 14
B8 47 89 78 FC 61 90 90 90 90 60 8B 85 64 EE FF FF 8B 0C 85 D8 BE 5D 00 8B 95 90 EB FF FF 33 C0
8A 04 11 8B 8D 50 EC FF FF E8 00 00 00 00 5A 81 EA 0E 02 00 00 8B DA 81 C3 00 C0 00 00 8B 7B FC
40 89 04 BB 47 89 7B FC 61 C3 90
```

//块的顶部是 2 个变量和 4 个 EFL 常量，其它的以后用上的再说。

真正的运行代码如下：

//注：说到的“搬运代码”都是壳的代码，因为要用到表，所以无法统一，只好用“搬”

`L004`: // 这里地址是：`00E6002B`

```
mov eax,dword ptr ss:[ebp-119C]    //搬运 5A1614 -- 5A171F 的代码来确定 EFL 跳不跳

mov ecx,dword ptr ds:[eax*4+5DBE90]

mov edx,dword ptr ss:[ebp-1470]

xor eax,eax

mov al,byte ptr ds:[ecx+edx]

mov dword ptr ss:[ebp-148C],eax

mov eax,dword ptr ss:[ebp-148C]

cdq

and edx,0F

add eax,edx

sar eax,4

mov dword ptr ss:[ebp-1484],eax

mov ecx,dword ptr ss:[ebp-148C]

and ecx,8000000F

jns L022

dec ecx

or ecx,FFFFFFF0

inc ecx

L022:

mov dword ptr ss:[ebp-1488],ecx

mov edx,dword ptr ss:[ebp-1484]

cmp edx,dword ptr ss:[ebp-1488]

jnz L034

mov eax,dword ptr ss:[ebp-1488]

add eax,1

and eax,8000000F

jns L033

dec eax

or eax,FFFFFFF0

inc eax

L033:

mov dword ptr ss:[ebp-1488],eax

L034:

mov ecx,dword ptr ss:[ebp-148C]

mov edx,dword ptr ss:[ebp-1484]

mov eax,dword ptr ds:[ecx*4+5DAFE8]

xor eax,dword ptr ds:[edx*4+5D52CC]

mov ecx,dword ptr ss:[ebp-1488]

xor eax,dword ptr ds:[ecx*4+5D52CC]
```



```
  mov dword ptr ss:[ebp-147C],eax  

  call L042  

L042:  

  pop edx  

  sub edx,0D7  

  mov eax,dword ptr ds:[edx]  

  mov edx,dword ptr ds:[edx+eax*4+4]  

  push edx  

  mov eax,dword ptr ss:[ebp-148C]  

  movsx ecx,byte ptr ds:[eax+5D9820]  

  call dword ptr ds:[ecx*4+5D9928]  

  add esp,4  

  mov dword ptr ss:[ebp-1478],eax  

  mov edx,dword ptr ss:[ebp-13BC]  

  push edx  

  mov eax,dword ptr ss:[ebp-1478]  

  push eax  

  call dword ptr ss:[ebp-147C]  

  add esp,8  

  push eax  

  mov ecx,dword ptr ss:[ebp-148C]  

  movsx edx,byte ptr ds:[ecx+5D9820]  

  call dword ptr ds:[edx*4+5D9968]  

  add esp,4  

  mov dword ptr ss:[ebp-1480],eax  

  mov eax,dword ptr ss:[ebp-1480]  

  and eax,1                   // 搬运代码结束  

  nop  

  nop  

  nop  

  nop  

  call L071  

L071:  

  pop ecx  

  sub ecx,149  

  mov ebx,dword ptr ds:[ecx+4]  

  shl ebx,1  

  add ebx,eax  

  mov dword ptr ds:[ecx+4],ebx  

  mov edx,dword ptr ds:[ecx]  

  inc edx  

  mov dword ptr ds:[ecx],edx  

  cmp edx,6  

  jnz L004  

  mov eax,ecx  

  add eax,8000                //设定 Log 跳转类型表的地址  

  mov edx,dword ptr ds:[eax-4]  

  mov dword ptr ds:[eax+edx*4],ebx  

  inc edx  

  mov dword ptr ds:[eax-4],edx  

  xor eax,eax  

  mov dword ptr ds:[ecx],eax  

  mov dword ptr ds:[ecx+4],eax  

  retn  

  nop  

  nop  

  nop  

  pushad  

  mov ecx,dword ptr ss:[ebp-119C]    //搬过来的确定跳转距离  

  mov ecx,dword ptr ds:[ecx*4+5DBDB0]  

  mov eax,dword ptr ss:[ebp-1470]  

  xor edx,edx  

  mov esi,10  

  div esi  

  mov eax,dword ptr ss:[ebp-1470]  

  mov ecx,dword ptr ds:[ecx+eax*4]  

  xor ecx,dword ptr ss:[ebp+edx*4-1174]  

  mov edx,dword ptr ss:[ebp-13B0]   //搬运结束  

  call L107  

L107:  

  pop eax  

  sub eax,1BC  

  mov ebx,eax  

  add ebx,0A000               //设定 Log 跳转距离表的地址  

  mov edi,dword ptr ds:[ebx-4]  

  inc ecx  

  mov dword ptr ds:[ebx+edi*4],ecx  

  inc edi  

  mov dword ptr ds:[ebx-4],edi  

  add eax,0E000               //设定 Log 跳转类型表的地址  

  mov edi,dword ptr ds:[eax-4]  

  dec edx  

  mov dword ptr ds:[eax+edi*4],edx  

  inc edi  

  mov dword ptr ds:[eax-4],edi  

  popad  

  nop  

  nop  

  nop  

  nop  

  nop  

  pushad  

  mov eax,dword ptr ss:[ebp-119C]    //搬过来的确定不跳转距离  

  mov ecx,dword ptr ds:[eax*4+5DBED8]  

  mov edx,dword ptr ss:[ebp-1470]  

  xor eax,eax  

  mov al,byte ptr ds:[ecx+edx]  

  mov ecx,dword ptr ss:[ebp-13B0]    //搬运结束  

  call L135  

L135:  

  pop edx  

  sub edx,20E  

  mov ebx,edx  

  add ebx,0C000               //设定 Log 不跳转距离表的地址  

  mov edi,dword ptr ds:[ebx-4]  

  inc eax  

  mov dword ptr ds:[ebx+edi*4],eax  

  inc edi  

  mov dword ptr ds:[ebx-4],edi  

  popad  

  retn //返回 005A172F  

  nop
```

```
///////////////////////////////////////////  
补丁到 Log 的代码  

5A171E:  

9C 60 33 C0 E8 1D EA 8B 00 61 9D 90 E8 55 EA 8B 00 85 C0 0F 84 9D 00 00 00 90 90 90 90 90 90 90  
90 90 90 90 90 90 90 90 90 90 90 90 90 90  

005A171E   pushfd  
005A171F   pushad  
005A1720   xor eax,eax  
005A1722   call 00E60144  
005A1727   popad  
005A1728   popfd  
005A1729   nop  
005A172A   call 00E60184  
005A172F   test eax,eax  
005A1731   je 005A17D4      ; SWFDecom.005A17D4  
005A1737   nop  

……  
```

运行后，点 `Continue` 按钮继续试用，这样就已经登记了我们要 `Log` 的 4 个表了（计算器的就不贴了，在表的上面），如下：

CC 地址表（顺序激活发生，好在没有重复，我这里的代码没有排除重复的）：

```
42835B  
428367  
42838C  
4A4B9B  
4A4BBC  
4A4346  
4A4362  
4A43EA  
4A440B  
4283CB  
4283EE  
```

```
Offset      0  1  2  3  4  5  6  7   8  9  A  B  C  D  E  F  
cc\_addr\_tab:  
0000E000   5B 83 42 00  67 83 42 00  8C 83 42 00  9B 4B 4A 00  
0000E010   BC 4B 4A 00  46 43 4A 00  62 43 4A 00  EA 43 4A 00  
0000E020   0B 44 4A 00  CB 83 42 00  EE 83 42 00  

jmp\_kind\_tab:  
00008000   1F 00 00 00  1B 00 00 00  04 00 00 00  1F 00 00 00  
00008010   1F 00 00 00  1F 00 00 00  14 00 00 00  1F 00 00 00  
00008020   1F 00 00 00  04 00 00 00  1F 00 00 00  

jmp\_of\_tab:  
0000A000   05 00 00 00  87 00 00 00  10 00 00 00  05 00 00 00  
0000A010   05 00 00 00  05 00 00 00  15 00 00 00  05 00 00 00  
0000A020   05 00 00 00  1E 00 00 00  05 00 00 00  

nojmp\_of\_tab:  
Offset      0  1  2  3  4  5  6  7   8  9  A  B  C  D  E  F  
0000C000   05 00 00 00  06 00 00 00  02 00 00 00  05 00 00 00  
0000C010   05 00 00 00  05 00 00 00  02 00 00 00  05 00 00 00  
0000C020   05 00 00 00  02 00 00 00  05 00 00 00  
```

跳转类型：

*   `1F` 是 `jmp` -> `11111`
*   `1B` 是 `jne` -> `11011`
*   `04` 是 `je`  -> `00100`
*   `14` 是 `jbe` -> `10100` `//`这个不太肯定，`CF` 也可以作用，不过它上面是这条指令：`test eax, eax`

推出修复：

`42835B`：

`jmp`

指令长度为：5 字节（不跳距离）

`jmp (42835B+5)`

`EB03` 或 `E900000000` (5-5)

`428367`：

`jne`

指令长度为：6 字节（不跳距离）

`jne (428367+87)`

`0F8581000000` (87-6)

`42838C`:

`je`

指令长度为：2 字节（不跳距离）

`je (42838C+10)`

`740E` (10-2)

`4A4362`:

`jbe`

指令长度为：2 字节（不跳距离）

`jbe (4A4362+15)`

`7613` (15-2)

还有其它的绝对跳转就不再举了。。。一样处理

当然可以再写代码修复，自己玩吧，修复后保存它，现在它可以正常运行了，但只修复到运行，加载 `swf` 文件，点其它菜单还有 `CC` 没修复，我是在 98 修的（猜也好玩）-- 我 `XP` 没有 `flash` 控件会非法操作的，按上述方式玩玩吧。

明白了，你就会玩，不明我也没办法。努力呀，大概在 20 来个 `CC`，有些破解了，就根本不用修它，因为根本不会再走到那里！

你能从头看到了这行，说明你很有耐心，很适合这份干工作... 你将会得到的奖品是满心喜欢，`Good Luck`!

Written by `askformore`![](img/d8960f6c7afa20ba0ca27920887f4540.jpg) 

`4dd466d0e1e0cc123afbe504da2462ca`
