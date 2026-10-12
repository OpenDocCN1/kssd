
# [原创] armadillo 中 code splicing 的几种处理方法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-684.htm`](https://www.pediy.com/kssd/pediy07/pediy7-684.htm)

Anti Tracks 5.9.8

下载页面：`http://www.download.com/3120-20_4-0.html?qt=AntiTracks&tg=dl-20&search.x=17&search.y=12`

Erase your Internet tracks, computer activities, and history. Read full review...

OS: Windows (all)

License: Free to try, $29.95 to buy

File Size: 3.12MB

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教

【调试环境】：`WinXP`、`OllyDbg`、`PEiD`、`LordPE`, `arminline0.7`

—————————————————————————————————

【脱壳过程】：

小弟脱这个壳，主要是学习处理 `code splicing`，参考了很多大侠的文章，总结出三种方法：

这个程序别人已经脱过壳，前面的部分内容基本上是他的翻版，希望他不要介意。

`od` 载入主程序 停在这里：

```
00540323  A>  55      push ebp
00540324    8BEC    mov ebp,esp
00540326    6A FF   push -1
00540328    68 20AB>push AntiTrac.0056AB20
0054032D    68 6000>push AntiTrac.00540060
00540332    64:A1 0>mov eax,dword ptr fs:[0]
00540338    50      push eax
```

下断点：`BP GetModuleHandleA+5`

不断按 `f9`

第一次：

```
0012FF38  005403EB  返回到 AntiTrac.005403EB 来自 kernel32.GetModuleHandleA
0012FF3C  00000000
0012FF40  00000000
0012FF44  00141F29
```

第二次：

```
0012CED8  77C059FC  返回到 msvcrt.77C059FC 来自 kernel32.GetModuleHandleA
0012CEDC  77BE31AC  ASCII "kernel32.dll"
0012CEE0  77C2CA20  msvcrt.77C2CA20
```

。。。。。。

。。。。。。

第七次：

```
00126884  00C835E7  返回到 00C835E7 来自 kernel32.GetModuleHandleA
00126888  00C97474  ASCII "kernel32.dll"
0012688C  00C98744  ASCII "VirtualAlloc"
00126890  00000001
00126894  00DF00D0
```

第八次：

```
00126884  00C83604  返回到 00C83604 来自 kernel32.GetModuleHandleA
00126888  00C97474  ASCII "kernel32.dll"
0012688C  00C98738  ASCII "VirtualFree"
00126890  00000001
```

第九次：（`shift+f9`）

```
001265F4  00C6ABA5  返回到 00C6ABA5 来自 kernel32.GetModuleHandleA
001265F8  00126738  ASCII "kernel32.dll"
001265FC  00000000
00126600  CCE00000
```

到返回时机了，去掉断点，`alt+f9` 返回

```
00C6ABAE    A1 E4C9>mov eax,dword ptr ds:[C9C9E4]
00C6ABB3    391C06  cmp dword ptr ds:[esi+eax],ebx
00C6ABB6    75 16   jnz short 00C6ABCE
00C6ABB8    8D85 B4>lea eax,dword ptr ss:[ebp-14C]
00C6ABBE    50      push eax
00C6ABBF    FF15 E0>call dword ptr ds:[C920E0]        ; kernel32.LoadLibraryA
00C6ABC5    8B0D E4>mov ecx,dword ptr ds:[C9C9E4]
00C6ABCB    89040E  mov dword ptr ds:[esi+ecx],eax
00C6ABCE    A1 E4C9>mov eax,dword ptr ds:[C9C9E4]
00C6ABD3    391C06  cmp dword ptr ds:[esi+eax],ebx
00C6ABD6    0F84 31>je 00C6AD0D          ************magic jmp
00C6ABDC    33C9    xor ecx,ecx
00C6ABDE    8B07    mov eax,dword ptr ds:[edi]
00C6ABE0    3918    cmp dword ptr ds:[eax],ebx
```

把 `je 00C6AD0D` 改成 `jmp 00C6AD0D`。

下断点 `he SetProcessWorkingSetSize`，`f9`，这里会停几秒钟 到这里

```
77E5346F k>  55      push ebp
77E53470    8BEC    mov ebp,esp
77E53472    83EC 20 sub esp,20
77E53475    8B4D 0C mov ecx,dword ptr ss:[ebp+C]
77E53478    85C9    test ecx,ecx
77E5347A    53      push ebx
```

