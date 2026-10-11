
# UltraEdit 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6439.htm`](https://www.pediy.com/kssd/pediy06/pediy6439.htm)

实战 `Armadillo3.60` `Original CopyMem-ll +Debug-Blocker` ----- `UltraEdit`   

大家好，上次我已写了 `Armadillo3.60` 的标准壳的脱壳方法，相信大家一定会了吧。呵呵，小妹今天要带领大家一起来脱掉 `Original CopyMem-ll +Debug-Blocker` 的方式壳。不要害怕，一起来吧。   

在这里，我首选感谢以前写过 `AM` 教程的人，虽然都是 3.0 之类的，都这些都是基础啊，特别是 `FTBirthday` `FLY` 等人，尤其是 `FTBirthday` 翻译的 `Ricardo Narvaja` 的文章，可谓非常经典，小妹用的方法也是参照他的方法，我试图找到一种不一样的方法，但都失败了，因为这种方式的加壳实在是太难了，当然他的有些做法在 3.6 中已不适用，但我都改变了一下，并且在修复时，我使用了自己的一些做法，这样可以更快地完成，并且更简单，更保险，一句话，为广大 `CRACKER` 着想，确保大家都能学会。呵呵。好的，我们的教程马上要开始了，这次小妹仍然使用本本来调试，呵呵，我把本本放在被子上，自己躺在床上调试，因为，我们面对的是一个复杂的壳，需要一个好的环境，旁边放上一杯果真，OK，让我们开始吧。   

调试环境：本本迅弛 1.4G 256MDDR 1M 的 `L2` 二级缓存，调试起来非常快。如果你的机上有问题请和我联系，因为本本调试可能会有问题。还有请在 `XP` 以上系统下调试，因为其中用到的 `DebugActiveProcessStop` 只有 `XP` 以上系统才有，   

在教程最后，我将总结一些经验给大家。   

小妹不想在开头讲 `AM` 的一些理论知识，我想放在中间分开来讲解比较好，毕竟这是一门有别于其它的学科，呵呵，我们开始了。   

这次我脱壳的对象是 `Ultraedit.exe`  10.00   

首选我用 `Armadillo3.60` 专业版为 `Ultraedit.exe` 加上 `Original CopyMem-ll +Debug-Blocker` 的壳。为了方便，我去掉了 `AM` 提醒框，这会稍微增加一点难度，后面小妹会提到的。   

后来，打开 `OD1.1b` 载入它，忽略所有异常事件。如下：   

```
005EB000 >  60                 PUSHAD

005EB001     E8 00000000      CALL Uedit32.005EB006

005EB006     5D                 POP EBP

005EB007     50                 PUSH EAX

005EB008     51                 PUSH ECX

005EB009     EB 0F             JMP SHORT Uedit32.005EB01A
```

说明一下，不能再用标准壳的脱壳方法来脱 `Original CopyMem-ll +Debug-Blocker` 的壳，并且断点也不一样，因为这是一种完全不一样的加壳方式，`armadillo` 自己生成 2 个进程，父进程做调试器，子进程做被调试者，父进程负责对子进程的代码进行解码，而对脱壳来说很重要的 `IAT` 部分却是子进程负责进行解密并 `hook`。`Ollydbg` 现在却没办法 `attach` 上子进程。   

载入之后，隐藏 `OD` 吧。这一步是必须的。   

好的，接着在命令行里下断 `BP WaitForDebugEvent` 下面我会讲解为什么要用这个函数来下断。我们先运行一下，让它断下来。按 `SHIFT＋F9` 两次忽略两次异常后断下，如下：这是堆栈窗口：   

```
0012DA98  005D0D67  CALL 到 WaitForDebugEvent 来自 Uedit32.005D0D61

0012DA9C    0012EB5C  |pDebugEvent = 0012EB5C

0012DAA0    000003E8  Timeout = 1000. ms

0012DAA4    0012FF04

0012DAA8    00000000

0012DAAC    005E0999  Uedit32.005E0999
```

上面的 `0012EB5C` 就是发生调试事件时具体的内容存放地址，每次调用 `WaitForDebugEvent` 时都会使用这个地址。因为 `AM` 的两个进程会用到这个函数，下面马上要讲到了。   

在 `0012EB5C` 上右击选择转存中跟随，在左下角内存窗口如下：   

留意这个窗口，一会儿我们的 `OEP` 将出现在这里面，呵呵，小妹一会儿告诉你为什么。   

清除掉上面的断点，在命令行下断 `Bp WriteProcessMemory`  （注意，类似这样的断点需要注意大小写）`F9` 一下，稍等一下，因为它要创建一个新线程，马上中断下来。观察堆栈窗口如下：   

```
0012D938 005D4C6E   /CALL 到 WriteProcessMemory 来自 Uedit32.005D4C68

0012D93C    0000004C   |hProcess = 0000004C (window)

0012D940    00504000   |Address = 504000

0012D944    003A7FB8   |Buffer = 003A7FB8

0012D948    00001000   |BytesToWrite = 1000 (4096.)

0012D94C    0012DA54   pBytesWritten = 0012DA54

0012D950    00000000

0012D954    00001030
```

好了，小妹在这里要讲解一些理论知识。   

