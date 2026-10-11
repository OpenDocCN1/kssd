
# Armadillo4.40.CUSTOM.最大保护下脱壳教程

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-526.htm`](https://www.pediy.com/kssd/pediy08/pediy8-526.htm)

**译者说:**

1. 声明:我翻译的所有文章仅限于翻译,没有其它目的.
   一切其它用途及目的造成的后果由使用者自负.
2. 来源:`http://www.exetools.com/forum/showthread.php?t=8919`
3. 作者:`DappA`

![](img/fb3a632d9a866cf9dff0a67c81deb890.jpg)

**基本信息:**

*   壳:`Armadillo 4.40 CopyMem-II, IAT Elimination, Strategic Code Splicing`
*   MemoryPatch protection, Custom Build
*   目标:`dUP2.10 patcher`
*   工具:`OllyDbg1.10, HideOD, PUPE,` 一些需要的 `ollyscripts`
*   难度:`6/10`

**扫描目标:**

最新 `PEID` 可以扫描出最新的 `Armadillo`,扫描后得到:

![](img/49a6d8a05241a604db17e092ccda897f.jpg)

### 设置"异常"

打开程序前,我们需要进一步设置"异常"

![](img/88e589e9ce8dc83caaf92796f821cb53.jpg)

### 一.解决 CopyMemII 和 DebugBlocker

进入正题.

这部分有些枯燥,如果想跳过,可以使用 `DetachFarther_MethodTenketsu_hipu_benina.txt`.

我将会解释怎样手动解决.我们进入:

![](img/2c8c07dad87c3c20c02019f4a4d9c8e1.jpg)

现在 `bp WaitForDebugEvent`

![](img/80bc8fa5f5e33237897d6fb50cd4801c.jpg)

由于内存补丁保护是开启的,`Armadillo` 将会检测我们的断点是否下在 API 的头 5 个字节上.

因此我们需要把断点移向 API 5 个字节后或使用内存断点.我使用后者.

![](img/cc3f0a147bad5b0515b3c937d1c93d2c.jpg)

我不会每次都用图,而会用文字说明代替.

断点下好后,运行一下.经过一些非 API 的地方后,我们来到:

![](img/8d443b31fd4eec0f41ff45ba4921ee59.jpg)

保存以下信息:

```text
0012DC64 00431D86 /CALL to WaitForDebugEvent from test.00431D80
0012DC68 0012ED54 |pDebugEvent = 0012ED54
0012DC6C 000003E8 \Timeout = 1000. ms
```

`CTRL+F2` 重起程序,在 `WriteProcessMemory` 上下新的断点.`SHIFT+F9` 运行直到看到 1000 字节转换为止.

![](img/f2754ebf5eab60e2077058e08c67fa81.jpg)

注意 `BytesToWrite==1000`.这里是我们首个 `CopyMem crypttransfer`.也保存一下

```text
0012DB04 00435D57 /CALL to WriteProcessMemory from test.00435D51
0012DB08 00000048 |hProcess = 00000048 (window)
0012DB0C 00401000 |Address = 401000
0012DB10 003A3988 |Buffer = 003A3988
0012DB14 00001000 |BytesToWrite = 1000 (4096.)
0012DB18 0012DC20 \pBytesWritten = 0012DC20
```

在同一个窗口中,下滚知道发现主程序返回.你还会发现有个小循环:

![](img/83c2167f78afab2c4e1f4aae9da3ed84.jpg)

我把它粘贴到这里:`0012DC2C |004349C5 RETURN to test.004349C5 from test.00434D0D`

这部分很重要,它将会带领我们到达用来控制 `CopyMem` 函数的 `cryptocall` 中.

![](img/f439780629ae92bf811dc36b701dcfd7.jpg)

`CTRL+G` 前往表达.你会来到 `PUSH EBP` 指令.选中它,`CTRL+R` 得到所有关于此指令的参考.

![](img/ea1f731b84a8888bec06c85dc409508d.jpg)

双击第二个 `CALL`,你会来到此 `CALL`.`NOP` 掉它.

