
# 非常好色 V3.10

> 原文：[`www.pediy.com/kssd/pediy03/forum522.htm`](https://www.pediy.com/kssd/pediy03/forum522.htm)

**软件下载**：`http://ilike.myrice.com:8080/cgi-bin/dl.pl?url=ftp://ftp.bj.software.chinese.com/software/soft_photo/pm30.exe`

**执行文件**：`Tpmagid.exe`

**文件大小**：`535048 bytes`

**说明**：我这个脱壳并没有什么新意，主要思路均参考看雪教学。

**前言**：我一直想找一个能够制作出带农历的日历软件，这个软件就是。不过，该软件只有 30 天的使用寿命。

下载后进行安装，用 `FI.EXE` 看不出被什么软件加了壳，用 `UltraEdit` 打开 `Tpmagid.exe`，观察 PE 文件头，在 `Section` 中有 `PREVIEW`、`WeijunLi`，怎么这么熟悉?！先运行 `Tpmagid.exe`，再运行 `Procdump`，在 `mask` 中选择 `Tpmagid.exe`，再看看 `moudle`，哈哈，`vboxt420.dll`，照妖镜告诉是被 `vbox4.2` 加壳，看雪教学里正好有这方面的脱壳例子，就拿它练手吧。打开看雪教学，看一看 `vbox4.2` 的脱壳过程，记下关键要点，上路。

**工具**：`SoftICE 4.05`、`Procdump 1.6`

**步骤**：

1. 运行非常好色，`Ctrl+D` 激活 `SoftICE`，设断：`bpx getprocaddress`，`F5` 返回。
2. 选择第一个按钮，我不知道这个按钮说的是什么，`big5` 码，反正应该是试用之类的，这时被 `SoftICE` 拦回，`bd *`，`F12` 共 6 次，其中第 2 次时间较长。（`F12` 几次不重要，关键是 `F12` 后光标应该停留在 `CALL ESI` 的后面），至如下代码：

   ```
   :02800483 FFD6       call esi    ====> 在此处设断
   :02800485 5B         pop ebx     ====> 光标停在此处
   :02800486 85C0       test eax,eax
   :02800488 742E       jz 028004B8
   ```

3. 在 `02800483` 处设断，每次运行时，`0280` 可能不同，但 `0483` 肯定相同，`F5`，会又被 `SoftICE` 拦回，`bd *`，`F8`。
4. 一直按 `F8` 或 `F10`，至如下代码处：

   （此段特征是：只有一个 `CALL`，简便的方法是用 `PageDown` 向下翻页，找到 `CALL` 后将光标移到此 `CALL`，`F7`）

   ```
   :02830100 FF75F8     push dword ptr [EBP-08]
   :02830103 FF75FC     push dword ptr [EBP-04]
   :02830106 FF7508     push dword ptr [EBP+08]
   :02830109 E808000000 call 02830116       ====> F8 进入此 CALL
   ```

5. 一直按 `F8` 或 `F10`，至如下代码处：

   （此段特征是：只有一个 `JMP EBX`，简便的方法是用 `PageDown` 向下翻页，找到 `JMP EBX` 后将光标移到此处，`F7`）

   ```
   :0283037C 8B45F8     mov eax, [EBP-08]
   :0283037F 8B4814     mov ecx, [EAX+14]
   :02830382 894DEC     mov [EBP-14], ecx
   :02830385 8B5DEC     mov ebx, [EBP-14]
   :02830388 FFE3       jmp ebx             ====> ebx 中的值即为正确的入口点，用笔记下，ebx=457D90
   ```

6. 程序执行到 `02830388` 时，下 `A EIP`，输入 `JMP EIP`，`F5`。
7. 启动 `Procdump`，在 `Options` 中选择 `Rebuild import table` 后，在 `Task` 中选择 `Tpmagid.exe`，鼠标右键 `dump (full)`，`dump` 出来的文件大小为 `1636864` 个字节。
8. 用 `Procdump` 中的 `PE editer` 编辑被 `dump` 出来的文件，设置入口点为 `457D90-400000=57D90`。
9. `dump` 后，别忘了 `Ctrl+D`，将 `jmp eip` 改回 `jmp ebx`，尽管我们已经大功告成。

将 `dump` 出来的文件改名为 `Tpmagid.exe`，覆盖原来的文件，运行一下，已经没有壳了，再运行 `Procdump`，在 `task` 中选择 `Tpmagid.exe`，`Module` 中已经没有 `vboxt420.dll` 了，脱壳成功。因这个软件是 `big5` 码，如果不嫌麻烦的话，还可以用 `exescope` 等将其改成简体中文显示，将什么“印表机”之类的改为“打印机”，又成为简体中文版了。

另外，我用 `Procdump` 下的 `vbox42` 进行脱壳，虽然能够把壳脱掉，但脱壳后的文件并不能够运行。用 `PE editor` 观看两个脱壳的文件，入口点没错，都是 `57D90`，但 `size of image` 不同。

这种壳用 `TRW2000` 脱起来可能更简单，但我的机子中安装了超级保镖后，不能运行 `TRW2000`。我的另一个分区的系统没有被超级保镖保护，运行 `TRW2000` 就没有问题，不过我可不敢在那个分区乱装软件。

`4dd466d0e1e0cc123afbe504da2462ca`