父进程的一个缓冲区从 `003A7FB8` 开始，然后它会复制给子进程一个从 `504000` 开始的 `1000 bytes` 的块，很明显啊，子进程这时的第一个 `section` 是完全空的，所以当执行到 `OEP` 时，它当然要报告一个错误，父进程得到这个信息后会停止自身运行，而去复制另一个数据块，然后再自己运行，这样循环下去，被复制的数据块的大小是 `1000 bytes`，所以当程序试图执行任何超过这个大小的块外的指令时， 另一个错误将会发生，然后这个错误会被通知给父进程， 父进程会复制另外 `1000 bytes` 的块然后继续。大家都可以想到上面的第一个错误很有可能是因为它的子进程调用了 `OEP`，所以父进程会为了解决这个错误为复制 `1000bytes` 的数据块，呵呵，那么子进程的 `OEP` 就应该在里面了。确切地说 `OEP` 应该是在 `00504000＋1000－1＝00504FFF` 内。好的，我们看一下内存窗口：   

看见上面的三个 `005045AD` 了吗，这就是 `OEP`。   

```
0012EB5C  01 00 00 00     DebugEventCode 01 表示 EXCEPTION\_DEBUG\_EVENT

0012EB60   F0 06 00 00     ProcessId 发生调试事件的进程 id 

0012EB64  A8030000  ThreadId   发生调试时间的线程 id

0012EB68  01 00 00 00    因为 12eb5C 处为 1，所以这里代表的意思是 ExceptionCode
```

`WriteProcessMemory` 函数是内存复制函数，大家知道 `Armadillo` 的 `copymem-II` 方式会在内存中生成同名的 2 个进程，其中一个是父进程，一个是子进程。那么子进程是怎么生成的呢，就是从父进程中用 `WriteProcessMemory` 函数复制来的。（这二个进程的窗口句柄不同，并且互相掌握着对方的 `OEP`）`S` 因为复制是按 `1000` 的块进行的，所以第一个复制的块中必定包含了子进程的 `OEP` 地址。这个地址的值在第一块的值之间。所以上面的转存窗口中的那个 `005045AD` 就是真正的 `OEP` 了。   

虽然 `OEP` 找到了，但 `DUMP` 出来的还是加密的，因为还有一个加密 `call` 用来重新加密或者破坏已使用的块来避免被 `dump`，OK，我们来找到这个加密 `CALL` 吧。   

先清除掉上面的断点，以住的方法是查看调用堆栈，找有关信息，但这一方法在 3.6 中不再适用，怎么办呢，根据以往的经验，我找到了一个办法  ，可以正确并快速地找到这个地方。原理吗，就是 3.6 中也用了相关指令，我们可以通过这一途径来找到。OK，先返回到 `005D4C68` 这个调用 `WriteProcessMemory` 的下方，然后右击  查找所有命令：填入  `xor al,al`  回车，   

如下图：   

上图中的红色部分是当前指令  往上数 5 行，也就是现在鼠标停留在的 `005D38E8` 处，好双击那个地方如下：   

```
005D38D2    51               PUSH ECX

005D38D3    8B55 08         MOV EDX,DWORD PTR SS:[EBP+8]

005D38D6    52               PUSH EDX

005D38D7    E8 48030000     CALL Uedit32.005D3C24

005D38DC    83C4 0C         ADD ESP,0C

005D38DF    25 FF000000     AND EAX,0FF

005D38E4    85C0            TEST EAX,EAX

005D38E6    75 07           JNZ SHORT Uedit32.005D38EF

005D38E8    32C0            XOR AL,AL

005D38EA    E9 2E030000     JMP Uedit32.005D3C1D

005D38EF    50               PUSH EAX
```

看一下上面 `005D38D7` 有个 `CALL 005D3C24`  这就是一个用来解密的 `CALL`，对应的还有一个用来加密的以 `CALL 005D3C24` 右击  查找全部命令  填入：`CALL 005D3C24`  回车  OK，终于找到了另外一个地方也调用了这个 `CALL`：   

```
005D3BAC    E8 73000000     CALL Uedit32.005D3C24

005D3BB1    83C4 0C         ADD ESP,0C

005D3BB4    50               PUSH EAX

005D3BB5    F7D0            NOT EAX

005D3BB7    0FC8            BSWAP EAX

005D3BB9    58               POP EAX

005D3BBA    73 00           JNB SHORT Uedit32.005D3BBC
```

在 `005D3BAC` 处也调用了它，但在那里，那个 `CALL` 的作用是加密块，好的我们 `NOP` 它，空格，然后 `NOP`，现在我们要得到不加密的块，我们要执行 `WriteProcessMemory`，返回到刚才那个 `005D4C68` 处，   

```
0012D938 005D4C6E   /CALL 到 WriteProcessMemory 来自 Uedit32.005D4C68

0012D93C    0000004C   |hProcess = 0000004C (window)

0012D940    00504000   |Address = 504000

0012D944    003A7FB8   |Buffer = 003A7FB8

0012D948    00001000   |BytesToWrite = 1000 (4096.)

0012D94C    0012DA54   pBytesWritten = 0012DA54
```

快速返回的方法在上面堆栈窗口第一行上，右击选择反汇编跟随就可以了，   

```
005D4C68     FF15 64B15F00    CALL NEAR DWORD PTR DS:[<&KERNEL32.Write>; kernel32.WriteProcessMemory

005D4C6E     85C0            TEST EAX,EAX

005D4C70     75 4B           JNZ SHORT Uedit32.005D4CBD

005D4C72     50               PUSH EAX

005D4C73     F7D0            NOT EAX

005D4C75     0FC8            BSWAP EAX

005D4C77     58               POP EAX

005D4C78     73 00           JNB SHORT Uedit32.005D4C7A
```