先不取消断点，再下断点： `he GetCurrentThreadId` 到这里

```
77E5A7DF ker>  64:A1 1>mov eax,dword ptr fs:[18]
77E5A7E5     8B40 24 mov eax,dword ptr ds:[eax+24]
77E5A7E8     C3      retn
77E5A7E9 ker>  8B4C24 >mov ecx,dword ptr ss:[esp+4]
77E5A7ED     8B5424 >mov edx,dword ptr ss:[esp+8]
```

堆栈如下：

```
0012D59C  00C8C34B  /CALL 到 GetCurrentThreadId 来自 00C8C345
0012D5A0  0012FF2C
0012D5A4  00000000
0012D5A8  7FFDF000
0012D5AC  40000060  ASCII "t be run under Win32
$7"
0012D5B0  77F5166A  返回到 ntdll.77F5166A 来自 ntdll.77F78C4E
```

取消上面两个硬件断点：

`alt+f9` 到这里

```
00C8C34B     A3 0C20>mov dword ptr ds:[CA200C],eax
00C8C350     E8 E5EB>call 00C6AF3A
00C8C355     53      push ebx
00C8C356     E8 F54C>call 00C71050
00C8C35B     A1 F81E>mov eax,dword ptr ds:[CA1EF8]
00C8C360     59      pop ecx
00C8C361     8B48 68 mov ecx,dword ptr ds:[eax+68]
00C8C364     3348 04 xor ecx,dword ptr ds:[eax+4]
00C8C367     3308    xor ecx,dword ptr ds:[eax]
00C8C369     F6C1 40 test cl,40
00C8C36C     75 08   jnz short 00C8C376
00C8C36E     6A 01   push 1
00C8C370     E8 17B9>call 00C67C8C
00C8C375     59      pop ecx
00C8C376     53      push ebx
00C8C377     C705 E0>mov dword ptr ds:[C979E0],0C987C0   ; ASCII "RC"
00C8C381     E8 6AE5>call 00C6A8F0
00C8C386     59      pop ecx
```

（有人把 `ASCII "RC"` 看着快到入口标志）

`f8` 单步执行：

到 `call edi` 这里后 `f7` 就到达入口 `oep` 了

```
00401000   /EB 10   jmp short AntiTrac.00401012
00401002   |66:623A bound di,dword ptr ds:[edx]
00401005   |43      inc ebx
00401006   |2B2B    sub ebp,dword ptr ds:[ebx]
00401008   |48      dec eax
00401009   |4F      dec edi
0040100A   |4F      dec edi
0040100B   |4B      dec ebx
0040100C   |90      nop
0040100D  -|E9 B8B6>jmp 0085C6CA
00401012   \A1 ABB6>mov eax,dword ptr ds:[45B6AB]
00401017    C1E0 02 shl eax,2
0040101A    A3 AFB6>mov dword ptr ds:[45B6AF],eax
0040101F    52      push edx
```

现在对付 `code splicing`：

查找 `antitracks` 的进程为 `468`

`alt + m` 打开内存窗口

内存映射，项目 `23`

```
地址=00401000
大小=0005A000 (368640.)
物主=AntiTrac 00400000
区段=.text
类型=Imag 01001002
访问=R
初始访问=RWE
```

然后点访问，按访问排序：

第一个类型为 `priv` 访问=`RWE` 的项目如下：

内存映射，项目 `317`

```
地址=02C10000
大小=00020000 (131072.)
物主=        02C10000 （自身）
区段=
类型=Priv 00021040
访问=RWE
初始访问=RWE
```

(这个地址不是固定的)

打开 `arminline0.7`，依次填入：

`468`，`401000`，`5a000`

`02c1000`，`20000`

点 `remove splicing`

结果出现：

```
process memory buffered successfully
0 splices repaired
no splicing found
```

奇怪，怎么没找到？

这个地址应该没错呀，我按另一种方法找也是这样的地址

后来我试了几次，终于成功了,我不知道是什么原因？脱壳的步骤都是一样的

提示成功：

```
2293splics repaired
patch successful
```

然后用 `lordpe` `dump` 出来，修复 `iat` （具体步骤下面有），成功运行

### `code splicing` 处理方法二：

下面说另一种对付 `code` 的办法，参照，

`Armadillo 4.xx- Code Splicing+Import Elimination`

Target: `GameJack ?v5.0.3.3`

好象是德文写的，我看不懂，只是看他的图。