![](img/274d0c19bb7f3de0aea7b7c0f4a5a949.jpg)

很好.现在你将去找子进程的 `OEP`.`CTRL+F9,F8` 退出 `WriteProcessMemory`,`DUMP` 窗口中 `CTRL+G`,键入 `pDebugEvent` 地址:`0012ED54`

![](img/eb3e0698206672e068039f7f4a8bed3a.jpg)

第一行,子进程 `OEP` 为 `00401FAA`.我们稍后会需要这三个地址.

```text
0012ED6C 00401FAA test.00401FAA (OEP)
0012ED70 00000002
0012ED74 00000000
0012ED78 00401FAA test.00401FAA
0012ED7C 00401FAA test.00401FAA
```

#### 修改子进程为无限循环

现在我们将修改 `OEP` 处的子进程为无限循环.打开 `PUPE`,打开程序(记得使用正确的进程),键入 `OEP`.

![](img/db90c3c082eb36a17bd7175708ba2a2e.jpg)

记下原字节:`8D 9B`

修改它.

关闭 `PUPE`.回到 `Olly`,在 `WaitForDebugEvent` 上设置内存断点.中断后,前往正确的 `dump` 节并"反汇编跟随":`0012DC64 00431D86 /CALL to WaitForDebugEvent from test.00431D80`

在 CPU 窗口中 `NOP` 掉 `WaitForDebugEvent`,在"`TEST EAX,EAX`"选上择"新的起源":

![](img/809da935ba06582f6a5e70a79139e49e.jpg)

现在你将会跳到 `.text` 节,你可以写下自己的代码:

![](img/caee13da146c4d449e6ff2b48b42749c.jpg)

走至 `JMP`,回车.这就是我们将会用一些代码处理 `CopyMemII` 和 `DebugBlocker` 的地方.

添加以下内容:

```text
0040103C 8105 6CED1200 00100000 ADD DWORD PTR DS:[12ED6C],1000
00401046 8105 78ED1200 00100000 ADD DWORD PTR DS:[12ED78],1000
00401050 8105 7CED1200 00100000 ADD DWORD PTR DS:[12ED7C],1000
0040105A 813D 7CED1200 00104000 CMP DWORD PTR DS:[12ED7C],00401000
00401064 ^ 74 D6 JE SHORT 00401000
00401066 813D 7CED1200 00304000 CMP DWORD PTR DS:[12ED7C],00403000
00401070 - 0F85 150D0300 JNZ 00431D8D
00401076 68 400B0000 PUSH 0B40 (PID!)
0040107B E8 1193457C CALL DebugActiveProcessStop
00401080 90 NOP
```

我来解释一下.记得先前的 3 行吗?

```text
0012ED6C 00401FAA test.00401FAA
0012ED70 00000002
0012ED74 00000000
0012ED78 00401FAA test.00401FAA
0012ED7C 00401FAA test.00401FAA
```

因此我们添加了 1000 字节到这三个地址,接着用最后的地址与 `.text` 节的 `401000` 进行比较.相等则重新执行直到完成.

接下来比较 `.rdata` 节.非 0 则跳回"`JMP 401000`"下的 `NOP`.

首先修正指针:

![](img/7c477af8545d78be0f8a82b981378621.jpg)

在最后的 `NOP` 上设置断点,`SHIFT+F9` 运行:

![](img/f5fcc77e98245dbc72ac51564c1cd91c.jpg)

现在添加子进程,运行,暂停(`SHIFT+F9,F12`).把无限循环改回原代码:`8D 9B`

![](img/6791f78aa7890f405e4440ded8a90659.jpg)

祝贺你,你现在得到了"干净"的 `OEP`.我们解决了 `CopyMemII` 和 `DebugBlocker`.不要关闭.下面我们解决 `IAT` 重定向,`IAT Elimination` 和 `Code Splicing`.

### 二.输入表重定向和 Elimination+Spliced Code

现在探讨重定向部分.

`Armadillo` 的 `IAT` 重定向有一张重定向表,它被用来抽取原代码,以自己的方式来处理.

我们的目标就是阻止 `Armadillo` 重定向任何 `API`.