在 `005D4C6E` 处下个断，`F9` 一下，会断下来。OK，清除断点，现在我们必须把父进程杀死，然后再调试子进程，首先我们要把子进程进入死循环，OK，拿出 `PUPE` 吧，最新版本是 2002，我从西班牙的一个站点上搞来的，运行 `PUPE`：   

可以看到有两个 `uedit32.exe` 进程，上面那个也就是句柄是 `6F0` 的那个，是子进程，我们在它上面右击，选择第一个命令，   

然后按上图的方式填好，`005045AD` 是子进程 `OEP`，`EBFE` 是让它死循环，`Buscar` 按钮是相当于回车键，在输入地上之后，在右边的灰色框中显示具体值是多少一，`Parchear` 是在下面的白色框中输入新值之后让它马上生效，`Salir` 是关闭。   

实就是用 `PUPE` 修改入口的代码，让他变成跳转到 `EIP` 地址的代码。这样程序在入口点就会停下。跳向自己的地址，程序会向下执行吗？   

因为父进程和子进程之间有调试关联，所以我们下断 `BP WaitForDebugEvent`  断下：   

```
0012DA98  005D0D67  /CALL 到 WaitForDebugEvent 来自 Uedit32.005D0D61

0012DA9C    0012EB5C  |pDebugEvent = 0012EB5C

0012DAA0    000003E8  Timeout = 1000. ms

0012DAA4    0012FF04

0012DAA8    00000000

0012DAAC    005E0999  Uedit32.005E0999

0012DAB0    00000001

0012DAB4    80000004
```

上面是堆栈窗口  ，好，小妹和大家一起到  `005D0D61`  处去吧。   

```
005D0D52    A0 336168E8     MOV AL,BYTE PTR DS:[E8686133] 这里看出是  3.6 程序

005D0D57    0300             ADD EAX,DWORD PTR DS:[EAX]

005D0D59    008B 95E0F5FF    ADD BYTE PTR DS:[EBX+FFF5E095],CL

005D0D5F    FF52 FF          CALL NEAR DWORD PTR DS:[EDX\-1]

005D0D62    15 A0B05F00      ADC EAX,<&KERNEL32.WaitForDebugEvent>

005D0D67    85C0            TEST EAX,EAX

005D0D69    0F84 9A260000    JE Uedit32.005D3409

005D0D6F    60               PUSHAD

005D0D70    33C0            XOR EAX,EAX

005D0D72    75 02           JNZ SHORT Uedit32.005D0D76

005D0D74    EB 15           JMP SHORT Uedit32.005D0D8B
```

我们返回到 `005D0D67` 处，好，现在我们要把父进程杀死，在这里，当然不能运行任何命令，所以我们在 `005D0D67` 上右击，选择此处新建 `EIP`，因为我们的目的是想让程序运行我们当前自己的命令，呵呵。   

然后，把 `005D0D52` 到 `005D0D62` 这里都 `NOP`，因为我们不想运行这个函数，不要让父进程再调试子进程。改好之后，如下：   

```
005D0D52    90               NOP

005D0D53    90               NOP

005D0D54    90               NOP

005D0D55    90               NOP

005D0D56    90               NOP

005D0D57    90               NOP

005D0D58    90               NOP

005D0D59    90               NOP

005D0D5A    90               NOP

005D0D5B    90               NOP

005D0D5C    90               NOP

005D0D5D    90               NOP

005D0D5E    90               NOP

005D0D5F    90               NOP

005D0D60    90               NOP

005D0D61    90               NOP
```



005D0D62` `90` `NOP`
005D0D63` `90` `NOP`
005D0D64` `90` `NOP`
005D0D65` `90` `NOP`
005D0D66` `90` `NOP`
005D0D67` `85C0` `TEST EAX,EAX`
005D0D69` `0F84 9A260000` `JE Uedit32.005D3409`
005D0D6F` `60` `PUSHAD`
005D0D70` `33C0` `XOR EAX,EAX`
005D0D72` `75 02` `JNZ SHORT Uedit32.005D0D76`

好，现在我们做杀死父进程的最后步骤——打上补丁。

把上面的 `005D0D69` 处的命令改成 `jmp 00401000`

因为父进程的偏移是 `00401000`，如下：

`005D0D67` `85C0` `TEST EAX,EAX`
`005D0D69` `-` `E9 9202E3FF` `JMP Uedit32.00401000`
`005D0D6E` `90` `NOP`

因为我们是在 `005D0D67` 建立了 `EIP`，所以，下面当然是 `jmp`，让程序跳到我们的补丁处去执行啊，呵呵，好，现在我们跳到 `00401000` 处去打补丁吧。

