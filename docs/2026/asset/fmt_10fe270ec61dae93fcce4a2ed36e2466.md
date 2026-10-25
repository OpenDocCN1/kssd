
![](https://bbs.kanxue.com/upload/attach/201405/603227_dbjjnqynebwullf.png)

现在会调用另一个函数。如下：

![](https://bbs.kanxue.com/upload/attach/201405/603227_a1pw2951dt3ouhj.png)

按 F9 运行起来。

![](https://bbs.kanxue.com/upload/attach/201405/603227_kh5kk0ll089oces.png)

因此，比较是验证序列号的一个关键点。如果 `EAX` 和 `EBX` 的值相等，则弹出 Great work 消息框；否则弹出 No luck 消息框。我们注意到代码中有两处 No luck 的消息框提示，只有一个是最后一次的提示。CrackMe 当名称中包含数字的时候会提示 No luck 消息框，当序列号错误的时候还会提示 No luck 消息框。

![](https://bbs.kanxue.com/upload/attach/201405/603227_q7oujewx3sh5fw4.png)

输入包含数字的名称。单击 OK。

![](https://bbs.kanxue.com/upload/attach/201405/603227_49a9n363l7j5bi3.png)

弹出消息框，单击确定，就会在条件判断处中断下来。

![](https://bbs.kanxue.com/upload/attach/201405/603227_118bra6qiyoigu5.png)

F9 运行以后，又弹出消息框。

![](https://bbs.kanxue.com/upload/attach/201405/603227_lstddetg54zip2d.png)

返回地址为 `4013C1`。

![](https://bbs.kanxue.com/upload/attach/201405/603227_em0gdyt5wnnhuz1.png)

OD 的分析显示该子过程开始于 `40137E`，结束于 `4013C1`。调用 `MessageBox` 后就返回了。

请注意，`4013AC` 的箭头（`>`）表示该处是一个跳转地址。我们顺着红线可以找到跳转来自哪里。

![](https://bbs.kanxue.com/upload/attach/201405/603227_9ol5vpm2bd8yr13.png)

现在我们面对的是更多的比较和条件跳转指令，可以导致弹出 No luck 消息框。我们在这里设置断点。

![](https://bbs.kanxue.com/upload/attach/201405/603227_58bf9iiwfe82uxo.png)

再次让程序运行起来，还是输入刚才的用户名和序列号，单击确定。

![](https://bbs.kanxue.com/upload/attach/201405/603227_hm8hs8auq1inc87.png)

这里的代码会循环检测用户名的所有字母。一旦用户名中包含数字的话，就会跳转到 No luck 消息框处。每检测一个字母，就会中断在 `40138B` 处。按 F9 键就会接着检测下一个字母。

当循环到第 7 次的时候（也就是用户名 `ricnar456` 中的第一个数字 `4` 的时候），跳转就会发生，但是我们可以修改进位标志位 `C` 的值。

![](https://bbs.kanxue.com/upload/attach/201405/603227_4h9w3pvneks1sbk.png)

`JB` 条件跳转是根据进位标志位 `C` 来决定是否跳转的，我们通过双击 `C` 标志位修改其值。

![](https://bbs.kanxue.com/upload/attach/201405/603227_45rzpj27vciesn4.png)

这样，跳转就不会发生了。针对于其余的数字我们同样可以这么做让跳转不发生。

![](https://bbs.kanxue.com/upload/attach/201405/603227_hhtkgcrov7mxn1c.png)

接下来我依然要来比较 `EAX` 和 `EBX` 的值，像之前一样通过修改零标志位 `Z` 来让跳转发生。

![](https://bbs.kanxue.com/upload/attach/201405/603227_afpuygkgkfglypk.png)

按 F9 键。

![](https://bbs.kanxue.com/upload/attach/201405/603227_pjyq31gjnkxpalo.png)

我们不可能通过修改 `C` 和 `Z` 标志位来实现功能注册吧！我们需要通过别的方式来让 CrackMe 通过注册。

选择第一个条件分支（设置一个断点）。

![](https://bbs.kanxue.com/upload/attach/201405/603227_9qs8l3uksxmerrt.png)

这次我们不改变进位标志位 `C`，而是直接简单粗暴地把该指令填为 `NOP`。在该指令上单击鼠标右键，选择 `Assemble`，输入 `NOP`。

![](https://bbs.kanxue.com/upload/attach/201405/603227_1f1xks6wb2eqfie.png)

可以将断点去掉了，只需要按下 F2 键。

![](https://bbs.kanxue.com/upload/attach/201405/603227_m6qczpqgwrahx7p.png)

转到第二个关键跳转处。

![](https://bbs.kanxue.com/upload/attach/201405/603227_hb8rcf2a27cuguo.png)

这里我们需要跳转始终成立，所以我们将 `JE` 改成 `JMP`。

![](https://bbs.kanxue.com/upload/attach/201405/603227_d9zo23l05nmfhs9.png)

![](https://bbs.kanxue.com/upload/attach/201405/603227_rfg11bqlh7garo6.png)

然后还是按 F2 删除断点。然后按 F9 键。输入用户名和序列号。

![](https://bbs.kanxue.com/upload/attach/201405/603227_hv86iuoj1ra2vlr.png)

单击确定。

![](https://bbs.kanxue.com/upload/attach/201405/603227_w9pvz9kbi3i42fv.png)

当前这些修改是针对于内存的，并不会影响可执行文件。如果需要保存修改到可执行文件的话，请执行以下操作：

![](https://bbs.kanxue.com/upload/attach/201405/603227_yjitqxzual10u38.png)

在反汇编窗口的任意位置单击鼠标右键，选择 `Copy to executable` → `All modifications`。然后会显示：

![](https://bbs.kanxue.com/upload/attach/201405/603227_ovn5ydb9tk6ftw6.png)

选择 `Copy all` 确保拷贝我们所做的所有修改。

![](https://bbs.kanxue.com/upload/attach/201405/603227_dt7o3ghh0sya6t3.png)

弹出一个新窗口，再次单击鼠标右键，选择 `Save File`。

![](https://bbs.kanxue.com/upload/attach/201405/603227_bpbiz29vdtgk8oo.png)

将文件保存一个新名字（原来的可执行文件我们还有用处，其实这个时候覆盖也是行不通的，这个时候可执行文件正在被占用）。新的可执行文件我们取名叫 `CRACKME2.EXE`。

![](https://bbs.kanxue.com/upload/attach/201405/603227_xwc804nv3s5yxil.png)

这时候，我们可以关闭 OD 了。

![](https://bbs.kanxue.com/upload/attach/201405/603227_iy8zh8xwnjk3kcb.png)

我们双击打开我们的 `CRACKME2` 来看一看修改的效果。

![](https://bbs.kanxue.com/upload/attach/201405/603227_ji92xwyunlgh3ks.png)

选择 `Help` → `Register`。

![](https://bbs.kanxue.com/upload/attach/201405/603227_798mpdtnlzg06pu.png)

单击 OK。

![](https://bbs.kanxue.com/upload/attach/201405/603227_sc6dlvmxk4hned3.png)

我们成功破解这个 CrackMe！但是这是不够的——以后我们会尝试找到正确的注册码，而非通过对该程序打补丁的方式来通过注册。我们还有很多的东西需要学习。

（下面的一点点内容是针对于 Windows 9x 系列系统的，考虑到 Windows 9x 已经淘汰很久了。连 XP 系统前不久都已经退役了。这里就不做翻译了。）

注：本帖由看雪论坛志愿者 PEstone 重新将附件整理排版，若和原文有出入，以原作者附件为准

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [使用OllyDbg从零开始Cracking 第九章-基本概念.doc](javascript:void(0);) （908.13kb，1106次下载）
- [1.png](javascript:void(0);) （15.27kb，18次下载）
- [2.png](javascript:void(0);) （8.34kb，4次下载）
- [3.png](javascript:void(0);) （25.36kb，29次下载）
- [4.png](javascript:void(0);) （8.48kb，5次下载）
- [5.png](javascript:void(0);) （27.44kb，20次下载）
- [6.png](javascript:void(0);) （7.46kb，3次下载）
- [7.png](javascript:void(0);) （27.51kb，9次下载）
- [8.png](javascript:void(0);) （7.54kb，3次下载）
- [9.png](javascript:void(0);) （15.33kb，4次下载）
- [10.png](javascript:void(0);) （18.56kb，12次下载）
- [11.png](javascript:void(0);) （33.12kb，13次下载）
- [12.png](javascript:void(0);) （7.69kb，3次下载）
- [13.png](javascript:void(0);) （12.79kb，8次下载）
- [14.png](javascript:void(0);) （3.91kb，4次下载）
- [15.png](javascript:void(0);) （8.09kb，4次下载）
- [16.png](javascript:void(0);) （14.46kb，3次下载）
- [17.png](javascript:void(0);) （13.68kb，7次下载）
- [18.png](javascript:void(0);) （26.78kb，10次下载）
- [19.png](javascript:void(0);) （13.79kb，14次下载）
- [20.png](javascript:void(0);) （14.04kb，17次下载）
- [21.png](javascript:void(0);) （8.52kb，3次下载）
- [22.png](javascript:void(0);) （7.00kb，3次下载）
- [23.png](javascript:void(0);) （2.37kb，3次下载）
- [24.png](javascript:void(0);) （4.43kb，3次下载）
- [25.png](javascript:void(0);) （8.10kb，4次下载）
- [26.png](javascript:void(0);) （11.97kb，8次下载）
- [27.png](javascript:void(0);) （14.16kb，11次下载）
- [28.png](javascript:void(0);) （12.68kb，3次下载）
- [29.png](javascript:void(0);) （6.89kb，3次下载）
- [30.png](javascript:void(0);) （13.44kb，4次下载）
- [31.png](javascript:void(0);) （12.53kb，4次下载）
- [32.png](javascript:void(0);) （10.61kb，8次下载）
- [33.png](javascript:void(0);) （9.32kb，4次下载）
- [34.png](javascript:void(0);) （16.82kb，4次下载）
- [35.png](javascript:void(0);) （15.75kb，6次下载）
- [36.png](javascript:void(0);) （20.33kb，6次下载）
- [37.png](javascript:void(0);) （16.42kb，11次下载）
- [38.png](javascript:void(0);) （152.58kb，10次下载）
- [39.png](javascript:void(0);) （19.48kb，4次下载）
- [40.png](javascript:void(0);) （14.25kb，3次下载）
- [41.png](javascript:void(0);) （4.80kb，4次下载）
- [42.png](javascript:void(0);) （13.63kb，3次下载）
- [43.png](javascript:void(0);) （3.10kb，3次下载）
- [44.png](javascript:void(0);) （7.19kb，3次下载）
- [45.png](javascript:void(0);) （5.49kb，3次下载）
- [46.png](javascript:void(0);) （8.72kb，5次下载）
- [47.png](javascript:void(0);) （2.18kb，3次下载）
- [48.png](javascript:void(0);) （6.83kb，3次下载）
- [49.png](javascript:void(0);) （7.59kb，4次下载）