首先,我们需要找一个当前进程的 `API`.进入第一个 `CALL`,查找 `API`.任选一个.我选择了以下:

`00402AAE - FF25 980BD800 JMP DWORD PTR DS:[D80B98] ; GDI32.SetTextColor`

`DUMP` 窗口中跟随地址.修改"查看"为"长型地址".你可以看到许多 `API`,上滚至 `IAT` 表头:

![](img/faf23eddf4283be211c4ac615b4aa2c7.jpg)

`00000000` 表示开始.现在下滚至:

![](img/a6aca10dc58b814ba617ceaed5f12603.jpg)

现在我们打开新的 `Armadillo` 程序.我们不需要干净的代码,只要干净的 `IAT` 就可以了.我们将修正 `IAT` 重定向.

运行,暂停,修改字节.就这样做.

`SHIFT+F9` 运行,直到 `Privileged Instruction` 异常停止.

`bp VirtualProtect`,运行,中断.`CTRL+F9`,退出 `API`.你到达了 `Armadillo` 代码中.

`CTRL+F` 查找命令 `PUSH 100`.来到 `PUSH EBP`.修改为 `RET`.

![](img/0577a072b135fb4c5855a6ea625ee059.jpg)

搞定 `IAT` 重定向.运行直到 `OEP`.很简单.

`bp CreateThread`,`SHIFT+F9`,`CTRL+F9`,`F8`,下滚直到 `CALL ECX`,`bp`,`F7` 进入.

到达 `OEP`.但我们需要的是干净的 `IAT`.得到先前 `API` 的 16 进制代码:

![](img/bc84540781fb0aa1f6ee91e03a26ab43.jpg)

`ALT+M` 打开内存镜像,查找二进制.记得选中头地址,查找整个镜像.

![](img/0acc14cc0c2b9bb934da94cd43ef0ccc.jpg)

这就是完成的过程.现在我们要做的就是复制整个新的 `IAT` 到已损坏的 `IAT` 中.

![](img/04b677116f1b0a18028dc06e18b650d6.jpg)

回到 `OEP`,你会看见 `GetCommandLineA`, 之前是没有的,因为壳把它重定向到自己的代码中了.

现在我们用 `ArmInline` 修正 `spliced code` 和 `IAT elimination`.很简单.我们先找到 `spliced code`.进入修正的 `IAT` 进程的第二个 `CALL` 中.发现 `00402BF9 - E9 12E4C001 JMP 02011010`

它前往高内存地址,因此它就是 `spliced code`.

`ALT+M` 打开内存镜像,查找分配的部分:

![](img/2539ef28f9bcc153fe07739d5652592c.jpg)

`spliced` 部分往往在分配地址的底部.在每次重起程序时,分配地址是任意的,这是因为壳通过经时间 `API` 的任意函数分配地址.打开 `ArmInline`,填入以下数据:

![](img/ae3f96dc81703c6b05592dc3ff4d32e2.jpg)

目标代码开始于 `.text` 节,长度为 `.text` 节的长度.选择"remove Splices".解决!

下面是 `IAT Elimination`:

![](img/e28c4624975589512f284c84395e9b0a.jpg)

`00D809CC - IAT START`
`00D80D20 - IAT END`
`IAT SIZE: 354`

新的 `RVA` 基地址在 `.adata` 节中,分配给 `00455000`.

我们现在用 `LordPE` 来 `dump` 目标程序.完成后,打开 `ImportREC` 重建 `IAT`.使用当前的 `OEP`,`IAT` 大小前面有,`"cut thunk(s)"`,添加 `IAT` 到 `dumped.exe` 中.试试.正确运行!

![](img/5bd0c9553032aa22c519b6f5c567f892.jpg)

致 `DappA`

感谢:`Hacnho` 的精彩教程和视频,向这些不错的工具,`Tenketsu` 和 `Ricardo` 写的一些 `Ollystripts` 致敬.

鸣谢:`ICU, TSRH, SND, ARTEAM` 和 `MP2K[U]`

`4dd466d0e1e0cc123afbe504da2462ca`