`00401000` `0000` `ADD BYTE PTR DS:[EAX],AL`
`00401002` `0000` `ADD BYTE PTR DS:[EAX],AL`
`00401004` `0000` `ADD BYTE PTR DS:[EAX],AL`
`00401006` `0000` `ADD BYTE PTR DS:[EAX],AL`
`00401008` `0000` `ADD BYTE PTR DS:[EAX],AL`
`0040100A` `0000` `ADD BYTE PTR DS:[EAX],AL`
`0040100C` `0000` `ADD BYTE PTR DS:[EAX],AL`
`0040100E` `0000` `ADD BYTE PTR DS:[EAX],AL`
`00401010` `0000` `ADD BYTE PTR DS:[EAX],AL`
`00401012` `0000` `ADD BYTE PTR DS:[EAX],AL`
`00401014` `0000` `ADD BYTE PTR DS:[EAX],AL`
`00401016` `0000` `ADD BYTE PTR DS:[EAX],AL`
`00401018` `0000` `ADD BYTE PTR DS:[EAX],AL`
`0040101A` `0000` `ADD BYTE PTR DS:[EAX],AL`

还是空白的，准备写命令进去。`Alt+M` 打开内存镜象窗口：

程序的 `text` 从 `00401000` 到 `0054CFFF`，我们需要修改 `OEP`，好的，来到内存窗口吧。

把上面的三个 `005045AD` 都改成 `00400000`

我们将要欺骗父进程，使他相信从第一个 `section` 开始的所有块都存在错误，一个接一个所有的块都会被正确的复制进来。

开始从 `00401000` 处打补丁。

`00401000` `8105 74EB1200 0>` `ADD DWORD PTR DS:[12EB74],1000`
`0040100A` `8105 80EB1200 0>` `ADD DWORD PTR DS:[12EB80],1000`
`00401014` `8105 80EB1200 0>` `ADD DWORD PTR DS:[12EB84],1000`
`0040101E` `813D 80EB1200 0>` `CMP DWORD PTR DS:[12EB84],Uedit32.0054D00>`
`00401028` `-` `0F85 40FD1C00` `JNZ Uedit32.005D0D6E`
`0040102E` `90` `NOP`
`0040102F` `90` `NOP`
`00401030` `0000` `ADD BYTE PTR DS:[EAX],AL`
`00401032` `0000` `ADD BYTE PTR DS:[EAX],AL`
`00401034` `0000` `ADD BYTE PTR DS:[EAX],AL`

按照上面的从 `00401000` 一直打到 `0040102F`

上面的 `jnz` 是比较是否解压块结束，后面的 `NOP` 是为了我们在那儿中断，在 `0040102E` 上下断，OK, `F9` 一下，被断下来了。大家看一下内存窗口：

可以看到数据被完全的复制过来了。好的，下面我们要把父进程杀死。

`00401000` `8105 74EB1200 0>` `ADD DWORD PTR DS:[12EB74],1000`
`0040100A` `8105 80EB1200 0>` `ADD DWORD PTR DS:[12EB80],1000`
`00401014` `8105 80EB1200 0>` `ADD DWORD PTR DS:[12EB84],1000`
`0040101E` `813D 80EB1200 0>` `CMP DWORD PTR DS:[12EB84],Uedit32.0054D00>`
`00401028` `-` `0F85 40FD1C00` `JNZ Uedit32.005D0D6E`
`0040102E` `90` `NOP`
`0040102F` `90` `NOP`
`00401030` `0000` `ADD BYTE PTR DS:[EAX],AL`
`00401032` `0000` `ADD BYTE PTR DS:[EAX],AL`
`00401034` `0000` `ADD BYTE PTR DS:[EAX],AL`

现在断点断在 `0040102E` 处，我们在这里用 `CALL kernel32.DebugActiveProcessStop` 这个 `API`，把它们分离出来。首选我们用知道子进程的句柄，可以用 `PUPE` 看一下，刚才我们也看到了，但现在我重新载入程序了，（是个意外，不好意思）所以句柄发生了变化，现在查看一下是 `49C`，注意如果第一个是字母，比如 `B95` 那么修改的时候用 `0B95` 在前面增加一个 `0`，好，我们修改如下：

`00401000` `8105 74EB1200 0>` `ADD DWORD PTR DS:[12EB74],1000`
`0040100A` `8105 80EB1200 0>` `ADD DWORD PTR DS:[12EB80],1000`
`00401014` `8105 84EB1200 0>` `ADD DWORD PTR DS:[12EB84],1000`
`0040101E` `813D 84EB1200 0>` `CMP DWORD PTR DS:[12EB84],Uedit32.0054D000`
`00401028` `-` `0F85 40FD1C00` `JNZ Uedit32.005D0D6E`
`0040102E` `68 9C040000` `PUSH 49C`
`00401033` `E8 FBE1A877` `CALL kernel32.DebugActiveProcessStop`
`00401038` `90` `NOP`
`00401039` `90` `NOP`

我们在 `00401038` 处，下个断点，这样的话就正好运行到把父进程杀死，呵呵。`F9` 一下，好，被断下来了，并且看一下 `EAX` 的值为 `1`，可以确定子进程与他的父进程已经分离，现在我们可以关闭 `OllyDbg` 了。而如果 `EAX = 0`，那是因为你写入有点不对劲（可能是句柄），则你必须要从头到尾核对那些行。你可以重新写入代码，再来一次。反正小妹是第一次就成功了，大家一定要非常小心。

OK，可以关掉 `OD` 了，这样父进程就被杀死了，而子进程还在死循环中，呵呵。

关掉 `OD`，重新打开 `OD`。选择文件菜单中的附加命令，可以看到只有一个够本为 `49C` 的子进程了，呵呵把它附加上吧。OK，成功附加上，运行一下，程序就会在死循环运行，因此按 `F12` 来暂停程序，然后在 `PUPE` 中把 `OEP` 的代码还原成 `55 8B`，再“Parchear”一下就会还原成原来的代码了。呵呵，我们成功了。

