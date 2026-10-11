
# PE-ShiELD v0.2b2

> 原文：[`www.pediy.com/kssd/pediy03/forum397.htm`](https://www.pediy.com/kssd/pediy03/forum397.htm)

标题: 小丁丁 和 `WinDos2000` 过来看看(多谢帮助 ^\_^),哪位 1212 也过来看看吧 ----对 `PE-ShiELD` v0.2b2 加壳程序的脱壳的一点分析

## 内容

对 `PE-ShiELD` v0.2b2 加壳程序的脱壳的一点分析

### 【声明】

我写文章以交流为主，希望大家在转载时能保持文章的完整性。

### 【前言】

脱 `PE-ShiELD` v0.2b2 的壳很花功夫，不过为了感谢丁丁虾和 `WinDos2000` 的帮助，还是赶写了此文，选这个壳的原因，并不是它很简单，只因为正好碰上一位 1212 想了解这个壳，而我又有过这个壳的脱壳经验。所以就选它了。不过现在这个版本据说已经有脱壳机了，而我当时脱壳时还没有。所以，如果你只为了脱壳，就没有必要看此文了。找脱壳机就是了。

这个程序的手动脱壳很花功夫，这里写这篇文章主要侧重一点它的保护机制。在我跟踪此程序的过程中发现他反跟踪的地方相当多，不过幸亏没有多少故意让你死机的代码。列举一下他反跟踪的种类：

1. 检测 `\\.\SICE` 来防范 `SoftICE` 的跟踪
2. 检测 `\\.\TRW` 来防范 `TRW2000` 的跟踪
3. 在对 `UnhandledExceptionFilter` 及其要引入的 API 函数在调用之前检查入口代码是否为 `0xCC`。防范在 API 函数上设断点跟踪。
4. 在对要引入的 API 函数在调用时，并不从入口处执行，而是进入其中的代码执行。防范在入口几处的代码上设断点跟踪。
5. 检测 `EntryPoint` 和 `NumberOfSections` 是否被修改，来防范程序被修改，或者是 `SMC` 的方法进行破解。
6. 把文件头部的所有数据进行累加和 `CRC` 比较是否被修改，来防范程序被修改或者破解。
7. 使用"花指令"和 `SMC` 的方法来分段还原代码，来防范静态分析或者增加动态跟踪的难度。
8. 修改了代码段中的 `JMP [xxxxxxxx]` 使之指向指向内存中动态申请的地址，防范用一般方法脱壳。
9. 运行时破坏了 `PE` 结构中的所有数据，来防范用 `ProcDump` 等工具脱壳或者 `DUMP` 内存映象。
10. 修改了 `WINDOWS` 加载环境中保留的 `PE` 结构，来防范 `ProcDump` 等工具脱壳或者 `DUMP` 内存映象。

还有一些，不足列举。

**样例文件：** `znzdatII.exe` （指南针数据转换程序）

**下载地址：** `http://202.102.230.15/tanfu/soft/mdat/znzdatII.exe`

**加壳方式：** `PE-ShiELD` v0.2b2

**检测工具：** `FileInfo` v2.30

**目标：** 手动脱壳

**脱壳工具：** `SoftICE` 4.05，`ICEDUMP` 6.0168，`PEditor` 1.6，`HEX WorkShop` 2.10，`HIEW` 5.91

**作者：** `ljttt`

**写作日期：** 2000-10-28

1. 启动 `ICEDUMP`。

2. `Ctrl-D` 进入 `SoftICE`，设下断点

   ```
   bpint 3
   ```

3. 用 `PEditor` 1.6 打开 `znzdatII.exe`，记下其中的关键，然后单击 `break'n'enter` 按钮，单击 `Run` 按钮。将中断进入 `SoftICE`。

   - Entry Point: `AA000`
   - Image Base: `400000`
   - Image Size: `AD000`
   - Size of Headers: `600`

4. 现在开始了我们的艰苦历程。由于显示的都是"花指令"，所以无法正确显示出正确的反汇编结果。列举代码也没有意义。不过跟踪了一次后，心里有数以后，可以在某些自己认为比较关键的代码处设下断点，以便下次跟踪时可以快速按 `F5` 到此。

5. 我们来看看 `UnhandleExceptionFilter` 函数入口代码的比较代码：

   ```
   015F:004AA2F9  EB01                 JMP       004AA2FC                 <--反汇编正确的代码
   015F:004AA2FB  E98D9DB511           JMP       1200408D                 <--反汇编不正确的代码

   015F:004AA300  0000                 ADD       [EAX],AL

   015F:004AA302  EB02                 JMP       004AA306
   015F:004AA304  CD20                 INT       20 VXDJmp 000D,4DE8

   015F:004AA30A  00EB                 ADD       BL,CH
   015F:004AA30C  01EA                 ADD       EDX,EBP

   015F:004AA30E  8038CC               CMP       BYTE PTR [EAX],CC        <--这句是关键！
   015F:004AA311  EB01                 JMP       004AA314
   015F:004AA313  E90F84AF03           JMP       03FA2727

   015F:004AA318  0000                 ADD       [EAX],AL
   ```

   可以看到许多代码反汇编的结果不正确。这是其中一处在主程序中反跟踪代码，还有许多在子程序中的反跟踪代码就不一一列举了。

   我们可以记住 `CMP BYTE BTR [EAX],CC` 的代码地址 `004AA30E` ，那么设下断点后下次跟踪时就方便了。其他类似。

6. 我们再来看看 检测 `TRW2000` 的地方。（记住代码地址 `004AA5C3` ）

   ```
   015F:004AA59C  6800008000           PUSH      00800000
   015F:004AA5A1  EB01                 JMP       004AA5A4                 <--反汇编正确的代码
   015F:004AA5A3  C78D8560100000EB02EAMOV       DWORD PTR [EBP+00106085],EA02EB00   <--反汇编不正确的代码

   015F:004AA5AD  0450                 ADD       AL,50
   015F:004AA5AF  EB01                 JMP       004AA5B2
   015F:004AA5B1  C78B858D110000EB01EAMOV       DWORD PTR [EBX+00118D85],EA01EB00
   015F:004AA5BB  E821070000           CALL      004AACE1                 <--反跟踪，检测 \\.\TRW.VXD 来防范 TRW2000 的跟踪
   015F:004AA5C0  EB01                 JMP       004AA5C3
   015F:004AA5C2  C783F8FFEB01EA0F85E3MOV       DWORD PTR [EBX+01EBFFF8],E3850FEA
   015F:004AA5C3  83F8FF               CMP       EAX,-01                  <--比较的地方
   015F:004AA5C6  EB01                 JMP       004AA5C9
   015F:004AA5C8  EA0F85E3000000       JMP       0000:00E3850F

   015F:004AA5CF  8D8585100000         LEA       EAX,[EBP+00001085]
   015F:004AA5D5  EB01                 JMP       004AA5D8
   015F:004AA5D7  EA8D9D92100000       JMP       0000:10929D8D
   015F:004AA5DE  EB02                 JMP       004AA5E2
   ```

7. 好了，现在快到了关键的地方了。我们来看看：

   ```
   015F:004AA638  6A00                 PUSH      00
   015F:004AA63A  EB01                 JMP       004AA63D                 <--反汇编正确的代码
   015F:004AA63C  E96A00EB01           JMP       0235A6AB                 <--反汇编不正确的代码

   015F:004AA641  C78B8592100000EB01EAMOV       DWORD PTR [EBX+00109285],EA01EB00
   015F:004AA64B  E891060000           CALL      004AACE1                 <--好象是 CreateThread 调用
   015F:004AA650  EB01                 JMP       004AA653
   015F:004AA652  E9FF95A510           JMP       10F03C56
   015F:004AA653  FF95A5100000         CALL      [EBP+000010A5]          <--这里是 ExitThread 调用
   015F:004AA659  8B859D110000         MOV       EAX,[EBP+0000119D]      <--注意要先在此处设下断点
   015F:004AA65F  E97D060000           JMP       004AACE1
   015F:004AA664  8B85E0110000         MOV       EAX,[EBP+000011E0]
   015F:004AA66A  E972060000           JMP       004AACE1
   015F:004AA66F  8B85E4110000         MOV       EAX,[EBP+000011E4]
   ```

   在 `CALL [EBP+000010A5]` 处我们停下来，在其后的代码 `MOV EAX,[EBP+0000119D]` 处设下断点

   ```
   bpx 004AA659
   ```

   然后按 `F10` 继续，否则程序将直接执行。那么这一次的跟踪就结束了。如果对比在 `CALL [EBP+0000119D]` 执行前后的 `401000` 代码处的数据就可以发现执行完后代码加载到内存中了。这时的代码是还原后的代码。不过还有些没有还原。这也是 `PE-SHiELD` 防范脱壳的技巧吧。其中一处是代码段中 `JMP [xxxxxxxxx]` 语句没有完全还原，另外就是 `.idata` Section 所在的 Import Table 部分没有还原了。

8. 我们继续跟踪，现在我们来看看如何得到完整的 `.idata` Section 。当我们到如下代码处时：

   ```
   015F:004AA94B  0FB6C9               MOVZX     ECX,CL
   015F:004AA94E  EB01                 JMP       004AA951
   015F:004AA950  EA8B06EB01C889       JMP       89C8:01EB068B

   015F:004AA957  16                   PUSH      SS
   015F:004AA958  EB01                 JMP       004AA95B
   015F:004AA95A  6833D3EB01           PUSH      01EBD333

   015F:004AA95F  CA8907               RETF      0789
   015F:004AA962  EB01                 JMP       004AA965

   015F:004AA964  C783C604EB01E283C704MOV       DWORD PTR [EBX+01EB04C6],04C783E2
   015F:004AA96E  EB01                 JMP       004AA971
   015F:004AA970  684B75DAEB           PUSH      EBDA754B

   015F:004AA975  02EA                 ADD       CH,DL
   015F:004AA977  04F3                 ADD       AL,F3
   015F:004AA979  A4                   MOVSB
   015F:004AA97A  8B9549110000         MOV       EDX,[EBP+00001149]   <--从这里开始的代码将没有"花指令"了，我们终于走出了沼泽地
   015F:004AA980  2B9541110000         SUB       EDX,[EBP+00001141]
   015F:004AA986  0395C9110000         ADD       EDX,[EBP+000011C9]
   015F:004AA98C  89954D110000         MOV       [EBP+0000114D],EDX
   015F:004AA992  8BB589110000         MOV       ESI,[EBP+00001189]
   015F:004AA998  0BF6                 OR        ESI,ESI
   015F:004AA99A  7433                 JZ        004AA9CF
   015F:004AA99C  03B5C9110000         ADD       ESI,[EBP+000011C9]
   015F:004AA9A2  83C610              ADD       ESI,10
   015F:004AA9A5  8B9549110000         MOV       EDX,[EBP+00001149]
   015F:004AA9AB  2B95C5110000         SUB       EDX,[EBP+000011C5]
   015F:004AA9B1  0BD2                 OR        EDX,EDX
   015F:004AA9B3  7405                 JZ        004AA9BA
   015F:004AA9B5  E854040000           CALL      004AAE0E
   015F:004AA9BA  8B95C9110000         MOV       EDX,[EBP+000011C9]
   015F:004AA9C0  2B95D1110000         SUB       EDX,[EBP+000011D1]
   015F:004AA9C6  0BD2                 OR        EDX,EDX
   015F:004AA9C8  7405                 JZ        004AA9CF
   015F:004AA9CA  E83F040000           CALL      004AAE0E
   015F:004AA9CF  80BD3C11000000       CMP       BYTE PTR [EBP+0000113C],00
   015F:004AA9D6  7424                 JZ        004AA9FC
   015F:004AA9D8  6A04                 PUSH      04
   015F:004AA9DA  6800100000           PUSH      00001000
   015F:004AA9DF  FFB5AD110000         PUSH      DWORD PTR [EBP+000011AD]
   015F:004AA9E5  6A00                 PUSH      00
   015F:004AA9E7  E86DFCFFFF           CALL      004AA659
   015F:004AA9EC  0BC0                 OR        EAX,EAX
   015F:004AA9EE  0F8469FDFFFF         JZ        004AA75D
   ```
