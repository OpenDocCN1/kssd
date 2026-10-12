
# Asprotect v1.1 分析

> 原文：[`www.pediy.com/kssd/pediy03/forum399.htm`](https://www.pediy.com/kssd/pediy03/forum399.htm)

对 `Asprotect` v1.1 的手动脱壳的一点分析

## 声明

我写文章以交流为主，希望大家在转载时能保持文章的完整性。

## 前言

由于 `r!sc` 已经写出了脱壳机 `rAD.exe`，所以写此文实在没有多大意思。看此文说实话还不如去看 `rAD.exe` 的源代码更有意义。

样例文件： `Asprotect.exe` （`Asprotect` v1.1 的主程序）

加壳方式： `Asprotect` v1.1

目标： 手动脱壳

作者： `ljttt`

写作日期： 2000-10-13

## 1、启动 `PEEditor`

首先，启动 `PEEditor`，打开 `Asprotect.exe` 文件，得到对我们有用的信息。

```
Image Base     : 400000
Size of Image  : 9E000
```

## 2、进入 `SoftICE` 下断点

`Ctrl-D` 进入 `SoftICE`，下断点

```
bpint 3
```

## 3、修改入口代码

单击 `break'n'enter`，单击 `Run`，将中断在第一条指令处，此时我们按照提示输入如下指令

```
eb eip 60
```

这样，就修改了入口的代码由 `CC` 改为 `60`。这样屏幕显示如下：

```
015F:0048F001  60                  PUSHAD                        <--第一条指令处
015F:0048F002  E9D9040000          JMP       0048F4E0
015F:0048F007  2BD4                SUB       EDX,ESP
015F:0048F009  ED                  IN        EAX,DX
015F:0048F00A  AC                  LODSB
```

## 4、跟踪至变形引擎处

跟踪一段，我们可以到达如下地方：

```
015F:0048F58A  81EE02000000        SUB       ESI,00000002
015F:0048F590  81DB91D657C9        SBB       EBX,C957D691
015F:0048F596  0FBFEB              MOVSX     EBP,BX
015F:0048F599  81FFE6C50868        CMP       EDI,6808C5E6
015F:0048F59F  0F8588FFFFFF        JNZ       0048F52D
015F:0048F5A5  BA4166DFBC          MOV       EDX,BCDF6641
015F:0048F5AA  E928FFFFFF          JMP       0048F4D7          <--此处将跳走
```

从跟踪的情况来看，我觉得这里应该说用到了"花指令"和"变形引擎"两种技术。原来我设想的"变形引擎"会出现在加壳软件中，看来现在"如愿"了。这也说明加壳软件在不断地提高反跟踪的技术。看来我也得多下苦功了。不然，以后只有用用别人写的脱壳机的份了。

## 5、跟踪 `SMC` 技术运用处

继续跟踪到如下一段时：

```
015F:00C260DD  8BC8                MOV       ECX,EAX
015F:00C260DF  8DBD452A4400        LEA       EDI,[EBP+00442A45]
015F:00C260E5  8BB575294400        MOV       ESI,[EBP+00442975]
015F:00C260EB  F3A4                REPZ MOVSB                  <--此处是 SMC 技术的运用。将修改后面的代码。
015F:00C260ED  8B8575294400        MOV       EAX,[EBP+00442975] <--这里的代码将被修改
015F:00C260F3  6800800000          PUSH      00008000
015F:00C260F8  6A00                PUSH      00
015F:00C260FA  50                  PUSH      EAX
015F:00C260FB  FF957D294400        CALL      [EBP+0044297D]
015F:00C26101  8D0E                LEA       ECX,[ESI]
015F:00C26103  85512C              TEST      [ECX+2C],EDX
015F:00C26106  44                  INC       ESP
015F:00C26107  07                  POP       ES
015F:00C26108  50                  PUSH      EAX
015F:00C26109  C3                  RET
```

这是执行 `REPZ MOVSB` 指令以后的代码

```
015F:00C260E5  8BB575294400        MOV       ESI,[EBP+00442975]
015F:00C260EB  F3A4                REPZ MOVSB
015F:00C260ED  8B8575294400        MOV       EAX,[EBP+00442975] <--这是修改后的代码
015F:00C260F3  6800800000          PUSH      00008000
015F:00C260F8  6A00                PUSH      00
015F:00C260FA  50                  PUSH      EAX
015F:00C260FB  FF957D294400        CALL      [EBP+0044297D]
015F:00C26101  8D85512C4400        LEA       EAX,[EBP+00442C51]
015F:00C26107  50                  PUSH      EAX
015F:00C26108  C3                  RET
```

## 6、跟踪至 DLL 返回处

继续跟踪到如下：

```
015F:00C265B8  5B                  POP       EBX
015F:00C265B9  0BDB              OR        EBX,EBX
015F:00C265BB  8985112F4400      MOV       [EBP+00442F11],EAX
015F:00C265C1  61                  POPAD
015F:00C265C2  7508                JNZ       00C265CC
015F:00C265C4  B801000000          MOV       EAX,00000001
015F:00C265C9  C20C00              RET       000C
015F:00C265CC  689C0EC200          PUSH      00C20E9C          <--这个 00C20E9C 的地址也是程序用 SMC 的方式得到的
015F:00C265D1  C3                  RET                           <--我们将到此处
```

## 7、跟踪至 OEP 附近

在 `RET` 指令后将到如下一段代码处：

```
015F:00C20E9C  55                  PUSH      EBP
015F:00C20E9D  8BEC                MOV       EBP,ESP
015F:00C20E9F  83C4F4              ADD       ESP,-0C
015F:00C20EA2  E88922FFFF          CALL      00C13130
015F:00C20EA7  0F855F2FFFFF        JNZ       00C13E0C
015F:00C20EAD  E81634FFFF          CALL      00C142C8
015F:00C20EB2  E8917BFFFF          CALL      00C18A48
015F:00C20EB7  E86489FFFF          CALL      00C19820
015F:00C20EBC  E8BFACFFFF          CALL      00C1BB80
015F:00C20EC1  E8462FFFFF          CALL      00C13E0C          <--在这里，我们要按 F8 进入，否则程序将执行完毕
015F:00C20EC6  8BE5                MOV       ESP,EBP
015F:00C20EC8  5D                  POP       EBP
015F:00C20EC9  C20C00              RET       000C
```

## 8、在 DLL 中设断点

在 `CALL 00C13E0C` 处我们按 `F8` 键进入，此时我们不需要再继续直线跟踪下去了。因为如果你看了 `rAD.exe` 的源代码说明就知道，这不过是在 `DLL` 中的程序段中，由于 `DLL` 的入口函数部分没有什么实际意义。所以继续直线跟踪下去，只会花费我们大把大把的时间，这是有几种方法继续跟踪，看过 `rAD.exe` 的源代码后，就可以知道我们可以如下方式设断点。首先搜索特征代码，然后在特征代码处设断点继续跟踪。

第一种：

```
s eip l ffffffff 51,89,14,24,8B,F8,B2,01
```

如果搜索到的地址为 `<Address>`，则可以设断点

```
bpx <Address>-4
```

第二种：

```
s eip l ffffffff 81,C4,04,F0,FF,FF,50,81
```

如果搜索到的地址为 `<Address>`，则可以设断点

```
bpx <Address>
```

第三种：

```
s eip l ffffffff 55,8B,EC,83,C4,F8,53,56,57,8B,F9,89
```

如果搜索到的地址为 `<Address>`，则可以设断点

```
bpx <Address>
```

我一般设第二种方式的断点，以后的跟踪注意一旦出现 `RET` 指令时，首先查看一下堆栈中的内容，如果是将跳转到 `FindClose` 中时，则向下翻页找到 `RET` 语句处设断点，然后按 `F5` 键继续，（不要直接按 `F7`，这样无效）。直到如下代码：

```
015F:00C205B3  E860CEFFFF          CALL      00C1D418          <--此处可以按 F10 带过
015F:00C205B8  E801000000          CALL      00C205BE          <--此处只是一个跳转指令的作用，必须按 F8 键，否则程序将直接执行
015F:00C205BD  8183C404310424E80100ADD   DWORD PTR [EBX+043104C4],0001E824
015F:00C205C7  0000                ADD       [EAX],AL
015F:00C205C9  6883C4048B          PUSH      8B04C483
015F:00C205CE  051420C200          ADD       EAX,00C22014
015F:00C205D3  E802000000          CALL      00C205DA          <--同理
015F:00C205D8  E86883C404          CALL      05868945          <--同理
015F:00C205DD  010424              ADD       [ESP],EAX
015F:00C205E0  C3                  RET
```

## 9、接近 OEP

当我们在 `RET` 指令执行后，（此时将看到两个 `CALL` 调用，在第二个处要按 `F8` 进入）将离最后的目标不远了，所以可以小心地按 `F8` 继续。

```
015F:00C1F845  5D                  POP       EBP
015F:00C1F846  8B65FC              MOV       ESP,[EBP-04]
015F:00C1F849  8B45F8              MOV       EAX,[EBP-08]
015F:00C1F84C  8944241C            MOV       [ESP+1C],EAX
015F:00C1F850  61                  POPAD
015F:00C1F851  50                  PUSH      EAX
015F:00C1F852  6863F8C100          PUSH      00C1F863
015F:00C1F857  687BF8C100          PUSH      00C1F87B
015F:00C1F85C  E8F7FEFFFF          CALL      00C1F758
015F:00C1F861  50                  PUSH      EAX
015F:00C1F862  C3                  RET                           <--此处，我跟踪的情况，将跳转到 00C1F878 处
015F:00C1F863  5B                  POP       EBX
015F:00C1F864  6A10                PUSH      10
015F:00C1F866  E8DDFEFFFF          CALL      00C1F748
015F:00C1F86B  50                  PUSH      EAX
015F:00C1F86C  E8CFFEFFFF          CALL      00C1F740
015F:00C1F871  50                  PUSH      EAX
015F:00C1F872  6A00                PUSH      00
015F:00C1F874  E8D7FEFFFF          CALL      00C1F750
015F:00C1F879  53                  PUSH      EBX
015F:00C1F87A  50                  PUSH      EAX
015F:00C1F87B  C3                  RET                           <--这是我们要跟踪的最后的一条指令了，Yeah！[ESP]=OEP=4574F4。
015F:00C1F87C  90                  NOP
015F:00C1F87D  6845F8C100          PUSH      00C1F845
015F:00C1F882  C3                  RET                           <--此处，将向上跳转到 00C1F845 处
```

## 10、保存内存映象

在 `00C1F87B` 处我们下指令，保存内存映象

```
/dump 400000 9E000 c:\temp\dump.exe
```

## 11、修补工作

现在开始修补工作，用 `PEEditor` 打开 `dump.exe` 文件，选择 `dumpfixer`，即修改 `RS=VS` 等等。

## 12、运行 `ITOOLS`

运行我写的小工具 `ITOOLS`。先选择"修补文件头"和"写入进程空间"两项，然后打开 `Asprotect.exe` 文件，`ITOOLS` 将创建进程，此时直到进程运行出现在任务栏中时（最好再等一会儿），单击确定按钮，`ITOOLS` 将开始修补 `PE` 头和转换 `ThunkData`，然后弹出对话框警告，此时如果转换正确，（即没有出现 `Error` 字眼）选择确定，最后你还需要单击一下 `Asprotect` 进程中的打开菜单一下，如果没有出现错误，则你可以启动 `Imp_List` 进行 `Import Table` 的 `Rebuild` 工作。此时我们可以得到完整的 `Import Table` 文件 `Import0.bin`。



13. 用`Hex WorkShop`打开`dump.exe`和`Import0.bin`，将`Import0.bin`中内容替换到`dump.exe`中的`5F000`处。（这个值可以由`Address.txt`得知）然后保存。

14. 用`PEEditor`修改`Export Table`的`RVA=0`，`Size=0`，`Import Table`的`RVA=5F000`，`Size=`（大于`0`即可）。`Basereloce`的`RVA=0`，`Size=0`，`Tls Table`的`RVA=63000`（此值也可以不改），`Size`不变。保存。

15. 运行程序，发现出现错误提示在地址`C1C4A8`，通过查找`dump.exe`文件，可以知道其在`5D5BC`和`5D5A0`两处有此值，看来还有问题，重新运行`Asprotect`。如果我们前面所设的断点没有下`bd \*`禁止的话，又一次将依次中断在这些断点上，我们下指令

```
d C1C4A8
```

可以看到显示的内容为`408DC3`。`0K!`得到了我们想要的值。回到`Windows`中，修改其中两处的值为`408DC3`。重新运行，依然出错。这次地址很奇怪。看了半天，我终于想明白`5D5BC`和`5D5A0`保存的内容应该是指向内容为`408DC3`的指针。这好办。我把`5D5BC`和`5D5A0`两处的值修改为`5D5A4`，把`5D5A4`的内容修改为`408DC3`。再次运行。`OK!`终于成功。好辛苦。还是`rAD.exe`好！到底是大师的作品。呵呵！

### 后记

这里写得一点新意都没有。浪费了时间！再也不写这种没什么意思的文章了。仅有的一点作用可能就是在`rAD.exe`不起作用的时候，而且你有大把大把的时间脱壳的时候............这是什么逻辑！......:-& (口吐白沫,晕倒在地)

写那篇文章时`Imp_List`还不够利索，现在不一样了。

**标题:** 其实在`CodeSafe`和`PE-Crypt`中都用到了和`ASProtect`类似的手法---"没事"就往`FindClose`中跑，`FindClose`成`WC`了，呵呵......

**内容:**

（当然说"没事"其实是有事，有急事，为了反跟踪嘛，这对加壳软件来说，就象我们每天要去`WC`一样普通。）

这是一种特殊方式的`SEH`的应用。所以你设的内核处断点总能拦到它。

你会发现`ASProtect`还是挺温柔的。既没有让你当机，也没有破坏掉你的跟踪断点，只是出个破框当个警告而已。而其它两个就没有这么善良了。呵呵.....

不过我还是劝你不要把大把大把的时间浪费在脱壳上了，特别是有脱壳机的时候。不然你会和破解一样上瘾的。其实功夫在脱壳之外，技术性的东东你还得看哪些经典的文章书籍。这样要系统得多. `4dd466d0e1e0cc123afbe504da2462ca`