好的，现在打开 `LordPE` 然后搜索 `armadillo3.exe` 这个进程。选择这进程以及选择〔active dump engine〕| 〔IntelliDump〕|〔select〕，然后点击 `Dump full`，并且编辑一下，把 `OEP` 改成 `1045AD`,OK 成功了。

运行一下这个 `dump.exe`，当然不可以运行，呵呵，让我们来修复它吧。这里，我会采用一些小妹自已的方法，因为原来的一些方法在 `3.6` 中显得复杂并且有些不能用，请大家留意，并且我尽量，加快速度，包括简单化。

用 `OD` 载入这个 `dump.exe`，

用 `F8` 走下。发现走到 `005045D3` `FF15 98D45400` `CALL NEAR DWORD PTR DS:[54D498]`

再 `F8` 一下就飞了，所以留意这一行时的信息

`005045AD` `>` `55` `PUSH EBP`
`005045AE` `8BEC` `MOV EBP,ESP`
`005045B0` `6A FF` `PUSH -1`
`005045B2` `68 B06F5600` `PUSH dumped.00566FB0`
`005045B7` `68 907F5000` `PUSH dumped.00507F90`
`005045BC` `64:A1 00000000` `MOV EAX,DWORD PTR FS:[0]`
`005045C2` `50` `PUSH EAX`
`005045C3` `64:8925 0000000>` `MOV DWORD PTR FS:[0],ESP`
`005045CA` `83EC 58` `SUB ESP,58`
`005045CD` `53` `PUSH EBX`
`005045CE` `56` `PUSH ESI`
`005045CF` `57` `PUSH EDI`
`005045D0` `8965 E8` `MOV DWORD PTR SS:[EBP\-18],ESP`
`005045D3` `FF15 98D45400` `CALL NEAR DWORD PTR DS:[54D498]`

也就是 `005045D3`，我们运行到这里，

可以看到一些东西：

`00F1C3BB` `？？` 明显是错误的，OK，我们到 `0054D498` 中去，注意是内存窗口，

OK，鼠标滚轮向上走，一直到顶：

可以看出 `IAT` 从 `0054D0000` 开始，（为什么，因为可以看到很多 `77` 之类的东西，呵呵） 现在滚轮向下走，

现在 `IAT` 结束了，也就是在 `0054DAD8`，因为没有 `77` 了，呵呵，归类一下，`OEP＝0010ad45` `IAT` 起始应该是 `14D000` 大小为 `AD8` 。好记下这些重要内容，一会儿要用。

和标准壳一样，也有一个 `magicjump`，我们要找到它，现在难就难在，这些加解密都是子进程自己完成的，我们又要重新再次杀死父进程，而且我们要走在 `magicjump` 之前，不要害怕麻烦，小妹和大家再来吧。

用 `OD` 重新打开加壳的 `uledit.exe`，隐藏 `OD`，然后去掉所有异常选项的勾，因为我前面说过了，我加壳时去掉了 `AM` 注册提醒框，而现在我们要有个合适断点在合适时候停下来，所有如果不去掉勾的话，走两步就运行了。呵呵。把上面的工作做好。`F9` 运行，按 `SHIFT＋F9` 两次忽略掉两次开头的异常。然后单步事件而停下来。如下：

`005D0E75` `C745 FC 0000000>` `MOV DWORD PTR SS:[EBP\-4],0` ――――＞停在这。
`005D0E7C` `EB 33` `JMP SHORT Uedit32.005D0EB1`
`005D0E7E` `8B55 EC` `MOV EDX,DWORD PTR SS:[EBP\-14]`
`005D0E81` `8B02` `MOV EAX,DWORD PTR DS:[EDX]`
`005D0E83` `8B08` `MOV ECX,DWORD PTR DS:[EAX]`
`005D0E85` `898D 2CE7FFFF` `MOV DWORD PTR SS:[EBP\-18D4],ECX`
`005D0E8B` `8B85 2CE7FFFF` `MOV EAX,DWORD PTR SS:[EBP\-18D4]`
`005D0E91` `2D 04000080` `SUB EAX,80000004`
`005D0E96` `F7D8` `NEG EAX`
`005D0E98` `1BC0` `SBB EAX,EAX`
`005D0E9A` `24 FE` `AND AL,0FE`
`005D0E9C` `83C0 01` `ADD EAX,1`
`005D0E9F` `C3` `RETN`
`005D0EA0` `8B65 E8` `MOV ESP,DWORD PTR SS:[EBP\-18]`
`005D0EA3` `C685 44F6FFFF 0>` `MOV BYTE PTR SS:[EBP\-9BC],1`
`005D0EAA` `C745 FC 0000000>` `MOV DWORD PTR SS:[EBP\-4],0`
`005D0EB1` `8B95 E0F5FFFF` `MOV EDX,DWORD PTR SS:[EBP\-A20]`
`005D0EB7` `A1 D8156000` `MOV EAX,DWORD PTR DS:[6015D8]`

下面要格外小心了。

先重新打开 `PUPE`，因为我们随时要用到它，我们的原理是刚才那个地址

`0054D498`，被赋了一个错误值，从而跟出 `magicjump` 的地方，我们利用 `PUPE`，来观察 `0054D498` 被赋值的情况，呵呵，现在当然是 `0` 了：