重新打开 `od` 前面的步骤和上面的一样，一直到修改魔法跳转，前面的步骤都是一样的

修改魔法跳转后，下断点 `he VirtualAlloc`，`f9`

```
77E5AC72 ker>  55      push ebp
77E5AC73     8BEC    mov ebp,esp
77E5AC75     FF75 14 push dword ptr ss:[ebp+14]
77E5AC78     FF75 10 push dword ptr ss:[ebp+10]
77E5AC7B     FF75 0C push dword ptr ss:[ebp+C]
77E5AC7E     FF75 08 push dword ptr ss:[ebp+8]
```

堆栈

```
00125ED8  73391D07  /CALL 到 VirtualAlloc 来自 73391D05
00125EDC  00000000  |Address = NULL
00125EE0  00400000  |Size = 400000 (4194304.)
00125EE4  00002000  |AllocationType = MEM_RESERVE
00125EE8  00000004  \Protect = PAGE_READWRITE
00125EEC  00000000
00125EF0  00000001
```

`alt+f9` 返回后：

```
73391D07    8BF0    mov esi,eax
73391D09    85F6    test esi,esi
73391D0B    0F84 C5>je 733D8DD6
73391D11    6A 04   push 4
73391D13    68 0010>push 1000
73391D18    68 0000>push 10000                  ; UNICODE "=::=::\"
73391D1D    56      push esi
73391D1E    FFD7    call edi
73391D20    85C0    test eax,eax
```

再 `f9`，`alt+F9` 返回：

```
73391D20    85C0    test eax,eax
73391D22    0F84 A0>je 733D8DC8
73391D28    81FD 30>cmp ebp,7349B030
73391D2E    0F85 74>jnz 733D8DA8
73391D34    A1 30B0>mov eax,dword ptr ds:[7349B030]
73391D39    85C0    test eax,eax
73391D3B    0F84 49>je 733D8D8A
73391D41    A1 34B0>mov eax,dword ptr ds:[7349B034]
73391D46    85C0    test eax,eax
73391D48    0F84 4B>je 733D8D99
73391D4E    8D86 00>lea eax,dword ptr ds:[esi+400000]
```

再 `f9`，`alt+f9` 返回

```
00C86971    8985 58>mov dword ptr ss:[ebp-39A8],eax
00C86977    83BD 58>cmp dword ptr ss:[ebp-39A8],0
00C8697E    74 64   je short 00C869E4
00C86980    6A 40   push 40
00C86982    68 0010>push 1000
00C86987    FFB5 50>push dword ptr ss:[ebp-39B0]
00C8698D    FF35 9C>push dword ptr ds:[CA209C]
```



`00C86993`       `FF15` `C0>` `call dword ptr ds:[C921C0]`          ; `kernel32.VirtualAlloc`

`00C86999`       `8985` `58>` `mov dword ptr ss:[ebp-39A8],eax`

`00C8699F`       `83BD` `58>` `cmp dword ptr ss:[ebp-39A8],0`

现在按 `f8`，通过 `Function` `VirtuallAlloc`

到达这里

`00C86999`       `8985` `58>` `mov dword ptr ss:[ebp-39A8],eax`

（到 `00C86993` `f8` 到系统空间 去了   然后 `alt+f9` 就到 `00C86999` 了 ）

看到寄存器中 `eax=02980000`

（the value of `EAX` contain the Code to destroy our `IAT`.

 We must change this value to a real memory in the section of `arm`.

 Now, `alt+M` go to memory window）

`alt+m`，打开内存窗口

内存映射，项目 `31`

 地址=`00555000`

 大小=`00010000` (`65536.`)

 物主=`AntiTrac` `00400000`

 区段=`.adata`

 类型=`Imag` `01001002`

 访问=`R`

 初始访问=`RWE`

把寄存器窗口中的 `eax` 改为 `00555000`

we choose address `555000` to save the Code. Go to `FPU`（寄存器） Window! 

Change `EAX` to `555000`

删除断点 `hd` `VirtualAlloc`，现在 `code splicing` 处理完毕

下断点 `he` `SetProcessWorkingSetSize`. `F9`

大约 `10` 秒钟后停在这里：