好，开始按 `SHIFT＋F9` 一次一次往下走吧，我的经验至少要 `60` 次后，才会发生变化，`60` 次后，我按一次 `SHIFT＋F9`，再回到 `PUPE` 中，按一次 `Buscar` 按钮，看看目前值是多少，`62` 次后，如下：

这是它第一次被赋值，对我们来说太重要了。赶在 `magicjump` 之前了，好，现在小妹又要和大家来杀死父进程，我们又要制造一个死循环了，我们随意选取一个 `API` 就可以，在 `OD` 中查看模块，来到 `kernel32.dll`，在窗口中右击，查找当前模块指令，然后再下面这图：

`GetProcAddress` 这个函数的地址是 `77E5B332`

好，现在利用 `PUPE`，来让这个函数死循环，按下面的图做：

原指令是 `558B` 我们改成 `EBFE` .

好，在 `OD` 中回到模块 `uedit32.exe` 中，

我们来到：

`005BB000` `55` `PUSH EBP`
`005BB001` `8BEC` `MOV EBP,ESP`
`005BB003` `83EC 0C` `SUB ESP,0C`
`005BB006` `8B45 10` `MOV EAX,DWORD PTR SS:[EBP+10]`
`005BB009` `50` `PUSH EAX`
`005BB00A` `E8 A0480200` `CALL Uedit32.005DF8AF`
`005BB00F` `83C4 04` `ADD ESP,4`
`005BB012` `8945 F8` `MOV DWORD PTR SS:[EBP\-8],EAX`
`005BB015` `8B4D F8` `MOV ECX,DWORD PTR SS:[EBP\-8]`
`005BB018` `894D FC` `MOV DWORD PTR SS:[EBP\-4],ECX`
`005BB01B` `8B55 0C` `MOV EDX,DWORD PTR SS:[EBP+C]`



# Armadillo3.60 CopyMem-ll + Debug-Blocker 脱壳―――补充改进归纳总结

## 开篇

大家好，今天小妹将对 `Armadillo3.60 CopyMem-ll + Debug-Blocker` 脱壳的脱壳方法进行补充改进归纳总结，如果你看过我上一篇的话，你会觉得看不懂，或者很复杂，呵呵，相信看完这篇之后，你会有收获的，你会觉得脱掉 `AM` 的壳是件非常简单的事。

感谢 `jwh51` 提供的在标准双进程中的方法，我知道这一方法引自于 `mysqladm` 的一篇文章，所以我现在研究了这篇他的文章，感谢他优秀的文章，还要感谢 `Ricardo Narvaja` 的经典文章，他的文章可以说是开山作品，现在所以的方法都是从他的方法中改进出来的，呵呵。现在我将上面的内容结合自己的方法研究，摸索出一些方法和经验，希望对你有帮助。

## 找出 OEP

我不多说了，感觉这里面没有捷径，都一样，可以参考我前面的文章作者其它的都可以，一样的。

## DUMP 出程序

目前两种方法，一种是 `Ricardo Narvaja` 的，不过他的有些方法不适合新版本 `3.6`，我已做了修正改进，完全可行的，（详见文章）。一种是 `mysqladm` 的，感觉原理也一样，不过实现的途径不同，但两种方法复杂度都差不多，至于选哪一种，随你喜欢吧。

## 下面要谈的是重点：IAT 的修复

先找出 `IAT` 的起始地址和大小，这个不是难点，方法很多，如果你不会可以看我的文章。下面的是要找出 `magicjump`，并修改再修复。以前用的是 `Ricardo Narvaja` 的方法，原理如下：

找出一个错误的指针，然后借助 `PUPE` 查看进程中的值，找到时机，再杀掉父进程，然后绑定子进程，再调试直至完成。`Mysqladm` 的则是分离出子进程，然后直接调试子进程，注意这时是头开始的，并且通过中断禁止再产生子进程，这种方法可以说是上面的方法的改进，这种方法速度快，可行性也非常好，推荐使用这种方法，（感谢 `jwh51`）我用这种方法对 `ultraedit` 重新做了一下，非常好的。很快，错误率也低。

下面我就把这一过程做一次给大家，希望你能够学会。限于时间，我不再介绍理论知识，要看这些内容原由请看文章。

现在我们假设已 `DUMP` 出程序，并且找到了 `OEP`，`IAT` 的起始和大小。（详见文章）。

## 实际操作步骤

我们用 `OD` 载入加壳程序，忽略所有异常，`bp DebugActiveProcess`，`F9` 断下。看堆栈窗口：

```
0012DA9C   005D0BDB   /CALL 到 DebugActiveProcess 来自 Uedit32.005D0BD5
0012DAA0   00000584   ProcessId = 7F4   ―――＞子进程句柄
```

打开另一个 `OD` 绑定 `7F4` 这个子进程。

然后 `ALT+F9`，还原代码，不然是死循环，还原开头两行二进制：

```
60
E8 00000000
```

`OK`，`BP OpenMutexA`，`F9` 后中断，不要清除断点，留意一下堆栈：

```
0012F574   005C25F1   /CALL 到 OpenMutexA 来自 Uedit32.005C25EB
0012F578   001F0001   |Access = 1F0001
0012F57C   00000000   |Inheritable = FALSE
0012F580   0012FBB4   MutexName = "7F4::DAB78A59F5"  注意这里，
0012F584   0012FF04
0012F588   00000000
0012F58C   005E0999   Uedit32.005E0999
0012F590   003D003C
```

现在我们直接跳到 `00401000` 在这里新建 `EIP`，（为什么是这里，自己看吧，呵呵）然后修改如下：

```
00401000    60              PUSHAD   
00401001    9C              PUSHFD   
00401002    68 B4FB1200    PUSH 12FBB4 ――――＞这里和上面那个地方一致
00401007    33C0            XOR EAX,EAX   
00401009    50              PUSH EAX   
0040100A    50              PUSH EAX   
0040100B    E8 E694A677    CALL KERNEL32.CreateMutexA   
00401010    9D              POPFD   
```

## 后续处理与总结

我们在 `005BB000` 处右击新建 `EIP`，是为了让程序马上执行我们这里的命令。用 `PUPE` 查看一下当前子进程的句柄，我的是 `13C`。改代码如下：

```
005BB000    68 3C010000     PUSH 13C
005BB005    E8 29428D77     CALL kernel32.DebugActiveProcessStop
005BB00A    90              NOP  ――――＞下断。
005BB00B    90              NOP
005BB00C    90              NOP
005BB00D    90              NOP
005BB00E    90              NOP
005BB00F    83C4 04         ADD ESP,4
005BB012    8945 F8         MOV DWORD PTR SS:[EBP-8],EAX
```

再次利用 `kernel32.DebugActiveProcessStop` 杀死父进程，下面的 `NOP` 当然是用来下断的。

`F9` 一下，`OK`，被断下，`EAX=1`，说明父进程和子进程之间已分离，好，关闭 `OD`，重新打开 `OD`，附加这个子进程，运行一下，然后暂停，在 `PUPE` 中，把 `558B` 再写回去，不要让它再死循环了，呵呵。

下面我用自己的方法。

`BP VirtualProtect`，`F9` 中断，看一下堆栈窗口：

```
0012BEF0   00F318F8   /CALL 到 VirtualProtect 来自 00F318F2
0012BEF4   0054D9EC   |Address = Uedit32.0054D9EC
0012BEF8   00000058   |Size = 58 (88.)
0012BEFC   00000002   |NewProtect = PAGE_READONLY
0012BF00   0012D5F0   pOldProtect = 0012D5F0
0012BF04   00000001
0012BF08   003CC008
0012BF0C   00000000
0012BF10   00000039
0012BF14   00000000
```

好的，我们跳到 `00F318F2`，现在我们鼠标滚轮向上走，直到发现了有这样的 API：

```
; kernel32.GetModuleHandleA
```

好，我们很快找到了。

```
00F31349    FF15 C480F300   CALL NEAR DWORD PTR DS:[F380C4]          ; kernel32.GetModuleHandleA
00F3134F    3985 BCE8FFFF   CMP DWORD PTR SS:[EBP-1744],EAX
00F31355    75 0F           JNZ SHORT 00F31366
00F31357    C785 B8E8FFFF 3>MOV DWORD PTR SS:[EBP-1748],0F3C530
00F31361    E9 C4000000     JMP 00F3142A
00F31366    83A5 94E6FFFF 0>AND DWORD PTR SS:[EBP-196C],0
00F3136D    C785 90E6FFFF 4>MOV DWORD PTR SS:[EBP-1970],0F3CB48
00F31377    EB 1C           JMP SHORT 00F31395
00F31379    8B85 90E6FFFF   MOV EAX,DWORD PTR SS:[EBP-1970]
00F3137F    83C0 0C         ADD EAX,0C
00F31382    8985 90E6FFFF   MOV DWORD PTR SS:[EBP-1970],EAX
00F31388    8B85 94E6FFFF   MOV EAX,DWORD PTR SS:[EBP-196C]
00F3138E    40              INC EAX
00F3138F    8985 94E6FFFF   MOV DWORD PTR SS:[EBP-196C],EAX
00F31395    8B85 90E6FFFF   MOV EAX,DWORD PTR SS:[EBP-1970]
00F3139B    8338 00         CMP DWORD PTR DS:[EAX],0
00F3139E    0F84 86000000   JE 00F3142A
00F313A4    8B85 90E6FFFF   MOV EAX,DWORD PTR SS:[EBP-1970]
00F313AA    8B40 08         MOV EAX,DWORD PTR DS:[EAX+8]
00F313AD    83E0 01         AND EAX,1
00F313B0    85C0            TEST EAX,EAX
00F313B2    74 25           JE SHORT 00F313D9
00F313B4    A1 9455F400     MOV EAX,DWORD PTR DS:[F45594]
00F313B9    8B0D 9455F400   MOV ECX,DWORD PTR DS:[F45594]            ; Uedit32.005FB260
00F313BF    8B40 58         MOV EAX,DWORD PTR DS:[EAX+58]
00F313C2    3341 6C         XOR EAX,DWORD PTR DS:[ECX+6C]
00F313C5    8B0D 9455F400   MOV ECX,DWORD PTR DS:[F45594]            ;
```

清掉原来的断点，上面的 `00F3139E` 就是 `magicjump` 一定要让它跳，我们在那里下断，`F9` 后断下来。修改标志位，使 `Z=1`，让它强行跳。依次循环做，我改了 `17` 次，后来运行一下便提示无法处理异常了，不能再按任何键，不能就飞了，可异没有走到 `OEP`，小妹想通过标准壳中的方法强行下断走到 `OEP`，也不行，断点都断不下来，不关也没关系，可以看到内存窗口中正确的 `IAT` 出现了：