`77E5346F` `ker>`  `55`      `push ebp`
`77E53470`       `8BEC`    `mov ebp,esp`
`77E53472`       `83EC` `20` `sub esp,20`
`77E53475`       `8B4D` `0C` `mov ecx,dword ptr ss:[ebp+C]`
`77E53478`       `85C9`    `test ecx,ecx`
`77E5347A`       `53`      `push ebx`
`77E5347B`       `56`      `push esi`
`77E5347C`       `74` `56`   `je short kernel32.77E534D4`
`77E5347E`       `8B45` `10` `mov eax,dword ptr ss:[ebp+10]`

先不删除断点，再下断点 `he` `GetCurrentThreadId` ，`f9`

到达这里：

`77E5A7DF` `ker>`  `64:A1` `1>` `mov eax,dword ptr fs:[18]`
`77E5A7E5`       `8B40` `24` `mov eax,dword ptr ds:[eax+24]`
`77E5A7E8`       `C3`      `retn`
`77E5A7E9` `ker>`  `8B4C24` `>` `mov ecx,dword ptr ss:[esp+4]`
`77E5A7ED`       `8B5424` `>` `mov edx,dword ptr ss:[esp+8]`
`77E5A7F1`       `8B4424` `>` `mov eax,dword ptr ss:[esp+C]`

堆栈如下：

`0012D59C`   `00C8C34B`  `/CALL` 到 `GetCurrentThreadId` 来自 `00C8C345`
`0012D5A0`   `0012FF2C`
`0012D5A4`   `00000000`

删除上面的两个断点：

`alt+f9` 返回 到这里：

`00C8C34B`     `A3` `0C20>` `mov dword ptr ds:[CA200C],eax`
`00C8C350`     `E8` `E5EB>` `call 00C6AF3A`
`00C8C355`     `53`      `push ebx`
`00C8C356`     `E8` `F54C>` `call 00C71050`
`00C8C35B`     `A1` `F81E>` `mov eax,dword ptr ds:[CA1EF8]`
`00C8C360`     `59`      `pop ecx`
`00C8C361`     `8B48` `68` `mov ecx,dword ptr ds:[eax+68]`
`00C8C364`     `3348` `04` `xor ecx,dword ptr ds:[eax+4]`
`00C8C367`     `3308`    `xor ecx,dword ptr ds:[eax]`
`00C8C369`     `F6C1` `40` `test cl,40`
`00C8C36C`     `75` `08`   `jnz short 00C8C376`
`00C8C36E`     `6A` `01`   `push 1`
`00C8C370`     `E8` `17B9>` `call 00C67C8C`
`00C8C375`     `59`      `pop ecx`
`00C8C376`     `53`      `push ebx`
`00C8C377`     `C705` `E0>` `mov dword ptr ds:[C979E0],0C987C0`   ; `ASCII` `"RC"`
`00C8C381`     `E8` `6AE5>` `call 00C6A8F0`
`00C8C386`     `59`      `pop ecx`

（有人把 `ASCII` `"RC"` 看着快到入口标志）

`f8` 单步执行：

`00C8C3D1`     `83F9` `01` `cmp ecx,1`
`00C8C3D4`     `75` `21`   `jnz short 00C8C3F7`
`00C8C3D6`     `FF76` `04` `push dword ptr ds:[esi+4]`
`00C8C3D9`     `FF76` `08` `push dword ptr ds:[esi+8]`
`00C8C3DC`     `53`      `push ebx`
`00C8C3DD`     `E8` `74E2>` `call 00C7A656`
`00C8C3E2`     `50`      `push eax`
`00C8C3E3`     `A1` `F81E>` `mov eax,dword ptr ds:[CA1EF8]`
`00C8C3E8`     `8B48` `58` `mov ecx,dword ptr ds:[eax+58]`
`00C8C3EB`     `3348` `30` `xor ecx,dword ptr ds:[eax+30]`
`00C8C3EE`     `3308`    `xor ecx,dword ptr ds:[eax]`
`00C8C3F0`     `2BF9`    `sub edi,ecx`
`00C8C3F2`     `FFD7`    `call edi`          `****f7`  
`00C8C3F4`     `8945` `FC` `mov dword ptr ss:[ebp-4],eax`

看到这个 `call edi` 没有，执行到这里后  ，`f7` 就到 `oep` 了