我们拿出 `ImportREC`，然后修复它吧。`OEP` 填 `10ad45`。`IAT` 起始值为 `14D000` 大小为 `AD8`。不要按搜索 `OEP`，直接 `GET` 表吧，呵呵，`CUT` 掉几个无效的，`OK`。运行了。减小一下体积吧。呵呵。

## 关于 Magicjump 规律

感谢所有人，呵呵，不过总觉得最后那里有点问题，要花点时间研究一下，呵呵，另外 `magicjump` 是有规律的。

在 `3.6` 中，形式是这样的：

```
00F31349    FF15 C480F300   CALL NEAR DWORD PTR DS:[F380C4]          ; kernel32.GetModuleHandleA
00F3134F    3985 BCE8FFFF   CMP DWORD PTR SS:[EBP-1744],EAX
00F31355    75 0F           JNZ SHORT 00F31366
00F31357    C785 B8E8FFFF 3>MOV DWORD PTR SS:[EBP-1748],0F3C530
00F31361    E9 C4000000     JMP 00F3142A
00F31366    83A5 94E6FFFF 0>AND DWORD PTR SS:[EBP-196C],0
00F3136D    C785 90E6FFFF 4>MOV DWORD PTR SS:[EBP-1970],0F3CB48
00F31377    EB 1C           JMP SHORT 00F31395
00F31379    8B85 90E6FFFF   MOV EAX,DWORD PTR SS:[EBP-1970]
00F3137F    83C0 0C         ADD EAX,0C
00F31382    8985 90E6FFFF   MOV DWORD PTR SS:[EBP-1970],EAX
00F31388    8B85 94E6FFFF   MOV EAX,DWORD PTR SS:[EBP-196C]
00F3138E    40              INC EAX
00F3138F    8985 94E6FFFF   MOV DWORD PTR SS:[EBP-196C],EAX
00F31395    8B85 90E6FFFF   MOV EAX,DWORD PTR SS:[EBP-1970]
00F3139B    8338 00         CMP DWORD PTR DS:[EAX],0―――――
00F3139E    0F84 86000000   JE 00F3142A  ――――――――――重点
```

看到了吧，而且在 `3.6` 中这个跳转地址总是 `XXXX139E`――――开场白。

`OEP` 往下找找就可以了，`CALL EDI`，呵呵，写了 8 个小时，终于好了，希望大家能有所进步，呵呵，错误之处难免，请提出。

本本都快烫了，呵呵。

感谢我们的组织 `NUKE` 感谢老大 `伊万`，呵呵，欢迎大家来。



### 调试日志与反汇编分析

```
00401011    61                 POPAD   
00401012  - E9 8F9FA777       JMP KERNEL32.OpenMutexA   
```

继续`F9`，又会中断。清除断点。其实上面的步骤可以禁止再产生子进程。

`bp VirtualProtect`　几次中断，注意的这里的几次是多少次是有规律的，最好事先要忽略所有异常，然后会来到弹出一个异常框，（具体参考我的前一篇文章），好的再次越过，留意一下堆栈：

```
0012BEF0   00F3077C  /CALL 到 VirtualProtect 来自 00F30776   
0012BEF4   00401000  |Address = Uedit32.00401000   
0012BEF8   0014C000  |Size = 14C000 (1359872.)   
0012BEFC   00000004  |NewProtect = PAGE_READWRITE   
0012BF00   0012D71C  pOldProtect = 0012D71C   
```

一般情况下来到这里是 6 次。3.4 也这样。

现在有两种方法　一种看上面的　`00F30776`　呵呵，根据我发现的规律你直接跳到 `00F3139E` 吧，如果 `00F3139E` 还没解压出来，请再按两次 `F9`，再 `G` 过去就可以。

第二种下断 `bp GetModuleHandleA`　直接过来了。

好的，来到

```
00F3139B    8338 00           CMP DWORD PTR DS:[EAX],0   
00F3139E    0F84 86000000    JE 00F3142A   
00F313A4    8B85 90E6FFFF    MOV EAX,DWORD PTR SS:[EBP-1970]   
00F313AA    8B40 08          MOV EAX,DWORD PTR DS:[EAX+8]   
00F313AD    83E0 01          AND EAX,1   
00F313B0    85C0             TEST EAX,EAX   
00F313B2    74 25            JE SHORT 00F313D9   
00F313B4    A1 9455F400      MOV EAX,DWORD PTR DS:[F45594]   
00F313B9    8B0D 9455F400    MOV ECX,DWORD PTR DS:[F45594]                 ; Uedit32.005FB260   
00F313BF    8B40 58          MOV EAX,DWORD PTR DS:[EAX+58]   
```

在 `00F3139E` 处下断，每次中断改 `Z＝1`，由于走不到 `OEP`，所以最后一次 `F9`，会提示无法处理异常了，这时拿出 `IMPORT` 就可以，你不放心的话可以先前在内存窗口中 `G` 到 `0054D000` 也就是 `IAT` 处，最后中断时，你会发现这里正确还原了。好了，现在我们就成功了。

你应该不会害怕 `AM` 了吧，呵呵。

I’ll be back

我　是　`TX` 共享软件终结者。

`4dd466d0e1e0cc123afbe504da2462ca`