`00401000`    `/EB` `10`   `jmp short AntiTrac.00401012`
`00401002`    `|66:623A` `bound di,dword ptr ds:[edx]`
`00401005`    `|43`      `inc ebx`
`00401006`    `|2B2B`    `sub ebp,dword ptr ds:[ebx]`
`00401008`    `|48`      `dec eax`
`00401009`    `|4F`      `dec edi`
`0040100A`    `|4F`      `dec edi`
`0040100B`    `|4B`      `dec ebx`
`0040100C`    `|90`      `nop`
`0040100D`   `-|E9` `B8B6>` `jmp 0085C6CA`
`00401012`    `\\A1` `ABB6>` `mov eax,dword ptr ds:[45B6AB]`
`00401017`     `C1E0` `02` `shl eax,2`
`0040101A`     `A3` `AFB6>` `mov dword ptr ds:[45B6AF],eax`
`0040101F`     `52`      `push edx`

现在打开 `loadpe`  选择 `antitracks`  `dump`

再打开 `importrec1.6`   选择 `antitracks` 

填 `oep=1000`，点自动搜索 `iat`  下面出现 `arav=84bd8`  `size=bc`

`getimport`  居然只有 `46` 个指针 只有 `kernel32.dll` 一个 `api` 

`fixdump`  无法运行。

在 `od` 命令行中输入 `dd 00484bd8`，向上看 还有很多呢！！！

```
004837F0  00085DC5
004837F4  00085DE3
004837F8  00000000
004837FC  4000A314  VCL50.System::initialization      iat 的开始
00483800  4000A2AC  VCL50.System::Finalization
00483804  40008AE4  VCL50.System::TInterfacedObject::_Release
00483808  40008AD0  VCL50.System::TInterfacedObject::_AddRef
0048380C  40008AA8  VCL50.System::TInterfacedObject::QueryInterface
00483810  40008A98  VCL50.System::TInterfacedObject::NewInstance
00483814  40008A88  VCL50.System::TInterfacedObject::BeforeDestruction
。。。
```

向下看：

```
00484FC4  00C6BF3F
00484FC8  0008B473
00484FCC  0008B489
00484FD0  0008B49D
00484FD4  0008B4B7
00484FD8  00000000
00484FDC  7619AA55  WININET.DeleteUrlCacheEntryA
00484FE0  7618DA28  WININET.FindCloseUrlCache
00484FE4  7618F63D  WININET.FindFirstUrlCacheEntryA
00484FE8  7618F6C6  WININET.FindNextUrlCacheEntryA               iat 的结束
00484FEC  00C6BF44
00484FF0  354C4356
00484FF4  50422E30
00484FF8  4356004C
```

`iat` 的长度=`00484fe8`-`004837FC`=`17ec`

重新填 `rav=00084fe8`  `size=17ec`   

点 `getimport` ，`cut` 无效函数，剩下 `714` 个指针，`fixdump`  成功运行

把刚才输入表保存起来为 `import714.txt`

显示为注册版，脱壳前只能试用 `15` 天

`code splicing` 处理方法三：

参见：`Armadillo` `V4.0` 输入表乱序的简便修复方法——`PIMOne` `V2.3` （这个是双进程）

开始过程和我刚才前面的一样，到达 `oep` 后，`dump`，存为 `1.exe`

打开内存影射

然后点访问，按访问排序：

第一个类型为 `priv` 访问=`RWE` 的项目如下：

内存映射，项目 `317`

 地址=`01AC0000`

 大小=`00020000` (`131072.`)

 物主=        `02C10000` （自身）

 区段=

 类型=`Priv` `00021040`

 访问=`RWE`

 初始访问=`RWE`

(这个地址不是固定的)

记下这个地址，打开 `lordpe`，找到目标进程，选择区域脱壳（第三个选项），找到地址=`01AC0000`

然后把这部分 `dump` 出来，为 `Region01AC0000-01AE0000.dmp`

打开 `importrec`，找到目标进程，`load tree`，导入上面保存的 `iat`，`fixdump`，选择 `1.exe`，无法运行。

为什么？因为 `code splices` 作怪。

用 `lordpe`--点 `pe editor`，打开 `1.exe`，点 `sections`，右键，第三个选项，从磁盘导入 `section`，选择

刚才的 `Region01AC0000-01AE0000.dmp`，导入后，右键，选 `edit section header` 

修改 `virtual address` 为 `01AC0000`-`00400000`=`016C0000`,然后设置 `lordpe` 的选项，把 `rebuilder` 里面

的选项全部去掉，只保留 `LordPE` 的“`Validate PE`”选项，重建 `PE`。

然后用 `importrec` 修复输入表，成功运行。

修复输入表和重建 `PE` 的顺序可以交换。

`4dd466d0e1e0cc123afbe504da2462ca`
